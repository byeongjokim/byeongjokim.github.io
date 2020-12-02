---
layout: post
title:  "LeetCode 11 - Container With Most Water"
description: LeetCode - Top 100 Liked Questions
date:   2020-04-05 20:03:00 +0900
categories: Etc
use_math: true
---
길이가 정수인 여러 막대를 x축 위에 세우고(interval은 1) 임의의 두 막대 사이를 물로 채웠을 때, 가장 많은 물을 채울 수 있는 두 막대를 찾는 문제이다.

```
Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```

양쪽 끝을 시작으로 해서 서서히 범위를 좁히는 방식으로 문제를 풀었다.

소스코드는 아래와 같다. 시간 복잡도는 O(n)이다.

```python
def maxArea(self, height: List[int]) -> int:
    answer = 0
    
    left = 0
    right = len(height) - 1
    while left != right:
        answer = max(answer, min(height[left], height[right]) * (right-left))

        if height[left] < height[right]:
            left = left + 1
        else:
            right = right - 1
            
    return answer
```