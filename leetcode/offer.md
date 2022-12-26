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
