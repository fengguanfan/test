---
title: Java 多线程
date: 2019-07-25 22:58:40
categories: 
    - Java
tags: 
    - Java 多线程
mp3: /musics/肩上蝶.mp3
cover: /images/main/千寻.jpg
---

# 1.简单的集合
			public static void main(String[] args) {
				--线程不安全
				List<String> list1 = new ArrayList<String>();
				list1.add("test1");
				list1.get(0);

				--线程安全
				Vector<String> vector = new Vector<String>();
				vector.add("test2");
				vector.get(0);

				--线程不安全
				Map<String,String> map = new HashMap<String,String>();
				map.put("1","test3");
				map.get("1");

				--线程安全
				Map<String,String> map1 = new Hashtable<String, String>();
				map.put("2","test4");
				map.get("2");

				--Vector和Hashtable的方法用synchronized加锁 其中添加删除是安全的，get获取时也是添加了synchronized

				--由线程不安全变成安全的方法 1.Collections
				List<String> syList = Collections.synchronizedList(list1);
				Map<String,String> syMap = Collections.synchronizedMap(map);

				--2.ConcurrentHashMap 是一种分布加锁的工具类 最多分成16段 每个段是一个Hashtable

			}
# 2.CountDownLatch（闭锁）计数器
			class ThreadDemo extends Thread{
				private CountDownLatch countDownLatch;

				public ThreadDemo(CountDownLatch countDownLatch) {
					this.countDownLatch = countDownLatch;
				}

				@Override
				public void run() {
					try{
						sleep(500);
					}catch (Exception e){

					}
					System.out.println(getName()+"执行成功");
					--释发CountDownLatch，即数量减1
					countDownLatch.countDown();
				}
			}
			void CountDownLatchDemo() throws InterruptedException {
				CountDownLatch c = new CountDownLatch(2);

				ThreadDemo t1 = new ThreadDemo(c);
				ThreadDemo t2 = new ThreadDemo(c);
				t1.start();
				t2.start();

				Thread.sleep(300);
				--判断CountDownLatch的数量是否为0，是才会走下面的代码
				--目的是保证上面的线程1和2都要执行完成
				c.await();

				System.out.println("CountDownLatch的数量为0");

			}
# 3.CyclicBarrier 回环栅栏
			
			class ThreadDemo2 extends Thread{
				private CyclicBarrier cyclicBarrier;

				public ThreadDemo2(CyclicBarrier cyclicBarrier) {
					this.cyclicBarrier = cyclicBarrier;
				}

				@Override
				public void run() {
					System.out.println(getName()+"开始执行");
					try{
						Thread.sleep(500);

						--当一个线程进来时，数量加1
						--只有数量等于CyclicBarrier的数量时，才会执行下面的代码，线程各自执行操作
						cyclicBarrier.await();
					}catch (Exception e){
					}
					System.out.println(getName()+"执行成功");

				}
			}

			void cyclicBarrierDemo (){
				CyclicBarrier barrier = new CyclicBarrier(3);

				ThreadDemo2 t1 = new ThreadDemo2(barrier);
				ThreadDemo2 t2 = new ThreadDemo2(barrier);
				ThreadDemo2 t3 = new ThreadDemo2(barrier);
				t1.start();
				t2.start();
				t3.start();

			}
# 4.Semaphore
			--Semaphore  类似场景公共厕所   join就是插队
			class ThreadDemo3 extends Thread{
				private Semaphore semaphore;
				private String name;

				public ThreadDemo3(Semaphore semaphore, String name) {
					this.semaphore = semaphore;
					this.name = name;
				}

				@Override
				public void run() {
					System.out.println(getName()+"开始执行");
					try{
						int availablePermits = semaphore.availablePermits();
						if(availablePermits > 0){
							System.out.println(getName()+"还有位置");
						}else{
							System.out.println(getName()+"没有位置");
						}
						--等待，抢占位置
						semaphore.acquire();
						System.out.println(getName()+"抢到位置");
						Thread.sleep(new Random().nextInt(1000));--执行过程
						System.out.println(getName()+"结束释放位置");
						semaphore.release();
					}catch (Exception e){
					}
					System.out.println(getName()+"执行成功");

				}
			}

			void semaphoreDemo (){
				Semaphore wc = new Semaphore(3);

				for(int i = 0 ;i<10;i++){
					new ThreadDemo3(wc, "第"+i+"个人");
				}

			}
# 5.并发队列
1.并发队列  ConcurrentLinkedQueue 长度无界 是队列 先进先出
2.BlockingQueue 阻塞队列  LinkedBlockingQueue用于生产者和消费者模式
3.方法添加 add 和 offer 从尾部添加元素
4.删除 poll 和 peek  前者会删除元素，后者不会删除元素，都是从头取元素

