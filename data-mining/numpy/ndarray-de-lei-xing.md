# ndarray的类型

依然是先初始化一个 ndarray

```python
data = np.array([1.1, 2.2, 3.3])
```

将其输出出来看看

```python
data

array([1.1, 2.2, 3.3])
```

用`dtype` api来查询类型

```python
data.dtype

dtype('float64')
```

当然也可以在创建数组时指定类型

```python
np.array([1.1, 2.2, 3.3], dtype="float32")

np.array([1.1, 2.2, 3.3], dtype=np.float32)
```

再调用查询类型的api，会发现两个都类型是相同的
