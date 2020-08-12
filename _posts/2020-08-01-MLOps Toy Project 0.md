---
layout: post
title:  "MLOps - Toy Project #0"
description: Kubeflow를 통한 MLOps 버전 및 서버 정보
date:   2020-08-01 00:00:00 +0900
categories: MLOps_Project
use_math: true
---

클러스터 환경의 서버를 이용하여 모델 학습을 하기 위해 쿠버네티스(Kubernetes)와 쿠버플로우(Kubeflow)를 처음 접하게 되었다. 컴퓨터 네트워크라는 분야에 매우 약하기 때문에 공부하기가 쉽지 않았다. 하지만 찾아볼 수록 너무 매력적인 이야기들이고, MLOps라는 분야에 대해 큰 관심이 생겼다. 어떤 것이든지 직접 해봐야 지식으로 남는 편이라, MLOps의 pipeline을 구현해보는 Toy Project를 시작하고자 한다. kafka을 통해 데이터를 받고 학습을 진행하며 evaluation을 통해 roll out 되는 등의 과정을 설계해보며 블로깅 해볼 것이다.

kubernetes와 kubeflow 설치는 Reference의 지구별 여행자 블로그를 참고하였다. 설치하면서 생긴 에러들은 이전 [포스팅](https://byeongjokim.github.io/posts/Kubeflow-%EC%84%A4%EC%B9%98/)에 적어 놓았다. 아래는 사용하고 있는 쿠버네티스 서버 환경과 버전 정보이며, 프로젝트를 진행하면서 설치한 모든 것의 버전 정보를 이 곳에 추가해 나갈 것이다.

## Server
- Development + Jenkins + Kafka Server
    - n1-standard-1 (CentOS 7.8)
- Training + Production + Data Engineering Server
    - 1 master node (n1-standard-2, CentOS 7.8)
    - 1 worker node (n1-standard-2, CentOS 7.8)
- Data (Feature Store, ML Metadata Store, Trained Model Weight) Server
    - NFS

## kubernetes & kubeflow 
- Kubernetes: kubeadm v1.18.6
- CNI: Calico
- Kubeflow: kfctl_k8s_istio_v1.0.0.yaml
- istio: v1.4.2

## References
- [Kubernetes](https://kubernetes.io/docs/home/)
- [Kubeflow](https://www.kubeflow.org/docs/)
- 쿠버네티스 네트워크 개념
    - [IBM Developer 밋업 쿠버네티스 차근차근 다지기](https://www.youtube.com/watch?v=l42GttmnnZ4)
    - [쿠버네티스 연구회](https://www.youtube.com/watch?v=q1k_iOB3yig)
    - [커피고래의 노트](https://coffeewhale.com/)
    - [어쩐지 오늘은](https://zzsza.github.io/category/mlops/)
- 쿠버네티스 설치
    - [지구별 여행자](https://www.kangwoo.kr/2020/02/17/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-1%EB%B6%80-nvidia-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)