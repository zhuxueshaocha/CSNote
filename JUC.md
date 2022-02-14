# 多线程的实现方式

1. 实现 Runnable 接口

2. 实现 Callable 接口

3. 继承 Thread 类

4. 线程池方式创建

通过继承Thread类或者实现Runnable接口、Callable接口都可以实现多线程，不过实现Runnable接口与实现Callable接口的方式基本相同，只是Callable接口里定义的方法返回值，可以声明抛出异常而已。因此将实现Runnable接口和实现Callable接口归为一种方式。

实现接口 VS 继承 Thread：

实现接口会更好一些，因为：

Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；

类可能只要求可执行就行，继承整个 Thread 类开销过大。

# java中的4种线程池

Java 里面线程池的顶级接口是 Executor，但是严格意义上讲 Executor 并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是 ExecutorService。

## newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

底层分析：corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)

使用场景：执行很多短期异步的小程序或者负载较轻的服务器

##  newFixedThreadPool
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待

底层分析：返回ThreadPoolExecutor实例，接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列

使用场景：执行长期的任务，性能好很多

## newScheduledThreadPool
创建一个定长线程池，支持定时和周期性任务执行

底层分析：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列

使用场景：周期性执行任务的场景

## newSingleThreadExecutor

创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序（FIFO，LIFO，优先级）执行

底层分析：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列

使用场景：一个任务一个任务执行的场景


# 线程状态

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

## 新建（NEW）
创建后尚未启动。

## 可运行（RUNABLE）
正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

## 阻塞（BLOCKED）
请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。

## 无限期等待（WAITING）
等待其它线程显式地唤醒。

<img width="621" alt="截屏2022-01-26 下午3 24 11" src="https://user-images.githubusercontent.com/98211272/151120379-356527e2-bfe0-4b25-8a1f-aa9058893384.png">

## 超时等待（TIMED_WAITING）
无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

<img width="683" alt="截屏2022-01-26 下午3 24 28" src="https://user-images.githubusercontent.com/98211272/151120411-9fa25752-7ea6-4fe4-9f60-67dd3b91612b.png">


## 死亡（TERMINATED）
可以是线程结束任务之后自己结束，或者产生了异常而结束。


# 终止线程 4 种方式

# 正常运行结束
程序运行结束，线程自动结束。

## 使用退出标志退出线程
一般 run()方法执行完，线程就会正常结束，然而，常常有些线程是伺服线程。它们需要长时间的运行，只有在外部某些条件满足的情况下，才能关闭这些线程。使用一个变量来控制循环，例如：最直接的方法就是设一个 boolean 类型的标志，并通过设置这个标志为 true 或 false 来控制 while循环是否退出，代码示例：
定义了一个退出标志 exit，当 exit 为 true 时，while 循环退出，exit 的默认值为 false.在定义 exit时，使用了一个 Java 关键字 volatile，这个关键字的目的是使 exit 同步，也就是说在同一时刻只能由一个线程来修改 exit 的值。

## Interrupt 方法结束线程
使用 interrupt()方法来中断线程有两种情况：

1. 线程处于阻塞状态：当调用线程的interrupt()方法时，会抛出 InterruptException 异常。
阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后 break 跳出循环状态，从而让我们有机会结束这个线程的执行。通常很多人认为只要调用 interrupt 方法线程就会结束，实际上是错的，一定要先捕获 InterruptedException 异常之后通过 break 来跳出循环，才能正
常结束 run 方法。

2. 线程未处于阻塞状态：使用 isInterrupted()判断线程的中断标志来退出循环。当使用interrupt()方法时，中断标志就会置 true，和使用自定义的标志来控制循环是一样的道理。

