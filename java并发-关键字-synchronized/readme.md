##Synchronized可以作用在哪里? 分别通过对象锁和类锁进行举例。
对象锁：包括方法锁（默认锁对象是this，当前实例对象）
和同步代码块锁（自己指定对象，可以是this，也可以是自定义的锁）

类锁：指Synchronized修饰的静态方法或指定锁对象为Class对象

##Synchronized本质上是通过什么保证线程安全的? 
分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。

####加锁和释放锁的原理

监视器Monitor有两个指令Monitorenter和Monitorexit，会让对象在执行，
使其锁计数器+1或-1。

一个对象在同一时间只能和一个Monitor（锁）关联，一个Monitor同一时间只能被一个线程获得。

任意线程对对象（Object）的访问，首先要获得对象的监视器（Monitor），
如果获取失败进入同步队列（Synchronized），线程状态变为阻塞（Blocked），
当对象的监视器占有者释放后，在同步队列中的线程有机会重新获得监视器

####可重入原理：加锁次数计数器

可重入：若一个程序或子程序在运行时被打断，操作系统执行了其他代码，
而该代码又调用了该子程序，且不出现错误。re-entrant

可重入锁：递归锁，一个线程在外层方法获得了对象的锁，再进入该线程的内层方法时
直接获得锁（锁对象时同一个实例对象或class），不会因为之前没有释放锁而阻塞。

Synchronized具有可重入性：在同一锁程中，每一个对象拥有一个Monitor计数器，
当线程获取该对象的锁后，Monitor计数器+1，线程释放锁后，Monitor计数器-1，
线程不需要再次获得同一把锁。

####保证可见性原理：内存模型（JMM）和happens-before规则
Synchronized的happens-before：同一个监视器的解锁，happens-before于对该监视器的加锁

##Synchronized由什么样的缺陷? Java Lock是怎么弥补这些缺陷的。

####缺陷
1.效率低
2.不够灵活：加锁和释放锁的时机单一，每个锁仅有一个单一的条件(对象)
3.无法知道是否成功获取锁

####Lock弥补这些缺陷
1.lock()
2.unlock()
3.tryLock()
4.tryLock(long,TimeUtil)

Synchronized只有锁与一个条件相关联（是否获得锁），不灵活。
Condition和Lock解决了这个问题

多线程竞争一个锁时，其余未获得锁的线程只能不停的尝试获取锁，不能中断。
高并发的情况下会导致性能降低。
ReentrantLock的lockInterruptibly()可以优秀考虑响应中断。
一个线程等待时间过长，他可以中断自己，然后ReentrantLock响应这个中断，
不再让这个线程等待。有了这个机制ReentrantLock不会像Synchronized那样
产生死锁
    
    

##Synchronized和Lock的对比，和选择?

·存在层次上

    Synchronized：Java关键字，在JVM层面上
    Lock：是一个接口
    
·锁的释放
    
    Synchronized：1、获取锁的线程执行完同步代码，释放锁；
                2、线程执行出现异常，JVM会让线程释放锁
    Lock：在finally中必须释放，不然容易造成线程死锁
    
·锁的获取

    Synchronized：如果线程A获得锁，线程B就得等待，如果线程A阻塞，线程B会一直等待
    Lock：尝试获得锁，线程可以不用一直等待
    
·锁的释放（死锁产生）

    Synchronized：出现异常时JVM会自动释放所占有的锁，所有不会出现死锁
    Lock：发生异常时不会，不会主动释放锁，需要手动通过unlock释放锁，所以会出现死锁
    
·锁的状态

    Synchronized：无法判断
    Lock：可判断
    
·锁的类型

    Synchronized：可重入，不可中断，非公平
    Lock：可重入，可中断，可公平

    
·性能

    Synchronized：少量同步
    Lock：大量同步
    
    Lock可以提高多个线程进行读操作的效率（通过readwriteLock实现读写分离）。
    ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被interrupt的同步（Synchronized的同步不能被interrupt）。
    
    在资源竞争不是很激烈的情况下，Synchronized的性能优于ReentrantLock。
    但是当资源竞争很激烈时，Synchronized的性能会下降几十倍，而ReentrantLoack的性能还能维持常态

    
·调度

    Synchronized：使用Object的wait，notigy，notifyAll调度机制
    Lock：使用Condition惊喜线程间的调度？？？不明白

    
·用法

    Synchronized：在需要同步的对象中加入此控制，Synchronized可以作用于方法上，也可以加在需要同步的代码块中，括号中表示需要锁的对象
    Lock：一般使用ReentrantLock类作为锁。在加锁和解锁处使用lock()和unlock()指出，一般在finally中写unlock()防止死锁。

    
