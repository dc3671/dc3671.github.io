---
layout: post
title:  "Fedora分区扩容以及如何修复引导"
date:   2015-06-09 13:54:19
categories: Linux
excerpt: 分区扩容 修复引导 引导 grub linux 扩容 Fedora
---

* content
{:toc}


## 起因

Linux分区过小，想将Windows下的一个不用的区分出一部分空间来进行扩容。直接在Windows下将D盘减小容量后，重启出现grub rescue字样，无法进入系统。

---

## 解决过程

首先检索grub rescue，结果很多，总结步骤如下：

1.先输入ls回车，查看显示的内容。如：

<pre><code class="shell">(hd0)  (hd0,msdos14) (hd0,msdos13) (hd0,msdos12)....
</code></pre>

2.然后找出哪个盘安装了系统，尝试所有的，直到返回内容不再是error: unknown filesystem.为止。

<pre><code class="shell">ls (hd0,msdos1)/boot
ls (hd0,msdos2)/boot
</code></pre>

3.如果出现包含img文件的目录，即为linux系统所在目录，也是grub2所在目录

4.然后设置grub2启动

<pre><code class="shell">set root=(hd0,msdos13)/boot/grub2
set prefix=(hd0,msdos13)/boot/grub2
insmod normal
normal
</code></pre>

5.便进入之前的引导界面了。
 
但是这只是临时的办法，还需要修复grub之后才能完全解决这一问题，否则重启还会出现一样的界面。这里我因为还要进行扩容，所以就没有直接修复grub，而是先研究扩容去了。

既然grub的问题可以解决，那么我就更大胆了，直接在windows下，用diskdirector将swap分区移动到空闲分区右边，使得空闲分区与linux主分区相邻，便于之后的扩容。

但是这里一个很尴尬的问题是，diskdirector没法直接对linux的主分区进行扩容！在容量的设置上是锁死的！不知道是由于不支持ext4的原因还是本就如此。

没办法，只好换别的工具了。

在之前查如何扩容时，有资料说用fdisk是无法直接进行扩容的，还提到了一个工具叫Gparted，可以用其提供的LiveCD进行U盘启动从而调整分区大小，于是我如法炮制，很轻松地在U盘系统里将主分区完成了扩容。

正当我满心欢喜打算按照之前的方法进入系统再修复grub的时候，发现Linux系统卡在“Reached Target”这样一句话，无法进入系统，这真是当头一桶凉水。

    Reached target Initrd File Systems
    Reached target Initrd Default Target

进行检索之后发现如果等几分钟，系统会出现别的错误信息，我的大致是

    ALERT! /dev/disk/by-uuid/xxxxxxxxx does not exist. Dropping to a shell
initramfs:_

没办法，继续检索，试验解决方案。

在这期间，我也明白了这是什么缘故。因为我改变了swap分区的大小和位置（之前在windows下移动了swap，也改变了大小），导致swap分区的uuid失效了，而linux启动过程中是会按照之前记录的swap分区的uuid进行挂载等操作的，比如/etc/fstab文件里就进行了挂载操作，但是应该不止于此，因为我在liveCD里改了这个文件也并没有修复问题。

那么最终的解决方案是在[这里](http://askubuntu.com/questions/516217/alert-dev-disk-by-uuid-xxxxxxxxx-does-not-exist-dropping-to-a-shell)，Ubuntu下的代码为：

    sudo mount /dev/sda1 /mnt
    sudo mount --bind /dev /mnt/dev
    sudo mount --bind /proc /mnt/proc
    sudo mount --bind /sys /mnt/sys
    sudo chroot /mnt
    update-initramfs -u
    update-grub
    reboot

我又满心欢喜地去LiveCD里进行如法炮制，结果在chroot的时候又报错了，提示执行文件格式不正确。

好吧，查了一会之后发现是Gparted提供的liveCD是32位的，不能对64位系统进行chroot操作...

没办法，找别的LIVECD吧。在找的过程中，发现一个很好的网站，[The LiveCD List · The LiveCD List](http://livecdlist.com/)，列出了一系列的LiveCD和其大致大小以及用途，通过RESCUE标签和是否有64位进行筛选后，再进行U盘制作，启动测试，发现有些无法正常启动，OpenSUSE的RESCUE镜像无法进入桌面，CentOS直接进入也会黑屏，还是调成Basic Graphic模式才能进。

搞了这么久总算快弄完了，在CentOS里输入chroot，成功！但是，接下来两命令又报错...

好吧，这是Ubuntu的命令，我得找Fedora下等价的，如下：

    dracut -f /boot/initramfs-currentimage
    grub2-mkconfig -o "$(readlink /etc/grub2.conf)"

然后重启，还得再输一遍

    set root=(hd0,msdos13)/boot/grub2
    set prefix=(hd0,msdos13)/boot/grub2
    insmod normal
    normal

终于，成功进入了系统！

之后一点收尾工作，修复grub2，命令如下（因为我的系统在第二块硬盘上，所以是sdb，另外这里不需要写数字，比如sdb2之类的）：

    sudo /sbin/grub2-install /dev/sdb

---

## 感想

像grub rescue这种基础的问题，Baidu就能很好提供解决方案了，但是像找update-initramfs和update-grub在Fedora下的等价命令，还是得靠Google，用AOL也行。

以前弄过Windows PE做的U盘系统，现在都能做Linux的U盘系统了，感觉自己这么几年还是学到了一点东西嘛！