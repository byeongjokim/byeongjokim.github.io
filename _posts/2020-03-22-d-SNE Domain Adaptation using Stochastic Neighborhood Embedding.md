---
layout: post
title:  "d-SNE: Domain Adaptation using Stochastic Neighborhood Embedding"
description: writing review...
date:   2020-03-22 20:03:00 +0900
categories: Paper
use_math: true
---
CVPR 2019 에서 발표된 논문이다. ([Arxiv](https://arxiv.org/abs/1905.12775) || [Code](https://github.com/aws-samples/d-SNE))

## Domain Adaptation
Domain Adaptation 충분한 데이터가 없을 때에도 준수한 성능을 내기 위한 연구이다. Domain Adaptation 연구는 backpropagation 에 GRL(Gradient Reversal Layer)을 추가한 [DANN](https://arxiv.org/abs/1505.07818), Discriminator를 이용한 [ADDA](https://arxiv.org/abs/1702.05464), CycleGAN과 비슷한 방식으로 픽셀 레벨의 Domain Adaptation을 진행한 [CyCADA](https://arxiv.org/abs/1711.03213), 두 도메인 사이의 거리를 고려한 [DLOW](https://arxiv.org/abs/1812.05418) 등 여러 방법으로 꾸준히 연구되어 왔다. **Domain Adaptation**은 간단히 얘기를 한다면 **label이 존재하는 S(source) 도메인 데이터를 이용하여, label이 존재하지 않는 T(target) 도메인에서 높은 성능을 내는 모델을 만드는 연구**이다.

## Introduction

![Fig:1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/d_SNE/fig1.PNG){: width="60%"}

위 figure 상단 처럼 dataset $$D^T$$ 이 $$D^S$$ 안에 속한다면, 매우 운이 좋은 경우이다. 대개의 경우 하단과 같이 각 dataset의 distribution이 겹치지 않으며, 이로 인해 좋은 성능을 내기 어렵다. 이는 **domain-shift** 때문에 일어나는 현상이며, $$D^S$$으로 학습된 모델은 의미가 없어진다. **Domain Adaptation**은 위에 설명하듯이 다른 도메인의 성능을 높이는 일이며, 두 도메인 모두에서 동일한 성능을 갖게 된 경우를 **Domain Generalization**이라고 한다.

![Fig:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/d_SNE/fig2.PNG){: width="60%"}

Domain Adaptation 에는 두가지 방법이 있다.
1. Domain Transformation
    - source 도메인으로 부터 학습된 모델을 사용하기 위해
    - target 도메인을 source 도메인으로 변환하는 방법
    - 주로 GAN based model이 사용된다.
2. Latent-Space Transformation:
    - source 도메인에서 추출한 feature와 target 도메인에서 추출한 feature을 동일한 latent space로 임베딩 시키는 방법

본 논문에서 소개하는 d-SNE은 두번째 방법을 이용한다. 여러 기법과 모델을 사용하였고, 자세한 내용은 바로 밑에서 다루겠다.

# d-SNE



## Review
> ..

## Reference
- [..](https://......)