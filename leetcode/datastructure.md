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

### 232. 用栈实现队列

https://leetcode.cn/problems/implement-queue-using-stacks/

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：

实现 MyQueue 类：

void push(int x) 将元素 x 推到队列的末尾
int pop() 从队列的开头移除并返回元素
int peek() 返回队列开头的元素
boolean empty() 如果队列为空，返回 true ；否则，返回 false
说明：

你 只能 使用标准的栈操作 —— 也就是只有  push to top, peek/pop from top, size, 和  is empty  操作是合法的。
你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。

```cpp
class MyQueue {
public:
    stack<int> a, b;
    MyQueue() {}

    void push(int x) {
        a.push(x);
    }

    int pop() {
        while(a.size() > 1) b.push(a.top()), a.pop();
        int t = a.top();
        a.pop();
        while(b.size()) a.push(b.top()), b.pop();
        return t;
    }

    int peek() {
        while(a.size() > 1) b.push(a.top()), a.pop();
        int t = a.top();
        while(b.size()) a.push(b.top()), b.pop();
        return t;
    }

    bool empty() {
        return a.empty();
    }
};

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue* obj = new MyQueue();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->peek();
 * bool param_4 = obj->empty();
 */
```

难点在于对 `pop()` 和 `peek` 的操作，用一个临时栈来存放最后一个值以外的元素，最后再将其打回即可。

### 225. 用队列实现栈

https://leetcode.cn/problems/implement-stack-using-queues/

请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（push、top、pop 和 empty）。

实现 MyStack 类：

void push(int x) 将元素 x 压入栈顶。
int pop() 移除并返回栈顶元素。
int top() 返回栈顶元素。
boolean empty() 如果栈是空的，返回 true ；否则，返回 false 。

注意：

你只能使用队列的基本操作 —— 也就是  push to back、peek/pop from front、size 和  is empty  这些操作。
你所使用的语言也许不支持队列。  你可以使用 list （列表）或者 deque（双端队列）来模拟一个队列  , 只要是标准的队列操作即可。

```cpp
class MyStack {
public:
    queue<int> a, b;
    MyStack() {

    }

    void push(int x) {
        a.push(x);
    }

    int pop() {
        while(a.size() > 1) b.push(a.front()), a.pop();
        int t = a.front();
        a.pop();
        while(b.size()) a.push(b.front()), b.pop();
        return t;
    }

    int top() {
        while(a.size() > 1) b.push(a.front()), a.pop();
        int t = a.front();
        a.pop();
        while(b.size()) a.push(b.front()), b.pop();
        a.push(t);
        return t;
    }

    bool empty() {
        return a.empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack* obj = new MyStack();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->top();
 * bool param_4 = obj->empty();
 */
```

和上题类似。有一点点逻辑的不同，看代码就懂了。

### 1047. 删除字符串中的所有相邻重复项

https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/

给出由小写字母组成的字符串  S，重复项删除操作会选择两个相邻且相同的字母，并删除它们。

在 S 上反复执行重复项删除操作，直到无法继续删除。

在完成所有重复项删除操作后返回最终的字符串。答案保证唯一。

```cpp
class Solution {
public:
    string removeDuplicates(string s) {
        string res;
        for(auto c : s){
            if(res.size() && res.back() == c) res.pop_back();
            else res += c;
        }
        return res;
    }
};
```

用栈实现即可。

### 144. 二叉树的前序遍历

https://leetcode.cn/problems/binary-tree-preorder-traversal/

给你二叉树的根节点 root ，返回它节点值的 前序 遍历。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void traversal(TreeNode* cur, vector<int> &res){
        if(!cur) return;
        vec.push_back(cur->val);
        traversal(cur->left, res);
        traversal(cur->right, res);
    }
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        traversal(root, res);
        return res;
    }
};
```

递归即可。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        while(root || stk.size()){
            while(root){
                res.push_back(root->val);
                stk.push(root);
                root = root->left;
            }
            root = stk.top()->right;
            stk.pop();
        }
        return res;
    }
};
```

迭代法

### 145. 二叉树的后序遍历

https://leetcode.cn/problems/binary-tree-postorder-traversal/

