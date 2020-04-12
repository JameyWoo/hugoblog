---
title: "Leetcode链表专题"
date: 2020-04-02T10:41:45+08:00
draft: false
categories: ["算法"]
tags: ["链表"]
---



## 合并两个有序链表

[leetcode链接](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

今天重做这道题, 发现之前的代码写的太糟糕了(虽然这次也一般). 

刚好这一段可以用在排序链表那里.

空间复杂度为O(1). 

在这道题上, 我get到的收获是这种方法的空间复杂度为O(1). 而我之前经常写创建一个新的链表, 然后这个链表上又新建一个个值, 导致复杂度为O(n). 但其实现在的O(1)的代码跟O(n)的很像, 区别在于, O(1)的不是新建值然后添加到链表, 而是直接指向旧的链表里的结点. 这样就没有额外的更多开销了.

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int v) {
        val  = v;
        next = NULL;
    }
};

ListNode* vec2list(vector<int> vec) {
    ListNode* head  = new ListNode(0);
    ListNode* start = head;
    for (auto v : vec) {
        head->next = new ListNode(v);
        head       = head->next;
    }
    return start->next;
}

ListNode* mergeList(ListNode *left, ListNode *right) {
    ListNode *head = new ListNode(0);
    ListNode *start = head;
    while (left != NULL && right != NULL) {
        if (left->val <= right->val) {
            head->next = left;
            left = left->next;
        } else {
            head->next = right;
            right = right->next;
        }
        head = head->next;
    }
    if (left == NULL) swap(left, right);
    while (left != NULL) {
        head->next = left;
        left = left->next;
        head = head->next;
    }
    return start->next;
}

int main() {
    vector<int> vec1({1, 3, 7, 9}), vec2({0, 2, 5, 6});
    ListNode *left = vec2list(vec1), *right = vec2list(vec2);
    ListNode *newList = mergeList(left, right);
}
```



## 合并k个有序链表

k个怎么合并? 我使用了优先队列(最小堆)

首先将k个链表的头部压入队列中, 之后每次从队列中pop一个值, 然后将对应的链表的下一个值更新队列. 这样每次队列中弹出的值就是当前最小值. 

维护优先队列, 队列中同时最多有k个值, 因此空间复杂度为O(k), 时间复杂度为O(nlogk)

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<pair<int, int>, vector<pair<int, int> >, greater<pair<int, int> > > k_heap;
        ListNode* res = new ListNode(-1);
        ListNode* p = res;
        for (int i = 0; i < lists.size(); ++i) {
            if (lists[i] != NULL) {
                k_heap.push(make_pair(lists[i]->val, i));
                lists[i] = lists[i]->next;
            }
        }
        while (!k_heap.empty()) {
            pair<int, int> top_node = k_heap.top();
            k_heap.pop();
            p->next = new ListNode(top_node.first);
            p = p->next;
            
            if (lists[top_node.second] != NULL) {
                k_heap.push(make_pair(lists[top_node.second]->val, top_node.second));
                lists[top_node.second] = lists[top_node.second]->next;
            }
        }
        return res->next;
    }
};
```



##  排序链表