## stop 方法终止线程（线程不安全）
程序中可以直接使用 thread.stop()来强行终止线程，但是 stop 方法是很危险的，就象突然关闭计算机电源，而不是按正常程序关机一样，可能会产生不可预料的结果，不安全主要是：
thread.stop()调用之后，创建子线程的线程就会抛出 ThreadDeatherror 的错误，并且会释放子线程所持有的所有锁。一般任何进行加锁的代码块，都是为了保护数据的一致性，如果在调用thread.stop()后导致了该线程所持有的所有锁的突然释放(不可控制)，那么被保护数据就有可能呈现不一致性，其他线程在使用这些被破坏的数据时，有可能导致一些很奇怪的应用程序错误。因此，并不推荐使用 stop 方法来终止线程。

# 线程之间的协作
当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join()
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。





<img width="514" alt="截屏2022-01-26 下午3 32 36" src="https://user-images.githubusercontent.com/98211272/151121505-4fbc4682-862b-41b5-aa34-f8a3c4edddcd.png">


<img width="837" alt="截屏2022-01-26 下午3 32 49" src="https://user-images.githubusercontent.com/98211272/151121504-e02d32f6-1759-4d29-a17f-8d362df5d86a.png">

<img width="372" alt="截屏2022-01-26 下午3 32 59" src="https://user-images.githubusercontent.com/98211272/151121510-67dba25e-ce25-4b7c-aac3-fa4b2a7c373f.png">

## wait() notify() notifyAll()
调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。


### notify()和notifyAll()

先说两个概念：锁池和等待池。

#### 锁池

锁池:假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。

#### 等待池

等待池:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中然后再来说notify和notifyAll的区别 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。 

综上，所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程。


### wait() 和 sleep() 的区别

wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；

wait() 会释放锁，sleep() 不会。

## await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。


<img width="817" alt="截屏2022-01-26 下午3 38 02" src="https://user-images.githubusercontent.com/98211272/151122051-019fb0c0-1310-496a-8511-fd6345687e39.png">

<img width="811" alt="截屏2022-01-26 下午3 38 12" src="https://user-images.githubusercontent.com/98211272/151122101-f8fe161c-b2c6-4868-94e7-5cd3725bf4c8.png">

# 守护线程（Deamon）

## 定义
守护线程--也称“服务线程”，他是后台线程，它有一个特性，即为用户线程提供公共服务，在没有用户线程可服务时会自动离开。
## 优先级
守护线程的优先级比较低，用于为系统中的其它对象和线程提供服务。
## 设置
通过 setDaemon(true)来设置线程为“守护线程”；将一个用户线程设置为守护线程的方式是在 线程对象创建 之前 用线程对象的 setDaemon 方法。在 Daemon 线程中产生的新线程也是 Daemon 的。
## 例子
example: 垃圾回收线程就是一个经典的守护线程，当我们的程序中不再有任何运行的Thread,程序就不会再产生垃圾，垃圾回收器也就无事可做，所以当垃圾回收线程是 JVM 上仅剩的线程时，垃圾回收线程会自动离开。它始终在低级别的状态中运行，用于实时监控和管理系统中的可回收资源。
## 生命周期
守护进程（Daemon）是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。也就是说守护线程不依赖于终端，但是依赖于系统，与系统“同生共死”。当 JVM 中所有的线程都是守护线程的时候，JVM 就可以退出了；如果还有一个或以上的非守护线程则 JVM 不会退出。


# 线程基本方法

## 线程等待（wait）
调用该方法的线程进入 WAITING 状态，只有等待另外线程的通知或被中断才会返回，需要注意的是调用 wait()方法后，会释放对象的锁。因此，wait 方法一般用在同步方法或同步代码块中。
## 线程睡眠（sleep）
sleep 导致当前线程休眠，与 wait 方法不同的是 sleep 不会释放当前占有的锁,sleep(long)会导致线程进入 TIMED-WAITING状态，而 wait()方法会导致当前线程进入 WATING 状态
## 线程让步（yield）
yield 会使当前线程让出 CPU 执行时间片，与其他线程一起重新竞争 CPU 时间片。一般情况下，优先级高的线程有更大的可能性成功竞争得到 CPU 时间片，但这又不是绝对的，有的操作系统对线程优先级并不敏感。
## 线程中断（interrupt）
中断一个线程，其本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标识位。这个线程本身并不会因此而改变状态(如阻塞，终止等)。

