# 统计运算

使用之前使用的 temp

```python
temp.max(axis=0)
```

```python
array([1.1, 1.1, 1.1, 1.1])
```

也可以这样

```python
np.max(temp, axis=-1)
```

```python
array([1.1, 1.1, 1.1, 1.1])
```

还可以这样

```python
np.argmax(temp, axis=-1)
```

```python
array([0, 2, 1, 3])
```
