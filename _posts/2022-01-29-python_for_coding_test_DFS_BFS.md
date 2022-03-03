---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - DFS, BFS
data: 2022-01-29 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# DFS, BFS


## [특정 거리의 도시 찾기](https://www.acmicpc.net/problem/18352)
```python
import sys
input = sys.stdin.readline

n_cities, n_roads, target_distance, start_city = map(int, input().split())
roads = [[] for _ in range(n_cities + 1)]
distances = [1e9 for _ in range(n_cities + 1)]

for _ in range(n_roads):
    a, b = map(int, input().split())
    roads[a].append(b)

from collections import deque

def bfs(roads, distances, start_city):
    dq = deque([start_city])
    while dq:
        now_city = dq.popleft()
        for next_city in roads[now_city]:
            dist = distances[now_city] + 1
            if dist < distances[next_city]:
                distances[next_city] = dist
                dq.append(next_city)
    return distances

distances[start_city] = 0
final_distances = dfs(roads, distances, start_city)
ret = sorted([i for i in range(n_cities + 1) if final_distances[i] == target_distance])

if len(ret) == 0:
    print(-1)
else:
    for r in ret:
        print(r)
```


## [연구소](https://www.acmicpc.net/problem/14502)
```python
import sys
input = sys.stdin.readline

from itertools import combinations
from collections import deque
from copy import deepcopy

n, m = map(int, input().split())
lab = [list(map(int, input().split())) for _ in range(n)]

# n, m이 8이하이므로 lab에서 3개를 선택하는 경우의 수는 C(64, 3) = 41664이고
# 각각의 경우에 안전 영역의 크기를 구하는 시간복잡도는 O(N^2)이므로 전수조사를 해도 된다.


def bfs(lab, viruses):
    dq = deque(viruses)
    while dq:
        x, y = dq.popleft()
        for dx, dy in zip([-1, 1, 0, 0], [0, 0, -1, 1]):
            if 0 <= x + dx < n and 0 <= y + dy < m:
                if lab[x + dx][y + dy] == 0:
                    lab[x + dx][y + dy] = 2
                    dq.append((x + dx, y + dy))
    return lab


def get_area(lab):
    count = 0
    for i in range(n):
        for j in range(m):
            if lab[i][j] == 0:
                count += 1
    return count


# 벽을 세울 후보지 & 바이러스 찾기
wall_candidates = list()
viruses = list()
for i in range(n):
    for j in range(m):
        if lab[i][j] == 0:
            wall_candidates.append((i, j))
        if lab[i][j] == 2:
            viruses.append((i, j))

max_area = 0
for walls in combinations(wall_candidates, 3):
    lab_copy = deepcopy(lab)
    for wall in walls:
        i, j = wall
        lab_copy[i][j] = 1
    lab_copy = bfs(lab_copy, viruses)
    max_area = max(max_area, get_area(lab_copy))

print(max_area)
```


## [경쟁적 전염](https://www.acmicpc.net/problem/18405)
```python
import sys
input = sys.stdin.readline

from collections import deque

n, k = map(int, input().split())
lab = [list(map(int, input().split())) for _ in range(n)]
s, target_x, target_y = map(int, input().split())

# 바이러스 위치 찾기
viruses = list()
for i in range(n):
    for j in range(n):
        if lab[i][j] != 0:
            viruses.append((lab[i][j], i, j, 0))
viruses.sort()

dq = deque(viruses)
while dq:
    virus_num, x, y, time = dq.popleft()
    if time >= s:
        break
    for dx, dy in ((-1, 0), (1, 0), (0, -1), (0, 1)):  # 상하좌우
        if 0 <= x + dx < n and 0 <= y + dy < n:
            if lab[x + dx][y + dy] == 0:
                lab[x + dx][y + dy] = virus_num
                dq.append((virus_num, x + dx, y + dy, time + 1))

print(lab[target_x - 1][target_y - 1])
```


## [괄호 변환](https://programmers.co.kr/learn/courses/30/lessons/60058)
```python
def split_uv(w):
    count_left = 0
    count_right = 0
    for i in range(len(w)):
        if w[i] == "(":
            count_left += 1
        elif w[i] == ")":
            count_right += 1
        if count_left == count_right:
            return w[:i + 1], w[i + 1:]
        
def is_correct(u):
    count = 0 
    for v in u:
        if v == "(":
            count += 1
        elif v == ")":
            count -= 1
        if count < 0:
            return False
    return True

def solution(w):
    if len(w) == 0:
        return w
    
    u, v = split_uv(w)
    if is_correct(u):
        return u + solution(v)
    else:
        reversed_u = ""
        for ss in u[1:-1]:
            if ss == "(":
                reversed_u += ")"
            else:
                reversed_u += "("
        return "(" + solution(v) + ")" + reversed_u
```


