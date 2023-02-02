### 剑指 Offer 05. 替换空格

https://leetcode.cn/problems/ti-huan-kong-ge-lcof/

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

```cpp
class Solution {
public:
    string replaceSpace(string s) {
        int cnt = 0, l = s.length();
        for(int i = 0; i < l; i++){
            if(s[i] == ' ') cnt++;
        }
        s.resize(s.length() + 2 * cnt);
        for(int i = l - 1, j = s.length() - 1; i >= 0; i--){
            if(s[i] != ' '){
                s[j--] = s[i];
            }else{
                s[j--] = '0';
                s[j--] = '2';
                s[j--] = '%';
            }
        }
        return s;
    }
};
```

先把字符串开大，然后反着遍历字符串，将空格替换即可。

### 剑指 Offer 58 - II. 左旋转字符串

https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字 2，该函数将返回左旋转两位得到的结果"cdefgab"。

```cpp
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        reverse(s.begin(), s.end());
        reverse(s.begin(), s.end() - n);
        reverse(s.end() - n, s.end());
        return s;
    }
};
```

三个 `reverse` 就能解决战斗。

### 剑指 Offer 22. 链表中倒数第 k 个节点

https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/

输入一个链表，输出该链表中倒数第 k 个节点。为了符合大多数人的习惯，本题从 1 开始计数，即链表的尾节点是倒数第 1 个节点。

例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solutioxn {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        auto p1 = head, p2 = head;
        for(int i = 0; i < k; i++){
            p1 = p1->next;
        }
        while(p1){
            p1 = p1->next;
            p2 = p2->next;
        }
        return p2;
    }
};
```

首先，我们先让一个指针 p1 指向链表的头节点 head，然后走 k 步，现在的 p1，只要再走 n - k 步，就能走到链表末尾的空指针了，趁这个时候，再用一个指针 p2 指向链表头节点 head，
接下来就很显然了，让 p1 和 p2 同时向前走，p1 走到链表末尾的空指针时前进了 n - k 步，p2 也从 head 开始前进了 n - k 步，停留在第 n - k + 1 个节点上，即恰好停链表的倒数第 k 个节点上。

### 剑指 Offer 25. 合并两个排序的链表

https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(), p = dummy;
        while(l1 && l2){
            if(l1->val < l2->val){
                p = p->next = l1;
                l1 = l1->next;
            }else{
                p = p->next = l2;
                l2 = l2->next;
            }
        }
        if(l1) p->next = l1;
        if(l2) p->next = l2;
        return dummy->next;
    }
};
```

和主站 21 题一样，主要在于需要一个虚拟头结点和一个尾节点，清楚这两点这道题还是非常简单的。

### 剑指 Offer 52. 两个链表的第一个公共节点

https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/

输入两个链表，找出它们的第一个公共节点。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto a = headA, b = headB;
        while(a != b){
            if(a) a = a->next;
            else a = headB;
            if(b) b = b->next;
            else b = headA;
        }
        return a;
    }
};
```

遍历一边到头后去另一边，最后到入口处两边走的距离肯定相同。

### 剑指 Offer II 021. 删除链表的倒数第 n 个结点

https://leetcode.cn/problems/SLwz0R/

给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        auto dummy = new ListNode();
        dummy->next = head;
        auto p = dummy, q = dummy;
        for(int i = 0; i < n + 1; i++) q = q->next;
        while(q){
            p = p->next;
            q = q->next;
        }
        p->next = p->next->next;
        return dummy->next;
    }
};
```

我们先走到链表的倒数 n + 1 这个点，再删除他的下一个结点即可。注意使用虚拟头结点。

### 剑指 Offer II 022. 链表中环的入口节点

https://leetcode.cn/problems/c32eOV/

给定一个链表，返回链表开始入环的第一个节点。 从链表的头节点开始沿着 next 指针进入环的第一个节点为环的入口节点。如果链表无环，则返回  null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        auto p = head, q = head;
        while(q && q->next){
            p = p->next;
            q = q->next->next;
            if(p == q) break;
        }
        if(!q || !q->next) return NULL;
        p = head;
        while(q != p){
            p = p->next;
            q = q->next;
        }
        return p;
    }
};
```

经典题，先用快慢指针直到两个点相遇，如果快指针指到空那么说明没有环。有环的话则让慢指针回到头结点，再以相同的速度一起往前走，相遇的位置则是入口节点。

### 剑指 Offer II 078. 合并排序链表

https://leetcode.cn/problems/vvXgSW/

给定一个链表数组，每个链表都已经按升序排列。

请将所有链表合并到一个升序链表中，返回合并后的链表。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    struct Cmp {
        bool operator()(ListNode *a, ListNode *b){
            return a->val > b->val;
        }
    };

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto dummy = new ListNode(), p = dummy;
        priority_queue<ListNode*, vector<ListNode*>, Cmp> heap;
        for(auto h : lists) if(h) heap.push(h);

        while(heap.size()){
            auto t = heap.top();
            heap.pop();
            p = p->next = t;
            if(t->next) heap.push(t->next);
        }
        return dummy->next;
    }
};
```

