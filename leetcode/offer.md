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

```cpp
class Solution {
public:
    string replaceSpace(string s) {
        string res;
        for(auto c : s) {
            if(c == ' ') {
                res.push_back('%');
                res.push_back('2');
                res.push_back('0');
            } else res.push_back(c);
        }
        return res;
    }
};
```

简单做法，非原地。

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

### 剑指 Offer 06. 从尾到头打印链表

https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

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
    vector<int> reversePrint(ListNode* head) {
        vector<int> res;
        while(head) {
            res.push_back(head->val);
            head = head->next;
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

从前往后打印再翻转一下即可。

### 剑指 Offer 07. 重建二叉树

https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/

输入某二叉树的前序遍历和中序遍历的结果，请构建该二叉树并返回其根节点。

假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

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
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return dfs(preorder, 0, preorder.size() - 1,
        inorder, 0, inorder.size() - 1);
    }

    TreeNode* dfs(vector<int>& preorder, int pst, int ped, vector<int>& inorder, int ist, int ied) {
        if(pst > ped) return NULL;
        int pval = preorder[pst], idx = -1;
        auto root = new TreeNode(pval);
        for(int i = ist; i <= ied; i++) {
            if(inorder[i] == pval) {
                idx = i;
                break;
            }
        }
        root->left = dfs(preorder, pst + 1, pst + idx - ist, inorder, ist, idx - 1);
        root->right = dfs(preorder, pst + idx - ist + 1, ped, inorder, idx + 1, ied);
        return root;
    }
};
```

经典，注意左右递归时的条件，画图会很清晰。

### 剑指 Offer 09. 用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead  操作返回 -1 )

```cpp
class CQueue {
public:
    CQueue() {}
    stack<int> s1, s2;

    void appendTail(int value) {
        s1.push(value);
    }

    int deleteHead() {
        if(s1.empty() && s2.empty()) return -1;
        if(!s2.size()) {
            while(s1.size()) {
                s2.push(s1.top());
                s1.pop();
            }
        }
        int t = s2.top();
        s2.pop();
        return t;
    }
};

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue* obj = new CQueue();
 * obj->appendTail(value);
 * int param_2 = obj->deleteHead();
 */
