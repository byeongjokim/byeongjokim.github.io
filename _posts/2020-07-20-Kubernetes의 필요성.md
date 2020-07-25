---
layout: post
title:  "MLOps - Kubernetes의 필요성"
description: 쿠버네티스 네트워킹
date:   2020-07-20 20:03:00 +0900
categories: Kubernetes
use_math: true
---

> 클러스터 환경의 서버를 이용하여 모델 학습을 하기 위해 쿠버네티스(Kubernetes)와 쿠버플로우(Kubeflow)를 처음 접하게 되었다. 컴퓨터 네트워크라는 분야에 매우 약하기 때문에 공부하기가 쉽지 않았다. 공부하면서 접한 자료들과 지식들을 정리하기 위해 포스팅을 시작하였다. Kubernetes 정리 라는 포스팅에 쿠버네티스 기본 object들과 네트워크 등 부터 시작해서 Kubeflow로 MLOps를 진행하는 것 까지의 내용을 차근차근 담을 예정이다. 얼마나 오래 걸릴지 모르겠지만, 새로 접하는 분야이기 때문에 재미와 욕심이 생겨서 꾸준히 포스팅 하려고 한다. 밑의 Reference에는 가장 도움이 많이 된 사이트 및 유튜브 채널의 링크를 올려 놓았다.

## Kubernetes
쿠버네티스를 이해하기 위해서는 마이크로서비스 아키텍쳐(MSA)와 컨테이너에 대한 지식이 필요하다. 하지만 그 둘에 대한 설명은 최소한으로 하고 쿠버네티스에 대한 내용만을 적으려고 한다.

우선 쿠버네티스가 필요한 이유를 설명하기 위해 마이크로서비스 아키텍쳐(MSA)와 컨테이너에 대한 간단하게 적어본다면..

마이크로서비스 아키텍쳐(MSA)
- 소프트웨어 응용 프로그램을 독립적으로 배포 가능한 서비스의 묶은 형태로 설계하는 방식. 즉 가능한 쪼개는 것이다.
- 쪼개진 것들의 버전 문제, scaling 문제, 관리 문제 등등을 해결하기 위해 컨테이너를 사용.

컨테이너
- S/W 실행에 필요한 것을 패키지로 구성하여 표준화된 하나의 독립 컨테이너에 저장
- 개발 서버에서는 잘되던 것이 배포 서버에서는 버전 문제 등에 의해 에러..


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
    - 

