---
title: stack
date: '2015-9-30'
description:
categories:
- read-mark
- Blog
tags:
- stack
---

![Contiguous stack](https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub)


split stack原理与问题
----------------------

1. 栈满后每次调用将使用新的stack， 当调用返回会释放栈。如果同样的调用反复发生在循环里面 alloc/free的导致严重的消耗。

2. 栈的申请与释放工作不会停止的（这里我的理解是每次要将当前栈大小传递给函数），都有额外的工作需要做（检查工作负担很重啊）。

原文：

  Current split stack mechanism has a “hot split” problem - if the stack is almost full, a call will force a new stack chunk to be allocated.  When that call returns, the new stack chunk is freed.  If the same call happens repeatedly in a tight loop, the overhead of the alloc/free causes significant overhead.

  Stack allocation/deallocation work is never complete with split stacks - every time the stack size passes a threshold in either direction, extra work is required.


Contiguous stack:
------------

Contiguous stack 用于避免split stack带来的问题。

Contiguous stack原理就是当栈要用完的时候， 申请一个新的更大的， 将老栈数据拷贝进新栈， 当然还有一个麻烦的东西要做，指针（指向栈的指针）调整。

当检查（比split stack的检查优势）到当前栈大小不够时就要做copy stack工作了。

1. 分配一个新的更大的栈。

2. 拷贝老栈数据到新栈。

3. 调整真正的指针（我的理解是通过[stack map](http://joinhack.github.io/read-mark/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)辨别指针）

4. 调整指向栈的指针。








