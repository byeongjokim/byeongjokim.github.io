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
cosine similarity는 두 vector $$a, b \in R^{d}$$의 사이각을 이용한 유사도이다. 아래 식에서 $$<\cdot , \cdot>$$은 dot product를 뜻하며, $$\|\|_{p}$$는 $$L^{p}$$ norm을 뜻한다.

$$\sigma_{cos}(a, b) = cos(a \angle b) = \frac{<a,b>}{\|a\|_{2} \cdot \|b\|_{2}}$$

$$x \in X$$ 데이터와 $$y \in C$$ label이 주어질 때, $$f_{\theta} \colon X \rightarrow R^{d}$$ 를 통해 X는 d-dimensional feature space로 transform 된다. 그리고 $$\psi \colon R^{d} \rightarrow P$$ 와 $$\varphi \colon C \rightarrow P$$ 가 각각 feature와 classes를 prediction space P에 임베딩 시킨다.

$$\varphi$$ 를 고정시키고, $$f_{theta}$$ 의 파라미터를 학습 시켜서 이미지 feature와 각 class의 cosine simiairty를 극대화 시키는 것이 cosine loss function이며, 아래 식과 같다.

$$L_{cos}(x, y) = 1 - \sigma_{cos}(f_{\theta}(x), \varphi(y))$$

$$\psi = \frac{x}{\|x\|_{2}}$$ 와 one-hot vector인 $$\varphi$$ 를 사용하여 두 백터 $$f_{\theta}(x), \varphi(y)$$ 를 unit hypersphere로 바꾸었다. 이로써 cosine similarity를 단순한 dot product로 구할 수 있게 되었다.

$$L_{cos}(x, y) = 1 - <\varphi(y), \psi(f_{\theta}(x))>$$

## Comparison with Categorical Cross-Entropy and Mean Squared Error
cosine loss를 categorical cross-entropy loss 그리고 mean squared error(MSE)와 비교를 하였다. 가장 큰 차이점은 예측(predictions)과 ground-truth 와의 차이를 측정하는 방식에 있다.

categorical cross-entropy loss 그리고 mean squared error(MSE) 두 loss를 간단히 설명하면 
- MSE는 feature space에 transform을 적용하지 않고, Euclidean prediction space를 이용한다. 위 unit hypersphere를 통한 $$L_{cos}$$ 식을 풀어 쓰면 Euclidean distance와 매우 유사하다. ($$L^2$$ Norm 값이 1로 고정 이기 때문에)
- Categorical cross-entropy는 Kullback-Leibler divergence를 이용하여 확률 분포(probability distribution) 공간의 차이를 측정하는 loss 이다.  이는 softmax 함수를 이용하여 prediction space로 transform 시킨다. 

두 loss와 비교했을 때 cosine loss는 두 가지 특징이 있다.
- loss는 [0, 2] 경계로 되어있다. 반면에 다른 loss는 매우 큰 값을 가진다.
- feature vector의 direction 만을 고려하기 때문에, scaling에 invariant 하다.

![Fig:1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/cosine loss/fig1.PNG){: width="80%"}

위 figure을 보면 cross entropy loss는 급강하 영역과 두 넓은 지역으로 이루어져있다. 밝은 부분과 어두운 부분은 일정하진 않지만 매우 차이가 적다. 따라서 초기화 및 learning rate 설정을 잘 해야할 것이다. 반면, figure 1.c cosine loss을 보면 색이 고르게 분포되어 있어서 더 robust 할 것이다.

또한 cross-entropy loss 는 true class의 값이 다른 class의 값 보다 헌저하게 커야만(혹은 infinity) optimum 한 값을 얻어진다. 따라서 적은 dataset을 이용하여 학습을 하면, overfitting이 일어날 수 있다. 보통 이 문제를 해결하기 위해 label smoothing 을 적용한다(ground-truth distribution 에 noise를 주어서 regularization). 간단하게 설명하자면 [1, 0, 0] 이라는 ground-truth 대신 [$$1-\varepsilon$$, $$\frac{\varepsilon}{n-1}$$, $$\frac{\varepsilon}{n-1}$$] 로 설정하는 것이다. 이때 $$\varepsilon$$ 은 0.1 과 같은 작은 상수이다.

cosine loss는 $$L^{2}$$ normalization이 regularizer 역할을 하여 $$\varepsilon$$ 과 같은 hyper-parameter를 사용하지 않는다. 또한 높은 차원에서 문제가 되는 Euclidean distance를 사용하지 않고, 방향만을 고려하여 학습에 적용된다. 따라서 적은 dataset을 사용할 때, scaling에 invariance한 점이 좋은 regularizer로 작용된다.

## Semantic Class Embeddings



## Experiments

## Review
> d

## Reference