```

用一个栈当队列，另外一个用来辅助即可。

### 剑指 Offer 10- I. 斐波那契数列

https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项（即 F(N)）。斐波那契数列的定义如下：

F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

```cpp
class Solution {
public:
    int fib(int n) {
        if(n < 2) return n;
        int i_1 = 0, i_2 = 0, i_3 = 1;
        for(int i = 2; i <= n; i++) {
            i_1 = i_2;
            i_2 = i_3;
            i_3 = (i_1 + i_2) % 1000000007;
        }
        return i_3;
    }
};
```

用动态规划，斐波那契数的边界条件是 F(0)=0F(0)=0 和 F(1)=1F(1)=1。当 n>1n>1 时，每一项的和都等于前两项的和，因此有如下递推关系：

```cpp
F(n)=F(n-1)+F(n-2)
F(n)=F(n−1)+F(n−2)
```

由于斐波那契数存在递推关系，因此可以使用动态规划求解。动态规划的状态转移方程即为上述递推关系，边界条件为 F(0)F(0) 和 F(1)F(1)。

### 剑指 Offer 10- II. 青蛙跳台阶问题

https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/

一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级台阶。求该青蛙跳上一个 n  级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

```cpp
class Solution {
public:
    int numWays(int n) {
        if(n < 2) return 1;
        int i_1 = 0, i_2 = 1, i_3 = 1;
        for(int i = 2; i <= n; i++) {
            i_1 = i_2;
            i_2 = i_3;
            i_3 = (i_1 + i_2) % 1000000007;
        }
        return i_3;
    }
};
```

斐波那契数列问题，和上一题一样的思路。

### 剑指 Offer 11. 旋转数组的最小数字

https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

给你一个可能存在   重复   元素值的数组  numbers ，它原来是一个升序排列的数组，并按上述情形进行了一次旋转。请返回旋转数组的最小元素。例如，数组  [3,4,5,1,2] 为 [1,2,3,4,5] 的一次旋转，该数组的最小值为 1。

注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。

```cpp
class Solution {
public:
    int minArray(vector<int>& numbers) {
        int l = 0, r = numbers.size() - 1;
        while(l < r) {
            int mid = (l + r) / 2;
            if(numbers[mid] > numbers[r]) l = mid + 1;
            else if(numbers[mid] < numbers[r]) r = mid;
            else r--;
        }
        return numbers[r];
    }
};
```

这道题需要用到二分，当 nums[m] > nums[j] 时： mm 一定在 左排序数组 中，即旋转点 xx 一定在 [m + 1, j] 闭区间内，因此执行 i = m + 1；
当 nums[m] < nums[j] 时： mm 一定在 右排序数组 中，即旋转点 xx 一定在[i, m] 闭区间内，因此执行 j = m；
当 nums[m] = nums[j] 时： 无法判断 mm 在哪个排序数组中，即无法判断旋转点 xx 在 [i, m] 还是 [m + 1, j] 区间中。解决方案： 执行 j = j - 1 缩小判断范围。

### 剑指 Offer 12. 矩阵中的路径

https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/

给定一个  m x n 二维字符网格  board 和一个字符串单词  word 。如果  word 存在于网格中，返回 true ；否则，返回 false 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

```cpp
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(board.empty() || board[0].empty()) return false;
        int n = board.size(), m = board[0].size();
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(dfs(board, word, 0, i, j)) return true;
            }
        }
        return false;
    }

    int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};

    bool dfs(vector<vector<char>>& board, string word, int u, int x, int y) {
        if(board[x][y] != word[u]) return false;
        if(u == word.size() - 1) return true;
        int n = board.size(), m = board[0].size();
        int t = board[x][y];
        board[x][y] = '.';
        for(int i = 0; i < 4; i++) {
            int a = x + dx[i], b = y + dy[i];
            if(a < 0 || b < 0 || a >= n || b >= m || board[a][b] == '.') continue;
            if(dfs(board, word, u + 1, a, b)) return true;
        }
        board[x][y] = t;
        return false;
    }
};
```

通过深搜，上下左右找即可。

### 剑指 Offer 14- I. 剪绳子

https://leetcode.cn/problems/jian-sheng-zi-lcof/

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n 都是整数，n>1 并且 m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]_k[1]_...\*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是 8 时，我们把它剪成长度分别为 2、3、3 的三段，此时得到的最大乘积是 18。

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

### 剑指 Offer 14- II. 剪绳子 II

https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m  段（m、n 都是整数，n>1 并且 m>1），每段绳子的长度记为 k[0],k[1]...k[m - 1] 。请问 k[0],k[1]...\*k[m - 1] 可能的最大乘积是多少？例如，当绳子的长度是 8 时，我们把它剪成长度分别为 2、3、3 的三段，此时得到的最大乘积是 18。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

```cpp
class Solution {
public:
    int cuttingRope(int n) {
        long long res = 1;
        if(n < 4) return n - 1;
        if(n == 4) return n;
        while(n > 4) {
            res *= 3;
            res %= 1000000007;
            n -= 3;
        }
        return (res * n) % 1000000007;
    }
};
```

用动态规划会爆范围，只能贪心，数学证明太难，建议背过即可。

### 剑指 Offer 15. 二进制中 1 的个数

https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/

编写一个函数，输入是一个无符号整数（以二进制串的形式），返回其二进制表达式中数字位数为 '1' 的个数（也被称为 汉明重量).）。

```cpp
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int sum = 0;
        while(n > 0) {
            n = n & (n - 1);
            sum++;
        }
        return sum;
    }
};
```

通过 `n = n & (n - 1)` 每次都能消灭一个 1，直到 n 为 0 时退出即可。

### 剑指 Offer 16. 数值的整数次方

https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/

实现 pow(x, n) ，即计算 x 的整数 n 次幂函数（即，xn ）。

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        typedef long long ll;
        double res = 1;
        for(ll i = abs(ll(n)); i; i >>= 1) {
            if(i & 1) res *= x;
            x *= x;
        }
        if(n < 0) return 1 / res;
        return res;
    }
};
```

