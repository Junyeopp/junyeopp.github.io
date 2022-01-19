---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 구현
data: 2022-01-03 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# 구현
- 완전탐색: 모든 경우의 수를 다 계산하는 방법
- 시뮬레이션: 문제에서 제시한 내용을 한 단계씩 차례대로 직접 수행

# 문제풀이


## 상하좌우
```python
import sys
input = sys.stdin.readline

n = int(input())
routes = list(input().strip().split())

mapping = {
    "L": (0, -1),
    "R": (0, 1),
    "U": (-1, 0),
    "D": (1, 0),
}

x, y = 1, 1
for route in routes:
    dx, dy = mapping[route]
    if 1 <= x + dx <= n and 1 <= y + dy <= n:
        x += dx
        y += dy

print(f"{x} {y}")
```


## 시각
```python
import sys
input = sys.stdin.readline

n = int(input())

count = 0

for hour in range(0, n + 1):
    if "3" in str(hour):
        count += (6 * 10 * 6 * 10)
    else:  # 전체 경우의 수 - 3을 포함하지 않는 경우의 수
        count += (6 * 10 * 6 * 10) - (5 * 9 * 5 * 9)

print(count)
```


## 왕실의 나이트
```python
import sys
input = sys.stdin.readline

checklist = [
    (2, 1), (2, -1), (-2, 1), (-2, -1),
    (1, 2), (-1, 2), (1, -2), (-1, -2),
]

location = input().strip()
x = ord(location[0]) - 96
y = int(location[1])
count = 0

for check in checklist:
    dx, dy = check
    if 1 <= x + dx <= 8 and 1 <= y + dy <= 8:
        count += 1

print(count)
```


## 게임 개발
문제의 조건을 이해하기 조금 어려웠다.
```python
import sys
input = sys.stdin.readline

n, m = map(int, input().split())
x, y, cur_direction = map(int, input().split())
field = [list(map(int, input().split())) for _ in range(n)]

directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
field[x][y] = 2
count = 1

while True:
    print(x, y, cur_direction)
    dx, dy = directions[(cur_direction - 1) % 4]

    sub_count = 0
    for direction in directions:
        xx, yy = direction
        if field[x + xx][y + yy] != 0:
            sub_count += 1
    if sub_count == 4:
        if 0 <= x - dx < n and 0 <= y - dy < m and field[x - dy][y - dy] == 2:
            x -= dx
            y -= dy
            continue
        else:
            break

    if 0 <= x + dx < n and 0 <= y + dy < m:
        if field[x + dx][y + dy] == 0:
            cur_direction = (cur_direction - 1) % 4
            field[x + dx][y + dy] = 2
            count += 1
            x += dx
            y += dy
        else:
            cur_direction = (cur_direction - 1) % 4

print(count)
```


## [럭키 스트레이트](https://www.acmicpc.net/problem/18406)
```python
import sys
input = sys.stdin.readline

points = list(map(int, input().strip()))

if sum(points[:len(points)//2]) == sum(points[len(points)//2:]):
    print("LUCKY")
else:
    print("READY")
```


## 문자열 재정렬
```python
import sys
input = sys.stdin.readline

s = input().strip()

s_with_str = []
count = 0

for ch in s:
    if ch.isdigit():
        count += int(ch)
    else:
        s_with_str.append(ch)

print(f"{''.join(sorted(s_with_str))}{count}")
```


## [문자열 압축](https://programmers.co.kr/learn/courses/30/lessons/60057)
```python
def solution(s):
    compressed_len_min = 10e9
    for cut_length in range(1, len(s) // 2 + 2):
        compressed_len = 0
        prev_ch = ""
        count = 1

        for i in range(0, len(s), cut_length):
            ch = s[i:i+cut_length]
            if ch == prev_ch:
                count += 1
            else:
                if count > 1:  # 겹치는 부분이 있으면 숫자길이 추가
                    compressed_len += len(str(count))
                    count = 1
                compressed_len += len(prev_ch)
            prev_ch = ch
        if count > 1:
            compressed_len += len(str(count))
        compressed_len += len(prev_ch)

        compressed_len_min = min(compressed_len_min, compressed_len)
    return compressed_len_min
```

