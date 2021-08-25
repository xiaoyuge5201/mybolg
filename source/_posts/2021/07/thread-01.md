---
title: Java守护线程和非守护线程
translate_title: java-daemon-thread-and-nondaemon
tags: java
categories:
  - java
  - Thread
comments: false
date: 2021-08-15 14:14:56
---
用户线程：我们平常创建的普通线程。

守护线程：用来服务于用户线程；不需要上层逻辑介入

java线程分为守护线程和非守护线程，当java jvm检测主线程或其他子线程执行完之后，守护线程也会马上停止执行，我们可以使用Thread.setDaemon(ture或false)来设置一个线程是守护线程还是非守护线程，默认为false，可以通过Thread.isDaemon()方法查询该线程是否是守护线程

守护线程是所有的用户线程结束生命周期，守护线程才会结束生命周期，只要有一个用户线程存在，那么守护线程就不会结束，例如Java中的垃圾 回收器就是一个守护线程，只有应用程序中所有的线程结束，它才会结束。
```java
public class DaemonThread {
    public static void main(String[] args) {
        Thread thread = new Thread(DaemonThread::print);
        thread.setDaemon(true);
        thread.start();
        System.out.println("主线程main 结束");
    }

    public static void print() {
        int counter = 1;
        //写一个死循环的方法来测试
        while (true) {
            try {
                System.out.println("Counter:" + counter++);
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
输出：
```text
主线程main 结束
Counter:1
```
如果我们将daemon设置为非守护线程，代码如下:
```java
thread.setDaemon(false);
```
这个时候就不会退出while(true)循环了，会一直执行下去，结果如下：
```text
主线程main 结束
Counter:1
Counter:2
Counter:3
Counter:4
Counter:5
....
```

**总结：守护线程是为用户线程服务的，当用户线程全部结束，守护线程会自动结束。**

**注意事项：**
1. thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。
2. 在Daemon线程中产生的新线程也是Daemon的。
3. 守护线程不能用于去访问固有资源，比如读写操作或者计算逻辑。因为它会在任何时候甚至在一个操作的中间发生中断。
4. Java自带的多线程框架，比如ExecutorService，会将守护线程转换为用户线程，所以如果要使用后台线程就不能用Java的线程池。

**意义以及应用场景:**

当主线程结束时，结束其余的子线程（守护线程）自动关闭，就免去了还要继续关闭子线程的麻烦。如：Java垃圾回收线程就是一个典型的守护线程；内存资源或者线程的管理，但是非守护线程也可以。