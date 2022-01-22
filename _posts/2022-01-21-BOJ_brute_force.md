---
title: 백준 단계별 - 브루트 포스 알고리즘
data: 2022-01-21 22:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

# 문제풀이
## [블랙잭](https://www.acmicpc.net/problem/2798)
```python
import sys
input = sys.stdin.readline

from itertools import combinations
n, m = map(int, input().split())
cards = list(map(int, input().split()))

nearest_max = 0
for subset in combinations(cards, 3):  # 모든 경우의 수 탐색
    new_sum = sum(subset)
    if new_sum <= m and new_sum > nearest_max:
        nearest_max = new_sum

print(nearest_max)
```


## [분해합](https://www.acmicpc.net/problem/2231)
```python
import sys
input = sys.stdin.readline

n = int(input())

def find_generator(n):  # n이 1000000이하 이므로 전수조사
    for m in range(n):
        if (m + sum(map(int, str(m)))) == n:
            return m
    return 0

print(find_generator(n))
```

전수조사 범위를 조금 더 제한시키면 실행시간이 더 줄어든다. (백준 기준 1100ms에서 72ms로 줄어듬)

```python
import sys
input = sys.stdin.readline

n = int(input())

def find_generator(n):  # n이 1000000이하 이므로 전수조사
    for m in range(max(1, n - len(str(n))*9), n):
        if (m + sum(map(int, str(m)))) == n:
            return m
    return 0

print(find_generator(n))
```


## [덩치](https://www.acmicpc.net/problem/7568)
```python
import sys
input = sys.stdin.readline

n = int(input())
dungchis = [tuple(map(int, input().split())) for _ in range(n)]


# 전수조사를 할 경우 시간복잡도가 O(n^2)이고 n의 최대값이 50이므로 전수조사를 하였다.

for dch_target in dungchis:
    rank = 0
    for dch in dungchis:
        if dch_target[0] < dch[0] and dch_target[1] < dch[1]:
            rank += 1
    print(rank + 1, end=" ")
```


## [체스판 다시 칠하기](https://www.acmicpc.net/problem/1018)
```python
import sys
input = sys.stdin.readline

n, m = map(int, input().split())
board = [input().strip() for _ in range(n)]

# 전수조사하였을 때 시간복잡도는 O(n^4), n은 50이하이고 50^4 = 6250000이다. 그래서 전수조사를 진행했다.

piece_white = "WBWBWBWB"
piece_black = "BWBWBWBW"
board_white = ([piece_white] + [piece_black]) * 4
board_black = ([piece_black] + [piece_white]) * 4

brush_min = 8*8

for x_start in range(n - 8 + 1):
    for y_start in range(m - 8 + 1):
        brush_white = 0
        brush_black = 0
        for dx in range(8):
            for dy in range(8):
                if board[x_start + dx][y_start + dy] != board_white[dx][dy]:
                    brush_white += 1
                if board[x_start + dx][y_start + dy] != board_black[dx][dy]:
                    brush_black += 1
        brush_min = min(brush_min, brush_white, brush_black)
print(brush_min)
```


## [영화감독 숌](https://www.acmicpc.net/problem/1436)

시간을 조금 걸리지만(백준 기준 1352ms), 문제를 푸는 시간이 짧아서 좋은 것 같다.
```python
import sys
input = sys.stdin.readline

nth = int(input())

# 6660000은 10000번째로 작은 종말의 숫자보다 크기 때문에 최대 6660000까지만 전수조사를 하면 문제를 풀 수 있다.


def is_apocalypse(num):
    if str(num).find("666") >= 0:
        return True
    else:
        return False


num = 0
count = 0
while True:
    if is_apocalypse(num):
        count += 1
    if count == nth:
        break
    num += 1
print(num)
```
