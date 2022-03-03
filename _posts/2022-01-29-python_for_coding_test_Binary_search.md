---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - 이진 탐색
data: 2022-01-29 01:00:00 +09:00
categories: [Coding Test]
tag: [Algorithm]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

## Binary Search
정렬된 배열에서 사용할 수 있습니다.
```python
def binary_search(array, target, start, end):
    while start <= end:
        mid = (start + end) // 2
        if array[mid] == target:
            return mid
        elif array[mid] > target:
            end = mid - 1
        else:
            start = mid + 1
    return
```
## Binary Search Tree
- 왼쪽 자식 노드 < 부모 노드 < 오른쪽 자식 노드


## [공유기 설치](https://www.acmicpc.net/problem/2110)
```python

```
