---
title: 分页内存管理
date: '2015-12-22'
description:
categories:
- OS
- Blog
tags:
- PAGING
---


分页内存管理
=========================

32位x86体系cpu支持4G虚拟地址空间而64位cpu则支持256T（理论上是16E）。分页内存允许进程访问整个虚拟内存空间， 而不是直接访问实际的物理内存(与分段的目的相似)。

前面说过分页是可选的一种内存方式， 通过设置CR0的PG位来让页转换生效。

在x86上[MMU](http://wiki.osdev.org/MMU)映射内存是通过连续的2张表进行的，一张叫页目录，一张叫页表。

页帧
---------------------

页帧是4K-byte 为单位的连续物理内存地址

页目录
----------------------

![page directory (from OSDev wiki)](https://raw.githubusercontent.com/Joinhack/blog/master/images/Page_dir.png)

图片来源于OSWIKI


Flag  | Name           |Description   
---   |---             |--- 
S     |Page Size       |也大小标志位, 1: 页表是4M 0: 4K
A     |Accessed        |1 表示cpu正在访问 
D     |Cache Disable   |cache禁止位，1 不允许cache
W     |Write-Through   |write-through标志位
U     |User/Supervisor |特权位, 1 全局访问， 0 只能是supervisor能够访问。
R     |Read/Write      |读写标志位
P     |Present         |页是否在物理地址 1 在内存中， 0 被swapped。 如果在内存中能直接访问，如果被swapped， 访问时会报错。


![page table (from OSDev wiki)](https://raw.githubusercontent.com/Joinhack/blog/master/images/Page_table.png)

图片来源于OSWIKI
