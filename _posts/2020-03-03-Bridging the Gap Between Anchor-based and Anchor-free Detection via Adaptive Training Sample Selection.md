---
layout: post
title:  "Bridging the Gap Between Anchor-based and Anchor-free Detection via Adaptive Training Sample Selection"
description: 리뷰 작성 중 ... anchor-based와 anchor-free 방법의 차이를 실험적으로 설명하며, 성능의 차이는 positive와 negative sample의 정의에 있다고 하였다. 그리고 이에 object 특성에 따라 자동으로 positive, negative sample을 선택하는 Adaptive Training Sample Selection (ATSS)을 제안한다.
date:   2020-03-03 20:03:00 +0900
categories: Paper
use_math: true
---
2020 CVPR에서 발표될 논문이다. ([Arxiv](https://arxiv.org/abs/1912.02424) || [Code](https://github.com/sfzhang15/ATSS))

## Introduction
Object detection 문제는 주로 **anchor-based detector**에 대한 연구로 진행되었다. anchor-based detector은 미리 세팅해놓은 수 많은 anchor에서 category를 예측하고 coordinates를 조정하는 방식이다. two-stage method와 one-stage method가 존재한다.

하지만 최근에는 FPN과 Focal Loss의 출현으로 인해 **anchor-free detector** 방식에 대한 연구가 진행되고 있다. anchor-free detector은 anchor 없이 object를 바로 찾을 수 있다. 두 가지 방법이 있는데, 키 포인트를 이용하여 object의 위치를 예측하는 keypoint-based 방법과 object의 중앙을 예측한 후 positive인 경우 object boundary의 거리를 예측하는 center-based 방법이 있다. 이러한 anchor-free detector은 anchor에 관련된 hyperparameter를 사용하지 않고 anchor-based detector와 비슷한 성능을 얻기 때문에, object detection 분야에서 더 잠재력 있다고 여겨진다.

center-based detector은 anchor box 대신에 point를 이용하는 점에서 anchor-based detector와 유사하다. 그래서 one-stage anchor-based detector인 RetinaNet과 center-based anchor-free detector인 FCOS를 비교하였다.
- location 마다 설정해 놓은 anchor(point)의 개수
- positive 또는 negative sample의 정의
- regression starting status

> 논문에서는 위의 내용을 introduction 부터 설명을 하였지만, 뒤에 어떤 차이가 성능에 큰 영향을 미치는 지에 대한 실험을 포함한 반복적인 내용이 있기 때문에 뒤에 더 자세하게 설명을 하겠다.

논문에서는 anchor-based와 anchor-free 방법의 차이를 실험적으로 설명하며, 성능의 차이는 **positive와 negative sample의 정의**에 있다고 하였다. 그리고 이에 object 특성에 따라 자동으로 positive, negative sample을 선택하는 **Adaptive Training Sample Selection(ATSS)**을 제안한다. ATSS로 State-of-the-art AP 50.7% 성능을 얻었으며, 이 방법을 이용하면 많은 anchor를 사용하지 않아도 된다고 한다.

> 본 리뷰를 정확히 이해하기 위해서는 anchor-based detector와 anchor-free detector에 기본적인 이해가 우선되어야할 것 같다. 밑 reference 부분에 각각 한 가지 모델 논문의 링크를 올려 놓았다.

## Difference Analysis of Anchor-based and Anchor-free Detection
대표적인 anchor-based detector **RetinaNet**과 anchor-free detector **FCOS**를 비교 실험을 하였다. 각 모델의 논문은 하단에서 볼 수 있다. 위 세 가지 차이 중 두, 세번째 항목에 집중을 한 실험이다. 첫 번째 차이를 없애기 위해 RetinaNet의 location 당 anchor의 개수를 하나로 놓았다. 이로써 FCOS와 비슷한 모델이 완성된다. MS COCO Dataset으로 실험 하였으며 Training Detail과 Inference Detail은 논문을 참고하면 된다.

### Inconsistency Removal
우선 anchor box가 하나인 RetinaNet(#A=1) 모델은 32.5% AP 성능을 얻었으며, FCOS 모델은 37.1% AP 성능을 얻었다. 또한 FCOS는 GIoU loss function과 normalizing 기법을 이용하여 37.8% AP 까지 성능을 올릴 수 있다. 하지만 이 비교(37.8% vs 32.5%)는 FCOS에서 사용된 여러 기법들(각 feature pyramid에 사용된 GIoU, GroupNorm, positive sample 제한, centerness branch, 그리고 trainalbe scalar)로 인해 제대로된 비교가 아니다(anchor-free와 anchor-based detector의 본질적인 차이가 아니기 때문에). 따라서 위 기법들을 RetinaNet에도 적용하여 implementation inconsistency를 없앴다.

![Tab:1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/ATSS/Tab1.PNG){: width="60%"}

위 표에서 볼 수 있듯이, 이러한 irrelevant difference는 RetinaNet의 성능을 37.0% 까지 높였지만 아직 FCOS에 비해 0.8%의 차이가 존재하였다. 비로서 anchor-based와 anchor-free detector의 근본적인 차이(essential difference)를 파헤칠 수 있다.

> FCOS에 적용된 여러 기법들(각 feature pyramid에 사용된 GIoU, GroupNorm, positive sample 제한, centerness branch, 그리고 trainalbe scalar)을 RetinaNet에 적용해서 비교하는게 과연 **implementation inconsistency** 인지 의문이 들었다.

### Essential Difference
이제 RetinaNet(#A=1)과 FCOS에는 두 가지 차이만 존재한다.

1. classification sub-task (positive와 negative sample 정의)
2. regression sub-task (anchor box 또는 anchor point에서 시작되는 regression)

#### Classification
![Fig:1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/ATSS/Fig1.PNG){: width="60%"}

위 Figure 1을 보면 RetinaNet과 FCOS 모델이 positive, negative를 결정하는 대략적인 방법에 대해 알 수 있다. 우선 RetinaNet의 경우 IoU을 이용하여 anchor box를 positive 혹은 negative로 구분한다. IoU가 특정 threshold를 넘지 못하면 학습 단계에서 무시된다. 반면에 FCOS의 경우 spatial constraint, scale constraint를 이용하여 anchor point를 구분한다. ground truth box 안에 있는 anchor point를 positive sample의 후보로 생각을한 후, 미리 정의된 scale range를 통해 최종 positive sample을 결정한다. 이때 선택되지 않은 anchor point는 negative sample 이다.

![Tab:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/ATSS/Tab2.PNG){: width="60%"}

FCOS는 spatial로 후보를 정하고, scale로 선택을 하는 반면 RetinaNet은 IoU를 통해 spatial, scale을 한번에 고려한다. 이 차이는 성능으로 이어진다. 위 표를 보면 RetinaNet(#A=1) (첫 번째 column)에서 IoU 대신 spatial, scale constraint 방식을 사용하니 성능 향상 되는 것을 볼 수 있다. 반대로 FCOS에 IoU 기법을 적용하면 성능이 떨어지는 것을 알 수 있다. 이 부분이 anchor-based와 anchor-free의 본질적인 차이 중 하나이다.

> "위 부분이 어떻게 두 방식(anchor-based vs anchor-free)의 차이일까? anchor을 사용하든(anchor-based) 다른 방식(anchor-free)을 사용하든 spatial, scale constraint 기법 유무의 차이가 아닐까?" 라고 생각이 들 수 있다. 하지만 multiple anchor에 spatial, scale constraint 기법을 사용하는 것은 single anchor에 spatial, scale constraint을 사용하는 것과 다름이 없고, 이는 FCOS 방법과 다름이 없다. single point에 spatial, scale constraint을 사용하기 때문에 anchor-free가 가능한 것이다.

#### Regression
![Fig:2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/ATSS/Fig2.PNG){: width="60%"}

결정된 positive sample에 대해 bounding box를 찾는 regression이 이루어진다. 위 Figure 2를 보면 RetinaNet은 anchor box로 부터 4개의 offset에 대해 regression이 이루어진다. 반면에 FCOS는 점으로 부터 4개의 거리(distance)를 regression 한다. anchor-based와 anchor-free에는 regression의 시작이 box인지, point인지의 차이가 존재한다. 하지만 위 표를 보면 Box 또는 Point 모두 spatial, scale constraint를 적용하면 같은 성능(37.8% AP)을 얻었다. 이는 즉, regression starting status는 성능에 있어 본질적인 차이가 아닌 셈이다.

**결론은 positive sampe, negative sample을 정의하는 방법이 가장 중요하다.**

## Adaptive Training Sample Selection (ATSS)
위의 결론을 토대로 *how to define positive and negative training samples* 에 대해 Adaptive Training Sample Selection을 제안한다. 이 방법은 hyperparameter 없이 robust 하다고한다. ATSS 방법은 위 알고리즘을 통해 알 수 있다. 

1. 모든 ground-truth box $g$의 positive sample 후보를 정한다.
    - 각 pyramid level 마다 $g$와 가장 가까운 $k$개의 anchor box를 구한다.
    - $g$의 center 그리고 anchor box의 center와의 L2 distance 이용
2. ㅁ

$$
a = 1
$$
$a=1$

## Conclusion

## Review

## Reference
- anchor-based detector (e.g., [RetinaNet](https://arxiv.org/abs/1708.02002))
- keypoint-based anchor-free detector (e.g., [CornerNet](https://arxiv.org/abs/1808.01244))
- center-based anchor-free detector (e.g., [FCOS](https://arxiv.org/abs/1904.01355))