注意构造一个 Cmp 的结构体传入优先级队列。

### 剑指 Offer 57. 和为 s 的两个数字

https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/

输入一个递增排序的数组和一个数字 s，在数组中查找两个数，使得它们的和正好是 s。如果有多对数字的和等于 s，则输出任意一对即可。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int i = 0, j = nums.size() - 1;
        while(i < j){
            if(nums[i] + nums[j] < target) i++;
            else if(nums[i] + nums[j] > target) j--;
            else return {nums[i], nums[j]};
        }
        return {};
    }
};
```

这道题用哈希表做也是一样的，不过用快慢指针感觉更有普遍意义。

### 剑指 Offer II 006. 排序数组中两个数字之和

https://leetcode.cn/problems/kLl5u1/

给定一个已按照 升序排列   的整数数组  numbers ，请你从数组中找出两个数满足相加之和等于目标数  target 。

函数应该以长度为 2 的整数数组的形式返回这两个数的下标值。numbers 的下标 从 0  开始计数 ，所以答案数组应当满足 0 <= answer[0] < answer[1] < numbers.length 。

假设数组中存在且只存在一对符合条件的数字，同时一个数字不能使用两次。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int i = 0, j = numbers.size() - 1;
        while(i < j){
            if(numbers[i] + numbers[j] > target) j--;
            else if(numbers[i] + numbers[j] < target) i++;
            else return {i, j};
        }
        return {};
    }
};
```

和上一题是同样的思路，使用快慢指针即可。

### 剑指 Offer 55 - I. 二叉树的深度

https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(!root) return 0;
        int l = maxDepth(root->left);
        int r = maxDepth(root->right);
        return max(l, r) + 1;
    }
};
```

分解成找左右子树的深度即可，比传统的 dfs 更简单。

### 剑指 Offer II 103. 最少的硬币数目

https://leetcode.cn/problems/gaM7Ch/

给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回  -1。

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);
        dp[0] = 0;
        for(int i = 0; i < dp.size(); i++){
            for(int coin : coins) {
                if(i - coin < 0) continue;
                dp[i] = min(dp[i], dp[i - coin] + 1);
            }
        }
        return dp[amount] == amount + 1 ? -1 : dp[amount];
    }
};
```

动态规划题，这里是自底向上的迭代方法来完成。

### 剑指 Offer II 083. 没有重复元素集合的全排列

https://leetcode.cn/problems/VvJkup/

给定一个不含重复数字的整数数组 nums ，返回其 所有可能的全排列 。可以 按任意顺序 返回答案。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<bool> st;
    vector<vector<int>> permute(vector<int>& nums) {
        st = vector<bool>(nums.size());
        dfs(nums, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int u) {
        if(u == nums.size()) {
            ans.push_back(path);
            return;
        }

        for(int i = 0; i < nums.size(); i++) {
            if(!st[i]) {
                st[i] = true;
                path.push_back(nums[i]);
                dfs(nums, u + 1);
                st[i] = false;
                path.pop_back();
            }
        }
    }
};
```

经典回溯问题，用一个存答案，一个存状态，其他的就是经典模版。

### 剑指 Offer 53 - I. 在排序数组中查找数字 I

https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/

统计一个数字在排序数组中出现的次数。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        if(!nums.size()) return 0;
        int i, j;
        int l = 0, r = nums.size() - 1;
        while(l < r) {
            int mid = l + r >> 1;
            if(nums[mid] >= target) r = mid;
            else l = mid + 1;
        }
        if(nums[r] != target) return 0;
        i = r;
        l = 0, r = nums.size() - 1;
        while(l < r) {
            int mid = l + r + 1>> 1;
            if(nums[mid] <= target) l = mid;
            else r = mid - 1;
        }
        if(nums[l] != target) return 0;
        j = l;
        return j - i + 1;
    }
};
```

用二分分别算左右两边数字的下标，最后算差值即可。

### 剑指 Offer II 079. 所有子集

https://leetcode.cn/problems/TVdhkn/

给定一个整数数组 nums ，数组中的元素 互不相同 。返回该数组所有可能的子集（幂集）。