给你一棵二叉树的根节点 root ，返回其节点值的 后序遍历 。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void traversal(TreeNode *cur, vector<int>& res){
        if(!cur) return;
        traversal(cur->left, res);
        traversal(cur->right, res);
        res.push_back(cur->val);
    }
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        traversal(root, res);
        return res;
    }
};
```

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        while(root || stk.size()){
            while(root){
                res.push_back(root->val);
                stk.push(root);
                root = root->right;
            }
            root = stk.top()->left;
            stk.pop();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

### 94. 二叉树的中序遍历

https://leetcode.cn/problems/binary-tree-inorder-traversal/

给定一个二叉树的根节点 root ，返回 它的 中序 遍历 。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        while(root || stk.size()){
            while(root){
                stk.push(root);
                root = root->left;
            }
            root = stk.top();
            stk.pop();
            res.push_back(root->val);
            root = root->right;
        }
        return res;
    }
};
```

### 102. 二叉树的层序遍历

https://leetcode.cn/problems/binary-tree-level-order-traversal/

给你二叉树的根节点 root ，返回其节点值的 层序遍历 。 （即逐层地，从左到右访问所有节点）。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            vector<int> level;
            int len = q.size();
            while(len--){
                auto t = q.front();
                level.push_back(t->val);
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
            res.push_back(level);
        }
        return res;
    }
};
```

宽搜一下即可。

### 107. 二叉树的层序遍历 II

https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/

给你二叉树的根节点 root ，返回其节点值 自底向上的层序遍历 。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> res;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            vector<int> level;
            int len = q.size();
            while(len--){
                auto t = q.front();
                q.pop();
                level.push_back(t->val);
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
            res.push_back(level);
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

相较上一题，翻转一下即可。

### 199. 二叉树的右视图

https://leetcode.cn/problems/binary-tree-right-side-view/

给定一个二叉树的 根节点 root，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> res;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            int len = q.size();
            while(len--){
                auto t = q.front();
                q.pop();
                if(!len) res.push_back(t->val);
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
        }
        return res;
    }
};
```

用宽搜，然后找到每一排最后一个即可。

### 104. 二叉树的最大深度

https://leetcode.cn/problems/maximum-depth-of-binary-tree/

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明: 叶子节点是指没有子节点的节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        int res = 0;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            int len = q.size();
            res++;
            while(len--){
                auto t = q.front();
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
        }
        return res;
    }
};
```

一样的宽搜，每一层给结果加一即可。

### 111. 二叉树的最小深度

https://leetcode.cn/problems/minimum-depth-of-binary-tree/

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

说明：叶子节点是指没有子节点的节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int minDepth(TreeNode* root) {
        int res = 0;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            int len = q.size();
            res++;
            while(len--){
                auto t = q.front();
                q.pop();
                if(!t->left && !t->right) return res;
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
        }
        return res;
    }
};
```

在上一题的基础上，只要遇到叶子结点就返回答案。

### 226. 翻转二叉树

https://leetcode.cn/problems/invert-binary-tree/

给你一棵二叉树的根节点 root ，翻转这棵二叉树，并返回其根节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(!root) return NULL;
        swap(root->left, root->right);
        invertTree(root->left);
        invertTree(root->right);
        return root;
    }
};
```

递归翻转左右子树即可。

### 101. 对称二叉树

https://leetcode.cn/problems/symmetric-tree/

给你一个二叉树的根节点 root ， 检查它是否轴对称。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(!root) return true;
        return dfs(root->left, root->right);
    }

    bool dfs(TreeNode* left, TreeNode* right){
        if(!left && !right) return true;
        if(!left || !right || left->val != right->val) return false;
        return dfs(left->right, right->left) && dfs(left->left, right->right);
    }
};
```

爆搜左右两边，然后分别比较。

### 222. 完全二叉树的节点个数

给你一棵 完全二叉树 的根节点 root ，求出该树的节点个数。

完全二叉树 的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层，则该层包含 1~ 2h  个节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int countNodes(TreeNode* root) {
        if(!root) return NULL;
        int left = countNodes(root->left);
        int right = countNodes(root->right);
        return left + right + 1;
    }
};
```

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int countNodes(TreeNode* root) {
        int res = 0;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            int len = q.size();
            while(len--){
                res++;
                auto t = q.front();
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
        }
        return res;
    }
};
```

