---
title: java核心卷Ⅰ-并发
date: 2019-03-31 14:20:45
tags:
  - java
categories:
  - 读书笔记
  - java核心卷1
coments: true
---

看了很久的书，总算对多线程并发有所了解，记录一下

<!--more-->

# 了解知识



## 基础知识

操作系统

我们知道一台计算机的资源是有限的，我们需要利用有限的资源完成很多事情，但是又要保证公平，那么我们该怎么做呢？操作系统是这样做的，在这段时间内（时间片）将这些资源给你用，其他人不能做。过了这段时间，又将资源分给其他人做。就像小时候吃蛋糕，蛋糕只有一个，我想吃，姐姐也想吃，又不能同时吃，又不能切开，怎么办？你吃一口，然后我吃一口啊。



线程和进程：

进程就是我们电脑有很多软件同时运行，比如QQ、IDEA、360啊

进程就是我们在用QQ的时候，同时接受n个小姐姐的消息，我们又给别人发文件。

不懂得自行谷歌：放一个<a href="https://www.cnblogs.com/tiankong101/p/4229584.html">链接</a>



并发和并行

- 并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS或者QPS来反应这个系统的处理能力。
- 并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是真正的同时。



## 如何编写多线程程序捏~(￣▽￣)~*

1. 继承`Thread`类，重写该类的`run()`方法

```java
class MyThread extends Thread {

    private int i = 0;

    @Override
    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```

```java
public class ThreadTest {

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Thread myThread1 = new MyThread();     // 创建一个新的线程  myThread1  此线程进入新建状态
                Thread myThread2 = new MyThread();     // 创建一个新的线程 myThread2 此线程进入新建状态
                myThread1.start();                     // 调用start()方法使得线程进入就绪状态
                myThread2.start();                     // 调用start()方法使得线程进入就绪状态
            }
        }
    }
}
```

继承`Thread`类，重写`run`方法，`run`方法里面的就是线程执行体。当创建此线程类对象时一个新线程得以创建，并进入线程新建状态。通过调用线程对象引用的start()方法，使得该线程进入到就绪状态，此时<span style="red">此线程并不一定会马上得以执行，这取决于CPU调度时机。</span>

1. 实现`Runnable`接口，实现`run()`方法，该`run()`方法同样是线程执行体。创建`Runnable`实现类的实例，并以此实例作为`Thread`类的`target`来创建`Thread`对象，该`Thread`对象才是真正的线程对象。此处是不是很萌币，接下来看代码，要静心哦。

```java
class MyRunnable implements Runnable {
    private int i = 0;

    @Override
    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```

```java
public class ThreadTest {

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Runnable myRunnable = new MyRunnable(); // 创建一个Runnable实现类的对象
                Thread thread1 = new Thread(myRunnable); // 将myRunnable作为Thread target创建新的线程
                Thread thread2 = new Thread(myRunnable);
                thread1.start(); // 调用start()方法使得线程进入就绪状态
                thread2.start();
            }
        }
    }
}
```



走到这里是不是有些想法，怎么这么麻烦。有了Thread为什么还要用Runnable，是来抢我饭碗的吗？？？还真是的，来抢Thread的饭碗的。

1. 使用Runnable能够避免单继承的局限，可以实现多个接口
2. Runnable适合资源的共享，Thread每次创建一个，都是不同的对象，都分家了，不适合共享。
3. Thread和Runnable关系

```java
public class ThreadTest {

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Runnable myRunnable = new MyRunnable();
                Thread thread = new MyThread(myRunnable);
                thread.start();
            }
        }
    }
}

class MyRunnable implements Runnable {
    private int i = 0;

    @Override
    public void run() {
        System.out.println("in MyRunnable run");
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}

class MyThread extends Thread {

    private int i = 0;
    
    public MyThread(Runnable runnable){
        super(runnable);
    }

    @Override
    public void run() {
        System.out.println("in MyThread run");
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```





​	这个时候你可能会有疑惑，到底这里到底是使用Thread的run()方法呢，还是使用MyThread方法呢，咚咚脑子，发现是MyThread继承重写了Thread的Run()，那么现在是运行那个一目了然。

​	底层探究一波：

​	Runnable接口

```java
 public interface Runnable {
    
     public abstract void run();
     
 }
```

​	Thread类

​	其实在底层，Thread是实现了Runnable方法滴。Thread对run方法的实现

```java
@Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```



​	也就是说，当执行到Thread类中的run()方法时，会首先判断target是否存在，存在则执行target中的run()方法，也就是实现了Runnable接口并重写了run()方法的类中的run()方法。但是上述给到的列子中，由于多态的存在，根本就没有执行到Thread类中的run()方法，而是直接先执行了运行时类型即MyThread类中的run()方法。

​	现在我们是不是可以理解之前为什么要用Thread(Runnable)了呢。

4. 使用Callable和Future接口创建线程。具体是创建Callable接口的实现类，并实现call()方法。并使用FutureTask类来包装Callable实现类的对象，且以此FutureTask对象作为Thread对象的target来创建线程。

```java
public class ThreadTest {

    public static void main(String[] args) {

        Callable<Integer> myCallable = new MyCallable();    // 创建MyCallable对象
        FutureTask<Integer> ft = new FutureTask<Integer>(myCallable); //使用FutureTask来包装MyCallable对象

        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Thread thread = new Thread(ft);   //FutureTask对象作为Thread对象的target创建新的线程
                thread.start();                      //线程进入到就绪状态
            }
        }

        System.out.println("主线程for循环执行完毕..");
        
        try {
            int sum = ft.get();            //取得新创建的新线程中的call()方法返回的结果
            System.out.println("sum = " + sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}


class MyCallable implements Callable<Integer> {
    private int i = 0;

    // 与run()方法不同的是，call()方法具有返回值
    @Override
    public Integer call() {
        int sum = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            sum += i;
        }
        return sum;
    }

}
```

首先，我们发现，在实现Callable接口中，此时不再是run()方法了，而是call()方法，此call()方法作为线程执行体，同时还具有返回值！在创建新的线程时，是通过FutureTask来包装MyCallable对象，同时作为了Thread对象的target。那么看下FutureTask类的定义：

```java
 public class FutureTask<V> implements RunnableFuture<V> {
     
     //....
     
 }
```

```java
 public interface RunnableFuture<V> extends Runnable, Future<V> {
     
     void run();
     
 }
```

于是，我们发现FutureTask类实际上是同时实现了Runnable和Future接口，由此才使得其具有Future和Runnable双重特性。通过Runnable特性，可以作为Thread对象的target，而Future特性，使得其可以取得新创建线程中的call()方法的返回值。

执行下此程序，我们发现sum = 4950永远都是最后输出的。而“主线程for循环执行完毕..”则很可能是在子线程循环中间输出。由CPU的线程调度机制，我们知道，“主线程for循环执行完毕..”的输出时机是没有任何问题的，那么为什么sum =4950会永远最后输出呢？

