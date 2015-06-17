---
title: 读书笔记
date: '2015-06-05'
description:
categories:
- read mark
- Blog
tags:
- read mark
---

保守GC(conservative collector) 的缺点

1.不能保证栈上当前值是否是指向堆(指针)，也就是会被误认. 文章[Finding References in Java™](http://citeseer.ist.psu.edu/viewdoc/download?doi=10.1.1.47.6924&rep=rep1&type=pdf) 有相关描述，原文" A
conservative collector knows only that some region of memory may contain references, but doesn’t know whether or
not a given value in that region is a reference."

2.由于上面的不确定性造成在想移动对象的时候，没有办法修改栈上对应的值。因此保守GC不能出现在对象可移动的GC算法, 比如mark-compact, copy等。

精确GC(exact)的缺点

需要额外的信息来描述栈上的当前值是否是指向的堆(指针), 标记额外值有以下方式。

1.标记法(tagging) 值本身携带自描述信息

2.stack map 通过编译器找出偏移信息. 

stack map 方式的 倒是常见 java使用的OopMap. golang 使用的BitVector结构（// Information from the compiler about the layout of stack frames.） 



这种强类型语言通过编译器的帮助生成相应的信息。那如果是那些弱类型语言（比如lua, javascript）会怎办啦？他们在finder references 会怎么做啦？

lua 是通过将所有GC对象都放入一个链表 这样来避开finder references

	/*
	** create a new collectable object (with given type and size) and link
	** it to 'allgc' list.
	*/
	GCObject *luaC_newobj (lua_State *L, int tt, size_t sz)


在阅读[V8设计](https://developers.google.com/v8/design?csw=1#efficient-garbage-collection)的时候发现V8居然被设计成为精准GC方式。原文"always knows exactly where all objects and pointers are in memory. This avoids falsely identifying objects as pointers which can result in memory leaks."