1. 调用 interrupt()方法并不会中断一个正在运行的线程。也就是说处于 Running 状态的线程并不会因为被中断而被终止，仅仅改变了内部维护的中断标识位而已。
 
2. 若调用 sleep()而使线程处于 TIMED-WAITING 状态，这时调用 interrupt()方法，会抛出InterruptedException,从而使线程提前结束 TIMED-WATING 状态。

## 其他方法

1. isAlive()： 判断一个线程是否存活。

2. activeCount()： 程序中活跃的线程数。

3. enumerate()： 枚举程序中的线程。

4. currentThread()： 得到当前线程。

5. isDaemon()： 判断一个线程是否为守护线程。

6. setDaemon()： 设置一个线程为守护线程。(用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束) 

7. setName()： 为线程设置一个名称。

8. wait()： 强迫一个线程等待。

9. notify()： 通知一个线程继续运行。

10. setPriority()： 设置一个线程的优先级。

11. getPriority():：获得一个线程的优先级。


# 线程池原理
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。他的主要特点为：线程复用；控制最大并发数；管理线程。

## 线程池的构造函数
1. corePoolSize：指定了线程池中的核心线程数量。

2. maximumPoolSize：指定了线程池中的最大线程数量。

3. keepAliveTime：当前线程池数量超过 corePoolSize 时，多余的空闲线程的存活时间，即多少时间后会被销毁。

4. unit：keepAliveTime 的单位。

5. workQueue：任务队列，被提交但尚未被执行的任务。

6. threadFactory：线程工厂，用于创建线程，一般用默认的即可。

7. handler：拒绝策略，当任务太多来不及处理，如何拒绝任务。

## 拒绝策略

线程池中的线程已经用完了，无法继续为新任务服务，同时，等待队列也已经排满了，再也塞不下新任务了。这时候我们就需要拒绝策略机制合理的处理这个问题。

JDK 内置的拒绝策略如下：

1. AbortPolicy：直接抛出异常，阻止系统正常运行。

2. CallerRunsPolicy：只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降。

3. DiscardOldestPolicy：丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。

4. DiscardPolicy：该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢失，这是最好的一种方案。

以上内置拒绝策略均实现了RejectedExecutionHandler 接口，若以上策略仍无法满足实际需要，完全可以自己扩展RejectedExecutionHandler 接口。

## Java 线程池工作过程

1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。

2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断：

a) 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；

b) 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；

c) 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；

d) 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常 RejectExecutionException。

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。

4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。


<img width="438" alt="截屏2022-01-26 下午3 55 26" src="https://user-images.githubusercontent.com/98211272/151124157-52af5f05-1329-4718-9fcd-195f9ed14754.png">
# Java内存模型
Java Memory Molde (Java内存模型/JMM)，千万不要和Java内存结构（JVM划分的那个堆，栈，方法区）混淆。

Java内存模型，是Java虚拟机规范中所定义的一种内存模型，Java内存模型是标准化的，屏蔽掉了底层不同计算机的区别。 Java内存模型是一套规范，描述了Java程序中各种变量(线程共享变量)的访问规则，以及在JVM中将变量存储到内存和从内存中读取变量这样的底层细节，具体如下。

Java内存模型根据官方的解释，主要是在说两个关键字，一个是volatile，一个是synchronized。

## 主内存

主内存是所有线程都共享的，都能访问的。所有的共享变量都存储于主内存。

## 工作内存

每一个线程有自己的工作内存，工作内存只存储该线程对共享变量的副本。线程对变量的所有的操 作(读，取)都必须在工作内存中完成，而不能直接读写主内存中的变量，不同线程之间也不能直接 访问对方工作内存中的变量。
Java的线程不能直接在主内存中操作共享变量。而是首先将主内存中的共享变量赋值到自己的工作内存中，再进行操作，操作完成之后，刷回主内存。

