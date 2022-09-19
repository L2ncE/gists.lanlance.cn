# 完整原始折线图

#### 需求：画出某城市11点到12点1小时内每分钟的温度变化折线图，温度范围在15度\~18度

很明显，若需要表示温度变化的话依然需要用到折线图，并且需要两个数据。为了模拟温度变化，我们导入了`random` 包来实现此效果

```python
import random

# 1、准备数据 x y
x = range(60)
y_shanghai = [random.uniform(15, 18) for i in x]
```

接下来就是和以前一样的创建画布和绘制图像，但我们想要图像的信息更加丰富，例如增加x、y的刻度说明以及网格显示、描述信息等信息，那我们就需要进一步的改动

```python
import random

# 1、准备数据 x y
x = range(60)
y_shanghai = [random.uniform(15, 18) for i in x]

# 中文显示问题
plt.rcParams['font.sans-serif'] = ['Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False  #用来正常显示负号

# 2、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 3、绘制图像
plt.plot(x, y_shanghai)

# 修改x、y刻度
# 准备x的刻度说明
x_label = ["11点{}分".format(i) for i in x]
plt.xticks(x[::5], x_label[::5])
plt.yticks(range(0, 40, 5))

# 添加网格显示
plt.grid(linestyle="--", alpha=0.5)

# 添加描述信息
plt.xlabel("时间变化")
plt.ylabel("温度变化")
plt.title("某城市11点到12点每分钟的温度变化状况")

# 4、显示图
plt.show()
```

值得一提的是，这里为了防止出现中文乱码的现象调用了两个api来防止这个问题发生



![](https://picture.lanlance.cn/i/2022/09/19/63287f1423c75.png)
