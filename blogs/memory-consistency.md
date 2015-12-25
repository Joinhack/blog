---
title: Memory Consistency
date: '2015-10-21'
description:
categories:
- read-mark
- Blog
tags:
- memory-consistency
---


内存一致性模型([Memory Consistency Models](http://www.cs.nmsu.edu/~pfeiffer/classes/573/notes/consistency.html))
=========================

注解

W(var)value: 将变量var赋值为value.

R(var)value: 读取变量var的值是value.

例子: W(x)1 将变量x赋值为1.  R(y)3 读取变量y的值为3


严格一致性(Strict Consistency)
------------------------------------

严格一致性的定义：读取地址X返回的是最近写入X的值。（如果没有缓存的cpu，直接通过总线与内存交互，这时就是严格的一致性的内存模型）。


原文:"read to a memory location X returns the value stored by the most recent write operation to X"


顺序一致性(Sequential Consistency)
------------------------------------

Lamport对顺序一致性的定义:"the result of any execution is the same as if the reads and writes occurred in some order, and the operations of each individual processor appear in this sequence in the order specified by its program."

这里顺序一致性有2个含义

	全局观来讲 在读写循序一致的情况所有执行结果是一样的。
	单处理器情况下 顺序是被程序指定的。



内存一致性([Cache Coherence](https://en.wikipedia.org/wiki/Cache_coherence))
--------------------------------

cache coherence是指共享资源保存到每个cpu cache中的一致性。 

![An illustration showing multiple caches of some memory, which acts as a shared resource](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a1/Cache_Coherency_Generic.png/370px-Cache_Coherency_Generic.png)

图中上面的client已经将memory的数据加载到了cache中， 而下面的client修改memory， 这个时候上面的client只读到了cache的数据。而cache coherence就是来解决这种混乱的保持memory与cache的一致性。

很多时候人们将内存一致性与顺序一致性视为一致的。但是并不是这样的。顺序一致性是全局内存操作的全局视觉，而内存一致性是个局部视觉。下面例子就满足内存一致性但是不满足顺序一致性。

```
P1:  W(x)1 W(y)2
-----------------------
P2:        R(x)0 R(x)2 R(x)1 R(y)0 R(y)1
-----------------------
P3:        R(y)0 R(y)1 R(x)0 R(x)1
-----------------------
P4: W(x)2 W(y)1
```








参考：
[Shared Memory Consistency Models](http://www.hpl.hp.com/techreports/Compaq-DEC/WRL-95-7.pdf)