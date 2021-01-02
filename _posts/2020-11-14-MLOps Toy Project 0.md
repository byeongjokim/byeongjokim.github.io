---
layout: post
title:  "거대한 데이터인 척 하는 Mnist!, MLOps 구축하기 #0"
description: 프로젝트 소개 및 정보
date:   2020-11-14 00:00:00 +0900
categories: MLOps
use_math: true
---

- [프로젝트 소개 및 정보](https://byeongjokim.github.io/posts/MLOps-Toy-Project-0/)
- [MLOps System Design](https://byeongjokim.github.io/posts/MLOps-Toy-Project-1/)
- [CI/CD using Github Action](https://byeongjokim.github.io/posts/MLOps-Toy-Project-2/)

클러스터 환경의 서버를 이용한 모델 학습을 하기 위해 쿠버네티스(Kubernetes)와 쿠버플로우(Kubeflow)를 처음 접하게 되었다. 컴퓨터 네트워크라는 분야에 매우 약하기 때문에 공부하기가 쉽지 않았다. 하지만 찾아볼 수록 너무 매력적인 이야기들이고, MLOps라는 분야에 대해 큰 관심이 생겼다. 그러던 와중 회사일로 인해 CI/CD를 경험해보며 MLOps에 더욱 빠져들었다. 어떤 것이든지 직접 해봐야 공부가 되는 성격이라 local 부터 배포까지 전 MLOps 과정을 구축해보았다. 

## Project
서버 사양이나 속도면을 고려했을 때, 딥러닝에서 "Hello World"로 알려진 **Mnist** 데이터셋을 사용한 프로젝트를 기획하였다. 하지만 절대 간단하지 않도록 **여러 상황을 가정하고, 제약을 두어서** 진행하려고 한다. 예를들어 Mnist 데이터 셋의 경우 학습 이미지 전체를 램에 올려서 학습이 가능하지만, 타 데이터셋은 그렇지 못하는 경우가 많다. 또한 Mnist의 경우 image 데이터 이지만, table 데이터를 이용한 프로젝트가 있을 수 있다. 또 얼굴 인식의 경우 closed-set과 open-set의 차이로 인한 성능 저하도 생길 수 있다. 이들을 고려해서 MLOps 구축을 진행하였다. 한줄 요약을 하자면...

> 거대한 데이터인 척 하는 Mnist!, MLOps 구축하기

아래는 사용하고 있는 서버 환경과 프로그램 정보이며, 평소에 관심이 있는 오픈소스를 위주로(e.x., Github Action) 사용하였다.

## Server
- Minikube v1.15.1 ([How to Install](https://www.kubeflow.org/docs/started/workstation/minikube-linux/#install-minikube))
- Kubeflow v1.0.0 ([How to Install](https://byeongjokim.github.io/posts/install-kubeflow/))

## CI/CD/CT
- Github (Gtihub Action)
- Docker
- Slack
- Kubeflow
    - pipelines (kfp)
    - Katlib
    - KFServing
- TorchServe

## References
- [Kubernetes](https://kubernetes.io/docs/home/)
- [Kubeflow](https://www.kubeflow.org/docs/)
- kubernetes
    - [커피고래의 노트](https://coffeewhale.com/)
    - [어쩐지 오늘은](https://zzsza.github.io/category/mlops/)
    - [핵심만 콕! 쿠버네티스](http://www.yes24.com/Product/Goods/92426926)
- kubeflow
    - [쿠브플로우!](http://www.yes24.com/Product/Goods/89494414)