title: install archlinux from hard disk
tags: archlinux
---
1. 准备工作
从官方[下载地址](https://www.archlinux.org/download/)选择比较近的源下载最新的镜像文件。这里选择哈工大的[下载源](http://run.hit.edu.cn/archlinux/iso/2016.02.01/archlinux-2016.02.01-dual.iso).把镜像文件中的**VMLINUZ**、**ARCHISO.IMG**放到windows的系统盘根目录下.
下载引导**wingrub**或者**EasyBCD**，安装之后将menu.lst中的内容更换如下：
```
timeout 5
default 0

title  Install Arch Linux
root  (hd0,0)
kernel /vmlinuz archisolabel=ARCH_201602
initrd /archiso.img
```
