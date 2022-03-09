---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 다이나믹 프로그래밍
data: 2022-01-30 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# Dynamic Programming
한 번 계산된 문제는 다시 계산하지 않도록 하는 알고리즘


## [정수삼각형](https://www.acmicpc.net/problem/1932)
```python
import sys
input = sys.stdin.readline

n = int(input())
triangle = [list(map(int, input().split())) for _ in range(n)]
maxangle = triangle[:]

for i in range(1, n):
    for j in range(0, i):
        if j - 1 < 0:
            maxangle[i][j] = maxangle[i - 1][j] + triangle[i][j]
        else:
            if i - 1 < j:
                maxangle[i][j] = maxangle[i - 1][j - 1] + triangle[i][j]
            else:
                maxangle[i][j] = max(maxangle[i - 1][j - 1], maxangle[i - 1][j]) + triangle[i][j]
print(max(maxangle[-1]))
```


## [퇴사](https://www.acmicpc.net/problem/14501)
```python
import sys
input = sys.stdin.readline

n = int(input())
counsels = list()
max_pay = [0 for _ in range(n + 1)]

for start in range(1, n + 1):
    duration, pay = map(int, input().split())
    end = start + duration - 1
    if end <= n:
        counsels.append((start, end, pay))

# 끝나는 날짜 순으로 정렬
counsels.sort(key=lambda x: x[1])

for counsel in counsels:
    start, end, pay = counsel
    max_pay[start - 1] = max(max_pay[:start])
    max_pay[end] = max(max_pay[end], max_pay[start - 1] + pay)

print(max(max_pay))
```


## [병사 배치하기](https://www.acmicpc.net/problem/18353)
```python
import sys
input = sys.stdin.readline

n = int(input())
soldiers = list(map(int, input().split()))
LDS = [0 for _ in range(n)]

for i in range(n):
    LDS[i] = max([LDS[v] for v in range(i + 1) if soldiers[v] > soldiers[i]], default=0) + 1

print(n - max(LDS))
```
