---
title: 锁优化
tags: lock
comments: false
translate_title: lock-optimization
date: 2021-07-23 14:04:02
---
 

## 1. 优化思路以及方法

- 减少锁持有时间
- 减小锁粒度
- 锁分离
- 锁粗化
- 锁消除

### 1.1 减少锁持有时间

```java
public synchronized void syncMethod(){
    othercode1();
    mutextMethod();
    othercode2();
}
```

像上述代码，在进入方法前就要得到锁，其他线程就要在外面等待。

分析：锁里面的资源在同一时间只允许一个线程执行，我们不仅要减少其他线程等待的时间，也要尽力减少线程在锁里面的执行时间，所以，尽量只有在有线程安全要求的程序代码上加锁。

```java
public void syncMethod(){
    othercode1();
    synchronized(this){
        metextMethod();
    }
    othercode2();
}
```

### 1.2 减小锁粒度

将大对象（这个对象可能会被很多线程访问）拆成小对象，大大增加并行度。

降低锁竞争，那么偏向锁、轻量级锁成功率才会提高。

最最典型的减小锁粒度的案例就是ConcurrentHashMap。在HashMap的基础上进行优化，使用了cas与synchronized来确保安全性，在保证安全性的基础上为了充分利用线程资源，更是巧妙的设计了多线程同扩容的模式。



### 1.3 锁分离

最常见的锁分离就是读写锁ReadWriteLock，根据功能进行分离成读锁和写锁。这样读读不互斥，读写互斥，写写互斥。既保证了线程安全，又提高了性能。

分析：读写分离这种思想可以延伸到我们其他的设计中，只要操作上互不影响，那锁就可以进行分离，比如：LinkedBlockingQueue

<img src="..\images/read_writer_Lock1.png" alt="read_writer_Lock1" style="zoom: 67%;" />

从头部获取数据，从尾部放入数据，使用两把锁。



### 1.4 锁粗化

通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短，即在使用完公共资源后，应该立即释放锁，只有这样，等待在这个锁上的其他线程才能尽早的获取资源执行任务；但是凡事都有一个度，如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化。

```java
public void demoMethod(){
    synchronized{
    	//dow sth.
	}
    //....做其他不需要同步的工作，但能很快执行完毕
    synchronized{
        //do sth.
    }
}
```

这种情况，根据锁粗化的思想，应该合并：

```java
public void demoMethod(){
    //整合成一次锁请求,前提时中间哪些不需要同步的工作很快就执行完成
    synchronized(lock){
        //do sth.
        //....做其他不需要同步的工作，但能很快执行完毕
    }
}
```

再举一个极端的例子：

```java
for(int i =0; i < circle; i++){
    synchronized(lock){
        //.....
    }
}
```

在一个循环内不同得获得锁。虽然JDK内部会对这个代码做些优化，但是还不如直接写成：

```java
synchronized(lock){
    for(int i =0; i < circle; i++){
    }
}
```

当然如果有需求说，这样的循环太久，需要给其他线程不要等待太久，那只能写成上面那种。如果没有这样类似的需求，还是直接写成后者那种比较好。
**分析**: 锁粗化是JVM默认启动的一种机制，锁粗化针对的是对连续的区域进行分段加锁这种场景，JVM会自发进行优化。但作为开发者而言在满足业务的情况下，应该减少锁的使用。

### 1.5 锁消除

锁消除是在编译器级别的事情。在即时编译器(JIT)时，如果发现不可能被共享的对象，则可以消除这些对象的锁操作。也许你会觉得奇怪，既然有些对象不可能被多线程访问，那为什么要加锁呢？写代码时直接不加锁不就好了。
但是有时，这些锁并不是程序员所写的，有的是JDK实现中就有锁的，比如Vector和StringBuffer这样的类，它们中的很多方法都是有锁的。当我们在一些不会有线程安全的情况下使用这些类的方法时，达到某些条件时，编译器会将锁消除来提高性能。

```java
public static void main(String args[]) throws InterrruptedException{
    long start = System.currentTimeTimeMillis();
    for(int i = 0;i < 20000; i++){
        createStringBuffer("JVM","asdfasdfasdf");
    }
    long bufferCost = System.currentTimeTimeMillis() - start;
    System.out.println("createStringBuffer:"+bufferCost+"ms");
}
public static String createStringBuffer(String s1, String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
```

上述代码中的StringBuffer.append是一个同步操作，但是StringBuffer却是一个局部变量，并且方法也并没有把StringBuffer返回，所以不可能会有多线程去访问它。那么此时StringBuffer中的同步操作就是没有意义的。
开启锁消除是在JVM参数上设置的，当然需要在server模式下：

```tex
-server -XX:+DoEscapeAnalysis -XX:+EliminateLocks
```

并且要开启逃逸分析。 逃逸分析的作用呢，就是看看变量是否有可能逃出作用域的范围。
比如上述的StringBuffer，上述代码中craeteStringBuffer的返回是一个String，所以这个局部变量StringBuffer在其他地方都不会被使用。如果将craeteStringBuffer改成

```java
public static StringBuffer createStringBuffer(String s1, String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
```

那么这个 StringBuffer被返回后，是有可能被任何其他地方所使用的（譬如被主函数将返回结果put进map啊等等）。那么JVM的逃逸分析可以分析出，这个局部变量 StringBuffer逃出了它的作用域。
所以基于逃逸分析，JVM可以判断，如果这个局部变量StringBuffer并没有逃出它的作用域，那么可以确定这个StringBuffer并不会被多线程所访问，那么就可以把这些多余的锁给去掉来提高性能。
当JVM参数为：

```tex
-server -XX:+DoEscapeAnalysis -XX:+EliminateLocks
```

输出：

```tex
createStringBuffer: 302ms
```

JVM参数为：

```tex
-server -XX:+DoEscapeAnalysis -XX:-EliminateLocks
```

输出：

```tex
createStringBuffer: 660ms
```

显然，锁消除的效果还是很明显的。