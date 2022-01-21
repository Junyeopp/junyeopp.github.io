---
title: 백준 단계별 - 그리디 알고리즘
data: 2022-01-07 20:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

# 문제풀이
## [동전 0](https://www.acmicpc.net/problem/11047) 
```python
import sys
input = sys.stdin.readline

n, k = map(int, input().split())
values = [int(input()) for _ in range(n)]

cur_value_index = n - 1
count = 0
while k > 0:
    if values[cur_value_index] <= k:
        count += k // values[cur_value_index]
        k = k % values[cur_value_index]
    cur_value_index -= 1
print(count)
```

## [회의실 배정](https://www.acmicpc.net/problem/1931)
일단 되는대로 해봤는데, 역시나 시간초과였다.
```python
import sys
input = sys.stdin.readline

n = int(input())
classes = sorted([list(map(int, input().split())) for _ in range(n)], key=lambda x: x[1])

greedy_list = [0 for _ in range(max(max(x) for x in classes) + 1)]

for cls in classes:
    fst_max = max(greedy_list[:cls[0] + 1])
    snd_max = max(greedy_list[:cls[1] + 1])

    if snd_max < fst_max + 1:
        greedy_list[cls[1]] = fst_max + 1

print(max(greedy_list))
```
인터넷을 참고해서 풀었는데 역시 간단한 거였다.
```python
import sys
input = sys.stdin.readline

n = int(input())
classes = sorted([list(map(int, input().split())) for _ in range(n)], key=lambda x: (x[1], x[0]))

count = 0
ended_time = 0

for cls in classes:
    if cls[0] >= ended_time:
        count += 1
        ended_time = cls[1]
print(count)
```


## [ATM](https://www.acmicpc.net/problem/11399)
시간이 짧게 걸리는 사람부터 돈을 뽑으면 된다.
```python
import sys
input = sys.stdin.readline

n = int(input())
times = sorted(list(map(int, input().split())))

print(sum([t * (n - i) for i, t in enumerate(times)]))
```

## [잃어버린 괄호](https://www.acmicpc.net/problem/1541)
"-"가 나온 이후부터는 괄호를 사용해서 식의 총합을 계속 줄일 수 있다.
```python
import sys
input = sys.stdin.readline

eqa = input().strip()
minus_index = eqa.find("-")

if minus_index == -1:
    print(sum(map(int, eqa.split("+"))))
else:
    print(
        sum(map(int, eqa[:minus_index].split("+")))
        - sum(map(int, eqa[minus_index + 1:].replace("-", "+").split("+")))
    )
```

## [주유소](https://www.acmicpc.net/problem/13305)
다음 도시까지 가는데 필요한 도로에서 쓸 연료는 그 도로 전의 도시들 중 리터당 가격이 가장 싼 곳에서 사면 된다.
```python
import sys
input = sys.stdin.readline

n_cities = int(input())
roads = list(map(int, input().split()))
cities = list(map(int, input().split()))

cost = 0
cur_min = cities[0]
for i in range(n_cities - 1):
    cost += cur_min * roads[i]
    cur_min = min(cur_min, cities[i + 1])

print(cost)
```