---
title: pd.Series의 True 개수 구하기 속도 비교
data: 2021-08-04 19:00:00 +09:00
categories: [Python]
tag: [Python]
---
pandas Series에서 True인 값의 개수를 구해야하는 상황에서 아래와 같은 4가지 방법들의 시간차이가 얼마나나는지 궁금해서 간단하게 테스트해보았습니다.

테스트 결과, pd.Series는 numpy로 계산하는 것이 빨랐습니다.

## 조건

1. sum: sum(ser)

    ```python
    %%timeit
    sum(ser)
    ```

2. len: len(ser[ser == True])

    ```python
    %%timeit
    len(ser[ser == True])
    ```

3. .sum: ser.sum()
    
    ```python
    %%timeit
    ser.sum()
    ```

4. count: ser.value_counts().loc[True]

    ```python
    %%timeit
    ser.value_counts().loc[True]
    ```


## 테스트

1. `ser = pd.Series([True, False] * 10)` , `len(ser) == 20`
    1. sum(): `2.02 µs ± 93 ns per loop`
    2. len(): `107 µs ± 820 ns per loop`
    3. .sum(): `26 µs ± 775 ns per loop`
    4. .value_counts(): `252 µs ± 5.4 µs per loop`

2. `ser = pd.Series([*([True, False] * 100000)])`, `len(ser) == 200000`
    1. sum(): `10.8 ms ± 259 µs per loop`
    2. len(): `1.61 ms ± 30.2 µs per loop`
    3. .sum(): `180 µs ± 6.38 µs per loop`
    4. .value_counts(): `837 µs ± 11.8 µs per loop`

3. `ser = pd.Series([*([False] * 199999), True])`, `len(ser) == 200000`
    1. sum(): `9.1 ms ± 224 µs per loop`
    2. len(): `324 µs ± 2.41 µs per loop`
    3. .sum(): `179 µs ± 2.72 µs per loop`
    4. .value_counts(): `851 µs ± 18.5 µs per loop`

4. `ser = pd.Series([*([True] * 199999), False])`, `len(ser) == 200000`
    1. sum(): `10.5 ms ± 346 µs per loop`
    2. len(): `566 µs ± 13.1 µs per loop`
    3. .sum(): `174 µs ± 1.32 µs per loop`
    4. .value_counts(): `852 µs ± 11.2 µs per loop`

5. `ser = pd.Series([*([True] * 100000), *([False] * 100000)])`, `len(ser) == 200000`
    1. sum(): `11 ms ± 188 µs per loop`
    2. len(): `441 µs ± 5.33 µs per loop`
    3. .sum(): `174 µs ± 2.11 µs per loop`
    4. .value_counts(): `852 µs ± 14.7 µs per loop`


## 결론

- Series의 길이가 20000일 때, .sum()을 사용하는 것이 더 빠르게 측정되었습니다. 예상대로 numpy로 계산하는 것이 빨랐습니다.
- len의 경우 True의 위치에 따라 다르게 측정되었습니다.
- 계산하고자하는 타겟 series의 길이가 최대 100000 정도이므로 .sum()을 이용하였습니다.