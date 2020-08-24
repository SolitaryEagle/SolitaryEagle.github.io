---
layout:     post
title:      CentOS Serve 安装教程
subtitle:   CentOS Serve 初始化
date:       2020-08-24
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Linux CentOS
---
##### 安装资源
* [CentOS 8](https://mirrors.aliyun.com/centos/8/isos/x86_64/CentOS-8.2.2004-x86_64-boot.iso)
* 语言：English
* Time Zone：Asia/Shanghai
* 安装模式：最小安装
* 网络：NAT，单网卡
* 设置 root 密码
##### 初始化配置
* 修改 yum repos 为 阿里云镜像
```shell
# 备份
yum install -y wget curl
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
yum makecache
```