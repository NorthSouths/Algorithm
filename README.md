# demo
DataStructure
LeetCode刷题总结，参考Labulabudong的算法小抄


算法整理笔记

# · 数据结构

## Ⅰ、链表

### 一、递归反转链表

[递归反转链表的一部分 :: labuladong的算法小抄 (gitee.io)](https://labuladong.gitee.io/algo/2/18/17/)

LeetCode92

#### 1. 递归反转整个链表

![image-20210910100703311](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910100703311.png)

```c++
ListNode reverse(ListNode head) {
    if (head.next == null) return head;
    ListNode last = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return last;
}
```



#### 2. 反转链表前N个节点

![image-20210910101750122](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910101750122.png)



```c++
ListNode successor = null; // 后驱节点

// 反转以 head 为起点的 n 个节点，返回新的头结点
ListNode reverseN(ListNode head, int n) {
    if (n == 1) { 
        // 记录第 n + 1 个节点
        successor = head.next;
        return head;
    }
    // 以 head.next 为起点，需要反转前 n - 1 个节点
    ListNode last = reverseN(head.next, n - 1);

    head.next.next = head;
    // 让反转之后的 head 节点和后面的节点连起来
    head.next = successor;
    return last;
}    

```



#### 3. 反转链表的一部分

```c++
ListNode reverseBetween(ListNode head, int m, int n) {
    // base case
    if (m == 1) {
        return reverseN(head, n);
    }
    // 前进到反转的起点触发 base case
    head.next = reverseBetween(head.next, m - 1, n - 1);
    return head;
}

```



#### 4. 穿针引线法

LeetCode官方题解一

算法步骤：

第 1 步：先将待反转的区域反转；
第 2 步：把 pre 的 next 指针指向反转以后的链表头节点，把反转以后的链表的尾节点的 next 指针指向 succ

![image-20210910103218156](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910103218156.png)



```c++
class Solution {
private:
    void reverseLinkedList(ListNode *head) {
        // 也可以使用递归反转一个链表
        ListNode *pre = nullptr;
        ListNode *cur = head;

        while (cur != nullptr) {
            ListNode *next = cur->next;
            cur->next = pre;
            pre = cur;
            cur = next;
        }
    }

public:
    ListNode *reverseBetween(ListNode *head, int left, int right) {
        // 因为头节点有可能发生变化，使用虚拟头节点可以避免复杂的分类讨论
        ListNode *dummyNode = new ListNode(-1);
        dummyNode->next = head;

        ListNode *pre = dummyNode;
        // 第 1 步：从虚拟头节点走 left - 1 步，来到 left 节点的前一个节点
        // 建议写在 for 循环里，语义清晰
        for (int i = 0; i < left - 1; i++) {
            pre = pre->next;
        }

        // 第 2 步：从 pre 再走 right - left + 1 步，来到 right 节点
        ListNode *rightNode = pre;
        for (int i = 0; i < right - left + 1; i++) {
            rightNode = rightNode->next;
        }

        // 第 3 步：切断出一个子链表（截取链表）
        ListNode *leftNode = pre->next;
        ListNode *curr = rightNode->next;

        // 注意：切断链接
        pre->next = nullptr;
        rightNode->next = nullptr;

        // 第 4 步：同第 206 题，反转链表的子区间
        reverseLinkedList(leftNode);

        // 第 5 步：接回到原来的链表中
        pre->next = rightNode;
        leftNode->next = curr;
        return dummyNode->next;
    }
};
```



#### 5. 一次遍历反转链表（头插法）

LeetCode官方题解二

![image-20210910105118215](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910105118215.png)

整体思想：在需要反转的区间里，每遍历到一个节点，让这个新节点来到反转部分的起始位置

使用三个指针变量 pre、curr、next 来记录反转的过程中需要的变量，它们的意义如下：

curr：指向待反转区域的第一个节点 left；
next：永远指向 curr 的下一个节点，循环过程中，curr 变化以后 next 会变化；
pre：永远指向待反转区域的第一个节点 left 的前一个节点，在循环过程中不变

（需要注意 穿针引线 操作的先后顺序：**一般是倒着链接，先连接后面的指针**）



```c++
class Solution {
public:
    ListNode *reverseBetween(ListNode *head, int left, int right) {
        // 设置 dummyNode 是这一类问题的一般做法
        ListNode *dummyNode = new ListNode(-1);
        dummyNode->next = head;
        ListNode *pre = dummyNode;
        for (int i = 0; i < left - 1; i++) {
            pre = pre->next;
        }
        ListNode *cur = pre->next;
        ListNode *next;
        for (int i = 0; i < right - left; i++) {
            next = cur->next;
            cur->next = next->next;
            next->next = pre->next;
            pre->next = next;
        }
        return dummyNode->next;
    }
};
```



### 二、迭代k个一组反转链表

[如何k个一组反转链表 :: labuladong的算法小抄 (gitee.io)](https://labuladong.gitee.io/algo/2/18/18/)

LeetCode 25

#### 1.迭代反转整个链表

「反转以 `a` 为头结点的链表」其实就是「反转 `a` 到 null 之间的结点」

```c++
// 反转以 a 为头结点的链表
ListNode reverse(ListNode a) {
    ListNode pre, cur, nxt;
    pre = null; cur = a; nxt = a;
    while (cur != null) {
        nxt = cur.next;
        // 逐个结点反转
        cur.next = pre;
        // 更新指针位置
        pre = cur;
        cur = nxt;
    }
    // 返回反转后的头结点
    return pre;
}
```



#### 2.反转a到b之间的节点

只要更改函数签名，并把上面的代码中 `null` 改成 `b` 即可。**注意：reverse` 函数是反转区间 `[a, b)**

```c++
/** 反转区间 [a, b) 的元素，注意是左闭右开 */
ListNode reverse(ListNode a, ListNode b) {
    ListNode pre, cur, nxt;
    pre = null; cur = a; nxt = a;
    // while 终止的条件改一下就行了
    while (cur != b) {
        nxt = cur.next;
        cur.next = pre;
        pre = cur;
        cur = nxt;
    }
    // 返回反转后的头结点
    return pre;
}
```



#### 3. k个一组反转链表

算法流程：

1）先反转以 `head` 开头的 `k` 个元素

2）将第 `k + 1` 个元素作为 `head` 递归调用 `reverseKGroup` 函数。

3）将上述两个过程的结果连接起来。

![image-20210910110311519](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910110311519.png)

```c++
ListNode reverseKGroup(ListNode head, int k) {
    if (head == null) return null;
    // 区间 [a, b) 包含 k 个待反转元素
    ListNode a, b;
    a = b = head;
    for (int i = 0; i < k; i++) {
        // 不足 k 个，不需要反转，base case
        if (b == null) return head;
        b = b.next;
    }
    // 反转前 k 个元素
    ListNode newHead = reverse(a, b);
    // 递归反转后续链表并连接起来
    a.next = reverseKGroup(b, k);
    return newHead;
}

```



#### 4. 迭代反转链表并返回新头尾

```c++
pair<ListNode*, ListNode*> myReverse(ListNode* head, ListNode* tail) {
    	// pre由原来的指向null变为指向当前链表尾部的下一个元素
        ListNode* prev = tail->next;
        ListNode* p = head;
        while (prev != tail) {
            ListNode* nex = p->next;
            p->next = prev;
            prev = p;
            p = nex;
        }
        return {tail, head};
    }
```



#### 5. 迭代k个一组反转链表

LeetCode官方题解

把链表节点按照 k 个一组分组，所以可以使用一个指针 head 依次指向每组的头节点。这个指针每次向前移动 k 步，直至链表结尾。对于每个分组，我们先判断它的长度是否大于等于 k。若是，我们就翻转这部分链表，否则不需要翻转。对于一个子链表，除了翻转其本身之外，还需要将子链表的头部与上一个子链表连接，以及子链表的尾部与下一个子链表连接



**创建虚拟头结点pre**

1）处理对于第一个子链表，它的头节点 `head` 前面是没有节点 `pre` 的

2）处理返回的头结点：节点 pre 一开始被连接到了头节点的前面，而无论之后链表有没有翻转，它的 next 指针都会指向正确的头节点。那么我们只要返回它的下一个节点就好了

```c++
class Solution {
public:
    // 翻转一个子链表，并且返回新的头与尾
    pair<ListNode*, ListNode*> myReverse(ListNode* head, ListNode* tail) {
        ListNode* prev = tail->next;
        ListNode* p = head;
        while (prev != tail) {
            ListNode* nex = p->next;
            p->next = prev;
            prev = p;
            p = nex;
        }
        return {tail, head};
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        // 创建虚拟头结点
        ListNode* hair = new ListNode(0);
        hair->next = head;
        ListNode* pre = hair;

        while (head) {
            ListNode* tail = pre;
            // 查看剩余部分长度是否大于等于 k
            for (int i = 0; i < k; ++i) {
                tail = tail->next;
                if (!tail) {
                    return hair->next;
                }
            }
            ListNode* nex = tail->next;
            // 这里是 C++17 的写法，也可以写成
            // pair<ListNode*, ListNode*> result = myReverse(head, tail);
            // head = result.first;
            // tail = result.second;
            tie(head, tail) = myReverse(head, tail);
            // 把子链表重新接回原链表
            pre->next = head;
            tail->next = nex;
            pre = tail;
            head = tail->next;
        }

        return hair->next;
    }
};
```



### 三、回文链表

[如何判断回文链表 :: labuladong的算法小抄 (gitee.io)](https://labuladong.gitee.io/algo/2/18/19/)

LeetCode 234

#### 1.寻找回文串

**寻找**回文串的核心思想是从中心向两端扩展

```cpp
string palindrome(string& s, int l, int r) {
    // 防止索引越界
    while (l >= 0 && r < s.size()
            && s[l] == s[r]) {
        // 向两边展开
        l--; r++;
    }
    // 返回以 s[l] 和 s[r] 为中心的最长回文串
    return s.substr(l + 1, r - l - 1);
}
```

因为回文串长度可能为奇数也可能是偶数，长度为奇数时只存在一个中心点，而长度为偶数时存在两个中心点，所以上面这个函数需要传入`l`和`r`。



#### 2. 判断回文串

**判断**一个字符串是不是回文串就简单很多，不需要考虑奇偶情况，只需要「双指针技巧」，从两端向中间逼近即可：**因为回文串是对称的，所以正着读和倒着读应该是一样的，这一特点是解决回文串问题的关键**

```cpp
bool isPalindrome(string s) {
    int left = 0, right = s.length - 1;
    while (left < right) {
        if (s[left] != s[right])
            return false;
        left++; right--;
    }
    return true;
}
```



#### 3. 判断回文单链表（快慢指针）

方法一：后序遍历实现倒序遍历链表

Labuladong解法

```c++
// 左侧指针
ListNode left;

boolean isPalindrome(ListNode head) {
    left = head;
    return traverse(head);
}

boolean traverse(ListNode right) {
    if (right == null) return true;
    boolean res = traverse(right.next);
    // 后序遍历代码
    res = res && (right.val == left.val);
    left = left.next;
    return res;
}
```



LeerCode官方解法二

currentNode 指针是先到尾节点，由于递归的特性再从后往前进行比较。frontPointer 是递归函数外的指针。若 currentNode.val != frontPointer.val 则返回 false。反之，frontPointer 向前移动并返回 true

```c++
class Solution {
    ListNode* frontPointer;
public:
    bool recursivelyCheck(ListNode* currentNode) {
        if (currentNode != nullptr) {
            if (!recursivelyCheck(currentNode->next)) {
                return false;
            }
            if (currentNode->val != frontPointer->val) {
                return false;
            }
            frontPointer = frontPointer->next;
        }
        return true;
    }

    bool isPalindrome(ListNode* head) {
        frontPointer = head;
        return recursivelyCheck(head);
    }
};
```



方法二：双指针技巧（得到链表中点）翻转链表后半段

labuladong版本

```c++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr) {
            return true;
        }
        
        // 快慢指针返回后半部分链表的头节点
        ListNode *slow,*fast;
        slow = fast = head;
        while(fast != nullptr && fast->next != nullptr){
            slow = slow->next;
            fast = fast->next->next;
        }
        if(fast != nullptr){
            slow = slow->next;
        }

        // 反转链表的后半部分
        ListNode *left = head;
        ListNode *right = reverse(slow);

        while(right != nullptr){
            if(left->val != right->val)
                return false;
            left = left->next;
            right = right->next;
        }
   
    }

    // 迭代反转链表
    ListNode* reverse(ListNode *head){
        ListNode *pre = nullptr,*cur = head,*next;
        while(cur != nullptr){
            next = cur->next;
            cur->next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
};
```



LeetCode官方题解三

```c++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr) {
            return true;
        }

        // 找到前半部分链表的尾节点并反转后半部分链表
        ListNode* firstHalfEnd = endOfFirstHalf(head);
        ListNode* secondHalfStart = reverseList(firstHalfEnd->next);

        // 判断是否回文
        ListNode* p1 = head;
        ListNode* p2 = secondHalfStart;
        bool result = true;
        while (result && p2 != nullptr) {
            if (p1->val != p2->val) {
                result = false;
            }
            p1 = p1->next;
            p2 = p2->next;
        }        

        // 还原链表并返回结果
        firstHalfEnd->next = reverseList(secondHalfStart);
        return result;
    }

    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr != nullptr) {
            ListNode* nextTemp = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }

    // 快慢指针返回链表的前半部分链表的尾节点
    ListNode* endOfFirstHalf(ListNode* head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while (fast->next != nullptr && fast->next->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
        }
        return slow;
    }
};
```



### 四、链表基本操作

#### 1. 判断链表的环开始节点

```java
ListNode detectCycle(ListNode head) {
    ListNode fast, slow;
    fast = slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow) break;
    }
    // 上面的代码类似 hasCycle 函数
    if (fast == null || fast.next == null) {
        // fast 遇到空指针说明没有环
        return null;
    }

    // 重新指向头结点
    slow = head;
    // 快慢指针同步前进，相交点就是环起点
    while (slow != fast) {
        fast = fast.next;
        slow = slow.next;
    }
    return slow;
}
```



### 五、链表技巧

#### 1. 虚拟头结点

#### 2. 优先级队列的使用-快速得到最小节点

```java
public ListNode mergeKLists(ListNode[] lists) {
        if(lists.length == 0)
            return null;
        ListNode dummy = new ListNode(-1);
        ListNode p = dummy;
        PriorityQueue<ListNode> pq = new PriorityQueue<>(lists.length,(a,b)->(a.val-b.val));
        for(ListNode head:lists){
            if(head != null)
                pq.add(head);
        }

        while(!pq.isEmpty()){
            ListNode node = pq.poll();
            p.next = node;
            if(node.next!=null)
                pq.add(node.next);
            p = p.next;
        }
        return dummy.next;
    }
```



## II、二叉树

二叉树框架：

```c++
void traverse(TreeNode root) {
    // 前序遍历
    traverse(root.left)
    // 中序遍历
    traverse(root.right)
    // 后序遍历
}
```



常见代码逻辑：

```c++
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么
    if (root.val < target) 
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```



递归框架

**1）这个函数是干嘛的？****

**2）这个函数参数中的变量是什么的是什么？**

**3）得到函数的递归结果，你应该干什么？**

### 一、二叉树最近公共祖先

[用 Git 来讲讲二叉树最近公共祖先 (qq.com)](https://mp.weixin.qq.com/s/9RKzBcr3I592spAsuMH45g)

LeetCode 236

#### 方法一：后序遍历

函数定义：

情况 1，如果`p`和`q`都在以`root`为根的树中，函数返回的即使`p`和`q`的最近公共祖先节点。

情况 2，那如果`p`和`q`都不在以`root`为根的树中怎么办呢？函数理所当然地返回`null`呗。

情况 3，那如果`p`和`q`只有一个存在于`root`为根的树中呢？函数就会返回那个节点。



base case：

如果`root`为空，肯定得返回`null`。

如果`root`本身就是`p`或者`q`，比如说`root`就是`p`节点吧，如果`q`存在于以`root`为根的树中，显然`root`就是最近公共祖先；即使`q`不存在于以`root`为根的树中，按照情况 3 的定义，也应该返回`root`节点



递归结果left,right：

情况 1，如果`p`和`q`都在以`root`为根的树中，那么`left`和`right`一定分别是`p`和`q`（从 base case 看出来的）。【后序遍历是从下往上，就好比从`p`和`q`出发往上走，第一次相交的节点就是这个`root`，最近公共祖先必为root】

情况 2，如果`p`和`q`都不在以`root`为根的树中，直接返回`null`。

情况 3，如果`p`和`q`只有一个存在于`root`为根的树中，函数返回该节点。

```c++
TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    // base case
    if (root == null) return null;
    if (root == p || root == q) return root;

    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    // 情况 1
    if (left != null && right != null) {
        return root;
    }
    // 情况 2
    if (left == null && right == null) {
        return null;
    }
    // 情况 3
    return left == null ? right : left;
}
```



#### 方法二：递归

*f(x)* 表示 x*x* 节点的子树中是否包含 p节点或 q节点，如果包含为 `true`，否则为 `false`。那么符合条件的最近公共祖先 x*x* 一定满足如下条件：

![image-20210910212659262](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210910212659262.png)

判断条件一：说明左子树和右子树均包含 p*p* 节点或 q*q* 节点，如果左子树包含的是 p*p* 节点，那么右子树只能包含 q*q* 节点

判断条件二：考虑了 x 恰好是 p节点或 q节点且它的左子树或右子树有一个包含了另一个节点的情况

```c++
class Solution {
public:
    TreeNode* ans;
    bool dfs(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == nullptr) return false;
        bool lson = dfs(root->left, p, q);
        bool rson = dfs(root->right, p, q);
        if ((lson && rson) || ((root->val == p->val || root->val == q->val) && (lson || rson))) {
            ans = root;
        } 
        return lson || rson || (root->val == p->val || root->val == q->val);
    }
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        dfs(root, p, q);
        return ans;
    }
}
```



#### 方法三： 存储父节点

用哈希表存储所有节点的父节点，然后我们就可以利用节点的父节点信息从 p 结点开始不断往上跳，并记录已经访问过的节点，再从 q 节点开始不断往上跳，如果碰到已经访问过的节点，那么这个节点就是我们要找的最近公共祖先

算法

1）从根节点开始遍历整棵二叉树，用哈希表记录每个节点的父节点指针。
2）从 p 节点开始不断往它的祖先移动，并用数据结构记录已经访问过的祖先节点。
3）同样，我们再从 q 节点开始不断往它的祖先移动，如果有祖先已经被访问过，即意味着这是 p 和 q 的深度最深的公共祖先

```c++
class Solution {
public:
    unordered_map<int, TreeNode*> fa;
    unordered_map<int, bool> vis;
    // 先序遍历
    void dfs(TreeNode* root){
        if (root->left != nullptr) {
            fa[root->left->val] = root;
            dfs(root->left);
        }
        if (root->right != nullptr) {
            fa[root->right->val] = root;
            dfs(root->right);
        }
    }
    
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        fa[root->val] = nullptr;
        dfs(root);
        while (p != nullptr) {
            vis[p->val] = true;
            p = fa[p->val];
        }
        while (q != nullptr) {
            if (vis[q->val]) return q;
            q = fa[q->val];
        }
        return nullptr;
    }
};
```

### 

### 二、BST第K小的元素

二叉搜索树BST思路：

**1）利用 BST 左小右大的特性提升算法效率**

**2）利用中序遍历的特性满足题目的要求（中序遍历升序，反过来降序）**



[手把手刷二叉搜索树（第一期） (qq.com)](https://mp.weixin.qq.com/s/ioyqagZLYrvdlZyOMDjrPw)

LeetCode 230

#### 方法一：利用BST中序遍历升序获得第K小的元素

```c++
	// res结果，rank当前元素的排名
	int res = 0,rank = 0;

    int kthSmallest(TreeNode* root, int k) {
        traverse(root,k);
        return res;
    }

    void traverse(TreeNode* root,int k){
        if(root == NULL)
            return;
        traverse(root->left,k);

        // 对于二叉搜索树，中序遍历的结果是升序的
        rank++;
        if(rank == k){	// 找到第 k 小的元素
            res = root->val;
            return;
        }
        traverse(root->right,k);
    }
```



#### 方法二：一个在二叉搜索树中通过排名计算对应元素的方法

想找到第`k`小的元素，或者说找到排名为`k`的元素，如果想达到对数级复杂度，关键也在于每个节点得知道他自己排第几。

比如说你让我查找排名为`k`的元素，当前节点知道自己排名第`m`，那么我可以比较`m`和`k`的大小：

1、如果`m == k`，显然就是找到了第`k`个元素，返回当前节点就行了。

2、如果`k < m`，那说明排名第`k`的元素在左子树，所以可以去左子树搜索第`k`个元素。

3、如果`k > m`，那说明排名第`k`的元素在右子树，所以可以去右子树搜索第`k - m - 1`个元素。

这样就可以将时间复杂度降到`O(logN)`了。



如何让每一个节点知道自己的排名？

需要在二叉树节点中维护额外信息。**每个节点需要记录，以自己为根的这棵二叉树有多少个节点**

```c++
class TreeNode {
    int val;
    // 以该节点为根的树的节点总数
    int size;
    TreeNode left;
    TreeNode right;
}
```



### 三、BST转化累计树

LeetCode538

#### 方法一： BST 的中序遍历

修改递归顺序（降序），降序遍历BST元素值

```c++
void traverse(TreeNode root) {
    if (root == null) return;
    // 先递归遍历右子树
    traverse(root.right);
    // 中序遍历代码位置
    print(root.val);
    // 后递归遍历左子树
    traverse(root.left);
}
```

这段代码可以从大到小降序打印 BST 节点的值，再维护一个外部累加变量`sum`，然后把`sum`赋值给 BST 中的每一个节点，就将 BST 转化成累加树

```c++
	// 维护累加变量
	int sum = 0;
    TreeNode* convertBST(TreeNode* root) {
        traverse(root);
        return root;
    }

    void traverse(TreeNode* root){
        if(root == NULL)
            return;
        traverse(root->right);

        sum += root->val;
        root->val = sum;

        traverse(root->left);
    }
```



#### 方法二：Morris遍历

[神级遍历——morris - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/101321696)

morris遍历可以将非递归遍历中的空间复杂度降为O(1)。从而实现时间复杂度为O(N)，而空间复杂度为O(1)的精妙算法。
morris遍历利用的是树的叶节点左右孩子为空（树的大量空闲指针），实现空间开销的极限缩减。



**Morris遍历实现原则：**

记作当前节点为cur。

1. 如果cur无左孩子，cur向右移动（cur=cur.right）

2. 如果cur有左孩子，找到cur左子树上最右的节点，记为mostright

3. 1. 如果mostright的right指针指向空，让其指向cur，cur向左移动（cur=cur.left）	//在遍历完左子树后回退到根节点
   2. 如果mostright的right指针指向cur，让其指向空，cur向右移动（cur=cur.right）  //还原叶节点

实现以上的原则，即实现了morris遍历。



**Morris遍历的实质**：

建立一种机制，对于没有左子树的节点只到达一次，对于有左子树的会到达两次



**代码实现：**

中序遍历

```java
public static void morrisIn(Node head) {
    if(head == null){
        return;
    }
    Node cur = head;
    Node mostRight = null;
    while (cur != null){
        mostRight = cur.left;
        if(mostRight != null){
            while (mostRight.right !=null && mostRight.right != cur){
                mostRight = mostRight.right;
            }
            if(mostRight.right == null){
                mostRight.right = cur;
                cur = cur.left;
                continue;
            }else {		//mostRight->right = cur
                mostRight.right = null;
            }
        }
        // 操作部分
        System.out.print(cur.value+" ");
        
        cur = cur.right;
    }
}
```



### 四、判断BST合法性

使用辅助函数，在参数中携带额外信息，将约束传递给子树的所有节点

```c++
boolean isValidBST(TreeNode root) {
    return isValidBST(root, null, null);
}

/* 限定以 root 为根的子树节点必须满足 max.val > root.val > min.val */
boolean isValidBST(TreeNode root, TreeNode min, TreeNode max) {
    // base case
    if (root == null) return true;
    // 若 root.val 不符合 max 和 min 的限制，说明不是合法 BST
    if (min != null && root.val <= min.val) return false;
    if (max != null && root.val >= max.val) return false;
    // 限定左子树的最大值是 root.val，右子树的最小值是 root.val
    return isValidBST(root.left, min, root) 
        && isValidBST(root.right, root, max);
}
```



### 五、BST增删查

#### 1. BST 搜索一个数

```c++
TreeNode* searchBST(TreeNode* root, int val) {
        if(root == NULL)
            return NULL;
        if(root->val == val)
            return root;
        else if(root->val > val)
            return searchBST(root->left,val);
        else
            return searchBST(root->right,val);
    }
```



#### 2. BST 插入一个数

```c++
TreeNode insertIntoBST(TreeNode root, int val) {
    // 找到空位置插入新节点
    if (root == null) return new TreeNode(val);
    // if (root.val == val)
    //     BST 中一般不会插入已存在元素
    if (root.val < val) 
        root.right = insertIntoBST(root.right, val);
    if (root.val > val) 
        root.left = insertIntoBST(root.left, val);
    return root;
}

```



#### 3. BST删除一个节点

分析：

1）`A` 恰好是末端节点，两个子节点都为空，直接删除即可

2）`A` 只有一个非空子节点，那么它要让这个孩子接替自己的位置

3）`A` 有两个子节点，为了不破坏 BST 的性质，`A` 必须找到左子树中最大的那个节点，或者右子树中最小的那个节点来接替自己。（以下实现采用用右子树的最小节点替代的方法）

```c++
TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    if (root.val == key) {
        // 这两个 if 把情况 1 和 2 都正确处理了
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;
        // 处理情况 3
        TreeNode minNode = getMin(root.right);
        root.val = minNode.val;
        root.right = deleteNode(root.right, minNode.val);
    } else if (root.val > key) {
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    }
    return root;
}

TreeNode getMin(TreeNode node) {
    // BST 最左边的就是最小的
    while (node.left != null) node = node.left;
    return node;
} 
```

注：这个删除操作并不完美，因为我们一般不会通过 `root.val = minNode.val` 修改节点内部的值来交换节点，而是通过一系列略微复杂的链表操作交换 `root` 和 `minNode` 两个节点。



### 六、计算所有合法BST

#### 1. 计算不同BST的个数

##### 方法一：穷举递归

当 `lo > hi` 闭区间 `[lo, hi]` 肯定是个空区间，也就对应着空节点 null，虽然是空节点，但也是一种情况，所以要返回 1 而不能返回 0

时间复杂度非常高，肯定存在重叠子问题，前文动态规划相关的问题多次讲过消除重叠子问题的方法，无非就是加一个备忘录：

```c++
// 备忘录
int[][] memo;

int numTrees(int n) {
    // 备忘录的值初始化为 0
    memo = new int[n + 1][n + 1];
    return count(1, n);
}

int count(int lo, int hi) {
    if (lo > hi) return 1;
    // 查备忘录
    if (memo[lo][hi] != 0) {
        return memo[lo][hi];
    }
    
    int res = 0;
    for (int mid = lo; mid <= hi; mid++) {
        int left = count(lo, mid - 1);
        int right = count(mid + 1, hi);
        res += left * right;
    }
    // 将结果存入备忘录
    memo[lo][hi] = res;
    
    return res;
}
```



##### 方法二：动态规划

遍历每个数字 ii，将该数字作为树根，将 1 \cdots (i-1)1⋯(i−1) 序列作为左子树，将 (i+1) \cdots n(i+1)⋯n 序列作为右子树。接着我们可以按照同样的方式递归构建左子树和右子树



算法

题目要求是计算不同二叉搜索树的个数。为此，我们可以定义两个函数：

G(n): 长度为n的序列能构成的不同二叉搜索树的个数。

F(i, n): 以 i 为根、序列长度为 n 的不同二叉搜索树个数 1≤i≤n)

- *G*(*n*)=*i*=1∑*n**F*(*i*,*n*)
- *F*(*i*,*n*)=*G*(*i*−1)⋅*G*(*n*−*i*)
- *F*(*i*,*n*)=*G*(*i*−1)⋅*G*(*n*−*i*)

```c++
class Solution {
public:
    int numTrees(int n) {
        vector<int> G(n + 1, 0);
        G[0] = 1;
        G[1] = 1;

        for (int i = 2; i <= n; ++i) {
            for (int j = 1; j <= i; ++j) {
                G[i] += G[j - 1] * G[i - j];
            }
        }
        return G[n];
    }
};
```



方法三：卡塔兰数

*C*0=1,*C**n*+1=*n*+22(2*n*+1)*C**n*

```c++
class Solution {
public:
    int numTrees(int n) {
        long long C = 1;
        for (int i = 0; i < n; ++i) {
            C = C * 2 * (2 * i + 1) / (i + 2);
        }
        return (int)C;
    }
};
```



#### 2. 构建所有合法BST

1）穷举 `root` 节点的所有可能。

2）递归构造出左右子树的所有合法 BST。

3）给 `root` 节点穷举所有左右子树的组合【即在计算的基础上，左右子树来排列组合出不同的结果】

```c++
/* 主函数 */
public List<TreeNode> generateTrees(int n) {
    if (n == 0) return new LinkedList<>();
    // 构造闭区间 [1, n] 组成的 BST 
    return build(1, n);
}

/* 构造闭区间 [lo, hi] 组成的 BST */
List<TreeNode> build(int lo, int hi) {
    List<TreeNode> res = new LinkedList<>();
    // base case
    if (lo > hi) {
        res.add(null);
        return res;
    }

    // 1、穷举 root 节点的所有可能。
    for (int i = lo; i <= hi; i++) {
        // 2、递归构造出左右子树的所有合法 BST。
        List<TreeNode> leftTree = build(lo, i - 1);
        List<TreeNode> rightTree = build(i + 1, hi);
        // 3、给 root 节点穷举所有左右子树的组合。
        for (TreeNode left : leftTree) {
            for (TreeNode right : rightTree) {
                // i 作为根节点 root 的值
                TreeNode root = new TreeNode(i);
                root.left = left;
                root.right = right;
                res.add(root);
            }
        }
    }
    
    return res;
}
```



### 七、计算完全二叉树的节点个数

[如何计算完全二叉树的节点数 :: labuladong的算法小抄 (gitee.io)](https://labuladong.gitee.io/algo/2/19/31/)

LeetCode 222

```c++
int countNodes(TreeNode* root) {
        TreeNode *l = root,*r = root;
        int left = 0,right = 0;
        while(l != NULL){
            left++;
            l = l->left;
        }
        while(r != NULL){
            right++;
            r = r->right;
        }

        if(left == right){
            // 满二叉树
            return pow(2,left)-1;
        }
        return 1+countNodes(root->left)+countNodes(root->right);
    }
```

（时间复杂度部分待看）



### 八、二叉树序列化与反序列化

[二叉树的题，就那几个框架，枯燥至极🤔 (qq.com)](https://mp.weixin.qq.com/s/DVX2A1ha4xSecEXLxW_UsA)

LeetCode297

反序列化：

#### 方法一：递归先序遍历

**先确定根节点 `root`，然后遵循前序遍历的规则，递归生成左右子树即可**

```java
	String SEP = ",";
    String NULL = "#";

    /* 主函数，将二叉树序列化为字符串 */
    String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serialize(root, sb);
        return sb.toString();
    }

    /* 辅助函数，将二叉树存入 StringBuilder */
    void serialize(TreeNode root, StringBuilder sb) {
        if (root == null) {
            sb.append(NULL).append(SEP);
            return;
        }

        sb.append(root.val).append(SEP);

        serialize(root.left, sb);
        serialize(root.right, sb);
    }

    /* 主函数，将字符串反序列化为二叉树结构 */
    TreeNode deserialize(String data) {
        // 将字符串转化成列表
        LinkedList<String> nodes = new LinkedList<>();
        for (String s : data.split(SEP)) {
            nodes.addLast(s);
        }
        return deserialize(nodes);
    }

    /* 辅助函数，通过 nodes 列表构造二叉树 */
    TreeNode deserialize(LinkedList<String> nodes) {
        if (nodes.isEmpty()) return null;

        /****** 前序遍历位置 ******/
        // 列表最左侧就是根节点
        String first = nodes.removeFirst();
        if (first.equals(NULL)) return null;
        TreeNode root = new TreeNode(Integer.parseInt(first));
        /***********************/

        root.left = deserialize(nodes);
        root.right = deserialize(nodes);

        return root;
    }
```



#### 方法二：bsf迭代层级二叉树

二叉树层级遍历框架：

```java
// 输入一棵二叉树的根节点，层序遍历这棵二叉树
void levelTraverse(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    int depth = 1;
    // 从上到下遍历二叉树的每一层
    while (!q.isEmpty()) {
        int sz = q.size();
        // 从左到右遍历每一层的每个节点
        for (int i = 0; i < sz; i++) {
            TreeNode cur = q.poll();
            printf("节点 %s 在第 %s 层", cur, depth);

            // 将下一层节点放入队列
            if (cur.left != null) {
                q.offer(cur.left);
            }
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
        depth++;
    }
}
```



序列化

```java
String SEP = ",";
String NULL = "#";

/* 将二叉树序列化为字符串 */
String serialize(TreeNode root) {
    if (root == null) return "";
    StringBuilder sb = new StringBuilder();
    // 初始化队列，将 root 加入队列
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    while (!q.isEmpty()) {
        TreeNode cur = q.poll();

        /* 层级遍历代码位置 */
        if (cur == null) {
            // 需要保留空指针信息
            sb.append(NULL).append(SEP);
            continue;
        }
        sb.append(cur.val).append(SEP);
        /*****************/

        q.offer(cur.left);
        q.offer(cur.right);
    }

    return sb.toString();
}
```



反序列化

```java
/* 将字符串反序列化为二叉树结构 */
TreeNode deserialize(String data) {
    if (data.isEmpty()) return null;
    String[] nodes = data.split(SEP);
    // 第一个元素就是 root 的值
    TreeNode root = new TreeNode(Integer.parseInt(nodes[0]));

    // 队列 q 记录父节点，将 root 加入队列
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    // 此处无需自增，在代码中left加一次到达right的位置，right自增一次到达下一次迭代的位置
    for (int i = 1; i < nodes.length; ) {
        // 队列中存的都是父节点
        TreeNode parent = q.poll();
        // 父节点对应的左侧子节点的值
        String left = nodes[i++];
        if (!left.equals(NULL)) {
            parent.left = new TreeNode(Integer.parseInt(left));
            q.offer(parent.left);
        } else {
            parent.left = null;
        }
        // 父节点对应的右侧子节点的值
        String right = nodes[i++];
        if (!right.equals(NULL)) {
            parent.right = new TreeNode(Integer.parseInt(right));
            q.offer(parent.right);
        } else {
            parent.right = null;
        }
    }
    return root;
}
```



### 九、扁平化嵌套列表迭代器

#### 方法一：n叉树遍历

```java
class NestedIterator implements Iterator<Integer> {

    // 声明迭代器
    private Iterator<Integer> it;

    public NestedIterator(List<NestedInteger> nestedList) {
        // 存放将 nestedList 打平的结果
        List<Integer> result = new LinkedList<>();
        for (NestedInteger node : nestedList) {
            // 以每个节点为根遍历
            traverse(node, result);
        }
        // 得到 result 列表的迭代器
        this.it = result.iterator();
    }    

    // 遍历以 root 为根的多叉树，将叶子节点的值加入 result 列表
    private void traverse(NestedInteger root, List<Integer> result) {
        if (root.isInteger()) {
            // 到达叶子节点
            result.add(root.getInteger());
            return;
        }
        // 遍历框架
        for (NestedInteger child : root.getList()) {
            traverse(child, result);
        }
    }
}
```



### 十、对称二叉树

#### 方法一：递归

```java
public boolean isSymmetric(TreeNode root) {
        return isMirror(root,root);
    }

    public boolean isMirror(TreeNode p,TreeNode q){
        if(p == null && q == null)
            return true;
        if(p == null || q == null)
            return false;

        return p.val == q.val && isMirror(p.right,q.left)&&isMirror(p.left,q.right);
    }
```



### 十一、二叉树层序遍历

#### 方法一：dfs+层数

```java
List<List<Integer>> levels = new ArrayList<List<Integer>>();
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root==null)
            return levels;
        dfs(root,0);
        return levels;
    }

    public void dfs(TreeNode root,int level){
        if(levels.size() == level)
            levels.add(new ArrayList<Integer>());
        levels.get(level).add(root.val);
        if(root.left != null)
            dfs(root.left,level+1);
        if(root.right != null)
            dfs(root.right,level+1);
    }
```



#### 方法二：bfs+层数

```java
public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ret = new ArrayList<List<Integer>>();
        if (root == null) {
            return ret;
        }

        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            List<Integer> level = new ArrayList<Integer>();
            // 一次迭代一层节点的个数，当前队列的个数即当前层的个数
            int currentLevelSize = queue.size();
            for (int i = 1; i <= currentLevelSize; ++i) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
            ret.add(level);
        }
        
        return ret;
    }
```



### 十二、转化为平衡二叉树

#### 方法一：贪心算法

「平衡」要求它是一棵空树或它的左右两个子树的高度差的绝对值不超过 1，这很容易让我们产生这样的想法——左右子树的大小越「平均」，这棵树会不会越平衡？于是一种贪心策略的雏形就形成了：我们可以通过中序遍历将原来的二叉搜索树转化为一个有序序列，然后对这个有序序列递归建树，对于区间 [L,R]：

取 mid = L+R/2，即中心位置作为当前节点的值；

如果 L<=mid-1，那么递归地将区间[L,mid-1]作为当前节点的左子树；

如果 mid+1≤R，那么递归地将区间 [mid+1,R] 作为当前节点的右子树。

```java
List<Integer> inOrder = new ArrayList<>();

    public TreeNode balanceBST(TreeNode root) {
        getInOrder(root);
        return build(0,inOrder.size()-1);
    }

    public void getInOrder(TreeNode root){
        if(root == null)
            return;
        getInOrder(root.left);
        inOrder.add(root.val);
        getInOrder(root.right);
    }

    public TreeNode build(int l,int r){
        int mid = (l+r)/2;
        TreeNode root = new TreeNode(inOrder.get(mid));
        if(l <= mid-1)
            root.left = build(l,mid-1);
        if(mid + 1 <=r)
            root.right = build(mid+1,r);
        return root;
    }
```



#### 方法二：旋转平衡二叉树

插入的过程和二叉搜索树插入过程一致，小于root，往左子树插入，大于root，往右子树插入。节点插入后，就是要根据节点的高度，动态对节点进行旋转。然后更新路径上每个节点的高度。

旋转的情况一共有4种情况：

新加入节点为 node.left 的左孩子， height(node.left) - height(node.right) > 1 。直接对node节点右旋。
新加入节点为 node.left 的右孩子， height(node.left) - height(node.right) > 1 。这时候要先对node.left左旋，调整为1的情况，再进行右旋。
新加入节点为 node.right 的右孩子， height(node.right) - height(node.left) > 1 。直接对node节点左旋。
新加入节点为 node.right 的左孩子， height(node.right) - height(node.left) > 1 。这时候要先对node.right右旋，调整为3的情况，再进行左旋。
要注意的是，节点旋转的时候，高度不是简单的+-1，而是要根据从当前节点旋转调整后的左右节点高度中获取较大值+1（本题从缓存中读取左右子树高度）。旋转高度调整完成后，返回node节点时候，也要重新计算一下新的高度，其高度为左右子树最大值+1。



```java
class Solution {
    public TreeNode balanceBST(TreeNode root) {
        if (root == null){
            return null;
        }
        // node节点的高度缓存
        Map<TreeNode,Integer> nodeHeight = new HashMap<>();
        TreeNode newRoot = null;
        Deque<TreeNode> stack = new LinkedList<>();
        TreeNode node = root;
        // 先序遍历插入（其实用哪个遍历都行）
        while(node != null || !stack.isEmpty()){
            if (node != null){
                // 新树插入
                newRoot = insert(newRoot,node.val,nodeHeight);
                stack.push(node);
                node = node.left;
            }else {
                node = stack.pop();
                node = node.right;
            }
        }
        return newRoot;
    }

    /**
     * 新节点插入
     * @param root root
     * @param val 新加入的值
     * @param nodeHeight 节点高度缓存
     * @return 新的root节点
     */
    private TreeNode insert(TreeNode root,int val,Map<TreeNode,Integer> nodeHeight){
        if (root == null){
            root = new TreeNode(val);
            nodeHeight.put(root,1);// 新节点的高度
            return root;
        }
        TreeNode node = root;
        int cmp = val - node.val;
        if (cmp < 0){
            // 左子树插入
            node.left = insert(root.left,val,nodeHeight);
            // 如果左右子树高度差超过1，进行旋转调整
            if (nodeHeight.getOrDefault(node.left,0) - nodeHeight.getOrDefault(node.right,0) > 1){
                if (val > node.left.val){
                    // 插入在左孩子右边，左孩子先左旋
                    node.left = rotateLeft(node.left,nodeHeight);
                }
                // 节点右旋
                node = rotateRight(node,nodeHeight);
            }
        }else if (cmp > 0){
            // 右子树插入
            node.right = insert(root.right,val,nodeHeight);
            // 如果左右子树高度差超过1，进行旋转调整
            if (nodeHeight.getOrDefault(node.right,0) - nodeHeight.getOrDefault(node.left,0) > 1){
                if (val < node.right.val){
                    // 插入在右孩子左边，右孩子先右旋
                    node.right = rotateRight(node.right,nodeHeight);
                }
                // 节点左旋
                node = rotateLeft(node,nodeHeight);
            }
        }else {
            // 一样的节点，啥都没发生
            return node;
        }
        // 获取当前节点新高度
        int height =  getCurNodeNewHeight(node,nodeHeight);
        // 更新当前节点高度
        nodeHeight.put(node,height);
        return node;
    }

    /**
     * node节点左旋
     * @param node node
     * @param nodeHeight node高度缓存
     * @return 旋转后的当前节点
     */
    private TreeNode rotateLeft(TreeNode node,Map<TreeNode,Integer> nodeHeight){
        // ---指针调整
        TreeNode right = node.right;
        node.right = right.left;
        right.left = node;
        // ---高度更新
        // 先更新node节点的高度，这个时候node是right节点的左孩子
        int newNodeHeight = getCurNodeNewHeight(node,nodeHeight);
        // 更新node节点高度
        nodeHeight.put(node,newNodeHeight);
        // newNodeHeight是现在right节点左子树高度，原理一样，取现在right左右子树最大高度+1
        int newRightHeight = Math.max(newNodeHeight,nodeHeight.getOrDefault(right.right,0)) + 1;
        // 更新原right节点高度
        nodeHeight.put(right,newRightHeight);
        return right;
    }

    /**
     * node节点右旋
     * @param node node
     * @param nodeHeight node高度缓存
     * @return 旋转后的当前节点
     */
    private TreeNode rotateRight(TreeNode node,Map<TreeNode,Integer> nodeHeight){
        // ---指针调整
        TreeNode left = node.left;
        node.left = left.right;
        left.right = node;
        // ---高度更新
        // 先更新node节点的高度，这个时候node是right节点的左孩子
        int newNodeHeight = getCurNodeNewHeight(node,nodeHeight);
        // 更新node节点高度
        nodeHeight.put(node,newNodeHeight);
        // newNodeHeight是现在left节点右子树高度，原理一样，取现在right左右子树最大高度+1
        int newLeftHeight = Math.max(newNodeHeight,nodeHeight.getOrDefault(left.left,0)) + 1;
        // 更新原left节点高度
        nodeHeight.put(left,newLeftHeight);
        return left;
    }

    /**
     * 获取当前节点的新高度
     * @param node node
     * @param nodeHeight node高度缓存
     * @return 当前node的新高度
     */
    private int getCurNodeNewHeight(TreeNode node,Map<TreeNode,Integer> nodeHeight){
        // node节点的高度，为现在node左右子树最大高度+1
        return Math.max(nodeHeight.getOrDefault(node.left,0),nodeHeight.getOrDefault(node.right,0)) + 1;
    }
}
```



### 十三、先序+中序还原二叉树

#### 方法一：递归

对于任意一颗树而言，前序遍历的形式总是


[ 根节点, [左子树的前序遍历结果], [右子树的前序遍历结果] ]
即根节点总是前序遍历中的第一个节点。而中序遍历的形式总是


[ [左子树的中序遍历结果], 根节点, [右子树的中序遍历结果] ]
只要我们在中序遍历中定位到根节点，那么我们就可以分别知道左子树和右子树中的节点数目。



```java
class Solution {
    private Map<Integer, Integer> indexMap;

    public TreeNode myBuildTree(int[] preorder, int[] inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {
        if (preorder_left > preorder_right) {
            return null;
        }

        // 前序遍历中的第一个节点就是根节点
        int preorder_root = preorder_left;
        // 在中序遍历中定位根节点
        int inorder_root = indexMap.get(preorder[preorder_root]);
        
        // 先把根节点建立出来
        TreeNode root = new TreeNode(preorder[preorder_root]);
        // 得到左子树中的节点数目
        int size_left_subtree = inorder_root - inorder_left;
        // 递归地构造左子树，并连接到根节点
        // 先序遍历中「从 左边界+1 开始的 size_left_subtree」个元素就对应了中序遍历中「从 左边界 开始到 根节点定位-1」的元素
        root.left = myBuildTree(preorder, inorder, preorder_left + 1, preorder_left + size_left_subtree, inorder_left, inorder_root - 1);
        // 递归地构造右子树，并连接到根节点
        // 先序遍历中「从 左边界+1+左子树节点数目 开始到 右边界」的元素就对应了中序遍历中「从 根节点定位+1 到 右边界」的元素
        root.right = myBuildTree(preorder, inorder, preorder_left + size_left_subtree + 1, preorder_right, inorder_root + 1, inorder_right);
        return root;
    }

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int n = preorder.length;
        // 构造哈希映射，帮助我们快速定位根节点
        indexMap = new HashMap<Integer, Integer>();
        for (int i = 0; i < n; i++) {
            indexMap.put(inorder[i], i);
        }
        return myBuildTree(preorder, inorder, 0, n - 1, 0, n - 1);
    }
}
```



### 十四、二叉搜索树寻找最小公共祖先

#### 方法一：两次遍历

思路：根据BST的特性获得从根节点到目标节点的路径，在获得p,q的路径之后，获取路径上的相同的index最大的节点

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        List<TreeNode> pPath = getPath(root,p);
        List<TreeNode> qPath = getPath(root,q);
        TreeNode ancestor = null;
        for(int i = 0;i < pPath.size() && i < qPath.size();i++){
            if(pPath.get(i) == qPath.get(i))
                ancestor = pPath.get(i);
        } 
        return ancestor;
    }
    
    public List<TreeNode> getPath(TreeNode root,TreeNode target){
        if(root==null)
            return null;
        List<TreeNode> res = new ArrayList<>();
        while(root != target){
            res.add(root);
            if(target.val < root.val)
                root = root.left;
            else
                root = root.right;
        }
        res.add(root);
        return res;
    }
```



#### 方法二：一次迭代

整体的遍历过程与方法一中的类似：

我们从根节点开始遍历；

如果当前节点的值大于 p 和 q 的值，说明 p 和 q 应该在当前节点的左子树，因此将当前节点移动到它的左子节点；

如果当前节点的值小于 p 和 q 的值，说明 p 和 q 应该在当前节点的右子树，因此将当前节点移动到它的右子节点；

如果当前节点的值不满足上述两条要求，那么说明当前节点就是「分岔点」。此时，p 和 q 要么在当前节点的不同的子树中，要么其中一个就是当前节点。



```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode ancestor = root;
        while (true) {
            if (p.val < ancestor.val && q.val < ancestor.val) {
                ancestor = ancestor.left;
            } else if (p.val > ancestor.val && q.val > ancestor.val) {
                ancestor = ancestor.right;
            } else {
                break;
            }
        }
        return ancestor;
    }
```



### 十五、二维数组的查找

![image-20210925194635139](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20210925194635139.png)

从二维数组的右上角开始查找。如果当前元素等于目标值，则返回 true。如果当前元素大于目标值，则移到左边一列。如果当前元素小于目标值，则移到下边一行。

可以证明这种方法不会错过目标值。如果当前元素大于目标值，说明当前元素的下边的所有元素都一定大于目标值，因此往下查找不可能找到目标值，往左查找可能找到目标值。如果当前元素小于目标值，说明当前元素的左边的所有元素都一定小于目标值，因此往左查找不可能找到目标值，往下查找可能找到目标值。

- 若数组为空，返回 false
- 初始化行下标为 0，列下标为二维数组的列数减 1
  - 重复下列步骤，直到行下标或列下标超出边界
  - 获得当前下标位置的元素 num
  - 如果 num 和 target 相等，返回 true
  - 如果 num 大于 target，列下标减 1
  - 如果 num 小于 target，行下标加 1

- 环体执行完毕仍未找到元素等于 target ，说明不存在这样的元素，返回 false`

```java
public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if(matrix == null || matrix.length==0 || matrix[0].length==0)
            return false;
        int rows = matrix.length,cols = matrix[0].length;
        int row = 0,col = cols-1;
        while(row < rows && col >=0 ){
            int num = matrix[row][col];
            if(num == target)
                return true;
            else if(num > target)
                col--;
            else
                row++;
        }
        return false;
    }
```



### 十六、二叉树中和为某一值的路径

#### 方法一：深度搜索

枚举每一条从根节点到叶子节点的路径。当我们遍历到叶子节点，且此时路径和恰为目标和时，我们就找到了一条满足条件的路径

```java
class Solution {
    List<List<Integer>> ret = new LinkedList<List<Integer>>();
    Deque<Integer> path = new LinkedList<Integer>();

    public List<List<Integer>> pathSum(TreeNode root, int target) {
        dfs(root, target);
        return ret;
    }

    public void dfs(TreeNode root, int target) {
        if (root == null) {
            return;
        }
        path.offerLast(root.val);
        target -= root.val;
        if (root.left == null && root.right == null && target == 0) {
            ret.add(new LinkedList<Integer>(path));
        }
        dfs(root.left, target);
        dfs(root.right, target);
        path.pollLast();
    }
}
```



### 十七、BST的后续遍历序列

#### 方法一：递归分治

根据二叉搜索树的定义，可以通过递归，判断所有子树的 正确性 （即其后序遍历是否满足二叉搜索树的定义） ，若所有子树都正确，则此序列为二叉搜索树的后序遍历。

递归解析：
终止条件： 当 i \geq ji≥j ，说明此子树节点数量 \leq 1≤1 ，无需判别正确性，因此直接返回 truetrue ；
递推工作：
划分左右子树： 遍历后序遍历的 [i, j][i,j] 区间元素，寻找 第一个大于根节点 的节点，索引记为 mm 。此时，可划分出左子树区间 [i,m-1][i,m−1] 、右子树区间 [m, j - 1][m,j−1] 、根节点索引 jj 。
判断是否为二叉搜索树：
左子树区间 [i, m - 1][i,m−1] 内的所有节点都应 << postorder[j]postorder[j] 。而第 1.划分左右子树 步骤已经保证左子树区间的正确性，因此只需要判断右子树区间即可。
右子树区间 [m, j-1][m,j−1] 内的所有节点都应 >> postorder[j]postorder[j] 。实现方式为遍历，当遇到 \leq postorder[j]≤postorder[j] 的节点则跳出；则可通过 p = jp=j 判断是否为二叉搜索树。
返回值： 所有子树都需正确才可判定正确，因此使用 与逻辑符 \&\&&& 连接。
p = jp=j ： 判断 此树 是否正确。
recur(i, m - 1)recur(i,m−1) ： 判断 此树的左子树 是否正确。
recur(m, j - 1)recur(m,j−1) ： 判断 此树的右子树 是否正确。

```java
public boolean verifyPostorder(int[] postorder) {
        return recur(postorder, 0 , postorder.length-1);
    }

    public boolean recur(int[] postorder,int i,int j){
        if(i >= j)
            return true;
        int p = i;
        while(postorder[p] < postorder[j])
            p++;
        int m = p;
        // 判断有没有在右子树中是不是比根节点都大
        while(postorder[p] > postorder[j])
            p++;
        return p==j && recur(postorder,i,m-1) && recur(postorder,m,j-1);
    }
```





### 十八、BST与双向链表

#### 方法一：中序遍历

```java
	Node pre,head;
    public Node treeToDoublyList(Node root) {
        if(root==null)
            return null;
        dfs(root);
        // pre在遍历完之后成为了最后一个节点
        head.left = pre;
        pre.right = head;
        return head;
    }

    public void dfs(Node cur){
        if(cur==null)
            return;
        dfs(cur.left);
        if(pre!=null)
            pre.right = cur;
        else
            head = cur;
        cur.left = pre;
        pre = cur;
        dfs(cur.right);
    }
```



## Ⅲ、图

#### 图遍历框架

注意：取出和放入节点的操作是在for循环之外的，这样才能成功打印根节点

```java
Graph graph;
boolean[] visited;

/* 图遍历框架 */
void traverse(Graph graph, int s) {
    if (visited[s]) return;
    // 经过节点 s
    visited[s] = true;
    for (TreeNode neighbor : graph.neighbors(s))
        traverse(neighbor);
    // 离开节点 s
    visited[s] = false;   
}
```



### 一、判断图是否有环

#### 方法一：建图+dfs遍历图

建图函数

一般建立邻接表来表示图

```java
List<List<Integer>> buildGraph(int numCourses, int[][] prerequisites){
        List<List<Integer>> graph = new ArrayList<List<Integer>>();
        for(int i = 0;i < numCourses;i++)
            graph.add(new ArrayList<Integer>());
        for(int[] edge : prerequisites)
            graph.get(edge[0]).add(edge[1]);
        return graph;
    }
```



图的遍历

在进入节点 `s` 的时候将 `onPath[s]` 标记为 true，离开时标记回 false，如果发现 `onPath[s]` 已经被标记，说明出现了环

```java
	boolean[] onpath;		// 记录当前遍历的路径
    boolean[] isVisited;	// 记录已经遍历过的点
    boolean hasCycle;

    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = buildGraph(numCourses,prerequisites);
        isVisited = new boolean[numCourses];
        onpath = new boolean[numCourses];

        for(int i = 0;i <numCourses;i++)
            traverse(graph,i);
        return !hasCycle;
    }

    public void traverse(List<List<Integer>> graph,int s){
        if(onpath[s])
            hasCycle = true;
        if(isVisited[s] || hasCycle)
            return;
        isVisited[s] = true;
        // 开始遍历节点 s
        onpath[s] = true;
        for(int t:graph.get(s)){
            traverse(graph,t);
        }
         // 节点 s 遍历完成
        onpath[s] = false;
    }
```



#### 方法二：BFS

考虑拓扑排序中最前面的节点，该节点一定不会有任何入边，将一个节点加入答案中后，我们就可以移除它的所有出边。如果某个相邻节点变成了「没有任何入边的节点」，那么就代表着这门课可以开始学习了

算法

我们使用一个队列来进行广度优先搜索。初始时，所有入度为 0 的节点都被放入队列中，它们就是可以作为拓扑排序最前面的节点，并且它们之间的相对顺序是无关紧要的。

在广度优先搜索的每一步中，我们取出队首的节点 u：将 u 放入答案中；

我们移除 u的所有出边，也就是将 u 的所有相邻节点的入度减少 1。如果某个相邻节点 v 的入度变为 0，那么我们就将 v 放入队列中。

在广度优先搜索的过程结束后。如果答案中包含了这 n个节点，那么我们就找到了一种拓扑排序，否则说明图中存在环，也就不存在拓扑排序了

```java
class Solution {
    List<List<Integer>> edges;
    int[] indeg;	//入度

    public boolean canFinish(int numCourses, int[][] prerequisites) {
        // 建图并计算初始入度
        edges = new ArrayList<List<Integer>>();
        for (int i = 0; i < numCourses; ++i) {
            edges.add(new ArrayList<Integer>());
        }
        indeg = new int[numCourses];
        for (int[] info : prerequisites) {
            edges.get(info[1]).add(info[0]);
            ++indeg[info[0]];
        }
		// 将入度为0的点放入队列中
        Queue<Integer> queue = new LinkedList<Integer>();
        for (int i = 0; i < numCourses; ++i) {
            if (indeg[i] == 0) {
                queue.offer(i);
            }
        }
        // 已访问节点数
        int visited = 0;
        while (!queue.isEmpty()) {
            ++visited;	// 访问节点u而增加的已访问次数
            // 去掉从入度为0的点指出的边
            int u = queue.poll();
            for (int v: edges.get(u)) {
                --indeg[v];
                if (indeg[v] == 0) {
                    queue.offer(v);
                }
            }
        }
        return visited == numCourses;
    }
}
```



### 二、拓扑排序

#### 方法一：dfs+栈

**后序遍历的反转结果就是拓扑排序**

```java
class Solution {
    // 存储有向图
    List<List<Integer>> edges;
    // 标记每个节点的状态：0=未搜索，1=搜索中，2=已完成
    int[] visited;
    // 用数组来模拟栈，下标 n-1 为栈底，0 为栈顶
    int[] result;
    // 判断有向图中是否有环
    boolean valid = true;
    // 栈下标
    int index;

    public int[] findOrder(int numCourses, int[][] prerequisites) {
        edges = new ArrayList<List<Integer>>();
        for (int i = 0; i < numCourses; ++i) {
            edges.add(new ArrayList<Integer>());
        }
        visited = new int[numCourses];
        result = new int[numCourses];
        // 为了反转后续遍历的结果
        index = numCourses - 1;
        for (int[] info : prerequisites) {
            edges.get(info[1]).add(info[0]);
        }
        // 每次挑选一个「未搜索」的节点，开始进行深度优先搜索
        for (int i = 0; i < numCourses && valid; ++i) {
            if (visited[i] == 0) {
                dfs(i);
            }
        }
        if (!valid) {
            return new int[0];
        }
        // 如果没有环，那么就有拓扑排序
        return result;
    }

    public void dfs(int u) {
        // 将节点标记为「搜索中」
        visited[u] = 1;
        // 搜索其相邻节点，只要发现有环，立刻停止搜索
        for (int v: edges.get(u)) {
            // 如果「未搜索」那么搜索相邻节点
            if (visited[v] == 0) {
                dfs(v);
                if (!valid) {
                    return;
                }
            }
            // 如果「搜索中」说明找到了环
            else if (visited[v] == 1) {
                valid = false;
                return;
            }
        }
        // 将节点标记为「已完成」
        visited[u] = 2;
        // 将节点入栈
        result[index--] = u;
    }
}
```



#### 方法二：bfs

与上题类似，只是需要将入度为0的点加入返回的

```java
List<List<Integer>> graph;
    int[] indegree;
    int[] res;
    int index;

    public int[] findOrder(int numCourses, int[][] prerequisites) {
        graph = new ArrayList<List<Integer>>();
        indegree = new int[numCourses];
        res = new int[numCourses];
        index = 0;
        for(int i = 0;i < numCourses;i++)
            graph.add(new ArrayList<Integer>());
        for(int[] pre : prerequisites){
            graph.get(pre[1]).add(pre[0]);
            ++indegree[pre[0]];
        }

        Queue<Integer> queue = new LinkedList<Integer>();
        for(int i = 0;i < numCourses;i++){
            if(indegree[i]==0)
                queue.offer(i);
        }

        while(!queue.isEmpty()){
            int u = queue.poll();
            res[index++] = u;
            for(int v:graph.get(u)){
                indegree[v]--;
                if(indegree[v]==0)
                    queue.offer(v);
            }
        }
        if(index!=numCourses)
            return new int[0];
        return res;
    }
```



### 三、DIJKSTRA 算法

#### 方法一：枚举

```java
public int networkDelayTime(int[][] times, int n, int k) {
    final int INF = Integer.MAX_VALUE / 2;
    // 建图
    int[][] graph = new int[n][n];
    for (int i = 0; i < n; ++i) {
        Arrays.fill(graph[i], INF);
    }
    for (int[] t : times) {
        graph[t[0] - 1][t[1] - 1] = t[2];
    }

    int[] dist = new int[n];
    Arrays.fill(dist, INF);
    dist[k - 1] = 0;    // 数组从0开始
    boolean[] visited = new boolean[n];
    // 遍历图中的每个点获取各自的最短路径
    for (int i = 0; i < n; ++i) {
        int x = -1;
        for (int y = 0; y < n; ++y) {
            // 寻找还未找到最短路径并且到当前最后一个加入最短路径的最近的点x
            if (!visited[y] && (x == -1 || dist[y] < dist[x])) {
                x = y;
            }
        }
        visited[x] = true;
        for (int y = 0; y < n; ++y) {   //更新x的最短距离
            dist[y] = Math.min(dist[y], dist[x] + graph[x][y]);
        }
    }

    int ans = Arrays.stream(dist).max().getAsInt();
    return ans == INF ? -1 : ans;
    }
```



#### 方法二：优先级队列优化

```java
class Solution {
    public double maxProbability(int n, int[][] edges, double[] succProb, int start, int end) {
        List<List<Pair>> graph = new ArrayList<List<Pair>>();
        for(int i = 0;i < n;i++)
            graph.add(new ArrayList<Pair>());
        for(int i = 0;i < edges.length;i++){    //（起点到该点的概率，该点）
            int[] e = edges[i];
            graph.get(e[0]).add(new Pair(succProb[i],e[1]));
            graph.get(e[1]).add(new Pair(succProb[i],e[0]));
        }

        PriorityQueue<Pair> queue = new PriorityQueue<Pair>();
        double[] prob = new double[n];
        queue.offer(new Pair(1,start));
        prob[start] = 1;
        while(!queue.isEmpty()){
            // 最小的点
            Pair pair = queue.poll();
            double pr = pair.probability;
            int node = pair.node;
            if(pr < prob[node])
                continue;
            for(Pair next:graph.get(node)){
                double prNext = next.probability;
                int nodeNext = next.node;
                if(prob[nodeNext] < prob[node] * prNext){
                    prob[nodeNext] = prob[node] * prNext;
                    queue.offer(new Pair(prob[nodeNext], nodeNext));
                }
            }
        }
        return prob[end];
    }
}

class Pair implements Comparable<Pair> {
    double probability;
    int node;

    public Pair(double probability, int node) {
        this.probability = probability;
        this.node = node;
    }

    public int compareTo(Pair pair2) {
        if (this.probability == pair2.probability) {
            return this.node - pair2.node;
        } else {
            return this.probability - pair2.probability > 0 ? -1 : 1;
        }
    }
}
```



### 四、并查集

连通：即一种等价关系，满足自反性、对称性、传递性

Union-Find算法主要需要实现API

```java
class UF {
    /* 将 p 和 q 连接 */
    public void union(int p, int q);
    /* 判断 p 和 q 是否连通 */
    public boolean connected(int p, int q);
    /* 返回图中有多少个连通分量 */
    public int count();
}
```



```java
class UF {
    // 连通分量个数
    private int count;
    // 存储一棵树
    private int[] parent;
    // 记录树的“重量”
    private int[] size;

    public UF(int n) {
        this.count = n;
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }
    
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ)
            return;
        
        // 小树接到大树下面，较平衡
        if (size[rootP] > size[rootQ]) {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        } else {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        count--;
    }

    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }

    private int find(int x) {
        while (parent[x] != x) {
            // 进行路径压缩，将孙子接到爷爷上
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    public int count() {
        return count;
    }
}
```



处理元素是字母的

```java
UF uf = new UF(26);
char x = equ.charAt(0);
char y = equ.charAt(3);
uf.union(x-'a',y-'a');
```



### 五、二分搜索

区间开闭的区别：当left迭代到right时，还能否代入去计算

左闭右开：当right初始化为nums.length时，此时是越界的，所以应该终止left==right，即while(left < right)，但此时带来了问题，当收缩到中间时，left==right是漏掉没有判断的，所有需要加一行**return** nums[left] == target ? left : -1;之后的收缩区间，都应该给出左闭右开的区间，将可能漏掉的元素交给最后的判断语句

#### 1.搜索区间左闭右闭

```java
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}

int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界，如果收缩部分未找到的话，结束时left会等于target所在的位置
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}

int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 最后要检查 right 越界的情况
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}

```



#### 2.左闭右开

```java
public int searchRange(int[] nums, int target,boolean isFirst){
    // 包括了检查左边界leftBound(isFirst=true)和检查rightBound
        int left = 0,right = nums.length;
        while(left < right){
            int mid = left + (right-left)/2;
            if(nums[mid] == target){
                if(isFirst)
                    right = mid;
                else
                    left = mid+1;
            }
            else if(nums[mid] < target)
                left = mid+1;
            else if(nums[mid] > target)
                right = mid;
        }
        if(isFirst){
            // nums[left]!=target可以解决left==right区间不为空时漏掉的部分，也会解决left[0,length]时，=0没找到的情况
            if(left==nums.length || nums[left]!=target)
                return -1;
            return left;
        }else{
            if(right==0 || nums[left-1]!=target)
                return -1;
            return left-1;
        }
    }
```



#### 3.优化线性搜索复杂度

![image-20211009213732020](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211009213732020.png)



```java
for (int i = 1; i <= N; i++) {
    if (dp(K - 1, i - 1) == dp(K, N - i))
        return dp(K, N - i);
}
```



### 六、双指针

快慢指针的用途

1）存在步数差，此时先让fast走n步，再让slow和fast以相同的速率迭代

2）存在倍数差，让fast以n倍于slow的速度迭代

#### 1.快慢指针-判断是否有环

```java
boolean hasCycle(ListNode head) {
    ListNode fast, slow;
    fast = slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
        
        if (fast == slow) return true;
    }
    return false;
}
```



#### 2.快慢指针-返回环的起始位置

![image-20211006184432330](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211006184432330.png)

```java
ListNode detectCycle(ListNode head) {
    ListNode fast, slow;
    fast = slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow) break;
    }
    // 上面的代码类似 hasCycle 函数
    if (fast == null || fast.next == null) {
        // fast 遇到空指针说明没有环
        return null;
    }

    slow = head;
    while (slow != fast) {
        fast = fast.next;
        slow = slow.next;
    }
    return slow;
}

```



#### 3.快慢指针-寻找链表中点

```java
ListNode middleNode(ListNode head) {
    ListNode fast, slow;
    fast = slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
    }
    // slow 就在中间位置，如果有两个结点并需要返回前一个时
    //	if(fast != nullptr){
    //   	 slow = slow->next;
    //	}
    return slow;
}
```



#### 4.快慢指针-删除倒数第n个节点

也就是要寻找倒数第n个节点的前一个节点，即**构造步数差**为n的两个快慢指针，让fast先走n步

特殊情况，当fast先走n步后，fast=null，说明要删除的是第一个节点，直接返回head.next即可

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode slow = head,fast = head;
        for(int i = 0;i < n;i++)
            fast = fast.next;

        if (fast == null) {
            // 如果此时快指针走到头了，
            // 说明倒数第 n 个节点就是第一个节点
            return head.next;
        }
        while(fast.next!=null){
            slow = slow.next;
            fast = fast.next;
        }
        slow.next = slow.next.next;
        return head;
    }
```



#### 5.快慢指针-原地去重

让慢指针 `slow` 走在后面，快指针 `fast` 走在前面探路，找到一个不重复的元素就告诉 `slow` 并让 `slow` 前进一步。这样当 `fast` 指针遍历完整个数组 `nums` 后，**`nums[0..slow]` 就是不重复元素**

```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }
    int slow = 0, fast = 0;
    while (fast < nums.length) {
        if (nums[fast] != nums[slow]) {
            slow++;
            // 维护 nums[0..slow] 无重复
            nums[slow] = nums[fast];
        }
        fast++;
    }
    // 数组长度为索引 + 1
    return slow + 1;
}
```



#### 6. 滑动窗口

##### 1）框架

```c++
/* 滑动窗口算法框架 */
void slidingWindow(string s, string t) {
    // 初始化 window 和 need 两个哈希表，记录窗口中的字符和需要凑齐的字符
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;
    
    // 区间 [left, right) 是左闭右开的
    int left = 0, right = 0;
    //  valid 变量表示窗口中满足 need 条件的字符个数
    // 如果 valid 和 need.size 的大小相同，则说明窗口已满足条件，已经完全覆盖了串 T
    int valid = 0; 
    
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        ...

        /*** debug 输出的位置 ***/
        printf("window: [%d, %d)\n", left, right);
        /********************/
        
        // 判断左侧窗口是否要收缩
        while (window needs shrink) {
            // d 是将移出窗口的字符
            char d = s[left];
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新
            ...
        }
    }
}
```

**现在开始套模板，只需要思考以下四个问题**：

1、当移动 `right` 扩大窗口，即加入字符时，应该更新哪些数据？

2、什么条件下，窗口应该暂停扩大，开始移动 `left` 缩小窗口？

3、当移动 `left` 缩小窗口，即移出字符时，应该更新哪些数据？

4、我们要的结果应该在扩大窗口时还是缩小窗口时进行更新？



##### 2）例题：寻找最小覆盖子串

```c++
string minWindow(string s, string t) {
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;

    int left = 0, right = 0;
    int valid = 0;
    // 记录最小覆盖子串的起始索引及长度
    int start = 0, len = INT_MAX;
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        if (need.count(c)) {
            window[c]++;
            if (window[c] == need[c])
                valid++;
        }

        // 判断左侧窗口是否要收缩
        while (valid == need.size()) {
            // 在这里更新最小覆盖子串
            if (right - left < len) {
                start = left;
                len = right - left;
            }
            // d 是将移出窗口的字符
            char d = s[left];
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新
            if (need.count(d)) {
                if (window[d] == need[d])
                    valid--;
                window[d]--;
            }                    
        }
    }
    // 返回最小覆盖子串
    return len == INT_MAX ?
        "" : s.substr(start, len);
}

