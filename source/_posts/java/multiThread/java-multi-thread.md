---
title: java 多线程
date: 2021-05-24 10:31:01
categories:
  - java
tags:
  - java
  - 多线程
---

# 概述

 提到线程不得不提进程。因为线程是进程的一个执行单元。下面对线程和进程分别进行介绍。

## 进程

 进程是当前操作系统执行的任务，是并发执行的程序在执行过程中分配和管理资源的基本单位，是一个动态概念，竟争计算机系统资源的基本单位。一般而言，现在的操作系统都是多进程的。

  **进程的执行过程是线状的**， 尽管中间会发生中断或暂停，但该进程所拥有的资源只为该线状执行过程服务。一旦发生进程上下文切换，这些资源都是要被保护起来的。

## 线程

 线程，是进程的一部分，一个没有线程的进程可以被看作是单线程的。即：每个进程中至少包含一个线程。

线程本身是在CPU上执行的，CPU的每一个核在同一时刻只能执行一个线程，但CPU在底层会对线程进行快速的轮询切换。



# JAVA中如何定义线程

## 继承Thread

通过继承Thread，重写run方法，将要执行的逻辑放在run方法中，然后创建线程对象调用start方法来开启线程。示例如下：

```java
public class ThreadDemo {

    public static void main(String[] args) {

        TDemo t1 = new TDemo("A");
        // 启动线程
        // start方法中会给线程做很多的配置
        // 配置完成之后会自动调用run方法执行指定的任务
        t1.start();
        // t1.run();
        TDemo t2 = new TDemo("B");
        t2.start();
        // t2.run();

    }

}

class TDemo extends Thread {

    private String name;

    public TDemo(String name) {
        this.name = name;
    }

    // 打印0-9
    // 线程要执行的任务就是放在这个方法中
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(name + ":" + i);
        }
    }
}
```

## 继承Runnable

实现Runnable，重写run方法，然后利用Runnable对象来构建Thread对象，调用start方法来启动线程。示例如下：

```java
public class RunnableDemo {

    public static void main(String[] args) {

        RDemo r = new RDemo();
        // 包装 - 装饰设计模式
        Thread t = new Thread(r);
        t.start();

    }
}
class RDemo implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}
```

## 实现Callable<T>

实现Callable<T>，重写call方法，通过线程池定义线程。示例如下：

```java
public class CallableDemo {

    public static void main(String[] args) throws InterruptedException, ExecutionException {

        CDemo c = new CDemo();
        // 执行器服务 执行器助手
        ExecutorService es = Executors.newCachedThreadPool();
        Future<String> f = es.submit(c);
        System.out.println(f.get());
        es.shutdown();
    }

}

// 泛型表示要的结果类型
class CDemo implements Callable<String> {

    @Override
    public String call() throws Exception {
        return "SUCCESS";
    }

}
```

# 多线程的并发安全问题

线程之间是相互抢占执行，而且抢占是发生在线程执行的每一步；当线程重新抢回执行权之后，会沿着上次被抢占位置继续向下执行，而不是从头开始执行。

由于线程的抢占而导致出现了不合理的数据的现象：多线程的并发安全问题。

# 线程中的锁机制

## 概述

为了解决线程并发问题，引入了synchronized代码块，亦即同步代码块。同步代码块需要一个锁对象。

## 锁对象及其特点

锁对象要求被当前的所有线程都认识。共享资源，方法去中的资源和this都可以作为锁对象。

当使用this作为锁对象的时候，要求利用同一个Runnable对象来构建不同的Thread对象。

示例如下：利用多线程实现卖票机制