原因在于通过ft.get()方法获取子线程call()方法的返回值时，当子线程此方法还未执行完毕，ft.get()方法会一直阻塞，直到call()方法执行完毕才能取到返回值。



三部走下来是不是感觉他们似乎是包装在包装，增强在增强，我怀疑这是动态增强，不过源码我还没有深入，以后会深入的啦



以上编写多线程摘自：<a href="http://www.cnblogs.com/lwbqqyumidi/p/3804883.html">我是一个链接</a>



# 线程状态

放大招。。。

![线程状态](https://i.loli.net/2019/03/26/5c99f2699eec8.jpg)



线程有五个状态：

**新建状态（New）：**当线程对象对创建后，即进入了新建状态，如：Thread t = new MyThread();

**就绪状态（Runnable）：**当调用线程对象的start()方法（t.start();），线程即进入就绪状态。**处于就绪状态的线程，只是说明此线程已经做好了准备，随时等待CPU调度执行，并不是说执行了t.start()此线程立即就会执行；**

**运行状态（Running）：**当CPU开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。注：**就绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中；**

**阻塞状态（Blocked）：**处于运行状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才 有机会再次被CPU调用以进入到运行状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：

1. 等待阻塞：运行状态中的线程执行`wait()`方法，使本线程进入到等待阻塞状态，使该线程处于等待池`(wait blocked pool)`,直到`notify()/notifyAll()`，线程被唤醒被放到锁定池`(lock blocked pool )`，释放同步锁使线程回到可运行状态`（Runnable）`；
2. 同步阻塞 -- 线程在获取`synchronized`同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态（`lock blocked pool`）；
3. 其他阻塞 -- 通过调用线程的`sleep()`或`join()`或发出了I/O请求时，线程会进入到阻塞状态。当`sleep()`状态超时、`join()`等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。注：<span style="color:red">此时锁还在他身上。</span>

**死亡状态（Dead）：**线程执行完了或者因异常退出了`run()`方法，该线程结束生命周期。

> 特别是， 可以调用线程的`stop `方法杀死一个线程。该方法抛出`ThreadDeath `错误对象,由此杀死线程。但是，stop 方法已过时， 不要在自己的代码中调用这个方法。



特别的：在`runnable`状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread类中的`yield`方法可以让一个`running`状态的线程转入`runnable`。注：此时同样可以立刻被运行，如果此时线程没有阻塞、且他的优先级最高或并列最高





# 中断程序

**使用interrupt可以中断程序，但他不是真正的中断运行中的程序，只能改变中断状态**。我们可以使用stop方法中断线程，但这是不安全。让我吃饭吃到一半我不得和你拼命？？？

> 没有可以强制线程终止的方法。然而， interrupt 方法可以用来请求终止线程。当对一个线程调用interrupt 方法时，线程的中断状态将被置位。**这是每一个线程都具有的boolean 标志。每个线程都应该不时地检査这个标志， 以判断线程是否被中断。**

所以我们使用的时候会检测代码是否中断。

```java
while(!Thread.currentThread().isInterrupted){
    巴拉巴拉啊啦......
}
```

- 如果线程被阻塞， 就无法检测中断状态。这是产生InterruptedExceptioii 异常的地方。

- 当在一个被阻塞的线程（调用sleep 或wait ) 上调用interrupt 方法时， 阻塞调用将会被Interrupted Exception 异常中断。

  

> 中断一个线程不过是引起它的注意。被中断的线程可以决定如何响应中断。某些线程是如此重要以至于应该处理完异常后， 继续执行， 而不理会中断。但是，更普遍的情况是，线程将简单地将中断作为一个终止的请求



不要在try catch中线程异常处理 可能会有一些坑，有哪些坑我就不知道了，前辈的苦药还是要吃的

```java
void mySubTask() throws InterruptedException{
    sleep(delay) ;
}
```







# 线程属性

## 线程优先级

1. 线程的优先级用1-10，一般使用三个值：

   - 最低优先级 1：`Thread.MIN_PRIORITY`
   - 最高优先级 10：`Thread.MAX_PRIORITY`
   - 普通优先级 5：`Thread.NORM_PRIORITY`

2. 可以使用`setPriority`方法设置优先级

3. 使用`Thread.currentThread().getPriority()`获取线程优先级

4. **Java 默认的线程优先级是父线程的优先级，而非普通优先级Thread.NORM_PRIORITY**，因为**主线程默认优先级是普通优先级`Thread.NORM_PRIORITY`，**所以如果不主动设置线程优先级，则新创建的线程的优先级就是普通优先级`Thread.NORM_PRIORITY`

5. 最大优先级

   - 一般在使用线程组的时候设置线程组的优先级。
   - 系统线程组默认最大优先级`Thread.NORM_PRIORITY`
   - 如果在创建线程组之前已经有线程且线程级别比线程组最大优先级高，此时不会削减此线程的优先级，依旧使用此线程的优先级，只有在修改该线程优先级时此线程组的最大优先级才会起作用。
   - 在拥有线程组之后创建的子线程，最大优先级为线程组的优先级。修改也无法超过。

6. 使用线程的时候尽量少设置线程优先级，可能会有乱七八糟的bug。线程的优先级是基于操作系统的，优先级高只能说明他会优先被操作系统分配资源运行，其他线程同样有机会运行。`setPriority`只是局部的使用，应该和线程组、父线程组合使用。

7. 一般`Thread.吧利巴鲁`表示`main`线程。

   

## 守护线程

1. 线程分为前台线程和后台线程（守护线程）
2. 后台线程为前台线程提供服务，在前台线程运行完之后，后台线程才会dead
3. 使用`setDaemon()`方法将线程设置为后台线程
4. main线程为前台线程

## 未捕捉线程异常

处理器必须属于一个实现`Thread.UncaughtExceptionHandler `接口的类。这个接口只有—个方法。

```java
    public interface UncaughtExceptionHandler {
        /**
         * Method invoked when the given thread terminates due to the
         * given uncaught exception.
         * <p>Any exception thrown by this method will be ignored by the
         * Java Virtual Machine.
         * @param t the thread
         * @param e the exception
         */
        void uncaughtException(Thread t, Throwable e);
    }
```

- 可以用`setUncaughtExceptionHandler` 方法为任何线程安装一个处理器。
- 也可以用`Thread`类的静态方法`setDefaultUncaughtExceptionHandler` 为所有线程安装一个默认的处理器。

```java
    // null unless explicitly set
    private volatile UncaughtExceptionHandler uncaughtExceptionHandler;

    // null unless explicitly set
    private static volatile UncaughtExceptionHandler defaultUncaughtExceptionHandler;
```



