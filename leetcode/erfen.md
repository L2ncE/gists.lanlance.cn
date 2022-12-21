### 704. 二分查找

https://leetcode.cn/problems/binary-search

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

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

### 34. 在排序数组中查找元素的第一个和最后一个位置

https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        if(nums.empty()) return {-1, -1};
        int left = 0, right = nums.size() - 1;
        int i, j;
        while(left < right){
            int mid = left + right >> 1;
            if(nums[mid] >= target) right = mid;
            else left = mid + 1;
        }
        if(nums[right] != target) return {-1, -1};
        i = right;

        left = 0, right = nums.size() - 1;
        while(left < right){
            int mid = left + right + 1 >> 1;
            if(nums[mid] <= target) left = mid;
            else right = mid - 1;
        }
        j = left;

        return {i, j};
    }
};
```

和一般都二分区别不大，只是左右两种情况都要走一遍。

### 69. x 的平方根

https://leetcode.cn/problems/sqrtx/

给你一个非负整数 x ，计算并返回 x 的 算术平方根 。

由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。

注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x \*\* 0.5 。

```cpp
class Solution {
public:
    int mySqrt(int x) {
        int l = 0, r = x;
        while(l < r){
            int mid = l + 1ll + r >> 1;
            if(mid <= x / mid) l = mid;
            else r = mid - 1;
        }
        return r;
    }
};
```

有两处地方都有可能溢出，取了个巧。

### 367. 有效的完全平方数

https://leetcode.cn/problems/valid-perfect-square/

给定一个 **正整数** `num` ，编写一个函数，如果 `num` 是一个完全平方数，则返回 `true` ，否则返回 `false` 。

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        int left = 0, right = num;
        while(left < right){
            int mid = left + 1ll + right >> 1;
            if(mid <= num / mid) left = mid;
            else right = mid - 1;
        }
        if (right * right != num) return false;
        return true;
    }
};
```

和上题一样，注意溢出。
