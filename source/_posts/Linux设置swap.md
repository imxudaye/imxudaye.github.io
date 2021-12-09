---
title: Linux设置swap
date: 2018-05-26 14:12:57
categories:
    - linux
tags:
    - linux
    - swap
---

# Linux设置swap

## 1、创建一个 文件作为swap区

```shell
# 名字为/swapfile1  大小为bs*count = 1024*2000000=2G，count代表的是大小，这里是2G
dd if=/dev/zero of=/swapfile1 bs=1024 count=2000000

```

## 2、将其转化为swap文件

```shell
mkswap /swapfile1
```

## 3、将其改为只有root权限才能修改

```shell
# chown root:root /swapfile1
# chmod 0600 /swapfile1
```

## 4、将其激活

```shell
swapon /swapfile1
```

## 5、如果想要系统重启后生效，可以打开/etc/fstab在最后面加上一行

```shell
nano /etc/fstab

/swapfile1 swap swap defaults 0 0
```