이렇게 `zip`을 이용하는 방법도 코드를 이해하기 쉽다는 점에서 괜찮아보인다.

```python
def compress(s, cut_length):
    words = [s[i:i+cut_length] for i in range(0, len(s), cut_length)]
    res = []
    cur_word = words[0]
    count = 1
    for a, b in zip(words, words[1:] + ['']):
        if a == b:
            count += 1
        else:
            res.append([cur_word, count])
            cur_word = b
            count = 1
    return sum(len(word) + (len(str(cnt)) if cnt > 1 else 0) for word, cnt in res)


def solution(s):
    return min([
        min(
            compress(s, cut_length)
            for cut_length in list(range(1, len(s) // 2 + 2))
        ),
        len(s)
    ])
```


## [자물쇠와 열쇠](https://programmers.co.kr/learn/courses/30/lessons/60059)

key를 lock의 전체 위치에 대어보면서 겹치는 부분의 숫자를 더해서 1이 아닌 숫자가 있는지 확인하면 됩니다.

key를 회전하는데 쓴 코드인 `key = list(zip(*key[::-1]))`를 잘 기억해둡시다!

```python
def check(key, lock, dx, dy):
    m = len(key)
    n = len(lock)

    for i in range(n):
        for j in range(n):
            key_i, key_j = i + dx, j + dy
            if 0 <= key_i < m and 0 <= key_j < m:
                if (lock[i][j] + key[key_i][key_j]) != 1:
                    return False
            else:
                if lock[i][j] != 1:
                    return False
    return True

def solution(key, lock):
    m = len(key)
    n = len(lock)

    for _ in range(4):
        for dx in range(-n + 1, m):
            for dy in range(-n + 1, m):
                if check(key, lock, dx, dy):
                    return True
        key = list(zip(*key[::-1]))  # key 돌리기

    return False
```


## [뱀](https://www.acmicpc.net/problem/3190)
```python
import sys
input = sys.stdin.readline

from collections import deque

n = int(input())
k = int(input())
apples = {tuple(map(int, input().split())) for _ in range(k)}
l = int(input())
dq_changes = deque([tuple(input().split()) for _ in range(l)])

directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
cur_direction = 0

x, y = 1, 1
cur_time = 0
dq_snake = deque([(x, y)])
next_change = dq_changes.popleft()

while True:
    dx, dy = directions[cur_direction]
    if (
        1 <= x + dx <= n
        and 1 <= y + dy <= n
        and (x + dx, y + dy) not in dq_snake
    ):  # 보드를 벗어나지 않고 자신과 겹치지 않으면 전진
        if (x + dx, y + dy) in apples:
            apples.discard((x + dx, y + dy))
            dq_snake.append((x + dx, y + dy))
        else:
            dq_snake.append((x + dx, y + dy))
            dq_snake.popleft()
        x, y = x + dx, y + dy
        cur_time += 1
    else:
        break

    if cur_time == int(next_change[0]):  # 방향 전환
        if next_change[1] == "L":
            cur_direction = (cur_direction - 1) % 4
        else:
            cur_direction = (cur_direction + 1) % 4
        if len(dq_changes) > 0:
            next_change = dq_changes.popleft()

print(cur_time + 1)
```


## [기둥과 보 설치](https://programmers.co.kr/learn/courses/30/lessons/60061)

중간에 하나를 잘못적어서 찾아내는데 시간이 많이 걸렸다. 문제를 상상하면서 조건문을 많이 설정하니 한 번에 풀지 못했을 때, 해결이 어려운 것 같다.

조건문에 주석을 계속 달면서 풀니까 나중에 보기가 쉬웠다. 주석달면서 풀기!