```



## IV. 动态规划

**重叠子问题、最优子结构、状态转移方程**就是动态规划三要素

首先，动态规划问题的一般形式就是求最值，**求解动态规划的核心问题是穷举**。因为要求最值，肯定要把所有可行的答案穷举出来，然后在其中找最值。

动态规划的穷举有点特别，因为这类问题**存在「重叠子问题」**，如果暴力穷举的话效率会极其低下，所以需要「备忘录」或者「DP table」来优化穷举过程，避免不必要的计算。

而且，动态规划问题一定会**具备「最优子结构」**，才能通过子问题的最值得到原问题的最值。

「状态转移方程」？把 `f(n)` 想做一个状态 `n`，这个状态 `n` 是由状态 `n - 1` 和状态 `n - 2` 相加转移而来

**明确 base case -> 明确「状态」-> 明确「选择」 -> 定义 dp 数组/函数的含义**。

#### 模板

定义时 **dp[状态1] [状态2] = 需要求的值**

```java
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)

```



#### 备忘录（自顶向上）

带「备忘录」的递归算法，把一棵存在巨量冗余的递归树通过「剪枝」，改造成了一幅不存在冗余的递归图，极大减少了子问题

```java
int fib(int N) {
    // 备忘录全初始化为 0
    int[] memo = new int[N + 1];
    // 进行带备忘录的递归
    return helper(memo, N);
}

