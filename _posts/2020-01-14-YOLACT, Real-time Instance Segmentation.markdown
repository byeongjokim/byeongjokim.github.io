---
layout: post
title:  "YOLACT: Real-time Instance Segmentation"
description: Instance segmentation 문제를 real-time으로 해결할 수 없을까? 라는 의문으로 시작이 된다. 여태까지의 instance segmentation 모델은 잘 만들어진 object detection에 병렬적으로 모델을 추가하여 (e.g., mask R-CNN(Faster R-CNN), FCIS(R-FCN) 발전하였다. 하지만,
date:   2020-01-14 16:35:36 +0900
categories: Paper
---
ICCV 2019에 발표된 [논문](https://arxiv.org/pdf/1904.02689.pdf)이며, Official Code는 [pyTorch](https://github.com/dbolya/yolact)로 구현되어있다.

## Task: instance segmentation
Image Segmentation은 이미지를 픽셀 단위의 다양한 segments로 분할하는 task이다. 쉽게 말하자면, **이미지의 모든 픽셀에 라벨을 할당하는** task이다. 
Segmentation에는 두 가지 세부문제가 있다. 동일한 클래스에 해당하는 픽셀을 같은 색으로 칠하는 **Semantic Segmentation**. 동일한 클래스여도 다른 사물의 픽셀이면 다른 색으로 칠하는 **Instance Segmentation**. 아래 그림을 보면 확연한 차이를 알 수 있다.
![segmentation 차이](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/YOLACT/segmentation.png)

이 포스팅에서 소개하려는 논문 YOLACT는 instance segmentation 문제를 real-time으로 푸는 모델이다.

## Introduction
> Boxes are stupid anyway though, I’m probably a true believer in masks except I can’t get YOLO to learn them - Joseph Redmon, YOLOv3

**Instance segmentation 문제를 real-time으로 해결할 수 없을까?** 라는 의문으로 시작이 된다. 여태까지의 instance segmentation 모델은 잘 만들어진 object detection에 병렬적으로 모델을 추가하여 (e.g., mask R-CNN(Faster R-CNN), FCIS(R-FCN) 발전하였다. instance segmentation은 매우 어려운 task이여서, object detection의 SSD 그리고 YOLO 모델과 같이 one-stage로 모델을 짜기 힘들기 때문이다. 위의 two-stage 모델은 mask를 생성하기 위해 **feature localization**에 많은 신경을 썼다 (e.g., RoI align). 하지만 feature localization 후 mask를 예측하는 모델은 순차적으로 이루어질 수 밖에 없고, 스피드를 올릴 수(accelerate)가 없어진다. FCIS는 이를 병렬적으로 수행하였지만 과도한 post-processing 때문에 real-time 과는 거리가 있다.

**YOLACT는 localization 단계를 생략한다.**

대신에 두 가지 task를 병렬적으로 해결한다.
- 전체 이미지에 대한 **prototype mask** dictionary 생성
- instance 마다의 **linear combination coefficients** 예측

각 instance 마다 예측된 coefficients를 이용하여 prototype mask를 linear하게 합친다. 그 후, 예측된 bounding box로 crop한다. 자세한 내용은 Method 부분에서 설명하겠다.

저자는 위 두 task를 이용하면, 네트워크 스스로 **(시각적, 공간적, 의미적으로) 비슷한 instance**를 다르게 나타내는 **instance mask**가 잘 **localize** 될 수 있도록 학습 되어진다고 하였다. 

## Method
![모델의 전체 구조](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/YOLACT/architecture.PNG)

one-stage object detection 모델에 feature localization step 없이 mask branch를 추가하기 위해서 instance segmentation task를 두 가지의 간단한 task로 병렬 처리 한다. 위 그림을 보면 Protonet과 Prediction Head로 각각 병렬 처리 되는 것을 알 수 있다.
- FCN을 사용하여 instance에 의존하지 않은 image 크기의 **prototype masks** 생성하는 task
- prototype 공간에서 instance의 정보를 가진 **mask coefficients**를 예측하기 위한 object detection task

두 task의 결과물을 linear하게 합쳐서(combine), NMS를 통해 살아남은 instance의 mask를 생성한다.

저자는 masks는 공간적으로 일관성(spatially coherent)이 있기 때문에 위 방식을 선택했다고 한다. semantic한 결과를 얻을 수 있는 fc layer을 통해 mask coefficients를 예측하고, spatially coherent에 탁월한 conv layer을 통해 prototype masks를 생성하였다. 또한 두 결과물을 합칠때 생기는 계산량은 단순한 매트릭스 곱셈이기 때문에 빠르다.

### Prototype Generation
![protonet의 구조](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/YOLACT/protonet.PNG)



### Mask Coefficients
![head의 구조](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/YOLACT/head.PNG)



### Mask Assembly


## Experiments


## Reference
- http://research.sualab.com/introduction/2017/11/29/image-recognition-overview-2.html
- 