---
title: sychronized
date: '2015-08-26'
description:
categories:
- read-mark
- Blog
tags:
- read-mark jvm
---

Java Object 标记头
-------------------

![ScreenShot](https://raw.githubusercontent.com/Joinhack/blog/master/images/headmark.jpg)

轻量级锁(light weight lock)

当年JVM对轻量级锁的处理[JVM](https://www.usenix.org/legacy/event/jvm01/full_papers/dice/dice.pdf)

文中讲到monRec对于过来就是[ObjectMonitor](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/runtime/objectMonitor.hpp?#l77)

ObjectMontor的[分配操作](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp?#l944)是在Thread空间，因此没得线程安全问题。




