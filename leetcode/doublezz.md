### 977. 有序数组的平方

https://leetcode.cn/problems/squares-of-a-sorted-array/comments/

给你一个按 **非递减顺序** 排序的整数数组 `nums`，返回 **每个数字的平方** 组成的新数组，要求也按 **非递减顺序** 排序。

**暴力排序法**

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        for(int i = 0; i < nums.size(); i++){
            nums[i] = nums[i] * nums[i];
        }
        sort(nums.begin(), nums.end());
        return nums;
    }
};
```

**双指针法**

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int> &nums) {
        vector<int> result(nums.size(), 0);
        int k = nums.size() - 1;
        for (int i = 0, j = k; i < nums.size(), j >= i;) {
            if (nums[j] * nums[j] > nums[i] * nums[i]) {
                result[k--] = nums[j] * nums[j];
                j--;
            } else {
                result[k--] = nums[i] * nums[i];
                i++;
            }
        }
        return result;
    }
};
```

数组其实是有序的， 只不过负数平方之后可能成为最大数了。那么数组平方的最大值就在数组的两端，不是最左边就是最右边，不可能是中间。此时可以考虑双指针法了，i 指向起始位置，j 指向终止位置。定义一个新数组 result，和 nums 数组一样的大小，让 k 指向 result 数组终止位置。

### 189. 轮转数组

https://leetcode.cn/problems/rotate-array/

给你一个数组，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

**暴力轮转**

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        vector<int> result(nums.size(), 0);
        k = k % nums.size();
        for(int i = 0; i < nums.size(); i++){
            if(i >= nums.size() - k){
                result[i + k - nums.size()] = nums[i];
                continue;
            }
            result[i + k] = nums[i];
        }
        nums = result;
    }
};
```

**双指针原地算法**

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        k = k % nums.size();
        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};
```

先全部翻转一遍，再翻转前一段，最后翻转后一段。

### 88. 合并两个有序数组

https://leetcode.cn/problems/merge-sorted-array/

给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。

请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。

注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int k = m + n - 1;
        int i = m - 1;
        int j = n - 1;
        while(j >= 0 && i >= 0){
            if(nums1[i] > nums2[j]) nums1[k--] = nums1[i--];
            else nums1[k--] = nums2[j--];
        }
        while(j >= 0) nums1[k--] = nums2[j--];
    }
};
```

因为我们要放在 nums1 里面，所以我们从后往前遍历，从最大的开始比较，一个一个放进去，最后 nums2 还有剩就全部扔进去，nums1 还有剩说明已经全部放在了正确的位置，不用管。

### 283. 移动零

https://leetcode.cn/problems/move-zeroes

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int j = 0;
        for(int i = 0; i < nums.size(); i++){
            if(nums[i] != 0){
                nums[j++] = nums[i];
            }
        }
        for(int i = j; i < nums.size(); i++){
            nums[i] = 0;
        }
    }
};
```

i 来遍历整个数组，遇到不是 0 的就交给 j 进行填充，最后再补 0。

### 167. 两数之和 II - 输入有序数组

https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted

给你一个下标从 1 开始的整数数组 numbers ，该数组已按 非递减顺序排列 ，请你从数组中找出满足相加之和等于目标数 target 的两个数。如果设这两个数分别是 numbers[index1] 和 numbers[index2] ，则 1 <= index1 < index2 <= numbers.length 。

以长度为 2 的整数数组 [index1, index2] 的形式返回这两个整数的下标 index1 和 index2。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int i = 0;
        int j = nums.size() - 1;
        while(i < j){
            if(nums[i] + nums[j] < target){
                i++;
            }else if(nums[i] + nums[j] > target){
                j--;
            }else{
                return {i + 1, j + 1};
            }
        }
        return {};
    }
};
```

i 在最小，j 在最大，如果大了 j 就减一，如果小了 i 就加一。这样做可以减少很多不必要情况的判断。

### 344. 反转字符串

https://leetcode.cn/problems/reverse-string

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `s` 的形式给出。

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        for(int i = 0, j = s.size() - 1; i < s.size() / 2; i++, j--){
            swap(s[i], s[j]);
        }
    }
};
```

无脑题，甚至直接 `reverse` 也能过。