- 如果不安装默认的处理器， 默认的处理器为空。但是， 如果不为独立的线程安装处理器，此时的处理器就是该线程的`ThreadGroup` 对象。`ThreadGroup `类实现`Thread.UncaughtExceptionHandler `接口。它的`uncaughtException `方法做如下操作（下面的内容不是很了解）

  1 ) 如果该线程组有父线程组， 那么父线程组的`uncaughtException `方法被调用。
  2 ) 否则， 如果`Thread.getDefaultExceptionHandler` 方法返回一个非空的处理器， 则调用该处理器。
  3 ) 否则， 如果`Throwable `是`ThreadDeath` 的一个实例， 什么都不做。
  4 ) 否则， 线程的名字以及`Throwable` 的栈轨迹被输出到`System.err` 上。

```java
    public void uncaughtException(Thread t, Throwable e) {
        if (parent != null) {//如果该线程组有父线程组， 那么父线程组的`uncaughtException `方法被调用。
            parent.uncaughtException(t, e);
        } else {//如果`Thread.getDefaultExceptionHandler` 方法返回一个非空的处理器， 则调用该处理器
            Thread.UncaughtExceptionHandler ueh =
                Thread.getDefaultUncaughtExceptionHandler();
            if (ueh != null) {//如果`Throwable `是`ThreadDeath` 的一个实例， 什么都不做。
                ueh.uncaughtException(t, e);
            } else if (!(e instanceof ThreadDeath)) {//线程的名字以及`Throwable` 的栈轨迹被输出到`System.err` 上。
                System.err.print("Exception in thread \""
                                 + t.getName() + "\" ");
                e.printStackTrace(System.err);
            }
        }
    }
```

- 如果没有设置线程组，将会使用Main线程的异常处理，在`Thread.init()`中有处理，上源码，`ThreadGroup g`

```java
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
```





# 线程同步

## 锁对象

Java SE 5.0 引入了`ReentrantLock `类，该类实现`Lock`接口方法。Lock方式是JDK层面的提供给开发人员的接口，因此开发人员在使用它来解决并发问题时，需要手动获取锁和释放锁。Lock接口源码

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

主要使用方法：

`Lock()`:加锁,如果当前锁被其他线程获取，该线程将进入等待队列，直到使用当前锁的线程释放当前锁。**使用Lock必须在try…catch…块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生**

`unLock()`：解锁

其他方法：

`lockInterruptibly()`:**当通过这个方法去获取锁时，如果线程 正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态**

`tryLock`:加锁，有返回值。当前锁被其他线程占用的时候，会立刻返回false，不会等待，其重载方法会等待一段时间，若没有空闲锁，返回false

注意，当一个线程获取了锁之后，是不会被`interrupt()`方法中断的。因为**`interrupt()`方法只能中断阻塞过程中的线程而不能中断正在运行过程中的线程**。因此，当通过`lockInterruptibly()`方法获取某个锁时，如果不能获取到，那么只有进行等待的情况下，才可以响应中断的。与` synchronized `相比，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

代码：

```java
	package bingxing;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Bank {

    /**
     * 总金额
     */
    private final double[] accounts;
    /**
     * 锁
     */
    private Lock backLock;
    /**
     * 锁相关的条件对象
     */
    private Condition sufficientFunds;

    /**
     * 初始化
     * @param n
     * @param initialBalance
     */
    public Bank(int n, double initialBalance){
        accounts=new double[n];
        for(int i=0;i<accounts.length;i++){
            accounts[i]=initialBalance;
        }
        backLock=new ReentrantLock();//构建一个可重入保护区的锁
        sufficientFunds = backLock.newCondition();//返回一个与该锁相关的条件对象
    }

    /**
     * 获取总金额，使用锁防止查询的同时存在修改情况
     * @return
     */
    public double getTotalBalance(){
        backLock.lock();
        try{
            double sum=0;
            for (double a :
                    accounts) {
                sum += a;
            }
            return sum;
        }finally {
            backLock.unlock();
        }
    }

    /**
     * 转账 使用锁，防止多个对象对同一账户操作
     * 使用条件对象，防止出现总金额少于转账金额 设置条件对象 当账户金额多余转账金额，触发接下来的操作
     * @param from
     * @param to
     * @param amount
     * @throws InterruptedException
     */
    public void transfer(int from,int to,double amount) throws InterruptedException{
        //锁
        backLock.lock();
        try{
            //执行业务操作
            while(accounts[from]<amount){
                //总数少于转出的数量 设置条件保存对象 释放锁给其他线程，等待其他线程条件激活
                sufficientFunds.await();//将该条件放到线程等待集中
            }
            System.out.print(Thread.currentThread());//输出当前线程
            //执行操作
            accounts[from] -= amount;
            System.out.printf("%10.2f from %d to %d       ",amount,from,to);
            accounts[to] +=amount;
            System.out.printf("Total Balance: %10.2f%n",getTotalBalance());
            sufficientFunds.signalAll();//解除该线程等待集中的所有堵塞线程
        }finally {
            //解锁
            backLock.unlock();
        }

    }

    /**
     * 计算账户的个数
     * @return the number of accounts
     */
    public int size(){
        return accounts.length;
    }
}

```

```java
package bingxing;

public class SynchBankTest {
    /**
     * 设置账户数量
     */
    public static final int NACCOUNTS = 100;
    /**
     * 设置账户金额
     */
    public static final double INITIAL_BALANCE = 1000;

    public static void main(String[] args) {
        Bank b=new Bank(NACCOUNTS,INITIAL_BALANCE);
        int i;
        for(i=0;i<NACCOUNTS;i++){
            TransferRunnable r=new TransferRunnable(b,i,INITIAL_BALANCE);
            Thread t=new Thread(r);
            t.start();
        }
    }
}

```

```java
package bingxing;

public class TransferRunnable implements Runnable {

    private Bank bank;
    private int fromAccount;
    private  double maxAmount;
    private int DELAY=10;

    public TransferRunnable(Bank bank, int fromAccount, double maxAmount) {
        this.bank = bank;
        this.fromAccount = fromAccount;
        this.maxAmount = maxAmount;
    }

    @Override
    public void run() {
        try{
            while (true){
                int toAccount = (int)(bank.size()*Math.random());
                double amount = maxAmount*Math.random();
                bank.transfer(fromAccount,toAccount,amount);
                Thread.sleep((int)(DELAY*Math.random()));
            }
        }catch (InterruptedException e){

        }
    }
}

```





补充：

1. `ReentrantLock`，即可重入锁。**`ReentrantLock`是唯一实现了Lock接口的类，并且ReentrantLock提供了更多的方法。**
2. 使用锁时应该将锁放在全局变量，如果放在局部变量，那么每个调用的对象都会拥有自己锁，不能实现锁共享

```java
ReentrantLock()
//构建一个可以被用来保护临界区的可重入锁。
ReentrantLock(boo1ean fair )
//构建一个带有公平策略的锁。一个公平锁偏爱等待时间最长的线程。但是，这一公平的保证将大大降低性能。所以， 默认情况下， 锁没有被强制为公平的。
```



