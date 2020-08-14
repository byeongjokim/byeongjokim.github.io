---
layout: post
title:  "MLOps - Toy Project #2"
description: Dev Server 환경 세팅 및 테스트
date:   2020-08-14 00:00:00 +0900
categories: MLOps_Project
use_math: true
---

Dev 서버는 총 3가지를 위해 사용된다.

- Development + Jenkins + Kafka Server
    - 소스 코드 개발
    - CI/CD 수행
    - 신규 학습 데이터 전송

Jenkins와 Kafka는 Docker Image 로 실행을 할 것이며, 소스 코드는 Github와 Docker hub push 를 통해 CI/CD가 이루어 질것이다. 따라서 Docker CE를 우선 설치하였다.

Docker 설치는 [Post](https://byeongjokim.github.io/posts/install-kubeflow/)를 참고하면 된다.

```
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install docker-ce
$ sudo systemctl start docker && systemctl enable docker
```

# TODO

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
- MLOps
    - [MLOps CI/CD/CT Pipelines](https://cloud.google.com/solutions/machine-learning/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
    - [Blog 1](https://growingdata.com.au/mlops-ci-cd-for-machine-learning-pipelines-model-deployment-with-kubeflow/?preview=true&_thumbnail_id=5121)