int helper(int[] memo, int n) {
    // base case
    if (n == 0 || n == 1) return n;
    // 已经计算过，不用再计算了
    if (memo[n] != 0) return memo[n];
    memo[n] = helper(memo, n - 1) + helper(memo, n - 2);
    return memo[n];
}
```



#### **dp 数组的迭代解法**（自底向上）

```java
int fib(int N) {
    if (N == 0) return 0;
    int[] dp = new int[N + 1];
    // base case
    dp[0] = 0; dp[1] = 1;
    // 状态转移
    for (int i = 2; i <= N; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[N];
}
```



#### 状态压缩

如果我们发现每次状态转移只需要 DP table 中的一部分，那么可以尝试用状态压缩来缩小 DP table 的大小，只记录必要的数据，上述例子就相当于把DP table 的大小从 `n` 缩小到 2

```java
int fib(int n) {
    if (n < 1) return 0;
    if (n == 2 || n == 1) 
        return 1;
    int prev = 1, curr = 1;
    for (int i = 3; i <= n; i++) {
        int sum = prev + curr;
        prev = curr;
        curr = sum;
    }
    return curr;
}
```



```c++
for (int i = n - 2; i >= 0; i--) {
    // 存储 dp[i+1][j-1] 的变量
    int pre = 0;
    for (int j = i + 1; j < n; j++) {
        int temp = dp[j];
        if (s[i] == s[j])
            // dp[i][j] = dp[i+1][j-1] + 2;
            dp[j] = pre + 2;
        else
            dp[j] = max(dp[j], dp[j - 1]);
        // 到下一轮循环，pre 就是 dp[i+1][j-1] 了
        pre = temp;
    }
}
```



### 一、高楼扔鸡蛋

LeetCode 887

#### 1.二分搜索优化

dp(k-1,mid-1)和dp(k,n-mid)是单调递增的函数，求最小值即求交点

二分搜索求一个单调递增和单调递减函数的交点

```java
	int left = 0,right = n;
    while(left <= right){
        int mid = left+(right-left)/2;
        int f1 = func1(mid);    // 假设func1单调递增
        int f2 = func2(mid);    // 假设func2单调递增
        if(f1 > f2)
            right = mid-1;
        else
            left = mid+1;
    }
	return left;
```



```java
class Solution {
    int[][] memo;
    public int superEggDrop(int k, int n) {
        memo = new int[k+1][n+1];
        for(int[] m : memo)
            Arrays.fill(m,-1);
        return dp(k,n);
    }


    public int dp(int k,int n){
        if(k == 1 || n == 0)
            return n;
        if(memo[k][n] != -1)
            return memo[k][n];
        
        int res = n+1;
        int left = 1,right = n;
        while(left <= right){
            int mid = (left+right)/2;
            int broken = dp(k-1,mid-1);     //单调递增的函数
            int notBroken = dp(k,n-mid);    //单调递减的函数
            if(broken > notBroken){     // 需要减小查询的值
                right = mid-1;
                res = Math.min(res,broken+1);
            }else{                      // 增大
                left = mid+1;   
                res = Math.min(res,notBroken+1);
            }
            memo[k][n] = res;
        }
        return res;
    }
}
```



### 二、最长递增子序列+最大子数组

最大子数组和」就和「最长递增子序列」非常类似，`dp` 数组的定义是**「以 `nums[i]` 为结尾的最大子数组和/最长递增子序列为 `dp[i]`」**。因为只有这样定义才能将 `dp[i+1]` 和 `dp[i]` 建立起联系，利用数学归纳法写出状态转移方程。



#### 1.最长递增子序列

根据刚才我们对 `dp` 数组的定义，现在想求 `dp[5]` 的值，也就是想求以 `nums[5]` 为结尾的最长递增子序列。

**`nums[5] = 3`，既然是递增子序列，我们只要找到前面那些结尾比 3 小的子序列，然后把 3 接到最后，就可以形成一个新的递增子序列，而且这个新的子序列长度加一**。

显然，可能形成很多种新的子序列，但是我们只选择最长的那一个，把最长子序列的长度作为 `dp[5]` 的值即可。

```java
public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        if(n == 1)  return 1;
        int[] dp = new int[n];
        Arrays.fill(dp,1);
        for (int i = 0; i < nums.length; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) 
                    dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        int res = 0;
        for (int i = 0; i < dp.length; i++) {
            res = Math.max(res, dp[i]);
        }
        return res;
    }
