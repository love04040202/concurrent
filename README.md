# concurrent 笔记
# 并发工具类
 - Semaphore
	> Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可

		// 线程池
        ExecutorService exec = Executors.newCachedThreadPool();
        // 只能5个线程同时访问
        final Semaphore semp = new Semaphore(5);
        // 模拟50个客户端访问
        for (int index = 0; index < 50; index++) {
            final int NO = index;
            Runnable run = new Runnable() {
                public void run() {
                    try {
                        // 获取许可
                        semp.acquire();
                        System.out.println("Accessing: " + NO);
                        Thread.sleep((long) (Math.random() * 6000));
                        // 访问完后，释放
                        semp.release();
                        //availablePermits()指的是当前信号灯库中有多少个可以被使用
                        System.out.println("-----------------" + semp.availablePermits()); 
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            exec.execute(run);
        }
        // 退出线程池
        exec.shutdown();
 - CountDownLatch
	> CountDownLatch 在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

	
 - CyclicBarrier
	>	- CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续。 
 	>	- CyclicBarrier就象它名字的意思一样，可看成是个障碍， 所有的线程必须到齐后才能一起通过这个障碍。 
 	>	- CyclicBarrier初始时还可带一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。
		public class TestCyclicBarrier {
		
			private static final int THREAD_NUM = 5;
		
			public static class WorkerThread implements Runnable {
		
				CyclicBarrier barrier;
		
				public WorkerThread(CyclicBarrier b) {
					this.barrier = b;
				}
		
				@Override
				public void run() {
					// TODO Auto-generated method stub
					try {
						System.out.println("Worker's waiting");
						// 线程在这里等待，直到所有线程都到达barrier。
						barrier.await();
						System.out.println("ID:" + Thread.currentThread().getId() + " Working");
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
		
			}
		
			/**
			 * @param args
			 */
			public static void main(String[] args) {
				// TODO Auto-generated method stub
				CyclicBarrier cb = new CyclicBarrier(THREAD_NUM, new Runnable() {
					// 当所有线程到达barrier时执行
					@Override
					public void run() {
						// TODO Auto-generated method stub
						System.out.println("Inside Barrier");
		
					}
				});
		
				for (int i = 0; i < THREAD_NUM; i++) {
					new Thread(new WorkerThread(cb)).start();
				}
			}
		}

## 并发数据结构
		
		SynchronousQueue sq=new SynchronousQueue();
		
		LinkedBlockingQueue lbq=new LinkedBlockingQueue();
		
		DelayQueue dwq= new DelayQueue();  
		
		
		
  
# Java多线程（七）之同步器基础：AQS框架深入分析  
  
  `
  	一、什么是同步器

	多线程并发的执行，之间通过某种 共享 状态来同步，只有当状态满足 xxxx 条件，才能触发线程执行 xxxx 。
	这个共同的语义可以称之为同步器。可以认为以上所有的锁机制都可以基于同步器定制来实现的。

	而juc(Java.util.concurrent)里的思想是 将这些场景抽象出来的语义通过统一的同步框架来支持。
	juc 里所有的这些锁机制都是基于 AQS （ AbstractQueuedSynchronizer ）框架上构建的。下面简单介绍下 AQS（ AbstractQueuedSynchronizer ）。 		可以参考Doug Lea的论文The java.util.concurrent Synchronizer Framework

	我们来看下java.util.concurrent.locks大致结构
	二、AQS框架如何构建同步器

	0、同步器的基本功能

	一个同步器至少需要包含两个功能：
	1.       获取同步状态
	如果允许，则获取锁，如果不允许就阻塞线程，直到同步状态允许获取。
	2.       释放同步状态
	修改同步状态，并且唤醒等待线程。
	根据作者论文， aqs 同步机制同时考虑了如下需求：
	1.       独占锁和共享锁两种机制。
	2.       线程阻塞后，如果需要取消，需要支持中断。
	3.       线程阻塞后，如果有超时要求，应该支持超时后中断的机制。

	1、同步状态的获取与释放

	AQS实现了一个同步器的基本结构，下面以独占锁与共享锁分开讨论，来说明AQS怎样实现获取、释放同步状态。

	1.1、独占模式

	独占获取： tryAcquire 本身不会阻塞线程，如果返回 true 成功就继续，如果返回 false 那么就阻塞线程并加入阻塞队列。
	
	protected boolean tryAcquire(int arg) {    
	    throw new UnsupportedOperationException();    
	}    
	protected boolean tryRelease(int arg) {    
	    throw new UnsupportedOperationException();    
	}    
	protected int tryAcquireShared(int arg) {    
	    throw new UnsupportedOperationException();    

	}    
	protected boolean tryReleaseShared(int arg) {    
	    throw new UnsupportedOperationException();    
	}    
	
	从以上源码可以看出AQS实现基本的功能：
	AQS虽然实现了acquire，和release方法是可能阻塞的，但是里面调用的tryAcquire和tryRelease是由子类来定制的且是不阻塞的可。以认为同步状态的维护、获取、释放动作是由子类实现的功能，而动作成功与否的后续行为时有AQS框架来实现。

	3、状态获取、释放成功或失败的后续行为：线程的阻塞、唤醒机制

	有别于wait和notiry。这里利用 jdk1.5 开始提供的 LockSupport.park() 和 LockSupport.unpark() 的本地方法实现，实现线程的阻塞和唤醒。
	得到锁的线程禁用(park)和唤醒(unpark)，也是直接native实现（这几个native方法的实现代码在hotspot\src\share\vm\prims\unsafe.cpp文件中，但是关键代码park的最终实现是和操作系统相关的，比如windows下实现是在os_windows.cpp中，有兴趣的同学可以下载jdk源码查看）。唤醒一个被park()线程主要手段包括以下几种
	1. 其他线程调用以被park()线程为参数的unpark(Thread thread).
	2. 其他线程中断被park()线程,如waiters.peek().interrupt();waiters为存储线程对象的队列.
	3. 不知原因的返回。
	
	4、线程阻塞队列的维护

	阻塞线程节点队列 CHL Node queue 。
	根据论文里描述， AQS 里将阻塞线程封装到一个内部类 Node 里。并维护一个 CHL Node FIFO 队列。 CHL队列是一个非阻塞的 FIFO 队列，也就是说往里		面插入或移除一个节点的时候，在并发条件下不会阻塞，而是通过自旋锁和 CAS 保证节点插入和移除的原子性。实现无锁且快速的插入。关于非阻塞算法可		以参考  Java 理论与实践: 非阻塞算法简介 
	三、AQS在各同步器内的Sync与State实现

	1、什么是state机制：

	提供 volatile 变量 state;  用于同步线程之间的共享状态。通过 CAS 和 volatile 保证其原子性和可见性。对应源码里的定义
	
		以上4个AQS的使用是比较典型，然而有个问题就是这些状态存在哪里呢？并且是可以计数的。从以上4个example,我们可以很快得到答案，AQS提供给	了子类一个int state属性。并且暴露给子类getState()和setState()两个方法(protected)。这样就为上述状态解决了存储问题，RetrantLock可以将这个		state用于存储当前线程的重进入次数，Semaphore可以用这个state存储许可数，CountDownLatch则可以存储需要被countDown的次数，而Future则可以	存储当前任务的执行状态(RUNING,RAN,CANCELL)。其他的Synchronizer存储他们的一些状态。

	AQS留给实现者的方法主要有5个方法，其中tryAcquire,tryRelease和isHeldExclusively三个方法为需要独占形式获取的synchronizer实现的，比如线程		独占ReetranLock的Sync，而tryAcquireShared和tryReleasedShared为需要共享形式获取的synchronizer实现。

		ReentrantLock内部Sync类实现的是tryAcquire,tryRelease, isHeldExclusively三个方法(因为获取锁的公平性问题，tryAcquire由继承该		Sync类的内部类FairSync和NonfairSync实现)Semaphore内部类Sync则实现了tryAcquireShared和tryReleasedShared(与CountDownLatch相似，因为公		平性问题，tryAcquireShared由其内部类FairSync和NonfairSync实现)。CountDownLatch内部类Sync实现了tryAcquireShared和			tryReleasedShared。FutureTask内部类Sync也实现了tryAcquireShared和tryReleasedShared。
  `  
  
# Java 8：StampedLock 支持读锁 写锁 乐观锁  
  StampedLock是并发包里面jdk8版本新增的一个锁，该锁提供了三种模式的读写控制，三种模式分别如下：
写锁writeLock，是个排它锁或者叫独占锁，同时只有一个线程可以获取该锁，当一个线程获取该锁后，其它请求的线程必须等待，当目前没有线程持有读锁或者写锁的时候才可以获取到该锁，请求该锁成功后会返回一个stamp票据变量用来表示该锁的版本，当释放该锁时候需要unlockWrite并传递参数stamp。
 

悲观读锁readLock，是个共享锁，在没有线程获取独占写锁的情况下，同时多个线程可以获取该锁，如果已经有线程持有写锁，其他线程请求获取该读锁会被阻塞。这里讲的悲观其实是参考数据库中的乐观悲观锁的，这里说的悲观是说在具体操作数据前悲观的认为其他线程可能要对自己操作的数据进行修改，所以需要先对数据加锁，这是在读少写多的情况下的一种考虑,请求该锁成功后会返回一个stamp票据变量用来表示该锁的版本，当释放该锁时候需要unlockRead并传递参数stamp。
 

乐观读锁tryOptimisticRead，是相对于悲观锁来说的，在操作数据前并没有通过CAS设置锁的状态，如果当前没有线程持有写锁，则简单的返回一个非0的stamp版本信息，获取该stamp后在具体操作数据前还需要调用validate验证下该stamp是否已经不可用，也就是看当调用tryOptimisticRead返回stamp后到到当前时间间是否有其他线程持有了写锁，如果是那么validate会返回0，否者就可以使用该stamp版本的锁对数据进行操作。由于tryOptimisticRead并没有使用CAS设置锁状态所以不需要显示的释放该锁。该锁的一个特点是适用于读多写少的场景，因为获取读锁只是使用与或操作进行检验，不涉及CAS操作，所以效率会高很多，但是同时由于没有使用真正的锁，在保证数据一致性上需要拷贝一份要操作的变量到方法栈，并且在操作数据时候可能其他写线程已经修改了数据，而我们操作的是方法栈里面的数据，也就是一个快照，所以最多返回的不是最新的数据，但是一致性还是得到保障的。  
  
	public class Point {
	    // 成员变量
	    private double x, y;

	    // 锁实例
	    private final StampedLock sl = new StampedLock();

	    // 排它锁-写锁（writeLock）
	    void move(double deltaX, double deltaY) {
		long stamp = sl.writeLock();
		try {
		    x += deltaX;
		    y += deltaY;
		} finally {
		    sl.unlockWrite(stamp);
		}
	    }

	    // 乐观读锁（tryOptimisticRead）
	    double distanceFromOrigin() {

		// 尝试获取乐观读锁（1）
		long stamp = sl.tryOptimisticRead();
		// 将全部变量拷贝到方法体栈内（2）
		double currentX = x, currentY = y;

		// 检查在（1）获取到读锁票据后，锁有没被其他写线程排它性抢占（3）
		if (!sl.validate(stamp)) {
		    // 如果被抢占则获取一个共享读锁（悲观获取）（4）
		    stamp = sl.readLock();
		    try {
			// 将全部变量拷贝到方法体栈内（5）
			currentX = x;
			currentY = y;

		    } finally {
			// 释放共享读锁（6）
			sl.unlockRead(stamp);
		    }
		}
		// 返回计算结果（7）
		return Math.sqrt(currentX * currentX + currentY * currentY);
	    }

	    // 使用悲观锁获取读锁，并尝试转换为写锁
	    void moveIfAtOrigin(double newX, double newY) {
		// 这里可以使用乐观读锁替换（1）
		long stamp = sl.readLock();
		try {
		    // 如果当前点在原点则移动（2）
		    while (x == 0.0 && y == 0.0) {
			// 尝试将获取的读锁升级为写锁（3）
			long ws = sl.tryConvertToWriteLock(stamp);
			// 升级成功，则更新票据，并设置坐标值，然后退出循环（4）
			if (ws != 0L) {
			    stamp = ws;
			    x = newX;
			    y = newY;
			    break;
			} else {
			    // 读锁升级写锁失败则释放读锁，显示获取独占写锁，然后循环重试（5）
			    sl.unlockRead(stamp);
			    stamp = sl.writeLock();
			}
		    }
		} finally {
		    // 释放锁（6）
		    sl.unlock(stamp);
		}
	    }
	}
 
	   
	   
	   
	   