## 条件对象

> 通常， 线程进人临界区，却发现在某一条件满足之后它才能执行。要使用一个条件对象来管理那些已经获得了一个锁但是却不能做有用工作的线程。我们使用条件对象（条件变量conditional variable）

在书中如果A用户需要向B用户转账，但是A用户没有足够多的金额，这时候该怎么办呢？

此时A用户需要等待其他用户给他转账，直到他的总金额满足转账金额，此时A一直霸占着锁，其他用户无法获取锁，这就带来问题，于是使用`newCondition()`方法创建一个条件对象，当该用户由于总金额不足，调用`await()`方法时（`await()`方法没办法自己激活自己）。会将锁交给其他线程，自己进入等待集（阻塞）中，直到其他线程激活等待的线程。

```java
            while(accounts[from]<amount){
                //总数少于转出的数量 设置条件保存对象 释放锁给其他线程，等待其他线程条件激活
                sufficientFunds.await();//将该条件放到线程等待集中
            }
```

- `signal()`:随机解除等待集中某个线程的阻塞情况。有可能会出现死锁的情况
- `signalAll()`:解除等待线程的阻塞，以便这些线程可以在当前线程退出同步方法之后，通过竞争实现对对象的访问。
- 一个锁对象可以有多个相关大的条件对象

> 当一个线程拥有某个条件的锁时， 它仅仅可以在该条件上调用await、signalAll 或signal 方法

> 等待获得锁的线程和调用await 方法的线程存在本质上的不同。一旦一个线程调用await方法， 它进人该条件的等待集。当锁可用时，该线程不能马上解除阻塞。相反，它处于阻塞状态，直到另一个线程调用同一条件上的signalAll 方法时为止。



总结一下（书里面滴）<span style="color:red">重点</span>

- 锁用来保护代码片段， 任何时刻只能有一个线程执行被保护的代码。
- 锁可以管理试图进入被保护代码段的线程。
- 锁可以拥有一个或多个相关的条件对象。
- 每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程。



栗子代码在锁对象哪节里

```java
Condition newCondition()
//返回一个与该锁相关的条件对象。
    
void await( )
//将该线程放到条件的等待集中。
    
void signalA11( )
//解除该条件的等待集中的所有线程的阻塞状态。
    
void signal ( )
//从该条件的等待集中随机地选择一个线程， 解除其阻塞状态。
    
```



## synchronized关键字

> Java中的每一个对象都有一个内部锁。如果一个方法用synchronized 关键字声明，那么对象的锁将保护整个方法。也就是说，要调用该方法， 线程必须获得内部的对象锁。



> 内部对象锁只有一个相关条件。`wait `方法添加一个线程到等待集中，`notifyll /notify `方法解除等待线程的阻塞状态。换句话说，调用`wait `或`notityAll `等价于
>
> ```java
> intrinsicCondition.await();
> intrinsicCondition.signalAll() ;
> ```

内部锁和条件局限性

- 不能中断一个正在试图获得锁的线程。
- 试图获得锁时不能设定超时。
- 每个锁仅有单一的条件， 可能是不够的

使用方法(摘自书中)：

第一种作用于方法：

```java
class Bank
{
	private double accounts;
	public synchronized void transfer(int from，int to, int amount) throws InterruptedException
    {
    	while (accounts[from] < amount)
    		wait(); // wait on intrinsic object lock's single condition
    	accounts[from] -= amount ;
    	accounts[to] += amount ;
    	notifyAll()；// notify all threads waiting on the condition
    }
	public synchronized double getTotalBalanceO { . . . }
}
```

第二种 作用于代码块，如下，在多线程环境下，`synchronized`块中的方法获取了`lock`实例的`monitor`，如果实例相同，那么只有一个线程能执行该块内容：

```java
public class Thread1 implements Runnable {
   Object lock;
   public void run() {  
       synchronized(lock){
         ..do something
       }
   }
}
```



<span style="color:red">重点</span>：静态方法声明synchronized 和普通方法声明

java中锁的概念。一个是实例锁（锁在某一个实例对象上，如果该类是单例，那么该锁也具有全局锁的概念），一个是全局锁（该锁针对的是类，无论实例多少个对象，那么线程都共享该锁）。实例锁对应的就是synchronized关键字，而类锁（全局锁）对应的就是static synchronized（或者是锁在该类的class或者classloader对象上）**普通方法声明是指锁定那个方法，其他线程不能使用。而静态方法声明会锁定那个类对象，就是含有那个锁定方法的类将会被锁定**

不同对象，他们的锁是不一样的



## 同步阻塞

这一部分用的较少，理解不是很透彻

> 每一个Java 对象有一个锁。线程可以通过调用同步方法获得锁。还有另一种机制可以获得锁，通过进入一个同步阻塞

```java
public class Bank
{
	private doublet] accounts;
	private Object lock = new Object。;
	public void transfer(int from, int to, int amount)
	{
		synchronized (lock) // an ad-hoc lock
		{
			accounts[from] -= amount;
			accounts[to] += amount;
		}
		System.out.print1n(..
	}
}
```

他的目的就是调用传入对象的锁，每个对象的操作都是原子性的。但是他的粒度太小，导致无法保证原子性操作。比如我们传入的是这个对象，这个对象的操作是原子性的，是安全的，但是对这个对象的操作不是原子性的，是无法保证线程安全的。

离子：

