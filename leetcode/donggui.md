### 53. 最大子数组和

https://leetcode.cn/problems/maximum-subarray/

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。子数组是数组中的一个连续部分。

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

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));
        for(int i = 0; i < prices.size(); i++) {
            if(i == 0) {
                dp[i][0] = 0;
                dp[i][1] = -prices[i];
                continue;
            }
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], - prices[i]);
        }
        return dp[prices.size() - 1][0];
    }
};
```

用模板来写，更具普遍性。

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

### 509. 斐波那契数

https://leetcode.cn/problems/fibonacci-number/

斐波那契数 （通常用 F(n) 表示）形成的序列称为 斐波那契数列 。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和

```cpp
class Solution {
public:
    int fib(int n) {
        if(n == 0 || n == 1) return n;
        int dp_a = 0, dp_b = 1;
        for(int i = 2; i <= n; i++) {
            int dp_i = dp_a + dp_b;
            dp_a = dp_b;
            dp_b = dp_i;
        }
        return dp_b;
    }
};
```

此题用到了动态规划的思想，用到了状态转移方程，在最后使用滚动更新减少空间复杂度。

### 322. 零钱兑换

https://leetcode.cn/problems/coin-change/

给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。

计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回  -1 。

你可以认为每种硬币的数量是无限的。

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);
        dp[0] = 0;
        for(int i = 0; i < dp.size(); i++) {
            for(int coin : coins) {
                if(i - coin < 0) continue;
                dp[i] = min(dp[i], 1 + dp[i - coin]);
            }
        }
        return dp[amount] == amount + 1 ? -1 : dp[amount];
    }
};
```

目标金额作为变量。不过 dp 函数体现在函数参数，而 dp 数组体现在数组索引

dp 数组的定义：当目标金额为 i 时，至少需要 dp[i] 枚硬币凑出。

### 122. 买卖股票的最佳时机 II

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/

给你一个整数数组 prices ，其中  prices[i] 表示某支股票第 i 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候   最多   只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。

返回 你能获得的 最大 利润  。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));
        for(int i = 0; i < prices.size(); i++) {
            if(i == 0) {
                dp[i][0] = 0;
                dp[i][1] = -prices[i];
                continue;
            }
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[prices.size() - 1][0];
    }
};
```

套股票问题的框架即可。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int dp_i_0 = 0, dp_i_1 = INT_MIN;
        for(int i = 0; i < prices.size(); i++) {
            dp_i_0 = max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = max(dp_i_1, dp_i_0 - prices[i]);
        }
        return dp_i_0;
    }
};
```

空间复杂度优化版本。

### 309. 最佳买卖股票时机含冷冻期

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/

给定一个整数数组 prices，其中第   prices[i]  表示第  i  天的股票价格 。​

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));
        for(int i = 0; i < prices.size(); i++) {
            if(i == 0) {
                dp[i][0] = 0;
                dp[i][1] = -prices[i];
                continue;
            }
            if(i == 1) {
                dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
                dp[i][1] = max(dp[i - 1][1], -prices[i]);
                continue;
            }
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 2][0] -prices[i]);
        }
        return dp[prices.size() - 1][0];
    }
};
```

需要注意的是不能再第二天买入股票，因此在买入时应该吧 `i - 1` 改为 `i - 2`。

### 714. 买卖股票的最佳时机含手续费

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/

给定一个整数数组  prices，其中 prices[i]表示第  i  天的股票价格 ；整数  fee 代表了交易股票的手续费用。

你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。

注意：这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        vector<vector<int>> dp(prices.size(), vector<int>(2));
        for(int i = 0; i < prices.size(); i++) {
            if(i == 0) {
                dp[i][0] = 0;
                dp[i][1] = -prices[i] - fee;
                continue;
            }
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i] - fee);
        }
        return dp[prices.size() - 1][0];
    }
};
```

需要考虑每次手续费的问题，在购买时扣除最方便。

### 123. 买卖股票的最佳时机 III

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/

