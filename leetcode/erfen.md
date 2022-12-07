## 二分查找

### 704. 二分查找

https://leetcode.cn/problems/binary-search

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;
        while(left < right){
            int mid = left + right >> 1;
            if(nums[mid] >= target) right = mid;
            else left = mid + 1;
        }
        if(nums[right] != target) return -1;
        return right;
    }
};
```

板子题，注意判断 `nums[right] != target` 的情况

### 278. 第一个错误的版本

https://leetcode.cn/problems/first-bad-version

你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。

假设你有 n 个版本 [1, 2, ..., n]，你想找出导致之后所有版本出错的第一个错误的版本。

你可以通过调用 bool isBadVersion(version) 接口来判断版本号 version 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。

```cpp
class Solution {
public:
    long firstBadVersion(long n) {
        long left = 1;
        long right = n;
        while(left < right){
            long mid = (left + right) >> 1;
            if(isBadVersion(mid)) right = mid;
            else left = mid + 1;
        }
        return right;
    }
};
```

与板子题基本类似，只不过左右区间以及判断条件有所不同。

### 35. 搜索插入位置

https://leetcode.cn/problems/search-insert-position

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();
        while(left < right){
            int mid = left + right >> 1;
            if(nums[mid] >= target) right = mid;
            else left = mid + 1;
        }
        return right;
    }
};
```

需要注意的是 `right = nums.size()` 并不减一，因为有可能插入的位置会在最后一个。