两种暴力做法。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int countNodes(TreeNode* root) {
        if(!root) return 0;
        auto left = root->left, right = root->right;
        int ld = 0, rd = 0;
        while(left){
            left = left->left;
            ld++;
        }
        while(right){
            right =  right->right;
            rd++;
        }
        if(ld == rd) return (2 << ld) - 1;
        return countNodes(root->left) + countNodes(root->right) + 1;
    }
};
```

更快的做法，利用了完全二叉树的性质。

### 110. 平衡二叉树

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1 。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool ans;
    bool isBalanced(TreeNode* root) {
        ans = true;
        dfs(root);
        return ans;
    }

    int dfs(TreeNode* root){
        if(!root) return 0;
        int lh = dfs(root->left);
        int rh = dfs(root->right);
        if(abs(lh - rh) > 1) ans = false;
        return max(lh, rh) + 1;
    }
};
```

首先定义一个全局的变量保存答案，然后递归左右字数找到最深的点，取差的绝对值即可。

### 257. 二叉树的所有路径

给你一个二叉树的根节点 root ，按 任意顺序 ，返回所有从根节点到叶子节点的路径。

叶子节点 是指没有子节点的节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<string> ans;
    vector<int> path;

    vector<string> binaryTreePaths(TreeNode* root) {
        if(root) dfs(root);
        return ans;
    }

    void dfs(TreeNode* root){
        path.push_back(root->val);
        if(!root->left && !root->right){
            string line = to_string(path[0]);
            for(int i = 1; i < path.size(); i++){
                line += "->" + to_string(path[i]);
            }
            ans.push_back(line);
        }else{
            if(root->left) dfs(root->left);
            if(root->right) dfs(root->right);
        }
        path.pop_back();
    }
};
```

爆搜即可。

### 404. 左叶子之和

给定二叉树的根节点 root ，返回所有左叶子之和。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int res = 0;

    int sumOfLeftLeaves(TreeNode* root) {
        dfs(root);
        return res;
    }

    void dfs(TreeNode* root){
        if(!root) return;
        if(root->left){
            if(!root->left->left && !root->left->right) res += root->left->val;
        }
        dfs(root->left);
        dfs(root->right);
    }
};
```

爆搜即可。

### 513. 找树左下角的值

给定一个二叉树的 根节点 root，请找出该二叉树的 最底层 最左边 节点的值。

假设二叉树中至少有一个节点。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        int res = 0;
        queue<TreeNode*> q;
        if(root) q.push(root);

        while(q.size()){
            int len = q.size();
            vector<int> line;
            while(len--){
                auto t = q.front();
                line.push_back(t->val);
                q.pop();
                if(t->left) q.push(t->left);
                if(t->right) q.push(t->right);
            }
            res = line[0];
        }
        return res;
    }
};
```

就爱层序遍历，别的咳嗽。

### 112. 路径总和

给你二叉树的根节点  root 和一个表示目标和的整数  targetSum 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和  targetSum 。如果存在，返回 true ；否则，返回 false 。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool hasPathSum(TreeNode* root, int sum) {
        if(!root) return false;
        sum -= root->val;
        if(!root->left && !root->right) return !sum;
        return root->left && hasPathSum(root->left, sum) || root->right && hasPathSum(root->right, sum);
    }
};
```

当是叶子结点且 sum = 0 时返回真即可。

### 654. 最大二叉树

给定一个不重复的整数数组  nums 。  最大二叉树   可以用下面的算法从  nums 递归地构建:

创建一个根节点，其值为  nums 中的最大值。
递归地在最大值   左边   的   子数组前缀上   构建左子树。
递归地在最大值 右边 的   子数组后缀上   构建右子树。
返回  nums 构建的 最大二叉树 。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        return dfs(nums, 0, nums.size() - 1);
    }

    TreeNode* dfs(vector<int>& nums, int l, int r){
        if(l > r) return NULL;
        int idx = l;
        for(int i = l + 1; i <= r; i++){
            if(nums[i] > nums[idx]) idx = i;
        }
        TreeNode *res = new TreeNode(nums[idx]);
        res->left = dfs(nums, l, idx - 1);
        res->right = dfs(nums, idx + 1, r);
        return res;
    }
};
```

直接按题目描述进行模拟即可。用递归函数 dfs(nums,l,r)dfs(nums,l,r) 表示对于 `[numsl,numsr][numsl,numsr]` 区间构建一棵树。具体如下：找到 `[numsl,numsr][numsl,numsr]` 区间内的最大值，记为 numsidnumsid，这个数字就是根节点的值。分别递归左右子树：dfs(nums,l,id−1)dfs(nums,l,id−1) 和 dfs(nums,id+1,r)dfs(nums,id+1,r)。