给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成   两笔   交易。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int max_k = 2;
        vector<vector<vector<int>>> dp(prices.size(), vector<vector<int>>(max_k + 1, vector<int>(2)));
        for(int i = 0; i < prices.size(); i++) {
            for(int k = max_k; k > 0; k--) {
                if(i == 0) {
                    dp[i][k][0] = 0;
                    dp[i][k][1] = -prices[i];
                    continue;
                }
                dp[i][k][0] = max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i]);
                dp[i][k][1] = max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i]);
            }
        }
        return dp[prices.size() - 1][max_k][0];
    }
};
```

开始考虑 k 的值，需要多一层。

### 188. 买卖股票的最佳时机 IV

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/

给定一个整数数组  prices ，它的第 i 个元素  prices[i] 是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

```cpp
class Solution {
public:
    int maxProfit(int max_k, vector<int>& prices) {
        vector<vector<vector<int>>> dp(prices.size(), vector<vector<int>>(max_k + 1, vector<int>(2)));
        if(max_k > prices.size() / 2) {
            vector<vector<int>> dp2(prices.size(), vector<int>(2));
            for(int i = 0; i < prices.size(); i++) {
                if(i == 0) {
                    dp2[i][0] = 0;
                    dp2[i][1] = -prices[i];
                    continue;
                }
                dp2[i][0] = max(dp2[i - 1][0], dp2[i - 1][1] + prices[i]);
                dp2[i][1] = max(dp2[i - 1][1], dp2[i - 1][0] - prices[i]);
            }
            return dp2[prices.size() - 1][0];
        }
        for(int i = 0; i < prices.size(); i++) {
            for(int k = max_k; k > 0; k--) {
                if(i == 0) {
                    dp[i][k][0] = 0;
                    dp[i][k][1] = -prices[i];
                    continue;
                }
                dp[i][k][0] = max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i]);
                dp[i][k][1] = max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i]);
            }
        }
        return dp[prices.size() - 1][max_k][0];
    }
};
```

最复杂的情况，看似和上一题差不多，只改了 `max_k` 的大小，实际上由于 `max_k` 过大会导致超内存的情况，所以当 `max_k > prices.size() / 2` 的情况，也就是可以判断为 k 无穷大的情况时，直接将代码优化为 k 为无穷大的情况即可。

### 198. 打家劫舍

https://leetcode.cn/problems/house-robber/

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        vector<int> dp(nums.size() + 2);
        for(int i = nums.size() - 1; i >= 0; i--) {
            dp[i] = max(dp[i + 1], dp[i + 2] + nums[i]);
        }
        return dp[0];
    }
};
```

经典问题，找到状态转移方程即可。

### 213. 打家劫舍 II

https://leetcode.cn/problems/house-robber-ii/

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。

给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if(nums.size() == 1) return nums[0];
        return max(dp(nums, 0, nums.size() - 2), dp(nums, 1, nums.size() - 1));
    }

    int dp(vector<int>& nums, int st, int ed) {
        vector<int> res(nums.size() + 2);
        for(int i = ed; i >= st; i--) {
            res[i] = max(res[i + 1], res[i + 2] + nums[i]);
        }
        return res[st];
    }
};
```

与上题类似，区别在于由于是环形数组，所以要不然选最前面要不然选最后面一个。

### 337. 打家劫舍 III

https://leetcode.cn/problems/house-robber-iii/

小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为  root 。

除了  root  之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 两个直接相连的房子在同一天晚上被打劫 ，房屋将自动报警。

给定二叉树的  root 。返回   在不触动警报的情况下  ，小偷能够盗取的最高金额  。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int rob(TreeNode* root) {
        vector<int> res = dp(root);
        return max(res[0], res[1]);
    }

    vector<int> dp(TreeNode* root) {
        if(!root) return {0, 0};
        vector<int> left = dp(root->left);
        vector<int> right = dp(root->right);
        int rob = root->val + left[1] + right[1];
        int notrob = max(left[0], left[1]) + max(right[0], right[1]);
        return {rob, notrob};
    }
};
```

返回一个大小为 2 的数组，res[0] 表示不抢 root 的话，得到的最大钱数 res[1] 表示抢 root 的话，得到的最大钱数。

### 343. 整数拆分

https://leetcode.cn/problems/integer-break/

给定一个正整数 n ，将其拆分为 k 个 正整数 的和（ k >= 2 ），并使这些整数的乘积最大化。

返回 你可以获得的最大乘积 。

```cpp
class Solution {
public:
    int cuttingRope(int n) {
        vector<int> dp(n + 1);
         dp[2] = 1;
         for(int i = 3; i <= n; i++) {
             for(int j = 1; j <= i / 2; j++) {
                 dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]));
             }
         }
         return dp[n];
    }
};
```

列出状态转移方程 `dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]))` 就非常简单了。

### 64. 最小路径和

https://leetcode.cn/problems/minimum-path-sum/

```cpp
class Solution {
public:
    vector<vector<int>>memo;
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        memo = vector<vector<int>>(m, vector<int>(n, -1));
        return dp(grid, m - 1, n - 1);
    }

    int dp(vector<vector<int>>& grid, int i, int j) {
        if(i == 0 && j == 0) return grid[0][0];
        if(i < 0 || j < 0) return INT_MAX;
        if(memo[i][j] != -1) return memo[i][j];
        memo[i][j] =  min(dp(grid, i - 1, j), dp(grid, i, j - 1)) + grid[i][j];
        return memo[i][j];
    }
};
```

从左上角位置 (0, 0) 走到位置 (i, j) 的最小路径和为 dp(grid, i, j)，还可以用备忘录优化一下执行效率。

### 91. 解码方法

https://leetcode.cn/problems/decode-ways/

一条包含字母  A-Z 的消息通过以下映射进行了 编码 ：

'A' -> "1"
'B' -> "2"
...
'Z' -> "26"
要 解码 已编码的消息，所有数字必须基于上述映射的方法，反向映射回字母（可能有多种方法）。例如，"11106" 可以映射为：

