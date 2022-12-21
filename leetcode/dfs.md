### 733. 图像渲染

https://leetcode.cn/problems/flood-fill/

有一幅以 m x n 的二维整数数组表示的图画 image ，其中 image[i] [j] 表示该图画的像素值大小。

你也被给予三个整数 sr , sc 和 newColor 。你应该从像素 image[sr] [sc] 开始对图像进行 上色填充 。

为了完成 上色工作 ，从初始像素开始，记录初始坐标的 上下左右四个方向上 像素值与初始坐标相同的相连像素点，接着再记录这四个方向上符合条件的像素点与他们对应 四个方向上 像素值与初始坐标相同的相连像素点，……，重复该过程。将所有有记录的像素点的颜色值改为 newColor 。

最后返回 经过上色渲染后的图像 。

```cpp
class Solution {
public:
    vector<vector<int>> g;
    int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
    void dfs(int x, int y, int color, int newColor){
        g[x][y] = newColor;
        for(int i = 0; i < 4; i++){
            int a = x + dx[i];
            int b = y + dy[i];
            if(a >= 0 && b >= 0 && a < g.size() && b < g[0].size() && g[a][b] == color){
                dfs(a, b, color, newColor);
            }
        }
    }
    vector<vector<int>> floodFill(vector<vector<int>>& image, int x, int y, int newColor) {
        g = image;
        if(g[x][y] == newColor) return g;
        dfs(x, y, g[x][y], newColor);
        return g;
    }
};
```

难点在于模拟上下左右四个方向，其他的不算困难，板子题。

### 695. 岛屿的最大面积

https://leetcode.cn/problems/max-area-of-island/submissions/

给你一个大小为 m x n 的二进制矩阵 grid 。

岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在 水平或者竖直的四个方向上 相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。

岛屿的面积是岛上值为 1 的单元格的数目。

计算并返回 grid 中最大的岛屿面积。如果没有岛屿，则返回面积为 0 。

```cpp
class Solution {
public:
    vector<vector<int>> g;
    int n, m;
    int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};

    int dfs(int x, int y){
        int res = 1;
        g[x][y] = 0;
        for(int i = 0; i < 4; i++){
            int a = x + dx[i], b = y + dy[i];
            if(a >= 0 && b >= 0 && a < n && b < m && g[a][b]){
                res += dfs(a, b);
            }
        }
        return res;
    }

    int maxAreaOfIsland(vector<vector<int>>& grid) {
        g = grid;
        n = g.size(), m = g[0].size();
        int res = 0;
        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                if(g[i][j]){
                    res = max(res, dfs(i, j));
                }
            }
        }
        return res;
    }
};
```

注意上下左右的偏移量操作。

### 617. 合并二叉树

https://leetcode.cn/problems/merge-two-binary-trees/

给你两棵二叉树： root1 和 root2 。

想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，不为 null 的节点将直接作为新二叉树的节点。

返回合并后的二叉树。

注意: 合并过程必须从两个树的根节点开始。

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
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if(root2) swap(root1, root2);
        if(!root1) return NULL;
        if(root2) root1->val += root2->val;
        root1->left = mergeTrees(root1->left, root2 ? root2->left : NULL);
        root1->right = mergeTrees(root1->right, root2 ? root2->right : NULL);
        return root1;
    }
};
```

先确保 root1 肯定不为空，要不然直接返回 NULL，接着直接递归深搜左右两边即可。

### 77. 组合

https://leetcode.cn/problems/combinations/

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combine(int n, int k) {
        dfs(n, k, 1);
        return ans;
    }
    void dfs(int n, int k, int start){
        if(!k){
            ans.push_back(path);
            return;
        }
        for(int i = start; i <= n; i++){
            path.push_back(i);
            dfs(n, k - 1, i + 1);
            path.pop_back();
        }
    }
};
```

递归一下即可。

### 46. 全排列

https://leetcode.cn/problems/permutations

给定一个不含重复数字的数组 `nums` ，返回其 _所有可能的全排列_ 。你可以 **按任意顺序** 返回答案。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<bool> st;
    vector<int> path;
    vector<vector<int>> permute(vector<int>& nums) {
        path = vector<int>(nums.size());
        st = vector<bool>(nums.size());
        dfs(nums, 0);
        return ans;
    }
    void dfs(vector<int>& nums, int u){
        if(u == nums.size()){
            ans.push_back(path);
            return;
        }
        for(int i = 0; i < nums.size(); i++){
            if(st[i] == false){
                st[i] = true;
                path[u] = nums[i];
                dfs(nums, u + 1);
                st[i] = false;
            }
        }
    }
};
```

直接爆搜，注意存放状态的问题。

### 784. 字母大小写全排列

https://leetcode.cn/problems/letter-case-permutation/

给定一个字符串 `s` ，通过将字符串 `s` 中的每个字母转变大小写，我们可以获得一个新的字符串。

返回 _所有可能得到的字符串集合_ 。以 **任意顺序** 返回输出。

```cpp
class Solution {
public:
    vector<string> ans;
    vector<string> letterCasePermutation(string s) {
        dfs(s, 0);
        return ans;
    }
    void dfs(string &s, int u){
        if(u == s.length()){
            ans.push_back(s);
            return;
        }
        dfs(s, u + 1);
        if (s[u] >= 'A')
        {
            s[u] ^= 32;
            dfs(s, u + 1);
            s[u] ^= 32;
        }
    }
};
```

异或运算直接用 `^= 32` 来切换大小写，爆搜即可。