解集 不能 包含重复的子集。你可以按 任意顺序 返回解集。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> subsets(vector<int>& nums) {
        dfs(nums, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int u) {
        if(u == nums.size()) {
            ans.push_back(path);
            return;
        } else if(u < nums.size()) ans.push_back(path);
        else return;

        for(int i = u; i < nums.size(); i++) {
            path.push_back(nums[i]);
            dfs(nums, i + 1);
            path.pop_back();
        }
    }
};
```

注意退出的条件判断。

### 剑指 Offer II 080. 含有 k 个元素的组合

https://leetcode.cn/problems/uUsW3B/

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combine(int n, int k) {
        dfs(n, k, 1);
        return ans;
    }

    void dfs(int n, int k, int u) {
        if(path.size() == k) {
            ans.push_back(path);
            return;
        }

        for(int i = u; i <= n; i++) {
            path.push_back(i);
            dfs(n, k, i + 1);
            path.pop_back();
        }
    }
};
```

和上一题类似，不过只需要找一层的即可。

### 剑指 Offer II 081. 允许重复选择元素的组合

https://leetcode.cn/problems/Ygoe9J/

给定一个无重复元素的正整数数组  candidates  和一个正整数  target ，找出  candidates  中所有可以使数字和为目标数  target  的唯一组合。

candidates  中的数字可以无限制重复被选取。如果至少一个所选数字数量不同，则两种组合是不同的。

对于给定的输入，保证和为  target 的唯一组合数少于 150 个。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, target, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int target, int u) {
        if(!target) {
            ans.push_back(path);
            return;
        } else if(target < 0) return;

        for(int i = u; i < nums.size(); i++) {
            path.push_back(nums[i]);
            target -= nums[i];
            dfs(nums, target, i);
            path.pop_back();
            target += nums[i];
        }
    }
};
```

因为数字可以重复用，所以以往的 i + 1 在这里变成 i 就行了。

### 剑指 Offer II 082. 含有重复元素集合的组合

https://leetcode.cn/problems/4sjJUc/

给定一个可能有重复数字的整数数组  candidates  和一个目标数  target ，找出  candidates  中所有可以使数字和为  target  的组合。

candidates  中的每个数字在每个组合中只能使用一次，解集不能包含重复的组合。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        dfs(candidates, target, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int target, int u) {
        if(!target) {
            ans.push_back(path);
            return;
        } else if(target < 0) return;

        for(int i = u; i < nums.size(); i++) {
            if(i > u && nums[i] == nums[i - 1]) continue;
            path.push_back(nums[i]);
            target -= nums[i];
            dfs(nums, target, i + 1);
            path.pop_back();
            target += nums[i];
        }
    }
};
```

和上一题不一样的点在于这里需要剪枝优化，把一样数字的给剪掉。

### 剑指 Offer II 084. 含有重复元素集合的全排列

https://leetcode.cn/problems/7p8L0Z/

给定一个可包含重复数字的整数集合 nums ，按任意顺序 返回它所有不重复的全排列。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<bool> st;
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        st = vector<bool>(nums.size());
        dfs(nums, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int u) {
        if(u == nums.size()) {
            ans.push_back(path);
            return;
        }

        unordered_set<int> S;

        for(int i = 0; i < nums.size(); i++) {
            if(!st[i]) {
                if(S.count(nums[i])) continue;
                S.insert(nums[i]);
                path.push_back(nums[i]);
                st[i] = true;
                dfs(nums, u + 1);
                path.pop_back();
                st[i] = false;
            }
        }
    }
};
```

用了一个哈希集合来存是否有重复使用来进行剪枝。

### 剑指 Offer II 014. 字符串中的变位词

https://leetcode.cn/problems/MPnaiL/

给定两个字符串 s1 和 s2，写一个函数来判断 s2 是否包含 s1 的某个变位词。

换句话说，第一个字符串的排列之一是第二个字符串的 子串 。

```cpp
class Solution {
public:
    bool checkInclusion(string s1, string s2) {
        unordered_map<char, int> need, win;
        for(auto c : s1) need[c]++;
        int right = 0, left = 0, vl = 0;
        while(right < s2.size()) {
            char c1 = s2[right++];
            if(need.count(c1)) {
                win[c1]++;
                if(win[c1] == need[c1]) vl++;
            }
            while(vl == need.size()) {
                if(right - left == s1.size()) return true;
                char c2 = s2[left++];
                if(need.count(c2)) {
                    if(win[c2] == need[c2]) vl--;
                    win[c2]--;
                }
            }
        }
        return false;
    }
};
```

滑动窗口问题，当有效位与需要的字符相同并且长度和 s1 相同是返回 true。

### 剑指 Offer II 015. 字符串中的所有变位词

https://leetcode.cn/problems/VabMRr/

给定两个字符串  s  和  p，找到  s  中所有 p 的   变位词   的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

变位词 指字母相同，但排列不同的字符串。

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> res;
        unordered_map<char, int> need, win;
        for(auto c : p) need[c]++;
        int right = 0, left = 0, vl = 0;
        while(right < s.size()) {
            char c1 = s[right++];
            if(need.count(c1)) {
                win[c1]++;
                if(win[c1] == need[c1]) vl++;
            }
            while(right - left >= p.size()) {
                if(vl == need.size()) res.push_back(left);
                char c2 = s[left++];
                if(need.count(c2)) {
                    if(win[c2] == need[c2]) vl--;
                    win[c2]--;
                }
            }
        }
        return res;
    }
};
```

