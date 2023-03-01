### 566. 重塑矩阵

https://leetcode.cn/problems/reshape-the-matrix/

在 MATLAB 中，有一个非常有用的函数 reshape ，它可以将一个 m x n 矩阵重塑为另一个大小不同（r x c）的新矩阵，但保留其原始数据。

给你一个由二维数组 mat 表示的 m x n 矩阵，以及两个正整数 r 和 c ，分别表示想要的重构的矩阵的行数和列数。

重构后的矩阵需要将原始矩阵的所有元素以相同的 行遍历顺序 填充。

如果具有给定参数的 reshape 操作是可行且合理的，则输出新的重塑矩阵；否则，输出原始矩阵。

```cpp
class Solution {
public:
    vector<vector<int>> matrixReshape(vector<vector<int>>& nums, int r, int c) {
        int n = nums.size(), m = nums[0].size();
        if (n * m != r * c) return nums;
        vector<vector<int>> res(r, vector<int>(c));
        for (int i = 0; i < n * m; i ++ )
            res[i / c][i % c] = nums[i / m][i % m];
        return res;
    }
};
```

### 263. 丑数

https://leetcode.cn/problems/ugly-number/

丑数 就是只包含质因数 2、3 和 5 的正整数。

给你一个整数 n ，请你判断 n 是否为 丑数 。如果是，返回 true ；否则，返回 false 。

```cpp
class Solution {
public:
    bool isUgly(int n) {
        if(n < 1) return false;
        while(n % 2 == 0) n /= 2;
        while(n % 3 == 0) n /= 3;
        while(n % 5 == 0) n /= 5;
        if(n == 1) return true;
        return false;
    }
};
```

### 264. 丑数 II

https://leetcode.cn/problems/ugly-number-ii/submissions/

```cpp
class Solution {
public:
    int nthUglyNumber(int n) {
        int p1 = 1, p2 = 1, p3 = 1;
        int pd1 = 1, pd2 = 1, pd3 = 1;
        int p = 1;
        vector<int> ugly(n + 1);
        while(p <= n) {
            int minn = min(min(pd1, pd2), pd3);
            ugly[p++] = minn;
            if(pd1 == minn) pd1 = 2 * ugly[p1++];
            if(pd2 == minn) pd2 = 3 * ugly[p2++];
            if(pd3 == minn) pd3 = 5 * ugly[p3++];
        }
        return ugly[n];
    }
};
```

我们用 p1, p2, p3 分别代表三条丑数链表上的指针，用 pd1, pd2, pd3 代表丑数链表上节点的值，用 ugly 数组记录有序链表合并之后的结果。
