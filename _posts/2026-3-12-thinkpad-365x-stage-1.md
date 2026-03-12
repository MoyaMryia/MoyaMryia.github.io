---
title: Thinkpad 365x 折腾 PartA
description: 这只是一台奔腾I的机器，折腾起来应该不难吧？不难吧...？？？？
author: MoyaMryia
date: 2026-3-12 18:00:00 +0800
categories:
  - 折腾
tags:
  - 折腾
  - ThinkPad
math: true
---

### 序：

前两天很无聊，于是莫亚想整点东西玩玩，然后在闲鱼上看到了一台Thinkpad 365x，成色不错，没有硬盘（记住这里），有一张能进MSDOS的启动盘。莫亚觉得不错，于是买下来了。收到货后感觉很不错，卖家人挺好，还给了个启动盘，能玩马里奥。

然后这个故事就不出意外的出意外了。

先是我买的CF卡死活不读，然后是我买的垃圾USB软驱和Windows Defender把启动软盘干炸了，然后是买的第一根内存死活用不了（第二根没问题，感谢闲鱼上的IBM古董博物馆），然后是别的CF卡也用不了，这个时候我把这个机子扔柜子里打算哪天给别人算了。

### 一线转机

然后我突然有了个主意：这需要我们复习一下MBR分区表+传统BIOS的启动流程。

开机拉起BIOS，然后BIOS会直接读取硬盘的第一个512字节，然后再往后跳进系统。

那我们可以验证了，我们开机往第一个扇区里写汇编试试？但是x86汇编我也不会写啊（我只会写点riscv），于是我让Gemini随便写了个能塞进那个扇区的汇编试了试。我草真能用啊？那看起来就是启动盘什么的锅了？？？逆天啊？？？

行，那研究一下呗。

然后我的脑子又想到了一个更逆天的主意：我要是就想能用，干嘛要在Win上吊死？有道理啊？然后我翻了半天，发现有个东西叫grub4dos，这不就完了？

那写呗？

我草真进了？

### 那不就完了？我草真完了

对着这玩意傻乐了一晚上，然后打算拿这个东西引导DOS启动，结果死活不行，给我整不会了。然后我又把这个格式化了试图往里边修一个Win95引导手动启动DOS，结果还是不行？（以后来研究一下这是咋回事）。那算了吧，装Linux算了。

然后我犯了个抽（我为了这个傻子事情甚至对这玩意的引导开了反汇编），拷MBR引导的时候就把最前面扔进去了，于是：

No MBR Helper.

好吧...反汇编之后我才想起来这玩意不应该只有512字节吧？研究了一顿发现整个grub其实就占了前几个扇区，和后面一点关系没有。那简单了，我先重建分区表，然后手动把这玩意dd进去不就好了？还真是。

那就先重建分区，63扇区开头，分一个FAT16，一个ext2，一个linuxswap，格式化完成，设定第一个分区能启动。

```bash
sudo mkfs.vfat -F 16 -n "BOOT" /dev/sda1
sudo mkfs.ext2 -I 128 -L "BasLin" /dev/sda2
sudo mkswap /dev/sda3
```
然后手写引导：
    
```bash
sudo dd if=grldr.mbr of=/dev/sda bs=446 count=1 conv=notrunc
sudo dd if=grldr.mbr of=/dev/sda bs=512 skip=1 seek=1 count=17 conv=notrunc
```

我们安装的是Basic Linux，需要挂载一下：

```bash
mkdir -p ./mnt_boot
sudo mount /dev/sda1 ./mnt_boot
sudo cp grldr menu.lst ./mnt_boot/
sudo cp zimage.p1 ./mnt_boot/vmlinuz
ls -l ./mnt_boot/
sudo umount ./mnt_boot
```

我们需要把fs.img写进去再写一下新的fstab:

```bash
mkdir -p ./mnt_root
sudo mount /dev/sda2 ./mnt_root
mkdir -p ./mnt_source
sudo mount -o loop ./fs.img ./mnt_source
sudo cp -a ./mnt_source/. ./mnt_root/
sudo vim ./mnt_root/etc/fstab
```

```
#/etc/fstab:

/dev/hda2    /        ext2    defaults,noatime    1 1
/dev/hda3    none     swap    sw                  0 0
/dev/hda1    /mnt/dos vfat    noauto,user         0 0
none         /proc    proc    defaults            0 0
```

最后写一个menu.lst:

```
timeout 5
default 0

color cyan/blue white/magenta

title [1] BasicLinux 3.5 (Ext2 Root)
    root (hd0,0)
    kernel /vmlinuz root=/dev/hda2 rw vga=normal nomodeset

title [2] BasicLinux (Safe Mode - Low Memory)
    root (hd0,0)
    kernel /vmlinuz root=/dev/hda2 rw vga=normal single mem=24M

title [3] Enter Grub4Dos Shell
    commandline

title [4] Reboot System
    reboot

title [5] Shutdown System
    halt
```

### 开机了？

好吧这里有个小问题，这个远古的Linux不喜欢256的时间戳，所以需要这样：

```bash
sudo mkfs.ext2 -I 128 -L "BasLin" /dev/sdX1
```

不然他会因为认不出这个长度的时间戳闹Kernel Panic.

然后就启动了。终于开机了。

之后还是可以整点别的，比如说打开显卡支持，搞好驱动，启动图形界面，甚至想办法重新折腾一次装个Windows上去（？）

先这样，反正能把差点要进垃圾桶的东西搞成这样我已经很开心了（