## Java内存模型的作用
Java内存模型是一套在多线程读写共享数据时，对共享数据的可见性、有序性、和原子性的规则和保障。

# 内存模型三大特性
## 原子性
Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。


<img width="306" alt="截屏2022-01-26 下午4 00 45" src="https://user-images.githubusercontent.com/98211272/151124865-51eb74bd-15d8-478f-ba16-c548f12bff6b.png">

AtomicInteger 能保证多个线程修改的原子性。


<img width="335" alt="截屏2022-01-26 下午4 01 15" src="https://user-images.githubusercontent.com/98211272/151125099-4995dd50-d7ad-4bee-8d85-048b41fd55c5.png">

除了使用原子类之外，也可以使用 synchronized 互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

## 可见性
可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

volatile

synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。

final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。

## 有序性
有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

# 指令重排

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排。为什么指令重排序可以提高性能？简单地说，每一个指令都会包含多个步骤，每个步骤可能使用不同的硬件。因此，流水线技术产生了，它的原理是指令1还没有执行完，就可以开始执行指令2，而不用等到指令1执行结束之后再执行指令2，这样就大大提高了效率。但是，流水线技术最害怕中断，恢复中断的代价是比较大的，所以我们要想尽办法不让流水线中断。指令重排就是减少中断的一种技术。
指令重排一般分为以下三种：

1. 编译器优化重排（编译器重排序）

编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

2. 指令并行重排（处理器重排序）

现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性(即后一个执行的语句无需依赖前面执行的语句的结果)，处理器可以改变语句对应的机器指令的执行顺序。

3. 内存系统重排（处理器重排序）

由于处理器使用缓存和读写缓存冲区，这使得加载(load)和存储(store)操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。

指令重排可以保证串行语义一致，但是没有义务保证多线程间的语义也一致。所以在多线程下，指令重排序可能会导致一些问题。在多线程下,是通过内存屏障来实现的，在java1.5之后，引入了happens-before的概念。

## as-if-serial语义
as-if-serial语义的意思是：不管编译器和CPU如何重排序，必须保证在单线程情况下程序的结果是正确的，有依赖关系的数据不能重排序。

## happens-before语义

happens-before是通过内存屏障来实现的

如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM也允许这样的重排序。

## happens-before和as-if-serial：

as-if-serial语义保证单线程内重排序后的执行结果和程序代码本身应有的结果是一致的，happens-before关系保证正确同步的多线程程序的执行结果不被重排序改变。

# synchronized关键字

在多线程并发编程中synchronized一直是元老级角色，很多人都会称呼它为重量级锁。但是，随着JavaSE1.6对synchronized进行了各种优化之后，有些情况下它就并不那么重了。

Java中的每一个对象都可以作为锁。具体表现为以下3种形式。

对于普通同步方法，锁是当前实例对象。

对于静态同步方法，锁是当前类的Class对象。

对于同步方法块，锁是synchronized括号里配置的对象。当一个线程

试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时,必须释放锁。

从JVM规范中可以看到Synchronized在JVM里的实现原理，JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。

代码块同步是使用monitorenter和monitorexit指令实现的，而方法同步是使用另外一种方式实现的，细节在JVM规范里并没有详细说明。但是，方法的同步同样可以使用这两个指令来实现。monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，JVM要保证每个
monitorenter必须有对应的monitorexit与之配对。任何对象都有一个monitor之关联，当且一个monitor被持有后，它将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor的所有权，即尝试获得对象的锁。


## 锁的升级机制


从JDK1.6版本之后，synchronized本身也在不断优化锁的机制，有些情况下他并不会是一个很重量级的锁了。优化机制包括自适应锁、自旋锁、锁消除、锁粗化、轻量级锁和偏向锁。锁的状态从低到高依次为无锁->偏向锁->轻量级锁->重量级锁，升级的过程就是从低到高，降级在一定条件也是有可能发生的。

