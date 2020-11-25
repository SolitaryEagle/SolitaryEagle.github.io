---
layout:     post
title:      Java volatile、synchronized、AQS深度探索
subtitle:   (JMM) Java Memory Model
date:       2020-11-19
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       JDK
---

> MESI:
> * https://cloud.tencent.com/developer/article/1548942
> * https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE
> * https://blog.csdn.net/reliveIT/article/details/50450136
> 

# volatile

Java 虚拟机规范中的定义
* volatile 变量具有可见性
    * JVM 通过 CPU 的 MESI 协议来实现
* volatile 变量的指令禁止重排序
    * JVM 通过“内存屏障”来实现
    
    ![DYIQ8e.png](https://s3.ax1x.com/2020/11/23/DYIQ8e.png)


volatile 变量的可见性证明
```
class NotVolatile {

    final int MAX = 5;
    // 当 initValue 变量没有被 volatile 修饰时，线程 thread1 可能不会结束
    volatile int initValue = 0;

    public void reader() {
        int localValue = initValue;
        while (localValue < MAX) {
            if (initValue != localValue) {
                System.out.println(Thread.currentThread().getName() + " --> initValue: " + initValue);
                localValue = initValue;
            }
        }
    }

    public void updater() {
        int localValue = initValue;
        while (localValue < MAX) {
            System.out.println(Thread.currentThread().getName() + " --> localValue: " + ++localValue);
            initValue = localValue;
            sleep(2000);
        }
    }

    public static void main(String[] args) {
        NotVolatile notVolatile = new NotVolatile();
        Thread thread1 = new Thread(notVolatile::reader, "Reader");
        Thread thread2 = new Thread(notVolatile::updater, "Updater");
        thread1.start();
        thread2.start();
    }
}
```

JVM 中存在指令重排序证明
```
public class ReorderTest {
    private static int a;
    private static int b;
    private static int x;
    private static int y;

    public static void main(String[] args) {
        long count = 0;
        while (true) {
            ++count;
            a = b = 0;
            x = y = 0;
            Thread thread1 = new Thread(() -> {
                a = 1;
                x = b;
            });
            Thread thread2 = new Thread(() -> {
                b = 1;
                y = a;
            });
            thread1.start();
            thread2.start();
            join(thread1);
            join(thread2);
            // 只有存在指令重排序的情况下，这个条件才会满足
            if (x == 0 && y == 0) {
                System.out.println("程序循环了: " + count + " 次");
                break;
            }
        }
    }
    private static void join(Thread thread) {
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
![DYTppF.png](https://s3.ax1x.com/2020/11/23/DYTppF.png)

备注
* volatile 变量读操作的性能消耗与普通变量几乎没有什么差别，但是写操作则可能会慢上一些，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行

使用场景
* 一处写入，到处读取（可见性）
* 状态标记变量（可见性，禁止重排序）
* 懒汉式单例模式的 Double Check（禁止 new 指令重排序）

------------------------------------

> https://blog.csdn.net/tongdanping/article/details/79647337

# synchronized

约定:
* 监听器对象: synchronized 修饰的对象 --> mutex

底层实现逻辑
* java 代码: synchronized 关键字
* jvm 字节码: monitorenter、monitorexit 指令
* jvm 执行过程中: 锁升级（偏向锁 --> 轻量级锁 --> 重量级锁）
* CPU 汇编指令: lock comxchg 指令


锁升级（偏向锁 --> 轻量级锁 --> 重量级锁）

32位虚拟机中的 MarkWord:

![DUe5ZQ.jpg](https://s3.ax1x.com/2020/11/24/DUe5ZQ.jpg)

64位虚拟机中的 MarkWord:

![DUmPRx.jpg](https://s3.ax1x.com/2020/11/24/DUmPRx.jpg)

* 偏向锁: MarkWord 中会记录偏向线程的 Id
    * mutex 必须没有计算过 hashCode
    * mutex 没有偏向过，或（偏向的线程是自己或已经死亡）
* 轻量级锁: 线程首先在栈内存中分配一个 LockRecord 用来保存 MarkWord 中原来的值，并在 MarkWord 中记录 LockRecord 的指针；释放锁时，会把原来的 MarkWord 复制回去
    * 多个线程对共享内存只存在交替访问（并发）
* 重量级锁: 操作系统实现
    * 多个线程对共享内存存在同时访问（并行）

![DUmaYn.jpg](https://s3.ax1x.com/2020/11/24/DUmaYn.jpg)

# AQS
