---
date: '2025-10-01T21:40:06+08:00'
draft: false
title: '家庭内网安全访问'
tags: ['内网', 'shadowsocks', 'Cloudflare']
categories: ["内网"]
description: "打造家庭内网的安全访问"
---
## 目标
- 能通过远程连接(桌面)家庭局域网中的主机， 有mac以及windows系统的主机。
- 局域网内主机安装了很多的应用服务，如数据库，文件下载站，Nas云等需要在外部访问。 
- 不能通过端口映射的方式暴露各端口到公网。 

## 硬件&软件： 
- 一台刷了ImmortalWrt的Newifi D1路由器。 作为主路由，端口转发， DHCP分配内网IP.
- 一台星际蜗牛黑群晖NAS,版本 DS3617xs，Docker,安装 shadowsocks-libev ，用于创建ss服务，nginx 反向代理内网转发,DDNS 服务。
- 准备一个域名，托管到cloudflare，用于DNS解析。
- 客户端Clash，用于访问ss服务，设置代理组，url节点等，保证访问内网的安全性。

## 步骤
### 1. 安装路由器系统
- 刷机，从 [ImmortalWrt地址](https://firmware-selector.immortalwrt.org/) 下载符合自己硬件的固件，并刷入。
![固件下载](/images/post/lan/immortalWrt.png)
- 如果是第一次刷机，请先将路由器恢复出厂设置。先安装Kernel镜像， 安装进入系统后，再安装Sysupgrade镜像。
### 2. 设置路由器
- newifi D1默认的存储空间比较小，需要设置挂载点。插入16G的SD卡，分配1G给/overlay，用于软件包安装等系统空间。另外的分配/workspace 用于平时使用， 比如file browser等。
- 第一步操作就是去操作路由器设置， 否则其他设置做完再去设置挂载点， 设置都会无效。 
设置步骤可参考[挂载点SD卡扩容](https://doc.embedfire.com/openwrt/user_manal/zh/latest/User_Manual/openwrt/storage.html)
最终效果如下:![挂载点](/images/post/lan/mounts.png)
- 设置端口转发，将外网端口转发到内网设备。