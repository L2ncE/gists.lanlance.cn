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

数组其实是有序的， 只不过负数平方之后可能成为最大数了。那么数组平方的最大值就在数组的两端，不是最左边就是最右边，不可能是中间。此时可以考虑双指针法了，i 指向起始位置，j 指向终止位置。定义一个新数组 result，和 nums 数组一样的大小，让k指向result数组终止位置。

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