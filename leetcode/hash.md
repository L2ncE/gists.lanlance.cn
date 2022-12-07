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

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

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