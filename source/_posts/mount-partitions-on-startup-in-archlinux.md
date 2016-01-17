title: mount partitions on startup in archlinux
date: 2016-01-17 17:36:47
tags: [archlinux, mount partitions, grub rescue]
---

机器装了两个系统，一个win7备用，一个linux生产。但是最近linux的剩余磁盘空间仅10G多，  
眼看就要不够用了，于是想着挤挤win7，分些空间出来给linux。

#### grub rescue的处理
第一步，先在win7中压缩了20G出来，然后重启机器准备进入linux加载新分区，此时出现了  
问题，grub无法正常启动，进入了rescue界面。
解决办法是：
1. 列出所有分区
grub rescue> ls
2. 依次ls每个分区直到找到不报'unknown filesystem'的分区，比如n=5
grub rescue> ls (hd0,msdos5)/boot/grub
3. 依次执行以下命令，可进入正常的系统引导界面
grub rescue> set root=(hd0,msdos5)/boot/grub
grub rescue> set prefix=(hd0,msdos5)/boot/grub
grub rescue> insmod normal
grub rescue> normal
4. 进入linux操作系统，执行以下命令修复grub，重启后grub可恢复正常
$ sudo grub-install /dev/sda

#### archlinux挂载分区
第二步，配置新分区在linux启动的时候自动挂载，并映射到某一个文件路径，比如/extend
1. 找到要挂载的新分区硬盘号码，比如/dev/sda5
$ sudo fdisk -l
2. 格式化/dev/sda5为linux系统的文件格式(也可使用其它稳定的文件格式)
$ sudo mkfs.ext4 /dev/sda5
3. 配置新分区在系统启动的时候挂载，并映射为/extend
修改<code>/etc/fstab</code>文件，在最后添加一行，注意使用对应的UUID并修改加载顺序
UUID=59f8ba70-62d4-4d1d-982e-a33166d59394  /extend  ext4  rw,relatime,data=ordered   0 3