[leetcode 链接](https://leetcode-cn.com/problems/sort-list/)

刚开始我写了一版归并排序, 空间复杂度是O(nlogn)的, 也遇到了很多的问题. 总的来说是因为自己总是遗忘一些链表的特性, 所以debug了很久. 

下面是我的一些教训总结

1. 总是把变化了的指针依然当作它的头节点指针, 然而他分明是 head = head->next 了

2. 合并的时候想当然地把 head == NULL 当作条件, 但是其实我是在这个链表里面合并, 终止条件不是NULL, 而是另一半地起始地址

下面有些函数是来辅助debug的

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct List {
    int val;
    List* next;
    List(int v) {
        val  = v;
        next = NULL;
    }
};

void list2vec(List* head, int len) {
    vector<int> debug_vec;
    int i = 0;
    while (i < len) {
        debug_vec.push_back(head->val);
        cout << head->val << " ";
        head = head->next;
        i++;
    }
    cout << endl;
}

List* listSort(List* mylist, int len) {
    if (len == 0) return NULL;
    if (len == 1) return mylist;
    if (mylist == NULL) return NULL;
    if (mylist->next == NULL) return mylist;
    int mid    = len / 2;
    List* head = mylist;
    List *preMylist = mylist, *preHead = head;
    int i      = 0;
    while (i < mid) {
        i++;
        head = head->next;
    }
    mylist = listSort(mylist, mid);
    head = listSort(head, len - mid);
    List* newList = new List(0);
    List* newHead = newList;

    int left = mid, right = len - mid;
    int x = 0, y = 0;

    while (x < left && y < right) {
        if (head->val <= mylist->val) {
            newList->next = new List(head->val);
            newList       = newList->next;
            head          = head->next;
            y += 1;
        } else {
            newList->next = new List(mylist->val);
            newList       = newList->next;
            mylist        = mylist->next;
            x += 1;
        }
    }

    if (x == left && y != right) {
        while (y < right) {
            newList->next = new List(head->val);
            newList       = newList->next;
            head          = head->next;
            y++;
        }
    } else if (x != left && y == right) {
        while (x < left) {
            newList->next = new List(mylist->val);
            newList       = newList->next;
            mylist        = mylist->next;
            x++;
        }
    }

    mylist = newHead->next;
    list2vec(mylist, len);
    return mylist;
}

List* vec2list(vector<int> vec) {
    List* head  = new List(0);
    List* start = head;
    for (auto v : vec) {
        head->next = new List(v);
        head       = head->next;
    }
    return start->next;
}

int main() {
    vector<int> vec({4, 3, 8, 1, 2, 7, 9, 0, 5});
    List* head = vec2list(vec);
    head       = listSort(head, vec.size());
    while (head != NULL) {
        cout << head->val << ' ';
        head = head->next;
    }
    cout << endl;
}
```

又写了一版空间复杂度O(1)的. 这个版本中使用快慢指针法将链表分割为两段. 使用了合并有序链表相同的代码段.

```cpp
/**
 * 空间复杂度O(1)的版本
 */

#include <iostream>
#include <vector>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int v) {
        val  = v;
        next = NULL;
    }
};

void list2vec(ListNode* head) {
    vector<int> debug_vec;
    while (head != NULL) {
        debug_vec.push_back(head->val);
        cout << head->val << " ";
        head = head->next;
    }
    cout << endl;
}

ListNode* mergeList(ListNode* left, ListNode* right) {
    ListNode* head  = new ListNode(0);
    ListNode* start = head;
    while (left != NULL && right != NULL) {
        if (left->val <= right->val) {
            head->next = left;
            left       = left->next;
        } else {
            head->next = right;
            right      = right->next;
        }
        head = head->next;
    }
    if (left == NULL) swap(left, right);
    while (left != NULL) {
        head->next = left;
        left = left->next;
        head = head->next;
    }
    return start->next;
}

ListNode* listSort(ListNode* mylist) {
    if (mylist == NULL) return NULL;
    if (mylist->next == NULL) return mylist;
    // 快慢指针法求中点
    ListNode *slow = mylist, *fast = mylist->next, *last = NULL;
    while (fast != NULL) {
        last = slow;
        slow = slow->next;
        if (fast->next != NULL)
            fast = fast->next->next;
        else
            fast = fast->next;
    }
    // 分割链表
    if (last != NULL) {
        last->next = NULL;
    }
    ListNode *left = mylist, *right = slow;

    left  = listSort(left);
    right = listSort(right);

    list2vec(left);
    list2vec(right);

    ListNode *newList = mergeList(left, right);
    
    return newList;
}

// 快速构造链表的方法
ListNode* vec2list(vector<int> vec) {
    ListNode* head  = new ListNode(0);
    ListNode* start = head;
    for (auto v : vec) {
        head->next = new ListNode(v);
        head       = head->next;
    }
    return start->next;
}

