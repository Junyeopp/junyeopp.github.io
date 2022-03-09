---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 그래프 알고리즘
data: 2022-01-30 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# Graph Algorithms

- Union-Find

    경로 압축을 통해 시간 복잡도를 개선할 수 있다.

    ```python
    def find_parent(parent, x):
        if parent[x] != x:
            parent[x] = find_parent(parent, parent[x])
        return parent[x]


    def union_parent(parent, a, b):
        a = find_parent(parent, a)
        b = find_parent(parent, b)
        if a < b:
            parent[b] = a
        else:
            parent[a] = b
    ```

    무방향 그래프내에서의 사이클을 판별할 때 사용할 수 있다.
    (방향 그래프에서의 사이클 여부는 DFS를 이용해서 판별할 수 있다.)

- 신장 트리, Spanning Tree

    신장 트리란 하나의 그래프가 있을 때 모든 노드를 포함하면서 사이클이 존재하지 않는 부분 그래프를 의미한다.

    - Kruskal Algorithm

        **최소 신장 트리 알고리즘**(신장 트리 중에서 최소 비용으로 만들 수 있는 신장 트리를 찾는 알고리즘)

        그리디 알고리즘으로 분류됨.

        1. 간선 데이터를 비용에 따라 오름차순으로 정렬
        2. 모든 간선에 대해 간선이 사이클을 발생시키지 않는다면 최소 신장 트리에 포함시킴

        정렬에 가장 시간이 오래 걸리기 때문에 E개의 간선이 있을 때, 시간복잡도는 O(E log E)이다.

- Topology Algorithm

    방향 그래프의 모든 노드를 방향성에 거스르지 않도록 순서대로 나열하는 것

    예시: 선수과목을 고려한 학습 순서 결정

    1. 진입차수(indegree, 특정 노드로 들어오는 간선의 개수)가 0인 노드를 큐에 넣음
    2. 큐가 빌 때까지, 큐에서 원소를 꺼내 해당 노드에서 출발하는 간선을 그래프에서 제거 & 진입차수가 0이 된 노드를 큐에 넣음

    ```python
    from collections import deque

    indegree = 진입차수들

    def topology_sort():
        result = []
        q = deque()

        for i in range(1, v + 1):
            if indegree[i] == 0:
                q.append(i)

        while q:
            now = q.popleft()
            result.append(now)

            for i in graph[now]:
                indegree[i] -= 1
                if indegree[i] == 0:
                    q.append(i)

        return result
    ```


## [행성 터널](https://www.acmicpc.net/problem/2887)
처음에는 모든 행성 사이의 거리를 체크하였고, n(n-1)/2 개를 비교했어야 했다. 이 경우 메모리 초과가 되었다.
```python
import sys
input = sys.stdin.readline

n = int(input())
planets = [list(map(int, input().split())) for _ in range(n)]


def find_parent(x):
    global parents
    if parents[x] != x:
        parents[x] = find_parent(parents[x])
    return parents[x]


def union_parent(a, b):
    global parents
    a = find_parent(a)
    b = find_parent(b)
    if a < b:
        parents[b] = a
    else:
        parents[a] = b


def get_dist(a, b):
    ax, ay, az = a
    bx, by, bz = b
    return min(abs(ax - bx), abs(ay - by), abs(az - bz))


parents = [i for i in range(n)]
distances = []
for i in range(n):
    for j in range(i + 1, n):
        distances.append((i, j, get_dist(planets[i], planets[j])))

distances.sort(key=lambda x: x[2])

total_cost = 0
for dist in distances:
    a, b, cost = dist
    if find_parent(a) != find_parent(b):
        total_cost += cost
        union_parent(a, b)

print(total_cost)
```

행성 사이의 거리는 x, y, z의 차이 중 가장 작은 것이다. 따라서 x, y, z 각각을 기준을 정렬을 했을 때 서로 이웃간의 거리들 이상으로 통로가 연결될 일을 없다.

그래서 총 3*(n-1)개만 비교하면 된다.

```python
import sys
input = sys.stdin.readline

n = int(input())
planets = [list(map(int, input().split())) + [idx] for idx in range(n)]


def find_parent(x):
    global parents
    if parents[x] != x:
        parents[x] = find_parent(parents[x])
    return parents[x]


def union_parent(a, b):
    global parents
    a = find_parent(a)
    b = find_parent(b)
    if a < b:
        parents[b] = a
    else:
        parents[a] = b


def get_dist(a, b):
    ax, ay, az, aidx = a
    bx, by, bz, aidx = b
    return min(abs(ax - bx), abs(ay - by), abs(az - bz))


parents = [i for i in range(n)]
distances = []

# x, y, z 별로 정렬해서, 각각 가장 작은 distance n - 1개를 distances에 추가
# 총 3(n - 1)개의 distance만 확인하면 됨
for idx in range(3):
    planets.sort(key=lambda x: x[idx])
    for i in range(n - 1):
        distances.append((planets[i][3], planets[i + 1][3], get_dist(planets[i], planets[i + 1])))

distances.sort(key=lambda x: x[2])

total_cost = 0
for dist in distances:
    a, b, cost = dist
    if find_parent(a) != find_parent(b):
        total_cost += cost
        union_parent(a, b)

print(total_cost)
```


## [최종 순위](https://www.acmicpc.net/problem/3665)
```python
import sys
from collections import deque
input = sys.stdin.readline


def topology_sort():
    global n
    global indegrees
    result = []
    dq = deque()

    for i in range(1, n + 1):
        if indegrees[i] == 0:
            dq.append(i)
    if len(dq) == 0:
        return result

    while dq:
        now = dq.popleft()
        result.append(now)

        for i in orders[now]:
            indegrees[i] -= 1
            if indegrees[i] == 0:
                dq.append(i)
        if len(dq) > 1:
            return "?"
    return result


ntests = int(input())

for _ in range(ntests):
    n = int(input())
    last_ranks = list(map(int, input().split()))
    m = int(input())

    changes = dict()
    for _ in range(m):
        a, b = map(int, input().split())
        if a in changes.keys():
            changes[a].append(b)
        else:
            changes[a] = [b]
        if b in changes.keys():
            changes[b].append(a)
        else:
            changes[b] = [a]

    orders = [[] for _ in range(n + 1)]
    for i in range(n - 1):
        for j in range(i + 1, n):
            if last_ranks[j] in changes.keys() and last_ranks[i] in changes[last_ranks[j]]:
                orders[last_ranks[j]].append(last_ranks[i])
            else:
                orders[last_ranks[i]].append(last_ranks[j])

    indegrees = [0 for _ in range(n + 1)]
    for row in orders:
        for j in row:
            indegrees[j] += 1

    ret = topology_sort()
    if isinstance(ret, str):
        print(ret)
    elif len(ret) != n:
        print("IMPOSSIBLE")
    else:
        print(" ".join(map(str, ret)))
```
