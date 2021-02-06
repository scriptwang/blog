# Java线程池解析

Java线程是内核态的线程，执行线程要完成用户态到内核态的转换，频繁创建与销毁线程比较耗系统资源，因此有了线程池存在的意义。

主线程往线程池里面不断的扔任务，扔任务的过程不会阻塞，扔完就返回，线程池根据任务数量new线程执行任务，执行完任务之后线程是复用的，当下次有任务来临，线程是空闲状态时直接拿此线程执行任务，减小线程创建与销毁的开销。

线程池执行任务的过程有很多细节，比如线程池本身线程安全的处理，大量用到了CAS原子操作；还有线程池是可扩容可缩容的，用核心线程数和最大线程数表示，当任务不能被执行时还有拒绝策略，当然也可以实现自己的拒绝策略；任务执行前后还有钩子函数，也可以自己进行覆盖

任务会放到一个**阻塞队列**，当woker拿不到任务的时候会阻塞（park），等待任务的到来，而不会直接退出（普通线程会在某些条件下退出，比如队列为空并且拿任务超时了）。

## 原理

- CAS 维护状态字（线程池状态和线程/worker数量）
- park/unpark 阻塞队列中park让线程阻塞，避免while空转
- 阻塞队列

## 状态字：线程池的状态和线程数量

状态和数量都放到一个int中，高3位表示线程池的当前状态（runState），其余29位用于表示线程或者叫worker的数量（workerCount），就是下面这句话把这两个数据放到了一起，这样做是为了保证runState和workerCount的线程安全性，因为ctl本身是AtomicInteger，是原子操作。（题外话：29为表示worker数量，那么最大值就是2^29-1，说明线程池线程的数量是不能超过这个数的，源码注释里面也说了，如果未来不够用了，可以把AtomicInteger改成AtomicLong，目前来说用int更快更简单，其实很多时候源码注释就能学到很多）

```JAVA
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

那么线程池的状态有哪些？源码标明了：

- RUNNING：接受新任务并且能处理队列中的任务
- SHUTDOWN：不能接受新任务了，但是还可以处理队列中的任务
- STOP：不接受新任务，不处理队列中的任务，中断所有正在运行的任务任务
- TIDYING：所有任务被终止了，工作线程数 workCount 也被设为0，线程的状态也被设为TIDYING，并开始调用钩子函数terminated()
- TERMINATED：钩子函数 terminated() 执行完毕

接下来就是一系列很迷的位移操作了，参考文章

[深入浅出Java线程池ThreadPoolExecutor - 掘金](./参考文章/【线程池】深入浅出Java线程池ThreadPoolExecutor - 掘金.md)

如下：

```JAVA
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));//初始化状态字
private static final int COUNT_BITS = Integer.SIZE - 3;//分割位置
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;//线程池最大容量

// runState is stored in the high-order bits 线程池状态存储在高位
private static final int RUNNING    = -1 << COUNT_BITS;

