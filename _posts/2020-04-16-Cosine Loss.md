---
layout: post
title:  "PyTorch - Cosine Loss"
description: Pyotrch implements of Deep Learning on Small Datasets without Pre-Training using Cosine Loss
date:   2020-04-16 20:03:00 +0900
categories: Development
use_math: true
---

Deep Learning on Small Datasets without Pre-Training using Cosine Loss ([Arxiv](https://arxiv.org/abs/1901.09054), [Review](https://byeongjokim.github.io/posts/Deep-Learning-on-Small-Datasets-without-Pre-Training-using-Cosine-Loss/))의 cosine loss implements(Pytorch)

Semantic Class Embeddings를 사용하지 않고 **One-Hot Embedding**을 사용하여 **Cosine Loss + Cross Entropy Loss**를 implement 하였다.

$$L_{cos+xent}(x, y) = 1 - <\psi(f_{\theta}(x)), \varphi_{onehot}(y)> - \lambda <\varphi_{onehot}(y), log(g_{\theta}(\psi(f_{\theta}(x))))>$$

PyTorch의 [cosine_embedding_loss](https://pytorch.org/docs/stable/nn.functional.html#cosine-embedding-loss) 와 [cross_entropy](https://pytorch.org/docs/stable/nn.functional.html#cross-entropy) 를 이용하였다.

$$
\text{cosine_embedding_loss}(x_1, x_2, y) =
\begin{cases}
    1 - \cos(x_1, x_2), & \text{if } y = 1 \\
    \max(0, \cos(x_1, x_2) - \text{margin}), & \text{if } y = -1
\end{cases}
$$

$$\text{cross_entropy}(x, class) = -\log\left(\frac{\exp(x[class])}{\sum_j \exp(x[j])}\right)$$

cosine_embedding_loss 에는 y 값이 1이여야 원하는 식을 얻을 수 있기 때문에 __init__에 *self.y = torch.Tensor([1])* 를 선언한 후 이용하였다.

논문의 식에는 cross entropy loss 부분을 cosine similarity로 계산하였지만, 결국 cross entropy loss와 동일한 계산 식이기 때문에 **F.cross_entropy**를 사용하였으며, **F.normalize** 로 unit hypersphere 시킨 input을 인자 값으로 사용하였다. 

Pretrained weight 없이, class 당 50개의 이미지를 가진 ImageNet subset으로 90 Epochs 학습 시킨 결과는 아래를 통해 비교할 수 있다.

|Model (Loss)|Top-1 Accuracy(%)|
|------|---|
|ResNet-50 Same-Conv (Cross Entropy Loss)|26.39|
|ResNet-50 Same-Conv (Cosine Loss + 0.1 * Cross Entropy Loss) |34.01|

코드는 아래와 같다. 

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CosineLoss(nn.Module):
    def __init__(self, xent=.1, reduction="mean"):
        super(CosineLoss, self).__init__()
        self.xent = xent
        self.reduction = reduction
        
        self.y = torch.Tensor([1])
        
    def forward(self, input, target):
        cosine_loss = F.cosine_embedding_loss(input, F.one_hot(target, num_classes=input.size(-1)), self.y, reduction=self.reduction)
        cent_loss = F.cross_entropy(F.normalize(input), target, reduction=self.reduction)
        
        return cosine_loss + self.xent * cent_loss
```

사용법은 기존 Loss 사용법과 동일하다.

```python
criterion = CosineLoss().cuda(args.gpu)

...

loss = criterion(output, target)
```