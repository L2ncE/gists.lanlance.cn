# np.where（三元运算符）

使用上一节用到的 stock\_change。现在想要判断前四个股票前四天的涨跌幅 大于0的置为1，否则为0

```python
temp = stock_change[:4, :4]

np.where(temp > 0, 1, 0)
```

输出一下

```python
array([[1, 1, 0, 1],
       [0, 1, 1, 1],
       [0, 1, 1, 1],
       [0, 1, 0, 1]])
```

还可以这样

```python
np.where([[True, False, True, True],
          [True, True, True, True],
          [False, False, True, False],
          [True, False, True, False]], 1, 0)
```

输出一下

```python
array([[1, 0, 1, 1],
       [1, 1, 1, 1],
       [0, 0, 1, 0],
       [1, 0, 1, 0]])
```



```python
# 判断前四个股票前四天的涨跌幅 大于0.5并且小于1.5的，换为1，否则为0
# 判断前四个股票前四天的涨跌幅 大于0.5或者小于-0.5的，换为1，否则为0
# (temp > 0.5) and (temp < 1)
np.logical_and(temp > 0.5, temp < 1.5)
```

输出结果

```python
array([[ True,  True, False,  True],
       [False, False,  True,  True],
       [False,  True,  True,  True],
       [False, False, False,  True]])
```

使用三元运算符实现

```python
np.where(np.logical_and(temp > 0.5, temp < 1.5), 1, 0)
```

```python
array([[1, 1, 0, 1],
       [0, 0, 1, 1],
       [0, 1, 1, 1],
       [0, 0, 0, 1]])
```

不一定只能1或者0

```python
np.where(np.logical_or(temp > 0.5, temp < -0.5), 11, 3)
```

```python
array([[11, 11,  3, 11],
       [11,  3, 11, 11],
       [ 3, 11, 11, 11],
       [11,  3,  3, 11]])
```
