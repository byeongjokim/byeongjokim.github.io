---
layout: post
title:  "PyTorch - Cosine Loss"
description: Pyotrch implements of Deep Learning on Small Datasets without Pre-Training using Cosine Loss
date:   2020-04-16 20:03:00 +0900
categories: Development
use_math: true
---

Deep Learning on Small Datasets without Pre-Training using Cosine Loss ([Arxiv](https://arxiv.org/abs/1901.09054), [Review](https://byeongjokim.github.io/posts/Revisiting-the-Sibling-Head-in-Object-Detectormd/))의 cosine loss implements(Pytorch)

Semantic Class Embeddings를 사용하지 않고 **One-Hot Embedding**을 사용하여 **Cosine Loss + Cross Entropy Loss**를 implement 하였다.

$$L_{cos+xent}(x, y) = 1 - <\psi(f_{\theta}(x)), \varphi_{onehot}(y)> - \lambda <\varphi_{onehot}(y), log(g_{\theta}(\psi(f_{\theta}(x))))>$$

기존 PyTorch에 [CosineEmbeddingLoss](https://pytorch.org/docs/stable/nn.html#cosineembeddingloss)가 존재 하지만, 매번 인자 값에 Tensor([1])를 집어넣어야 하며, cuda를 사용할 시 매번 cuda()로 바꿔줘야하기 때문에 **F.cosine_similarity** 를 사용하였다.

> 매번 Tensor([1]).cuda()를 사용해도 괜찮은 것인지 아직 잘 모르겠다. 만약에 상관이 없으면 [F.cosine_embedding_loss](https://pytorch.org/docs/master/nn.functional.html?highlight=cosine_embedding#torch.nn.functional.cosine_embedding_loss) 함수를 사용하면 된다. ex) **cosine_losses = F.cosine_embedding_loss(input, F.one_hot(target, num_classes=input.size(-1)), torch.Tensor([1]).cuda(), reduction=self.reduction))**

위 식에서 cross entropy loss 부분을 cosine similarity로 계산이 가능하지만 결국 cross entropy loss와 동일한 계산 식이기 때문에 **F.cross_entropy**를 사용하였으며, **F.normalize** 로 unit hypersphere 시킨 input을 인자 값으로 사용하였다. 

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
        
    def forward(self, input, target):
        cent_loss = F.cross_entropy(F.normalize(input), target, reduction=self.reduction)
        cosine_losses = 1 - F.cosine_similarity(input, F.one_hot(target, num_classes=input.size(-1)))

        if self.reduction == "mean":
            cosine_loss = torch.mean(cosine_losses)
        elif self.reduction == "sum":
            cosine_loss = torch.sum(cosine_losses)
        else:
            cosine_loss = cosine_losses

        return cosine_loss + self.xent * cent_loss
```

사용법은 기존 Loss 사용법과 동일하다.

```python
criterion = CosineLoss().cuda(args.gpu)

...

loss = criterion(output, target)
```