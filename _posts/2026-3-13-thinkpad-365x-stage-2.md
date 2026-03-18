---
title: Thinkpad 365x 折腾 PartB
description: 约等于在Basic Linux上开荒
author: MoyaMryia
date: 2026-3-13 10:30:00 +0800
categories:
  - 折腾
tags:
  - 折腾
  - ThinkPad
math: true
---

## 顺着之前的来，反正已经有OS了

### 打开swap

之前已经分好区了，直接在开机的脚本里写一句
```bash
# /etc/rc.local 还是别的文件来着
swapon /dev/hda3
```
### 图形界面

```bash
startx
```
其实这样就能进图形界面了，只不过没有鼠标... 为啥呢？翻了一下Xorg的log来看，他似乎一直在试图找一个/etc/psaux.img（或者其他的玩意，没区别的都没有），然后就不出意外的找不到，于是没法用鼠标。

怎么解决呢？

```bash
# startx 我忘了在哪里了，先which startx一下

# mount -o loop /etc/$2.img /dev

xinit -- /usr/X11/bin/Xvesa -screen $1 -mouse /dev/psaux,psaux -terminate

# 以前的鼠标那部分写的是 -$3button
```

然后就解决了。看起来这个显卡的兼容性不错，Xorg下跑的很欢乐。唯一的问题是：太菜了，纯亮机卡。GNU LinCity的开头加载动画卡爆了。

### 日式键盘

过两天研究一下

### 往上拷东西

CF转PCMCIA死活不认识，直接拔主卡然后往上拷算了

### 声卡

直接加载驱动，把IRQ挂在并口的位置上就好了

### 上网

没错这玩意真能上网，Basic Linux已经有浏览器了，还是个带GUI的非文字浏览器，理论上是真的能玩点什么的。

### 安装Windows

为啥安装Windows呢？主要是Linux不认我的CF转PCMCIA，并且似乎声卡也一般，不太好使的样子。所以我还是打算装一个Windows（反正我现在一堆CF卡）。不过该怎么办呢？之前我尝试过让虚拟机直接挂载物理磁盘安装，结果出事了，开机就报Disk I/O ERROR.

但是现在咱知道咋回事了，请看VCR：

```bash
# Part of hexdump -C ~/MS_DOS/boot_sector.bin
# boot_selecter.bin is from Windows 95 OSR2 boot disk.

00000180  18 01 27 0d 0a 49 6e 76  61 6c 69 64 20 73 79 73  |..'..Invalid sys|
00000190  74 65 6d 20 64 69 73 6b  ff 0d 0a 44 69 73 6b 20  |tem disk...Disk |
000001a0  49 2f 4f 20 65 72 72 6f  72 ff 0d 0a 52 65 70 6c  |I/O error...Repl|
000001b0  61 63 65 20 74 68 65 20  64 69 73 6b 2c 20 61 6e  |ace the disk, an|
000001c0  64 20 74 68 65 6e 20 70  72 65 73 73 20 61 6e 79  |d then press any|
000001d0  20 6b 65 79 0d 0a 00 00  49 4f 20 20 20 20 20 20  | key....IO      |
000001e0  53 59 53 4d 53 44 4f 53  20 20 20 53 59 53 7f 01  |SYSMSDOS   SYS..|
000001f0  00 41 bb 00 07 80 7e 02  0e e9 40 ff 00 00 55 aa  |.A....~...@...U.|
```

也就是说，其实如果看到这句话，说明已经进入MBR了，这句话不是BIOS说的！（严格来说，是MBR里的代码给BIOS发中断让他说的）

这就给了我们一个很离谱的思路：也就是先给电脑写个汇编让他把硬盘参数吐出来，然后配置一个一模一样的虚拟硬盘安装上Win95，反汇编查看细微区别然后小修一下，最后直接写进去补驱动不就好了？

结果还是不行，淦。