自旋锁：由于大部分时候，锁被占用的时间很短，共享变量的锁定时间也很短，所有没有必要挂起线程，用户态和内核态的来回上下文切换严重影响性能。自旋的概念就是让线程执行一个忙循环，可以理解为就是啥也不干，防止从用户态转入内核态（操作系统知识），自旋锁可以通过设置-XX:+UseSpining来开启，自旋的默认次数是10次，可以使用-XX:PreBlockSpin设置。

自适应锁：自适应锁就是自适应的自旋锁，自旋的时间不是固定时间，而是由前一次在同一个锁上的自旋时间和锁的持有者状态来决定。

锁消除：锁消除指的是JVM检测到一些同步的代码块，完全不存在数据竞争的场景，也就是不需要加锁，就会进行锁消除。

锁粗化：锁粗化指的是有很多操作都是对同一个对象进行加锁，就会把锁的同步范围扩展到整个操作序列之外。

偏向锁：当线程访问同步块获取锁时，会在对象头和栈帧中的锁记录里存储偏向锁的线程ID，之后这个线程再次进入同步块时都不需要CAS来加锁和解锁了，偏向锁会永远偏向第一个获得锁的线程，如果后续没有其他线程获得过这个锁，持有锁的线程就永远不需要进行同步，反之，当有其他线程竞争偏向锁时，持有偏向锁的线程就会释放偏向锁。可以用过设置-XX:+UseBiasedLocking开启偏向锁。

轻量级锁：JVM的对象的对象头中包含有一些锁的标志位，代码进入同步块的时候，JVM将会使用CAS方式来尝试获取锁，如果更新成功则会把对象头中的状态位标记为轻量级锁，如果更新失败，当前线程就尝试自旋来获得锁。

整个锁升级的过程非常复杂，我尽力去除一些无用的环节，简单来描述整个升级的机制。简单点说，偏向锁就是通过对象头的偏向线程ID来对比，甚至都不需要CAS了，而轻量级锁主要就是通过CAS修改对象头锁记录和自旋来实现，重量级锁则是除了拥有锁的线程其他全部阻塞。

锁升级过程示意图

<img width="735" alt="截屏2022-01-26 下午4 24 52" src="https://user-images.githubusercontent.com/98211272/151128084-737f4284-c9b9-4777-adeb-3b9fede0532d.png">
 
 锁升级
锁的4中状态：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态（级别从低到高）

（1）偏向锁：

为什么要引入偏向锁？

因为经过HotSpot的作者大量的研究发现，大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。

偏向锁的升级

当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的threadID，因为偏向锁不会主动释放锁，因此以后线程1再次获取锁的时候，需要比较当前线程的threadID和Java对象头中的threadID是否一致，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要查看Java对象头中记录的线程1是否存活，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程1，撤销偏向锁，升级为轻量级锁，如果线程1 不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。

偏向锁的取消：

偏向锁是默认开启的，而且开始时间一般是比应用程序启动慢几秒，如果不想有这个延迟，那么可以使用-XX:BiasedLockingStartUpDelay=0；

如果不想要偏向锁，那么可以通过-XX:-UseBiasedLocking = false来设置；

（2）轻量级锁

为什么要引入轻量级锁？

轻量级锁考虑的是竞争锁对象的线程不多，而且线程持有锁的时间也不长的情景。因为阻塞线程需要CPU从用户态转到内核态，代价较大，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它自旋这等待锁释放。

轻量级锁什么时候升级为重量级锁？

线程1获取轻量级锁时会先把锁对象的对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间（称为DisplacedMarkWord），然后使用CAS把对象头中的内容替换为线程1存储的锁记录（DisplacedMarkWord）的地址；

如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间中，但是在线程2CAS的时候，发现线程1已经把对象头换了，线程2的CAS失败，那么线程2就尝试使用自旋锁来等待线程1释放锁。

但是如果自旋的时间太长也不行，因为自旋是要消耗CPU的，因此自旋的次数是有限制的，比如10次或者100次，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，这时又有一个线程3过来竞争这个锁对象，那么这个时候轻量级锁就会膨胀为重量级锁。重量级锁把除了拥有锁的线程都阻塞，防止CPU空转。