```python
def check(x, y, a, n):
    global ff    
    if a == 0:
        if not (0 <= x <= n and 0 <= y + 1 <= n):
            return False
        if {1, 2, 3}.intersection(ff[x][y]):
            return True
        else:
            return False
    elif a == 1:
        if not (0 <= x + 1 <= n and 0 <= y <= n):
            return False        
        if (
            {1}.intersection(ff[x][y])
            or {1}.intersection(ff[x + 1][y])
            or (
                {2}.intersection(ff[x + 1][y])
                and {3}.intersection(ff[x][y])
            )
        ):
            return True
        else:
            return False
        
def insert(x, y, a, n):
    global ff
    if check(x, y, a, n):
        if a == 0:  # 기둥
            ff[x][y].add(0)
            ff[x][y + 1].add(1)
        elif a == 1:  # 보
            ff[x][y].add(2)
            ff[x + 1][y].add(3)

def delete(x, y, a, n):
    global ff
    if a == 0:  # 기둥
        ff[x][y].remove(0)
        ff[x][y + 1].remove(1)
        if (
            0 <= y + 1 <= n
            and {0}.intersection(ff[x][y + 1])
            and not check(x,y+1,0,n)
        ):  # 위에 기둥이 있다면
            ff[x][y].add(0)
            ff[x][y + 1].add(1)
            return
        if (
            {2}.intersection(ff[x][y + 1])
            and not check(x,y + 1,1,n)
        ):  # 오른쪽 위에 보가 있다면
            ff[x][y].add(0)
            ff[x][y + 1].add(1)
            return
        if (
            {3}.intersection(ff[x][y + 1])
            and not check(x-1,y + 1,1,n)
        ):  # 왼쪽 위에 보가 있다면
            ff[x][y].add(0)
            ff[x][y + 1].add(1)
            return
    if a == 1:  # 보
        ff[x][y].remove(2)
        ff[x + 1][y].remove(3)
        if (
            {0}.intersection(ff[x][y])
            and not check(x,y,0,n)
        ):  # 위에 기둥이 있다면
            ff[x][y].add(2)
            ff[x + 1][y].add(3)
            return
        if (
            {0}.intersection(ff[x + 1][y])
            and not check(x + 1,y,0,n)
        ):  # 오른쪽 위에 기둥이 있다면
            ff[x][y].add(2)
            ff[x + 1][y].add(3)
            return
        if (
            {3}.intersection(ff[x][y])
            and not check(x-1,y,1,n)
        ):  # 왼쪽에 보가 있다면
            ff[x][y].add(2)
            ff[x + 1][y].add(3)
            return
        if (
            {2}.intersection(ff[x + 1][y])
            and not check(x+1,y,1,n)
        ):  # 오른쪽에 보가 있다면
            ff[x][y].add(2)
            ff[x + 1][y].add(3)
            return
        
def solution(n, build_frame):
    # 0: 기둥 시작
    # 1: 기둥 끝
    # 2: 보 시작
    # 3: 보 끝
    global ff
    ff = [[set() for _ in range(n + 1)] for _ in range(n + 1)]

    for x in range(n + 1):  # 바닥은 보의 끝부분으로 설정
        ff[x][0].add(1)
    
    for bf in build_frame:
        x, y, a, b = bf
        if b == 0:  # 삭제
            delete(x, y, a, n)
        elif b == 1:  # 설치
            insert(x, y, a, n)

    ret = list()
    for x in range(n + 1):
        for y in range(n + 1):
            if {0}.intersection(ff[x][y]):
                ret.append([x,y,0])
            if {2}.intersection(ff[x][y]):
                ret.append([x,y,1])
    ret.sort(key=lambda x: (x[0], x[1], x[2]))

    return ret
```

책에서 제시한 조금 더 짧은 코드를 따라해보았다. 책의 코드는 작업이 이루어질 때마다 전체상황을 판단하도록 되어있었다. 위의 코드와 비교했을 때 실행시간은 많게는 200배 가까이 차이가 발생하였다. 하지만 이 코드도 문제는 통과하였다.

더 간단하고 빠른 구조와 풀이를 찾기보다 시간복잡도와 제한시간을 고려해 최대한 간결한 코드를 작성하는 편이 문제를 더 빨리 풀기에는 좋은 것 같다. 문제 풀기 전에 허용가능한 시간복잡도 생각해보기!