```



#### 2.最长递增子序列进阶版-二维数组

需要排列进行预处理：

按照第一个元素升序排列，第二个元素降序排列，这样在第二个元素上寻找最长递增子序列即答案

第二个元素降序排列的原因：对于宽度 `w` 相同的数对，要对其高度 `h` 进行降序排序。因为两个宽度相同的信封不能相互包含的，逆序排序保证在 `w` 相同的数对中最多只选取一个(对于x相等，y降序的一组数，选择了某个数，那么比他大或者小的不能选(否则加上更大/更小的数不满足上升子序列))

```java
public int maxEnvelopes(int[][] envelopes) {
    	// lambada实现二维数组排序
        Arrays.sort(envelopes,(a,b) -> (a[0]==b[0] ? b[1]-a[1] : a[0]-b[0]));
        int n = envelopes.length;
        int[] top = new int[n];
        int piles = 0;
        for(int i = 0;i < n;i++){
            int poker = envelopes[i][1];
            int left = 0,right = piles;
            // 寻找左侧边界
            while(left < right){
                int mid = left+(right-left)/2;
                if(top[mid] >= poker)
                    right = mid;
                else if(top[mid] < poker)
                    left = mid+1;
            }

            if(left == piles)   piles++;
            top[left] = poker;
        }
        return piles;
    }
