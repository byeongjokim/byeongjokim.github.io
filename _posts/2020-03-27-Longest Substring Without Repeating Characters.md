---
layout: post
title:  "LeetCode 03 - Longest Substring Without Repeating Characters"
description: LeetCode - Top 100 Liked Questions
date:   2020-03-27 20:03:00 +0900
categories: Etc
use_math: true
---
문자열이 주어지면 그 중에서 가장 긴 substring을 찾는 문제이다. 밑 예제에도 있듯이 subsequence가 아닌 substring을 찾아야 한다.

```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 

Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.

Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

문자열을 처음부터 끝까지 돌면서, 들어오는 문자열과 같은 문자열을 index로 찾았다. 파이썬은 찾는 문자열이 없을 경우 index method가 ValueError를 내기 때문에 try/except 처리를 하였다.

```python
index = substring.index(now)
substring = substring[index+1:] + now
```

index를 찾으면 (중복 문자열이 있으면) index + 1 이후로 slice 한 후, 새 문자열을 이어 붙였다. 반면에, ValueError를 일으키면, 중복이 없는 것으로 파악하고 그대로 이어 붙였다.

소스코드는 아래와 같다. 처음 제출할 때는 Runtime, Memomry 모두 비교적 좋은 성능 편에 속했지만 (accepted 된 파이썬 코드에 비해), 제출할 때마다 Memory의 차이가 커서.. 잘 짠건지는 모르겠다..

```python
def lengthOfLongestSubstring(self, s: str) -> int:
    result = 0
    substring = ""
    
    for now in s:
        try:
            index = substring.index(now)
            substring = substring[index+1:] + now
        except ValueError:
            substring = substring + now
        
        if (len(substring) > result):
                result = len(substring)
        
    return result
```