```python
def possible(answer):
    for x, y, stuff in answer:
        if stuff == 0:  # 기둥
            if (
                y == 0  # 바닥이거나
                or [x - 1, y, 1] in answer  # 보의 오른쪽 끝이거나
                or [x, y, 1] in answer  # 보의 왼쪽 끝이거나
                or [x, y - 1, 0] in answer  # 기둥의 위거나
            ):
                continue
            return False
        elif stuff == 1:  # 보
            if (
                [x, y - 1, 0] in answer  # 왼쪽 밑에 기둥이 있거나
                or [x + 1, y - 1, 0] in answer  # 오른쪽 밑에 기둥이 있거나
                or (  # 왼쪽 오른쪽 모두에 보가 있거나
                    [x - 1, y, 1] in answer
                    and [x + 1, y, 1] in answer
                )
            ):
                continue
            return False
    return True


def solution(n, build_frame):
    answer = []
    for frame in build_frame:
        x, y, stuff, operate = frame
        if operate == 0:  # 삭제
            answer.remove([x, y, stuff])
            if not possible(answer):
                answer.append([x, y, stuff])
        if operate == 1:  # 설치
            answer.append([x, y, stuff])
            if not possible(answer):
                answer.remove([x, y, stuff])
    return sorted(answer)
```


## [치킨 배달](https://www.acmicpc.net/problem/15686)

`combinations(list(range(13)), 6) * 100 * 13 = 1716 * 100 * 13 = 2230800` 으로 전수조사를 해도 되겠다고 생각해서 전수조사했다.

```python
import sys
input = sys.stdin.readline

from itertools import combinations

n, m = map(int, input().split())
city = [list(map(int, input().split())) for _ in range(n)]

def get_dist(house, chicken):
    return abs(house[0] - chicken[0]) + abs(house[1] - chicken[1])

houses = list()
chikens = list()

# 집과 치킨집 리스트 구하기
for row in range(n):
    for col in range(n):
        if city[row][col] == 1:
            houses.append((row, col))
        elif city[row][col] == 2:
            chikens.append((row, col))

# 집에서 치킨집까지 거리를 모두 구하기
distances = [[get_dist(houses[i], chikens[j]) for i in range(len(houses))] for j in range(len(chikens))]

count_min = 10e9
for candidate in combinations(list(range(0, len(chikens))), m):
    count = 0
    for j in range(len(houses)):  # 각 집에서의 치킨 거리 더하기
        count += min(distances[i][j] for i in candidate)
    count_min = min(count_min, count)
print(count_min)
```


## [외벽 점검](https://programmers.co.kr/learn/courses/30/lessons/60062)

전수조사를 하면 최대 `len(list(permutations([1,2,3,4,5,6,7,8], 8))) * 8 * 15 * 8 = 38707200`
번의 연산이 필요하고 시간 내에 가능하다고 판단되어 전수조사를 진행했다.

```python
from itertools import permutations

def get_min_workers(new_intervals, perm):
    start_len = len(perm)
    ii = 0
    remaining_dist = perm.pop()
    while ii < len(new_intervals):
        if new_intervals[ii] <= remaining_dist:  # 더 갈 수 있으면
                remaining_dist -= new_intervals[ii]
                ii += 1
        else:  # 더 갈 수 없으면
            if ii + 1 == len(new_intervals):  # 마지막이면 종료
                ii += 1
                break
            if ii < len(new_intervals) and len(perm) != 0:
                remaining_dist = perm.pop()
                ii += 1
            else:
                break
    if ii == len(new_intervals):  # 모두 커버했으면
        return start_len - len(perm)
    else:
        return 101

def solution(n, weak, dist):
    intervals = [b - a for a, b in zip(weak[:-1], weak[1:])] + [(n - weak[-1]) + weak[0]]
    min_workers = 101
    
    for i in range(len(intervals)):
        new_intervals = intervals[i:] + intervals[:i]
        for perm in permutations(dist):
            new_min = get_min_workers(new_intervals, list(perm))
            if new_min < min_workers:
                min_workers = new_min
    if min_workers == 101:
        return -1
    else:
        return min_workers
```