```



#### 3.最大子数组和

`dp[i]` 有两种「选择」，要么与前面的相邻子数组连接，形成一个和更大的子数组；要么不与前面的子数组连接，自成一派，自己作为一个子数组

```java
dp[i] = Math.max(nums[i], nums[i] + dp[i - 1]);
```



##### 基础版

```java
public int maxSubArray(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];  //以nums[j]结尾的最长序列和
        dp[0] = nums[0];
        for(int i = 1; i < n;i++){
            dp[i] = Math.max(nums[i], nums[i] + dp[i - 1]);
        }

        int res = Integer.MIN_VALUE;
        for(int i = 0;i < n;i++){
            res = Math.max(res,dp[i]);
        }
        return res;
    }
```



##### 扑克牌游戏+二分优化

```java
public int lengthOfLIS(int[] nums) {
    int[] top = new int[nums.length];
    // 牌堆数初始化为 0
    int piles = 0;
    for (int i = 0; i < nums.length; i++) {
        // 要处理的扑克牌
        int poker = nums[i];

        /***** 搜索左侧边界的二分查找 *****/
        int left = 0, right = piles;
        while (left < right) {
            int mid = (left + right) / 2;
            if (top[mid] > poker) {
                right = mid;
            } else if (top[mid] < poker) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        /*********************************/
        
        // 没找到合适的牌堆，新建一堆
        if (left == piles) piles++;
        // 把这张牌放到牌堆顶
        top[left] = poker;
    }
    // 牌堆数就是 LIS 长度
    return piles;
}
```



### 三、最长公共子序列

![image-20211013185859866](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211013185859866.png)

#### 1. 自顶向下

```java
// 备忘录，消除重叠子问题
int[][] memo;

/* 主函数 */
int longestCommonSubsequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    // 备忘录值为 -1 代表未曾计算
    memo = new int[m][n];
    for (int[] row : memo) 
        Arrays.fill(row, -1);
    // 计算 s1[0..] 和 s2[0..] 的 lcs 长度
    return dp(s1, 0, s2, 0);
}

