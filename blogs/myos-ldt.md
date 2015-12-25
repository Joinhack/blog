---
title: LDT局部描述表
date: '2015-12-25'
description:
categories:
- OS
- Blog
tags:
- LDT
---


LDT局部描述表
===================

ldt在实现多任务功能中，让自己的程序使用的地址空间与其他任务相互不可见。每个程序有自己的LDT.

GDT中的ldt entry
--------------------
![lgt](https://raw.githubusercontent.com/Joinhack/blog/master/images/seg_descriptor.jpeg)

Name         |Description   
---          |--- 
Base Address |LDT的入口地址
Limit        |LDT的大小
Type         |0x2
S            |0 系统描述
DPL          |0
P            |1
AVL          |0
L            |0
D/B          |0
G            |0

LDT设定
Name         |Description
---          |--- 
Base Address |代码段的基址
Limit        |ldt大小
Type         |0x8 代码段
S            |1 数据或代码段描述
DPL          |0
P            |1
AVL          |0
L            |0
D/B          |1 32位段
G            |0



![selector](https://raw.githubusercontent.com/Joinhack/blog/master/images/selector.png)

TI： 1表示此descriptor是ldt