```java
import java.io.FileInputStream;
import java.util.Properties;

// 利用多线程机制模拟卖票场景
public class SellTicketDemo {
    public static void main(String[] args) throws Exception {
        // 利用properties做到改动数量但是不用改动代码的效果
        Properties prop = new Properties();
        prop.load(new FileInputStream("ticket.properties"));
        int count = Integer.parseInt(prop.getProperty("count"));
        // 利用ticket对象做到所有的线程共享一个对象
        Ticket t = new Ticket();
        t.setCount(count);
        // 表示四个售票员在分别卖票
        Thread t1 = new Thread(new Seller(t), "A");
        Thread t2 = new Thread(new Seller(t), "B");
        Thread t3 = new Thread(new Seller(t), "C");
        Thread t4 = new Thread(new Seller(t), "D");

        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}

// 定义了线程类表示售票员
class Seller implements Runnable {

    private Ticket t;
    public Seller(Ticket t) {
        this.t = t;
    }

    @Override
    public void run() {
        // 锁对象 --- 需要指定一个对象作为锁来使用
        while (true) {
            // 由于所有的Seller线程都在卖票t，所以t是被所有线程都认识的
            // synchronized (t) {
            // 由于所有的Seller线程都是Seller类产生的，所以Seller类也是被所有线程都认识的
            // synchronized (Seller.class) {
            // synchronized (Thread.class) {
            synchronized ("abc") {
                if (t.getCount() <= 0)
                    break;
                try {
                    // 让当前线程陷入休眠
                    // 时间单位是毫秒
                    // 不改变线程的执行结果
                    // 只会把线程的执行时间拉长
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 票数减少1张
                t.setCount(t.getCount() - 1);
                // currentThread()获取当前在执行的线程
                // 获取线程的名字
                String name = Thread.currentThread().getName();
                System.out.println(name + "卖了一张票，剩余" + t.getCount());
            }
        }
    }
}

class Ticket {
    private int count;

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}
```

## 线程的同步和异步

同步：在同一时刻内资源/逻辑只被一个线程占用/执行。

异步：在同一时刻内资源/逻辑可以被多个线程抢占使用。

## 多线程死锁

由于多个线程之间的锁形成了嵌套而导致代码无法继续执行，这种现象称之为死锁。

我们只能尽量避免出现死锁，在实际开发中，会做死锁的检验；如果真的出现了死锁，会根据线程的优先级打破其中一个或者多个锁。

死锁的示例如下：

```java
public class DeadLockDemo {

    static Printer p = new Printer();
    static Scan s = new Scan();

    public static void main(String[] args) {

        // 第一个员工
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                synchronized (p) {
                    p.print();
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (s) {
                        s.scan();
                    }
                }
            }
        };
        new Thread(r1).start();
        // 第二个员工
        Runnable r2 = new Runnable() {
            @Override
            public void run() {
                synchronized (s) {
                    s.scan();
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (p) {
                        p.print();
                    }
                }
            }
        };
        new Thread(r2).start();
    }
}

// 代表打印机的类
class Printer {
    public void print() {
        System.out.println("打印机在吱呦吱呦的打印~~~");
    }
}

// 代表扫描仪的类
class Scan {
    public void scan() {
        System.out.println("扫描仪在哼哧哼哧的扫描~~~");
    }
}
```

# 线程的优先级

Java中将线程的优先级分为1-10共十个等级。

理论上，数字越大优先级越高，那么该线程能抢到资源的概率也就越大；但实际上，相邻的两个优先级之间的差别非常不明显；如果想要相对明显一点，至少要相差5个优先级。

线程优先级示例如下:

```java
public class PriorityDemo {

    public static void main(String[] args) {

        Thread t1 = new Thread(new PDemo(), "A");
        Thread t2 = new Thread(new PDemo(), "B");

        // 在默认情况下，线程的优先级都是5
        // System.out.println(t1.getPriority());
        // System.out.println(t2.getPriority());

        // 设置优先级
        t1.setPriority(1);
        t2.setPriority(10);
        t1.start();
        t2.start();
    }
}

class PDemo implements Runnable {
    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        for (int i = 0; i < 10; i++) {
            System.out.println(name + ":" + i);
        }
    }
}
```

# 线程的等待唤醒机制

利用标记为以及wait、notify、notifyAll方法来调用线程之间的执行顺序；