滑动窗口题，与上一题不同的是这里需要每次把左边的下标保存到数组当中。

### 剑指 Offer II 016. 不含重复字符的最长子字符串

https://leetcode.cn/problems/wtcaE1/

给定一个字符串 s ，请你找出其中不含有重复字符的 最长连续子字符串 的长度。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char, int> win;
        int left = 0, right = 0, res = 0;
        while(right < s.size()) {
            char c1 = s[right++];
            win[c1]++;
            while(win[c1] > 1) {
                char c2 = s[left++];
                win[c2]--;
            }
            res = max(res, right - left);
        }
        return res;
    }
};
```

用哈希表或者滑动窗口来做其实都可以，用滑动窗口需要改一点框架，总体来说还更简单。

### 剑指 Offer II 017. 含有所有字符的最短字符串

https://leetcode.cn/problems/M1oyTv/

给定两个字符串 s 和  t 。返回 s 中包含  t  的所有字符的最短子字符串。如果 s 中不存在符合条件的子字符串，则返回空字符串 "" 。

如果 s 中存在多个符合条件的子字符串，返回任意一个。

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char, int> need, win;
        for(auto c : t) need[c]++;
        int right = 0, left = 0, vl = 0, st = 0, len = INT_MAX;
        while(right < s.size()) {
            char c1 = s[right++];
            if(need.count(c1)) {
                win[c1]++;
                if(win[c1] == need[c1]) vl++;
            }
            while(vl == need.size()) {
                if(right - left < len) {
                    len = right - left;
                    st = left;
                }
                char c2 = s[left++];
                if(need.count(c2)) {
                    if(win[c2] == need[c2]) vl--;
                    win[c2]--;
                }
            }
        }
        return len == INT_MAX ? "" : s.substr(st, len);
    }
};
```

最经典的滑动窗口问题，事实上和其他的没有什么大区别，理解逻辑就懂了。

### 剑指 Offer 63. 股票的最大利润

https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(!prices.size()) return 0;
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

股票题中最简单的一种，直接带公式即可。

### 剑指 Offer II 089. 房屋偷盗

https://leetcode.cn/problems/Gu0c2T/

一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响小偷偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组 nums ，请计算   不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

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

经典问题，写好状态转移方程就简单了。

### 剑指 Offer II 090. 环形房屋偷盗

https://leetcode.cn/problems/PzWKhm/

一个专业的小偷，计划偷窃一个环形街道上沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。

给定一个代表每个房屋存放金额的非负整数数组 nums ，请计算   在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

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

与上一题不同的是因为最前面和最后面不能同时偷，所以选一个偷就好了。

### 剑指 Offer 03. 数组中重复的数字

https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/

找出数组中重复的数字。
在一个长度为 n 的数组 nums 里的所有数字都在 0 ～ n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

```cpp
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        unordered_set<int> S;
        for(int i = 0; i < nums.size(); i++) {
            if(S.count(nums[i])) return nums[i];
            S.insert(nums[i]);
        }
        return 0;
    }
};
```

用哈希集合即可。

### 剑指 Offer 04. 二维数组中的查找

https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/

在一个 n \* m 的二维数组中，每一行都按照从左到右   非递减   的顺序排序，每一列都按照从上到下   非递减   的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```cpp
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        if(!matrix.size()) return false;
        if(!matrix[0].size()) return false;
        for(int i = 0; i < matrix.size(); i++) {
            if (func(matrix[i], target)) return true;
        }
        return false;
    }

    bool func(vector<int>& nums, int target) {
        int l = 0, r = nums.size() - 1;
        while(l < r) {
            int mid = (l + r) / 2;
            if(nums[mid] >= target) r = mid;
            else l = mid + 1;
        }
        if(nums[r] != target) return false;
        return true;
    }
};
```

二分即可。
