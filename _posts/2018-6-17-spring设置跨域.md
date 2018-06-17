---
layout: post
title: ubuntu忘记root密码
description: ubuntu忘记root密码
category: linux
---

## 1.进入Recovery Mode(恢复模式)
开机，在Grub启动菜单选择`Advance Options`，如果没有Grub启动菜单出现，开机时按方向键即可出现。

## 2.进入root用户
进入恢复菜单选择`root`

## 3.执行如下命令更改root密码
```shell
mount -o rw,remount /
passwd root
```