5.生产者和消费者简易模型

			class Prodcer implements Runnable{
				private BlockingQueue<String> queue;
				--volatile 保证flag的value值内存中是最新的
				public volatile boolean flag = true;
				--AtomicInteger 原子类 保证内存数据最新
				public AtomicInteger num = new AtomicInteger(0);


				public Prodcer(BlockingQueue<String> queue) {
					this.queue = queue;
				}

				@Override
				public void run() {
					System.out.println("生产者线程开始执行");
					while(flag){
						System.out.println("正在生产数据");
						--num.incrementAndGet() 相当 num++  num.getAndAdd(a)相当 num += a
						String data = num.incrementAndGet()+"";
						try {
							--阻塞2秒钟
							boolean offer = queue.offer(
								   data,2,TimeUnit.SECONDS
							);
							if(offer){
							   System.out.println("生产者，存入"+data+"到队列中成功");
							}else {
								System.out.println("生产者，存入"+data+"到队列中失败");
							}
							Thread.sleep(1000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}finally {
							System.out.println("生产者退出线程");
						}
					}
				}

				public void stop(){
					this.flag = false;
				}
			}

			class Consumer implements Runnable{
				private BlockingQueue<String> queue;
				--volatile 保证flag的value值内存中是最新的
				public volatile boolean flag = true;

				public Consumer(BlockingQueue<String> queue) {
					this.queue = queue;
				}

				@Override
				public void run() {
					System.out.println("消费者线程开始执行");
					while(flag){
						System.out.println("正在消费数据");
						try {
							--阻塞2秒钟 去获取数据
							String data = queue.poll(2,TimeUnit.SECONDS);
							if(data != null){
								System.out.println("消费者，将"+data+"到队列中取出");
								Thread.sleep(1000);
							}else {
								System.out.println("消费者，2秒内没有取到数据");
								flag = false;
							}
						} catch (InterruptedException e) {
							e.printStackTrace();
						}finally {
							System.out.println("消费者退出线程");
						}
					}
				}

				public void stop(){
					this.flag = false;
				}
			}

			void  concurrentDemo(){
				BlockingQueue<String> queue = new LinkedBlockingDeque<String>(10);

				Prodcer p1 = new Prodcer(queue);
				Prodcer p2 = new Prodcer(queue);

				Consumer c1 = new Consumer(queue);

				Thread t1 = new Thread(p1);
				Thread t2 = new Thread(p2);
				Thread t3 = new Thread(c1);


				try {
					t1.start();
					t2.start();
					t3.start();

					--执行10秒
					Thread.sleep(10*1000);

					t1.stop();
					t2.stop();
					t3.stop();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
# 6.线程池
## 优点：
1.减少new线程的操作，会占资源
2.提高效率
3.避免浪费资源

## 类型：
			
			class Pool implements Runnable{

				@Override
				public void run() {
					System.out.println("线程运行");
				}
			}

			--newCachedThreadPool  
			void newCachedDemo(){
				Pool pool = new Pool();
				Thread t1 = new Thread(pool,"线程a");

				ExecutorService service = Executors.newCachedThreadPool();
				service.execute(t1);

				--释放线程
				service.shutdown();


			}

			--newFixedThreadPool  定长 设置最大值 超过会阻塞
			void newFixedDemo(){
				Pool pool = new Pool();
				Thread t1 = new Thread(pool,"线程a");

				ExecutorService service = Executors.newFixedThreadPool(3);
				service.execute(t1);

				--释放线程
				service.shutdown();


			}

			--newScheduledThreadPool  定时线程
			void newScheduledDemo(){
				Pool pool = new Pool();
				Thread t1 = new Thread(pool,"线程a");

				ScheduledExecutorService service = Executors.newScheduledThreadPool(4);

				--间隔3秒执行线程
				service.schedule(t1,3,TimeUnit.SECONDS);

				--释放线程
				service.shutdown();

			}

			--newSingleThreadExecutor  单个线程，只有一个线程
			void newSingleThreadDemo(){
				Pool pool = new Pool();
				Thread t1 = new Thread(pool,"线程a");

				ExecutorService service = Executors.newSingleThreadExecutor();
				service.execute(t1);

				--释放线程
				service.shutdown();

			}
## 合理配置线程池
1.cpu密集型   要少配置线程数，和机器的cpu数量一致即可  可以保证每个线程都有执行
2.io密集型    要多配置线程数，2个cpu 即可  因为大部分线程都是阻塞的。


