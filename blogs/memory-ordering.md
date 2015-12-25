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
=========================

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

------------------------------------------------


编译时Memory Ordering
------------------------

编译时的reordering发生的原因是编译进行性能优化产生的。

通过下面例子来重现编译器的reordering

测试环境:
	
	* gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-3)
	* CentOS Linux release 6.0 (Final)

代码

```cpp
int A, B;

void test()
{
    A = B + 1;
    B = 0;
}
```

首先来看看在不使用优化参数的情况生成的指令， 使用以下命令。

```shell
$gcc -masm=intel  -S -o - test.c
```

输出的结果(关键部分)

```asm
mov	eax, DWORD PTR B[rip]  #将B放入寄存器eax
add	eax, 1                 #寄存器eax加1，结果还是在eax
mov	DWORD PTR A[rip], eax  #将eax值放入A
mov	DWORD PTR B[rip], 0    #将0 放入B
```

这里就是未乱序的情况下的输出。前三句完成操作A = B+1， 最后一句完成B = 0


下面使用优化参数-O2

```shell
$gcc -masm=intel -O2 -S -o - test.c
```

输出的结果(关键部分)


```asm
mov	eax, DWORD PTR B[rip]  #将B放入寄存器eax
mov	DWORD PTR B[rip], 0    #将0 放入B
add	eax, 1                 #寄存器eax加1，结果还是在eax	
mov	DWORD PTR A[rip], eax  #将eax值放入A
```


这里就是乱序后的输出。 
B = 0的操作， 插入了到中间， 这个中间的意思是完成整个操作后对A进行复制看做一个操作。

编译器为什么可以这么做啦？

我们可以看到就算乱序后，test方法返回时，最终结果是一致的，并不影响最终结果, 因此编译器可以放心大胆的去乱序进行性能优化。


让编辑器reordering 失效， 可以才用上面说的到的方法，插入asm volatile("" ::: "memory");

```cpp
   A = B + 1;
   asm volatile("" ::: "memory");
   B = 0;
```

再来看看结果。

```shell
$gcc -masm=intel -O2 -S -o - test.c


...

mov	eax, DWORD PTR B[rip]
add	eax, 1
mov	DWORD PTR A[rip], eax
mov	DWORD PTR B[rip], 0

...
```

结果和期待一致.

更加详细的说明可以参考[memory-ordering-at-compile-time](http://preshing.com/20120625/memory-ordering-at-compile-time/)

------------------------------------------------

CPU运行时的Memory Ordering
------------------------

[Memory Reordering Caught in the Act](http://preshing.com/20120515/memory-reordering-caught-in-the-act)一文中更加详细解释了memory reordering特性

为什么cpu需要执行时reordering呢？ 当然是为了更快。 现代cpu一般执行情况下每纳秒执行10几个指令，但是要从内存获取数据则需要几十个纳秒。因此，在不影响结果的情况下，cpu在遇到内存操作的时候会出现reorder。[Memory Barriers: a Hardware View for Software Hackers](http://www.rdrop.com/users/paulmck/scalability/paper/whymb.2010.06.07c.pdf)更加具体讲述了这个问题。

原文用到的测试代码GCC版本[ordering.cc](https://gist.github.com/Joinhack/2362552462f71d6d79ad)(已经支持macos), 可以用于重现运行时memory ordering特性.

编译命令

```shell
$gcc -o ordering -O2 ordering.cc -lpthread
```

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

代码分析:

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