![](https://i.loli.net/2019/03/28/5c9c3ef9edb6b.jpg)



## 监视器

​	**监视器是一种同步结构**，它允许线程同时互斥（使用锁）和协作，即使用等待集（**wait-set**）使线程等待某些条件为真的能力

​	使用比较形象的说明，监视器就像一个包含一个特殊房间（对象实例）的建筑物，每次只能占用一个线程。这个房间通常包含一些需要防止并发访问的数据。从一个线程进入这个房间到它离开的时间，它可以独占访问房间中的任何数据。进入监控的建筑被称为“进入监控监视器。”进入建筑内部特殊的房间叫做“获取监视器”。房间占领被称为“拥有监视器”，离开房间被称为“释放监视器。”让整个建筑被称为“退出监视器。”

​	当一个线程访问受保护的数据（进入特殊的房间）时，它首先在建筑物接收（**entry-set**）中排队。如果没有其他线程在等待（拥有监视器），线程获取锁并继续执行受保护的代码。当线程完成执行时，它释放锁并退出大楼（退出监视器）。

​	如果当一个线程到达并且另一个线程已经拥有监视器时，它必须在接收队列中等待（**entry-set**）。当当前所有者退出监视器时，新到达的线程必须与在入口集中等待的其他线程竞争。只有一个线程能赢得竞争并拥有锁。

![](https://i.loli.net/2019/03/28/5c9c96bd371c7.png)

​	正如上图所示，在该栋房子中，一共会有三个房间。如果一个客户(线程)想要占领这个特殊的房间，它必须首先进入到Entry Set 中进行等待。调度程序将会基于某种策略(比如，FIFO),从Entry Set 中选择某一个客户。如果这个客户(线程)因为某些事件或原因被挂起了，该客户进离开 特殊房间 而进入 等待房间Wait Room，以后，调度程序可能会重新选择它，把它从 等待房间 放入到 特殊房间中。

​	synchronized就是基于这个实现的。

## volatile域

首先理解一下多线程内存模型

​	Java 内存模型来屏蔽掉各种硬件和操作系统的内存差异，达到跨平台的内存访问效果。JLS(Java语言规范)定义了一个统一的内存管理模型*JMM*(Java Memory Model)

　　Java内存模型规定了所有的变量都存储在主内存中，此处的主内存仅仅是虚拟机内存的一部分，而虚拟机内存也仅仅是计算机物理内存的一部分（为虚拟机进程分配的那一部分）。

　　Java内存模型分为主内存，和工作内存。主内存是所有的线程所共享的，工作内存是每个线程自己有一个，不是共享的。

　　每条线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝。线程对变量的所有操作（读取、赋值），都必须在工作内存中进行，而不能直接读写主内存中的变量。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者之间的交互关系如下图：

![](https://i.loli.net/2019/03/28/5c9c411736eee.jpg)

​	JLS定义了线程对主存的操作指令：lock，unlock，read，load，use，assign，store，write。这些行为是不可分解的原子操作，在使用上相互依赖，read-load从主内存复制变量到当前工作内存，use-assign执行代码改变共享变量值，store-write用工作内存数据刷新主存相关内容。

- lock：把主内存变量标识为一条线程独占，此时不允许其他线程对此变量进行读写。
- unlock：解锁一个主内存变量。

- read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

摘自<a href="<https://www.cnblogs.com/chihirotan/p/6486436.html>">链接</a>



那么volatile的作用就是，

- 当一个线程修改了一个变量后，volatile立刻将该修改值写入共享内存中，同时通知其他线程该变量的变化。
- 禁止进行指令重排序
- volatile不能保证原子性，但是可以保定一定的有序性，可以保证有效性

详细参考该<a href="<https://www.cnblogs.com/dolphin0520/p/3920373.html>">链接</a> 写的特别棒



## final变量

安全地访问一个共享域， 即这个域声明为`final `时

```java
final Map<String, Double〉accounts = new HashKap<>()；
```

​	其他线程会在构造函数完成构造之后才看到这个accounts 变量。
​	如果不使用final，就不能保证其他线程看到的是accounts 更新后的值，它们可能都只是看到null , 而不是新构造的HashMap。
​	当然，对这个映射表的操作并不是线程安全的。如果多个线程在读写这个映射表，仍然需要进行同步。

## 原子性

> 假设对共享变量除了赋值之外并不完成其他操作， 那么可以将这些共享变量声明为volatile()

因为`volatile()`不能保证原子性

- `java.util.concurrent.atomic` 包中有很多类使用了很高效的机器级指令（而不是使用锁） 来保证其他操作的原子性

  ep：`Atomiclnteger `类提供了方法`incrementAndGet `和`decrementAndGet`, 它们分别以原子方式将一个整数自增或自减

```java
public static AtomicLong nextNumber = new AtomicLongO ;
// In some thread...
long id = nextNumber .increinentAndGet():
```

- 如果想完成更复杂的更新，必须使用`compareAndSet`方法

  ep：假设希望跟踪不同线程观察的最大值

```java
do {
	oldValue = largest.get() ;
	newValue = Math , max(oldValue, observed) ;
} while (llargest.compareAndSet(oldValue, newValue)) ;
```

​	如果另一个线程也在更新`largest`，就可能阻止这个线程更新。这样一来，`compareAndSet`会返回false, 而不会设置新值。在这种情况下， 循环会更次尝试，读取更新后的值，并尝试修改。最终， 它会成功地用新值替换原来的值。这听上去有些麻烦， **不过`compareAndSet `方法会映射到一个处理器操作， 比使用锁速度更快**

- 类`Atomiclnteger、AtomicIntegerArray、AtomicIntegerFieldUpdater、AtomicLongArray、AtomicLongFieldUpdater、AtomicReference、AtomicReferenceArray 和AtomicReference-FieldUpdater `也提供了这些方法
- 上面是针对数量较少的操作，但是如果我们由大连线程访问相同的原子值。性能会下降因为乐观更新需要太多次重试，使用`LongAdder` 和`LongAccumulator `类来解决这个问题**。`LongAdder `包括多个变量（加数)，**
  **其总和为当前值。可以有多个线程更新不同的加数，线程个数增加时会自动提供新的加数。通常情况下， 只有当所有工作都完成之后才需要总和的值**， 对于这种情况，这种方法会很高效。详细的使用自行谷歌



## 死锁

> Java 编程语言中没有任何东西可以避免或打破这种死锁现象。必须仔细设计程序， 以确保不会出现死锁。

注意前面说的`signal`和`signalAll`方法区别导致死锁



## 线程局部变量

- 使用`ThreadLocal `辅助类为各个线程提供各自的实例
- 通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。而使用ThreadLocal创建的变量只能被当前线程访问，其他线程则无法访问和修改。
- 支持泛型

```java
T get()
//得到这个线程的当前值。如果是首次调用get, 会调用initialize 来得到这个值。
protected initialize()
//应覆盖这个方法来提供一个初始值。默认情况下，这个方法返回null。
void set(T t)
//为这个线程设置一个新值。
void remove()
//删除对应这个线程的值。
```





## 锁测试和超时

```java
boolean tryLock()
//尝试获得锁而没有发生阻塞；如果成功返回真。这个方法会抢夺可用的锁， 即使该锁有公平加锁策略， 即便其他线程已经等待很久也是如此。
boolean tryLock(long time, TimeUnit unit)
//尝试获得锁，阻塞时间不会超过给定的值；如果成功返回true。
void lockInterruptibly()
//获得锁， 但是会不确定地发生阻塞。如果线程被中断， 抛出一个InterruptedException异常。

boolean await( 1ong time , TimeUnit unit )
//进人该条件的等待集， 直到线程从等待集中移出或等待了指定的时间之后才解除阻塞。如果因为等待时间到了而返回就返回false , 否则返回true。
void awaitUninterruptibly( )
//进人该条件的等待集， 直到线程从等待集移出才解除阻塞。如果线程被中断， 该方法不会抛出InterruptedException 异常。
```

其实就是在锁对象里说的

## 读/写锁

​	如果很多线程从一个数据结构读取数据而很少线程修改其中数据的话，使用`ReentrantReadWriteLock`类。此时允许读者线程共享访问，写线程依旧是互斥的

下面是使用读/ 写锁的必要步骤：
1 ) 构造一个ReentrantReadWriteLock 对象：

```java
private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
```

2 ) 抽取读锁和写锁：

