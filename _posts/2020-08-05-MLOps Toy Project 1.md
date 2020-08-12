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

우선 전체 파이프라인에 대해 설계를 해야한다. Google Cloud Platform의 [MLOps CI/CD/CT Pipelines](https://cloud.google.com/solutions/machine-learning/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) 에서는 3 단계의 ML 파이프라인을 소개한다. 이 중 **Ci/CD/CT 의 자동화**가 가능한 마지막 레벨의 파이프라인을 참고하여 약간 변형된? (거의 동일한) pipeline을 그려보았다.

![pipeline](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops1/pipeline.png){: width="100%"}

ML pipeline 에는 필수적으로 구성되어야하는 요소들이 있다. [URL](https://growingdata.com.au/mlops-ci-cd-for-machine-learning-pipelines-model-deployment-with-kubeflow/?preview=true&_thumbnail_id=5121)
- Prepare & Integrate Data
- Feature Engineering
- Model Training & Model Evaluation
- Hyper Parameter Tuning
- Model Store

### Prepare & Integrate Data
training 와 inference 할 때 data의 구조 혹은 양식은 동일해야 하기 때문에 동일한 Data Analysis pipeline을 사용한다. 이때 data의 유효성을 검사하는 **Data Validation**과, 모델 학습에 맞게 변형 시키는 **Data Preparation**이 필수 적이다.

### Feature Engineering
ML은 data의 feature을 주로 사용하기 때문에 이를 저장하는 **Feature Store**를 사용한다. Feature Store는 학습 및 서비스를 위해 저장 및 엑세스가 가능한 중앙 저장소이다. 이는 feature을 재사용하고, 최신 feature들로 update 될 수 있어야한다. 또한 Kafka로 전달된 새로운 데이터는 **Feature Extraction**을 통해 Feature Store에 저장된다.

### Model Training & Model Evaluation
ML Model은 학습으로만 끝나는 것이 아니라, **Evaluation**, **Validation**, 그리고 **Analysis** 과정을 통해 더 좋은 Model을 Fitting 해야한다.

### Hyper Parameter Tuning
ML Model을 학습할 때 여러 Hyper Parameter들이 사용된다. 그리고 이것들에 따라 ML Model의 성능이 좌우된다. 따라서, 각 모델의 Hyper Parameter을 **ML Metatdata Store** 라는 저장소에 저장을 하여, 모델에 따른 Hyper Parameter version 관리 및 엑세스가 가능해야 한다.

### Model Store
ML Model이 학습 된 후 inference 되기 위해서는 Weight 파일이 저장되어야 한다. 이는 Github, File Cloud 서비스, 혹은 서버를 통해 version 관리가 가능하다. 이번 프로젝트에는 weight 파일은 Google Drive에, 버전 정보는 Github에 관리를 할 예정이다.

각 pipeline의 요소들을 위해 6개의 서버를 사용 할 것이다.
- Data Server: feature 저장
- Development Server: 소스 코드 개발
- Training Server: 모델 학습(GPU 서버)
- Jenkins Server: CI/CD 수행
- Productino Server: 배포
- Kafka Server: 새로운 데이터 공급

물론 Toy Project로 진행 될 것이기 때문에, 6개의 서버를 모두 구축하기는 힘들고, 몇몇 요소들은 docker화 시켜 서버를 통합하여 실습해 볼 것이다.

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

