---
layout:     post
title:      使用 kubeadm 安装 Kubernetes v1.18
subtitle:   Kubernetes
date:       2020-12-25
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
* CRI（容器运行时接口）
* kubeadm
* kubelet


# 基础设施准备

* [CentOS-7-x86_64-Minimal-2009.iso 镜像下载](https://mirrors.aliyun.com/centos/7/isos/x86_64/)
* VMWare 虚拟机安装 CentOS 7 ==》k8s-master, k8s-node01, k8s-node02
    * 内存: 2GB 
    * CPU: 2核，开启 Virtualize CPU performance counters
    * 磁盘: 120GB
    * 网络: 公网 Bridge，内网 NAT
    * 语言: English United States
    * 时区: Asia/Shanghai
    * 网络和主机名: 按需设置
    * ROOT 密码: 按需设置
    * 重启

# 共同基础环境准备（所有节点都要安装）

* 基础环境准备
  ```text
  # 切换 yum repos 为 阿里云镜像
  yum install -y wget curl net-tools vim bash-completion
  mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
  wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
  yum makecache
  yum update

  # 关闭防火墙
  systemctl stop firewalld.service
  systemctl disable firewalld.service

  # 关闭交换分区 ==>  https://blog.csdn.net/yefun/article/details/102772368
  # 注释/etc/fstab关于swap的配置
  echo vm.swappiness=0 >> /etc/sysctl.conf

  # 设置 hosts
  192.168.31.89  k8s-master
  192.168.31.129 k8s-node01
  192.168.31.85  k8s-node02
  
  # 重启
  reboot
  ```  

* [安装 kubeadm](https://v1-18.docs.kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)      
    * 检查 Mac 地址: ifconfig 
    * 检查 product_uuid: cat /sys/class/dmi/id/product_uuid
    * [安装 runtime（选择 docker; 只安装 containerd 在拉取镜像时会失败）](https://v1-18.docs.kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#containerd)
      ```text
      # 安装 docker 前的环境准备
      cat > /etc/modules-load.d/containerd.conf <<EOF
      overlay
      br_netfilter
      EOF
      
      # 显式加载 br_netfilter 模块
      modprobe overlay
      modprobe br_netfilter
      
      # 设置必需的sysctl参数，这些参数在重新启动后仍然存在。
      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
      
      cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
      net.bridge.bridge-nf-call-iptables  = 1
      net.ipv4.ip_forward                 = 1
      net.bridge.bridge-nf-call-ip6tables = 1
      EOF
      sysctl --system
      
      # 安装 docker 引擎 ==> https://docs.docker.com/engine/install/centos/
      # 安装所需包
      yum install -y yum-utils device-mapper-persistent-data lvm2
      # 新增阿里云镜像 Docker 仓库
      yum-config-manager \
        --add-repo \
        https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
      
      # 查看 docker-ce 的版本
      yum list docker-ce --showduplicates | sort -r
      
      # 查看 containerd.io 版本
      yum list containerd.io --showduplicates | sort -r
      
      # 安装 docker 19.03.4-3.el7 和 containerd.io 1.3.9-3.1.el7
      yum install docker-ce-19.03.4-3.el7 docker-ce-cli-19.03.4-3.el7 containerd.io-1.3.9-3.1.el7
      # 配置 docker
      mkdir /etc/docker
      cat > /etc/docker/daemon.json <<EOF
      {
        "registry-mirrors": ["https://gm87dz12.mirror.aliyuncs.com"],
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
        "max-size": "100m"
      },
        "storage-driver": "overlay2",
        "storage-opts": [
          "overlay2.override_kernel_check=true"
        ]
      }
      EOF
      mkdir -p /etc/systemd/system/docker.service.d
      systemctl daemon-reload
      systemctl restart docker
      systemctl enable docker
      
      # 安装 kubeadm、kubelet 和 kubectl
      ## 确保在此步骤之前已加载了 br_netfilter 模块 (加载方式 modprobe br_netfilter)
      ## lsmod | grep br_netfilter
      
      # 新增阿里云 k8s 库
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
      
      # 查看 kubelet 的版本
      yum list kubelet --showduplicates | sort -r
      # 查看 kubeadm 的版本
      yum list kubeadm --showduplicates | sort -r
      # 查看 kubectl 的版本
      yum list kubectl --showduplicates | sort -r
      
      # 安装 1.18.14-0 版本
      yum install -y \
        kubelet-1.18.14-0 \
        kubeadm-1.18.14-0 \
        kubectl-1.18.14-0 \
        --disableexcludes=kubernetes
      systemctl enable --now kubelet
      # 配置 kubectl 自动补全
      echo "source <(kubectl completion bash)" >> ~/.bashrc
      
      # 在 master 节点上配置 kubelet 使用的 cgroup 驱动程序
      ## 修改 /etc/sysconfig/kubelet 
      ## KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
      ## 需要重新启动 kubelet：
      systemctl daemon-reload
      systemctl restart kubelet
      ```

# Master 节点需要的环境

```text
# 生成初始化文件
mkdir k8s
kubeadm config print init-defaults > k8s/kubeadm-init.yaml

# 修改 k8s/kubeadm-init.yaml
* advertiseAddress ==> 本机地址
* kubernetesVersion  ==>  v1.18.14
* imageRepository ==> registry.aliyuncs.com/google_containers (阿里云镜像仓库)
* serviceSubnet  ==>  192.168.0.0/16

# 下载镜像
kubeadm config images pull --config k8s/kubeadm-init.yaml

# 初始化 k8s master
kubeadm init --config k8s/kubeadm-init.yaml
# 将 kubeadm join 保存到 kubeadm-join.sh 中
cat > ~/k8s/kubeadm-join.sh <<EOF
#!/bin/bash
kubeadm join 192.168.220.128:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:3efda1d94742d2a7c9887ec4b389f557fab8665271f03be373ea3a3b9ff3de7b
EOF
# 配置环境, 让当前用户可以执行kubectl命令
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 安装 CNI（容器网络接口）选择 Calico ==>  https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises
mkdir -p "k8s/calico"
wget -P k8s/calico/ https://docs.projectcalico.org/manifests/calico.yaml
# 将之前设置在 k8s/kubeadm-init.yaml 中的 serviceSubnet 设置到 k8s/calico/calico.yaml 中的 192.168.0.0/16
kubectl apply -f k8s/calico/calico.yaml
```

# Node 节点需要的环境
      
* 安装 node 节点
    * 在 node 节点上执行之前保存在 master 节点上的 ~/k8s/kubeadm-join.sh