// Packing and unpacking ctl 打包解包状态字
private static int runStateOf(int c)     { return c & ~CAPACITY; }//输入状态字获取线程池状态
private static int workerCountOf(int c)  { return c & CAPACITY; }//输入状态字，获取worker数量
//rs | wc结果就是保留rs和wc里面所有的1
//runStateOf就是   取rs | wc高3位的1
//workerCountOf就是取rs | wc低29位的1
private static int ctlOf(int rs, int wc) { return rs | wc; }//生成状态字
```

要看懂这个操作首先要了解一下位操作：

- ```<<``` 左移，左移n位那么就乘2^n，如5 << 12 等于 5 * 2^12，右移就做除法

- ```|``` 按位或，有1为1，全是0为0，如0101 | 1001 等于1101

- ```&``` 按位与，有0为0，全是1为1，如0101 &1001 等于0001

- ```~```取反操作，如 ~0000等于1111

  上面代码中COUNT_BITS = 29，CAPACITY = (1<<29) - 1 = 1 * 2^29 - 1 = 536870911

  | 变量      | 表达式      | 十进制     | 二进制                                  |
  | --------- | ----------- | ---------- | --------------------------------------- |
  | CAPACITY  | (1<<29) - 1 | 536870911  | **00011111 11111111 11111111 11111111**‬ |
  | ~CAPACITY |             | -536870912 | **11100000 00000000 00000000 00000000** |
  | RUNNING   | -1<<29    | -536870912 | 11100000 00000000 00000000 00000000‬     |
  | ctl       | RUNNING按位或0  | -536870912 | 11100000 00000000 00000000 00000000‬     |

  runStateOf方法中```c & ~CAPACITY```，CAPACITY取反了，再按位与，看上面二进制就知道，实际上是取了c的高三位，因为按位与会丢弃掉含0的位，c是啥，c就是传进来的状态字

  workerCountOf方法中``` c & CAPACITY;```，直接与CAPACITY按位与，实际上是取的低29位。

  所以runState取状态字高三位，workerCount取状态字低三位。

  如下图所示

  ```
    高三位表示线程池状态
  { [---][----------------------------] } 整型一共32位
         低29位表示线程池中线程数量
  ```

  

## 核心线程数与最大线程数的区别

当前正在运行的线程数小于核心线程数时候，直接新建worker执行任务，否则入队列，入队失败（队列满）则创建新worker执行任务，后面创建新worker的时候，能创建设置的最大线程数的worker，直接看execute源码，Doug Lea大神都给你注释好了

```JAVA
public void execute(Runnable command) {
	if (command == null)
		throw new NullPointerException();
	/*
	 * Proceed in 3 steps:
	 *
	 * 如果线程数小于核心线程数，则尝试开了个worker来运行传进来的任务并且将该
	 * 任务作为该worker的第一个任务，addWorker方法自动检查runState和workerCount
	 * 1. If fewer than corePoolSize threads are running, try to
	 * start a new thread with the given command as its first
	 * task.  The call to addWorker atomically checks runState and
	 * workerCount, and so prevents false alarms that would add
	 * threads when it shouldn't, by returning false.
	 *
	 * 如果一个任务成功入队，仍然需要再次检查是否需要添加一个线程
	 * 2. If a task can be successfully queued, then we still need
	 * to double-check whether we should have added a thread
	 * (because existing ones died since last checking) or that
	 * the pool shut down since entry into this method. So we
	 * recheck state and if necessary roll back the enqueuing if
	 * stopped, or start a new thread if there are none.
	 *
	 * 如果不能入队，执行拒绝策略
	 * 3. If we cannot queue task, then we try to add a new
	 * thread.  If it fails, we know we are shut down or saturated
	 * and so reject the task.
	 */
	int c = ctl.get();//拿出状态字
    //重点来了，如果当前线程数小于核心线程数，新增worker执行任务
    //addWorker是自旋+CAS，多线程条件下可能竞争失败，如果失败则说明
    //核心线程数满了，走下面的流程
	if (workerCountOf(c) < corePoolSize) {
		if (addWorker(command, true))
			return;
		c = ctl.get();//如果添加worker失败，更新下状态字
	}
    //入任务队列
	if (isRunning(c) && workQueue.offer(command)) {
        //再次检查，入队后可能线程池状态已经发生了变化
		int recheck = ctl.get();
        //入队后线程池不是RUNNING状态，则出队并执行拒绝策略
		if (! isRunning(recheck) && remove(command))
			reject(command);
        //入队后发现没有worker了，在创建一个首任务为null的worker
		else if (workerCountOf(recheck) == 0)
			addWorker(null, false);
	}
    //如果入队失败了，并且添加worker也失败了，则直接执行拒绝策略
	else if (!addWorker(command, false))
		reject(command);
}
```

举例：核心线程数为5，最大线程数为10，则当前正在执行的任务超过5个的时候，则排队，队列满的时候，再根据任务数新建worker，如果任务数超过最大线程数10个了，则执行拒绝策略，案例代码

``` java
public static void testMaxThreadNum(){
	ThreadPoolExecutor pool = new ThreadPoolExecutor(
			5,//核心线程数5个
			10,//最大线程数10个
			5000,
			TimeUnit.MILLISECONDS,
			new LinkedBlockingQueue<>(5),//队列长度为5
			new ThreadPoolExecutor.AbortPolicy()
	);

	for (int i=0;i<15;i++) {//如果多于15个任务就会触发异常捕获机制,因为corePoolSize+workQueue的size = 15
		final int index = i;
		pool.execute(()-> {
			String name = Thread.currentThread().getName();
			System.out.println(name + " has die and index is " + index);
			//直接睡死，达到每次执行一个任务消耗一个线程的目的
			try {
				Thread.sleep(1000000000L);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

		});
	}
	//看线程池状态
	System.out.println(pool);
}
```

查看结果

```
pool-1-thread-1 has die and index is 0
pool-1-thread-4 has die and index is 3
pool-1-thread-3 has die and index is 2
pool-1-thread-2 has die and index is 1
pool-1-thread-5 has die and index is 4
pool-1-thread-6 has die and index is 10
pool-1-thread-7 has die and index is 11
pool-1-thread-8 has die and index is 12
pool-1-thread-9 has die and index is 13
pool-1-thread-10 has die and index is 14
java.util.concurrent.ThreadPoolExecutor@16b98e56
[Running, pool size = 10, active threads = 10, queued tasks = 5, completed tasks = 0]
# 此时刚好有10个线程在执行任务，5个任务在队列中，刚好15个，如果此时再来一个任务，就要执行拒绝策略了
```





## Worker线程核心函数

不难猜到线程池里面的每个线程肯定有个run函数，因为run函数是线程执行的入口函数，此时可能会误以为传入的Runnable任务的run函数是线程入口函数，其实不是，在线程池里面是直接调用的Runnable.run，相当于调用方法并不是开启线程，那真正的run函数在哪里呢？答案是在Worker里面，看下Worker的定义

```JAVA
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
```

看看这个Worker可不简单，继承自AQS，自己还带一把锁，实现Runnable，看看成员变量

``` java
/** Thread this worker is running in.  Null if factory fails. */
final Thread thread;//此Worker运行的线程
/** Initial task to run.  Possibly null. */
Runnable firstTask;//首个任务
/** Per-thread task counter */
volatile long completedTasks;//完成的任务
```

再来看run方法，重点是runWorker函数了，这个就是主要的循环

```JAVA
/** Delegates main run loop to outer runWorker  */
public void run() {
	runWorker(this);
}

final void runWorker(Worker w) {
	Thread wt = Thread.currentThread();
	Runnable task = w.firstTask;
	w.firstTask = null;
	w.unlock(); // allow interrupts
	boolean completedAbruptly = true;
	try {
        //注意此处是while循环，getTask是从队列里面去拿任务，拿不到任务会阻塞住！！！！
        //拿不到任务并不会while空转，那样会很耗CPU
		while (task != null || (task = getTask()) != null) {
             //此处会什么要加锁？这把锁是Worker自己的，不是整个线程池的，锁住有什么意义呢？
			w.lock();
			if ((runStateAtLeast(ctl.get(), STOP) ||
				 (Thread.interrupted() &&
				  runStateAtLeast(ctl.get(), STOP))) &&
				!wt.isInterrupted())
				wt.interrupt();
			try {
                 //钩子函数，执行任务前
				beforeExecute(wt, task);
				Throwable thrown = null;
				try {
                      //重点来了，直接调用的是传入任务的run方法
					task.run();
				} catch (RuntimeException x) {
					thrown = x; throw x;
				} catch (Error x) {
					thrown = x; throw x;
				} catch (Throwable x) {
					thrown = x; throw new Error(x);
				} finally {
                      //钩子函数，执行任务后
					afterExecute(task, thrown);
				}
			} finally {
				task = null;
				w.completedTasks++;
				w.unlock();
			}
		}
		completedAbruptly = false;
	} finally {
		processWorkerExit(w, completedAbruptly);
	}
}

```

## 线程在线程池中怎么被阻塞的？

上面说了runWorker的while循环不会空转，必定在哪个地方阻塞，跟进拿任务的方法getTask里面，发现线程拿不到任务的时候会在队列出阻塞，而线程池需要的正是阻塞队列BlockingQueue，有没有恍然大悟的感觉，具体看代码，找到getTask

```JAVA
private Runnable getTask() {
    ///省略
    for (;;) {
        ///省略
		try {
			Runnable r = timed ?
				workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
				workQueue.take();//重点看这句话，去队列拿元素
			if (r != null)
				return r;
			timedOut = true;
		} catch (InterruptedException retry) {
			timedOut = false;
		}
	}
}
```

以ArrayBlockingQueue为列，看看take方法

```JAVA
public E take() throws InterruptedException {
	final ReentrantLock lock = this.lock;
	lock.lockInterruptibly();
	try {
		while (count == 0)
			notEmpty.await();//没有元素await
		return dequeue();
	} finally {
		lock.unlock();
	}
}
```

跟进await方法，发现还是park让线程在阻塞，看到这里是不是想到和AQS实现原理是一样一样的嘛，果然并发变成离不开CAS park两大法宝

``` java
public final void await() throws InterruptedException {
	....
	while (!isOnSyncQueue(node)) {
        //看到没，就是park让线程阻塞的
		LockSupport.park(this);
		if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
			break;
	}
	...
}
```



## 超过核心线程数的线程是在哪里退出的？

证明超时的最大线程数产生的线程会退出

```java
/**
         * 证明超时的最大线程数产生的线程会退出 
         *
         */
public static void testWorkerRecycle(){
	ThreadPoolExecutor pool = new ThreadPoolExecutor(
			1,//核心线程数1个
			2,//最大线程数2个
			1000,//1s过期
			TimeUnit.MILLISECONDS,
			new LinkedBlockingQueue<>(5),//队列长度为5
			new ThreadPoolExecutor.AbortPolicy()
	);

	for (int i=0;i<7;i++) {
		final int index = i;
		pool.execute(()-> {
			String name = Thread.currentThread().getName();
			System.out.println(name + " has die and index is " + index);

			//pool-1-thread-2理论应该是最大线程数所产生的那个线程
			if (!name.equals("pool-1-thread-2")){
				//让核心线程睡死
				try {
					Thread.sleep(1000000000L);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
	}
	//看线程池状态
	System.out.println(pool);
	//等pool-1-thread-2过期了再看线程池状态，看看pool-1-thread-2是否已经被取消
	sleep(3000);
	System.out.println(pool);
}
```

输出

```
pool-1-thread-1 has die and index is 0
java.util.concurrent.ThreadPoolExecutor@7699a589
# 此时线程池里面有两个线程
[Running, pool size = 2, active threads = 2, queued tasks = 5, completed tasks = 0]
pool-1-thread-2 has die and index is 6
pool-1-thread-2 has die and index is 1
pool-1-thread-2 has die and index is 2
pool-1-thread-2 has die and index is 3
pool-1-thread-2 has die and index is 4
pool-1-thread-2 has die and index is 5
java.util.concurrent.ThreadPoolExecutor@7699a589
# 此时只有一个线程，说明另外一个退出了
[Running, pool size = 1, active threads = 1, queued tasks = 0, completed tasks = 6]

```

关键看getTask方法和runWorker方法

```JAVA
private Runnable getTask() {
	boolean timedOut = false; // Did the last poll() time out?

	for (;;) {
		
		...
         //第二次自旋 timed && timedOut 为true   workQueue.isEmpty()为true
         //也就是说超时了，并且队列是空的，那么workerCount减1并且直接返回null
		if ((wc > maximumPoolSize || (timed && timedOut))
			&& (wc > 1 || workQueue.isEmpty())) {
			if (compareAndDecrementWorkerCount(c))
				return null;
			continue;
		}

		try {
			Runnable r = timed ?
                 //看这里，第一次自旋，如果是非核心线程并且超时了返回r为null
				workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
				workQueue.take();
			if (r != null)
				return r;
			timedOut = true;//r为null 则timeOut为true
		} catch (InterruptedException retry) {
			timedOut = false;
		}
	}
}
```

getTask返回null再看runWorker

```JAVA
final void runWorker(Worker w) {
	...
	try {
        //此时getTask返回null，while条件不成立退出while循环
		while (task != null || (task = getTask()) != null) {
			...
		}
		completedAbruptly = false;
	} finally {
        //退出循环后执行这句话就彻底退出了线程！！！！
        //最终调用线程的exit()方法
		processWorkerExit(w, completedAbruptly);
	}
}
```

最后得出结论：如果拿任务超时了，任务队列里面也没有任务了则直接结束while循环退出线程！

## 自定义线程池工厂

```JAVA
/**
 *
 * 测试自定义线程池工厂
 */
public static void testCustomThreadFactory(){
    //自定义线程池工厂
    ThreadFactory factory = new ThreadFactory(){
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread( r, " My custom thread ");
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    };

    ThreadPoolExecutor pool = new ThreadPoolExecutor(
            1,//核心线程数1个
            2,//最大线程数2个
            1000,//1s过期
            TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<>(),
            factory,//线程工厂
            new ThreadPoolExecutor.AbortPolicy()
    );

    for (int i=0;i<10;i++) {
        final int index = i;
        pool.execute(()-> {
            String name = Thread.currentThread().getName();
            System.out.println(name + " is running and index is " + index);
        });
    }
    sleep(100);
    System.out.println(pool);
}
```

输出如下

```
 My custom thread  is running and index is 0
 My custom thread  is running and index is 1
...
java.util.concurrent.ThreadPoolExecutor@3b9a45b3
[Running, pool size = 1, active threads = 0, queued tasks = 0, completed tasks = 10]
```



## 自定义任务执行前和任务执行后方法

继承ThreadPoolExecutor类，重写beforeExecute和afterExecute方法即可

```JAVA
static class MyThreadPool extends ThreadPoolExecutor{

    public static void main(String[] args) {
        ThreadPoolExecutor pool = new MyThreadPool(
                1,//核心线程数1个
                2,//最大线程数2个
                1000,//1s过期
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(),
                Executors.defaultThreadFactory()
        );
        for (int i=0;i<10;i++) {
            final int index = i;
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    String name = Thread.currentThread().getName();
                    System.out.println(name + " is running and index is " + index);
                }
                @Override
                public String toString(){
                    return " I am task " + index;
                }
            });
        }
    }

    //直接调用父类构造方法
    public MyThreadPool(
            final int corePoolSize,
            final int maximumPoolSize,
            final long keepAliveTime,
            final TimeUnit unit,
            final BlockingQueue<Runnable> workQueue,
            final ThreadFactory threadFactory) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
    }

    /**
     * 重写父类方法，在任务执行之前执行
     * @param t
     * @param r
     */
    @Override
    protected void beforeExecute(final Thread t, final Runnable r) {
        super.beforeExecute(t,r);
        String name = Thread.currentThread().getName();
        System.out.println("before " + name + " running task " + r);
    }

    /**
     * 重写父类方法，在任务执行之后执行
     * @param t
     * @param r
     */
    @Override
    protected void afterExecute(final Runnable r, final Throwable t) {
        super.afterExecute(r,t);
        String name = Thread.currentThread().getName();
        System.out.println("after " + name + " running task " + r);
        System.out.println();
    }
}
```

输出如下

```
before pool-1-thread-1 running task  I am task 0
pool-1-thread-1 is running and index is 0
after pool-1-thread-1 running task  I am task 0

before pool-1-thread-1 running task  I am task 1
pool-1-thread-1 is running and index is 1
after pool-1-thread-1 running task  I am task 1
...
```



## 拒绝策略

当线程池不能添加任务的时候，比如当前提交任务数大于了最大线程数maximumPoolSize或者当前线程池状态属于关闭状态SHUTDOWN，或者当前线程数大于了总容量CAPACITY的时候就会执行拒绝策略
- AbortPolicy 中止策略抛异常
当不能添加任务的时候抛出异常RejectedExecutionException

- DiscardPolicy 抛弃策略不抛出异常
当不能添加任务的时候直接抛弃任务，不会抛出异常

- CallerRunsPolicy 让调用着执行策略
当不能添加任务的时候直接让调用者线程（即添加任务的线程）执行任务，前提是在线程池没有shutdown的状态，源码如下
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    //线程池没有处于关闭的状态
    if (!e.isShutdown()) {
        r.run();//让调用者执行任务
    }
}
```
- DiscardOldestPolicy
  队列（queue）：只允许在一端进行插入操作，而在另一端进行删除操作的线性表。队列是一种先进先出（First In First Out）的线性表，简称FIFO。**允许插入的一端称为队尾，允许删除的一端称为队头**。

  网上资料很多都说移除最老的任务，即队尾的任务，根据queue允许插入的一端称为队尾，允许删除的一端称为队头可知，队头才是最老的元素，即把队头的一个元素扔掉，执行刚才添加的被拒绝的任务，源码如下

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    //线程池没有处于关闭的状态
    if (!e.isShutdown()) {
        e.getQueue().poll();//拿走队头的元素
        e.execute(r);//执行刚才添加的被拒绝的任务
    }
}
```

## 所有核心线程都初始化后，第二轮任务怎么执行？
第二轮任务全部塞进队列，因为此时核心线程全部在阻塞在队列中，不用重新初始化，所以塞进队列就能立即被核心线程执行
```JAVA
public void execute(Runnable command) {
if (command == null)
    throw new NullPointerException();
    int c = ctl.get();
    //当核心线程初始化完成后 workerCountOf(c) < corePoolSize 此条件是不成立的
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    //重点在这里，直接将任务塞进workQueue
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```
验证：    

# 阻塞队列

参考文章[BlockingQueue阻塞队列详解以及5大实现](./参考文章/【JUC-AQS】BlockingQueue阻塞队列详解以及5大实现（ArrayBlockingQueue、DelayQueue、LinkedBlockingQueue...）_Java_BAT的乌托邦-CSDN博客.md)

## ArrayBlockingQueue

- 有界队列
- 内部维护了一个定长数组，还保存着两个整形变量，分别标识队头队尾位置
- 生产者和消费者共用`同一个锁`对象，意味着两者**无法真正并行运行**
- 其实ArrayBlockingQueue完全是可以使用分离锁的。但是作者Doug Lea并没有这么去干，理由如下：**ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。**
- 可以指定公平锁或非公平锁

## LinkedBlockingQueue

- 无界队列/有界队列(可指定长度)
- 内部维护了一个链表，生产者往队列放入数据后**立即返回**
- 缓冲区满则阻塞，无论对于生产者还是消费者
- 生产者和消费者采用`独立的锁`来控制数据同步，高并发下生产者和消费者可以并行操作数据
- 构造LinkedBlockingQueue时最好指定缓冲区大小，否则缓冲区是Integer.MAX_VALUE，当生产者的速度大于大于消费者的速度，系统内存很快会被耗尽

## **DelayQueue**

- 无界队列
- 是一个没有大小限制的队列，**因此生产者永远不会被阻塞**，只有消费者才会被阻塞。
- DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。

## **PriorityBlockingQueue**

- 无界队列
- 基于优先级的阻塞队列，通过传入Compator判断优先级
- **不会阻塞生产者**，而在没有可消费数据时，阻塞消费者。生产者速度**绝对不能快于消费者，否则时间一长，会耗尽内存**。

## SynchronousQueue

- 不存储元素
- 无缓冲的等待队列，**类似于无中介的直接交易**
- 可以指定公平锁和非公平锁