// 定义：计算 s1[i..] 和 s2[j..] 的最长公共子序列长度
int dp(String s1, int i, String s2, int j) {
    // base case
    if (i == s1.length() || j == s2.length()) {
        return 0;
    }
    // 如果之前计算过，则直接返回备忘录中的答案
    if (memo[i][j] != -1) {
        return memo[i][j];
    }
    // 根据 s1[i] 和 s2[j] 的情况做选择
    if (s1.charAt(i) == s2.charAt(j)) {
        // s1[i] 和 s2[j] 必然在 lcs 中
        memo[i][j] = 1 + dp(s1, i + 1, s2, j + 1);
    } else {
        // s1[i] 和 s2[j] 至少有一个不在 lcs 中
        memo[i][j] = Math.max(
            dp(s1, i + 1, s2, j),
            dp(s1, i, s2, j + 1)
        );
    }
    return memo[i][j];
}
```



#### 2. 自底向上

```java
int longestCommonSubsequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    // 定义：s1[0..i-1] 和 s2[0..j-1] 的 lcs 长度为 dp[i][j]
    // 目标：s1[0..m-1] 和 s2[0..n-1] 的 lcs 长度，即 dp[m][n]
    // base case: dp[0][..] = dp[..][0] = 0

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // 现在 i 和 j 从 1 开始，所以要减一
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                // s1[i-1] 和 s2[j-1] 必然在 lcs 中
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                // s1[i-1] 和 s2[j-1] 至少有一个不在 lcs 中
                dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
            }
        }
    }

    return dp[m][n];
}
```



#### 3. 最长公共上升子序列

定义dp[i][j]表示以a串的前i个字符，b串的前j个字符且以b[j]为结尾构成的LCIS的长度

```java
a[i]!=b[j]:   F[i][j]=F[i-1][j]
a[i]==b[j]:   F[i][j]=max(F[i-1][k])+1 1<=k<=j1&&b[j]>b[k]

1）在a[i]!=b[j]的情况下必然有F[i][j]=F[i-1][j]，即如果F[i][j]>0那么就说明a[1]…a[i]中必然有一个字符a[k]等于b[j]（如果F[i][j]=0，不影响结果）
2）a[i]==b[j]，需要去找一个最长的且能让b[j]接在其末尾的LCIS，枚举b[1]…b[j-1]，得到最长且小于b[j]的序列


private static List<Integer> seq = new LinkedList<>();
      public static int[] LCIS(int[] a, int[] b) {
        int m = a.length-1;
        int n = b.length-1; // 数据a[1][m]
        int[][] dp = new int[m+2][n+2];
        for(int i = 1;i <= m;i++){
            for(int j = 1;j <= n;j++){
                if(a[i] != b[j])
                    dp[i][j] = dp[i][j-1];
                else{
                    int max = 0;
                    // 寻找小于a[i]的最长
                    for(int k = 1;k < i;k++){
                        if(a[k]<a[i] && max<dp[k][j-1])
                            max = dp[k][j-1];
                    }
                    dp[i][j] = max+1;
                }
            }
        }
        int res = 0,index = 0;  // index即最长序列的最后一位
        for(int i = 1;i <= m;i++){
            if(res < dp[i][n]){
                res = dp[i][n];
                index = i;
            }
        }
        if(res != 0)
            generate(index,n,a,b,dp);
        int[] LCIS = new int[seq.size()];
        for(int i = 0;i < seq.size();i++)
            LCIS[i] = seq.get(i);

        return LCIS;
      }

      // 生成LCIS序列，即每次寻找dp[s][n]的最大值，就是s的上一位
      public static void generate(int s,int t,int[] a,int[] b,int[][] dp){
          if(s==0 || t==0)
              return;
          if(a[s] != b[t])
              generate(s,t-1,a,b,dp);
          else {
              int min = 0,index = 0;
              // 寻找s之前的那个不大于a[s]的最长序列的结尾index
              for(int k = 1;k < s;k++){
                  if(a[k]<a[s] && min<dp[k][t-1]){
                      min = dp[k][t-1];
                      index = k;
                  }
              }
              if(min != 0)
                  generate(index,t-1,a,b,dp);
              seq.add(a[s]);
          }
      }
```



### 四、子序列问题

1.一维dp数组

```java
int n = array.length;
int[] dp = new int[n];

for (int i = 1; i < n; i++) {
    for (int j = 0; j < i; j++) {
        dp[i] = 最值(dp[i], dp[j] + ...)
    }
}
```

例如：最长递增子序列， **以\**`array[i]`\**结尾的目标子序列（最长递增子序列）的长度是`dp[i]`**。



2.二维dp数组

```java
int n = arr.length;
int[][] dp = new dp[n][n];

for (int i = 0; i < n; i++) {
    for (int j = 1; j < n; j++) {
        if (arr[i] == arr[j]) 
            dp[i][j] = dp[i][j] + ...
        else
            dp[i][j] = 最值(...)
    }
}
```

**2.1** **涉及两个字符串/数组时**（比如最长公共子序列），dp 数组的含义如下：

​	**在子数组`arr1[0..i]`和子数组`arr2[0..j]`中，我们要求的子序列（最长公共子序列）长度为`dp[i][j]`**。

**2.2** **只涉及一个字符串/数组时**（比如本文要讲的最长回文子序列），dp 数组的含义如下：

​	**在子数组`array[i..j]`中，我们要求的子序列（最长回文子序列）的长度为`dp[i][j]`**。	



### 五、背包问题

#### 1.01背包问题

`dp[i][w]` 的定义如下：对于前 `i` 个物品，当前背包的容量为 `w`，这种情况下可以装的最大价值是 `dp[i][w]`。

```java
int knapsack(int W, int N, vector<int>& wt, vector<int>& val) {
    // base case 已初始化,base case dp[i][0] = 0,dp[0][i]=0
    vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));
    for (int i = 1; i <= N; i++) {
        for (int w = 1; w <= W; w++) {
            if (w - wt[i-1] < 0) {
                // 这种情况下只能选择不装入背包
                dp[i][w] = dp[i - 1][w];
            } else {
                // 装入或者不装入背包，择优
                dp[i][w] = max(dp[i - 1][w - wt[i-1]] + val[i-1], 
                               dp[i - 1][w]);
            }
        }
    }
    
    return dp[N][W];
}
```



#### 2. 子集背包

`dp[i][j] = x` 表示，对于前 `i` 个物品，当前背包的容量为 `j` 时，若 `x` 为 `true`，则说明可以恰好将背包装满，若 `x` 为 `false`，则说明不能恰好将背包装满。

```java
bool canPartition(vector<int>& nums) {
    int sum = 0;
    for (int num : nums) sum += num;
    // 和为奇数时，不可能划分成两个和相等的集合
    if (sum % 2 != 0) return false;
    int n = nums.size();
    sum = sum / 2;
    vector<vector<bool>> 
        dp(n + 1, vector<bool>(sum + 1, false));
    // base case
    for (int i = 0; i <= n; i++)
        dp[i][0] = true;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= sum; j++) {
            if (j - nums[i - 1] < 0) {
               // 背包容量不足，不能装入第 i 个物品
                dp[i][j] = dp[i - 1][j]; 
            } else {
                // 装入或不装入背包，装的的话前i-1个物品只需要装满j-nums[i-1]即可
                dp[i][j] = dp[i - 1][j] || dp[i - 1][j-nums[i-1]];
            }
        }
    }
    return dp[n][sum];
}
```



状态压缩

**注意到 `dp[i][j]` 都是通过上一行 `dp[i-1][..]` 转移过来的**，之前的数据都不会再使用了

**唯一需要注意的是 `j` 应该从后往前反向遍历，因为每个物品（或者说数字）只能用一次，以免之前的结果影响其他的结果**。

```java
bool canPartition(vector<int>& nums) {
    int sum = 0, n = nums.size();
    for (int num : nums) sum += num;
    if (sum % 2 != 0) return false;
    sum = sum / 2;
    vector<bool> dp(sum + 1, false);
    // base case
    dp[0] = true;

    for (int i = 0; i < n; i++) 
        for (int j = sum; j >= 0; j--) 
            if (j - nums[i] >= 0) 
                dp[j] = dp[j] || dp[j - nums[i]];

    return dp[sum];
}
```



#### 3. 完全背包

完全背包与01背包最大的区别在于：完全背包的物品是无限个的，也即可以重复使用，这就使得它的状态转移方程

如果放入第i个物品，则dp[i] [j]  = dp[i] [j-coins]，使得当前物品i可以在子问题中继续使用

```java
int change(int amount, int[] coins) {
    int n = coins.length;
    int[][] dp = int[n + 1][amount + 1];
    // base case
    for (int i = 0; i <= n; i++) 
        dp[i][0] = 1;

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= amount; j++)
            if (j - coins[i-1] >= 0)
                dp[i][j] = dp[i - 1][j] 
                         + dp[i][j - coins[i-1]];	// 重点：放入i时，是dp[i][j - coins[i-1]]
            else 
                dp[i][j] = dp[i - 1][j];
    }
    return dp[n][amount];
}
```



### 六、转盘问题（自由之路）

难点在于：对于当前的选项，仍需要考虑本次选择对于下一次选择的影响（即当前选择+子问题dubProblem的值）

```c++
// 字符 -> 索引列表
unordered_map<char, vector<int>> charToIndex;
// 备忘录
vector<vector<int>> memo;

/* 主函数 */
int findRotateSteps(string ring, string key) {
    int m = ring.size();
    int n = key.size();
    // 备忘录全部初始化为 0
    memo.resize(m, vector<int>(n, 0));
    // 记录圆环上字符到索引的映射
    for (int i = 0; i < ring.size(); i++) {
        charToIndex[ring[i]].push_back(i);
    }
    // 圆盘指针最初指向 12 点钟方向，
    // 从第一个字符开始输入 key
    return dp(ring, 0, key, 0);
}

// 计算圆盘指针在 ring[i]，输入 key[j..] 的最少操作数
int dp(string& ring, int i, string& key, int j) {
    // base case 完成输入
    if (j == key.size()) return 0;
    // 查找备忘录，避免重叠子问题
    if (memo[i][j] != 0) return memo[i][j];
    
    int n = ring.size();
    // 做选择
    int res = INT_MAX;
    // ring 上可能有多个字符 key[j]
    for (int k : charToIndex[key[j]]) {
        // 拨动指针的次数
        int delta = abs(k - i);
        // 选择顺时针还是逆时针
        delta = min(delta, n - delta);
        // 将指针拨到 ring[k]，继续输入 key[j+1..]
        int subProblem = dp(ring, k, key, j + 1);
        // 选择「整体」操作次数最少的
        // 加一是因为按动按钮也是一次操作
        res = min(res, 1 + delta + subProblem);
    }
    // 将结果存入备忘录
    memo[i][j] = res;
    return res;
}
```

#### 3. 完全背包

```java
int change(int amount, int[] coins) {
    int n = coins.length;
    int[][] dp = int[n + 1][amount + 1];
    // base case
    for (int i = 0; i <= n; i++) 
        dp[i][0] = 1;

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= amount; j++)
            if (j - coins[i-1] >= 0)
                dp[i][j] = dp[i - 1][j] 
                         + dp[i][j - coins[i-1]];
            else 
                dp[i][j] = dp[i - 1][j];
    }
    return dp[n][amount];
}
```



### 七、博弈问题

核心思路是在二维 dp 的基础上使用元组分别存储两个人的博弈结果



[877. 石子游戏 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/stone-game/)

dp[i][j].fir 表示，对于 piles[i...j] 这部分石头堆，先手能获得的最高分数。 

dp[i][j].sec 表示，对于 piles[i...j] 这部分石头堆，后手能获得的最高分数。



```java
class Solution {
    public boolean stoneGame(int[] piles) {
        int n = piles.length;
        // 初始化 dp 数组
        Pair[][] dp = new Pair[n][n];
        for (int i = 0; i < n; i++) 
            for (int j = i; j < n; j++)
                dp[i][j] = new Pair(0, 0);
        // 填入 base case
        for (int i = 0; i < n; i++) {
            dp[i][i].fir = piles[i];
            dp[i][i].sec = 0;
        }
        // 斜着遍历数组
        for (int l = 2; l <= n; l++) {
            for (int i = 0; i <= n - l; i++) {
                int j = l + i - 1;
                // 先手选择最左边或最右边的分数
                int left = piles[i] + dp[i+1][j].sec;
                int right = piles[j] + dp[i][j-1].sec;
                // 套用状态转移方程，在先手选完之后，后手只能获得剩下的最大值的可能
                if (left > right) {
                    dp[i][j].fir = left;
                    dp[i][j].sec = dp[i+1][j].fir;
                } else {
                    dp[i][j].fir = right;
                    dp[i][j].sec = dp[i][j-1].fir;
                }
            }
        }
        Pair res = dp[0][n-1];
        return res.fir - res.sec > 0 ;
    }

    class Pair {
        int fir, sec;
        Pair(int fir, int sec) {
            this.fir = fir;
            this.sec = sec;
        }
    }
}
```



### 八、股票问题

**每天都有三种「选择」**：买入、卖出、无操作，我们用 `buy`, `sell`, `rest` 表示这三种选择。但问题是，并不是每天都可以任意选择这三种选择的，因为 `sell` 必须在 `buy` 之后，`buy` 必须在 `sell` 之后。那么 `rest` 操作还应该分两种状态，一种是 `buy` 之后的 `rest`（持有了股票），一种是 `sell` 之后的 `rest`（没有持有股票）。而且别忘了，我们还有交易次数 `k` 的限制，就是说你 `buy` 还只能在 `k > 0` 的前提下操作。



**「状态」有三个**，第一个是天数，第二个是允许交易的最大次数，第三个是当前的持有状态（即之前说的 `rest` 的状态）

```c++
dp[i][k][0/1]定义：第i天至多进行k次交易，并且0不持有股票/1持有股票的情况下，的最大价值

