---
layout: post
title:  "Relation Networks for Object Detection"
description: 저자는 딥러닝이 유행하기 이전의 object relation을 이용하여 이 문제를 해결하고자 하였다. 그러나 보통 object는 각기 다른 위치, 다른 크기, 다른 종류 그리고 다른 이미지로 검출 되기 때문에 NLP에서 성공을 얻은 attention model (Attention is all you need)을 이용하였다.
date:   2020-02-22 17:20:00 +0900
categories: Paper
---
2018 CVPR에서 Oral로 발표된 논문이다. ([Paper](https://arxiv.org/pdf/1711.11575) || [Code](https://github.com/msracver/Relation-Networks-for-Object-Detection))

## Introduction
기존 object detection 모델들을 보면 각 proposal에 object classification과 bounding box regression이 개별적으로 이루어 졌다. 또한 non-maximum suprression (NMS)도 heuristic하게 post-processing step에서 시행되었다.

저자는 딥러닝이 유행하기 이전의 object relation을 이용하여 이 문제를 해결하고자 하였다. 그러나 보통 object는 각기 다른 위치, 다른 크기, 다른 종류 그리고 다른 이미지로 검출 되기 때문에 object-object relation 모델을 만드는 것은 어렵다. 이때 저자는 NLP에서 성공을 얻은 attention model (Attention is all you need)에 영감을 받았다고 한다. 각기 다른 위치, 특징에 상관 없이 모든 elements 사이의 dependency를 모델화 시킬 수 있기 때문이다.

본 논문에서 original weight와 geometric weight를 이용한 attention module을 제안한다. 기존 방식에 geometric weight를 추가로 사용한 이유는 object간의 상대적인 공간의 관계를 파악하기 위해서 라고 한다. 이 module은 **object relation module** 이라 한다.

## Method
### Object Relation Module
기본 "Scaled Dot-Product Attention"의 과정을 기반으로 설계 되었다. Scaled Dot-Product Attention을 간략하게 설명하면, 쿼리 q와 key K의 요소간의 유사도를 계산하여 value V의 값을 변화 시키는 메커니즘이다. 계산된 유사도에 따라 V에 영향을 끼치는 정도가 달라지며, 이를 attention이라 생각하면 된다. V는 결국 관련 있는 요소들에 영향을 받아 값이 변한다.

이제 object relation module에 대한 설명을 하겠다. 우선 각 object는 **geometric feature**(f_G)와 **appearance feature**(f_A)로 이루어져 있다. 당연히 geometric feature은 4-d 이며, appearance feature은 architecture 구조에 따라 달라진다. 

논문에서는 top-down 방식으로 수식을 설명했지만 원활한 설명을 위해 down-top 방식으로 설명하겠다. 

![equ:4](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/equation_4.PNG){: width="40%"}

위는 m object와 n object의 appearance feature를 이용하여 **appearance weight**를 Scaled Dot-product로 구하는 식이다. 각 appearance feature은 같은 공간으로 임베딩 되어 유사도가 계산된다. 그리고 유사도로 weight가 결정된다. 이 weight의 softmax 값을 이용하면 일반적인 attention 모델이 되지만, 아까 말했듯이 한가지의 weight를 더 고려하였다.

![equ:5](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/equation_5.PNG){: width="40%"}

위는 m object와 n object의 geometry feature를 이용하여 **geometry weight**를 구하는 식이다. 우선 각 geometry feature은 높은 차원으로 임베딩된다 (위 식의 EG). translation invariant, scale transformation 등을 위해 ![equ:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/positional.PNG){: width="30%"} 로 바꾸어 계산한다. 그 후 Attention is all you need 에서 소개한 Positional Encoding을 이용하여 d_g 차원으로 임베딩 된다. 그 후 WG를 이용하여 scalar weight가 되고, ReLU function을 이용한다. ReLU를 거치는 이유는, 두 object 간의 geometric 관계이기 때문이다. 

![equ:3](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/equation_3.PNG){: width="40%"}

이로서 object m과 n의 attention weight(relation weight)를 구할 수 있게 된다. 기존 attention 식과 달리 geometric weight이 추가되었다. 이 relation weight은 다른 object로 부터 받는 영향을 뜻한다. 식을 보면 전체 object 중 m이 n에게 미치는 영향을 확률 값으로 나타낼 수 있게 되었고,

![equ:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/equation_2.PNG){: width="40%"}

위 식을 이용하여 모든 object 간의 weight sum을 한다. 이를 통해 다른 object의 영향이 첨가된 relation feature를 얻을 수 있다. 

![equ:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/equation_6.PNG){: width="40%"}

object relation module은 총 N_r개의 relation feature을 이용하였다. 즉 multi-head attention과 같이 다양한 relation feature들을 concat 하였다. 인풋과 아웃풋의 dimension을 맞추기 위해 각 relation feature의 dimension은 appearance feature의 1/N_r 로 정하였다.

각 object의 appearance feature의 차원을 d_f, key의 차원을 d_k로 두면, 파라미터 공간의 크기가 계산이된다.
- W_K: d_k x d_f (d_f 차원(object m)을 d_k 차원으로 임베딩)
- W_Q: d_k x d_f (d_f 차원(object n)을 d_k 차원으로 임베딩)
- W_G: d_g (임베딩된 d_g차원을 scalar weight로 바꿈)
- W_V: d_f x d_f/N_r (d_f 차원(object m)을 d_f/N_r(relation features))

![parameters](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/relation_networks_for_object_detection/parameter.PNG){: width="40%"}
따라서 총 N_r 개의 relation feature을 만들기 때문에 위 식이 성립된다. 본 논문에서는 N_r = 16, d_k = 64, d_g = 64 로 놓았고, N과 d_f는 수백으로 놓았다.

**위 까지 포스팅을 쓰다보니.. 얼른 수식을 쓸 수 있게 블로그 손봐야겠다..**

### Relation Networks


#### Relation for Instance Recognition
#### Relation for Duplicate Removal





### a

## Experiments

## Reference
