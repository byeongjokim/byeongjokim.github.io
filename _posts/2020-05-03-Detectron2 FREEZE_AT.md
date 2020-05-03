---
layout: post
title:  "PyTorch - Detectron2, when train using scratch"
description: Trial and Error
date:   2020-05-03 20:03:00 +0900
categories: Development
use_math: true
---

Facebook Research에서 제공하는 detection 관련 toolkit [Detectron2](https://github.com/facebookresearch/detectron2)을 사용하고 있다.

pretrained weight 없이 scratch dataset으로 학습을 진행하고 있었는데, 계속되는 학습에도 불구하고 성능이 너무 낮게 나와서 코드를 하나씩 살펴 보기 시작하였다.

그러던 와중, 내 2주일의 시간이 무의미해진 코드 한줄을 발견해서 포스트로 남긴다.

[Code](https://github.com/facebookresearch/detectron2/blob/master/detectron2/config/defaults.py#L131) 위 코드의 131번째 line을 보면 

```python
_C.MODEL.BACKBONE.FREEZE_AT = 2
```

기존 backbone(나의 경우 resnet50)의 파라미터 업데이트를 freeze 할 것인지에 대한 flag를 설정할 수 있었다.

여태까지 이것을 2로 두고 학습을 해서 성능이 항상 0.05AP를 기록하였던 것..

위 FREEZE_AT 파라미터는 아래의 freeze 함수를 통해 freeze 될 파라미터가 결정된다. 

```python
def freeze(self, freeze_at=0):
    """
    Freeze the first several stages of the ResNet. Commonly used in
    fine-tuning.
    Args:
        freeze_at (int): number of stem and stages to freeze.
            `1` means freezing the stem. `2` means freezing the stem and
            the first stage, etc.
    Returns:
        nn.Module: this ResNet itself
    """
    if freeze_at >= 1:
        self.stem.freeze()
    for idx, (stage, _) in enumerate(self.stages_and_names, start=2):
        if freeze_at >= idx:
            for block in stage.children():
                block.freeze()
    return self
```

즉 위 freeze_at flag를 0으로 해야 기존 backbone 모델도 함께 학습이 될 수 있다.

detectron 기반 모델 ([ATSS official](https://github.com/sfzhang15/ATSS/blob/0f66e2812e4fcbb267fa2cf0aab012ee3ec981ac/atss_core/config/defaults.py#L99) 등)에 모두 포함 되어있는 파라미터 이므로 항상 확인을 해야겠다.