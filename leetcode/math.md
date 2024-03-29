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

### 400. 第 N 位数字

https://leetcode.cn/problems/nth-digit/

给你一个整数 n ，请你在无限的整数序列 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...] 中找出并返回第 n 位上的数字。

```cpp
class Solution {
public:
    int findNthDigit(int n) {
        int digit = 1;
        long base = 1;

        while(n > 9 * base * digit) {
            n -= 9 * base * digit;
            base *= 10;
            digit++;
        }

        long val = base + (n - 1) / digit;
        int index = (n - 1) % digit;
        string s = to_string(val);
        return s[index] - '0';
    }
};
```

找数学规律，一位数有几个？1~9 共 9 _ 1 = 9 个。共几位？共 1 _ 9 = 9 位。

二位数有几个？10~99 共 9 _ 10 = 90 个。共几位？共 2 _ 90 = 180 位。

三位数有几个？100~999 共 9 _ 100 = 900 个。共几位？共 3 _ 900 = 2700 位。

以此类推，我们可以通过这个规律推断第 n 位的数字到底是什么。所以这道题的难点在于如何把上述规律写成算法代码。

### 470. 用 Rand7() 实现 Rand10()

https://leetcode.cn/problems/implement-rand10-using-rand7/

给定方法  rand7  可生成 [1,7] 范围内的均匀随机整数，试写一个方法  rand10  生成 [1,10] 范围内的均匀随机整数。

你只能调用  rand7()  且不能调用其他方法。请不要使用系统的  Math.random()  方法。

每个测试用例将有一个内部参数 n，即你实现的函数 rand10() 在测试时将被调用的次数。请注意，这不是传递给 rand10() 的参数。

```cpp
// The rand7() API is already defined for you.
// int rand7();
// @return a random integer in the range 1 to 7

class Solution {
public:
    int rand10() {
        int t = (rand7() - 1) * 7 + rand7();
        if(t > 40) return rand10();
        return (t - 1) % 10 + 1;
    }
};
```

### 48. 旋转图像

https://leetcode.cn/problems/rotate-image/

给定一个 n × n 的二维矩阵  matrix 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        for (int i = 0, j = n - 1; i < j; i ++, j --) swap(matrix[i], matrix[j]);
        for(int i = 0; i < n; i++) for(int j = i; j < n; j++) swap(matrix[i][j], matrix[j][i]);
    }
};
```
