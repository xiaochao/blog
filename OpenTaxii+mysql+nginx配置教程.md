Title: OpenTaxii+mysql+nginx 配置教程
Meta: OpenTaxii+mysql+nginx 配置教程
Date: 2017-07-07
Tags: opentaxii,mysql,nginx,stix,威胁情报订阅
Category: 威胁情报
Slug: opentaxii-install-nginx-mysql
Author: 笨熊

# OpenTaxii配置教程
## 安装过程中注意点
- 官方给出的service.yml配置文件中默认使用https作为请求方式，所以如果不能提供https服务的话，请注释掉
- opentaxii默认安装是0.1.9，但是0.1.9有个关于格式的问题，所以，如果实际环境中发现无法push 和pull 数据，请使用0.1.8版本
- 经测试，opentaxii+mysql 不如 opentaxii+sqllite的性能，很是奇怪。所以实际使用过程中可以都测试看看。

## 环境需求：
python版本：2.7,3.4
pip:python包管理工具
mysql: 无版本需求
nginx: 无版本需求

## 安装步骤：
1. pip install taxii gunicorn
2. 创建mysql数据库:taxii_data,taxii_auth
3. 修改配置文件taxii.yml文件，9行和15行里的mysql连接信息配置
4. 修改配置文件taxii.yml第一行域名为实际的域名
5. 运行命令export OPENTAXII_CONFIG=taxii.yml文件的路径
6. 根据实际的情况，修改service.yml和collections.yml
7. 创建服务：opentaxii-create-services -c services.yml
8. 创建集合：opentaxii-create-collections -c collections.yml
9. 添加一个用户名密码：opentaxii-create-account -u username -p password
10. 运行opentaxii(注意修改日志路径):gunicorn opentaxii.http:app --workers 2 --log-level debug --log-file /home/worker/log/opentaxii.log --timeout 30 --bind 0.0.0.0:19000 -D

## 测试安装是否成功
1. pip install cabby
2. taxii-discovery --path http://localhost:9000/services/discovery-a：discovery-a为service.xml里配置的discovery，没有输出错误信息，则成功
3. taxii-collections --path http://localhost:9000/services/collection-management-a:collection-management-a为service.xml里配置的collection_management，没有输出错误信息，则成功

## HTTPS配置
1. 准备好https证书
2. 生成dbparam：openssl dhparam -out dhparam.pem 2048
3. nginx 增加server配置如下(注意修改 cert.pem,cert.key,ssl_dhparam,access_log,error_log配置项的文件路径)：

        server {
            listen 443 ssl;
            server_name taxii.example.com;
            ssl_certificate      /usr/local/nginx/ssl/cert.pem;
            ssl_certificate_key  /usr/local/nginx/ssl/cert.key;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
            ssl_prefer_server_ciphers on;
            ssl_session_cache shared:SSL:10m;
        
            access_log  /var/log/nginx/taxii_access.log;
            error_log  /var/log/nginx/taxii_error.log;
            location / {
                proxy_pass http://127.0.0.1:19000;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
4. 重启nginx