按照定义，计算 x 的 n 次方是将 nn 个 x 连乘，效率比较低，会超时。因为乘法具有结合律，考虑每次将一部分连乘批量计算好，作为最终答案的一部分。这就可以将 n 进行二进制拆分，若 n 的二进制位的第 k 位是 1，则 ans 可以乘上 x2k。
而计算 x2k，只需每次将自身做平方即可。

### 剑指 Offer 17. 打印从 1 到最大的 n 位数

https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/

```cpp
class Solution {
public:
    vector<int> printNumbers(int n) {
        int max = 1;
        for(int i = 0; i < n; i++) max *= 10;
        vector<int> res;
        for(int i = 1; i < max; i++) res.push_back(i);
        return res;
    }
};
```

没搞懂这道题存在的意义。

### 剑指 Offer 18. 删除链表的节点

https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/

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
    ListNode* deleteNode(ListNode* head, int val) {
        if(!head) return head;
        auto dummy = new ListNode();
        dummy->next = head;
        auto p = dummy;
        while(head) {
            if(head->val == val) p->next = p->next->next;
            head = head->next;
            p = p->next;
        }
        return dummy->next;
    }
};
```

用一个虚拟头结点轻松秒杀。

### 剑指 Offer 20. 表示数值的字符串

https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。

数值（按顺序）可以分成以下几个部分：

若干空格
一个   小数   或者   整数
（可选）一个  'e'  或  'E' ，后面跟着一个   整数
若干空格
小数（按顺序）可以分成以下几个部分：

（可选）一个符号字符（'+' 或 '-'）
下述格式之一：
至少一位数字，后面跟着一个点 '.'
至少一位数字，后面跟着一个点 '.' ，后面再跟着至少一位数字
一个点 '.' ，后面跟着至少一位数字
整数（按顺序）可以分成以下几个部分：

（可选）一个符号字符（'+' 或 '-'）
至少一位数字
部分数值列举如下：

["+100", "5e2", "-123", "3.1416", "-1E-16", "0123"]
部分非数值列举如下：

["12e", "1a3.14", "1.2.3", "+-5", "12e+5.4"]

```cpp
class Solution {
public:
    bool isNumber(string s) {
        //去掉首尾空格
        int i = 0;
        while (i < s.size() && s[i] == ' ') i++;
        s = s.substr(i);
        while (s.back() == ' ') s.pop_back();

        bool numFlag = false;
        bool dotFlag = false;
        bool eFlag = false;
        for (int i = 0; i < s.size(); i++) {
            // 判定为数字，则标记numFlag
            if (isdigit(s[i])) numFlag = true;
            // 判定为'.'需要没出现过'.'并且没出现过'e'
            else if (s[i] == '.' && !dotFlag && !eFlag) dotFlag = true;
            // 判定为'e'，需要没出现过'e'，并且出现过数字
            else if ((s[i] == 'e' || s[i] == 'E') && !eFlag && numFlag) {
                eFlag = true;
                numFlag = false; // 'e'后面必须跟着一个整数，所以出现'e'之后就标志为false
            }
            // 判定为'+''-'符号，只能出现在第一位或者紧接'e'后面
            else if ((s[i] == '+' || s[i] == '-') && (i == 0 || s[i - 1] == 'e' || s[i - 1] == 'E')) continue;
            // 其他情况，都是非法的
            else return false;
        }
        return numFlag;
    }
};
```

详看注释。

### 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面

https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数在数组的前半部分，所有偶数在数组的后半部分。

```cpp
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int s = 0, f = 0;
        while(f < nums.size()) {
            if(nums[f] % 2 == 1) {
                swap(nums[s], nums[f]);
                s++;
            }
            f++;
        }
        return nums;
    }
};
```

### 剑指 Offer 24. 反转链表

https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/

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
    ListNode* reverseList(ListNode* head) {
        if(!head) return head;
        auto a = head, b = head->next;
        while(b) {
            auto t = b->next;
            b->next = a;
            a = b;
            b = t;
        }
        head->next = NULL;
        return a;
    }
};
```

