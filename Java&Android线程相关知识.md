#Java&Android 线程

## 一.java多线程-概念&创建启动&中断&守护线程&优先级&线程状态

### 1.什么是线程以及多线程与进程的区别
		进程：正在运行的程序，是系统进行资源分配和调用的独立单位。每一个进程都有它自己的内存空间和系统资源。
		线程：是进程中的单个顺序控制流，是一条执行路径一个进程如果只有一条执行路径，则称为单线程程序。一个进程如果有多条执行路径，则称为多线程程序。
	
### 2.多线程的创建和启动
		继承Thread
		实现runnable

### 3.中断线程和守护线程以及线程优先级
		3.1 中断线程的几种方式
			1.设置标志位
			2.Stop  已经被弃用
			3.调用interrupt方法
       
		3.2 interrupt 的两种正确处理方式
``` java 
		public void run() {   
    		try {   
        		/*  
         		* 不管循环里是否调用过线程阻塞的方法如sleep、join、wait，这里还是需要加上  
         		* !Thread.currentThread().isInterrupted()条件，虽然抛出异常后退出了循环，显  
         		*得用阻塞的情况下是多余的，但如果调用了阻塞方法但没有阻塞时，这样会更安全、更及时。  
        		 */  
        		while (!Thread.currentThread().isInterrupted()&& more work to do) {   
            do more work    
        		}   
    		} catch (InterruptedException e) {   
        	//线程在wait或sleep期间被中断了   
    		} finally {   
        	//线程结束前做一些清理工作   
    		}   
		} 
```
---	
``` java
			public void run() {  
			在线程wait、sleep阻塞的时候,调用interrupt 会抛出异常并且重置线程的中断状态 
			    while (!Thread.currentThread().isInterrupted()&& more work to do) {   
			        try {   
			            ...   
			            sleep(delay);   
			        } catch (InterruptedException e) {   
			            Thread.currentThread().interrupt();//重新设置中断标示   
			        }   
			    }   
			}  
```

		3.3 interrupt相关的三种方法
			void interrupt()：向线程发送中断请求，线程的中断状态将会被设置为true，如果当前线程被一个sleep调用阻塞，那么将会抛出interrupedException异常。
			static boolean interrupted测试当前线程（当前正在执行命令的这个线程）是否被中断。注意这是个静态方法，调用这个方法会产生一个副作用那就是它会将当前线程的中断状态重置为false。
			boolean isInterrupted()：判断线程是否被中断，这个方法的调用不会产生副作用即不改变线程的当前中断状态。
		3.4 调用interrupt不会抛出异常的其他几种情况
			java.io中的异步socket I/O 
			读写scket的时候，InputStream和OutputStream的read和write方法会阻塞等待，但不会响应java中断。不过，调用Socket的close方法后，被阻塞线程会抛出SocketException异常。
		利用Selector实现的异步I/O 
			如果线程被阻塞于Selector.select（在java.nio.channels中），调用wakeup方法会引起ClosedSelectorException异常。 
		锁获取 
			如果线程在等待获取一个内部锁，我们将无法中断它。但是，利用Lock类的lockInterruptibly方法，我们可以在等待锁的同时，提供中断能力。 
		3.5 守护线程
			守护线程的唯一作用就是为其他线程提供服务.
			计时线程就是一个典型的例子，它定时地发送“计时器滴答”信号告诉其他线程去执行某项任务。当只剩下守护线程时，虚拟机就退出了，因为如果只剩下守护线程，程序就没有必要执行了。另外JVM的垃圾回收、内存管理等线程都是守护线程。还有就是在做数据库应用时候，使用的数据库连接池，连接池本身也包含着很多后台线程，监控连接个数、超时时间、状态等等。最后还有一点需要特别注意的是在java虚拟机退出时Daemon线程中的finally代码块并不一定会执行
		3.6 线程优先级
			默认5,可设置1-10

### 4.线程的各种状态
		新建状态（New）：新创建了一个线程对象。
		就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
		运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
		阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
        - 等待阻塞（WAITING）：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
        - 同步阻塞（Blocked）：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
        - 超时阻塞（TIME_WAITING）：运行的线程执行sleep(long)或join(long)方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。
	 	死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。
	 
### 5.yield()的理解
		sleep 方法使当前运行中的线程睡眠一段时间，进入不可以运行状态，这段时间的长短是由程序设定的，yield方法使当前线程让出CPU占有权，但让出的时间是不可设定的。
		yield()也不会释放锁标志。
		实际上，yield()方法对应了如下操作；先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把CPU的占有权交给次线程，否则继续运行原来的线程，所以yield()方法称为“退让”，它把运行机会让给了同等级的其他线程。

## 二.java多线程同步以及线程间通信详解&消费者生产者模式&死锁&Thread.join()

