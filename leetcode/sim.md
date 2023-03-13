### 59. 螺旋矩阵 II

https://leetcode.cn/problems/spiral-matrix-ii/

给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

**模拟**

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n, vector<int>(n, 0));
        int sx = 0, sy = 0, offset = 1;
        int loop = n / 2, count = 1;
        while(loop --){
            int i = sx, j = sy;

            for(; j < n - offset; j++)
                res[i][j] = count++;
            for(; i < n - offset; i++)
                res[i][j] = count++;
            for(; j > sx; j--)
                res[i][j] = count++;
            for(; i > sy; i--)
                res[i][j] = count++;

            sx++, sy++, offset++;
        }
        if(n % 2 != 0) res[n / 2][n / 2] = count;
        return res;
    }
};
```

模拟即可，注意每次要更新 x, y 的起点与边界值 offset 。

**偏移量**

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n, vector<int>(n, 0));
        int dx[4] = {0, 1, 0, -1}, dy[4] = {1, 0, -1, 0};
        vector<vector<bool>> st(n, vector<bool>(n, false));
        for(int i = 0, x = 0, y = 0, d = 0, count = 1; i < n * n; i++){
            res[x][y] = count++, st[x][y] = true;
            int a = x + dx[d], b = y + dy[d];
            if(a < 0 || a >= n || b < 0 || b >= n || st[a][b]){
                d = (d + 1) % 4;
                a = x + dx[d], b = y + dy[d];
            }
            x = a, y = b;
        }
        return res;
    }
};
```

模拟狗都不写。

### 54. 螺旋矩阵

https://leetcode.cn/problems/spiral-matrix/

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> res;
        int n = matrix.size(), m = matrix[0].size();
        if(!n) return res;
        vector<vector<bool>> st(n, vector<bool>(m, false));
        int dx[4] = {0, 1, 0, -1}, dy[4] = {1, 0, -1, 0};
        for(int i = 0, x = 0, y = 0, d = 0; i < n * m; i++){
            res.push_back(matrix[x][y]);
            st[x][y] = true;
            int a = x + dx[d], b = y + dy[d];
            if(a < 0 || b < 0 || a >= n || b >= m || st[a][b]){
                d = (d + 1) % 4;
                a = x + dx[d], b = y + dy[d];
            }
            x = a, y = b;
        }
        return res;
    }
};
```

使用偏移量来做，模拟？狗都不写！

### 415. 字符串相加

https://leetcode.cn/problems/add-strings/

给定两个字符串形式的非负整数  num1 和 num2 ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 BigInteger），  也不能直接将输入的字符串转换为整数形式。

```cpp
class Solution {
public:
    vector<int> add(vector<int>& A, vector<int>& B) {
        vector<int> C;
        for(int i = 0, t = 0; i < A.size() || i < B.size() || t; i++) {
            if(i < A.size()) t += A[i];
            if(i < B.size()) t += B[i];
            C.push_back(t % 10);
            t /= 10;
        }
        return C;
    }

    string addStrings(string num1, string num2) {
        vector<int> A, B;
        for(int i = num1.size() - 1; i >= 0; i--) A.push_back(num1[i] - '0');
        for(int i = num2.size() - 1; i >= 0; i--) B.push_back(num2[i] - '0');
        auto C = add(A, B);
        string c;
        for(int i = C.size() - 1; i >= 0; i--) c += to_string(C[i]);
        return c;
    }
};
```
