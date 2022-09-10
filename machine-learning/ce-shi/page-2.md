---
description: 数据分析工具——matplotlib
---

# matplotlib

#### 快速上手

{% code overflow="wrap" lineNumbers="true" %}
```python
import matplotlib.pyplot as plt

plt.figure()
plt.plot([1, 0, 9], [4, 5, 6])
plt.show()
```
{% endcode %}

![](https://picture.lanlance.cn/i/2022/09/10/631ca5c5a7465.png)

简单分析一下快速入门的四行代码。首先是一个包引入，然后使用`plt.figure()` 来创建一张画布，接下来用下面一行代码进行画图

```python
plt.plot([1, 0, 9], [4, 5, 6])
```

最后使用`plot.show()` 将图像展示出来
