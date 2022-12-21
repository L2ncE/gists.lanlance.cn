### 53. 最大子数组和

https://leetcode.cn/problems/maximum-subarray/

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。子数组\*\* 是数组中的一个连续部分。

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int res = INT_MIN;
        for(int i = 0, last = 0; i < nums.size(); i++){
            last = nums[i] + max(0, last);
            res = max(res, last);
        }
        return res;
    }
};
```

我们自己想象一个 f[i]，f[i] 为所有以下标 i 结尾的数组之和。在 f[i] 中又可以分成有一个数的和有两个数的，有一个数的则就是它本身。我们再看有两个或更多数的，这些数组都含有 i ，所以这些数组可以转化成 `f[i - 1] + nums[i]`。我们将一个数和多个数的进行比较即得 `max = （f[i - 1] + nums[i], nums[i])`，将 nums[i] 提出来即得 `res = max(f[i - 1], 0) + nums[i]`。

### 121. 买卖股票的最佳时机

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int fit = 0;
        int minpre = INT_MAX;
        for(int i = 0; i < prices.size(); i++){
            fit = max(fit, prices[i] - minpre);
            minpre = min(minpre, prices[i]);
        }
        return fit;
    }
};
```

我们从第一天开始遍历，然后通过 minpre 来记录到达第 i 天时，之前的股票最低价，然后每次都用当天的价格减去最低价查看是否是最大的利润即可。

### 118. 杨辉三角

https://leetcode.cn/problems/pascals-triangle/

给定一个非负整数 *`numRows`，*生成「杨辉三角」的前 _`numRows`_ 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

```cpp
class Solution {
public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int>> res;
        for(int i = 0; i < numRows; i++){
            vector<int> l(i + 1);
            l[0] = l[i] = 1;
            for(int j = 1; j < i; j++){
                l[j] = res[i - 1][j - 1] + res[i - 1][j];
            }
            res.push_back(l);
        }
        return res;
    }
};
```

模拟就完事了。

### 70. 爬楼梯

https://leetcode.cn/problems/climbing-stairs/

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

```cpp
class Solution {
public:
    int climbStairs(int n) {
        int a = 1, b = 1;
        while(-- n){
            int c = a + b;
            a = b, b = c;
        }
        return b;
    }
};
```

小学奥数题，找规律就完事了。

### 198. 打家劫舍

https://leetcode.cn/problems/house-robber/

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<int> f(n + 1), g(n + 1);
        // 房屋下标从 1 开始
        for(int i = 1; i <= n; i++){
            f[i] = g[i - 1] + nums[i - 1];
            g[i] = max(f[i - 1], g[i - 1]);
        }
        return max(f[n], g[n]);
    }
};
```

先分析状态表示，`f[i]` 为必选 i 的情况，`g[i]` 是必不选 i 的情况。通过状态分析可得，`f[i] = g[i - 1] + w[j]` ，`g[i] = max(f[i - 1], g[i - 1])` ，取最大即可。

### 120. 三角形最小路径和

给定一个三角形 triangle ，找出自顶向下的最小路径和。

每一步只能移动到下一行中相邻的结点上。相邻的结点 在这里指的是 下标 与 上一层结点下标 相同或者等于 上一层结点下标 + 1 的两个结点。也就是说，如果正位于当前行的下标 i ，那么下一步可以移动到下一行的下标 i 或 i + 1 。

```cpp
class Solution {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
        for(int i = triangle.size() - 2; i >= 0; i--){
            for(int j = 0; j <= i; j++){
                triangle[i][j] += min(triangle[i + 1][j], triangle[i + 1][j + 1]);
            }
        }
        return triangle[0][0];
    }
};
```

需要注意的是最后一行不要算，所以 `i = triangle.size()` 。
