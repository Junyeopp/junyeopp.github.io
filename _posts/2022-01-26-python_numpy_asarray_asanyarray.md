---
title: Python - numpy asarray 비교
data: 2022-01-26 23:00:00 +09:00
categories: [Python]
tag: [Python]
---
# Array creation routines

- 이 [질문](https://github.com/numpy/numpy/pull/20505#discussion_r767460395)을 이해하기 위해서 [np.ma.asarray](https://numpy.org/doc/stable/reference/generated/numpy.ma.asarray.html#numpy-ma-asarray)와 [np.asanyarray](https://numpy.org/doc/stable/reference/generated/numpy.asanyarray.html#numpy.asanyarray)의 차이를 찾아보았습니다
- [MaskedArray](https://numpy.org/doc/stable/reference/maskedarray.baseclass.html#the-maskedarray-class) Class는 missing data를 다루기 위한 ndarray의 subclass입니다.
    
    MaskedArray는 일반적인 np.ndarray로 된 data 부분과 해당 data의 boolean mask부분으로 되어있습니다.
    
1. `np.asarray`
    
    : Convert the input to an array.
    
    : No copy is performed if the input is already an ndarray with matching dtype and order.
    
    : If *a* is a subclass of ndarray, a base class ndarray is returned.
    
2. `np.asanyarray`
    
    : Convert the input to an ndarray, but pass ndarray subclasses through.
    
    : If *a* is an ndarray or a subclass of ndarray, it is returned as-is and no copy is performed.
    
3. `np.ma.asarray`
    
    : Convert the input to a masked array of the given data-type.
    
    : No copy is performed if the input is already an **`ndarray`**. If *a* is a subclass of **`MaskedArray`**, a base class **`MaskedArray`** is returned.
    
4. `np.ma.asanyarray`
    
    : Convert the input to a masked array, conserving subclasses.
    
    : If `a` is a subclass of `MaskedArray`, its class is conserved. No copy is performed if the input is already an `ndarray`.
    
    → [code](https://github.com/numpy/numpy/blob/v1.21.0/numpy/ma/core.py#L7956-L8002)를 보면 `subok=True` condition에서 asarry와 차이가 있는 것을 확인할 수 있습니다.

- Example

![example]({{ site.baseurl }}/assets/images/2022-01-26-python_numpy_asarray_asanyarray_example.png)

MaskedArray를 input으로 주었을 때,
- np.asarray는 ndarray를 return하고,
- np.asanyarray, np.ma.asarry, np.ma.asanyarray는 MaskedArray를 return하는 것을 확인할 수 있습니다.

asanyarray의 경우 input이 MaskedArray인 경우 아래와 같이 input을 그대로 반환하기에 is 비교시 True값을 얻을 수 있었습니다.

```python
def asanyarray(a, dtype=None):
	if isinstance(a, MaskedArray) and (dtype is None or dtype == a.dtype):
		return a
	return masked_array(a, dtype=dtype, copy=False, keep_mask=True, subok=True)
```

- 참고자료

    [https://numpy.org/doc/stable/reference/routines.array-creation.html#routines-array-creation](https://numpy.org/doc/stable/reference/routines.array-creation.html#routines-array-creation)