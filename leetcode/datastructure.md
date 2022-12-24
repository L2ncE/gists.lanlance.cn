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

### 707. 设计链表

https://leetcode.cn/problems/design-linked-list/

设计链表的实现。您可以选择使用单链表或双链表。单链表中的节点应该具有两个属性：val  和  next。val  是当前节点的值，next  是指向下一个节点的指针/引用。如果要使用双向链表，则还需要一个属性  prev  以指示链表中的上一个节点。假设链表中的所有节点都是 0-index 的。

在链表类中实现这些功能：

get(index)：获取链表中第  index  个节点的值。如果索引无效，则返回-1。
addAtHead(val)：在链表的第一个元素之前添加一个值为  val  的节点。插入后，新节点将成为链表的第一个节点。
addAtTail(val)：将值为  val 的节点追加到链表的最后一个元素。
addAtIndex(index,val)：在链表中的第  index  个节点之前添加值为  val  的节点。如果  index  等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果 index 小于 0，则在头部插入节点。
deleteAtIndex(index)：如果索引  index 有效，则删除链表中的第  index 个节点。

```cpp
class MyLinkedList {
public:
    // 定义链表节点结构体
    struct LinkedNode {
        int val;
        LinkedNode* next;
        LinkedNode(int val):val(val), next(nullptr){}
    };

    // 初始化链表
    MyLinkedList() {
        _dummyHead = new LinkedNode(0); // 这里定义的头结点 是一个虚拟头结点，而不是真正的链表头结点
        _size = 0;
    }

    // 获取到第index个节点数值，如果index是非法数值直接返回-1， 注意index是从0开始的，第0个节点就是头结点
    int get(int index) {
        if (index > (_size - 1) || index < 0) {
            return -1;
        }
        LinkedNode* cur = _dummyHead->next;
        while(index--){ // 如果--index 就会陷入死循环
            cur = cur->next;
        }
        return cur->val;
    }

    // 在链表最前面插入一个节点，插入完成后，新插入的节点为链表的新的头结点
    void addAtHead(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        newNode->next = _dummyHead->next;
        _dummyHead->next = newNode;
        _size++;
    }

    // 在链表最后面添加一个节点
    void addAtTail(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while(cur->next != nullptr){
            cur = cur->next;
        }
        cur->next = newNode;
        _size++;
    }

    // 在第index个节点之前插入一个新节点，例如index为0，那么新插入的节点为链表的新头节点。
    // 如果index 等于链表的长度，则说明是新插入的节点为链表的尾结点
    // 如果index大于链表的长度，则返回空
    // 如果index小于0，则在头部插入节点
    void addAtIndex(int index, int val) {

        if(index > _size) return;
        if(index < 0) index = 0;
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while(index--) {
            cur = cur->next;
        }
        newNode->next = cur->next;
        cur->next = newNode;
        _size++;
    }

    // 删除第index个节点，如果index 大于等于链表的长度，直接return，注意index是从0开始的
    void deleteAtIndex(int index) {
        if (index >= _size || index < 0) {
            return;
        }
        LinkedNode* cur = _dummyHead;
        while(index--) {
            cur = cur ->next;
        }
        LinkedNode* tmp = cur->next;
        cur->next = cur->next->next;
        delete tmp;
        _size--;
    }
};
```

有空多练，经典链表操作。

### 24. 两两交换链表中的节点

https://leetcode.cn/problems/swap-nodes-in-pairs/

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

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
    ListNode* swapPairs(ListNode* head) {
        auto dummy = new ListNode();
        dummy->next = head;
        auto p = dummy;
        while(p->next != NULL && p->next->next != NULL){
            auto n1 = p->next, n2 = p->next->next;
            n1->next = n2->next;
            n2->next = n1;
            p->next = n2;
            p = n1;
        }
        return dummy->next;
    }
};
```

主要是一个画图的问题，在图上理解清楚交换的逻辑就简单了。

### 160. 相交链表

https://leetcode.cn/problems/intersection-of-two-linked-lists/

给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 null 。

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
            a = a ? a->next : headB;
            b = b ? b->next : headA;
        }
        return a;
    }
};
```

做法很简单，但是想到不容易，把自己的跑完就去跑另外一个链表，最后相遇的时候距离相等刚好就是相交的节点。

### 142. 环形链表 II

https://leetcode.cn/problems/linked-list-cycle-ii/

给定一个链表的头节点  head ，返回链表开始入环的第一个节点。  如果链表无环，则返回  null。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

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
        if(!head || !head->next) return NULL;
        auto s = head, f = head->next;
        while(f->next && f->next->next){
            s = s->next, f = f->next->next;
            if(s == f){
                s = head, f = f->next;
                while(s != f){
                    s = s->next;
                    f = f->next;
                }
                return s;
            }
        }
        return NULL;
    }
};
```

画图才能解决，这道题比较绕，简单说就是到相遇点之后，让一个指针回 head 那个位置，然后两个指针以相同的速度重新跑，最后相遇点就是我们的答案。