```java
private Lock readLock = rwl . readLock() ;
private Lock writeLock = rwl .writeLock();
```

3 ) 对所有的获取方法加读锁：

```java
public double getTotalBalanceO
{
	readLock.lock()；
	try { . . . }
	finally { readLock.unlock() ; }
}
```

4 ) 对所有的修改方法加写锁：

```java
public void transfer(. . .)
{
	writeLock.lockO;
	try { . . . }
	finally { writeLock.unlock(); }
}
```



```java
Lock readLock()
//得到一个可以被多个读操作共用的读锁， 但会排斥所有写操作。
Lock writeLock()
//得到一个写锁， 排斥所有其他的读操作和写操作。
```





# 阻塞队列

> 使用队列，可以安全地从一个线程向另一个线程传递数据

当我们想从向队列添加元素而队列已满， 或是想从队列移出元素而队列为空的时候， 阻塞队列（ blocking queue ) 导致线程阻塞

![](https://i.loli.net/2019/03/28/5c9c98941ddf1.jpg)

java.util.concurrent 包中有几个阻塞队列变种

![](https://i.loli.net/2019/03/28/5c9c9b7ad9a6a.jpg)

`LinkedBlockingQueue`的容量是没有上边界的，但是，也可以选择指定最大容量。

`LinkedBlockingDeque `是一个双端的版本。

`ArrayBlockingQueue` 在构造时需要**指定容量**，并且有一个可选的参数来指定是否需要公平性。若设置了公平参数， 则那么等待了最长时间的线程会优先得到处理。通常，公平性会降低性能， 只有在确实非常需要时才使用它。
`PriorityBlockingQueue `是一个带优先级的队列， 而不是先进先出队列。元素按照它们的优先级顺序被移出。该队列是没有容量上限， 但是，**如果队列是空的， 取元素的操作会阻塞。**

`DelayQueue`包含实现Delayed 接口的对象：

```java
interface Delayed extends Comparable<Delayed>
{
	long getDel ay(TimeUnit unit);
}
```

​	getDelay 方法返回对象的残留延迟。负值表示延迟已经结束。元素只有在延迟用完的情况下才能从DelayQueue 移除。还必须实现compareTo 方法。DelayQueue 使用该方法对元素进行排序

`TranSferQueUe `接口，允许生产者线程等待， 直到消费者准备就绪可以接收一个元素



```java
java.util.concurrent.ArrayBlockingQueue<E> 5.0
ArrayBlockingQueue( int capacity)
ArrayBlockingQueue( int capacity, boolean fair )
//构造一个带有指定的容量和公平性设置的阻塞队列。该队列用循环数组实现。

java.util.concurrent.LinkedBlockingDeque<E> 6
LinkedBlockingQueue( )
LinkedBlockingDeque( )
//构造一个无上限的阻塞队列或双向队列，用链表实现。
LinkedBlockingQueue( int capacity )
LinkedBlockingDeque( int capacity )
//根据指定容量构建一个有限的阻塞队列或双向队列，用链表实现。
    
java.util.concurrent.DelayQueue<E extends Delayed〉5.0
DelayQueue( )
//构造一个包含Delayed 元素的无界的阻塞时间有限的阻塞队列。只有那些延迟已经超过时间的元素可以从队列中移出。
    
java.util.concurrent.Delayed 5.0
long getDelay(Timellnit unit )
//得到该对象的延迟，用给定的时间单位进行度量。
    
java.util.concurrent.PriorityBlockingQueue<E> 5.0
PriorityBlockingQueue( )
PriorityBlockingQueiie( int initialCapaci ty)
PriorityBlockingQueue( int initialCapacity, Comparator ? super E> comparator )
/*构造一个无边界阻塞优先队列，用堆实现。
参数： initialCapacity 优先队列的初始容量。默认值是11。comparator 用来对元素进行比较的比较器， 如果没有指定， 则元素必须实现Comparable 接口。*/
    
    
```



# 线程安全的集合

​	一个线程可能要开始向表中插入一个新元素。假定在调整散列表各个桶之间的链接关系的过程中， 被剥夺了控制权。如果另一个线程也开始遍历同一个链表， 可能使用无效的链接并造成混乱， 会抛出异常或者陷人死循环。

​	阻塞队列就是线程安全的集合

## 高效的映射、集和队列

​	`java.util.concurrent `包提供了映射、有序集和队列的高效实现：`ConcurrentHashMap、
ConcurrentSkipListMap 、 ConcurrentSkipListSet 和ConcurrentLinkedQueue`

```java
ConcurrentLinkedQueue< E>()
//构造一个可以被多线程安全访问的无边界非阻塞的队列。
    
ConcurrentSkipListSet<E>()
ConcurrentSkipListSet<E>(Comparator<? super E> comp)
//构造一个可以被多线程安全访问的有序集。第一个构造器要求元素实现Comparable接口。
    
ConcurrentHashMapCK, V>()
ConcurrentHashMap<K, V>(1nt 1n1t1 alCapacity)
ConcurrentHashMapCK, V>(int initialCapacity, float 1oadFactor, 1nt concurrencyLevel)
/*构造一个可以被多线程安全访问的散列映射表。
参数: initialCapacity    集合的初始容量。默认值为16。
	 loadFactor         控制调整： 如果每一个桶的平均负载超过这个因子，表的大小会被重新调整。默认值为0.75。
	 concurrencyLevel   并发写者线程的估计数目。*/

ConcurrentSkipListMap<K, V>()
ConcurrentSkipListSet<K, V>(Comparator<? super K> comp)
//构造一个可以被多线程安全访问的有序的映像表。第一个构造器要求键实现Comparable 接口
    
```

和一般的集合的size方法不同，size方法不会再常量时间内操作，确定集合的当前大小需要遍历。容易出错

## 映射条目的原子更新

多线程统计总数的问题，可能某个线程在统计某个词时被中断，其他线程修改后续内容了，导致统计结果出错。

- 使用`ConcurrentHashMap`实现原子更新。`ConcurrentHashMap<String，AtomicLong>`或者`ConcurrentHashMap<String，LongAdder>`

```java
map.putlfAbsent(word, new LongAdderO)；
map.get(word) .increment();
```

- 调用`compute` 方法时可以提供一个键和一个计算新值的函数。这个函数接收键和相关联的值（如果没有值，则为null), 它会计算新值,离子

```java
map.compute(word, (k, v) -> v = null ? 1: v + 1);
```

- LongAdder 构造器只在确实需要一个新的计数器时才会调用。首次增加一个键时通常需要做些特殊的处理。这个方法有一个参数表示键不存在时使用的初始值。否则， 就会调用你提供的函数来结合原值与初始值

```java
map.merge(word, 1L, (existi ngValue, newValue) -> existingValue + newValue) ;
```



## 对并发散列映射的批操作

不懂

## 并发集视图

> 使用`ConcurrentHashMap`实现，并没有`ConcurrentHashSet`类。这会得到一个映射而不是集， 而且不能应用Set 接口的操作。静态`newKeySet `方法会生成一个`Set<K>`, 这实际上是`ConcurrentHashMap<K, Boolean〉`的一个包装器。（所有映射值都为`Boolean.TRUE`, 不过因为只是要把它用作一个集， 所以并不关心具体的值。）

```java
Set<String> words = ConcurrentHashMap.<String>newKeySet() ;
```



> 如果原来有一个映射，`keySet `方法可以生成这个映射的键集。这个集是可变的。如果删除这个集的元素，这个键（以及相应的值）会从映射中删除

其实就是用Map表示Set，只用index，不用value(映射值为true)

## 写数组的拷贝

> CopyOnWriteArrayList 和CopyOnWriteArraySet 是线程安全的集合， 其中所有的修改线
> 程对底层数组进行复制。如果在集合上进行迭代的线程数超过修改线程数， 这样的安排是
> 很有用的。当构建一个迭代器的时候， 它包含一个对当前数组的引用。如果数组后来被修改
> 了，迭代器仍然引用旧数组， 但是，集合的数组已经被替换了。因而，旧的迭代器拥有一致
> 的（可能过时的）视图，访问它无须任何同步开销。

其实就是多线程的内存模型

## 较早的线程安全集合

Vector 和Hashtable 类就提供了线程安全的动态数组和散列表的实现。现在这些类被弃用了， 取而代之的是AnayList 和HashMap 类。这些类不是线程安全的，而集合库中提供了不同的机制。任何集合类都可以通过使用同步包装器（ synchronization wrapper) 变成线程安全的：

