##  				java 线程的生命周期和状态

#### 概述:

  java 在运行时的生命周期的指定时刻分为6种不同的状态:

			+ **New**  初始状态,线程被构建,但是没有调用start()方法
			+ **Runnable** 运行状态,内部分为Ready和Running两种状态,通过Thread.yield()方式进行转换
			+ **Blocked** 阻塞状态,便是线程阻塞与锁 ,通过调用Synchronized 方法或者 AQS 模块锁转换
   + **Waiting**  表示当前线程需要等待其他线程做出一些特定的动作才能醒来
     - 调用Object.wait(),Object.join().LockSupport.part()方法进行阻塞
     - 调用 Object.notify()  Object.join() LockSupport() 退出此状态
   + **Time_Waiting** 超时等待状态,可以在指定的时间自行返回,
     - Thread.sleep(long),Object.wait(long),Thread.join(long),LockSupport.parkNanos(),LockSupport.parkUtil()方法进去此状态
     - Object.notify() Object.notifyAll()  LokSupport.unpark(Thread) 退出此状态
			+ **Terminated** 终止状态,表示线程已经执行完毕了

