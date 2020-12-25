---
layout:     post
title:      使用 kubeadm 安装 Kubernetes v1.18
subtitle:   Kubernetes
date:       2020-08-30
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Kubernetes
---

cluster 节点上需要安装
* CRI（容器运行时接口）
* kubeadm 
* kubelet
* kubectl
* CNI（容器网络接口）
  

node 节点上需要安装


# 准备

* [CentOS-7-x86_64-Minimal-2009.iso 镜像下载](https://mirrors.aliyun.com/centos/7/isos/x86_64/)
* VMWare 虚拟机安装 CentOS 7
    * 内存: 2GB 
    * CPU: 2核，开启 Virtualize CPU performance counters
    * 磁盘: 120GB
    * 网络: NAT
    * 语言: 中文 简体中文(中国)
    * 网络和主机名: 按需设置
    * ROOT 密码: 按需设置
    * 重启
    * 切换 yum repos 为 阿里云镜像
      ```shell
      yum install -y wget curl net-tools vim
      mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
      curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
      yum makecache
      yum update
      ```    
    * 关闭防火墙
      ```shell
      systemctl stop firewalld.service
      systemctl disable firewalld.service
      ```
    * [关闭交换分区](https://blog.csdn.net/yefun/article/details/102772368)
      ```shell
      # 注释/etc/fstab关于swap的配置
      echo vm.swappiness=0 >> /etc/sysctl.conf
      ```
    * 重启
* [安装 kubeadm](https://v1-18.docs.kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)      
    * 检查 Mac 地址: ifconfig 
    * 检查 product_uuid: cat /sys/class/dmi/id/product_uuid
    * [安装 runtime（选择 containerd）](https://v1-18.docs.kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#containerd)
      ```shell
      cat > /etc/modules-load.d/containerd.conf <<EOF
      overlay
      br_netfilter
      EOF
        
      modprobe overlay
      modprobe br_netfilter
        
      # 设置必需的sysctl参数，这些参数在重新启动后仍然存在。
      cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
      net.bridge.bridge-nf-call-iptables  = 1
      net.ipv4.ip_forward                 = 1
      net.bridge.bridge-nf-call-ip6tables = 1
      EOF
      sysctl --system
      
      # 安装 containerd
      ## 设置仓库
      ### 安装所需包
      yum install -y yum-utils device-mapper-persistent-data lvm2
      ### 新增 Docker 仓库
      yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
      ## 安装 containerd
      yum update && yum install containerd.io
      # 配置 containerd
      mkdir -p /etc/containerd
      containerd config default > /etc/containerd/config.toml
      # 在 /etc/containerd/config.toml 中设置 plugins.cri.systemd_cgroup = true
      # 重启 containerd
      systemctl restart containerd
      ```
    * 安装 kubeadm、kubelet 和 kubectl
      ```shell
      # 确保在此步骤之前已加载了 br_netfilter 模块 (加载方式 modprobe br_netfilter)
      lsmod | grep br_netfilter
      
      # 谷歌 k8s 库
      cat <<EOF > /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
      enabled=1
      gpgcheck=1
      repo_gpgcheck=1
      gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
      EOF
      
      # 阿里云 k8s 库
      cat <<EOF > /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
      enabled=1
      gpgcheck=0
      repo_gpgcheck=0
      gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
      EOF
    
      # 将 SELinux 设置为 permissive 模式（相当于将其禁用）
      setenforce 0
      sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
      yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0 --disableexcludes=kubernetes
      systemctl enable --now kubelet
      ```
    * 在控制平面节点上配置 kubelet 使用的 cgroup 驱动程序



