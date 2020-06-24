---
layout: post
title:  "Paper Reivew - Rethinking Pre-training and Self-training"
description: Pre-training과 Data Augmentation, 그리고 Self-training에 대한 실험
date:   2020-06-23 20:03:00 +0900
categories: Paper
use_math: true
---
Pre-training과 Data Augmentation, 그리고 Self-training에 대한 실험에 관한 논문 ([Paper](https://arxiv.org/pdf/2006.06882v1.pdf))

Object Detection 뿐만 아니라 여러 Vision Task에서 **ImageNet으로 학습된 Pre-train**은 필수로 사용된다. 하지만 [Rethinking ImageNet PreTraining](https://arxiv.org/abs/1811.08883) 에서 이에 반대 되는 입장을 내었다. 저 논문에서는 Pre-Training은 빠른 학습을 돕긴 하지만 **Scratch(w/o Pre-Training)로 학습을 해도 충분히 좋은 성능**을 낼 수 있다고 여러 실험을 통해 증명하였다. 

본 논문에서는 다시한번 Pre-training의 필요성을 부정하며, 오히려 **Self-training**을 하면 더 좋은 성능을 낼 수 있음을 강조한다. 또한, **강력한 Data Augmentation을 이용할 경우** 오히려 Pre-training이 성능을 **악화** 시킨다고 실험을 통해 밝혔다.

## Data Augmentation
본 논문에서는 Data Augmentation을 네단계로 나누어 실험하였다.

1. Augment-S1
    - 가장 약한 Augmentation, Flip과 Crop으로만 구성
2. Augment-S2
    - 두번째로 약한 Augmentation, Augment-S1에 AutoAugment 추가
3. Augment-S3
    - 두번째로 강한 Augmentation, Augment-S2에 Large Scale Jittering을 추가
4. Augment-S4
    - 가장 강한 Augmentation, Large Scale Jittering과 RandAugment 그리고 Flip, Crop으로 구성

## Pre-training
세가지의 weight을 사용하여 실험하였다.

1. Rand Init
    - pre-train 없이 랜덤으로 초기화된 weights 사용
2. ImageNet Init
    - ImageNet으로 학습된 weights 사용(84.5% top-1 accuracy)
3. ImageNet++ Init
    - Noisy Student 방법으로 학습된 weights 사용(86.9% top-1 accuracy)

## Self-training
Noisy Student 방법으로 학습을 진행하였다.

- Teacher model
    - label된 COCO dataset으로 학습
- unlabeld ImageNet dataset을 통해 pseudo label 생성
- Student model
    - label된 COCO dataset과 pseudo label된 ImageNet dataset으로 학습

## Experiments

## Pre-training - Augmentation, Labeled dataset size

## Self-training - Augmentation, Labeled dataset size

## Self-supervised Pre-training

## Discussion
