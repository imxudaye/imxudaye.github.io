
---
title: 自建windows激活服务器
date: 2018-09-26 23:12:56
categories:
    - Windows
tags:
    - Windows
    - KMS
---

### 自建windows激活服务器

#### 前言

> Windows系统中能够通过KMS进行激活的一般称为VL版,即VOLUME授权版。我们可以自行搭建KMS激活服务器，实现每180天一次的自动激活，使得系统一直保持激活状态。这次就跟大家分享一下如何利用Docker搭建KMS服务器，并演示如何利用我们自己的KMS服务器激活Windows操作系统与Microsoft Office。

#### KMS搭建操作步骤

1. 拉取镜像

   ```shell
   docker pull luodaoyi/kms-server
   ```

2. 创建启动容器

   ```shell
   docker run -itd -p 1688:1688 --name kms luodaoyi/kms-server
   ```

3. 防火墙开放端口

   ```shell
   firewall-cmd --zone=public --add-port=1688/tcp --permanent
   ```

#### 激活步骤

##### 	--激活windows

1. 先确认系统版本，`设置-系统-关于`  或者命令窗口执行 `wmic os get caption`
   
2. 打开 **https://url.zeruns.tech/kms_key**  ，查找获取对应版本的key， 例如：**NRG8B-VKK3Q-CXVCJ-9G2XF-6Q84J**
   
   

3. 管理员身份打开命令窗口，依次执行如下命令

   ```shell
   # ip:port为你部署的KMS服务地址 ， 例如123.123.123.123:1688
   slmgr /skms ip:port 
   
   # key为对应版本的key
   slmgr /ipk NRG8B-VKK3Q-CXVCJ-9G2XF-6Q84J
   
   #自动激活操作
   slmgr /ato
   ```

   

#####  --激活Microsoft Office

首先先确认下我们的Office是否为VOL版，方法如下：

​	**管理员身份运行命令提示符，输入 `cd C:\Program Files\Microsoft Office\Office16` 切换目录 （这里请根据您自己的Office版本更改相应路径），再输入 `cscript ospp.vbs /dstatus` 。可以看到这里有VL字样即为VOL版**

开始激活：

```shell
#这是默认安装目录，如果有自定义目录请酌情调整
cd C:\Program Files\Microsoft Office\Office16

cscript ospp.vbs /sethst:nas.zeruns.tech

cscript ospp.vbs /act
#最后看到Product activation successful字样即为激活成功
```

