### 53. 最大子数组和

https://leetcode.cn/problems/maximum-subarray/

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。子数组** 是数组中的一个连续部分。

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int res = INT_MIN;
        for(int i = 0, last = 0; i < nums.size(); i++){
            last = nums[i] + max(0, last);
            res = max(res, last);
        }
        return res;
    }
};
```

我们自己想象一个 f[i]，f[i] 为所有以下标 i 结尾的数组之和。在 f[i] 中又可以分成有一个数的和有两个数的，有一个数的则就是它本身。我们再看有两个或更多数的，这些数组都含有 i ，所以这些数组可以转化成 `f[i - 1] + nums[i]`。我们将一个数和多个数的进行比较即得 `max = （f[i - 1] + nums[i], nums[i])`，将 nums[i] 提出来即得 `res = max(f[i - 1], 0) + nums[i]`。
