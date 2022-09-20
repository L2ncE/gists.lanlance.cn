# 绘制数学函数图像

在这里会需要用到 numpy 准备x、y的数据，而后的创建画布、绘制图像、显示图像都无任何区别

```python
import numpy as np

# 1、准备x，y数据
x = np.linspace(-1, 1, 1000)
y = 2 * x * x

# 2、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 3、绘制图像
plt.plot(x, y)

# 添加网格显示
plt.grid(linestyle="--", alpha=0.5)

# 4、显示图像
plt.show()
```



![](https://picture.lanlance.cn/i/2022/09/20/6329c7814f63c.png)
