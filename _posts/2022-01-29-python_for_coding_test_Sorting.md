---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 정렬
data: 2022-01-29 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# Sorting

## quick sort
이미 데이터가 정렬되어 있는 경우에는 느리게 동작합니다. 아래 코드는 책에서 소개된 일반적인 quick sort보다는 비효율적이지만 코드가 간결한 방법입니다.
```python
def quick_sort(array):

    if len(array) <= 1:
        return array

    pivot = array[0]
    tail = array[1:]

    left_side = [x for x in tail if x <= pivot]
    right_side = [x for x in tail if x > pivot]

    return quick_sort(left_side) + [pivot] + quick_sort(right_side)
```

## count sort
데이터 크기의 범위가 제한되어 있을 떄 사용가능하며 빠르게 동작합니다. 동일한 값을 가지는 데이터가 여러 개일 때 효과적입니다.

## sort()
파이썬 내장 정렬 라이브러니는 최악의 경우에도 시간복잡도 O(N log N)을 보장해줍니다.


## [국영수](https://www.acmicpc.net/problem/10825)
```python
import sys
input = sys.stdin.readline

n = int(input())
scores = [tuple() for _ in range(n)]
for i in range(n):
    name, kor, eng, math = input().split()
    scores[i] = (str(name), int(kor), int(eng), int(math))
scores.sort(key=lambda x: (-x[1], x[2], -x[3], x[0]))

for v in scores:
    print(v[0])
```


## [안테나](https://www.acmicpc.net/problem/18310)
```python
import sys
input = sys.stdin.readline

n = int(input())
houses = sorted(list(map(int, input().split())))

print(houses[(len(houses) - 1) // 2])
```


## [실패율](https://programmers.co.kr/learn/courses/30/lessons/42889)
```python
def solution(N, stages):
    stages.sort(reverse=True)
    answer = []
    
    n_users = len(stages)
    cur_num = [0 for _ in range(N + 2)]
    try_num = [0 for _ in range(N + 2)]
    fail_rate = [[0, i] for i in range(N + 2)]
    for i, s in enumerate(stages):
        cur_num[s] += 1
        try_num[s] = i + 1
    for i in range(N, 0, -1):
        if try_num[i] == 0:
            try_num[i] = try_num[i + 1]
    for i in range(N + 1):
        if try_num[i] != 0:
            fail_rate[i][0] = cur_num[i] / try_num[i]
    
    return [v[1] for v in sorted(fail_rate[1:N + 1], key=lambda x: (-x[0], x[1]))]
```


## [카드 정렬하기](https://www.acmicpc.net/problem/1715)
```python
import sys
input = sys.stdin.readline

import heapq

n = int(input())
cards = []
for _ in range(n):
    heapq.heappush(cards, int(input()))

count = 0
while len(cards) >= 2:
    a = heapq.heappop(cards)
    b = heapq.heappop(cards)
    count += a + b
    heapq.heappush(cards, a + b)

print(count)
```
