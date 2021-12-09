
---
title: WireGuard 服务建记录
date: 2019-08-11 09:12:57
categories:
    - linux
tags:
    - linux
    - WireGuard
    - VPN
---

# WireGuard 服务建记录

## 环境

```
系统:Debian 10
默认内核:linux-headers-4.19.0-14-amd64
```

## 准备工作

```shell
# 查看当前内核版本
uname -r
# 安装过程需要的软件包(已装跳过)
sudo apt update
sudo apt install apt-transport-https vim -y
# 搜索内核
sudo apt search linux-image
# 安装5.10
sudo apt-get install linux-image-5.10.0-0.bpo.9-amd64
# 卸载原来的内核
sudo apt-get remove linux-headers-4.19.0-14-amd64
# 重启
sudo reboot
```

## 安装WireGuard服务端

```shell
# 添加backports 源
sudo sh -c "echo 'deb https://deb.debian.org/debian buster-backports main contrib non-free' > /etc/apt/sources.list.d/buster-backports.list"

# 安装软件包
sudo apt update
sudo apt -t buster-backports install wireguard -y

# 配置服务端
cd /etc/wireguard
umask 077
## 生成配置需要的密钥和公钥
wg genkey | tee privatekey | wg pubkey > publickey
## 编辑服务端配置文件 ， 内容如下
sudo vim /etc/wireguard/wg0.conf
	
```

### 	服务端配置文件内容

```
[Interface]
# wireguard 内网服务器地址
Address = 192.168.5.1/32
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# wireguard 监听端口
ListenPort = 12345 
# wireguard 生成的服务端密钥
PrivateKey = KDGU6q5EG2Pv1ayJHIqAlmG2hGklF9fJ0ZGCp09GJXk=
```

### 防火墙配置

```shell
sudo ufw allow 12345/udp
```

### 服务启停

```shell
sudo systemctl enable wg-quick@wg0

sudo systemctl start wg-quick@wg0

sudo systemctl status wg-quick@wg0

sudo systemctl stop wg-quick@wg0
```

### 启动成功后查看

```shell
sudo wg

sudo ip a show wg0
```

## 客户端安装（Debian）

### 	wireguard安装

​		等同服务端软件包安装	

### 	客户端配置

```
[Interface]
PrivateKey = 客户端私钥
Address = 192.168.5.2/32
DNS = 192.168.5.1
 
[Peer]
PublicKey = 服务端公钥
AllowedIPs = 0.0.0.0/0
Endpoint = 服务端外网ip:服务端配置监听端口

```

### 	增加服务端配置

```
[Peer]
PublicKey = 客户端公钥
AllowedIPs = 192.168.5.2/32
```

## Enjoy～