·底层实现

    Synchronized：底层使用指令码方式来控制锁，映射成字节码文件新增两个指令：monitorenter和monitorexit。
    当线程执行遇到monitorenter指令时，尝试获得内置锁，如果成功获得锁，则锁计数器+1，没有获得锁线程阻塞。
    当线程执行遇到monitorexit指令时，锁计数器-1，当计数器为0时释放锁
    Lock：底层是CAS乐观锁，依赖AQS类，把所有请求线程构成一个请求队列。而对该队列的操作通过Lock-Free（CAS）操作？？？


##Synchronized在使用时有何注意事项?
·锁对象不能为空，因为锁的信息都保存在对象头里

·作用于不宜过大，影响程序执行速度，控制范围过大，编写代码容易出错

·避免死锁

·在能选出的情况下，尽量不要选择Lock和Synchronized关键字，用JUC包中的类，
如果不使用JUC包中的类，在满足业务的情况下，可以使用Synchronized关键字，
因为代码量少，避免出错


##Synchronized修饰的方法在抛出异常时,会释放锁吗?
会

##多个线程等待同一个Synchronized锁的时候，JVM如何选择下一个获取锁的线程?
非公平，抢占式

如果刚好有线程请求获取锁，直接得到刚释放的锁。否则从同步队列中依次获取

新来的线程有可能立即获得监视器，而在等待区中等候已久的线程继续等待，
有利于提高性能，但是可能导致饥饿现象

##Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?
JVM中的Monitorenter和Monitorexit依赖于底层操作系统的Mutex Lock来实现。
使用Mutex Lock需要将当前线程挂起并从用户态切换到核心态来执行，这种切换代价十分昂贵。
然而现实中大部分情况下，同步方法是运行在单线程环境，如果每次都调用Mutex Lock
那么将严重影响程序性能。

jdk1.6对锁的实现引入了大量优化

Lock Coarsening（锁粗化）

    减少不必要的连续的lock和unlock操作，将多个不必要的锁扩展成一个更大范围的锁

Lock Elimination（锁消除）
    
    虚拟机即时编译器在运行时，对一些代码要求同步，但是对
    被检测到不可能存在共享数据竞争的锁进行消除。
    锁消除的主要判断依据是逃逸分析的数据。JVM会判断一段
    程序中的同步 明显不会逃逸出去 从而被其他线程访问到，
    那JVM就把这些数据当作栈上数据对待，认为这些数据是线程
    独有的，不需要加同步。此时就会进行锁消除（无锁了）
    
    
Lightweight Locking（轻量化锁）

Biased Locking（偏向锁）

Adaptive Spinning（适应性自旋）
    
    当线程在获取轻量级锁的过程中追寻CAS操作失败，在进入
    于monitor相关的操作系统重量级锁前会进入忙等待(spinning)
    然后再次尝试，当尝试一定此时后仍然没有成功，调用
    minitor关联的samephore(互斥锁)进入阻塞状态

##我想更加灵活的控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办?

##什么是锁的升级和降级? 
jdk1.6 Synchronized同步锁一共有4种状态：无锁、偏向锁、轻量级锁、重量级锁，
他们会随着竞争情况逐渐升级。锁可以升级但不能降级，目的为了提供锁和释放锁的效率

##什么是JVM里的偏向锁、轻量级锁、重量级锁?
偏向锁：为了在无锁竞争的情况下在获取锁的过程中执行不必要的CAS操作

轻量级锁：假设在真实情况下程序中的大部分同步代码处于无锁竞争状态（单线程执行环境）
可以避免调用操作系统的重量级互斥锁，取而代之是在monitorenter和monitorexit中
只需要一条CAS原子指令就可以完成锁的获取和释放。当存在锁竞争的情况下，执行CAS指令
失败的线程操作系统互斥锁进入阻塞状态，当锁被释放时唤醒。

重量级锁：当一个线程获取锁后，其余所有等待获取该锁的线程都处于阻塞状态

锁|优点|缺点|使用场景
---|---|---|---
偏向锁|加锁和解锁不需要进行CAS操作，没有额外的性能消耗|如果线程间存在锁竞争，会带来额外的锁撤销消耗|适用于只有一个线程访问同步块的场景
轻量级锁|竞争的线程不会阻塞，提供了响应速度|如果线程始终等不到竞争的锁，自旋会消耗cpu性能|追求响应时间，同步块执行速度非常快
重量级锁|线程竞争不适用于自旋，不会消耗cpu|线程阻塞，响应时间慢，多线程下，频繁的获取释放锁，带来巨大的性能消耗|最求吞吐量，同步块执行时间较长


##不同的JDK中对Synchronized有何优化?
jdk1.6对锁的实现引入了大量的优化，锁粗化，锁消除，轻量级锁，偏向锁，适应性自旋


