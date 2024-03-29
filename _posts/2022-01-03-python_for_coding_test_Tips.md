---
title: 이것이 취업을 위한 코딩 테스트다 with 파이썬 - Tips
data: 2022-01-03 01:00:00 +09:00
categories: [Coding Test]
tag: [Tips]
---

- [이것이 취업을 위한 코딩 테스트다 with 파이썬](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=247882118)을 읽고 필요한 부분을 요약 정리하였습니다.

# 코딩 테스트 환경
Jupyter나 VSCode를 이용해서 코테 문제를 풀었었는데, 이 책에서 온라인 IDE로 [리플릿](https://replit.com/~)을 소개해주어서 사용해해보려고 합니다.

# 복잡도
## 1. 시간 복잡도
보통 코딩 테스트 문제의 시간 제한은 1 ~ 5초 정도로 연산 횟수가 10억을 넘어가지 않도록 하는 것이 필요합니다. 1초에 1~2000만 번의 연산을 수행한다고 가정하면 안정적입니다. 문제를 풀 때 책에서 소개한 다음의 예시 몇 가지를 생각하는 것이 필요합니다.
- N=500: 시간 복잡도가 O(N^3)인 알고리즘을 설계
- N=2000: 시간 복잡도가 O(N^2)인 알고리즘을 설계
- N=100,000: 시간 복잡도가 O(N log N)인 알고리즘을 설계
- N=10,000,000: 시간 복잡도가 O(N)인 알고리즘을 설계
## 2. 공간 복잡도
보통 코딩 테스트 문제의 메모리 제한은 128MB ~ 512MB 정도로 데이터 개수가 1000만개 이상 넘어가지 않도록 하는 것이 필요합니다.

# 컴퓨터공학 지식
- [Technical Interview Guidelines for Beginners](https://github.com/JaeYeopHan/Interview_Question_for_Beginner)

# Python tips

- 소수점 값을 비교하는 작업이 필요하면 ```round()``` 함수를 이용하는게 좋습니다. (컴퓨터가 실수를 정확하게 표현하지 못하기 때문에)
- 리스트 관련 매서드의 시간복잡도
    - append(): O(1)
    - sort(): O(N log N)
    - reverse(): O(N)
    - insert(): O(N)
    - count(): O(N)
    - remove(): O(N)
- 집합 관련 매서드의 시간복잡도
    - add(): O(1)
    - remove(): O(1)