2、锁粗化
 
按理来说，同步块的作用范围应该尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，缩短阻塞时间，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。 
但是加锁解锁也需要消耗资源，如果存在一系列的连续加锁解锁操作，可能会导致不必要的性能损耗。 
锁粗化就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。

3、锁消除
 
Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，经过逃逸分析，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间


# volatile关键字
在多线程并发编程中synchronized和volatile都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。

可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。

## volatile的实现原理

volatile在汇编层面给指令加了一个lock前缀

1. Lock前缀指令会引起处理器缓存回写到内存。Lock前缀指令导致在执行指令期间，声言处理器的LOCK#信号。在多处理器环境中，LOCK#信号确保在声言该信号期间，处理器可以独占任何共享内存。但是在最近的处理器里，LOCK＃信号一般不锁总线，而是锁缓存，毕竟锁总线开销比较大。
2. 一个处理器的缓存回写到内存会导致其他处理器的缓存无效。IA-32处理器和、Intel64处理器使用MESI（修改、独占、共享、无效）控制协议去维护内部缓存和其他处理器缓存的一致性。

## MESI协议

现在的CPU一般都有自己的内部缓存，根据一些规则将内存中的数据读取到内部缓存中来，以加快频繁读取的速度。现在服务器通常是多 CPU，更普遍的是，每块CPU里有多个内核，而每个内核都维护了自己的缓存，那么这时候多线程并发就会存在缓存不一致性，这会导致严重问题。
所有内存传输都发生在一条共享总线上，而所有的处理器都能看到这条总线：缓存本身是独立的，但是内存是共享资源，所有的内存访问都要经过仲裁（同一个指令周期中，只有一个CPU缓存可以读内存）。
CPU缓存不仅仅在做内存传输时才与总线打交道，而是不停在嗅探总线上发生的数据交换，跟踪其他缓存在做什么。所以当一个缓存代表它所属的处理器去读写内存时，其他处理器都会得到通知，它们以此来使自己的缓存保持同步。只要某个处理器一写内存，其它处理器马上知道这块内存在它们的缓存段中已失效。
MESI协议是当前最主流的缓存一致性协议，在MESI协议中，每个缓存行有4个状态
<img width="734" alt="截屏2022-01-26 下午4 47 29" src="https://user-images.githubusercontent.com/98211272/151131411-02bd5b99-ed9c-4d3e-9493-d00fcfefc551.png">


这里的I、S和M状态已经有了对应的概念：失效/未载入、干净以及脏的缓存段。所以这里新的知识点只有E状态，代表独占式访问，这个状态解决了"在我们开始修改某块内存之前，我们需要告诉其它处理器"这一问题：只有当缓存行处于E或者M状态时，处理器才能去写它，也就是说只有在这两种状态下，处理器是独占这个缓存行的。当处理器想写某个缓存行时，如果它没有独占权，它必须先发送一条"我要独占权"的请求给总线，这会通知其它处理器把它们拥有的同一缓存段的拷贝失效（如果有）。只有在获得独占权后，处理器才能开始修改数据----并且此时这个处理器知道，这个缓存行只有一份拷贝，在我自己的缓存里，所以不会有任何冲突。反之，如果有其它处理器想读取这个缓存行（马上能知道，因为一直在嗅探总线），独占或已修改的缓存行必须先回到"共享"状态。如果是已修改的缓存行，那么还要先把内容回写到内存中。

# synchronized和ReentrantLock

1. 两者都是可重入锁

“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象 的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0 时才能释放锁。 

2. synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API。
 synchronized 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。 ReentrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成。

3. ReentrantLock 比 synchronized 增加了了一些高级功能。

主要来说主要有三点：
1）等待可中断；

2）可实现公平锁；

3）可实现选择性通知 