base case:
i = 0时：
dp[0][k][0] = 0				// 第一天时，如果不持有，不管交易次数的上限是多少都是0
dp[0][k][1] = -prices[0];	// 第一天时，如果持有，不管交易次数的上限是多少都是-prices[0]
k = 0时
dp[i][0][0] = 0						// 当交易次数的上限为0时，不持有说明没有发生过交易，所以为0
dp[i][1][1] = Integer.MIN_VALUE;	// 当交易次数的上限为0时，持有是不合法的，所以为Integer.MIN_VALUE

状态转移方程：
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
```

`dp[i][k]` 不会依赖 `dp[i][k - 1]`，而是依赖 `dp[i - 1][k - 1]`，对于 `dp[i - 1][...]`，都是已经计算出来的。所以不管你是 `k = max_k, k--`，还是 `k = 1, k++`，都是可以得出正确答案的

k=infinite模板

```java
public int maxProfit(int k, int[] prices) {
        int n = prices.length;
        if(k == 0 || n==0)  return 0;
        k = Math.min(k,n/2);	
    	//一次交易由买入和卖出构成，至少需要两天。
    	//所以说有效的限制 k 应该不超过 n/2，如果超过，就没有约束作用了，相当于 k = +infinity
        int[][][] dp = new int[n][k+1][2];
        for(int i = 0;i < n;i++){
            for(int j = k;j >= 1;j--){
                if(i == 0){		// base case
                    dp[i][j][0] = 0;
                    dp[i][j][1] = -prices[i];
                }else{
                    dp[i][j][0] = Math.max(dp[i-1][j][0],dp[i-1][j][1]+prices[i]);
                    dp[i][j][1] = Math.max(dp[i-1][j][1],dp[i-1][j-1][0]-prices[i]);
                }
            }
        }
        return dp[n-1][k][0];
    }
```





### 九、戳气球

技巧：**定义dp[i] [j]为开区间(i,j)上的最大值**，使得子问题之间独立

增加两侧的**虚拟气球**，处理边界

**`dp[i][j] = x`表示，戳破气球`i`和气球`j`之间（开区间，不包括`i`和`j`）的所有气球，可以获得的最高分数为`x`**。

```java
int maxCoins(int[] nums) {
    int n = nums.length;
    // 添加两侧的虚拟气球
    int[] points = new int[n + 2];
    points[0] = points[n + 1] = 1;
    for (int i = 1; i <= n; i++) {
        points[i] = nums[i - 1];
    }
    // base case 已经都被初始化为 0
    int[][] dp = new int[n + 2][n + 2];
    // 开始状态转移
    // i 应该从下往上
    for (int i = n; i >= 0; i--) {
        // j 应该从左往右
        for (int j = i + 1; j < n + 2; j++) {
            // 最后戳破的气球是哪个？
            for (int k = i + 1; k < j; k++) {
                // 择优做选择
                dp[i][j] = Math.max(
                    dp[i][j], 
                    dp[i][k] + dp[k][j] + points[i]*points[j]*points[k]
                );
            }
        }
    }
    return dp[0][n + 1];
}
```



## V 贪心算法

### 一、区间问题

#### 1.区间调度问题

将活动按结束时间fi升序排列，每次选择fi最小的活动，子问题即Si = { j属于S| sj >= fi}

```java
n = length(S);
A = {1}	//将fi最小的活动加入调度长度最长的序列A中
j = 1	// 即上一个选择的活动index
For i <- 2 to n Do:
	if si >= fi
    Then A <- AU{i};j-<i;
return A
```

```java
public int findMinArrowShots(int[][] points) {
        int n = points.length;
        Arrays.sort(points,Comparator.comparingInt(a -> a[1]));
        int sum = 1,j = 0;
        for(int i = 1;i < n;i++){
            if(points[i][0] > points[j][1]){
                sum++;
                j = i;
            }
        }
        return sum;
    }
```



#### 2.求覆盖给定区间的最小个数

是先按照起点升序排序，如果起点相同的话按照终点降序排序

对于当前已被选择的区间有curEnd，需要找到开始时间小于curEnd(才能保证全部覆盖)的区间中，结束时间最晚的区间

```java
int videoStitching(int[][] clips, int T) {
    if (T == 0) return 0;
    // 按起点升序排列，起点相同的降序排列
    Arrays.sort(clips, (a, b) -> {
        if (a[0] == b[0]) {
            return b[1] - a[1];
        }
        return a[0] - b[0];
    });
    // 记录选择的短视频个数
    int res = 0;

    int curEnd = 0, nextEnd = 0;
    int i = 0, n = clips.length;
    while (i < n && clips[i][0] <= curEnd) {
        // 在第 res 个视频的区间内贪心选择下一个视频
        while (i < n && clips[i][0] <= curEnd) {
            nextEnd = Math.max(nextEnd, clips[i][1]);
            i++;
        }
        // 找到下一个视频，更新 curEnd
        res++;
        curEnd = nextEnd;
        if (curEnd >= T) {
            // 已经可以拼出区间 [0, T]
            return res;
        }
    }
    // 无法连续拼出区间 [0, T]
    return -1;
}
```



#### 3.删除被覆盖区间

![image-20211025214419130](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211025214419130.png)

对于这三种情况，我们应该这样处理：

对于情况一，找到了覆盖区间。

对于情况二，两个区间可以合并，成一个大区间。

对于情况三，两个区间完全不相交。

```java
int removeCoveredIntervals(int[][] intvs) {
    // 按照起点升序排列，起点相同时降序排列
    Arrays.sort(intvs, (a, b) -> {
        if (a[0] == b[0]) {
            return b[1] - a[1];
        }
        return a[0] - b[0]; 
    });

    // 记录合并区间的起点和终点
    int left = intvs[0][0];
    int right = intvs[0][1];
    
    int res = 0;
    for (int i = 1; i < intvs.length; i++) {
        int[] intv = intvs[i];
        // 情况一，找到覆盖区间
        if (left <= intv[0] && right >= intv[1]) {
            res++;
        }
        // 情况二，找到相交区间，合并
        if (right >= intv[0] && right <= intv[1]) {
            right = intv[1];
        }
        // 情况三，完全不相交，更新起点和终点
        if (right < intv[0]) {
            left = intv[0];
            right = intv[1];
        }
    }
    
    return intvs.length - res;
}
```



#### 4.区间合并问题

按开始时间排序，寻找相交区间中，end最大的点作为合并后的新end

```java
public int[][] merge(int[][] intervals) {
        int n = intervals.length;
        Arrays.sort(intervals,new Comparator<int[]>(){
            public int compare(int[] a,int[] b){
                return a[0]-b[0];
            }
        });
        List<int[]> res = new ArrayList<>();
        for(int i = 0;i < n;i++){
            int left = intervals[i][0],right = intervals[i][1];
            if(res.size()==0 || res.get(res.size()-1)[1] < left){
                res.add(new int[]{left,right});
            }else{
                res.get(res.size()-1)[1] = Math.max(res.get(res.size()-1)[1],right);
            }
        }
        return res.toArray(new int[res.size()][]);
    }
```



#### 5. 区间列表的交集

![image-20211026200527458](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211026200527458.png)



![image-20211026200557810](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211026200557810.png)

```java
public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
        int m = firstList.length,n = secondList.length;
        List<int[]> res = new ArrayList<>();
        int i = 0,j = 0;
        while(i < m && j < n){
            int a1 = firstList[i][0];
            int a2 = firstList[i][1];
            int b1 = secondList[j][0];
            int b2 = secondList[j][1];
            if(b2 >= a1 && a2 >= b1)
                res.add(new int[]{Math.max(a1,b1),Math.min(a2,b2)});
            if(b2 < a2) j++;
            else i++;
        }
        return res.toArray(new int[res.size()][]);
    }
```



#### 6.重合区间个数

![image-20211026201831631](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211026201831631.png)

```java
int minMeetingRooms(int[][] meetings) {
    int n = meetings.length;
    int[] begin = new int[n];
    int[] end = new int[n];
    for(int i = 0; i < n; i++) {
        begin[i] = meetings[i][0];
        end[i] = meetings[i][1];
    }
    Arrays.sort(begin);
    Arrays.sort(end);

    // 扫描过程中的计数器
    int count = 0;
    // 双指针技巧
    int res = 0, i = 0, j = 0;
    while (i < n && j < n) {
        if (begin[i] < end[j]) {
            // 扫描到一个红点
            count++;
            i++;
        } else {
            // 扫描到一个绿点
            count--;
            j++;
        }
        // 记录扫描过程中的最大值
        res = Math.max(res, count);
    }
    
    return res;
}
```





## VI. 算法技巧

### 一、单调栈

单调栈常用于Next Greater Number问题

首先，讲解 Next Greater Number 的原始问题：给你一个数组，返回一个等长的数组，对应索引存储着下一个更大元素，如果没有更大的元素，就存 -1。不好用语言解释清楚，直接上一个例子：

给你一个数组 [2,1,2,4,3]，你返回数组 [4,2,4,-1,-1]。

解释：第一个 2 后面比 2 大的数是 4; 1 后面比 1 大的数是 2；第二个 2 后面比 2 大的数是 4; 4 后面没有比 4 大的数，填 -1；3 后面没有比 3 大的数，填 -1。



#### 1. 单向数组

```c++
vector<int> nextGreaterElement(vector<int>& nums) {
    vector<int> ans(nums.size()); // 存放答案的数组
    stack<int> s;
    for (int i = nums.size() - 1; i >= 0; i--) { // 倒着往栈里放
        while (!s.empty() && s.top() <= nums[i]) { // 判定个子高矮
            s.pop(); // 矮个起开，反正也被挡着了。。。
        }
        ans[i] = s.empty() ? -1 : s.top(); // 这个元素身后的第一个高个
        s.push(nums[i]); // 进队，接受之后的身高判定吧！
    }
    return ans;
}
```



#### 2. 循环数组

```c++
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n); // 存放结果
    stack<int> s;
    // 假装这个数组长度翻倍了
    for (int i = 2 * n - 1; i >= 0; i--) {
        while (!s.empty() && s.top() <= nums[i % n])
            s.pop();
        res[i % n] = s.empty() ? -1 : s.top();
        s.push(nums[i % n]);
    }
    return res;
}
```



### 二、KMP匹配

[2021.08.30 前缀函数和KMP - eleveni - 博客园 (cnblogs.com)](https://www.cnblogs.com/eleveni/p/15216837.html)

```c++
int strStr(string haystack, string needle) {
        int m = haystack.length();
        int n = needle.length();
        if(n == 0)  return 0;
        int next[n];    // k值实际是j位前的子串的最大重复子串的长度
        getNext(needle,next);
        int i = 0,j = 0;
        while(i < m && j < n){
            if(j==-1 || haystack[i] == needle[j])
                i++,j++;
            else
                j = next[j];
        }
        return j == n ? i-j : -1; 
    }

    void getNext(string s,int next[]){
        int j = 0,k = -1;
        next[0] = -1;
        int len = s.length()-1;
        while(j < len){
            if(k == -1 || s[j]==s[k])
                next[++j] = ++k;
            else
                k = next[k];    // 当不适配的时候，就需要[0,k]之间的最大重复子串的长度，即next[k]
        }
    }
```



### 三、回溯算法dfs

#### 1. 思想

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。你只需要思考 3 个问题：

1、路径：也就是已经做出的选择。

2、选择列表：也就是你当前可以做的选择。

3、结束条件：也就是到达决策树底层，无法再做选择的条件

```
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```



**写 `backtrack` 函数时，需要维护走过的「路径」和当前可以做的「选择列表」，当触发「结束条件」时，将「路径」记入结果集**

#### 2. n皇后

因为皇后是一行一行从上往下放的，所以左下方，右下方和正下方不用检查（还没放皇后）；因为一行只会放一个皇后，所以每行不用检查。也就是最后只用检查上面，左上，右上三个方向。

##### 1）求所有问题的解

```c++
class Solution {
public:
    vector<vector<string>> res;
    vector<vector<string>> solveNQueens(int n) {
        vector<string> board(n,string(n,'.'));
        backTrack(board,0);
        return res;
    }

    void backTrack(vector<string>& board,int row){
        if(row == board.size()){
            res.push_back(board);
            return;
        }
        int n = board[row].size();
        for(int col = 0;col < n;col++){
            if(!isValid(board,row,col))
                continue;
            board[row][col] = 'Q';
            backTrack(board,row+1);
            board[row][col] = '.';
        }
    }

    bool isValid(vector<string>& board,int row,int col){
        int n = board.size();
        for(int i = 0;i < n;i++){
            if(board[i][col] == 'Q')
                return false;
        }
        for(int i = row-1,j = col+1;i >= 0 && j < n;i--,j++){
            if(board[i][j] == 'Q')
                return false;
        }
        for(int i = row-1,j = col-1;i>=0 && j>=0;i--,j--){
            if(board[i][j] == 'Q')
                return false;
        }
        return true;
    }
};
```



##### 2）求是否有解

```c++
// 函数找到一个答案后就返回 true
bool backtrack(vector<string>& board, int row) {
    // 触发结束条件
    if (row == board.size()) {
        res.push_back(board);
        return true;
    }
    ...
    for (int col = 0; col < n; col++) {
        ...
        board[row][col] = 'Q';

        if (backtrack(board, row + 1))
            return true;
        
        board[row][col] = '.';
    }

    return false;
}
```



### 四、bfs

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路
    
    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```



## VII、实现技巧

### 一、.二维数组排序

最好写成以下形式，不然在某些场合可能因为溢出出错

```java
public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
```



```java
（1）lambada表达式
// 先按第一列元素升序排序，如果第一列相等再按第二列元素升序；
Arrays.sort(arr, (e1,e2)->(e1[0]==e2[0]?(e1[1]-e2[1]):(e1[0]-e2[0])));

（2）Comparator 内部类
Arrays.sort(arr, new Comparator<int[]>() {    // 匿名内部类
	@Override
	public int compare(int[] e1, int[] e2) {
		// 如果第一列元素相等，则比较第二列元素
		if (e1[0]==e2[0]) return e1[1]-e2[1];   // e1[1]-e2[1]表示对于第二列元素进行升序排序
		return e1[0]-e2[0];                     // e1[0]-e2[0]表示对于第一列元素进行升序排序
	}
});

（3） comparingInt方法
static <T> Comparator<T> comparingInt(ToIntFunction <T> keyExtractor)
参数：此方法接受单个参数keyExtractor，该参数是用于提取整数排序键的函数。
Arrays.sort(intervals,Comparator.comparingInt(a -> a[1]));
```



### 二、位运算

#### 1. n&(n-1)

##### 消除数字 `n` 的二进制表示中的最后一个 1

例：计算1的个数

```c++
int hammingWeight(int n) {
    int res = 0;
    while (n != 0) {
        n = n & (n - 1);
        res++;
    }
    return res;
}
```

例：判断一个数是不是二的倍数

一个数如果是 2 的指数，那么它的二进制表示一定只含有一个 1

```c++
boolean isPowerOfTwo(int n) {
    if (n <= 0) return false;
    return (n & (n - 1)) == 0;
}
```



#### 2. a ^ a = 0

成对儿的数字就会变成 0，落单的数字和 0 做异或还是它本身

