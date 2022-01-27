---
title: Python - numpy issue 등록해보기
data: 2022-01-26 22:00:00 +09:00
categories: [Python]
tag: [Python]
---
numpy를 사용하던 중 이상한 부분을 확인하여 GitHub issue 등록을 하였습니다.


# 문제 발견

weights에 np.nan이 포함되어 있을 경우 mask된 위치라도 결과가 np.nan으로 나왔습니다.

```python
np.ma.average(
    np.ma.array([1., 2., 3., 4.], mask=[False, False, True, True]),
    weights=np.array([1, 1, 1, np.nan]),
    returned=True
)
->
(nan, nan)
```

- [docs](https://numpy.org/doc/stable/reference/generated/numpy.ma.average.html)에서 `avg`값을 `sum(weights)`로 나눠서 계산하는 부분 때문에 문제가 생길까 싶었지만, masked된 값은 계산에 사용되지 않는다고해서 정확한 문제가 무엇인지 알 수 없었습니다.

    weights 부분에서는 "The importance that each element has in the computation of the average."로 weights 부분은 계산에 사용된다는 설명이 있었습니다.

- 그래서 [sorucecode](https://github.com/numpy/numpy/blob/v1.21.0/numpy/ma/extras.py#L528-L631)를 찾아보았습니다.
    
    위의 problem 상황에서 sourcecode를 간단히하면 다음과 같이 계산됩니다.

    ```python
    np.ma.average(a, weights)
    ->
    wgt = np.asanyarray(weights)
    wgt = wgt*(~a.mask)

    scl = wgt.sum()
    avg = np.multiply(a, wgt).sum() / scl
    ```

    여기서 `wgt*(~a.mask)` 를 계산할 때 `wgt`에 있는 `np.nan`의 경우 사라지지 않고 그대로 유지되어 `scl`을 계산하게되고, 따라서 `scl`이 `np.nan`이 됩니다.


# Issue 등록

- 2021-11-15

    관련된 설명을 찾기가 힘들었고, 이 문제에 대해 GitHub에 Issue([#20375](https://github.com/numpy/numpy/issues/20375))를 등록해보았습니다.

- 2021-12-03

    아무도 답변을 달아주지 않아서 잘못올렸나 싶었지만, 몇 주 뒤에 한 분이 문제를 확인하고 PR([#20505](https://github.com/numpy/numpy/pull/20505))까지 해주셨습니다. PR 코드에 테스트 케이스도 추가되어있는 것을 보고 이렇게 하는 거구나라는 생각이 들었고, 저도 PR을 해보고 싶었습니다.

- 2022-12-09

    며칠 뒤에 몇 분이 해당 PR을 검토해주셨습니다. 그 과정에서 asarray와 asanyarray에 대해 말이 오갔었고 해당 내용을 개인적으로 찾아보았습니다. ([Python - numpy asarray 비교](https://junyeopp.github.io/posts/python_numpy_asarray_asanyarray/))

- 2022-12-24

    머지되었습니다!


실제 사용하는 numpy에도 반영될때까지 지켜볼 예정입니다. 작은 경험이었지만 오픈소스가 이렇게 발전되고 있구나라는 생각을 하게된 좋은 경험이었습니다. 다음에는 PR도 생성해보고 싶네요.