ReentrantLock提供了一种能够中断等待锁的的线程的的机制，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。ReentrantLock可以指定是是公平锁还是非公平锁。而synchronized只能是是非公平锁。。所谓的公平锁就是先等待的线程先获得锁。 ReentrantLock默认情况是非公平的，可以通过 ReentrantLock类的 ReentrantLock(boolean fair) 构造方法来制定是否是公平的。 

synchronized关键字与wait()和notify()/notifyAll()方法相结合可以实现等待/通知机制，ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。Condition是JDK1.5之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有有选择性的进行线程通知，在调度线程上更加灵活。在使用notify()/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合 Condition实例可以实现“选择性通知” ，这个功能非常重要，而且是Condition接口默认提供的。而 synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果 执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的 signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。 如果你想使用上述功能，那么选择ReentrantLock是一个不错的选择。 

# 线程安全
多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

## 不可变
不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

1. final 关键字修饰的基本数据类型

2. String

3. 枚举类型

## 互斥同步
synchronized 和 ReentrantLock。

## 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

### CAS
乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

compareAndSwap 中文叫做比较并交换，一种无锁原子算法
它包含 3 个参数 CAS（V，E，N），V表示要更新变量的值，E表示预期值，N表示新值。
仅当 V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程则什么都不做。
最后，CAS 返回当前V的真实值。CAS 操作时抱着乐观的态度进行的，它总是认为自己可以成功完成操作。 它是一种乐观锁

<img width="718" alt="截屏2022-01-26 下午5 19 52" src="https://user-images.githubusercontent.com/98211272/151136209-0d99d792-a15a-4428-911c-c6ab261480e1.png">

应用： java中的Atomic系列就是使用CAS实现的

<img width="704" alt="截屏2022-01-26 下午5 20 19" src="https://user-images.githubusercontent.com/98211272/151136273-4ebc2a54-2de9-4811-ac29-f86d7cda6289.png">



cmpxchg(void* ptr, int old, int new)，如果ptr和old的值一样，则把new写到ptr内存，否则返回ptr的值，整个操作是原子的。

<img width="730" alt="截屏2022-01-26 下午5 20 27" src="https://user-images.githubusercontent.com/98211272/151136302-b79e736e-6b19-456d-9246-ad424b012971.png">

### AtomicInteger
Java.Util.Concurrent包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

#### CAS带来的ABA问题

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

Java.Util.Concurrent包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。


## 无同步方案
要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

1. 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

2. 线程本地存储（Thread Local Storage）

作用：

ThreadLocal的作用主要是做数据隔离，填充的数据只属于当前线程，变量的数据对别的线程而言是相对隔离的，在多线程环境下，如何防止自己的变量被其它线程篡改。 

原理：

每个Thread维护着一个ThreadLocalMap的引用ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value。正因为这个原理，所以ThreadLocal能够实现“数据隔离”，获取当前线程的局部变量值，不受其他线程影响～key为使用弱引用的ThreadLocal实例，value为线程变量的副本。


### 内存泄露

内存泄露为程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光，广义并通俗的说，就是：不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露。
Thread Local引用原理图


<img width="668" alt="截屏2022-01-26 下午5 15 13" src="https://user-images.githubusercontent.com/98211272/151135520-2222d8d7-c250-489a-a300-dc45fb4d8de4.png">


#### 那为什么使用弱引用而不是强引用？？

##### 如果key 使用强引用

当ThreadLocalMap的key为强引用回收ThreadLocal时，因为ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。

##### 如果key 使用弱引用

当ThreadLocalMap的key为弱引用回收ThreadLocal时，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。当key为null，在下一次ThreadLocalMap调用set(),get()，remove()方法的时候会被清除value值。由于Thread中包含变量ThreadLocalMap，因此ThreadLocalMap与Thread的生命周期是一样长，如果都没有手动删除对应key，都会导致内存泄漏。
但是使用弱引用可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set(),get(),remove()的时候会被清除。因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

# AQS

## 概念

AQS的全称为（AbstractQueuedSynchronizer）。AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如 我们提到的ReentrantLock，Semaphore其他的诸如 ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

## 核心思想
如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。 

CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个 结点（Node）来实现锁的分配。

