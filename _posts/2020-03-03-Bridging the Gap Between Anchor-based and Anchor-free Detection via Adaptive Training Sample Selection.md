---
layout: post
title:  "Bridging the Gap Between Anchor-based and Anchor-free Detection via Adaptive Training Sample Selection"
description: 리뷰 작성 중 ... anchor-based와 anchor-free 방법의 차이를 실험적으로 설명하며, 성능의 차이는 positive와 negative sample의 정의에 있다고 하였다. 그리고 이에 object 특성에 따라 자동으로 positive, negative sample을 선택하는 Adaptive Training Sample Selection (ATSS)을 제안한다.
date:   2020-03-03 20:03:00 +0900
categories: Paper
---
2020 CVPR에서 발표될 논문이다. ([Arxiv](https://arxiv.org/abs/1912.02424) || [Code](https://github.com/sfzhang15/ATSS))

## Introduction
Object detection 문제는 주로 **anchor-based detector**에 대한 연구로 진행되었다. anchor-based detector은 미리 세팅해놓은 수 많은 anchor에서 category를 예측하고 coordinates를 조정하는 방식이다. two-stage method와 one-stage method가 존재한다.

하지만 최근에는 FPN과 Focal Loss의 출현으로 인해 **anchor-free detector** 방식에 대한 연구가 진행되고 있다. anchor-free detector은 anchor 없이 object를 바로 찾을 수 있다. 두 가지 방법이 있는데, 키 포인트를 이용하여 object의 위치를 예측하는 keypoint-based 방법과 object의 중앙을 예측한 후 positive인 경우 object boundary의 거리를 예측하는 center-based 방법이 있다. 이러한 anchor-free detector은 anchor에 관련된 hyperparameter를 사용하지 않고 anchor-based detector와 비슷한 성능을 얻기 때문에, object detection 분야에서 더 잠재력 있다고 여겨진다.

center-based detector은 anchor box 대신에 point를 이용하는 점에서 anchor-based detector와 유사하다. 그래서 one-stage anchor-based detector인 RetinaNet과 center-based anchor-free detector인 FCOS를 비교하였다.
- location 마다 설정해 놓은 anchor(point)의 개수
- positive 또는 negative sample의 정의
- regression starting status

```
논문에서는 위의 내용을 introduction 부터 설명을 하였지만, 뒤에 어떤 차이가 성능에 큰 영향을 미치는 지에 대한 실험을 포함한 반복적인 내용이 있기 때문에 뒤에 더 자세하게 설명을 하겠다.
```

논문에서는 anchor-based와 anchor-free 방법의 차이를 실험적으로 설명하며, 성능의 차이는 **positive와 negative sample의 정의**에 있다고 하였다. 그리고 이에 object 특성에 따라 자동으로 positive, negative sample을 선택하는 **Adaptive Training Sample Selection (ATSS)**을 제안한다. ATSS로 State-of-the-art AP 50.7% 성능을 얻었으며, 이 방법을 이용하면 많은 anchor를 사용하지 않아도 된다고 한다.

```
본 리뷰를 정확히 이해하기 위해서는 anchor-based detector와 anchor-free detector에 기본적인 이해가 우선되어야할 것 같다. 밑 reference 부분에 각각 한 가지 모델 논문의 링크를 올려 놓았다.
```

## Method

## Experiments

## Review

## Reference
- anchor-based detector (e.g., [RetinaNet](https://arxiv.org/abs/1708.02002))
- keypoint-based anchor-free detector (e.g., [CornerNet](https://arxiv.org/abs/1808.01244))
- center-based anchor-free detector (e.g., [FCOS](https://arxiv.org/abs/1904.01355))