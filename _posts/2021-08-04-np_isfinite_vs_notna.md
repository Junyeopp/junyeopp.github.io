---
title: np.isfinite(pd.Series) vs (pd.Series).notna() 비교
data: 2021-08-04 19:00:00 +09:00
categories: [Python]
tag: [Python]
---
numpy.isfinite()와 pandas.Series.notna()가 어떤 값들을 확인해주는지 궁금해서 비교해보았습니다.

1. [numpy.isfinite()](https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html)

    : infinity가 아니고 NaN이 아닌 값에 대해서 True를 반환합니다.

    ```python
    np.isfinite(pd.Series([np.nan, np.inf, None, 0, 1, 100]))

    ->

    0    False
    1    False
    2    False
    3     True
    4     True
    5     True
    dtype: bool
    ```

2. [pandas.Series.notna()](https://pandas.pydata.org/docs/reference/api/pandas.Series.notna.html)

    : not NA인 값에 대해 True를 반환합니다.

    ```python
    pd.Series([np.nan, np.inf, None, 0, 1, 100]).notna()

    ->

    0    False
    1     True
    2    False
    3     True
    4     True
    5     True
    dtype: bool
    ```

- [pandas.Series.notna](https://pandas.pydata.org/docs/reference/api/pandas.Series.notna.html)의 설명에 따르면 empty string이나 np.inf는 NA값으로 고려되지 않습니다.

    > Characters such as empty strings “” or numpy.inf are not considered NA values (unless you set pandas.options.mode.use_inf_as_na = True). NA values, such as None or numpy.NaN, get mapped to False values.
    >
