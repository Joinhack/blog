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
------------------------------------------


严格一致性(Strict Consistency)
------------------------------------

严格一致性的定义：读取地址X返回的是最近写入X的值。（如果没有缓存的cpu，直接通过总线与内存交互，这时就是严格的一致性的内存模型）。


原文:"read to a memory location X returns the value stored by the most recent write operation to X"


顺序一致性(Sequential Consistency)
------------------------------------

Lamport对顺序一致性的定义:"the result of any execution is the same as if the reads and writes occurred in some order, and the operations of each individual processor appear in this sequence in the order specified by its program."

这里顺序一致性有2个含义

	1. 全局观来讲 在读写循序一致的情况所有执行结果是一样的。

	2. 单处理器情况下 顺序是被程序指定的。






参考：
[Shared Memory Consistency Models](http://www.hpl.hp.com/techreports/Compaq-DEC/WRL-95-7.pdf)