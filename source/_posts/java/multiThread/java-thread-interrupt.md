---
title: java thread interrupt 方法概述
date: 2021-06-08 20:11:10
categories:
  - java
tags:
  - java
  - 多线程
---

### 中断
中断（Interrupt）一个线程意味着在该线程完成任务之前停止其正在进行的一切，有效地中止其当前的操作。线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序。虽然初次看来它可能显得简单，但是，你必须进行一些预警以实现期望的结果


在java中，想要安全让一个线程停下来：
(1)采用退出标志，使得run方法执行完之后线程自然终止。
(2)使用中断机制。

#### 退出标志位

```java
public class test1 {

    public static volatile boolean exit =false;  //退出标志
    
    public static void main(String[] args) {
        new Thread() {
            public void run() {
                System.out.println("线程启动了");
                while (!exit) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("线程结束了");
            }
        }.start();
        
        try {
            Thread.sleep(1000 * 5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        exit = true;//5秒后更改退出标志的值,没有这段代码，线程就一直不能停止
    }
}
```

#### 中断机制

代码详见：https://github.com/idream68/java-demo/blob/master/src/main/java/com/demo/java/multi_thread/ThreadStop.java

