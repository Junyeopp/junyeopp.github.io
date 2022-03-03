---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 최단 경로
data: 2022-01-30 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# Sortest path
- Dijkstra

    방문하지 않은 노드 중에서 갖아 최단 거리가 짧은 노드를 선택하여 진행한다. 한 단계에 하나의 노드에 대한 최단 거리를 확정하면서 진행된다. 그리디 알고리즘.

    각 단계에서 최단 거리가 짧은 노드를 선택할 때 최소 힙 라이브러인 heapq를 이용해 시간복잡도를 O(E log V)로 줄인다.

    ```python
    import heapq
    # graph = edges
    # distance = 각 노드까지의 최단 거리


    def dijkstra(start_node):
        q = []

        heapq.heappush(q, (0, start_node))
        distance[start_node] = 0

        while q:
            dist, now = heapq.heappop(q)
            if distance[now] < dist:
                continue
            else:
                for i in graph[now]:
                    cost = dist + i[1]
                    if cost < distance[i[0]]:
                        distance[i[0]] = cost
                        heapq.heappush(q, (cost, i[0]))
    ```

- Floyd-Warchall

    **모든** 지점에서 다른 모든 지점까지의 최단 경로를 모두 구해야 하는 경우에 사용한다. 다이나믹 프로그래밍.

    n개의 타겟 노드에 대해서 타겟 노드를 지나가는 최단 거리를 갱신(각각 시간복잡도가 O(N^2))한다. 시간복잡도가 O(N^3)이다.

    ```python
    # 모든 k, a, b에 대해서 a -> b의 최단 경로와 a -> k -> b의 최단 거리를 비교
    for k in range(1, n + 1):
        for a in range(1, n + 1):
            for b in range(1, n + 1):
                graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])
    ```
