---
layout: post
title:  "Paper Reivew - End-to-End Object Detection with Transformers"
description: Facebook Research에서 발표한 Object Detection 모델
date:   2020-06-11 20:03:00 +0900
categories: Paper
use_math: true
---
Facebook Research에서 발표한 Object Detection 모델([Paper](https://ai.facebook.com/research/publications/end-to-end-object-detection-with-transformers), [OfficialCode](https://github.com/facebookresearch/detr))

## Introduction
기존 Object Detection 모델인 경우 NMS와 같은 postprocessing step이 불가피하다. DETR(DEtection TRansformer)은 이러한 pipeline을 간단히 한 end-to-end 모델이다. 이름에도 알 수 있듯이 **Transformer** 기반의 encoder-decoder을 사용하여 동일한 예측을 없앴다. 또한 모든 object들을 하나의 set으로 생각 하고, ground-truth objects와 **Bipartite Matching**을 통해 loss function을 계산하였다. 

## Bipartite Matching Loss
DETR은 decoder을 통해 고정된 사이즈인 N개의 object를 예측 한다. 이 N개의 object와 ground-truth object간의 **optimal bipartite matching**이 이루어진다.

![Equ:1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/detr/equ1.PNG){: width="50%"}

우선 N은 ground-truth object 수보다 넉넉하게 큰 수로 설정해놓고 예측한 N object set($$\hat(y)$$)의 permutation(순열) 중 ground-truth object set($$y$$) 과의 $$L_{match}$$가 가장 작은 순열($$\hat(\sigma)$$)을 찾는다. ground-truth set은 N 사이즈 되도록 pad를 채워놓는다. 이 때 pad은 $$\phi$$(no object)를 의미한다.

![Equ:1-1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/detr/equ1-1.PNG){: width="50%"}

$$L_{match}$$은 각 쌍의 class와 box에 대한 pair-wise matching cost이다. c는 class label이며, b는 0과 1사이의 이미지 사이즈와 상대적인 box 중심, height 그리고 width로 이루어져있다. 그리고 p는 예측 확률이다. object가 있는 ground-truth와 그에 해당하는 쌍과의 matching cost이다. 이 때 위 matching cost는 anchor와 같은 역할을 한다고 한다. anchor을 이용하면 duplicate 될 수 있지만, 위 방법은 one-to-one 매칭으로 duplicate를 방지한다.








![Equ:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/detr/equ2.PNG){: width="60%"}
![Equ:3](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/detr/equ3.PNG){: width="50%"}
![Fig:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/detr/fig2.PNG){: width="80%"}

## Transformer

## Experiments
### Datasets

## Review
> a

## Reference
- a