int main() {
    vector<int> vec({4, 3, 8, 1, 2, 7, 9, 0, 5});
    ListNode* head = listSort(vec2list(vec));
    cout << "order:\n";
    while (head != NULL) {
        cout << head->val << ' ';
        head = head->next;
    }
    cout << endl;
}
```



## 分隔链表

[leetcode链接]([分割链表](https://leetcode-cn.com/problems/partition-list-lcci/))

> 编写程序以 x 为基准分割链表，使得所有小于 x 的节点排在大于或等于 x 的节点之前。如果链表中包含 x，x 只需出现在小于 x 的元素之后(如下所示)。分割元素 x 只需处于“右半部分”即可，其不需要被置于左右两部分之间。
>
> ```
> 输入: head = 3->5->8->5->10->2->1, x = 5
> 输出: 3->1->2->10->5->5->8
> ```
>
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/partition-list-lcci
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

这道题虽然是中等题, 但其实很简单. 初看还以为是类似快排的分割, 再看没想到更简单.

**只需要将所有比x小的数移出来, 建立新的链表, 然后再连接原链表**

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
// 只需要将所有比x小的数移出来, 建立新的链表, 然后再连接原链表
    ListNode* partition(ListNode* head, int x) {
        ListNode *start = new ListNode(-1);
        start->next = head;
        ListNode *last = start, *now = head;
        ListNode *less = new ListNode(-1);
        ListNode *headLess = less;
        while (now != NULL) {
            if (now->val >= x) {
                now = now->next;
                last = last->next;
            } else {
                less->next = now;
                now = now->next;
                last->next = now;
                less = less->next;
            }
        }
        less->next = start->next;
        return headLess->next;
    }
};
```



## 倒数第k个结点

> 输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。
>
> 示例：
>
> 给定一个链表: 1->2->3->4->5, 和 k = 2.
>
> 返回链表 4->5.
>
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

这道题还挺有意思, 要想知道倒数第k个结点, 只需要用两个指针, 第一个先走k步, 然后同步走. 慢的那个最后就是倒数第k个结点了.

```cpp
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode *start = new ListNode(0);
        start->next = head;
        ListNode *slow = start, *fast = start;
        for (int i = 0; i < k; i++) {
            fast = fast->next;
        }
        while (fast != NULL) {
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }
};
```





## 环形链表

[leetcode链接](https://leetcode-cn.com/problems/linked-list-cycle/)

这道题只需要判断是否有环, 而不需要判断环入口的地址.

一道非常经典的题目, 双指针的方法比较妙. 有多种解法. 

1. hashTable解法. hashTable的复杂度为O(1)

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
        bool hasCycle(ListNode *head) {
            unordered_map<ListNode*, int> addr;
            while (head != NULL) {
                if (addr[head] == 1) return true;
                addr[head] = 1;
                head = head->next;
            }
            return false;
        }
    };
    ```

2. 双指针(快慢指针)解法: 一个一次走一步, 一个一次走两步, 走到他们的值(指向的地址值)相等了, 那就是有环了.

    ```cpp
    class Solution {
    public:
        bool hasCycle(ListNode *head) {
            ListNode *slow = head, *fast = head;
            while (fast != NULL) {
                slow = slow->next;
                if (fast->next == NULL) return false;
                fast = fast->next->next;
                if (slow == fast) return true;
            }
            return false;
        }
    };
    ```

    

## 环形链表2

[leetcode链接](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

这个环形链表要求找出是第一个结点是环的开始(也就是tail结点指向的结点)

同样的可以用hashTable来做, 就不放代码了.

这道题用双指针法非常妙. 方法就是, 当快慢指针相遇之后, 将快指针放到开头, 然后同时一步一步向前, 第二次相遇点就是他们环的起点了.

[这篇题解很好地证明了原因](https://leetcode-cn.com/problems/linked-list-cycle-ii/solution/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/)

代码

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *fast = head, *slow = head;
        while (fast != NULL) {
            if (fast->next == NULL) return NULL;
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) break;
        }
        if (fast == NULL) return NULL;
        fast = head;
        while (fast != slow) {
            fast = fast->next;
            slow = slow->next;
        }
        return fast;
    }
};
```





## 删除排序链表中的重复元素

[leetcode链接](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/) 

