# 生成数组的方法

### 生成0和1的数组

```python
np.zeros(shape=(3, 4), dtype="float32")
```

输出以下结果

```python
array([[0., 0., 0., 0.],
       [0., 0., 0., 0.],
       [0., 0., 0., 0.]], dtype=float32)
```



```python
np.ones(shape=[2, 3], dtype=np.int32)
```

输出以下结果

```python
array([[1, 1, 1],
       [1, 1, 1]], dtype=int32)
```



### 从现有数组生成

拿到之前的 score 数组

```python
array([[80, 89, 86, 67, 79],
       [78, 97, 89, 67, 81],
       [90, 94, 78, 67, 74],
       [91, 91, 90, 67, 69],
       [76, 87, 75, 67, 86],
       [70, 79, 84, 67, 84],
       [94, 92, 93, 67, 64],
       [86, 85, 83, 67, 80]])
```

使用`array()` api

```python
# np.array()
data1 = np.array(score)
```

将data1输出

```python
array([[80, 89, 86, 67, 79],
       [78, 97, 89, 67, 81],
       [90, 94, 78, 67, 74],
       [91, 91, 90, 67, 69],
       [76, 87, 75, 67, 86],
       [70, 79, 84, 67, 84],
       [94, 92, 93, 67, 64],
       [86, 85, 83, 67, 80]])
```

还可以用`asarray` api

```python
# np.asarray()
data2 = np.asarray(score)
```

将data2输出

```python
array([[80, 89, 86, 67, 79],
       [78, 97, 89, 67, 81],
       [90, 94, 78, 67, 74],
       [91, 91, 90, 67, 69],
       [76, 87, 75, 67, 86],
       [70, 79, 84, 67, 84],
       [94, 92, 93, 67, 64],
       [86, 85, 83, 67, 80]])
```

同样还可以使用`copy()` api

```python
# np.copy()
data3 = np.copy(score)
```

将data3输出

```python
array([[80, 89, 86, 67, 79],
       [78, 97, 89, 67, 81],
       [90, 94, 78, 67, 74],
       [91, 91, 90, 67, 69],
       [76, 87, 75, 67, 86],
       [70, 79, 84, 67, 84],
       [94, 92, 93, 67, 64],
       [86, 85, 83, 67, 80]])
```

除此之外，我们还可以给数组中特定的位置重新赋值

```python
score[3, 1] = 10000
```

在此之后将score重新输出

```python
array([[   80,    89,    86,    67,    79],
       [   78,    97,    89,    67,    81],
       [   90,    94,    78,    67,    74],
       [   91, 10000,    90,    67,    69],
       [   76,    87,    75,    67,    86],
       [   70,    79,    84,    67,    84],
       [   94,    92,    93,    67,    64],
       [   86,    85,    83,    67,    80]])
```

有意思的地方来了，这个时候我们分别输出data1 data2 data3，会发现只有data2的值发生了变化，这是Python中深浅拷贝不同所带来的不同



### 生成固定范围的数组

调用`linspace` api

```python
np.linspace(0, 10, 5)

array([ 0. ,  2.5,  5. ,  7.5, 10. ])
```

调用`arange` api

```python
np.arange(0, 21, 5)

array([ 0,  5, 10, 15, 20])
```



### 生成随机数组

使用`random.uniform` api，并设置参数。生成1000000个-1到1的数的随机数组

```python
data1 = np.random.uniform(low=-1, high=1, size=1000000)
```

把 data1 输出出来看看

```python
array([-0.71158366, -0.74950877,  0.15876375, ...,  0.76520229,
        0.64071715,  0.59968039])
```

用 matplotlib 把这个随机数组用直方图输出出来

```python
import matplotlib.pyplot as plt

# 1、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 2、绘制直方图
plt.hist(data1, 1000)

# 3、显示图像
plt.show()
```

![](https://picture.lanlance.cn/i/2022/09/26/63319c0e4a6e0.png)

同样可以用`random.normal` 来生成正态分布是随机数组

```python
# 正态分布
data2 = np.random.normal(loc=1.75, scale=0.1, size=1000000)
```

再使用 matplotlib 输出图像

```python
# 1、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 2、绘制直方图
plt.hist(data2, 1000)

# 3、显示图像
plt.show()
```

![](https://picture.lanlance.cn/i/2022/09/26/63319d0061f5d.png)
