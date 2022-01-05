---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - Greedy
data: 2022-01-03 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# Greedy
현재 상황에서 가장 좋아 보이는 것만을 선택하는 알고리즘

# 문제풀이
## 큰 수의 법칙
```python
# time: 7mins
import sys
input = sys.stdin.readline
n, m, k = map(int, input().split())
nums = sorted(list(map(int, input().split())), reverse=True)

print(
    (k * nums[0] + nums[1]) * (m // (k + 1))
    + nums[0] * (m % (k + 1))
)
```

## 숫자 카드 게임
```python
# time: 4mins
import sys
input = sys.stdin.readline

n, m = map(int, input().split())
mat = [list(map(int, input().split())) for _ in range(n)]

print(max([min(row) for row in mat]))
```

## 1이 될 때까지
```python
# time: 6mins
import sys
input = sys.stdin.readline

n, k = map(int, input().split())
count = 0

while n != 1:
    if n % k == 0:
        n //= k
    else:
        n -= 1
    count += 1

print(count)
```

# 모험가 길드
```python
# time: 8mins
import sys
input = sys.stdin.readline

n = int(input())
advs = sorted(list(map(int, input().split())))
members = 1
count = 0

for adv in advs:
    if adv <= members:
        members = 1
        count += 1
    else:
        members += 1

print(count)
```

# 곱하기 혹은 더하기
```python
# time: 6mins
import sys
input = sys.stdin.readline

nums = list(map(int, str(input().strip())))

out = nums[0]
for i in range(1, len(nums)):
    if nums[i] <= 1 or out <= 1:
        out += nums[i]
    else:
        out *= nums[i]
print(out)
```

# 문자열 뒤집기
```python
# time: 8mins
import sys
input = sys.stdin.readline

chars = str(input().strip())
count = 0
for i in range(1, len(chars)):
    if chars[i] != chars[i - 1]:
        count += 1
if chars[0] == chars[-1]:
    print(count // 2)
else:
    print(count // 2 + 1)
```

# 만들 수 없는 금액
조금 어려웠다.. 다시 풀어보기!
```python
# time: 25mins
import sys
input = sys.stdin.readline

n = int(input())
nums = sorted(list(map(int, input().split())))

def get_impossible_min():
    prev_range = [0, nums[0]]
    next_range = [0, 0]
    for i in range(1, n):
        next_range = [prev_range[0] + nums[i], prev_range[1] + nums[i]]
        if prev_range[1] + 1 < next_range[0]:
            return prev_range[1] + 1
        else:
            prev_range = [0, next_range[1]]
        
    return next_range[1] + 1

print(get_impossible_min())
```
이걸 더 간단히하면 이렇게 된다.
```python
import sys
input = sys.stdin.readline

n = int(input())
nums = sorted(list(map(int, input().split())))

def get_impossible_min():
    possible_min = 0
    for num in nums:
        if possible_min + 1 < num:
            return possible_min + 1
        else:
            possible_min += num
    return possible_min + 1

print(get_impossible_min())
```

# 볼링공 고르기
전체 경우의 수에서 두 사람이 같은 무게를 고르는 경우의 수들을 빼주었다.
```python
# time: 13mins
import sys
input = sys.stdin.readline

n, m = map(int, input().split())
balls = list(map(int, input().split()))

counts = [0 for _ in range(m + 1)]
for ball in balls:
    counts[ball] += 1

out = n * (n - 1) // 2
for c in counts:
    if c > 1:
        out -= (c * (c - 1) // 2)

print(out)
```

# 무지의 먹방 라이브
시간이 많이 걸렸다.
```python
# time: > 60mins
def solution(food_times, k):
    sorted_food_times = sorted(food_times)

    prev_food_time = 0
    for i, food_time in enumerate(sorted_food_times):
        val = (food_time - prev_food_time) * (len(sorted_food_times) - i)
        if val > k:
            break
        else:
            k -= val
            prev_food_time = food_time

    remained_food = []
    for i, food_time in enumerate(food_times):
        if food_time - prev_food_time > 0:
            remained_food.append(i + 1)
    if len(remained_food) == 0:
        return -1
    else:
        return remained_food[k % len(remained_food)]
```

책에서 해설로 제시된 `heapq`를 이용한 풀이는 다음과 같다.
메모리와 실행시간 모두 위의 풀이가 더 좋았으며, heapq에서 작은 값은 계속 작은 위치를 유지함으로 굳이 heapq를 사용할 필요는 없는 것 같다.
```python
import heapq

def solution(food_times, k):
    if sum(food_times) <= k:
        return -1
    
    hq = []
    for food_time in enumerate(food_times):
        heapq.heappush(hq, (food_time, i + 1))
    
    sum_values = 0  # 먹기 위해 사용한 시간
    previous = 0  # 직전에 다 먹은 음식 시간
    length = len(food_times)  # 남은 음식 개수
    
    # sum_value + (현재의 음식 시간 - 이전 음식 시간) * 현재 음식 개수와 k 비교
    # 
    while sum_value + ((hq[0][0] - previous) * length) <= k:
        now = heapq.heappop(hq)[0]
        sum_value += (now - previous) * length
        length -= 1  # 다 먹은 음식 제외
        previous = now  # 이전 음식 시간 재설정
        
    # 남은 음식 중에서 몇 번쨰 음식인지 확인하여 출력
    result = sorted(hq, key=lambda x: x[1])
    return result[(k - sum_value) % length][1]
```
