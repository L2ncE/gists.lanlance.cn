### 217. 存在重复元素

https://leetcode.cn/problems/contains-duplicate

给你一个整数数组 `nums` 。如果任一值在数组中出现 **至少两次** ，返回 `true` ；如果数组中每个元素互不相同，返回 `false` 。

```cpp
class Solution {
public:
    bool containsDuplicate(vector<int>& nums) {
        unordered_set<int> Hash;
        for(auto i : nums){
            if(Hash.count(i)) return true;
            else Hash.insert(i);
        }
        return false;
    }
};
```

哈希表基础使用，若插入后还能 `count` 到就说明在数组中出现了至少两次。

### 1. 两数之和

https://leetcode.cn/problems/two-sum/

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target 的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> Hash;
        for(int i = 0; i < nums.size(); i++){
            int need = target - nums[i];
            if(Hash.count(need)) return {Hash[need], i};
            Hash[nums[i]] = i;
        }
        return {};
    }
};
```

开一个哈希表，每过一个就把值和下标存表里，这样查询时只需要 O(1) 的时间复杂度。

### 350. 两个数组的交集 II

https://leetcode.cn/problems/intersection-of-two-arrays-ii/

给你两个整数数组 nums1 和 nums2 ，请你以数组形式返回两数组的交集。返回结果中每个元素出现的次数，应与元素在两个数组中都出现的次数一致（如果出现次数不一致，则考虑取较小值）。可以不考虑输出结果的顺序。

```cpp
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        unordered_multiset<int> Hash;
        vector<int> result;
        for(int i = 0; i < nums1.size(); i++){
            Hash.insert(nums1[i]);
        }
        for(int i = 0; i < nums2.size(); i++){
            if(Hash.count(nums2[i])){
                Hash.erase(Hash.find(nums2[i]));
                result.push_back(nums2[i]);
            }
        }
        return result;
    }
};
```

不难，但需要注意我们这里使用的是 `unodered_multiset` ，这个哈希集支持有多个相同的数，并且在删除的时候使用 `Hash.find()` ，因为如果直接删除的话会把所有这个数删掉。

### 387. 字符串中的第一个唯一字符

https://leetcode.cn/problems/first-unique-character-in-a-string/

给定一个字符串 `s` ，找到 _它的第一个不重复的字符，并返回它的索引_ 。如果不存在，则返回 `-1` 。

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        unordered_map<char, int> Hash;
        for(int i = 0; i < s.length(); i++){
            Hash[s[i]]++;
        }
        for(int i = 0; i < s.length(); i++){
            if(Hash[s[i]] == 1) return i;
        }
        return -1;
    }
};
```

用哈希表存，然后遍历两次即可。

### 383. 赎金信

https://leetcode.cn/problems/ransom-note/

给你两个字符串：ransomNote 和 magazine ，判断 ransomNote 能不能由 magazine 里面的字符构成。

如果可以，返回 true ；否则返回 false 。

magazine 中的每个字符只能在 ransomNote 中使用一次。

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        unordered_map<char, int> h;
        for(int i = 0; i < magazine.length(); i++){
            h[magazine[i]]++;
        }
        for(int i = 0; i < ransomNote.length(); i++){
            h[ransomNote[i]]--;
            if(h[ransomNote[i]] < 0) return false;
        }
        return true;
    }
};
```

用哈希表存，遍历两个字符串即可。

### 242. 有效的字母异位词

https://leetcode.cn/problems/valid-anagram/

给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

注意：若 s 和 t 中每个字符出现的次数都相同，则称 s 和 t 互为字母异位词。

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        unordered_map<char, int> h1, h2;
        for(int i = 0; i < s.length(); i++){
            h1[s[i]]++;
        }
        for(int i = 0; i < t.length(); i++){
            h2[t[i]]++;
        }
        return h1 == h2;
    }
};
```

用两个哈希表来存，最后判断是否相等即可。

### 141. 环形链表

https://leetcode.cn/problems/linked-list-cycle/

给你一个链表的头节点 head ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。注意：pos 不作为参数进行传递 。仅仅是为了标识链表的实际情况。

如果链表中存在环 ，则返回 true 。 否则，返回 false 。

**哈希表**

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
    bool hasCycle(ListNode *head) {
        unordered_set<ListNode*> seen;
        while (head) {
            if (seen.count(head)) return true;
            seen.insert(head);
            head = head->next;
        }
        return false;
    }
};
```

遍历链表观察是否有相同的节点，如果有就说明是环形链表。

**快慢指针**

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
    bool hasCycle(ListNode *head) {
        if(!head || !head->next) return false;
        auto slow = head, fast = head->next;
        while (fast && fast->next) {
            slow = slow->next, fast = fast->next->next;
            if(fast == slow) return true;
        }
        return false;
    }
};
```

快的比慢的跑得快，总的遇到，要不然就是没有，直接返回 false。

### 349. 两个数组的交集

https://leetcode.cn/problems/intersection-of-two-arrays/

给定两个数组 nums1 和 nums2 ，返回 它们的交集 。输出结果中的每个元素一定是 唯一 的。我们可以 不考虑输出结果的顺序 。

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        vector<int> res;
        unordered_map<int, int> h1, h2;
        for(int i = 0; i < nums1.size(); i++){
            h1[nums1[i]]++;
        }
        for(int i = 0; i < nums2.size(); i++){
            if(h1[nums2[i]] != 0 && h2[nums2[i]] == 0){
                res.push_back(nums2[i]);
                h2[nums2[i]]++;
            }
        }
        return res;
    }
};
```

两个哈希表即可解决问题，第二个哈希表用于判断答案里是否已经有这个元素了。

### 454. 四数相加 II

https://leetcode.cn/problems/4sum-ii/

给你四个整数数组 nums1、nums2、nums3 和 nums4 ，数组长度都是 n ，请你计算有多少个元组 (i, j, k, l) 能满足：

0 <= i, j, k, l < n
nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0

```cpp
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int, int> h;
        int n = nums1.size();
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                h[nums1[i] + nums2[j]]++;
            }
        }
        int res = 0;
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                res += h[- nums3[i] - nums4[j]];
            }
        }
        return res;
    }
};
```

用一个哈希表存两个数的和，再遍历另外两个数看是不是相反数，总和为 0 时就把哈希表中存的个数加到答案里面。
