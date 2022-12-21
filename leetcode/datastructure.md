### 19. 删除链表的倒数第 N 个结点

https://leetcode.cn/problems/remove-nth-node-from-end-of-list/

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

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
        auto fake = new ListNode();
        fake->next = head;

        int cnt = 0;
        while(head){
            head = head->next;
            cnt++;
        }

        auto p = fake;
        for(int i = 0; i < cnt - n; i++){
            p = p->next;
        }
        p->next = p->next->next;
        return fake->next;
    }
};
```

难点在于找到倒数第 n 个结点，通过遍历 head 获得结点数，再从头遍历即可。值得一提的是需要用一个虚拟头结点进行操作，这样可以防止头结点被更改的问题。

### 21. 合并两个有序链表

https://leetcode.cn/problems/merge-two-sorted-lists/

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

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
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        auto dummy = new ListNode(-1), tail = dummy;
        while(l1 && l2){
            if(l1->val > l2->val){
                tail = tail->next = l2;
                l2 = l2->next;
            }else{
                tail = tail->next = l1;
                l1 = l1->next;
            }
        }
       if(l1) tail->next = l1;
       if(l2) tail->next = l2;
       return dummy->next;
    }
};
```

主要是设置一个虚拟头结点即可，然后从头到尾遍历两个链表就 OK 了。

### 206. 反转链表

https://leetcode.cn/problems/reverse-linked-list/

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

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
    ListNode* reverseList(ListNode* head) {
         if(!head) return NULL;
         auto a = head, b = a->next;
         while(b){
             auto tmp = b->next;
             b->next = a;
             a = b;
             b = tmp;
         }
         head->next = NULL;
         return a;
    }
};
```

画图就能搞懂，主要在于 `b->next = a` 相当于指反过来，然后 a 和 b 都往后移，直到 b 为 NULL。

### 203. 移除链表元素

https://leetcode.cn/problems/remove-linked-list-elements/

给你一个链表的头节点 `head` 和一个整数 `val` ，请你删除链表中所有满足 `Node.val == val` 的节点，并返回 **新的头节点** 。

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
    ListNode* removeElements(ListNode* head, int val) {
        auto dummy = new ListNode(-1);
        dummy->next = head;
        for(auto p = dummy; p; p = p->next){
            auto q = p->next;
            while(q && q->val == val) q = q->next;
            p->next = q;
        }
        return dummy->next;
    }
};
```

用一个多的指针，当遇到需要删除的数字时就开始判断，判断结束后让前面的指针指向后面的那个指针即可。

### 83. 删除排序链表中的重复元素

https://leetcode.cn/problems/remove-duplicates-from-sorted-list/

给定一个已排序的链表的头 `head` ， _删除所有重复的元素，使每个元素只出现一次_ 。返回 _已排序的链表_ 。

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
    ListNode* deleteDuplicates(ListNode* head) {
        if(!head) return head;
        auto dummy = head;
        while(dummy->next){
            if(dummy->val == dummy->next->val) dummy->next = dummy->next->next;
            else dummy = dummy->next;
        }
        return head;
    }
};
```

如果值相同就跳到下下个结点，相当于把值进行了删除。

### 20. 有效的括号

https://leetcode.cn/problems/valid-parentheses/

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
每个右括号都有一个对应的相同类型的左括号。

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> stk;
        for(auto c : s){
            if(c == '(' || c == '[' || c == '{') stk.push(c);
            else {
                if (stk.size() && abs(stk.top() - c) <= 2) stk.pop();
                else return false;
            }
        }
        return stk.empty();
    }
};
```

因为括号之间的 ASCII 码的差都在 2 以内，所以可以通过这个进行判断是否匹配。很简单的一个栈的问题。
