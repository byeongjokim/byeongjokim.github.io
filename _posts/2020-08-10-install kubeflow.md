---
layout: post
title:  "CentOs에 Kubernetes & Kubeflow 설치 명령어 모음"
description: Kubernetes & Kubeflow 설치
date:   2020-08-10 00:00:00 +0900
categories: MLOps
use_math: true
---

Google Cloud Engine VM 인스턴스 CentOS 7.8에 kubernetes with Calico와 kubeflow을 설치하는 스크립트만 모아놓았습니다.

## 서버 환경 세팅 (Master, Worker node)
```
$ sudo setenforce 0
$ sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
$ sudo modprobe br_netfilter
$ sudo echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
$ sudo modprobe br_netfilter
$ sudo swapoff -a
```

## Docker 설치 (Master, Worker node)
```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install -y docker-ce
```

## Kubernetese 설치 (Master, Worker node)
```
$ sudo vi /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```
```
$ sudo yum install -y kubelet kubeadm kubectl
$ sudo systemctl start docker && systemctl enable docker
$ sudo systemctl start kubelet && systemctl enable kubelet 
```
```
$ sudo vi /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```
```
$ sysctl --system
$ sudo reboot
```

### Master Node
```
# CNI로 [Calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart) 사용
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config 
$ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
$ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

### Worker Node
```
# 마스터 노드에서 sudo kubeadm init --pod-network-cidr=192.168.0.0/16 명령어 실행 후 출력된 명령어 실행
$ kubeadm join x.x.x.x:x --token 7r5792.5xuktr48txdrwnbj --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Kubeflow 설치 (v1.0.0)
```
$ kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
$ kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
$ mkdir kubeflow
$ cd kubeflow
$ curl -L -O https://github.com/kubeflow/kfctl/releases/download/v1.0/kfctl_v1.0-0-g94c35cf_linux.tar.gz
$ tar -xvf kfctl_v1.0-0-g94c35cf_linux.tar.gz
$ mkdir yaml
$ cd yaml
$ export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.0.yaml"
$ kfctl apply -V -f ${CONFIG_URI}
```

## References
- [Kubernetes](https://kubernetes.io/docs/home/)
- [Kubeflow](https://www.kubeflow.org/docs/)
- 쿠버네티스 설치
    - [지구별 여행자](https://www.kangwoo.kr/2020/02/17/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-1%EB%B6%80-nvidia-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)