# JUC概述

## 什么是JUC

JUC 就是 java.util .concurrent工具包的简称。这是一个处理线程的工具包，JDK 1.5开始出现的。

## 进程与线程

### 进程

指在系统中正在运行的一个应用程序；程序一旦运行就是进程；进程——资源分配的最小单位。

### 线程

系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位。

## 线程的状态

### 线程状态的枚举类

Java的Thread.State中定义了线程的状态

```java
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,//新建

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,//准备就绪

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,//阻塞

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,//不见不散

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,//过时不候

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;//
    }
```

### wait和sleep

- sleep 是 Thread 的静态方法，wait 是 Object 的方法，任何对象实例都 能调用。
- sleep 不会释放锁，它也不需要占用锁。wait 会释放锁，但调用它的前提 是当前线程占有锁(即代码要在 synchronized 中。
- 它们都可以被 interrupted 方法中断。他们都是在哪里睡就在哪醒。

## 并发与并行

###  串行模式

串行表示所有任务都一一按先后顺序进行。

串行是一次只能取得一个任务，并执行这个任务。

### 并行模式

并行意味着可以同时取得多个任务，并同时去执行所取得的这些任务。并行模式相当于将长长的一条队列，划分成了多条短队列，所以并行缩短了任务队列 的长度。并行的效率从代码层次上强依赖于多进程/多线程代码，从硬件角度上 则依赖于多核 CPU。

### 并发

并发(concurrent)指的是多个程序可以同时运行的现象，更细化的是多进程可以同时运行或者多指令可以同时运行。这里的"同时运行"表示的不是真的同一时刻有多个 线程运行的现象，这是并行的概念，而是提供一种功能让用户看来多个程序同 时运行起来了，但实际上这些程序中的进程不是一直霸占 CPU 的，而是执行一 会停一会。

要解决大并发问题，通常是将大任务分解成多个小任务, 由于操作系统对进程的 调度是随机的，所以切分成多个小任务后，可能会从任一小任务处执行。这可 能会出现一些现象：

- 可能出现一个小任务执行了多次，还没开始下个任务的情况。这时一般会采用 队列或类似的数据结构来存放各个小任务的成果
- 可能出现还没准备好第一步就执行第二步的可能。这时，一般采用多路复用或 异步的方式，比如只有准备好产生了事件通知才执行某个任务。
- 可以多进程/多线程的方式并行执行这些小任务。也可以单进程/单线程执行这 些小任务，这时很可能要配合多路复用才能达到较高的效率

### 小结

- 并发：同一时刻多个线程在访问同一个资源，多个线程对一个点例子：春运抢票、电商秒杀……
- 并行：多项工作一起执行，之后再汇总例子：泡方便面，电水壶烧水，一边撕调料倒入桶中

## 管程

管程(monitor)是保证了同一时刻只有一个进程在管程内活动,即管程内定义的操作在同 一时刻只被一个进程调用(由编译器实现)。但是这样并不能保证进程以设计的顺序执行

JVM 中同步是基于进入和退出管程(monitor)对象实现的，每个对象都会有一个管程 (monitor)对象，管程(monitor)会随着 java 对象一同创建和销毁

执行线程首先要持有管程对象，然后才能执行方法，当方法完成之后会释放管程，方法在执行时候会持有管程，其他线程无法再获取同一个管程

## 用户线程和守护线程

用户线程：平时用到的普通线程，自定义线程

守护线程：运行在后台，是一种特殊的线程，比如垃圾回收。

当主线程结束后，用户线程还在运行，JVM 存活。

如果没有用户线程，都是守护线程，JVM 结束。守护线程和JVM一起结束

查看是否为守护线程：

```java
//看当前线程是否为守护线程
Thread.currentThread().isDaemon()
```

设置线程为守护线程：

```java
//演示用户线程和守护线程
public class Main {

    public static void main(String[] args) {
        Thread aa = new Thread(() -> {
            //Thread.currentThread().isDaemon()打印当前线程是否为守护线程
            System.out.println(Thread.currentThread().getName() + "::" + Thread.currentThread().isDaemon());
            while (true) {
            }
        }, "aa");
        //设置守护线程
        aa.setDaemon(true);
        aa.start();

        System.out.println(Thread.currentThread().getName()+" over");
    }
}
```

# Lock接口

## Synchronized

### Synchronized 关键字回顾

synchronized 是 Java 中的关键字，是一种同步锁。它修饰的对象有以下几种：

- 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{} 括起来的代码，作用的对象是调用这个代码块的对象；
- 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用 的对象是调用这个方法的对象；
  - 虽然可以使用 synchronized 来定义方法，但 synchronized 并不属于方法定义的一部分，因此，synchronized 关键字不能被继承。如果在父类中的某个方法使用了 synchronized 关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上 synchronized 关键字才可以。当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。
- 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的 所有对象；
- 修改一个类，其作用的范围是 synchronized 后面括号括起来的部分，作用主的对象是这个类的所有对象。

### 售票案例

```java
//第一步  创建资源类，定义属性和和操作方法
class Ticket {

    //票数
    private int number = 30;

    //操作方法：买票
    public synchronized void sale() {
        //判断：是否有票
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + " : 卖出：" + (number--) + " 剩下：" + number);
        }
    }
}

public class SaleTicket {
    //第二步：创建多个线程，调用资源类的操作方法
    public static void main(String[] args) {
        //创建ticket对象
        Ticket ticket = new Ticket();
        //创建三个线程
        //线程一
        new Thread(new Runnable() {
            @Override
            public void run() {
                //调用
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        }, "AA").start();
        //线程二
        new Thread(new Runnable() {
            @Override
            public void run() {
                //调用
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        }, "BB").start();
        //线程三
        new Thread(new Runnable() {
            @Override
            public void run() {
                //调用
                for (int i = 0; i < 40; i++) {
                    ticket.sale();
                }
            }
        }, "CC").start();
    }
}
```

如果一个代码块被 synchronized 修饰了，当一个线程获取了对应的锁，并执 行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里 获取锁的线程释放锁只会有两种情况：

- 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
- 线程执行发生异常，此时 JVM 会让线程自动释放锁；

那么如果这个获取锁的线程由于要等待 IO 或者其他原因（比如调用 sleep 方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，试想一 下，这多么影响程序执行效率。

因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等 待一定的时间或者能够响应中断），通过 Lock 就可以办到。

## 什么是 Lock

Lock 锁实现提供了比使用同步方法和语句可以获得的更广泛的锁操作。它们允许更灵活的结构，可能具有非常不同的属性，并且可能支持多个关联的条件对象。Lock 提供了比 synchronized 更多的功能。

Lock 与的 Synchronized 区别：

- Lock 不是 Java 语言内置的，synchronized 是 Java 语言的关键字，因此是内 置特性。Lock 是一个类，通过这个类可以实现同步访问；
- Lock 和 synchronized 有一点非常大的不同，采用 synchronized 不需要用户 去手动释放锁，当 synchronized 方法或者 synchronized 代码块执行完之后， 系统会自动让线程释放对锁的占用；而 Lock 则必须要用户去手动释放锁，如 果没有主动释放锁，就有可能导致出现死锁现象。

### Lock 接口

```java
public interface Lock {

    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();

    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    void unlock();

    Condition newCondition();
}
```

### 接口中的方法

#### lock

lock()方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。 采用 Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一 般来说，使用 Lock 必须在 try{}catch{}块中进行，并且将释放锁的操作放在 finally 块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用 Lock 来进行同步的话，是以下面这种形式去使用的：

```java
//第一步  创建资源类，定义属性和和操作方法
class LTicket {
    //票数量
    private int number = 30;

    //创建可重入锁
    private final ReentrantLock lock = new ReentrantLock(true);

    //卖票方法
    public void sale() {
        //上锁
        lock.lock();
        try {
            //判断是否有票
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + " ：卖出" + (number--) + " 剩余：" + number);
            }
        } finally {
            //解锁
            lock.unlock();
        }
    }
}

public class LSaleTicket {
    //第二步 创建多个线程，调用资源类的操作方法
    //创建三个线程
    public static void main(String[] args) {

        LTicket ticket = new LTicket();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "AA").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "BB").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "CC").start();
    }
}
```

#### newCondition

关键字 synchronized 与 wait()/notify()这两个方法一起使用可以实现等待/通 知模式，Lock 锁的 newContition()方法返回 Condition 对象，Condition 类 也可以实现等待/通知模式。 用 notify()通知时，JVM 会随机唤醒某个等待的线程， 使用 Condition 类可以进行选择性通知， Condition 比较常用的两个方法：

- await()会使当前线程等待,同时会释放锁,当其他线程调用 signal()时,线程会重 新获得锁并继续执行。
- signal()用于唤醒一个等待的线程。

**注意**：在调用 Condition 的 await()/signal()方法前，也需要线程持有相关 的 Lock 锁，调用 await()后线程会释放这个锁，在 singal()调用后会从当前 Condition 对象的等待队列中，唤醒 一个线程，唤醒的线程尝试获得锁， 一旦 获得锁成功就继续执行。

#### ReentrantLock

ReentrantLock，意思是“可重入锁”，关于可重入锁的概念将在后面讲述。

ReentrantLock 是唯一实现了 Lock 接口的类，并且 ReentrantLock 提供了更 多的方法。

#### ReadWriteLock

ReadWriteLock 也是一个接口，在它里面只定义了两个方法：

```java
public interface ReadWriteLock {
 /**
 * Returns the lock used for reading.
 *
 * @return the lock used for reading.
 */
 Lock readLock();
 /**
 * Returns the lock used for writing.
 *
 * @return the lock used for writing.
 */
 Lock writeLock();
}
```

## 小结（重点）

Lock 和 synchronized 有以下几点不同：

1. Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内 置的语言实现；
2. synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁；
3. Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断；
4. 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
5. Lock 可以提高多个线程进行读操作的效率。 在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时 Lock 的性能要远远优于synchronized。

# 线程间的通信

线程间通信的模型有两种：共享内存和消息传递，以下方式都是基本这两种模型来实现的。我们来基本一道面试常见的题目来分析

场景：两个线程，一个线程对当前数值加 1，另一个线程对当前数值减 1，要求用线程间通信

## synchronized 方案

```java
class Share {
    //初始值
    private int number = 0;

    //+1的方法
    public synchronized void incr() throws InterruptedException {
        //第二步 判断 干活 通知
        //应为wait()是在那睡就在那醒，所以导致虚假唤醒（多个线程导致唤醒出错）
        /*if (number != 0) {//判断number值是否为0，如果不是则等待
            this.wait();
        }*/
        while(number != 0) { //判断number值是否是0，如果不是0，等待
            this.wait(); //在哪里睡，就在哪里醒
        }
        //如果number值是0，就+1操作
        number++;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知其他线程
        this.notifyAll();
    }

    //-1的方法
    public synchronized void decr() throws InterruptedException {
        //第二步 判断 干活 通知
        //应为wait()是在那睡就在那醒，所以导致虚假唤醒（多个线程导致唤醒出错）
        /*if (number != 1) {//判断number值是否为0，如果不是则等待
            this.wait();
        }*/
        while(number != 0) { //判断number值是否是0，如果不是0，等待
            this.wait(); //在哪里睡，就在哪里醒
        }
        //如果number值是1，就-1操作
        number--;
        System.out.println(Thread.currentThread().getName() + " :: " + number);
        //通知其他线程
        this.notifyAll();
    }
}

public class ThreadDemo1 {
    public static void main(String[] args) {
        Share share = new Share();
        //+1
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        share.incr();
                        System.out.println("123");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "AA").start();
        //-1
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        share.decr();
                        System.out.println("456");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "BB").start();
    }
}
```



##  Lock 方案



# 线程间制定通信

# 集合的线程的安全

# 多线程锁

# Callable接口

# JUC强大的辅助类

# ReentrantReadWriteLock读写锁

# BlockingQueue阻塞队列

# ThreadPool线程池

# Fork/Join分支合并框架

# CompletableFuture异步回调