原地算法，经典问题。

### 剑指 Offer 26. 树的子结构

https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/

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
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if(!A || !B) return false;
        return isSub(A, B) || isSubStructure(A->left, B) || isSubStructure(A->right, B);
    }

    bool isSub(TreeNode* A, TreeNode* B) {
        if(!B) return true;
        if(!A || A->val != B->val) return false;
        return isSub(A->left, B->left) && isSub(A->right, B->right);
    }
};
```

递归，主要有两种情况，可能是根节点开始的子树，也有可能是后继节点开始的子树，所以需要把三个或起来，其中 `isSub` 进行了对两边的递归。

### 剑指 Offer 27. 二叉树的镜像

https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/

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
    TreeNode* mirrorTree(TreeNode* root) {
        dfs(root);
        return root;
    }

    void dfs(TreeNode* root) {
        if(!root) return;

        auto t = root->left;
        root->left = root->right;
        root->right = t;

        dfs(root->left);
        dfs(root->right);
    }
};
```

用一个 dfs 函数遍历每个节点，让每个节点的左右子节点颠倒过来就行了。

### 剑指 Offer 28. 对称的二叉树

https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/

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
    bool isSymmetric(TreeNode* root) {
        if(!root) return true;
        return dfs(root->left, root->right);
    }

    bool dfs(TreeNode* left, TreeNode* right) {
        if(!left && !right) return true;
        if(!left || !right || left->val != right->val) return false;
        return dfs(left->left, right->right) && dfs(left->right, right->left);
    }
};
```

遍历两边进行比较即可。

### 剑指 Offer 29. 顺时针打印矩阵

https://leetcode.cn/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if(matrix.size() == 0) return {};
        vector<int> res;
        int n = matrix.size(), m = matrix[0].size();
        vector<vector<bool>> st(n, vector<bool>(m));
        int dx[4] = {0, 1, 0, -1}, dy[4] = {1, 0, -1, 0};
        for(int i = 0, x = 0, y = 0, d = 0; i < n * m; i++) {
            res.push_back(matrix[x][y]);
            st[x][y] = true;
            int a = x + dx[d], b = y + dy[d];
            if(a < 0 || b < 0 || a >= n || b >= m || st[a][b]) {
                d = (d + 1) % 4;
                a = x + dx[d], b = y + dy[d];
            }
            x = a, y = b;
        }
        return res;
    }
};
```

用偏移量实现，不用模拟。

### 剑指 Offer 30. 包含 min 函数的栈

https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

```cpp
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int> s;
    stack<int> t;
    MinStack() {
        t.push(INT_MAX);
    }

    void push(int x) {
        s.push(x);
        t.push(std::min(t.top(), x);
    }

    void pop() {
        s.pop();
        t.pop();
    }

    int top() {
        return s.top();
    }

    int min() {
        return t.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->min();
 */
```

用一个辅助栈来存每次的最小值即可。

### 剑指 Offer 32 - I. 从上到下打印二叉树

https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

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
    vector<int> levelOrder(TreeNode* root) {
        vector<int> res;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()) {
            int len = q.size();
            while(len--) {
                auto t = q.front();
                res.push_back(t->val);
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
        }
        return res;
    }
};
```

经典层序遍历。

### 剑指 Offer 32 - II. 从上到下打印二叉树 II

https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/

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
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()) {
            int len = q.size();
            vector<int> path;
            while(len--) {
                auto t = q.front();
                q.pop();
                path.push_back(t->val);
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
            res.push_back(path);
        }
        return res;
    }
};
```

同样是层序遍历

### 剑指 Offer 31. 栈的压入、弹出序列

https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/

