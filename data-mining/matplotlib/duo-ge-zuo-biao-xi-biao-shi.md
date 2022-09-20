---
description: plt.subplots(面向对象的画图方法)
---

# 多个坐标系表示

#### 需求：再添加一个城市的温度变化

准备数据阶段不需要有改动，但是在创建画布api上会有改动。

```python
# 2、创建画布
# plt.figure(figsize=(20, 8), dpi=80)
figure, axes = plt.subplots(nrows=1, ncols=2, figsize=(20, 8), dpi=80)
```

