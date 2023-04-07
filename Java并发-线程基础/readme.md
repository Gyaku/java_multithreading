##线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?
##通常线程有哪几种使用方式?
##基础线程机制有哪些?
##线程的中断方式有哪些?
##线程的互斥同步方式有哪些? 如何比较和选择?
##线程之间有哪些协作方式?



##线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?

新建（new）

可运行（runnable）

阻塞（blocking）

无限等待（waiting）

限期等待（timed waiting）

死亡（terminated）

start()

synchronized/Lock

Object.wait()

Object.notify()

Thread.join()

Thread.sleep()

线程结束/异常

##通常线程有哪几种使用方式?

三种方式

1.实现Runnable接口

2.实现Callable接口

实现Runnable接口和Callable接口的类只能当作线程中的一个任务，还需要通过Thread
调用，任务是通过线程驱动从而执行的

3.继承Thread

###使用那种方式更好？

实现接口

Java不支持多继承

继承整个Thread开销过大

##基础线程机制有哪些?

###Executor
    
    CachedThreadPool
    FixedThreadPool
    SingleThreadExecutor

###Daemon

###sleep

###yield

##线程的中断方式有哪些?

    InterruptedExcuption
    interrupt()
    interrupted()
    
    Executor
    shutdown()
    shutdownNow()
    submit()
    future.cancel

##线程的互斥同步方式有哪些? 如何比较和选择?

|方式|synchronized|ReentrantLock|
|---|---|---|
|实现|JVM|JDK JUC|
|性能|新版优化
|等待可中断|否|是
|公平锁|否|否/是
|锁绑定多个条件|无|同时绑定多个condition

等待可中断： 当持有锁的线程长时间不释放锁时，
正等待的线程可以放弃等待，去处理其他任务

公平锁：多个线程等待同一个锁，
必须按照等待的顺序依次获得锁

优先选择synchronized,JVM实现，JVM
原生都支持。ReentrantLock不是所有JDK版本都支持。
synchronized不用担心没有释放锁导致的死锁问题，JVM会确保锁的释放

    
##线程之间有哪些协作方式?

####join()

####wait()  notify()  notifyAll()

属于Object

必须在同步方法或同步块中使用

#####wait()与sleep()的区别

wait()属于Object，sleep()是Thread类的静态方法

wait()需要释放锁（让其他线程将其唤醒），sleep()不会释放锁

    
####await() signal()  signalAll()

JUC 提供Condition类实现线程之间的协调，
可在Condition上调用await()使线程等待
其他线程调用signal()或signalAll()唤醒等待的线程

相比wait()，await() 可以指定等待条件

###线程间的协作
