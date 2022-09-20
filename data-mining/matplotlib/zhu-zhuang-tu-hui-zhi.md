# 柱状图绘制

需求1：对比不同语言的年薪高低

需要切换绘制api为柱状体api`bar` ，并可以自定义颜色

```python
# 1、准备数据
movie_names = ['Golang', 'Ruby', 'Python', 'Java', 'JavaScript', 'Android', 'iOS', 'Rust',
               'C', 'PHP', 'SQL']
tickets = [120577, 119550, 114904, 112013, 111922, 109377, 108783, 108744, 101734, 92867, 85845]

# 2、创建画布
plt.figure(figsize=(23, 10), dpi=100)

# 3、绘制柱状图
x_ticks = range(len(movie_names))
plt.bar(x_ticks, tickets, color=['b', 'r', 'g', 'y', 'c', 'm', 'y', 'k', 'c', 'g', 'y'])

# 修改x刻度
plt.xticks(x_ticks, movie_names)

# 添加网格显示
plt.grid(linestyle="--", alpha=0.5)

# 4、显示图像
plt.show()
```



![](https://picture.lanlance.cn/i/2022/09/20/6329c99a9a718.png)

#### 需求2-如何对比电影票房收入才更能加有说服力？

这里我们准备了三部电影的数据，包含他们的首日票房以及首周票房。我们绘制了两个柱状图分别进行表示。在修改刻度阶段会有讲究，只有修改在中间才会让电影名字出现在两个柱状图的中间

```python
# 1、准备数据
movie_name = ['雷神3：诸神黄昏', '正义联盟', '寻梦环游记']

first_day = [10587.6, 10062.5, 1275.7]
first_weekend = [36224.9, 34479.6, 11830]

# 2、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 3、绘制柱状图
plt.bar(range(3), first_day, width=0.2, label="首日票房")
plt.bar([0.2, 1.2, 2.2], first_weekend, width=0.2, label="首周票房")

# 显示图例
plt.legend()

# 修改刻度
plt.xticks([0.1, 1.1, 2.1], movie_name)

# 4、显示图像
plt.show()
```



![](https://picture.lanlance.cn/i/2022/09/20/6329ca549863f.png)
