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

### 剑指 Offer 22. 链表中倒数第 k 个节点

https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/

输入一个链表，输出该链表中倒数第 k 个节点。为了符合大多数人的习惯，本题从 1 开始计数，即链表的尾节点是倒数第 1 个节点。

例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solutioxn {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        auto p1 = head, p2 = head;
        for(int i = 0; i < k; i++){
            p1 = p1->next;
        }
        while(p1){
            p1 = p1->next;
            p2 = p2->next;
        }
        return p2;
    }
};
```

首先，我们先让一个指针 p1 指向链表的头节点 head，然后走 k 步，现在的 p1，只要再走 n - k 步，就能走到链表末尾的空指针了，趁这个时候，再用一个指针 p2 指向链表头节点 head，
接下来就很显然了，让 p1 和 p2 同时向前走，p1 走到链表末尾的空指针时前进了 n - k 步，p2 也从 head 开始前进了 n - k 步，停留在第 n - k + 1 个节点上，即恰好停链表的倒数第 k 个节点上。

### 剑指 Offer 25. 合并两个排序的链表

https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

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
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(), p = dummy;
        while(l1 && l2){
            if(l1->val < l2->val){
                p = p->next = l1;
                l1 = l1->next;
            }else{
                p = p->next = l2;
                l2 = l2->next;
            }
        }
        if(l1) p->next = l1;
        if(l2) p->next = l2;
        return dummy->next;
    }
};
```

和主站 21 题一样，主要在于需要一个虚拟头结点和一个尾节点，清楚这两点这道题还是非常简单的。

### 剑指 Offer 52. 两个链表的第一个公共节点

https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/

输入两个链表，找出它们的第一个公共节点。

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
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto a = headA, b = headB;
        while(a != b){
            if(a) a = a->next;
            else a = headB;
            if(b) b = b->next;
            else b = headA;
        }
        return a;
    }
};
```

遍历一边到头后去另一边，最后到入口处两边走的距离肯定相同。

### 剑指 Offer II 021. 删除链表的倒数第 n 个结点

https://leetcode.cn/problems/SLwz0R/

给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

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
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        auto dummy = new ListNode();
        dummy->next = head;
        auto p = dummy, q = dummy;
        for(int i = 0; i < n + 1; i++) q = q->next;
        while(q){
            p = p->next;
            q = q->next;
        }
        p->next = p->next->next;
        return dummy->next;
    }
};
```

我们先走到链表的倒数 n + 1 这个点，再删除他的下一个结点即可。注意使用虚拟头结点。

### 剑指 Offer II 022. 链表中环的入口节点

https://leetcode.cn/problems/c32eOV/

给定一个链表，返回链表开始入环的第一个节点。 从链表的头节点开始沿着 next 指针进入环的第一个节点为环的入口节点。如果链表无环，则返回  null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

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
    ListNode *detectCycle(ListNode *head) {
        auto p = head, q = head;
        while(q && q->next){
            p = p->next;
            q = q->next->next;
            if(p == q) break;
        }
        if(!q || !q->next) return NULL;
        p = head;
        while(q != p){
            p = p->next;
            q = q->next;
        }
        return p;
    }
};
```

经典题，先用快慢指针直到两个点相遇，如果快指针指到空那么说明没有环。有环的话则让慢指针回到头结点，再以相同的速度一起往前走，相遇的位置则是入口节点。

### 剑指 Offer II 078. 合并排序链表

https://leetcode.cn/problems/vvXgSW/

给定一个链表数组，每个链表都已经按升序排列。

请将所有链表合并到一个升序链表中，返回合并后的链表。

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
    struct Cmp {
        bool operator()(ListNode *a, ListNode *b){
            return a->val > b->val;
        }
    };

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto dummy = new ListNode(), p = dummy;
        priority_queue<ListNode*, vector<ListNode*>, Cmp> heap;
        for(auto h : lists) if(h) heap.push(h);

        while(heap.size()){
            auto t = heap.top();
            heap.pop();
            p = p->next = t;
            if(t->next) heap.push(t->next);
        }
        return dummy->next;
    }
};
```

注意构造一个 Cmp 的结构体传入优先级队列。