## [연산자 끼워 넣기](https://www.acmicpc.net/problem/14888)
`set`을 이용해서 중복을 제거해주지 않으면 시간초과가 발생한다. 백준기준 596ms가 걸렸다.
```python
import sys
input = sys.stdin.readline

# 10! = 3628800이고 시간 제한은 2초이므로 전수조사로 진행
from itertools import permutations

n = int(input())
nums = list(map(int, input().split()))
n_plus, n_minus, n_multiply, n_divide = map(int, input().split())

operators = (
    ["+"] * n_plus
    + ["-"] * n_minus
    + ["*"] * n_multiply
    + ["/"] * n_divide
)

def calculate(nums, ops):
    ret = nums[0]
    for i in range(len(ops)):
        if ops[i] == "+":
            ret += nums[i + 1]
        elif ops[i] == "-":
            ret -= nums[i + 1]
        elif ops[i] == "*":
            ret *= nums[i + 1]
        elif ops[i] == "/":
            if ret < 0:
                ret = -((-ret) // nums[i + 1])
            else:
                ret = ret // nums[i + 1]
    return ret

max_value = int(-1e9)
min_value = int(1e9)
for ops in set(permutations(operators, len(operators))):
    new_value = calculate(nums, list(ops))
    min_value = min(min_value, new_value)
    max_value = max(max_value, new_value)

print(max_value)
print(min_value)
```

dfs를 사용하면 중복되는 계산을 줄일 수 있고 시간도 더 단축된다.(백준기준 104ms)
```python
import sys
input = sys.stdin.readline

n = int(input())
nums = list(map(int, input().split()))
n_plus, n_minus, n_multiply, n_divide = map(int, input().split())


def dfs(depth, new_value):
    global min_value, max_value
    global n_plus, n_minus, n_multiply, n_divide
    if depth == n:
        min_value = min(min_value, new_value)
        max_value = max(max_value, new_value)
    else:
        if n_plus > 0:
            n_plus -= 1
            dfs(depth + 1, new_value + nums[depth])
            n_plus += 1
        if n_minus > 0:
            n_minus -= 1
            dfs(depth + 1, new_value - nums[depth])
            n_minus += 1
        if n_multiply > 0:
            n_multiply -= 1
            dfs(depth + 1, new_value * nums[depth])
            n_multiply += 1
        if n_divide > 0 :
            n_divide -= 1
            if new_value < 0:
                dfs(depth + 1, -((-new_value) // nums[depth]))
            else:
                dfs(depth + 1, new_value // nums[depth])
            n_divide += 1


max_value = int(-1e9)
min_value = int(1e9)
dfs(1, nums[0])

print(max_value)
print(min_value)
```


## [감시 피하기](https://www.acmicpc.net/problem/18428)
```python
import sys
input = sys.stdin.readline

from itertools import permutations

n = int(input())
field = [list(map(str, input().split())) for _ in range(n)]

# T의 위치 & 빈칸 위치 구하기
ts = list()
candidates = list()
for i in range(n):
    for j in range(n):
        if field[i][j] == "T":
            ts.append((i, j))
        if field[i][j] == "X":
            candidates.append((i, j))


def check():
    global field, ts
    for x, y in ts:
        for new_x in range(x + 1, n, 1):
            if field[new_x][y] == "O":
                break
            elif field[new_x][y] == "S":
                return False
        for new_x in range(x - 1, -1, -1):
            if field[new_x][y] == "O":
                break
            elif field[new_x][y] == "S":
                return False
        for new_y in range(y + 1, n, 1):
            if field[x][new_y] == "O":
                break
            elif field[x][new_y] == "S":
                return False
        for new_y in range(y - 1, -1, -1):
            if field[x][new_y] == "O":
                break
            elif field[x][new_y] == "S":
                return False
    return True


def try_candidates():
    for candidate in permutations(candidates, 3):
        for i, j in candidate:
            field[i][j] = "O"
        if check():
            print("YES")
            return
        for i, j in candidate:
            field[i][j] = "X"
    print("NO")
    return


try_candidates()
```


## [인구 이동](https://www.acmicpc.net/problem/16234)
틀렸는데, 반례를 못찾겠다.
```python
import sys
input = sys.stdin.readline

from collections import deque

n, l, r = map(int, input().split())
a = [list(map(int, input().split())) for _ in range(n)]


def move(x, y, visited):
    if visited[x][y]:
        return 0

    uni = [(x, y)]
    uni_sum = a[x][y]

    dq = deque([(x, y)])
    while dq:
        x, y = dq.popleft()
        if visited[x][y]:
            continue
        else:
            visited[x][y] = True
        for dx, dy in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            if (
                0 <= x + dx < n
                and 0 <= y + dy < n
                and not visited[x + dx][y + dy]
                and l <= abs(a[x][y] - a[x + dx][y + dy]) <= r
            ):
                uni_sum += a[x + dx][y + dy]
                uni.append((x + dx, y + dy))
                dq.append((x + dx, y + dy))
    for i, j in uni:
        a[i][j] = uni_sum // len(uni)
    return len(uni) - 1


time = 0
while True:
    is_moved = 0
    visited = [[False for _ in range(n)] for _ in range(n)]
    for x in range(n):
        for y in range(n):
            is_moved += move(x, y, visited)
    if is_moved == 0:
        break
    time += 1

print(time)
```
