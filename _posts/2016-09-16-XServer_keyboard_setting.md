---
layout: post
title:  "XServer中的键盘设置问题"
date:   2016-09-16 12:00:00
categories: system
excerpt:
---

* content
{:toc}

** 一

首先是键盘映射问题，在系统设置中交换CapsLk与LeftCtrl经常遇到重启后设置失效的问题，今天检索之后，找到了强制设置的方法：

创建`/etc/X11/xorg.conf.d/90-custom-kbd.conf`文件，内容如下：

```
Section "InputClass"
    Identifier "keyboard defaults"
    MatchIsKeyboard "on"

    Option "XKbOptions" "ctrl:swapcaps"
EndSection
```

[Link](https://wiki.archlinux.org/index.php/Keyboard_configuration_in_Xorg#Frequently_used_XKB_options)

** 二

fcitx输入法问题，在`~/.xprofile`中添加以下内容：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

[Link](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E9.9D.9E.E6.A1.8C.E9.9D.A2.E7.8E.AF.E5.A2.83)
