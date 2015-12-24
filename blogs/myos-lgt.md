---
title: GDT全局描述表
date: '2015-12-18'
description:
categories:
- OS
- Blog
tags:
- GDT
---


GDT全局描述表
------------------------

实模式下内存的地址访问限制是1M， 这是由于8086cpu的使用的寄存器是16位的 而地址总线是20位。1M = 1<<20。

现在cpu的寄存器早就不是什么16位的了，但是计算机现代计算机启动后还是采用实模式。

实模式中就有内存访问限制，要突破这个限制，计算机必须进入保护模式。

实模式中内存寻址才用方式:
```
Segment:Offset
```
Segment 是段寄存器: CS, ES, DS, FS, GS


![segment in real model ](https://raw.githubusercontent.com/Joinhack/blog/master/images/real_mode_mem_seg.jpeg)


转换成物理地址公式
```
PhysicalAddress = Segment * 16 + Offset
```

比如:

```asm
mov $0x7c0, %ax
mov %ax, %ds
mov %ds:(0x20), %ax
```

这里访问的物理地址就是0x7c0*0x10 + 0x20


![segment in protect model ](https://raw.githubusercontent.com/Joinhack/blog/master/images/protected_mode_mem_seg.jpeg)


保护模式的内存寻址(32bits)
-----------------

保护模式突破实模式的内存限制， 能够使用4G的内存。 保护模式中，内存的管理模式分为两种，段模式和页模式。


段模式就采用GDT来进行内存管理。GDT在内存中任意位置， 所以lgdt指令用于让cpu设置gdtr的值， gdtr是一个48bits的寄存器。

gdtr结构如下图:

![gdtr (from OSDev wiki)](https://raw.githubusercontent.com/Joinhack/blog/master/images/gdtr.png)

图片来源于OSWIKI

Offset    | Name        |Description   
---       |---          |--- 
0..15     |Size         |整个GDT表的大小 2Bytes
16..47    |Offset       |GDT的入口位置

```asm
gdtptr:
	.word	(GdtLen - 1)			/* limit */
	.long	0


lgdt gdtptr	

```

使用lgdt来设置.


![GDT (from OSDev wiki)](https://raw.githubusercontent.com/Joinhack/blog/master/images/gdt-descriptor.png)

图片来源于OSWIKI

Offset    | Name        |Description   
---       |---          |--- 
0..15     |Limit        |limit的底4Bytes
16..31    |Base         |base的底4Bytes    
32..39    |Base         |base的中间2Bytes
40..47    |Access Byte  |内存访问控制标志字节
48..51    |Limit        |limit的高4Bytes
52..55    |Flags        |4个段大小标记
56..63    |Base         |Base的高2Bytes

这里比较让人迷惑的是BASE与LIMIT不是完整的内存表示，而是分散分布在里面。


Base Address: 32bits基址 分布在在描述符中分成三块存在。 可以指向4G内存的任意地方。

Limit: 20bits的限制， 由于是20bits所以一般情况限制最大值是1M， 但是可以通过设置页粒度值让限制由1M变成1M*4096(4G)

![access and flags (from OSDev wiki)](https://raw.githubusercontent.com/Joinhack/blog/master/images/gdt-descriptor-flags2.png)

Bit       | Name                 |Description   
---       |---                   |--- 
Pr        |Present               |1 selector 现在不可用， 0 可用.
Privl     |Privilege Level       |3Bits特权级别标识.
Ex        |Executable            |1 表示内存是代码区， 0表示是数据区.
DC(code)  |Direction             |1 表示代码从低优先级开始执行 代码段有效.
DC(data)  |Conforming            |1 端从下向上增加。 0 端从上向下增加.
RW(code)  |Readable              |1 允许读，永远不允许写入
RW(data)  |Readable              |1 允许写入，永远都能读
Ac        |Accessed              |CPU使用此端的时候设置为1， 初始化是0
Gr        |Granularity           |0 表示Limit的单位Byte, 1 limit单位是4096Bytes
Sz        |Size                  |0 表示 16-bit 保护模式， 1 表示32位保护模式


```asm
.macro SEG_DESC Base, Limit, Attr
	.2byte (\Limit & 0xFFFF)
	.2byte (\Base & 0xFFFF)
	.byte  ((\Base >> 16) & 0xFF)
	.2byte ((\Attr & 0xF0FF) | ((\Limit >> 8) & 0x0F00))
	.byte  ((\Base >> 24) & 0xFF)
.endm

......

gdt:
	GDT_DESC_NULL: SEG_DESC 0, 0, 0
	GDT_DESC_C32: SEG_DESC 0, (c32len - 1), (0x9A | 0x4000)
	GDT_DESC_VIDEO: SEG_DESC     0xB8000, 0xFFFF, (0x92)
```

定义一段宏帮助设定描述。gdt是自定义的全局描述表。




