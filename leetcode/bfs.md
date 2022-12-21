### 116. 填充每个节点的下一个右侧节点指针

https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/

给定一个 完美二叉树 ，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

struct Node {
int val;
Node *left;
Node *right;
Node \*next;
}
填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。

初始状态下，所有 next 指针都被设置为 NULL。

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* left;
    Node* right;
    Node* next;

    Node() : val(0), left(NULL), right(NULL), next(NULL) {}

    Node(int _val) : val(_val), left(NULL), right(NULL), next(NULL) {}

    Node(int _val, Node* _left, Node* _right, Node* _next)
        : val(_val), left(_left), right(_right), next(_next) {}
};
*/

class Solution {
public:
    Node* connect(Node* root) {
        if(!root) return root;
        auto src = root;
        while(root->left){
            for(auto p = root; p; p = p->next){
                p->left->next = p->right;
                if(p->next) p->right->next = p->next->left;
                else p->right->next = 0;
            }
            root = root->left;
        }
        return src;
    }
};
```

从根节点开始宽度优先遍历，每次遍历一层，遍历时按从左到右的顺序，对于每个节点，先让左儿子指向右儿子，然后让右儿子指向下一个节点的左儿子。最后让这一层最右侧的节点指向 NULL。遍历到叶节点所在的层为止。

### 542. 01 矩阵

https://leetcode.cn/problems/01-matrix/

给定一个由 0 和 1 组成的矩阵 mat ，请输出一个大小相同的矩阵，其中每一个格子是 mat 中对应位置元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        if(matrix.empty() || matrix[0].empty()) return matrix;

        int n = matrix.size(), m = matrix[0].size();
        vector<vector<int>> res(n, vector<int>(m, -1));

        typedef pair<int, int> PII;
        queue<PII> q;

        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                if(matrix[i][j] == 0){
                    q.push({i, j});
                    res[i][j] = 0;
                }
            }
        }

        int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
        while(q.size()){
            auto t = q.front();
            q.pop();

            for(int i = 0; i < 4; i++){
                int a = t.first + dx[i], b = t.second + dy[i];
                if(a >= 0 && b >= 0 && a < n && b < m && res[a][b] == -1){
                    res[a][b] = res[t.first][t.second] + 1;
                    q.push({a, b});
                }
            }
        }
        return res;
    }
};
```

首先判空返回，然后开一个存结果的数组，全部置为-1。接着开一个队列用来跑每个格子，用两个 for 循环把为 0 的格子放到队列中，并且把结果数组的值置为 0，因为此时自己就是 0，距离自然也为 0。接下来是 BFS 宽搜，用到经典 dx、dy 。遍历上下左右四个格子，为 -1 的就是还没有进行检查的，检查完后放进队列，最后返回结果数组。

### 994. 腐烂的橘子

https://leetcode.cn/problems/rotting-oranges/

在给定的 m x n 网格 grid 中，每个单元格可以有以下三个值之一：

值 0 代表空单元格；
值 1 代表新鲜橘子；
值 2 代表腐烂的橘子。
每分钟，腐烂的橘子 周围 4 个方向上相邻 的新鲜橘子都会腐烂。

返回 直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1 。

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& g) {
        int n = g.size(), m = g[0].size();
        queue<pair<int, int>> q;
        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                if(g[i][j] == 2) q.push({i, j});
            }
        }

        int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
        int res = 0;
        if(q.size()) res--;
        while(q.size()){
            res++;
            int cnt = q.size();
            while(cnt--){
                auto t = q.front();
                q.pop();
                for(int i = 0; i < 4; i++){
                    int a = t.first + dx[i], b = t.second + dy[i];
                    if(a >= 0 && b >= 0 && a < n && b < m && g[a][b] == 1){
                        g[a][b] = 2;
                        q.push({a, b});
                    }
                }
            }
        }

        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                if(g[i][j] == 1) return -1;
            }
        }
        return res;
    }
};
```

和上面的 01 矩阵基本类似。
