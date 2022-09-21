# ndarray与Python原生list运算效率对比

我们都知道 numpy 的 ndarray 好用，但是我们也需要有证据。接下来我们用 ndarray 与 Python 原生list运算效率进行对比

因为我们需要生成一个大数组，所以这里需要引入`random` 包

```python
import random
import time
```

接下来进行数组的生成

```python
# 生成一个大数组
python_list = []

for i in range(100000000):
    python_list.append(random.random())
    
ndarray_list = np.array(python_list)
```

然后我们使用`len(ndarray_list)` 来获得长度为100000000，与我们生成的原生list无异

在此之后我们就要进行效率的对比了，我这里的方案是将其分别进行求和并获取完成任务所消耗的时间，这样就很直观了

```python
# 原生pythonlist求和
t1 = time.time()
a = sum(python_list)
t2 = time.time()
d1 = t2 - t1

# ndarray求和
t3 = time.time()
b = np.sum(ndarray_list)
t4 = time.time()
d2 = t4 - t3
```

分别获取`d1` 与`d2` 的值

![](https://picture.lanlance.cn/i/2022/09/21/632b239052c2c.png)