![image](http://img.blog.csdn.net/20160313164027781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
                                                                                         
### 1.线程同步问题的产生
        多个线程在操作共享的数据（num）
        
### 2.解决线程安全的两种方案
		2.1 synchronized
			同步方法
			同步代码块
		2.2 lock
		 	是java se 5.0 concurrent提供的
		 	ReentrantLock lock = new ReentrantLock(); //参数默认false，不公平锁    
			ReentrantLock lock = new ReentrantLock(true); //公平锁 
	
### 3.线程之间的通信
		3.1 synchronized 关键字等待/通知机制
			notify()/notifyAll(),wait()/wait(long),wait(long,int)只能在同步方法或者同步代码块里面使用
		3.2 条件对象的等待/通知机制
			通过锁的条件对象来实现通知和等待
			Condition conditionObj=ticketLock.newCondition();  
	
### 4.生产者消费者模式
		4.1 单生产者单消费者模式
		4.2 多生产者多消费者模式
		
### 5.线程死锁
		死锁的四个必要条件
		互斥条件：一个资源每次只能被一个进程使用。
		请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
		不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
		循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。 
### 6.Thread.join
		把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B
		
## 三.java&android线程池-Executor框架之ThreadPoolExcutor&ScheduledThreadPoolExecutor浅析
![image](http://img.blog.csdn.net/20160314222910689?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

	Excutor类图结构
![image](http://img.blog.csdn.net/20160314223014236?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 1.Executor框架浅析 
		1.1 Executor框架的两级调度模型
			在上层，java多线程程序通过把应用分为若干个任务，然后使用用户级的调度器（Executor框架）将这些任务映射为固定数量的线程；
			在底层，操作系统内核将这些线程映射到硬件处理器上。
	
		1.2 Executor框架的结构
			任务：包括被执行任务需要实现的接口：Runnable接口或Callable接口
			任务的执行：包括任务执行机制的核心接口Executor，以及继承自Executor的EexcutorService接口。Exrcutor有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor）。
			异步计算的结果：包括接口Future和实现Future接口的FutureTask类
### 2.ThreadPoolExecutor浅析 
```java
	public ThreadPoolExecutor(int corePoolSize,  
	                              int maximumPoolSize,  
	                              long keepAliveTime,  
	                              TimeUnit unit,  
	                              BlockingQueue<Runnable> workQueue,  
	                              ThreadFactory threadFactory) {  
	        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,  
	             threadFactory, defaultHandler);  
	    } 
```
		corePoolSize：线程池的核心线程数，默认情况下，核心线程数会一直在线程池中存活，即使它们处理闲置状态。如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务到来时会执行超时策略，这个时间间隔由keepAliveTime所指定，当等待时间超过keepAliveTime所指定的时长后，核心线程就会被终止。
		maximumPoolSize：线程池所能容纳的最大线程数量，当活动线程数到达这个数值后，后续的新任务将会被阻塞。
		keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。当ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true时，		keepAliveTime同样会作用于核心线程。
		unit：用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有		TimeUnit.MILLISECONDS(毫秒)，TimeUnit.SECONDS(秒)以及TimeUnit.MINUTES(分钟)等。
		workQueue：线程池中的任务队列，通过线程池的execute方法提交Runnable对象会存储在这个队列中。
		threadFactory：线程工厂，为线程池提供创建新线程的功能。ThreadFactory是一个接口，它只有一个方法：Thread newThread（Runnable r）
	
### 3.Java通过Executors提供四种线程池
		newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
		newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
		newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
		newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
### 4.Runnable,Callable,Future,FutureTask之间的关系
		Runnable实现的是void run()方法,无返回值,Runnable可以提交给Thread来包装下，直接启动一个线程来执行
		Callable实现的是 V call()方法，并且可以返回执行结果,Callable则一般都是提交给ExecuteService来执行。
		Future就是对于具体的调度任务的执行结果进行查看，最为关键的是Future可以检查对应的任务是否已经完成，也可以阻塞在get方法上一直等待任务返回结果。Runnable和Callable的差别就是Runnable是没有结果可以返回的，就算是通过Future也看不到任务调度的结果的。
		FutureTask则是一个RunnableFuture<V>，即实现了Runnbale又实现了Futrue<V>这两个接口，另外它还可以包装Runnable和Callable<V>，所以一般来讲是一个符合体了，它可以通过Thread包装来直接执行，也可以提交给ExecuteService来执行，并且还可以通过v get()返回执行结果，在线程体没有执行完成的时候，主线程一直阻塞等待，执行完则直接返回结果。
### 5.ThreadFactory 
		Thread是一个接口,我们可以实现它,他使用了工厂模式,可以使用它替代默认的new Thread，而且在自定义工厂里面，我们能创建自定义化的Thread，并且计数，或则限制创建Thread的数量，给每个Thread设置属性等等.
		
### 6.ThreadLocal 
		线程局部变量
		ThreadLocal.set(new Looper),Looper源码
		ThreadLocal.get(),Handler 源码
	