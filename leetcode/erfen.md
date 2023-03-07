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

### 154. 寻找旋转排序数组中的最小值 II

https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/

已知一个长度为 n 的数组，预先按照升序排列，经由 1 到 n 次 旋转 后，得到输入数组。例如，原数组 nums = [0,1,4,4,5,6,7] 在变化后可能得到：
若旋转 4 次，则可以得到 [4,5,6,7,0,1,4]
若旋转 7 次，则可以得到 [0,1,4,4,5,6,7]
注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。

给你一个可能存在 重复 元素值的数组 nums ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 最小元素 。

你必须尽可能减少整个过程的操作步骤。

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int l = 0, r = nums.size() - 1;
        while(l < r) {
            int mid = (l + r) / 2;
            if(nums[mid] > nums[r]) l = mid + 1;
            else if(nums[mid] < nums[r]) r = mid;
            else r--;
        }
        return nums[r];
    }
};
```

这道题需要用到二分，当 nums[m] > nums[j] 时： mm 一定在 左排序数组 中，即旋转点 xx 一定在 [m + 1, j] 闭区间内，因此执行 i = m + 1；
当 nums[m] < nums[j] 时： mm 一定在 右排序数组 中，即旋转点 xx 一定在[i, m] 闭区间内，因此执行 j = m；
当 nums[m] = nums[j] 时： 无法判断 mm 在哪个排序数组中，即无法判断旋转点 xx 在 [i, m] 还是 [m + 1, j] 区间中。解决方案： 执行 j = j - 1 缩小判断范围。

### 153. 寻找旋转排序数组中的最小值

https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/

已知一个长度为 n 的数组，预先按照升序排列，经由 1 到 n 次 旋转 后，得到输入数组。例如，原数组 nums = [0,1,2,4,5,6,7] 在变化后可能得到：
若旋转 4 次，则可以得到 [4,5,6,7,0,1,2]
若旋转 7 次，则可以得到 [0,1,2,4,5,6,7]
注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。

给你一个元素值 互不相同 的数组 nums ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 最小元素 。

你必须设计一个时间复杂度为  O(log n) 的算法解决此问题。

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int l = 0, r = nums.size() - 1;
        while(l < r) {
            int mid = (l + r) / 2;
            if(nums[mid] > nums[r]) l = mid + 1;
            else if(nums[mid] <= nums[r]) r = mid;
        }
        return nums[r];
    }
};
```

与上一题类似，不过少一个条件判断。

### 33. 搜索旋转排序数组

https://leetcode.cn/problems/search-in-rotated-sorted-array/

整数数组 nums 按升序排列，数组中的值 互不相同 。

在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为  [4,5,6,7,0,1,2] 。

给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回  -1 。

你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        if(nums.empty()) return -1;
        int left = 0, right = nums.size() - 1;
        while(left < right) {
            int mid = left + right + 1 >> 1;
            if(nums[mid] >= nums[0]) left = mid;
            else right = mid - 1;
        }
        if(target >= nums[0]) left = 0;
        else left += 1, right = nums.size() - 1;
        while(left < right) {
            int mid = left + right >> 1;
            if(nums[mid] >= target) right = mid;
            else left = mid + 1;
        }
        if(nums[right] == target) return right;
        return -1;
    }
};
```

先二分找出分界点，再二分找答案。