wait、notify、notifyAll和锁有关，用那个对象作为锁对象使用，那么就用该锁对象来调用wait、notify。

等待和唤醒示例如下： 

```java
public class WaitNotifyAllDemo {
    public static void main(String[] args) {
        Product p = new Product();

        new Thread(new Supplier2(p)).start();
        new Thread(new Supplier2(p)).start();
        new Thread(new Consumer2(p)).start();
        new Thread(new Consumer2(p)).start();
    }
}

// 生产者
class Supplier2 implements Runnable {
    private Product p;

    public Supplier2(Product p) {
        this.p = p;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (p) {
              //因为线程被抢断后，会沿着停止出继续执行，因为用while循环强制对其进行判断，满足条件时才能执行
              //不满足条件就让其等待
                while (p.flag == false){
                    try {
                        // 让当前线程陷入等待
                        p.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 计算本次生产的商品数量
                int count = (int) (Math.random() * 1000);
                p.setCount(count);
                System.out.println("生产者生产了" + count + "件商品~~~");
                p.flag = false;
                //当多个线程执行时，要唤醒所有的线程，否则可能连续唤起一个线程，导致程序执行混乱
                p.notifyAll();
            }
        }
    }
}

// 消费者
class Consumer2 implements Runnable {

    private Product p;

    public Consumer2(Product p) {
        this.p = p;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (p) {
                while (p.flag == true){
                    try {
                        p.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                int count = p.getCount();
                p.setCount(0);
                System.out.println("消费者消费了" + count + "件商品~~~");
                p.flag = true;
                // 唤醒在等待的线程
                p.notifyAll();
            }
        }
    }
}
```

# 线程的状态

线程从创建到开始消亡一般会经历如下几种状态:

![images](/images/java-multy-thread.png)

# 守护线程

## 概述

守护其他的线程被称为守护线程，只要被守护的线程结束，那么守护线程就会随之结束。

## 守护线程的特点

- 一个线程要么是守护线程，要么是被守护线程
- 守护线程可以守护其他的守护线程
- 在Java中，最常见的一个守护线程是GC

守护线程的示例如下：

```java
public class DaemonDemo {

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(new Monster(), "小怪1号");
        Thread t2 = new Thread(new Monster(), "小怪2号");
        Thread t3 = new Thread(new Monster(), "小怪3号");
        Thread t4 = new Thread(new Monster(), "小怪4号");

        // 设置为守护线程
        t1.setDaemon(true);
        t2.setDaemon(true);
        t3.setDaemon(true);
        t4.setDaemon(true);

        t1.start();
        t2.start();
        t3.start();
        t4.start();

        for (int i = 10; i > 0; i--) {
            System.out.println("Boss掉了一滴血，剩余" + i);
            Thread.sleep(50);
        }
    }

}
//守护boss的小怪线程
class Monster implements Runnable {

    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        for (int i = 1000; i > 0; i--) {
            System.out.println(name + "掉了一滴血，剩余" + i);
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

总结：sleep和wait的区别

sleep：在使用的时候需要指定休眠时间，单位是毫秒，到点自然醒。在无锁状态下，会释放CPU；在有锁状态下，不释放CPU。sleep方法是一个静态方法，被设计在了Thread类上。

wait：可以指定等待时间也可以不指定。如果不指定等待时间则需要被唤醒。wait必须结合锁使用，当线程在wait的时候会释放锁。wait方法设计在了Object类上。

# 线程产生和结束的场景

## 线程产生的场景

- 系统自启动：开机默认启动的程序；
- 用户请求：QQ好友聊天；
- 线程之间的启动：App软件之间带有的插件。

## 线程结束的场景

- 寿终正寝：线程自然结束
- 他杀：被其他线程kill 
- 意外：线程因为报错崩溃而退出 

# JAVA虚拟机方法区和线程的关系

类是存储在方法区中的，方法区是被所有的线程共享的空间。

每一个线程独有一个栈内存。



原文

https://www.cnblogs.com/chhyan-dream/p/10786043.html
