---
layout: post
title:  "Deep Learning on Small Datasets without Pre-Training using Cosine Loss"
description: Writing Review...
date:   2020-04-01 20:03:00 +0900
categories: Paper
use_math: true
---
([Arxiv](https://arxiv.org/abs/1901.09054))

## Introduction
하드웨어의 발전으로 인해 매우 큰 dataset을 사용하고있다. 하지만 학습 데이터를 많이 모으기에는 한계가 존재하며 이를 위해 pre-trained 모델과 fine-tuning을 통하여 이를 극복할 수 있다. 하지만, pre-trained 모델을 사용한 *transfer learning*에는 문제 될만한 것이 두가지가 있다. 우선 의료 이미지 같이 target domain이 매우 special한 경우 기존(imageNet) 이미지와 큰 차이가 있으며, license의 문제 또한 존재한다.

이에 본 논문에는 softmax+cross-entropy에 대해 문제를 제기하면서, cosine loss function을 사용하여 **small data without external information** 문제를 해결하는 방법을 제시한다.

small dataset을 해결하기 위해 few-shot learning, metric learning, 그리고 meta-learning 기법이 존재한다. 하지만 이 기법들 모두 우선 large dataset을 사용해야 한다. 이 논문에서는 large dataset 없이 각 클래스당 20 ~ 100 개 미만의 학습 이미지로 이루어진 **small dataset** 만을 이용한다.

## Cosine Loss
cosine similarity는 두 vector $$a, b \in R^{d}$$의 사이각을 이용한 유사도이다.

$$\sigma_{cos}(a, b) = cos(a \angle b) = \frac{<a,b>}{\|a\|_{2} \cdot \|b\|_{2}}$$

위 식에서 $$<\cdot , \cdot>$$은 dot product를 뜻하며, $$\|\|_{p}$$는 $$L^{p}$$ norm을 뜻한다.

$$x \in X$$인 도메인과 $$y \in C$$인 label이 있을 때, $$f_{\theta} \colon X \rightarrow R^{d}$$ 를 통해 X를 d-dimensional feature space로 transform 시킨다. 그리고 $$\psi \colon R^{d} \rightarrow P$$와 $$\varphi \colon C \rightarrow P$$가 각각 feature와 classes를 prediction space P에 임베딩 시킨다.

$$\varphi$$가 고정되어 지면, $$f_{theta}$$의 파라미터를 학습 시켜서 이미지 feature와 각 class의 cosine simiairty를 극대화 시키는 것이 cosine loss function이며, 아래 식과 같다.

$$L_{cos}(x, y) = 1 - \sigma_{cos}(f_{\theta}(x), \varphi(y))$$

이때 cosine similarity가 dot product로 변할 수 있도록 두 백터 $$f_{\theta}(x), \varphi(y)$$를 unit hypersphere로 바꾸었다. $$\psi = \frac{x}{\|x\|_{2}}$$를 사용하고, $$\varphi$$를 one-hot vector로 사용하면 두 벡터 $$f_{\theta}(x), \varphi(y)$$는 각각 unit hypersphere가 된다.

$$L_{cos}(x, y) = 1 - \sigma_{cos}(\varphi(y), \psi(f_{\theta}(x)))$$


## Experiments

## Review
> d

## Reference