"AAJF" ，将消息分组为 (1 1 10 6)
"KJF" ，将消息分组为 (11 10 6)
注意，消息不能分组为   (1 11 06) ，因为 "06" 不能映射为 "F" ，这是由于 "6" 和 "06" 在映射中并不等价。

给你一个只含数字的 非空 字符串 s ，请计算并返回 解码 方法的 总数 。

题目数据保证答案肯定是一个 32 位 的整数。

```cpp
class Solution {
public:
    int numDecodings(string s) {
        int n = s.size();
        if(n < 1) return 0;
        vector<int>dp(n + 1);
        dp[0] = 1, dp[1] = s[0] == '0' ? 0 : 1;
        for(int i = 2; i <= n; i++) {
            char c = s[i - 1], d = s[i - 2];
            if('1' <= c && c <= '9') dp[i] += dp[i - 1];
            if(d == '1' || d == '2' && c <= '6') dp[i] += dp[i - 2];
        }
        return dp[n];
    }
};
```

### 300. 最长递增子序列

https://leetcode.cn/problems/longest-increasing-subsequence/

给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

子序列   是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(), 1);
        for(int i = 0; i < nums.size(); i++) {
            for(int j = 0; j < i; j++) {
                if(nums[i] > nums[j]) dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        int res = 0;
        for(int i = 0; i < dp.size(); i++) res = max(res, dp[i]);
        return res;
    }
};
```

### 72. 编辑距离

https://leetcode.cn/problems/edit-distance/

给你两个单词  word1 和  word2， 请返回将  word1  转换成  word2 所使用的最少操作数  。

你可以对一个单词进行如下三种操作：

插入一个字符
删除一个字符
替换一个字符

```cpp
class Solution {
public:
    vector<vector<int>> memo;
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        memo = vector<vector<int>>(n, vector<int>(m, -1));
        return dp(word1, n - 1, word2, m - 1);
    }

    int dp(string s1, int i, string s2, int j) {
        if(i == -1) return j + 1;
        if(j == -1) return i + 1;
        if(memo[i][j] != -1) return memo[i][j];
        if(s1[i] == s2[j]) memo[i][j] = dp(s1, i - 1, s2, j - 1);
        else memo[i][j] = minFunc(
            dp(s1, i - 1, s2, j - 1) + 1,
            dp(s1, i, s2, j - 1) + 1,
            dp(s1, i - 1, s2, j) + 1
        );
        return memo[i][j];
    }

    int minFunc(int a, int b, int c) {
        return min(a, min(b, c));
    }
};
```

### 1143. 最长公共子序列

https://leetcode.cn/problems/longest-common-subsequence/

给定两个字符串  text1 和  text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。

一个字符串的   子序列   是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

```cpp
class Solution {
public:
    int longestCommonSubsequence(string s1, string s2) {
        int m = s1.size(), n = s2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        for(int i = 1; i <= m; i++) {
            for(int j = 1; j <= n; j++) {
                if(s1[i - 1] == s2[j - 1]) dp[i][j] = 1 + dp[i - 1][j - 1];
                else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[m][n];
    }
};
```

### 221. 最大正方形

https://leetcode.cn/problems/maximal-square/

在一个由 '0' 和 '1' 组成的二维矩阵内，找到只包含 '1' 的最大正方形，并返回其面积。

```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        if(!m || !n) return 0;
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        int res = 0;
        for(int i = 1; i <= m; i++) {
            for(int j = 1; j <= n; j++) {
                if(matrix[i - 1][j - 1] == '1') {
                    dp[i][j] = min(dp[i - 1][j - 1], min(dp[i - 1][j], dp[i][j - 1])) + 1;
                    res = max(res, dp[i][j]);
                }
            }
        }
        return res * res;
    }
};
```

### 62. 不同路径

https://leetcode.cn/problems/unique-paths/

一个机器人位于一个 m x n  网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

```cpp
class Solution {
public:
    vector<vector<int>> memo;
    int uniquePaths(int m, int n) {
        memo = vector<vector<int>>(m, vector<int>(n, -1));
        return dp(m - 1, n - 1);
    }

    int dp(int x, int y) {
        if(x == 0 && y == 0) return 1;
        if(x < 0 || y < 0) return 0;
        if(memo[x][y] != -1) return memo[x][y];
        memo[x][y] = dp(x - 1, y) + dp(x, y - 1);
        return memo[x][y];
    }
};
```

### 152. 乘积最大子数组

https://leetcode.cn/problems/maximum-product-subarray/

给你一个整数数组 nums ，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个  32-位 整数。

子数组 是数组的连续子序列。

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp1(n);
        vector<int> dp2(n);
        dp1[0] = nums[0], dp2[0] = nums[0];
        int res = dp2[0];
        for(int i = 1; i < n; i++) {
            dp1[i] = min(dp1[i - 1] * nums[i], min(dp2[i - 1] * nums[i], nums[i]));
            dp2[i] = max(dp1[i - 1] * nums[i], max(dp2[i - 1] * nums[i], nums[i]));
            res = max(res, dp2[i]);
        }
        return res;
    }
};
```
