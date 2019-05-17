---
layout: post
title: Arch Linux 傻瓜安装笔记
categories: development
tags: [Linux]
---

因为没有傻瓜的安装引导程序，安装Arch的工作是一个对linux菜鸟很艰难的工作，但是当你能安装的时候，你就会对linux有更好的理解，我把我的安装Arch的经验与大家分享一下，希望对大家有帮助。

首先你需要一个到arch的官方网站下载一个镜像，然后先用镜像在虚拟机中安装，我是在虚拟机中尝试三次后安装到物理环境中的。

安装步骤

首先确认你可以连接到互联网上面，因为Arch的安装需要在联网环境下进行。

可以用以下两个命令去连接互联网：

有线连接 `# dhcpcd`

无线连接 `# wifi-menu`

## 分区

需要手动分区，fdisk和cfdisk都可以，前者是交互式命令行，后者是命令行下的图像化操作。至于如何操作，请搜索这两个工具的使用教程，因为不是这篇文章的重点，所以跳过。

可以把linux全部安装在一起，也可以选择不同的的系统部件安装在不同的分区中。

因为我是1个128G的SSD和1T的普通硬盘我的分区方案是这样的

将SSD分为3部分
系统内核 200M /boot
变量数据 16G /var
根分区 112G /

普通硬盘直接格式化挂载到/home

没有使用交换分区，因为我的内存8G我觉得交换分区是不需要的。

分区结束后执行格式化命令

```shell
mkfs.ext4 -b 4096 /dev/sda1
mkfs.ext4 -b 4096 /dev/sdb1
mkfs.ext4 -b 4096 /dev/sdb2
mkfs.ext4 -b 4096 /dev/sdb3
````

在我的系统分区中sda是普通硬盘，sdb是SSD

当分区完成后就是挂载，需要将硬盘挂载到镜像的/mnt目录，这个目录是linux中专门用来挂载外部设备的，比如U盘，光驱之类的 。

```shell
# mount -t ext4 /dev/sda1 /mnt/home
# mount -t ext4 /dev/sdb1 /mnt/boot
# mount -t ext4 /dev/sdb2 /mnt/var
# mount -t ext4 /dev/sdb3 /mnt
````

修改软件源，软件源是arch的软件仓库，虽然不修改使用默认的也可，但速度体验就不是很好了。

`https://www.archlinux.org/mirrorlist/`

在官方的软件源生成页面选择china可以获取到最新的国内软件源

修改软件源配置文件

```shell
vim /etc/pacman.d/mirrorlist
nano /etc/pacman.d/mirrorlist
````

以上只是用两种不同的编辑器进行编辑，看你喜欢哪种了。

```shell
sed -i "s/^/#/g" /etc/pacman.d/mirrorlist //
```

该命令可以使文件全部被注释，记得备份。然后把上面网址获取的最新源粘贴进去。

```shell
pacman -Syy
````

更新一下软件源

```shell
pacstrap /mnt base base-devel vim
```

安装基本系统，base是基础软件包组，base-devel是基础开发包组，vim是世界上最好的编辑器。

生成fstab

```shell
genfstab -U -p /mnt > /mnt/etc/fstab
```

切换到新安装的系统中

```shell
arch-chroot /mnt
```

```shell
passwd root
```

运行passwd，设置root密码，要敲两遍，不要忘了它。

## 主机名

用vim打开/etc/hostname，往里面写一个作为主机名的名字，只要字母、横线和数字。

## 语言环境

然后用vim打开/etc/locale.gen，然后找到以下四行，取消注释:

```shell
en_US.UTF-8
zh_CN.UTF-8
zh_CN.GBK
zh_CN.GB2312
```

然后运行 `# locale-gen`

再编辑`/etc/locale.conf`，里面写上

```shell
LANG="en_US.UTF-8"
```

现在生成启动要用到的ramdisk

```shell
mkinitcpio -p linux
```

```shell
pacman -S wpa_supplicant dialog
```

保证新系统可以联网，现在你可以安装其他觉得需要的包。


## 引导器

```shell
pacman -S grub
grub-install --target=i386-pc --recheck --debug /dev/sdb
grub-mkconfig -o /boot/grub/grub.cfg
```

我的系统是安装到/dev/sdb中的，根据自己实际情况修改此参数

接下来执行

```shell
exit
umount -R /mnt
reboot
```

这些命令分别是：

退出硬盘的linux系统

取消/mnt下的所用挂载

重启

进入新安装的系统后执行

```shell
useradd -m admin
passwd admin
```

新建一个叫admin用户给其设置密码，平时使用linux不应该在root用户下进行。

记得执行最开始的那两个连接网络的命令的其中一个，不然没有网络，接下来你是无法安装软件的。

其实这已经算安装完成了arch，但这在服务器上可以，在你的个人电脑上体验还是差些，你可以安装一个桌面环境。

我选择的是gnome，看你的喜好了。

其他的软件啊，配置啊，等等这些有两位前辈已经写好了，我参考了他们的教程来安装我的arch，大家可以拿来参考。

[官方安装教程](https://wiki.archlinux.org/index.php/Installation_guide_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)

最后官方的wiki是相当好的教程，你遇到的arch的基础问题基本都可以在里面找到解决方法，学会使用它。












