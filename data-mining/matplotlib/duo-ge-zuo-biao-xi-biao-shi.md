---
description: plt.subplots(面向对象的画图方法)
---

# 多个坐标系表示

#### 需求：再添加一个城市的温度变化

准备数据阶段不需要有改动，但是在创建画布api上会有改动

```python
# 2、创建画布
# plt.figure(figsize=(20, 8), dpi=80)
figure, axes = plt.subplots(nrows=1, ncols=2, figsize=(20, 8), dpi=80)
```

在接下来的修改刻度，添加网格显示已经添加描述信息阶段都需要去索引你选择的坐标系进行实现。最后调用`plt.show` 即可

```python
# 3、绘制图像
axes[0].plot(x, y_shanghai, color="r", linestyle="-.", label="上海")
axes[1].plot(x, y_beijing, color="b", label="北京")

# 显示图例
axes[0].legend()
axes[1].legend()

# 修改x、y刻度
# 准备x的刻度说明
x_label = ["11点{}分".format(i) for i in x]
axes[0].set_xticks(x[::5])
axes[0].set_xticklabels(x_label[::5])
axes[0].set_yticks(range(0, 40, 5))
axes[1].set_xticks(x[::5])
axes[1].set_xticklabels(x_label[::5])
axes[1].set_yticks(range(0, 40, 5))

# 添加网格显示
axes[0].grid(linestyle="--", alpha=0.5)
axes[1].grid(linestyle="--", alpha=0.5)

# 添加描述信息
axes[0].set_xlabel("时间变化")
axes[0].set_ylabel("温度变化")
axes[0].set_title("上海11点到12点每分钟的温度变化状况")
axes[1].set_xlabel("时间变化")
axes[1].set_ylabel("温度变化")
axes[1].set_title("北京11点到12点每分钟的温度变化状况")

# 4、显示图
plt.show()
```



![](https://picture.lanlance.cn/i/2022/09/20/6329c62a5e363.png)
