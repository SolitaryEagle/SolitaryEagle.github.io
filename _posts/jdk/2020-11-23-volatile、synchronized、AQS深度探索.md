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