```cpp
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int> stk;
        int n = pushed.size();
        for (int i = 0, j = 0; i < n; ++i) {
            stk.push(pushed[i]);
            while (!stk.empty() && stk.top() == popped[j]) {
                stk.pop();
                j++;
            }
        }
        return stk.empty();
    }
};
```

模拟一下。

### 剑指 Offer 32 - III. 从上到下打印二叉树 III

https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/

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
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        queue<TreeNode*> q;
        if(root) q.push(root);
        bool flag = true;
        while(q.size()) {
            int len = q.size();
            vector<int> path;
            while(len--) {
                auto t = q.front();
                path.push_back(t->val);
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
            if(flag) {
                res.push_back(path);
                flag = false;
            } else {
                reverse(path.begin(), path.end());
                res.push_back(path);
                flag = true;
            }
        }
        return res;
    }
};
```

层序遍历问题，用一个变量来存当前应该正序输入还是倒序输入。

### 剑指 Offer 33. 二叉搜索树的后序遍历序列

https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。

```cpp
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        return check(postorder, 0, postorder.size() - 1);
    }

    bool check(vector<int>& postorder, int i, int j) {
        if(i >= j) return true;
        int root = postorder[j];
        int left = i;
        while(left < j && postorder[left] < root) left++;
        int right = left;
        while(right < j && postorder[right] > root) right++;
        if(right != j) return false;
        return check(postorder, i, left - 1) && check(postorder, left, j - 1);
    }
};
```

1、先找到根节点元素

2、根据根节点元素找到左子树元素，递归检查左子树是否是 BST

3、根据根节点元素找到右子树元素，递归检查右子树是否是 BST

### 剑指 Offer 34. 二叉树中和为某一值的路径

https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/

给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。

叶子节点 是指没有子节点的节点。

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
    vector<vector<int>> res;
    vector<int> path;
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        if(!root) return {};
        dfs(root, target);
        return res;
    }

    void dfs(TreeNode* root, int target) {
        path.push_back(root->val);
        target -= root->val;
        if(!root->left && !root->right) {
            if(!target) res.push_back(path);
        } else {
            if(root->left) dfs(root->left, target);
            if(root->right) dfs(root->right, target);
        }
        path.pop_back();
    }
};
```

遍历所有的可能，记得回溯。

### 剑指 Offer 35. 复杂链表的复制

https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;

    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(!head) return NULL;
        Node* cur = head;
        unordered_map<Node*, Node*> map;
        while(cur) {
            map[cur] = new Node(cur->val);
            cur = cur->next;
        }
        cur = head;
        while(cur) {
            map[cur]->next = map[cur->next];
            map[cur]->random = map[cur->random];
            cur = cur->next;
        }
        return map[head];
    }
};
```

利用哈希表的查询特点，考虑构建 原链表节点 和 新链表对应节点 的键值对映射关系，再遍历构建新链表各节点的 next 和 random 引用指向即可。

### 剑指 Offer 36. 二叉搜索树与双向链表

https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* left;
    Node* right;

    Node() {}

    Node(int _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }

    Node(int _val, Node* _left, Node* _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
public:
    Node* head = NULL;
    Node* pre = NULL;
    Node* treeToDoublyList(Node* root) {
        if(!root) return NULL;
        dfs(root);
        pre->right = head;
        head->left = pre;
        return head;
    }

    void dfs(Node* root) {
        if(!root) return;
        dfs(root->left);
        if(pre) pre->right = root;
        else head = root;
        root->left = pre;
        pre = root;
        dfs(root->right);
    }
};
```

1、我们定义两个指针 pre 和 head，pre 指针用于保存中序遍历的前一个节点，head 指针用于记录排序链表的头节点。

2、中序遍历二叉树，因为是中序遍历，所以遍历顺序就是双线链表的建立顺序。我们只需要在中序遍历的过程中，修改每个节点的左右指针，将零散的节点连接成双向循环链表。

### 剑指 Offer 38. 字符串的排列

https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/

