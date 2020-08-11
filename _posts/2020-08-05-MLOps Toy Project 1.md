---
layout: post
title:  "MLOps - Toy Project #1"
description: Kubeflow를 통한 MLOps
date:   2020-08-05 00:00:00 +0900
categories: MLOps_Project
use_math: true
---

Kubeflow를 이용하여 MLOps를 실습하기 위한 프로젝트인 만큼 여러 상황을 가정하고, 제약을 두어서 진행하려고 한다. 모델은 최대한 학습하는데 많은 시간 및 자원을 소모하지 않는(하지만 흥미가 있는) 선에서 결정하였다. 그리고 Kafka, Jenkins 등 여러 오픈소스를 경험해 볼 것이다.

Toy Project
- 개요: 기존 이미지를 새로운 스타일의 이미지로 변형
- 사용 모델: GAN
- 인풋 및 아웃풋
    - 인풋: 이미지 n장
    - 아웃풋: 스타일이 변형된 이미지 n장
- 상황 가정 및 제약
    - Docker 및 Kubernetes, Kubeflow 사용
    - 지속적인 새로운 데이터 학습(Kafka)
    - 지속적인 성능 모니터링
    - 학습 및 배포 시 multi-GPU 사용
    - CI/CD/CT 사용(Jenkins)

우선 전체 파이프라인에 대해 설계를 해야한다. Google Cloud Platform의 [MLOps 포스팅](https://cloud.google.com/solutions/machine-learning/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) 에서는 3 단계의 ML 파이프라인을 소개한다. 이 중 Ci/CD/CT 의 자동화가 가능한 마지막 레벨의 파이프라인을 참고하여 약간 변형된? (거의 동일한) pipeline을 그려보았다.

![pipeline](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops1/pipeline.png){: width="100%"}

우선 6개의 서버를 사용 할 것이다.
- Data Server: feature 저장
- Development Server: 소스 코드 개발
- Training Server: 모델 학습(GPU 서버)
- Jenkins Server: CI/CD 수행
- Productino Server: 배포
- Kafka Server: 새로운 데이터 공급

물론 Toy Project로 진행 될 것이기 때문에, 6개의 서버를 모두 구축하기는 힘들고, 몇몇 서버는 통합하여 실습해 볼 것이다.

모든 pipeline은 docker로 구축을 할 것이며, 

.. TBA


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

