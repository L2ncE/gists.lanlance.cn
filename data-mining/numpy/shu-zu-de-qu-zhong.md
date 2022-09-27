# 数组的去重

首先初始化一个数组

```python
temp = np.array([[1, 2, 3, 4], [3, 4, 5, 6]])
```

再使用去重的api

```python
np.unique(temp)
```

输出出来

```python
array([1, 2, 3, 4, 5, 6])
```



```python
set(temp.flatten())  # 先变成一维的才能用set
```

输出出来

```python
{1, 2, 3, 4, 5, 6}
```