```java
List<E> synchArrayList = Col lections ,synchronizedList (new ArrayList<E>()) ;
Map<K , V> synchHashMap = Col1ections.synchronizedMap(new HashMap<K , V>())；
```

<span style="color:red">最好使用java.Util.COnciirrent 包中定义的集合</span>

## 并行数组算法

静态`Arrays.parallelSort` 方法可以对一个基本类型值或对象的数组排序

- 对对象排序时，可以提供一个`Comparator`

```java
Arrays,parallelSort(words, Comparator.comparing(String::length));
```

- 对于所有方法都可以提供一个范围的边界

```java
values,parallelSort(values,length / 2, values,length) ; // Sort the upper half
```





# Callable和Future

`Runnable `封装一个异步运行的任务，可以把它想象成为一个没有参数和返回值的异步方法。`Callable `与`Runnable `类似， 但是有返回值。`Callable `接口是一个参数化的类型， 只有一个方法`call`

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

Future 保存异步计算的结果。可以启动一个计算，将Future 对象交给某个线程，然后忘掉它。Future 对象的所有者在结果计算好之后就可以获得它。其实就是在其它线程完成计算后，他才会启动计算，并将计算结果返回。

```java
public interface Future<V> {
	
    boolean cancel(boolean mayInterruptIfRunning);//撤销一个正在执行的任务，中断该线程（打上中断标记）如果计算还没有开始，它被取消且不再开始。如果计算处于运行之中，那么如果maylnterrupt 参数为true, 它就被中断。

    boolean isCancelled();//判断是否撤销

    boolean isDone();//表示任务是否已经完成	如果任务结束，无论是正常结束、中途取消或发生异常， 都返回true。

    V get() throws InterruptedException, ExecutionException;//获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;//获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。
}
```



`FutureTask` 类图

![](https://i.loli.net/2019/03/29/5c9e220f7be8a.jpg)



计算匹配的文件数目

```java
package future;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class FutureTest {
    public static void main(String[] args) {
        try (Scanner in=new Scanner(System.in)) {
            System.out.print("Enter base directory (e.g. /usr/local/jdk1.6.0/src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyWord = in.nextLine();

            MatchCounter counter=new MatchCounter(new File(directory),keyWord);
            /*
            构造一个既是Future又是Runnable对象
             */
            FutureTask<Integer> task=new FutureTask<>(counter);
            Thread t=new Thread(task);
            t.start();
            try {
                System.out.println(task.get()+" matching files");
            }catch (ExecutionException e){
                e.printStackTrace();
            }
        }catch (InterruptedException e){

        }
    }


}

/**
 * 计算包含keyword的数量
 */
class MatchCounter implements Callable<Integer>{
    private File directory;
    private String keyWord;

    public MatchCounter(File directory, String keyWord) {
        this.directory = directory;
        this.keyWord = keyWord;
    }

    /**
     * 运行一个将产生结果的任务
     * @return
     * @throws Exception
     */
    @Override
    public Integer call() throws Exception {
        int count=0;
        try{
            File[] files = directory.listFiles();
            List<Future<Integer>> results=new ArrayList<>();
            for (File file :
                    files) {
                if (file.isDirectory()) {
                    MatchCounter counter=new MatchCounter(file , keyWord);
                    FutureTask<Integer> task=new FutureTask<>(counter);
                    results.add(task);
                    Thread t = new Thread(task);
                    t.start();
                }else {
                    if (search(file))
                        count++;
                }
                for (Future<Integer> result :
                        results) {
                    try {
                        count += result.get();
                    }catch (ExecutionException e){
                        e.printStackTrace();
                    }
                }
            }
        }catch (InterruptedException e){

        }
        return count;
    }

    public boolean search(File file){
        try {
            try (Scanner in=new Scanner(file,"UTF-8")){
                boolean found=false;
                while (!found&&in.hasNextLine()){
                    String line=in.nextLine();
                    if(line.contains(keyWord))
                        found=true;
                }
                return found;
            }
        }catch (IOException e){
            return false;
        }
    }
}
```



# 执行器

执行器（ Executor) 类有许多**静态工厂方法**用来构建线程池

