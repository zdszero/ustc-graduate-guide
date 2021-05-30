---
title: Arch Linux的安装教程
date: 2021-01-01 10:52:11
draft: true
---

## 安装前的准备

将iso文件写入U盘，在linux中使用dd命令

```
dd if=iso文件路径 of=U盘对应的文件 status="progress"
```

在windows中可以用rufus写入

## 安装过程

### 设置启动顺序

在开机时根据电脑型号按动特定键进入bios，设置优先从U盘中启动，同时关闭secure boot，插入U盘后启动进入安装界面

### 检查引导方式

现在的电脑一般支持UEFI，但是为了保险还是检查一下电脑的启动方式，这一步会影响到后续的分区以及boot loader的安装

```
ls /sys/firmware/efi/efivars
```

如果结果显示`no such file or directory`则说明电脑以EFI方式引导，否则说明电脑以BIOS方式引导

### 联网

+ 如果是在虚拟机中，虚拟机设置使用`PAC`模式即可，如果使用`桥接`模式，按照接下来的方式进行操作
+ 如果是有线网，执行命令`dhcpcd`
+ 如果是无线网，可以使用`iwctl`来进行网络连接，具体用法请查询[这个链接](https://wiki.archlinux.org/index.php/Iwd_)
### 更新系统时间

```
timedatectl set-ntp true
```

### 分区和格式化

可以采用cfdisk或者fdisk来进行分区，注意如果电脑中本来就已经存在EFI分区的话，就不需要额外的引导分区了。

```
# 格式化EFI分区
mkfs.fat -F32 /dev/对应磁盘文件
# 格式化swap分区
mkswap /dev/swap分区文件
swapon /dev/swap分区文件
```

### 挂载分区

```
mount /dev/linux根目录 /mnt
mkdir /mnt/windows
mount /dev/windows所在磁盘分区  /mnt/windows
mkdir /mnt/boot
mount /dev/EFI分区 /mnt/boot
```

### 选择镜像源

```
vim /etc/pacman.conf
# 然后跳转编辑其中所引用的mirrowlist文件，将China对应的镜像源放到最前面，删除其他的镜像源
```

### 安装基本包

注意在启动Arch Linux前必须安装一些网络管理的相关包，否则可能会导致在启动系统时连不上网的情况

```
pacstrap /mnt base base-devel linux linux-firmware neovim networkmanager
```

### 配置fstab

```
genfstab -U /mnt > /mnt/etc/fstab
# 验证是否正确
cat /mnt/etc/fstab
```

### chroot

将操控权交给新系统，此时就已经进入你要安装arch linux的磁盘了，此时我们的操作都相当于新系统的root用户

```
arch-chroot /mnt
```

### 设置时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 设置locale

编辑`/etc/locale.gen`文件，移除`en_GB.UTF-8`前面的`#`，然后执行命令

```
locale-gen
```

接着创建并编辑`/etc/locale.conf`

```
LANG=en_GB-UTF-8
```
### 网络文件配置

编辑`/etc/hostname`文件

```
zds-pc
```

编辑`/etc/hosts`文件

```
127.0.0.1   localhost
::1         localhost
127.0.0.1   zds-pc.localdomain zds-pc
```

### 设置root密码，添加新用户

```
passwd
useradd -m -G wheel zds
passwd zds
pacman -S sudo
EDITOR=nvim visudo
```

接着找到如下内容取消注释

```
# %wheel ALL=(ALL)ALL
```

### 安装bootloader

```
pacman -S grub efibootmgr intel-ucode(amd-ucode) os-prober ntfs-3g
grub-install --target=x86_64-efi --efi-directory=/boot bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
# 检查引导是否安装成功，搜索menuentry
nvim /boot/grub/grub.cfg
# 检查内核是否正确部署
ls /boot
# 如果有initramfs-linux-fallback.img initramfs-linux.img intel-ucode.img vmlinuz-linux这几个文件，则说明没有问题
```

### 启动系统服务

```
systemctl disable netctl
systemctl enable NetworkManager
```

### 重启

重启时拔掉U盘

```
exit
umount /mnt/boot
umount /mnt/windows
umount /mnt
reboot
```

## 安装完成后

### 安装显卡驱动

请参考[这个网站](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)

### 安装xorg和桌面环境

```
sudo pacman -S xorg xorg-server
sudo pacman -S plasma kde-applications
sudo pacman -S sddm
systemctl enable sddm
reboot
```