```cpp
class Solution {
public:
    vector<string> ans;
    vector<bool> st;
    string path;
    vector<string> permutation(string s) {
        st = vector<bool>(s.size());
        dfs(s, 0);
        return ans;
    }

    void dfs(string s, int u) {
        if(u == s.size()) {
            ans.push_back(path);
            return;
        }

        unordered_set<int> S;

        for(int i = 0; i < s.size(); i++){
            if(!st[i]) {
                if(S.count(s[i])) continue;
                S.insert(s[i]);
                st[i] = true;
                path.push_back(s[i]);
                dfs(s, u + 1);
                st[i] = false;
                path.pop_back();
            }
        }
    }
};
```

和全排列一样的思路。

### 剑指 Offer 39. 数组中出现次数超过一半的数字

https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/

```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int len = nums.size() / 2;
        unordered_map<int, int> S;
        for(int i = 0; i < nums.size(); i++) {
            S[nums[i]]++;
            if(S[nums[i]] > len) return nums[i];
        }
        return 0;
    }
};
```

按题意使用哈希表即可。

### 剑指 Offer 40. 最小的 k 个数

https://leetcode.cn/problems/zui-xiao-de-kge-shu-lcof/

```cpp
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        vector<int> res;
        sort(arr.begin(), arr.end());
        for(int i = 0; i < k; i++) res.push_back(arr[i]);
        return res;
    }
};
```

建议使用快排。

### 剑指 Offer 41. 数据流中的中位数

https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/

```cpp
class MedianFinder {
public:
    priority_queue<int, vector<int>, less<int>> left;
    priority_queue<int, vector<int>, greater<int>> right;
    /** initialize your data structure here. */
    MedianFinder() {
    }

    void addNum(int num) {
        // 右侧堆的每一个值一定是来源于左侧堆，这样才能保证右侧堆的每一个元素都比左侧堆大
        left.push(num);
        right.push(left.top());
        left.pop();
        if (right.size() > left.size()){
            left.push(right.top());
            right.pop();
        }
    }

    double findMedian() {
        if (left.size() > right.size())
            return left.top();
        else
            return (left.top() + right.top()) / 2.0;
    }
};
```

左侧堆的数量大于等于右侧堆，等于时表示数列偶数个，中位数就是左侧数的最大值和右侧数的最小值求平均，大于时表示数列奇数个，左侧堆的 top 就是答案。

### 剑指 Offer 19. 正则表达式匹配

https://leetcode.cn/problems/zheng-ze-biao-da-shi-pi-pei-lcof/

请实现一个函数用来匹配包含'. '和 _ 的正则表达式。模式中的字符'.'表示任意一个字符，而 _ 表示它前面的字符可以出现任意次（含 0 次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab\*ac\*a"匹配，但与"aa.a"和"ab\*a"均不匹配。

```cpp
class Solution {
public:
    // 备忘录
    vector<vector<int>> memo;

    bool isMatch(string s, string p) {
        int m = s.size(), n = p.size();
        memo = vector<vector<int>>(m, vector<int>(n, -1));
        // 指针 i，j 从索引 0 开始移动
        return dp(s, 0, p, 0);
    }

    /* 计算 p[j..] 是否匹配 s[i..] */
    bool dp(string& s, int i, string& p, int j) {
        int m = s.size(), n = p.size();
        // base case
        if (j == n) return i == m;
        if (i == m) {
            if ((n - j) % 2 == 1) return false;
            for (; j < n - 1; j += 2) if (p[j + 1] != '*') return false;
            return true;
        }

        // 查备忘录，防止重复计算
        if (memo[i][j] != -1) return memo[i][j];
        bool res = false;
        if (s[i] == p[j] || p[j] == '.') {
            if (j < n - 1 && p[j + 1] == '*') res = dp(s, i, p, j + 2) || dp(s, i + 1, p, j);
            else res = dp(s, i + 1, p, j + 1);
        } else {
            if (j < n - 1 && p[j + 1] == '*') res = dp(s, i, p, j + 2);
            else res = false;
        }
        // 将当前结果记入备忘录
        memo[i][j] = res;
        return res;
    }
};
```
