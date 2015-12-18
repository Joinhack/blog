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
40..47    |Acess Byte   |内存访问控制标志字节
48..51    |Limit        |limit的高4Bytes
52..55    |Flags        |4个段大小标记
56..63    |Base         |Base的高2Bytes

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


