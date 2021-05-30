---
title: Arch Linux安装完之后的工作
date: 2021-01-04 12:53:58
tags: ["linux"]
draft: true
---

## 连接网络

首先需要开启`NetworkManager`服务，再采用`nmtui`连接网络

```
systemctl start NetworkManager
systemctl enable NetworkManager
nmcli dev wifi list
nmtui
```

## 创建交换文件

如果在分区时没有创建`swap`分区，可以在这一步创建交换文件，注意以下命令都以`root`用户执行

```
dd if=/dev/zero of=/swapfile bs=1M count=2048 status="progress"
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

接着编辑`/etc/fstab`文件，在末尾添加如下内容

```
/swapfile none swap defaults 0 0
```

## 添加archlinuxcn源

`archlinuxcn`是一个由国人管理的package repository，其中包含一些日常所需的软件源，添加如下内容到`/etc/pacman.conf`文件中

## 代理的设置

+ chromium的代理设置

首先从terminal中启动chromium

```
export SOCKS_VERSION=5
export SOCKS_SERVER='127.0.0.1:1080'
chromium
```

等完成google帐号的登陆以及同步后从dotfiles中恢复SwithyOmega的配置，就可以科学上网啦。

+ 终端程序的代理

安装`proxychains-ng`，然后编辑'/etc/proxychains.conf'进而设置代理。

## 图形界面的安装

首先安装X服务器

```
sudo pacman -S xorg xorg-server
```

启动图形界面有两种方式，一种是安装`xorg-init`通过`xinit`启动，另一种方式是通过`display manager`来选择`xsession`来进行启动。

+ 以DM的形式进行启动

如果是通过display manager的方式来进行图形界面的启动的话，可以选择安装sddm或者lightDM，并在系统服务中开启它们，重启后即可选择相应的`session`进而启动图形界面。

```
sudo pacman -S sddm
sudo systemctl enable sddm
```

+ 以xinit的形式启动

首先安装`xorg-init`，接着编辑`.xinitrc`的内容，最后执行`startx`来根据`.xinitrc`中的内容来启动图形界面。

```
sudo pacman -S xorg-init
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

## 用i3wm来定制你自己的desktop enviroment

界面展示如下：

![demo](/images/demo.png)

### wm and status bar

```
sudo pacman -S i3-gaps i3status-rust
```

### applets

安装蓝牙模块

```
sudo pacman -S blueman
```

### program launcher

使用rofi来启动应用程序

```
sudo pacman -S rofi papirus-icon-theme
```

### notification manager

使用dunst来管理应用通知

```
sudo pacman -S dunst
```

### compositor

使用archlinuxcn源中的`compton-tryone-git`来进行窗口渲染

```
sudo pacman -S compton-tryone-git
```

### screen lock

使用`beautifullockscreen`来进行熄屏操作

```
pacman -S i3lock-color imagemagick feh xorg-xrandr xorg-xdpyinfo
yay -S betterlockscreen
```

### tools

```
flameshot -- screenshot
pcmanfm -- gui file manager
libreoffice-still-zh-cn -- office
font-manager -- font
```

### theme config

我们需要对gtk应用和qt应用分别进行主题配置，以获得一致的显示效果

```
sudo pacman -S lxappreance # gtk应用的gui配置工具
sudo pacman -S qt5ct # qt应用的gui配置工具
```

同时添加如下环境变量

```
QT_QPA_PLATFORMTHEME=qt5ct
```

## 输入法

输入法应该安装最新的`fcitx5`，安装以下包

```
sudo pacman -S fcitx5-im             # fcitx应用组
sudo pacman -S fcitx5-chinese-addons # 中文输入法支持
sudo pacman -S fcitx5-pinyin-zhwiki fcitx5-pinyin-moegirl # 中文字库
sudo pacman -S fcitx5-material-color # 主题样式
```

如果使用`xinit`启动，则在`.xinitrc`中添加如下环境变量，如果使用`DM`启动，在`.xprofile`中添加

```
export INPUT_METHOD=fcitx5
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS="@im=fcitx5"
```

设置完成后重新登录即可。

## 字体配置

首先安装如下字体

```
sudo pacman -S ttf-jetbrains-mono nerd-fonts-jetbrains-mono ttf-symbola # st所需字体
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji # noto字体系列
```

再导入windows及自己收集的字体

```
mkdir /usr/share/fonts/myfonts
mkfontscale
mkfontdir
fc-cache
```