![](https://i.loli.net/2019/03/30/5c9ee8df076f1.png)

![](https://i.loli.net/2019/03/30/5c9edc3919550.png)



## 线程池

`newCachedThreadPool`方法构建了一个线程池， 对于每个任务， 如果有空闲线程可用， 立即让它执行任务， 如果没有可用的空闲线程， 则创建一个新线程。

`newFixedThreadPool` 方法构建一个具有固定大小的线程池。如果提交的任务数多于空闲的线程数， 那么把得不到服务的任务放置到队列中。当其他任务完成以后再运行它们。

`newSingleThreadExecutor` 是一个退化了的大小为1 的线程池： 由一个线程执行提交的任务， 一个接着一个。

这3 个方法返回实现了`ExecutorService `接口的`ThreadPoolExecutor `类的对象。



使用以下方法将一个`Runnable`对象或`Callable` 对象提交给`ExecutorService`:

```java
    <T> Future<T> submit(Callable<T> task);
	//提交一个Callable, 并且返回的Future 对象将在计算结果准备好的时候得到它。

    <T> Future<T> submit(Runnable task, T result);
	//提交一个Runnable， 并且Future 的get 方法在完成的时候返回指定的result 对象。

    Future<?> submit(Runnable task);
	//调用isDone、cancel 或isCancelled。但是， get 方法在完成的时候只是简单地返回null。
```

使用过程：

当用完一个线程池的时候， 调用`shutdown`。该方法启动该池的关闭序列。被关闭的执行器不再接受新的任务。当所有任务都完成以后，线程池中的线程死亡。另一种方法是调用`shutdownNow`。该池取消尚未开始的所有任务并试图中断正在运行的线程。

**流程**：

1. 调用`Executors `类中静态的方法`newCachedThreadPool `或`newFixedThreadPool`。
2. 调用`submit `提交`Runnable` 或`Callable `对象。
3. 调用involve等方法执行任务
4. 如果想要取消一个任务， 或如果提交`Callable` 对象， 那就要保存好返回的`Future`对象。
5. 当不再提交任何任务时，调用`shutdown`。



```java
package ThreadPoolTest;


import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.concurrent.*;

public class ThreadPoolTest {
    public static void main(String[] args) {
        try (Scanner in=new Scanner(System.in)) {
            System.out.print("Enter base directory (e.g. /usr/local/jdk1.6.0/src): ");
            String directory = in.nextLine();
            System.out.print("Enter keyword (e.g. volatile): ");
            String keyWord = in.nextLine();
			//创建线程池对象
            ExecutorService pool= Executors.newCachedThreadPool();//线程池对象
			//操作流
            MatchCounter counter=new MatchCounter(new File(directory),keyWord,pool);
            Future<Integer> result=pool.submit(counter);

            try {
                System.out.println(result.get()+" matching files");
            }catch (ExecutionException e){
                e.printStackTrace();
            }catch (InterruptedException e){

            }
            pool.shutdown();

            int largestPoolSize = ((ThreadPoolExecutor) pool).getLargestPoolSize();
            System.out.println("largest pool size="+ largestPoolSize);
        }

    }
}

//实现Callable接口 call方法有返回值
class MatchCounter implements Callable<Integer> {

    private File directory;
    private String keyWord;
    private ExecutorService pool;
    private int count;

    public MatchCounter(File directory, String keyWord, ExecutorService pool) {
        this.directory = directory;
        this.keyWord = keyWord;
        this.pool = pool;
    }

    @Override
    public Integer call() throws Exception {
        count=0;
        try {
            File[] files=directory.listFiles();
            List<Future<Integer>> results=new ArrayList<>();
            for (File file :
                    files) {
                if(file.isDirectory()){
                    MatchCounter counter=new MatchCounter(file,keyWord,pool);
                    Future<Integer> result=pool.submit(counter);
                    results.add(result);
                }else {
                    if(search(file))//查询的内容加加
                        count++;
                }

                for (Future<Integer> result:results){//总共文件内的加加
                    try{
                        count+=result.get();
                    }catch (ExecutionException e){
                        e.printStackTrace();
                    }
                }
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        return count;
    }
    public boolean search(File file){
        try{
            try(Scanner in=new Scanner(file,"UTF-8")){
                boolean found=false;
                while (!found&&in.hasNextLine()){
                    String line=in.nextLine();
                    if (line.contains(keyWord))
                        found=true;
                }
                return found;
            }
        }catch (IOException e){
            return false;
        }
    }


}



```

## 预定执行

​	`ScheduledExecutorService` 接口具有为预定执行（ `Scheduled Execution` ) 或重复执行任务而设计的方法。它是一种允许使用线程池机制的`java.util.Timer `的泛化。`Executors` 类的
`newScheduledThreadPool `和`newSingleThreadScheduledExecutor `方法将返回实现了`ScheduledExecutorService` 接口的对象。

​	可以预定`Runnable` 或`Callable `在初始的延迟之后只运行一次。也可以预定一个`Runnable`对象周期性地运行

## 控制任务组

```java
T invokeAny( Co11ection<Cal 1able<T>> tasks )
T invokeAny(Col 1ection<Cal 1able<T>> tasks , long timeout , TimeUnit unit )
//执行给定的任务， 返回其中一个任务的结果。第二个方法若发生超时， 抛出一个Timeout Exception 异常。
    
List<Future<T>> invokeAll (Col 1ection<Cal 1 able<T> > tasks )
List<Future<T>> invokeAll( Col 1 ection<Cal 1 abl e<T > > tasks , long timeout , TimeUnit unit )
//执行给定的任务， 返回所有任务的结果。第二个方法若发生超时， 拋出一个TimecmtException 异常。
    
ExecutorCompletionService(Executor e )
//构建一个执行器完成服务来收集给定执行器的结果。
Future< V> submit(Cal 1 able< V > task )
Future< V > submit( Runnable task , V result )
//提交一个任务给底层的执行器。
Future< V > take( )
//移除下一个已完成的结果， 如果没有任何已完成的结果可用则阻塞。
Future< V > poll( )
Future< V > poll( 1 ong time , TimeUnit unit )
//移除下一个已完成的结果， 如果没有任何已完成结果可用则返回null。第二个方法将等待给定的时间。
```



## Fork-Join框架

将一个大任务分为多个任务，分别执行。提高效率

## 可完成Future

不懂

# 同步器

管理线程

![](https://i.loli.net/2019/03/30/5c9ef0c43e2ca.png)

## 信号量

## 倒计时门栓`CountDownLatch`

一个倒计时门栓（ CountDownLatch) 让一个线程集等待直到计数变为0。倒计时门栓是一次性的。一旦计数为0, 就不能再重用了。

就是赛跑时，预备起跑。口令到了，大家才能跑

## 障栅`CyclicBarrier`

​	考虑大量线程运行在一次计算的不同部分的情形。当所有部分都准备好时，需要把结果组合在一起。当一个线程完成了它的那部分任务后， 我们让它运行到障栅处。一旦所有的线程都到达了这个障栅，障栅就撤销， 线程就可以继续运行。

## 交换器`Exchanger`

​	当两个线程在同一个数据缓冲区的两个实例上工作的时候， 就可以使用交换器( Exchanger) 典型的情况是， 一个线程向缓冲区填人数据， 另一个线程消耗这些数据。当它们都完成以后，相互交换缓冲区。

## 同步队列`SynchronousQueue`

​	同步队列是一种**将生产者与消费者线程配对的机制**。当一个线程调用SynchronousQueue的put 方法时， 它会阻塞直到另一个线程调用take 方法为止， 反之亦然。与Exchanger 的情况不同， **数据仅仅沿一个方向传递，从生产者到消费者。**
​	即使SynchronousQueue 类实现了BlockingQueue 接口， 概念上讲， 它依然不是一个队列。它没有包含任何元素，它的size 方法总是返回0。

