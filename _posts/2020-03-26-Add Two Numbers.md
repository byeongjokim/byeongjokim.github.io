---
layout: post
title:  "LeetCode 02 - Add Two Numbers"
description: LeetCode - Top 100 Liked Questions
date:   2020-03-26 20:03:00 +0900
categories: Problems
use_math: true
---

reverse order로 수가 저장된 두개의 linked list가 주어졌을 때 두 수의 합을 구하는 문제이다.
마찬가지로 linked list로 return 해야한다.

ListNode라는 클래스로 노드 하나가 관리된다. val와 next라는 속성(attribute)를 갖고 있다.

2 -> 4 -> 3 은 342를 뜻한다.

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

l1 ListNode와 l2 ListNode를 일의 자리 부터 하나씩 더해줄 것이다. 두 수의 합을 10으로 나눈 나머지를 val로 가지는 ListNode 인스턴스를 result에 append 시키고, 몫은(0 혹은 1) next_plus 변수에 넣어 다음 자릿수에 함께 더해지도록 하였다.

```python
now_sum = now_1 + now_2 + next_plus

next_plus = now_sum // 10
result.append(ListNode(now_sum % 10))
```

l1와 l2는 다른 자릿수를 가질 수 있기 때문에, 한 쪽의 linked list가 끝나더라도 더해질 값을 0으로 두어 다른 linked list 값과 덧셈 계산이 이루어질 수 있도록 하였다. 계산이 끝난 후 자릿수가 넘어갈 수 있기 때문에 result에 몫을 val로 가진 LinstNode를 한번 더 append 시켰다. 그 후 result안의 ListNode 들을 연결 시켰다.

소스 코드는 아래와 같다. 첫번째 ListNode인 result[0]을 return 시켰다.

```python
def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
    i = 0
    next_plus = 0
    result = []    
    while (l1 or l2):
        now_1 = l1.val if l1 else 0
        now_2 = l2.val if l2 else 0
        
        now_sum = now_1 + now_2 + next_plus
        
        next_plus = now_sum // 10
        result.append(ListNode(now_sum % 10))
        
        l1 = l1.next if l1 else None
        l2 = l2.next if l2 else None

    if next_plus:
        result.append(ListNode(next_plus))
    
    for i in range(len(result)-1):
        result[i].next = result[i+1]
    
    return result[0]
```