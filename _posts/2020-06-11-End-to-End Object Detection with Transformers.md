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

## Transformer

## Experiments
### Datasets

## Review
> a

## Reference
- a