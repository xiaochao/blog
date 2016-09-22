Title: 开源人脸识别seetaface入门教程(一)
Date: 2016-09-22
Tags: seetaface,入门,教程,编译方法,人脸识别
Category: 人脸识别
Slug: seetaface-tutorial
Author: 笨熊


### 简述
- seetaface由中科院计算所山世光研究员带领的人脸识别研究组研发。代码基于C++实现，不依赖第三方库。然而，目前开源的代码，是在windows vs上编译的，对于我们这帮mac/linux用户来说，用起来还是挺麻烦的。经过这几天的学习，对seetaface总算有了全面的了解。下面，听我娓娓道来。
- 注意：本文章不涉及代码逻辑和原理，只是教大家如何使用seetaface做人脸识别。

### 引擎
#### FaceDetection
- 人脸识别模块，用于识别出照片中的人脸，染回每个人脸的坐标和人脸总数。
#### FaceAlignment
- 特征点识别模块，主要识别两个嘴角、鼻子、两个眼睛五个点的坐标。测试下来，发现图片模糊时，识别不准。
#### FaceIdentification
- 人脸比较模块，根据官方的说法，先提取特征值，然后比较。给出的测试程序是seetaface提取人脸的特征值和caffe训练库里的人脸做对比。

*以下教程都是在MacOSX编译运行通过。使用cmake和make编译*

*以下的编译方法是把FaceDetect测试程序也编译了，而测试程序是依赖opencv的，所以，在这之前，确认opencv是否安装*
### 人脸识别教程
#### 编译
##### 由于代码是在windows平台编译的，所以，这地方要做些修改。
1. 进入FaceDetection目录
2. 修改include/common.h，修改38行

		#ifdef SEETA_EXPORTS
		#define SEETA_API __declspec(dllexport)
		#else
		#define SEETA_API __declspec(dllimport)
		#endif
	为
	
		#if defined _WIN32
		#ifdef SEETA_EXPORTS
		#define SEETA_API __declspec(dllexport)
		#else
		#define SEETA_API __declspec(dllimport)
		#endif
		#else
		#define SEETA_API
		#endif
3. 修改include/feat/surf_feature_map.h文件，在前面加上#include <cstring>
4. 修改include/util/image_pyramid.h文件，在前面加上#include <cstring>
5. 修改src/feat/surf_feature_map.cpp文件，在前面加上#include <cmath>
6. 增加CMakeLists.txt，内容如下：
		
		cmake_minimum_required(VERSION 3.3)

		project(seeta_facedet_lib)

		# Build options
		option(BUILD_EXAMPLES  "Set to ON to build examples"  ON)
		option(USE_OPENMP      "Set to ON to build use openmp"  ON)

		# Use C++11
		set(CMAKE_CXX_STANDARD 11)

		set(CMAKE_CXX_STANDARD_REQUIRED ON)
		message(STATUS "C++11 support has been enabled by default.")

		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.1")

		# Use OpenMP
		if (USE_OPENMP)
    		find_package(OpenMP QUIET)
    		if (OpenMP_FOUND)
        		message(STATUS "Use OpenMP")
        		add_definitions(-DUSE_OPENMP)
        		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
    		endif()
		endif()

		include_directories(include)

		set(src_files
    		src/util/nms.cpp
    		src/util/image_pyramid.cpp
    		src/io/lab_boost_model_reader.cpp
    		src/io/surf_mlp_model_reader.cpp
    		src/feat/lab_feature_map.cpp
    		src/feat/surf_feature_map.cpp
    		src/classifier/lab_boosted_classifier.cpp
    		src/classifier/mlp.cpp
    		src/classifier/surf_mlp.cpp
    		src/face_detection.cpp
    		src/fust.cpp
    		)

		add_library(face_detect SHARED ${src_files})
		set(facedet_required_libs face_detect)

		if (BUILD_EXAMPLES)		
   			message(STATUS "Build with examples.")
    		find_package(OpenCV)
    		if (NOT OpenCV_FOUND)
        		message(WARNING "OpenCV not found. Test will not be built.")
    		else()
    			include_directories(${OpenCV_INCLUDE_DIRS})
        		list(APPEND facedet_required_libs ${OpenCV_LIBS})

        		add_executable(facedet_test src/test/facedetection_test.cpp)
        		target_link_libraries(facedet_test ${facedet_required_libs})
    		endif()
		endif()
7. 建立build目录，mkdir build
8. 编译，cd build && cmake .. && make
9. 当前目录下生成可执行文件

#### 运行
1. 执行完make命令以后，当前的目录下会生成一个可执行文件facedet_test
2. 由于默认的程序读取的是当前路径下的test_image.jpg和seeta_fd_frontal_v1.0.bin，test_image.jpg是人脸图片，seeta_fd_frontal_v1.0是识别的引擎。
3. 确保以上的两个文件在当前路径下存在了，既可以./facedet_test运行了。
4. 你可以修改位于src/test目录下的文件，来达到自己的目的。

#### 使用
##### 我们可以参考src/test/facedetection_test.cpp这个测试程序，来达到我们人脸识别的目的
1. 头文件
		
		#include "opencv2/highgui/highgui.hpp"
		#include "opencv2/imgproc/imgproc.hpp"
		#include "face_detection.h"
	opencv头文件主要用来加载图像，face_detection.h是人脸识别的主要程序。
	
2. 加载人脸识别引擎

		seeta::FaceDetection detector(‘seeta_fd_frontal_v1.0’);
		
3. 设置最小人脸大小

		detector.SetMinFaceSize(40);
		
	这个根据实际情况调整，图片中，人脸越大，这个值也越大，因为这个值越小，人脸识别速度越慢。
	
4. 识别图片中的人脸
		
		std::vector<seeta::FaceInfo> faces = detector.Detect(img_data);
	在这之前，需要对图片进行处理，这里略过
5. 输出人脸识别的结果

		for (int32_t i = 0; i < num_face; i++) {
	        face_rect.x = faces[i].bbox.x;
	        face_rect.y = faces[i].bbox.y;
	        face_rect.width = faces[i].bbox.width;
	        face_rect.height = faces[i].bbox.height;
	        cv::rectangle(img, face_rect, CV_RGB(0, 0, 255), 4, 8, 0);
	    }
    faces[i].bbox.x; faces[i].bbox.y;是人脸的左上角坐标。faces[i].bbox.width;faces[i].bbox.height;是人脸的长和宽。 
    
### 结语
seetaface的确是个很好用的人脸识别库，调用、编译都很简单，但是由于文档的缺少，所以刚开始看的时候，会比较乱，不知道如何下手。本片文章主要介绍了FaceDetect的使用，接下来我会讲解如何识别人脸的特征点，也就是嘴、鼻子、眼。敬请期待。