---
layout: post
title:  "Relation Networks for Object Detection"
description: 저자는 딥러닝이 유행하기 이전의 object relation을 이용하여 이 문제를 해결하고자 하였다. 그러나 보통 object는 각기 다른 위치, 다른 크기, 다른 종류 그리고 다른 이미지로 검출 되기 때문에 NLP에서 성공을 얻은 attention model (Attention is all you need)을 이용하였다.
date:   2020-02-22 17:20:00 +0900
categories: Paper
use_math: true
---
$$f(x) = \frac{\left | x - x_2 \right |}{\left | x - x_1 \right | + \left | x - x_2 \right |}f(x_1) + \frac{\left | x - x_1 \right |}{\left | x - x_1 \right | + \left | x - x_2 \right |}f(x_2)$$