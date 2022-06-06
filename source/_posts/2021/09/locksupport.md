---
title: LockSupport一个很灵活的线程工具类
comments: true
tags: thread
categories: java
translate_title: locksupport-learning
abbrlink: 41846
date: 2021-09-25 12:59:36
---
LockSupport是一个编程工具类， 主要是为了阻塞和唤醒线程用的。所有的方法都是静态方法，可以让线程在任意位置阻塞，也可以在任意位置唤醒

主要的方法： park(阻塞线程)  和  unpark(启动唤醒线程)
```java
//源码
package java.util.concurrent.locks;
import sun.misc.Unsafe;
 
public class LockSupport {
    private LockSupport() {} // Cannot be instantiated.

    private static void setBlocker(Thread t, Object arg) {
        // Even though volatile, hotspot doesn't need a write barrier here.
        UNSAFE.putObject(t, parkBlockerOffset, arg);
    }

    /**
     * @param thread the thread to unpark, or {@code null}, in which case
     *        this operation has no effect
     */
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }

    /**
     * 阻塞当前线程
     * blocker是用来记录线程被阻塞时被谁阻塞的。用于线程监控和分析工具来定位原因的。
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @since 1.6
     */
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        //setBlocker作用是记录t线程是被broker阻塞的
        setBlocker(t, blocker);
        //UNSAFE是一个非常强大的类，他的的操作是基于底层的
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }

    /**
     * 暂停当前线程，有超时时间
     * blocker是用来记录线程被阻塞时被谁阻塞的。用于线程监控和分析工具来定位原因的。
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @param nanos the maximum number of nanoseconds to wait
     * @since 1.6
     */
    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            UNSAFE.park(false, nanos);
            setBlocker(t, null);
        }
    }

    /**
     * 暂停当前线程，知道某个时间
     * blocker是用来记录线程被阻塞时被谁阻塞的。用于线程监控和分析工具来定位原因的。
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @param deadline the absolute time, in milliseconds from the Epoch,
     *        to wait until
     * @since 1.6
     */
    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(true, deadline);
        setBlocker(t, null);
    }

    /**
     * Returns the blocker object supplied to the most recent
     * invocation of a park method that has not yet unblocked, or null
     * if not blocked.  The value returned is just a momentary
     * snapshot -- the thread may have since unblocked or blocked on a
     * different blocker object.
     *
     * @param t the thread
     * @return the blocker
     * @throws NullPointerException if argument is null
     * @since 1.6
     */
    public static Object getBlocker(Thread t) {
        if (t == null)
            throw new NullPointerException();
        return UNSAFE.getObjectVolatile(t, parkBlockerOffset);
    }

    /**
     * 无期限暂停当前线程
     */
    public static void park() {
        UNSAFE.park(false, 0L);
    }

    /**
     * 暂停当前线程，不过有超时时间限制
     */
    public static void parkNanos(long nanos) {
        if (nanos > 0)
            UNSAFE.park(false, nanos);
    }

    /**
     * 暂停当前线程，知道某个时间
     * @param deadline 暂停结束时间
     */
    public static void parkUntil(long deadline) {
        UNSAFE.park(true, deadline);
    }

    /**
     * Returns the pseudo-randomly initialized or updated secondary seed.
     * Copied from ThreadLocalRandom due to package access restrictions.
     */
    static final int nextSecondarySeed() {
        int r;
        Thread t = Thread.currentThread();
        if ((r = UNSAFE.getInt(t, SECONDARY)) != 0) {
            r ^= r << 13;   // xorshift
            r ^= r >>> 17;
            r ^= r << 5;
        }
        else if ((r = java.util.concurrent.ThreadLocalRandom.current().nextInt()) == 0)
            r = 1; // avoid zero
        UNSAFE.putInt(t, SECONDARY, r);
        return r;
    }

    // Hotspot implementation via intrinsics API
    private static final sun.misc.Unsafe UNSAFE;
    private static final long parkBlockerOffset;
    private static final long SEED;
    private static final long PROBE;
    private static final long SECONDARY;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> tk = Thread.class;
            parkBlockerOffset = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("parkBlocker"));
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception ex) { throw new Error(ex); }
    }
}

```
### 与wait / notify对比
LockSupport是用来阻塞和环线线程的，wait/notify同样也是，那么两者的区别是什么？
- wait和notify都是Object中的方法，在调用这两个方法前必须获得锁对象，但是park不需要获取某个对象的锁就可以锁住线程
- notify只能随机选择一个线程唤醒，无法唤醒指定的线程，unpark可以唤醒一个指定的线程

### LockSupport使用
#### 1. 先interrupt在park
```java
public class LockSupportTest {

    public static class MyThread extends  Thread{
        @Override
        public void run() {
            System.out.println(getName() + "进入线程");
            LockSupport.park();
            System.out.println("运行结束");
            System.out.println("是否中断："+Thread.currentThread().isInterrupted());
        }
    }

    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
        System.out.println("线程启动了，但是在内部进行了park");
        thread.interrupt();
        System.out.println("main 线程结束");
    }
}
//输出
//       线程启动了，但是在内部进行了park
//       main 线程结束
//       Thread-0进入线程
//       运行结束
```
#### 2. 先park在interrupt
```java
public static class MyThread extends  Thread{
    @Override
    public void run() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(getName() + "进入线程");
        LockSupport.park();
        System.out.println("运行结束");
    }
}
/**
 * 输出：
 * 线程启动了，但是在内部进行了park
 * main 线程结束
 * Thread-0进入线程
 * 运行结束
 */
```

### 趣味题
用两个线程，一个输出字母，一个输出数字交替输出如：1A2B3C4D...
```java
public class ThreadDemoTest {
    static Thread t1 = null, t2 = null;
    public static void main(String[] args) {
        char[] a = "1234567".toCharArray();
        char[] b = "ABCDEFG".toCharArray();

        t1 = new Thread(() -> {
            for (char i : a) {
                System.out.print(i);
                LockSupport.unpark(t2);
                LockSupport.park();
            }
        }, "t1");

        t2 = new Thread(() -> {
            for (char i : b) {
                LockSupport.park();
                System.out.print(i);
                LockSupport.unpark(t1);
            }
        }, "t1");
        t1.start();
        t2.start();
    }
}
//输出：  1A2B3C4D5E6F7G
```
使用自旋锁也可以实现上面的结果
```java
public class CasTest {
    //定义枚举，包含两个变量
    enum ReadyToRun{T1, T2};

    static volatile ReadyToRun r = ReadyToRun.T1;

    public static void main(String[] args) {

        char[] a = "1234567".toCharArray();
        char[] b = "ABCDEFG".toCharArray();

        new Thread(()->{
            for (char c : a){
                //当r不为T1时， 空转占着cpu等待，然后输出字符，将r的值设置为T2
                while (r != ReadyToRun.T1){}
                System.out.print(c+" ");
                r = ReadyToRun.T2;
            }
        },"t1").start();
        new Thread(()->{
            for (char c : b){
                while (r != ReadyToRun.T2){}
                System.out.print(c+" ");
                r = ReadyToRun.T1;
            }
        },"t2").start();
    }
}
```

