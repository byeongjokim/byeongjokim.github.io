---
layout: post
title:  "LeetCode 05 - Longest Palindromic Substring"
description: LeetCode - Top 100 Liked Questions
date:   2020-03-31 20:03:00 +0900
categories: Problems
use_math: true
---

string s가 주어졌을 경우, s의 substring 중 가장 긴 palindromic substring을 찾는 문제이다.

```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.

Input: "cbbd"
Output: "bb"
```

s의 i+1 ~ j-1 substring이 palindromic 이고, s[i] == s[j] 이면, s의 i ~ j 또한 palindromic 이다 라는 점을 이용하였다.

하지만 aba/abba와 같이 center가 홀수/짝수 인 경우를 고려해서 두 경우로 탐색을 하였다.

```python
length1 = self.extendPalindrome(s, i, i)
length2 = self.extendPalindrome(s, i, i+1)
```

extendPalindrome method는 중앙부터 문자열 하나씩 양쪽으로 확장해가며 마지막 길이를 return 해주는 method이다. 

전체 코드는 아래와 같다.

```python
def longestPalindrome(self, s: str) -> str:
    start = 0
    end = 0
    
    for i in range(len(s)):
        length1 = self.extendPalindrome(s, i, i)
        length2 = self.extendPalindrome(s, i, i+1)
        
        now_max = max(length1, length2)
        
        if (now_max > end - start + 1):
            start = i - (now_max-1)//2
            end = i + now_max//2
    
    return s[start:end+1]

def extendPalindrome(self, s: str, left: int, right: int) -> int:
    L = left
    R = right
    
    while(L>=0 and R<len(s)):
        if (s[L] == s[R]):
            L = L - 1
            R = R + 1
        else:
            break
    
    return R - L + 1 - 2
```