一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`

```java
int singleNumber(int[] nums) {
    int res = 0;
    for (int n : nums) {
        res ^= n;
    }
    return res;
}
```



#### 3. n!尾0的个数

计算因子5的个数

```java
int trailingZeroes(int n) {
    int res = 0;
    for (int d = n; d / 5 > 0; d = d / 5) {
        res += d / 5;
    }
    return res;
}
```



#### 4. 欧拉筛

**让每一个合数被其最小质因数筛到**

if (i % prime[j] == 0)这步的理解，

当i是prime[j]的整数倍时，记 m = i / prime[j]，那么 i * prime[j+1] 就可以变为 (m * prime[j+1]) * prime[j]，这说明 i * prime[j+1] 是 prime[j] 的整数倍，不需要再进行标记(在之后会被 prime[j] * 某个数 标记，**留给未来i迭代到更大的数,也即更后的迭代，与素数质因子划去，使得素数永远是作为最小质因子存在**)

```java
当i % primes.get(j) == 0，即 i= prime[j]*m
下一步划掉的数：i*prime[j+1] = prime[j]*m * prime[j+1]，是prime[j]的倍数，可以由prime[j]*i来划去
```



```java
public int countPrimes(int n) {
        List<Integer> primes = new ArrayList<Integer>();
        int[] isPrime = new int[n];
        Arrays.fill(isPrime, 1);
        for (int i = 2; i < n; ++i) {
            if (isPrime[i] == 1) {
                primes.add(i);
            }
            for (int j = 0; j < primes.size() && i * primes.get(j) < n; ++j) {
                isPrime[i * primes.get(j)] = 0;
                // i*prime[j+..]之后的数都可以有prime[j]来划掉
                if (i % primes.get(j) == 0) {
                    break;
                }
            }
        }
        return primes.size();
    }。
```



#### 5. 埃氏筛法

```java
int countPrimes(int n) {
    boolean[] isPrim = new boolean[n];
    Arrays.fill(isPrim, true);
    // i只需要划到sqrt(n)
    for (int i = 2; i * i < n; i++) 
        if (isPrim[i]) 
            // j从i^2开始，i*(i-1)已经在此之前被计算过了
            for (int j = i * i; j < n; j += i) 
                isPrim[j] = false;
    
    int count = 0;
    for (int i = 2; i < n; i++)
        if (isPrim[i]) count++;
    
    return count;
}
```



#### 6. 模运算

**(a \* b) % k = (a % k)(b % k) % k**

```java
int base = 1337;
// 计算 a 的 k 次方然后与 base 求模的结果，转换为a^k % base = (a%base)^k % base = (a%base%base)^k
int mypow(int a, int k) {
    // 对因子求模
    a %= base;
    int res = 1;
    for (int _ = 0; _ < k; _++) {
        // 这里有乘法，是潜在的溢出点
        res *= a;
        // 对乘法结果求模
        res %= base;
    }
    return res;
}

int superPow(int a, vector<int>& b) {
    if (b.empty()) return 1;
    int last = b.back();
    b.pop_back();
    
    int part1 = mypow(a, last);
    int part2 = mypow(superPow(a, b), 10);
    // 每次乘法都要求模
    return (part1 * part2) % base;
}
```



#### 7. 快速幂

![image-20211102211601809](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211102211601809.png)

```c++
//递归快速幂
int qpow(int a, int n)
{
    if (n == 0)
        return 1;
    else if (n % 2 == 1)
        return qpow(a, n - 1) * a;
    else
    {
        int temp = qpow(a, n / 2);
        return temp * temp;
    }
}
```

注意，这个temp变量是必要的，因为如果不把![[公式]](https://www.zhihu.com/equation?tex=a%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D)记录下来，直接写成qpow(a, n /2)*qpow(a, n /2)，那会计算两次![[公式]](https://www.zhihu.com/equation?tex=a%5E%7B%5Cfrac%7Bn%7D%7B2%7D%7D)，整个算法就退化为了 ![[公式]](https://www.zhihu.com/equation?tex=O%28n%29) 



```cpp
//非递归快速幂
int qpow(int a, int n){
    int ans = 1;
    while(n){
        if(n&1)        //如果n的当前末位为1
            ans *= a;  //ans乘上当前的a
        a *= a;        //a自乘
        n >>= 1;       //n往右移一位
    }
    return ans;
}
```



对一个大素数取模，这是因为计算结果可能会非常巨大，但是在这里考察高精度又没有必要。这时我们的快速幂也应当进行取模，此时应当注意，原则是**步步取模**，如果MOD较大，还应当**开long long**

```cpp
//递归快速幂（对大素数取模）
#define MOD 1000000007
typedef long long ll;
ll qpow(ll a, ll n)
{
    if (n == 0)
        return 1;
    else if (n % 2 == 1)
        return qpow(a, n - 1) * a % MOD;
    else
    {
        ll temp = qpow(a, n / 2) % MOD;
        return temp * temp % MOD;
    }
}
```



#### 8. 数组 寻找缺失和重复的元素

**关键点在于元素和索引是成对儿出现的，常用的方法是排序、异或、映射**。

映射的思路就是我们刚才的分析，将每个**索引和元素映射起来，通过正负号记录某个元素是否被映射**。

排序的方法也很好理解，对于这个问题，可以想象如果元素都被从小到大排序，如果发现索引对应的元素如果不相符，就可以找到重复和缺失的元素。

异或运算也是常用的，因为异或性质 `a ^ a = 0, a ^ 0 = a`，如果将索引和元素同时异或，就可以消除成对儿的索引和元素，留下的就是重复或者缺失的元素。

```java
public int[] findErrorNums(int[] nums) {
        int n = nums.length;
        int dup = -1;
        for(int i = 0;i < n;i++){
            int index = Math.abs(nums[i])-1;
            if(nums[index]<0)
                dup = Math.abs(nums[i]);
            else
                nums[index] *= -1;
        }
        int missing = -1;
        for(int i = 0;i < n;i++)
            if(nums[i]>0)
                missing = i+1;
        return new int[]{dup,missing};
    }
```



#### 9. 水塘抽样算法

##### 1) 选择1个元素

**当你遇到第 `i` 个元素时，应该有 `1/i` 的概率选择该元素，`1 - 1/i` 的概率保持原有的选择**

![image-20211103192612957](C:\Users\QN\AppData\Roaming\Typora\typora-user-images\image-20211103192612957.png)

```java
/* 返回链表中一个随机节点的值 */
int getRandom(ListNode head) {
    Random r = new Random();
    int i = 0, res = 0;
    ListNode p = head;
    // while 循环遍历链表
    while (p != null) {
        i++;
        // 生成一个 [0, i) 之间的整数
        // 这个整数等于 0 的概率就是 1/i
        if (0 == r.nextInt(i)) {
            res = p.val;
        }
        p = p.next;
    }
    return res;
}
```



##### 2 选择k个元素

**如果要随机选择 `k` 个数，只要在第 `i` 个元素处以 `k/i` 的概率选择该元素，以 `1 - k/i` 的概率保持原有选择即可**。

```java
/* 返回链表中 k 个随机节点的值 */
int[] getRandom(ListNode head, int k) {
    Random r = new Random();
    int[] res = new int[k];
    ListNode p = head;

    // 前 k 个元素先默认选上
    for (int j = 0; j < k && p != null; j++) {
        res[j] = p.val;
        p = p.next;
    }
    
    int i = k;
    // while 循环遍历链表
    while (p != null) {
        // 生成一个 [0, i) 之间的整数
        int j = r.nextInt(++i);
        // 这个整数小于 k 的概率就是 k/i
        if (j < k) {
            res[j] = p.val;
        }
        p = p.next;
    }
    return res;

}
```



#### 10. 前缀和数组

**前缀和主要适用的场景是原始数组不会被修改的情况下，频繁查询某个区间的累加和**

使用前缀和技巧，将 `sumRange` 函数的时间复杂度降为 `O(1)`，在prevSum[i]中存放前i个元素和

```java
	private int[] preSum;
    public NumArray(int[] nums) {
        int n = nums.length;
        preSum = new int[n+1];
        for(int i = 0;i < n;i++){
            preSum[i+1] = preSum[i]+nums[i];
        }
    }
    
    public int sumRange(int left, int right) {
        return preSum[right+1]-preSum[left];
    }
```



#### 11. 差分数组

**差分数组的主要适用场景是频繁对原始数组的某个区间的元素进行增减**。

`diff[i]`就是`nums[i]`和`nums[i-1]`之差

如果你想对区间`nums[i..j]`的元素全部加 3，那么只需要让`diff[i] += 3`，然后再让`diff[j+1] -= 3`

```java
int[] diff = new int[nums.length];
// 构造差分数组
diff[0] = nums[0];
for (int i = 1; i < nums.length; i++) {
    diff[i] = nums[i] - nums[i - 1];
}


```



#### 12. 快速选择算法

##### 1）二叉堆实现

注意！！！**Java的Priority是小根堆**

```java
int findKthLargest(int[] nums, int k) {
    // 小顶堆，堆顶是最小元素
    PriorityQueue<Integer> 
        pq = new PriorityQueue<>();
    for (int e : nums) {
        // 每个元素都要过一遍二叉堆
        pq.offer(e);
        // 堆中元素多于 k 个时，删除堆顶元素
        if (pq.size() > k) {
            pq.poll();
        }
    }
    // pq 中剩下的是 nums 中 k 个最大元素，
    // 堆顶是最小的那个，即第 k 个最大元素
    return pq.peek();
}
```



##### 2）快速排序

基础快速排序算法

```java
/* 快速排序主函数 */
void sort(int[] nums) {
    // 一般要在这用洗牌算法将 nums 数组打乱，
    // 以保证较高的效率，我们暂时省略这个细节
    sort(nums, 0, nums.length - 1);
}

/* 快速排序核心逻辑 */
void sort(int[] nums, int lo, int hi) {
    if (lo >= hi) return;
    // 通过交换元素构建分界点索引 p
    int p = partition(nums, lo, hi);
    // 现在 nums[lo..p-1] 都小于 nums[p]，
    // 且 nums[p+1..hi] 都大于 nums[p]
    sort(nums, lo, p - 1);
    sort(nums, p + 1, hi);
}

int partition(int[] nums, int lo, int hi) {
    if (lo == hi) return lo;
    // 将 nums[lo] 作为默认分界点 pivot
    int pivot = nums[lo];
    // j = hi + 1 因为 while 中会先执行 --
    int i = lo, j = hi + 1;
    while (true) {
        // 保证 nums[lo..i] 都小于 pivot
        while (nums[++i] < pivot) {
            if (i == hi) break;
        }
        // 保证 nums[j..hi] 都大于 pivot
        while (nums[--j] > pivot) {
            if (j == lo) break;
        }
        if (i >= j) break;
        // 如果走到这里，一定有：
        // nums[i] > pivot && nums[j] < pivot
        // 所以需要交换 nums[i] 和 nums[j]，
        // 保证 nums[lo..i] < pivot < nums[j..hi]
        swap(nums, i, j);
    }
    // 将 pivot 值交换到正确的位置
    swap(nums, j, lo);
    // 现在 nums[lo..j-1] < nums[j] < nums[j+1..hi]
    return j;
}

// 交换数组中的两个元素
void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```



快速选择

```java
int findKthLargest(int[] nums, int k) {
    int lo = 0, hi = nums.length - 1;
    // 索引转化
    k = nums.length - k;
    while (lo <= hi) {
        // 在 nums[lo..hi] 中选一个分界点
        int p = partition(nums, lo, hi);
        if (p < k) {
            // 第 k 大的元素在 nums[p+1..hi] 中
            lo = p + 1;
        } else if (p > k) {
            // 第 k 大的元素在 nums[lo..p-1] 中
            hi = p - 1;
        } else {
            // 找到第 k 大元素
            return nums[p];
        }
    }
    return -1;
}

// 对数组元素进行随机打乱
void shuffle(int[] nums) {
    int n = nums.length;
    Random rand = new Random();
    for (int i = 0 ; i < n; i++) {
        // 从 i 到最后随机选一个元素
        int r = i + rand.nextInt(n - i);
        swap(nums, i, r);
    }
}
```



#### 13. 计算器实现

```python
class Solution:
    def calculate(self, s: str) -> int:
        def helper(s: List) -> int:
            stack = []
            sign = '+'
            num = 0

            while len(s) > 0:
                c = s.popleft()
                if c.isdigit():
                    num = num*10+int(c)
                # 括号是有递归性质的，相对于调用自身函数来算括号以内的表达式
                if c=='(':
                    num = helper(s)
                if not c.isdigit() and c != ' ' or len(s) == 0:
                    if sign == '+':
                        stack.append(num)
                    elif sign == '-':
                        stack.append(-num)
                    elif sign == '*':
                        stack[-1] = stack[-1]*num
                    elif sign == '/':
                        stack[-1] = int(stack[-1] / float(num))
                    num = 0
                    sign = c
                # 递归的边界 到)就可以返回之前计算的结果了
                if c == ')':
                    break
            return sum(stack)
		# 必须使用collections.deque(双向队列，能在O(1)的时间内完成对队首尾元素的增删)，否则使用list会超时
        return helper(collections.deque(s))
```



#### 14. 完美矩形

将判断转化为 面积 和 顶点 两个维度去判断

1. 首先找到完美矩形的左下角和右上角两个顶点
2. 计算累加面积
3. 得到出现次数为奇数的顶点(存在则删去，不存在则添加)
4. 判断标准：面积是否相同，顶点集是否等于4，且是否是理论顶点坐标

```python
def isRectangleCover(self, rectangles: List[List[int]]) -> bool:
        X1, Y1 = float('inf'), float('inf')
        X2, Y2 = -float('inf'), -float('inf')
        
        points = set()
        actual_area = 0
        for x1, y1, x2, y2 in rectangles:
            # 计算完美矩形的理论顶点坐标
            X1, Y1 = min(X1, x1), min(Y1, y1)
            X2, Y2 = max(X2, x2), max(Y2, y2)
            # 累加小矩形的面积
            actual_area += (x2 - x1) * (y2 - y1)
            # 记录最终形成的图形中的顶点
            p1, p2 = (x1, y1), (x1, y2)
            p3, p4 = (x2, y1), (x2, y2)
            for p in [p1, p2, p3, p4]:
                if p in points: points.remove(p)
                else:           points.add(p)
        # 判断面积是否相同
        expected_area = (X2 - X1) * (Y2 - Y1)
        if actual_area != expected_area:
            return False
        # 判断最终留下的顶点个数是否为 4
        if len(points) != 4:       return False
        # 判断留下的 4 个顶点是否是完美矩形的顶点
        if (X1, Y1) not in points: return False
        if (X1, Y2) not in points: return False
        if (X2, Y1) not in points: return False
        if (X2, Y2) not in points: return False
        # 面积和顶点都对应，说明矩形符合题意
        return True
```



#### 15. 岛屿问题

[200. 岛屿数量 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/number-of-islands/)

岛屿问题：

即用dfs/bfs去遍历二维数组

```java
// 主函数，计算岛屿数量
int numIslands(char[][] grid) {
    int res = 0;
    int m = grid.length, n = grid[0].length;
    // 遍历 grid
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == '1') {
                // 每发现一个岛屿，岛屿数量加一
                res++;
                // 然后使用 DFS 将岛屿淹了
                dfs(grid, i, j);
            }
        }
    }
    return res;
}

// 从 (i, j) 开始，将与之相邻的陆地都变成海水
void dfs(char[][] grid, int i, int j) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        // 超出索引边界
        return;
    }
    if (grid[i][j] == '0') {
        // 已经是海水了
        return;
    }
    // 将 (i, j) 变成海水
    grid[i][j] = '0';
    // 淹没上下左右的陆地
    dfs(grid, i + 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i - 1, j);
    dfs(grid, i, j - 1);
}
```