### 557. 反转字符串中的单词 III

https://leetcode.cn/problems/reverse-words-in-a-string-iii

给定一个字符串 `s` ，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

```cpp
class Solution {
public:
    string reverseWords(string s) {
        int left = 0;
        for(int i = 0; i < s.length(); i++){
            if(s[i] == 32){
                reverse(s.begin() + left, s.begin() + i);
                left = i + 1;
            }
        }
        reverse(s.begin() + left, s.end());
        return s;
    }
};
```

遇到空格就反转之前的，最后再反转一下最后一个单词即可。`reverse` 可以用双指针实现。

### 876. 链表的中间结点

https://leetcode.cn/problems/middle-of-the-linked-list

给定一个头结点为 `head` 的非空单链表，返回链表的中间结点。

如果有两个中间结点，则返回第二个中间结点。

**普通双指针**

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
    ListNode* middleNode(ListNode* head) {
        int n = 0;
        auto q = head, p = head;
        while(p){
            p = p -> next;
            n++;
        }
        for(int i = 0; i < n / 2; i++){
            q = q -> next;
        }
        return q;
    }
};
```

遍历第一遍求长度，第二遍走到 n / 2 即可。

**快慢指针**

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
    ListNode* middleNode(ListNode* head) {
        auto q = head, p = head;
        while(q && q->next){
            p = p->next;
            q = q->next->next;
        }
        return p;
    }
};
```

奇数情况是 q 到最后一个就不行了，偶数情况是到最后一个的下一个，也就是空。

### 3. 无重复字符的最长子串

https://leetcode.cn/problems/longest-substring-without-repeating-characters/

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<int, int> Hash;
        int ans = 0;
        for(int i = 0, j = 0; i < s.size(); i++){
            Hash[s[i]] ++;
            while(Hash[s[i]] > 1) Hash[s[j++]]--;
            ans = max(ans, i - j + 1);
        }
        return ans;
    }
};
```

用一个哈希表来维护，当有重复字符的时候，j 指针就往右移，直到没有重复字符，最后取 max 即可。

### 567. 字符串的排列

https://leetcode.cn/problems/permutation-in-string/

给你两个字符串 s1 和 s2 ，写一个函数来判断 s2 是否包含 s1 的排列。如果是，返回 true ；否则，返回 false 。

换句话说，s1 的排列之一是 s2 的 子串 。

```cpp
class Solution {
public:
    unordered_map<int, int> h1;
    unordered_map<int, int> h2;
    bool check(int c){
        if(h1.count(c) && h1[c] == h2[c]) return true;
        return false;
    }
    bool checkInclusion(string s1, string s2) {
        for(int i = 0; i < s1.size(); i++){
            h1[s1[i]]++;
        }
        for(int i = 0, j = 0, cnt = 0; i < s2.size(); i++){
            if(check(s2[i])) cnt--;
            h2[s2[i]]++;
            if(check(s2[i])) cnt++;
            if(i - j >= s1.size()){
                if(check(s2[j])) cnt--;
                h2[s2[j]]--;
                if(check(s2[j])) cnt++;
                j++;
            }
            if(cnt == h1.size()) return true;
        }
        return false;
    }
};
```

用两个哈希表进行维护，最后比较两个哈希表相同的“柱子数”是否为 h1 的总长度即可。

### 27. 移除元素

https://leetcode.cn/problems/remove-element/

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

**暴力**

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int size = nums.size();
        for(int i = 0; i < size; i++){
            if(nums[i] == val) {
                for(int j = i + 1; j < size; j++){
                    nums[j - 1] = nums[j];
                }
                size--;
                i--;
            }
        }
        return size;
    }
};
```

遇到需要删除的就将后面的元素一起往前移一格即可。因为少了一个，所以 i 也需要往前一格。

