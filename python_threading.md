Title: Python多线程同步教程
Meta: 多线程给我们带来的好处是可以并发的执行多个任务，特别是对于I/O密集型的业务，使用多线程，可以带来成倍的性能增长。
Date: 2017-08-28
Tags: Python,threading,sync,同步
Category: python库
Slug: python_threading
Author: 笨熊

## 概述
- 多线程给我们带来的好处是可以并发的执行多个任务，特别是对于I/O密集型的业务，使用多线程，可以带来成倍的性能增长。
- 可是当我们多个线程需要修改同一个数据，在不做任何同步控制的情况下，产生的结果往往是不可预料的，比如两个线程，一个输出hello，一个输出world，实际运行的结果，往往可能是一个是hello world，一个是world hello。
- python里提供了多个用于控制多线程同步的同步原语，这些原语，包含在python的标准库threading.py当中。我今天简单的介绍一下python里的这些控制多线程同步的原语，包括：Locks、RLocks、Semaphores、Events、Conditions和Barriers，你也可以继承这些类，实现自己的同步控制原语。

## Lock(锁)
- Locks是python里最简单的同步原语，只包括两个状态：locked和unlocked，刚创建时状态是unlocked。Locks有两个方法，acquire和release。acquire方法加锁，release方法释放锁，如果acquire枷锁失败，则阻塞，表明其他线程已经加锁。release方法只有当状态是locked调用方法True，如果是unlocked状态，调用release方法会抛出RunTimeError异常。例如代码：

        from threading import Lock, Thread
        lock = Lock()
        g = 0
        
        def add_one():
           """
           Just used for demonstration. It’s bad to use the ‘global’
           statement in general.
           """
           global g
           lock.acquire()
           g += 1
           lock.release()
        
        def add_two():
           global g
           lock.acquire()
           g += 2
           lock.release()
        
        threads = []
        for func in [add_one, add_two]:
           threads.append(Thread(target=func))
           threads[-1].start()
        
        for thread in threads:
           """
           Waits for threads to complete before moving on with the main
           script.
           """
           thread.join()
    
        print(g)

- 最终输出的结果是3，通过Lock的使用，虽然在两个线程中修改了同一个全局变量，但两个线程是顺序计算出结果的。

## RLock(循环锁)
- 上面的Lock对象虽然能达到同步的效果，但是无法得知当前是那个线程获取到了锁。如果锁没被释放，则其他获取这个锁的线程都会被阻塞住。如果不想阻塞，可以使用RLock，例如：
    
        # 使用Lock
        import threading
        num = 0
        lock = Threading.Lock()
        
        lock.acquire()
        num += 1
        lock.acquire() # 这个地方阻塞
        num += 2
        lock.release()
        
        # 使用RLock
        lock = Threading.RLock()
        lock.acquire()
        num += 3
        lock.acquire() # 这不会阻塞
        num += 4
        lock.release()
        lock.release() # 这个地方注意是释放两次锁

## Semaphores
- Semaphores是个最简单的计数器，有两个方法acquire()和release()，如果有多个线程调用acquire()方法，acquire()方法会阻塞住，每当调用次acquire方法，就做一次减1操作，每当release()方法调用此次，就加1，如果最后的计数数值大于调用acquire()方法的线程数目，release()方法会抛出ValueError异常。下面是个生产者消费者的示例。

        import random, time
        from threading import BoundedSemaphore, Thread
        max_items = 5
        container = BoundedSemaphore(max_items)
        def producer(nloops):
            for i in range(nloops):
                time.sleep(random.randrange(2, 5))
                print(time.ctime(), end=": ")
                try:
                    container.release()
                    print("Produced an item.")
                except ValueError:
                    print("Full, skipping.")
        def consumer(nloops):
            for i in range(nloops):
                time.sleep(random.randrange(2, 5))
                print(time.ctime(), end=": ")
                if container.acquire(False):
                    print("Consumed an item.")
                else:
                    print("Empty, skipping.")
        threads = []
        nloops = random.randrange(3, 6)
        print("Starting with %s items." % max_items)
        threads.append(Thread(target=producer, args=(nloops,)))
        threads.append(Thread(target=consumer, args=(random.randrange(nloops, nloops+max_items+2),)))
        for thread in threads:  # Starts all the threads.
            thread.start()
        for thread in threads:  # Waits for threads to complete before moving on with the main script.
            thread.join()
        print("All done.")

