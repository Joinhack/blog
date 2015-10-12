---
title: Memory Ordering
date: '2015-10-12'
description:
categories:
- read-mark
- Blog
tags:
- memory-ordering
---


Memory Ordering
------------------------

Memory Ordering指的cpu访问内存的顺序， 这个术语包含2个层面的意义， 编译时（编译器乱序）与运行时（运行时乱序）。

现今的微处理器都能通过memory ordering特性让cpu重排内存指令， 这种特性叫做[out-of-order execution](https://en.wikipedia.org/wiki/Out-of-order_execution)(最终结果是不会受到影响的。)

通过加入memory barrier来屏蔽memory ordering.

编译时memory barrier(GNU):

asm volatile("" ::: "memory");

或者

__asm__ __volatile__ ("" ::: "memory");

运行时memory barrier(x86):
asm volatile("mfence" ::: "memory");

更多详细的需要查看[Memory Ordering](https://en.wikipedia.org/wiki/Memory_ordering)

[Memory Reordering Caught in the Act](http://preshing.com/20120515/memory-reordering-caught-in-the-act)一文中更加详细解释了memory reordering特性

原文用到的测试代码GCC版本[ordering.cc](https://gist.github.com/Joinhack/2362552462f71d6d79ad)(已经支持macos), 可以用于重现运行时memory ordering特性.

编译命令
$gcc -o ordering -O2 ordering.cc -lpthread

当USE_CPU_FENCE=0的时候程序没有加入memory barrier, 运行结果:

```
...
545 reorders detected after 81138 iterations
546 reorders detected after 81149 iterations
547 reorders detected after 81207 iterations
548 reorders detected after 81231 iterations
549 reorders detected after 81246 iterations
550 reorders detected after 81264 iterations
551 reorders detected after 81280 iterations
```

此时reorder被检测到了。

共享变量

```cpp
int X, Y;
int r1, r2;
SEMAPHORE beginSema1;
SEMAPHORE beginSema2;
SEMAPHORE endSema;
```

线程1

```cpp
void *thread1Func(void *param)
{
    MersenneTwister random(1);
    for (;;)
    {
        SEM_WAIT(&beginSema1);  //等待信号通知
        while (random.integer() % 8 != 0) {}  // Random delay

        // ----- 事务! -----
        X = 1;
        asm volatile("" ::: "memory");  // 防止编译时乱序
        r1 = Y;
        SEM_POST(&endSema);  // 通知main线程
    }
    return NULL;  // Never returns
};
```

线程2

```cpp
void *thread2Func(void *param)
{
    MersenneTwister random(2);
    for (;;)
    {
        SEM_WAIT(&beginSema2);  //等待信号通知
        while (random.integer() % 8 != 0) {}  // Random delay

        // ----- 事务! -----
        Y = 1;
        asm volatile("" ::: "memory");  // 防止编译时乱序
        r2 = X;
        SEM_POST(&endSema);  // 通知main线程
    }
    return NULL;  // Never returns
};
```

main线程

```cpp
for (int iterations = 1; ; iterations++)
  {
      // Reset X and Y
      X = 0;
      Y = 0;
      // 当设置好X Y初始值后， 通知2个线程
      SEM_POST(&beginSema1);
      SEM_POST(&beginSema2);
      //等待线程1与线程2
      SEM_WAIT(&endSema);
      SEM_WAIT(&endSema);
      //检查线程1与线程2修改的的结果
      if (r1 == 0 && r2 == 0)
      {
          detected++;
          printf("%d reorders detected after %d iterations\n", detected, iterations);
      }
  }
```
当线程1 与线程2 都运行完后 r1 与 r2 应该是都为1，我们期望他是1. 但是reorder的存在改变了这种情况。

解决办法， 插入memory barrier(运行时), 将线程1与线程2方法里面的asm volatile("" ::: "memory");替换成asm volatile("mfenece" ::: "memory");(只限于x86 CPU)