**快慢指针**

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slow = 0;
        for(int fast = 0; fast < nums.size(); fast++){
            if(nums[fast] != val) {
                nums[slow++] = nums[fast];
            }
        }
        return slow;
    }
};
```

当快指针遇到需要删除的元素，慢指针不动，直到快指针指向不需要删除的数字，然后让快指针把不需删除的一个一个覆盖过去。

### 26. 删除有序数组中的重复项

https://leetcode.cn/problems/remove-duplicates-from-sorted-array/

给你一个 升序排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。

由于在某些语言中不能改变数组的长度，所以必须将结果放在数组 nums 的第一部分。更规范地说，如果在删除重复项之后有 k 个元素，那么 nums 的前 k 个元素应该保存最终结果。

将最终结果插入 nums 的前 k 个位置后返回 k 。

不要使用额外的空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int j = 0;
        for(int i = 0; i < nums.size(); i++){
            if(!i || nums[i] != nums[i - 1]){
                 nums[j++] = nums[i];
            }
        }
        return j;
    }
};
```

一个指在前面一个指在后面，经典双指针问题。注意特判 i = 0 的情况。

### 844. 比较含退格的字符串

https://leetcode.cn/problems/backspace-string-compare/

给定 s 和 t 两个字符串，当它们分别被输入到空白的文本编辑器后，如果两者相等，返回 true 。# 代表退格字符。

注意：如果对空文本输入退格字符，文本继续为空。

```cpp
class Solution {
public:
    void changestring(string &s) {
        int slow=0;
        for(int fast = 0; fast < s.size(); fast++)
        {
            if(s[fast] != '#') s[slow++] = s[fast];
            else if(slow) slow--;
        }
        s.resize(slow);
    }

    bool backspaceCompare(string s, string t) {
        changestring(s);
        changestring(t);
        return s==t;
    }
};
```

注意使用 `resize()` 将后面无用的字符给去掉。

### 209. 长度最小的子数组

https://leetcode.cn/problems/minimum-size-subarray-sum/

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int res = INT_MAX;
        for(int i = 0, j = 0, sum = 0; i < nums.size(); i++){
            sum += nums[i];
            while(sum >= target){
                res = min(res, i - j + 1);
                sum -= nums[j++];
            }
        }
        if(res == INT_MAX) res = 0;
        return res;
    }
};
```

经典滑动窗口问题，注意判断 res 没有被更新的情况。

### 904. 水果成篮

https://leetcode.cn/problems/fruit-into-baskets/

你正在探访一家农场，农场从左到右种植了一排果树。这些树用一个整数数组 fruits 表示，其中 fruits[i] 是第 i 棵树上的水果 种类 。

你想要尽可能多地收集水果。然而，农场的主人设定了一些严格的规矩，你必须按照要求采摘水果：

你只有 两个 篮子，并且每个篮子只能装 单一类型 的水果。每个篮子能够装的水果总量没有限制。
你可以选择任意一棵树开始采摘，你必须从 每棵 树（包括开始采摘的树）上 恰好摘一个水果 。采摘的水果应当符合篮子中的水果类型。每采摘一次，你将会向右移动到下一棵树，并继续采摘。
一旦你走到某棵树前，但水果不符合篮子的水果类型，那么就必须停止采摘。
给你一个整数数组 fruits ，返回你可以收集的水果的 最大 数目。

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int res = 0;
        unordered_map<int, int> h;
        for(int i = 0, j = 0, s = 0; i < fruits.size(); i++){
            if(++h[fruits[i]] == 1) s++;
            while(s > 2){
                if(--h[fruits[j++]] == 0) s--;
            }
            res = max(res, i - j + 1);
        }
        return res;
    }
};
```

用哈希表，并且用一个变量来保存哈希表中有几个不同的水果种类，每次这个种类清空了或者存在了都会对 s 有所改变。

### 76. 最小覆盖子串

https://leetcode.cn/problems/minimum-window-substring/

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
         unordered_map<char, int> hs;
         unordered_map<char, int> ht;
         for(auto c : t) ht[c]++;

         string res;
         for(int i = 0, j = 0, cnt = 0; i < s.length(); i++){
             if(++ hs[s[i]] <= ht[s[i]]) cnt++;
             while(hs[s[j]] > ht[s[j]]) hs[s[j++]]--;
             if(cnt == t.length()){
                 if(res.empty() || i - j + 1 < res.length()){
                     res = s.substr(j, i - j + 1);
                 }
             }
         }
         return res;
    }
};
```

用两个哈希表和一个 cnt 进行维护，经典滑动窗口。
