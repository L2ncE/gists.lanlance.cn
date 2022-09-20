# 饼图绘制

如果我们想要在圆饼中直观获取到各事物所占比例，就可以调用饼图api进行绘制，参数中不仅可以设置颜色还可以进行格式化数据显示

```python
# 1、准备
movie_name = ['雷神3：诸神黄昏', '正义联盟', '东方快车谋杀案', '寻梦环游记', '全球风暴', '降魔传', '追捕', '七十七天',
              '密战', '狂兽', '其它']

place_count = [60605, 54546, 45819, 28243, 13270, 9945, 7679, 6799, 6101, 4621, 20105]

# 2、创建画布
plt.figure(figsize=(20, 8), dpi=80)

# 3、绘制饼图
plt.pie(place_count, labels=movie_name, colors=['b', 'r', 'g', 'y', 'c', 'm', 'y', 'k', 'c', 'g', 'y'],
        autopct="%1.2f%%")

# 显示图例
plt.legend()

plt.axis('equal')

# 4、显示图像
plt.show()
```



![](https://picture.lanlance.cn/i/2022/09/20/6329cd5b17a48.png)
