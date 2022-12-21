### 231. 2 的幂

https://leetcode.cn/problems/power-of-two/

给你一个整数 n，请你判断该整数是否是 2 的幂次方。如果是，返回 true ；否则，返回 false 。

如果存在一个整数 x 使得 n == 2x ，则认为 n 是 2 的幂次方。

```cpp
class Solution {
public:
    bool isPowerOfTwo(int n) {
        return n > 0 && (n & -n) == n;
    }
};
```

通过 `n & -n` 获得最后一位 1，在二进制中只会有一位 1，所以那个 1 转化为十进制就为 n 。

### 191. 位 1 的个数

https://leetcode.cn/problems/number-of-1-bits/

编写一个函数，输入是一个无符号整数（以二进制串的形式），返回其二进制表达式中数字位数为 '1' 的个数（也被称为[汉明重量](https://baike.baidu.com/item/汉明重量)）。

```cpp
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int sum = 0;
        while(n > 0){
            n = n & (n - 1);
            sum++;
        }
        return sum;
    }
};
```

通过 `n = n & (n - 1)` 每次都能消灭一个 1，直到 n 为 0 时退出即可。

### 190. 颠倒二进制位

https://leetcode.cn/problems/reverse-bits/

颠倒给定的 32 位无符号整数的二进制位。

```cpp
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        uint32_t res = 0;
        for(int i = 0; i < 32; i++){
            res = (res << 1) + (n >> i & 1);
        }
        return res;
    }
};
```

每次将 res 左移一位，值得一提的是 `res << 1` 与 `res * 2` 相等。然后取出 n 的第 i 位即可。
