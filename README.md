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