一道简单题, 链表是有序的. 主要考察一些链表的操作.

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode *start = new ListNode(-1);
        start->next = head;
        ListNode *last = start, *now = head;
        while (now != NULL) {
            if (now->next != NULL) {
                if (now->val == now->next->val) {
                    last->next = now->next;
                } else {
                    last = now;
                }
            }
            now = now->next;
        }
        return start->next;
    }
};
```



## 删除排序链表中的重复元素2

[leetcode链接](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/) 

和上一题的区别是, 如果一个数重复出现了, 那么全部删掉, 一个都不保留

只需要加一个`flag`标志, 表示是否是重复的元素. 如果是, 那么在到达下一个新的值的结点的时候删除这个结点.

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode *start = new ListNode(-1);
        start->next = head;
        ListNode *last = start, *now = head;
        bool flag = false;  // 表示重复标志
        while (now != NULL) {
            if (now->next != NULL) {
                if (now->val == now->next->val) {
                    last->next = now->next;
                    flag = true;
                } else {
                    if (flag) last->next = now->next;
                    else last = now;
                    flag = false;
                }
            } else if (flag) {
                last->next = NULL;
            }
            now = now->next;
        }
        return start->next;
    }
};
```



## 奇偶链表

[leetcode链接](https://leetcode-cn.com/problems/odd-even-linked-list/) 

分别建立奇数偶数的头部, 然后指向奇数和偶数索引的结点.

需要注意在偶数结束的时候, 要将它的最后一个结点指向NULL. 否则会形成环.

```cpp
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        ListNode *ji = new ListNode(-1), *ou = new ListNode(-1);
        ListNode *preJi = ji, *preOu = ou;
        int i = 0;
        while (head != NULL) {
            i += 1;
            if (i % 2) {
                ji->next = head;
                ji = ji->next;
            } else {
                ou->next = head;
                ou = ou->next;
            }
            head = head->next;
        }
        ou->next = NULL;
        ji->next = preOu->next;
        return preJi->next;
    }
};
```



## 重排链表

> 给定一个单链表 L：L0→L1→…→Ln-1→Ln ，
> 将其重新排列后变为： L0→Ln→L1→Ln-1→L2→Ln-2→…
>
> 你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。
>
> 示例 1:
>
> 给定链表 1->2->3->4, 重新排列为 1->4->2->3.
> 示例 2:
>
> 给定链表 1->2->3->4->5, 重新排列为 1->5->2->4->3.
>
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/reorder-list
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

把所有链表结点的值存到vector中, 然后两个指针, 一个从头一个从尾, 分别遍历过来.

```cpp
class Solution {
public:
    void reorderList(ListNode* head) {
        if (head == NULL) return;
        ListNode *iter = head;
        vector<ListNode*> ps;
        while (iter != NULL) {
            ps.push_back(iter);
            iter = iter->next;
        }
        debug_list(head);
        int a = 0, b = ps.size() - 1;
        while (a < b) {
            ps[b]->next = ps[a]->next;
            ps[a]->next = ps[b];
            a++;
            b--;
        }
        ps[a]->next = NULL;
        debug_list(head);
    }
};
```

或者也可以将链表分为左右两半, 然后第二半调用反转链表的方法, 之后合并.



## 反转链表

这种题是真的烦, 操作太细节了.

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (head == NULL) return NULL;
        if (head->next == NULL) return head;
        ListNode *toBeZero = head;
        ListNode* start = new ListNode(0);
        start->next = head;
        ListNode* pos = head->next;
        while (pos != NULL) {
            ListNode *tmp = pos->next;
            pos->next = start->next;
            start->next = pos;
            pos = tmp;
        }
        head->next = NULL;
        return start->next;
    }
};
```





## 反转链表2

[leetcode链接

遍历找出在[m, n]位置的链表, 然后拆分成三段, 调用反转链表函数反转这部分, 再拼接起来. 

```cpp
class Solution {
   public:
    ListNode *reverseBetween(ListNode *head, int m, int n) {
        if (head == NULL) return NULL;
        int ind       = 0;
        ListNode *start = new ListNode(0);
        start->next = head;
        ListNode *sub = start, *ano = NULL, *last, *next, *last0;
        while (sub != NULL) {
            if (ind == m - 1) last0 = sub;
            if (ind == m) {
                ano = sub;
            }
            if (ind == n) {
                last = sub;
                next = last->next;
                break;
            }
            ind++;
            sub = sub->next;
        }
        last->next = NULL;
        ListNode *newSub = reverseList(ano);
        last0->next = newSub;
        ano->next = next;
        return start->next;
    }

