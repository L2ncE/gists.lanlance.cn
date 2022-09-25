# ndarray的形状

首先我们初始化三个 ndarray

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
b = np.array([1, 2, 3, 4])
c = np.array([[[1, 2, 3], [4, 5, 6]], [[1, 2, 3], [4, 5, 6]]])
```

现在我们依次将每一个 ndarray 输出出来

```python
a  # (2, 3)
array([[1, 2, 3],
       [4, 5, 6]])
       
b  # (4,)
array([1, 2, 3, 4])

c  # (2, 2, 3)
array([[[1, 2, 3],
        [4, 5, 6]],

       [[1, 2, 3],
        [4, 5, 6]]])
```

再调用`shape` api，将形状打印出来

```python
a.shape
(2, 3)

b.shape
(4,)

c.shape
(2, 2, 3)
```
