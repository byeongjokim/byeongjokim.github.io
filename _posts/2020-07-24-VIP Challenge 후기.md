---
layout: post
title:  "[ECCV2020] - VIPriors Workshop Challenge(Image Classification) 3rd 후기"
description: VIPriors Workshop Challenge 후기
date:   2020-07-24 20:03:00 +0900
categories: Others
use_math: true
---

ECCV2020의 VIPriors Workshop에서 개최한 VIPriors Challenge에 참가하였다. 

VIPriors Workshop 이란 "1st Visual Inductive Priors for Data-Efficient Deep Learning Workshop" 로, 성능을 높이기 위해 Data를 효과적으로 사용하는 방법에 대해 다룬다. [URL](https://vipriors.github.io/)

이 Workshop에는 총 4가지 태스크로 Challenge가 진행되었다.
- Image Classfication (ImageNet)
- Semantic Segmentation (Cityscapes)
- Object Detection (MS COCO)
- Action Recognition (UFC-101)

모두 각 분야에서 가장 유명한 데이터 셋으로 대회가 진행된다. 하지만 다른 대회와는 다르게 몇 가지 제약이 존재한다.
- Transfer Learning 금지
- 주어진 데이터셋만을 사용 (외부 Knowledge 및 데이터 등 사용 불가)

우선 Transfer Learning이 금지되었고, 외부 데이터 사용이 불가능 하였기 때문에, Pretrained weight을 사용할 수 가 없다. 그리고 주어진 데이터셋은 기존 데이터셋의 극히 일부에 불가했다. 예를들어 Image Classification의 경우 1000개의 class에 대해 training dataset, validation dataset이 각 50장씩 밖에 존재하지 않았다. 즉 **1000개의 class를 분류하는데 pretrained weight 없이 50000장 만을 사용해서 학습을 해야한다.**

대회는 2020년 3월 1일 부터 7월 10일까지 [Codalab](https://competitions.codalab.org/competitions/23713)에서 진행되었다. 또한 Baseline 코드는 [Github](https://github.com/VIPriors/vipriors-challenges-toolkit)을 통해 제공되었다.

우리 팀은 Image Classfication과 Object Detection 분야에 참가를 하였다. 하지만 중반 이후 Image Classification 하나의 대회에만 집중하였다. 우리는 대회에서 제공한 코드를 사용하지 않고, [timm](https://github.com/rwightman/pytorch-image-models) 오픈소스를 사용하였다. 이 코드에 여러 기법을 붙여서 train, inference 하며 실험을 하였다.

외부 데이터 없이, 주어진 데이터셋을 효율적으로 사용하기 위해 Data Augmentation 이 불가피하였고, 성능을 높이기 위해 여러 Ensemble을 실험하였다. Scratch 데이터셋으로 좋은 성능을 낸 [Cosine Loss](https://byeongjokim.github.io/posts/Deep-Learning-on-Small-Datasets-without-Pre-Training-using-Cosine-Loss/) 또한 사용하였고, 여기에 Focal Loss를 붙여서 학습이 잘 안되고 있는 class에 집중하였다. 기법들을 자세히 설명하고 싶지만, 이는 모두 Technical report에 적혀 있기 떄문에 아래 Arxiv을 통해 확인 할 수 있다.

data augmentation 등 여러 기법을 통해 학습된 10가지의 모델을 Ensemble을 시킨 결과 0.67의 top-1 Accuracy를 얻어낼 수 있었다. 대회 마지막 날까지 1등을 기록하고 있었는데, 대회 종료 1시간 전에 세 팀이 높은 성능의 모델을 제출을 하여 4등으로 leaderboard가 마무리 되었다. Codalab은 점수를 숨길 수가 있다... 하지만 후에 제출해야하는 Technical report에서 한팀이 탈락하게되어, 이번 Challenge는 3등으로 마무리 되었다.

우리가 제출한 Technical report는 [Arxiv](https://arxiv.org/abs/2007.07805)에서 확인 할 수 있다. 

> 주로 사용했던 gpu 서버의 고장과, 처음 제출해보는 Arxiv에 고생을 하였지만, 처음으로 참가한 대회에서 좋은 성적을 낼 수 있어서 다행이라 생각한다.