AQS(AbstractQueuedSynchronizer)原理图：



AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS 使用CAS对该同步状态进行原子操作实现对其值的修改。


## 对资源的共享方式 

AQS定义两种资源共享方式：独占和共享
### Exclusive（独占）
只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁： 

公平锁：按照线程在队列中的排队顺序，先到者先拿到锁 

非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的 
### Share（共享）

多个线程可同时执行，

如Semaphore、 CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。 ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。 不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。

## CountDownLatch
用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。



<img width="368" alt="截屏2022-01-26 下午5 25 32" src="https://user-images.githubusercontent.com/98211272/151137249-71942a0e-12b4-4a06-8163-d6a109c55772.png">


<img width="682" alt="截屏2022-01-26 下午5 25 43" src="https://user-images.githubusercontent.com/98211272/151137250-0c9d210f-5379-41c8-be0e-70565c603df1.png">

## CyclicBarrier
用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

## Semaphore
Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。


以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。
<img width="654" alt="截屏2022-01-26 下午5 32 37" src="https://user-images.githubusercontent.com/98211272/151138320-baa39811-a8d6-4cbc-885c-9b45abb8e612.png">

# 多线程开发良好的实践
给线程起个有意义的名字，这样可以方便找 Bug。

缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。

多用同步工具少用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。

使用 BlockingQueue 实现生产者消费者问题。

多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。

使用本地变量和不可变类来保证线程安全。

使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。


## 交替打印abc10次



‘’‘
public class Demo  {

    static ReentrantLock lock = new ReentrantLock();
    static Condition condition1 = lock.newCondition();
    static Condition condition2 = lock.newCondition();
    static Condition condition3 = lock.newCondition();
    private static int num = 1;


    static CountDownLatch countDownLatch = new CountDownLatch(10);
    public static void main(String[] args) throws InterruptedException {

        long loop = countDownLatch.getCount();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i = 0; i < loop; ++i){
                    try {
                        printA();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i = 0; i < loop; ++i){
                    try {
                        printB();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i = 0; i < loop; ++i){
                    try {
                        printC();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();

        countDownLatch.await();

        System.out.println("打印完毕.........");

    }

    public static void printA() throws InterruptedException {

        lock.lock();
        while(num != 1){
            condition1.await();
        }
        System.out.println('A');
        num = 2;
        condition2.signal();
        lock.unlock();

    }

    public static void printB() throws InterruptedException {

        lock.lock();
        while(num != 2){
            condition2.await();
        }
        System.out.println('B');
        num = 3;
        condition3.signal();
        lock.unlock();

    }

    public static void printC() throws InterruptedException {

        lock.lock();
        while(num != 3){
            condition3.await();
        }
        System.out.println('C');
        num = 1;
        condition1.signal();
        countDownLatch.countDown();
        lock.unlock();

    }


}

‘’‘

## 生产者消费者模型

'''
public class ProduceConsumer  {



    static ReentrantLock lock = new ReentrantLock();
    static Condition produce = lock.newCondition();
    static Condition consume = lock.newCondition();
    static int size = 10;
    static int count = 0;

    public static void main(String[] args) {

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for(int i = 0; i < 10; ++i) {
                        Thread.sleep(300);
                        enqueue();
                    }

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for(int i = 0; i < 10; ++i) {
                        Thread.sleep(300);
                        enqueue();
                    }

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for(int i = 0; i < 10; ++i) {
                        Thread.sleep(500);
                        dequeue();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    public static void enqueue() throws InterruptedException {

        lock.lock();
        while(count == size){
            produce.await();
        }
        count++;
        System.out.println(Thread.currentThread() + "目前一共有" + count + "包子");
        consume.signal();
        lock.unlock();
    }

    public static void dequeue() throws InterruptedException {

        lock.lock();
        while(count == 0){
            consume.await();
        }
        count--;
        System.out.println(Thread.currentThread() +"目前一共有" + count + "包子");
        produce.signal();
        lock.unlock();
    }

}

'''
