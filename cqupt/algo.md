## 算法设计与分析

### 选择排序

每一轮选择最小的那个 一个简单的两重循环。

![](https://picture.lanlance.cn/i/2023/06/13/64886a3de647f.png)

### 冒泡排序

也是一个两重循环，但是每次只要剩余数中的最小值。

时间复杂度为 O(n^2)

![](https://picture.lanlance.cn/i/2023/06/13/64886ac34a24f.png)

### 顺序查找

用一个 `target` 来做一个限定。

![](https://picture.lanlance.cn/i/2023/06/13/64886b282ef0f.png)

### 蛮力字符串匹配

无脑循环，如果模式串中和匹配串中的不匹配直接往前走。

![](https://picture.lanlance.cn/i/2023/06/13/64886ba8c5b3a.png)

### 最近对问题

一个简单的初中数学问题，通过计算两数距离即可，甚至可以不用开平方根简化计算。

时间复杂度为 O(n^2)

![](https://picture.lanlance.cn/i/2023/06/13/64886bee8cade.png)

### 凸包问题

> 问题：对于平面上 n 个点，找包围它们的最小凸多边形。

蛮力算法：对于每对点 p1 和 p2，判断是否所有其他点都在连接 p1 和 p2 的直线的同一侧;

算出直线 y = ax + b 即可，将所有点带到这个直线中判断符号是否相同。

时间复杂度为 O(n^3)

### 旅行商问题

值得一提的一个点是可以看到有三对不同的路线，每对路线之间仅有方向不同，因此可以把顶点排列的数量减半。

![](https://picture.lanlance.cn/i/2023/06/13/6488707dc8785.png)

### 背包问题

教材上的图就是一个简单的图表，这边比较重要的点是用穷举法解决背包问题需要 O(2^n) 的时间复杂度。他和上面的旅行商问题都是典型的 NP 困难问题。

### 深度优先查找

![](https://picture.lanlance.cn/i/2023/06/13/6488745edf4c7.png)

V 和 E 分别为图的顶点和边的数量。数据结构为栈，顶点顺序有两种。

### 广度优先查找

![](https://picture.lanlance.cn/i/2023/06/13/648874d23c558.png)

V 和 E 分别为图的顶点和边的数量。数据结构为队列，顶点顺序只有一种。

### 插入排序

每一步将一个待排序的数据插入到前面已经排好序的有序序列中，直到插完所有元素为止。

最坏时间复杂度与平均时间复杂度均为 O(n^2)，最好时间复杂度为 O(n)。

![](https://picture.lanlance.cn/i/2023/06/13/648875af59463.png)

### 拓扑排序

拓扑排序这里提供了两种解法，分别是 DFS 遍历栈和基于减治技术的算法。

![](https://picture.lanlance.cn/i/2023/06/13/648879f4e26b2.png)

减治技术会在每次迭代时，删除没有输入边的节点。

![](https://picture.lanlance.cn/i/2023/06/13/64887a013a372.png)

### 生成排列

满足最小变化要求。

![](https://picture.lanlance.cn/i/2023/06/13/64887b42eec76.png)

时间复杂度为 O(n!)，生成的排序不为字典序。

![](https://picture.lanlance.cn/i/2023/06/13/64887b8c70174.png)

### 生成子集

能够生成 2^n 个子集，也就是幂集。后一组的每一个元素都可以通过把 a(n) 添加到 {a(1) ... a(n - 1)} 中子集来获得。

![](https://picture.lanlance.cn/i/2023/06/13/64887cbb76b5b.png)

### 折半查找

经典二分问题

![](https://picture.lanlance.cn/i/2023/06/14/6489b4032fbcd.png)

### 俄式乘法

仅有折半、加倍以及相加三个操作，硬件实现非常快速。

![](https://picture.lanlance.cn/i/2023/06/14/6489b667406d6.png)

### 选择问题

![](https://picture.lanlance.cn/i/2023/06/14/6489bb45054f9.png)

![](https://picture.lanlance.cn/i/2023/06/14/6489bdbac1e05.png)

![](https://picture.lanlance.cn/i/2023/06/14/6489bdc637859.png)

### 插值查找

插值查找计算要查找的值在数组中的大致位置，然后将数组分为两部分，其中一部分必定不可能包含要查找的值，另一部分可能包含要查找的值，然后在可能包含要查找的值的那一部分中继续递归查找，直到找到要查找的值或者确定要查找的值不存在于数组中。

在数组分布不均匀的情况下，它的效率可能会比二分查找还低。

![](https://picture.lanlance.cn/i/2023/06/14/6489bfcb65885.png)

### 分治法适用条件

1. 原始可分解，且分解出来的子问题和原始问题就有相同的类型。
2. 分解出来的子问题到很小时可以很容易（在很短的时间和空间内能求解）求解。
3. 子问题的解能合并。

### 归并排序

归并排序的平均时间复杂度也是 O(nlogn)。最好和最坏情况下的时间复杂度都是 O(nlogn)。

![](https://picture.lanlance.cn/i/2023/06/14/6489c09ada546.png)

### 快速排序

平均时间复杂度也是 O(nlogn)。最好情况下，即每次都能平分数组，时间复杂度为 O(nlogn)；最坏情况下，即每次划分只能减少一个元素，时间复杂度为 O(n^2)。但是最坏情况发生的概率非常小，所以快速排序通常被认为是一种非常高效的排序算法。

![](https://picture.lanlance.cn/i/2023/06/14/6489c0e24b724.png)

![](https://picture.lanlance.cn/i/2023/06/14/6489c0fec542b.png)

### 大数乘法

![](https://picture.lanlance.cn/i/2023/06/14/6489c39f335cb.png)

### 预排序

- 检查数组中元素的唯一性

先排序再比对是否唯一，时间复杂度基于排序 O(nlogn)。

### 霍纳法则

![](https://picture.lanlance.cn/i/2023/06/14/6489c7880e1c0.png)

### 动态规划解决背包问题

![](https://picture.lanlance.cn/i/2023/06/14/6489cb378fc26.png)

### 最优二叉查找树

![](https://picture.lanlance.cn/i/2023/06/14/6489cdefeb4bd.png)

### Warshall

![](https://picture.lanlance.cn/i/2023/06/15/648ae1634c75c.png)

`R[i][j][k] == R[i][j][k - 1] || R[i][k][k - 1] && R[k][j][k - 1]`

### Floyd

![](https://picture.lanlance.cn/i/2023/06/15/648ae1cf3aa51.png)

`D[i][j] == min(D[i][j], D[i][k] + D[k][j])`

### 哈夫曼编码

二叉树左分支为 0，右分支为 1。

### P 类

![](https://picture.lanlance.cn/i/2023/06/15/648b0916539be.png)

### NP 类

![](https://picture.lanlance.cn/i/2023/06/15/648b0931d91a8.png)

![](https://picture.lanlance.cn/i/2023/06/15/648b09d91deb8.png)

![](https://picture.lanlance.cn/i/2023/06/15/648b09e83eb9d.png)