![](http://7xpx6h.com1.z0.glb.clouddn.com/1--BjV8tcNk4TzprX-5NJnhg.gif)

- threading模块还提供了一个Semaphore对象，它允许你可以任意次的调用release函数，但是最好还是使用BoundedSemaphore对象，这样在release调用次数过多时会报错，有益于查找错误。Semaphores最长用来限制资源的使用，比如最多十个进程。

## Events
- event可以充当多进程之间的通信工具，基于一个内部的标志，线程可以调用set()和clear()方法来操作这个标志，其他线程则阻塞在wait()函数，直到标志被设置为True。下面的代码展示了如何利用Events来追踪行为。

        import random, time
        from threading import Event, Thread
        
        event = Event()
        
        def waiter(event, nloops):
            for i in range(nloops):
            print(“%s. Waiting for the flag to be set.” % (i+1))
            event.wait() # Blocks until the flag becomes true.
            print(“Wait complete at:”, time.ctime())
            event.clear() # Resets the flag.
            print()
        
        def setter(event, nloops):
            for i in range(nloops):
            time.sleep(random.randrange(2, 5)) # Sleeps for some time.
            event.set()
        
        threads = []
        nloops = random.randrange(3, 6)
        
        threads.append(Thread(target=waiter, args=(event, nloops)))
        threads[-1].start()
        threads.append(Thread(target=setter, args=(event, nloops)))
        threads[-1].start()
        
        for thread in threads:
            thread.join()
        
        print(“All done.”)
![](http://7xpx6h.com1.z0.glb.clouddn.com/1--BjV8tcNk4TzprX-5NJnhg.gif)

## Conditions
- conditions是比events更加高级一点的同步原语，可以用户多线程间的通信和通知。比如A线程通知B线程资源已经可以被消费。其他的线程必须在调用wait()方法前调用acquire()方法。同样的，每个线程在资源使用完以后，要调用release()方法，这样其他线程就可以继续执行了。下面是使用conditions实现的一个生产者消费者的例子。

        import random, time
        from threading import Condition, Thread
        condition = Condition()
        box = []
        def producer(box, nitems):
            for i in range(nitems):
                time.sleep(random.randrange(2, 5))  # Sleeps for some time.
                condition.acquire()
                num = random.randint(1, 10)
                box.append(num)  # Puts an item into box for consumption.
                condition.notify()  # Notifies the consumer about the availability.
                print("Produced:", num)
                condition.release()
        def consumer(box, nitems):
            for i in range(nitems):
                condition.acquire()
                condition.wait()  # Blocks until an item is available for consumption.
                print("%s: Acquired: %s" % (time.ctime(), box.pop()))
                condition.release()
        threads = []
        nloops = random.randrange(3, 6)
        for func in [producer, consumer]:
            threads.append(Thread(target=func, args=(box, nloops)))
            threads[-1].start()  # Starts the thread.
        for thread in threads:
            thread.join()
        print("All done.")

![](http://7xlrq6.com1.z0.glb.clouddn.com/1-tTYcI9yP6XrnZcFSRGA_vw.gif)

- conditions还有其他很多用户，比如实现一个数据流API，当数据准备好了可以通知其他线程去处理数据。

## Barriers
- barriers是个简单的同步原语，可以用户多个线程之间的相互等待。每个线程都调用wait()方法，然后阻塞，直到所有线程调用了wait(),然后所有线程同时开始运行。例如：

        from random import randrange
        from threading import Barrier, Thread
        from time import ctime, sleep
        
        num = 4
        b = Barrier(num)
        names = [“Harsh”, “Lokesh”, “George”, “Iqbal”]
        
        def player():
            name = names.pop()
            sleep(randrange(2, 5))
            print(“%s reached the barrier at: %s” % (name, ctime()))
            b.wait()
            
        threads = []
        print(“Race starts now…”)
        
        for i in range(num):
            threads.append(Thread(target=player))
            threads[-1].start()
        for thread in threads:
            thread.join()
        print()
        print(“Race over!”)
![](http://7xlrq6.com1.z0.glb.clouddn.com/1-CYnUEjVV8Ztq1dwwQ9EnHA.gif)

## 总结
- 多线程同步，说难也难，说不难也很容易，关键是要看你的业务场景和解决问题的思路，尽量降低多线程之间的依赖，理清楚业务流程，选择合适的方法，则事尽成。