    // 反转链表
    ListNode *reverseList(ListNode *head) {
        if (head == NULL) return NULL;
        if (head->next == NULL) return head;
        ListNode *toBeZero = head;
        ListNode *start    = new ListNode(0);
        start->next        = head;
        ListNode *pos      = head->next;
        while (pos != NULL) {
            ListNode *tmp = pos->next;
            pos->next     = start->next;
            start->next   = pos;
            pos           = tmp;
        }
        head->next = NULL;
        return start->next;
    }
};
```



## k个一组反转链表

没有按题目要求做到O(1)的空间复杂度, 而是用了vector做辅助.

先将所有结点存起来, 然后分别反转, 再连接起来.

要用到O(1)的空间太细节了啊, debug好久做不出来.

```cpp
class Solution {
   public:
    ListNode *reverseKGroup(ListNode *head, int k) {
        if (head == NULL) return NULL;
        ListNode *pos = head;
        vector<ListNode *> ps, allPs;
        while (pos != NULL) {
            ps.push_back(pos);
            pos = pos->next;
            if (ps.size() == k) {
                int x = 0, y = ps.size() - 1;
                while (x < y) {
                    swap(ps[x], ps[y]);
                    x++, y--;
                }
                allPs.insert(allPs.end(), ps.begin(), ps.end());
                ps.clear();
            }
        }
        allPs.insert(allPs.end(), ps.begin(), ps.end());
        
        ListNode *start = new ListNode(0), *loc = start;
        for (auto p : allPs) {
            loc->next = p;
            loc       = loc->next;
        }
        allPs.back()->next = NULL;
        return start->next;
    }
};
```



## 有序链表转二叉树

> 给定一个单链表，其中的元素按升序排序，将其转换为高度平衡的二叉搜索树。
>
> 本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。
>
> 示例:
>
> 给定的有序链表： [-10, -3, 0, 5, 9],
>
> 一个可能的答案是：[0, -3, 9, -10, null, 5], 它可以表示下面这个高度平衡二叉搜索树：
>
>           0
>          / \
>        -3   9
>        /   /
>      -10  5
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

这道题的重点在于保持二叉搜索树高度平衡.

刚开始没有严格考虑这一条件, 就直接按照模拟中序遍历的方式, 自底向上地建立搜索树.

后来发现这样会断层, 于是引入了count, 没有剩余可扩展节点了就停止扩展, 而不是一直往最深入扩展. 但是还是错了, 因为树不平衡. 之前count是到它的左子树-1, 右子树-1, 考虑到平均分配, 就给左子树cnt/2, 右子树cnt -- 1 - cnt/2. 

**下面是实现的代码**  

```cpp
class Solution {
   public:
	TreeNode *sortedListToBST(ListNode *head) {
		if (head == NULL) return NULL;
		int cnt       = 0;
		ListNode *pos = head;
		while (pos != NULL) {
			cnt += 1;
			pos = pos->next;
		}

		TreeNode *ans = build(head, cnt);
		return ans;
	}

	TreeNode *build(ListNode *&head, int cnt) {
		if (head == NULL) return NULL;
		if (cnt <= 0) return NULL;
		int nowCnt     = cnt;
		int leftCnt = nowCnt / 2;
		int rightCnt = nowCnt - 1 - leftCnt;
		TreeNode *left = build(head, leftCnt);
		TreeNode *root = new TreeNode(head->val);
		head            = head->next;
		TreeNode *right = build(head, rightCnt);
		root->left      = left;
		root->right     = right;
		return root;
	}
};
```

我这个方法在官方题解中属于中序遍历模拟方法. 



