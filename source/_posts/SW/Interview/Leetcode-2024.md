---
layout: post
title: Leetcode主要算法汇总
date: 2024-01-19 16:52:57
index_img: /img/post_pics/index_img/LeetCode.png
sticky: 95
tags: 
    - C/C++
    - 算法
categories: 
    - 面试
---

Leetcode题解汇总。统计到`NO.500`。

<!-- more -->

## 目录

- [目录](#目录)
- [链表](#链表)
  - [19. 删除链表的倒数第N个结点](#19-删除链表的倒数第n个结点)
  - [21. 合并两个有序链表](#21-合并两个有序链表)
  - [23. 合并 K 个升序链表](#23-合并-k-个升序链表)
  - [24. 两两交换链表中的节点](#24-两两交换链表中的节点)
  - [25. K 个一组翻转链表](#25-k-个一组翻转链表)
  - [61. 旋转链表](#61-旋转链表)
  - [82. 删除排序链表中的重复元素 II](#82-删除排序链表中的重复元素-ii)
  - [83. 删除排序链表中的重复元素](#83-删除排序链表中的重复元素)
  - [86. 分隔链表](#86-分隔链表)
  - [92. 反转链表 II](#92-反转链表-ii)
  - [141. 环形链表](#141-环形链表)
  - [142. 环形链表 II](#142-环形链表-ii)
  - [148. 排序链表](#148-排序链表)
  - [206. 反转链表](#206-反转链表)
  - [328. 奇偶链表](#328-奇偶链表)
- [DFS](#dfs)
  - [22. 括号生成](#22-括号生成)
  - [39. 组合总和](#39-组合总和)
  - [40. 组合总和 II](#40-组合总和-ii)
  - [46. 全排列](#46-全排列)
  - [77. 组合](#77-组合)
  - [78. 子集](#78-子集)
  - [130. 被围绕的区域](#130-被围绕的区域)
  - [200. 岛屿数量](#200-岛屿数量)
  - [207. 课程表](#207-课程表)
  - [210. 课程表 II](#210-课程表-ii)
- [动态规划](#动态规划)
  - [5. 最长回文子串](#5-最长回文子串)
  - [32. 最长有效括号](#32-最长有效括号)
  - [45. 跳跃游戏 II](#45-跳跃游戏-ii)
  - [53. 最大子数组和](#53-最大子数组和)
  - [55. 跳跃游戏](#55-跳跃游戏)
  - [62. 不同路径](#62-不同路径)
  - [63. 不同路径 II](#63-不同路径-ii)
  - [64. 最小路径和](#64-最小路径和)
  - [72. 编辑距离](#72-编辑距离)
  - [79. 单词搜索](#79-单词搜索)
  - [97. 交错字符串](#97-交错字符串)
  - [122. 买卖股票的最佳时机 II](#122-买卖股票的最佳时机-ii)
  - [123. 买卖股票的最佳时机 III](#123-买卖股票的最佳时机-iii)
  - [188. 买卖股票的最佳时机 IV](#188-买卖股票的最佳时机-iv)
  - [309. 买卖股票的最佳时机含冷冻期](#309-买卖股票的最佳时机含冷冻期)
  - [714. 买卖股票的最佳时机含手续费](#714-买卖股票的最佳时机含手续费)
  - [198. 打家劫舍](#198-打家劫舍)
  - [213. 打家劫舍 II](#213-打家劫舍-ii)
  - [221. 最大正方形](#221-最大正方形)
  - [279. 完全平方数](#279-完全平方数)
  - [300. 最长递增子序列](#300-最长递增子序列)
- [二叉树](#二叉树)
  - [144. 二叉树的前序遍历](#144-二叉树的前序遍历)
  - [94. 二叉树的中序遍历](#94-二叉树的中序遍历)
  - [145. 二叉树的后序遍历](#145-二叉树的后序遍历)
  - [95. 不同的二叉搜索树 II](#95-不同的二叉搜索树-ii)
  - [96. 不同的二叉搜索树](#96-不同的二叉搜索树)
  - [98. 验证二叉搜索树](#98-验证二叉搜索树)
  - [99. 恢复二叉搜索树](#99-恢复二叉搜索树)
  - [100. 相同的树](#100-相同的树)
  - [101. 对称二叉树](#101-对称二叉树)
  - [102. 二叉树的层序遍历](#102-二叉树的层序遍历)
  - [104. 二叉树的最大深度](#104-二叉树的最大深度)
  - [105. 从前序与中序遍历序列构造二叉树](#105-从前序与中序遍历序列构造二叉树)
  - [106. 从中序与后序遍历序列构造二叉树](#106-从中序与后序遍历序列构造二叉树)
  - [107. 二叉树的层序遍历 II](#107-二叉树的层序遍历-ii)
  - [110. 平衡二叉树](#110-平衡二叉树)
  - [111. 二叉树的最小深度](#111-二叉树的最小深度)
  - [112. 路径总和](#112-路径总和)
  - [113. 路径总和 II](#113-路径总和-ii)
  - [114. 二叉树展开为链表](#114-二叉树展开为链表)
  - [124. 二叉树中的最大路径和](#124-二叉树中的最大路径和)
  - [337. 打家劫舍 III](#337-打家劫舍-iii)
- [字符串](#字符串)
  - [28. 找出字符串中第一个匹配项的下标（KMP）](#28-找出字符串中第一个匹配项的下标kmp)
  - [43. 字符串相乘](#43-字符串相乘)
  - [58. 最后一个单词的长度](#58-最后一个单词的长度)
  - [71. 简化路径](#71-简化路径)
  - [93. 复原 IP 地址](#93-复原-ip-地址)
- [数学](#数学)
  - [50. Pow(x, n)](#50-powx-n)
  - [136. 只出现一次的数字](#136-只出现一次的数字)
  - [172. 阶乘后的零](#172-阶乘后的零)
  - [365. 水壶问题](#365-水壶问题)
  - [461. 汉明距离](#461-汉明距离)
- [数组](#数组)
  - [48. 旋转图像](#48-旋转图像)
  - [54. 螺旋矩阵](#54-螺旋矩阵)
  - [56. 合并区间](#56-合并区间)
  - [66. 加一](#66-加一)
  - [85. 最大矩形](#85-最大矩形)
  - [209. 长度最小的子数组](#209-长度最小的子数组)
- [栈](#栈)
  - [20. 有效的括号](#20-有效的括号)
  - [232. 用栈实现队列](#232-用栈实现队列)
- [双指针](#双指针)
  - [11. 盛最多水的容器](#11-盛最多水的容器)
  - [26. 删除有序数组中的重复项](#26-删除有序数组中的重复项)
  - [27. 移除元素](#27-移除元素)
  - [31. 下一个排列](#31-下一个排列)
  - [42. 接雨水](#42-接雨水)
  - [75. 颜色分类](#75-颜色分类)
  - [80. 删除有序数组中的重复项 II](#80-删除有序数组中的重复项-ii)
  - [88. 合并两个有序数组](#88-合并两个有序数组)
  - [283. 移动零](#283-移动零)
  - [345. 反转字符串中的元音字母](#345-反转字符串中的元音字母)
  - [455. 分发饼干](#455-分发饼干)
- [哈希表](#哈希表)
  - [1. 两数之和](#1-两数之和)
  - [3. 无重复字符的最长子串](#3-无重复字符的最长子串)
  - [76. 最小覆盖子串](#76-最小覆盖子串)
  - [128. 最长连续序列](#128-最长连续序列)
  - [202. 快乐数](#202-快乐数)
- [二分](#二分)
  - [74. 搜索二维矩阵](#74-搜索二维矩阵)
  - [162. 寻找峰值](#162-寻找峰值)
- [贪心](#贪心)
  - [122. 买卖股票的最佳时机 II](#122-买卖股票的最佳时机-ii-1)

## 链表

```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
 };
```

### 19. 删除链表的倒数第N个结点

给你一个链表，删除链表的倒数第`n`个结点，并且返回链表的头结点。

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummy = new ListNode(0, head);
        ListNode* first = head;
        ListNode* second = dummy;
        for (int i = 0; i < n; ++i) {
            first = first->next;
        }
        while (first) {
            first = first->next;
            second = second->next;
        }
        second->next = second->next->next;
        ListNode* ans = dummy->next;
        delete dummy;
        return ans;
    }
};
```

### 21. 合并两个有序链表

将两个升序链表合并为一个新的**升序**链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if(!list1){
            return list2;
        } else if(!list2){
            return list1;
        } else if(list1->val < list2->val){
            list1->next = mergeTwoLists(list1->next, list2);
            return list1;
        } else {
            list2->next = mergeTwoLists(list2->next, list1);
            return list2;
        }
    }
};
```

### 23. 合并 K 个升序链表

给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if(!list1){
            return list2;
        } else if(!list2){
            return list1;
        } else if(list1->val < list2->val){
            list1->next = mergeTwoLists(list1->next, list2);
            return list1;
        } else {
            list2->next = mergeTwoLists(list2->next, list1);
            return list2;
        }
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode* ret = nullptr;
        for (int i = 0; i < lists.size(); i++){
            ret = mergeTwoLists(ret, lists[i]);
        }
        return ret;
    }
};
```

### 24. 两两交换链表中的节点

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }

        ListNode* tmp = head->next;
        head->next = swapPairs(tmp->next);
        tmp->next = head;
        return tmp;
    }
};
```

### 25. K 个一组翻转链表

给你链表的头节点`head`，每`k`个节点一组进行翻转，请你返回修改后的链表。
`k`是一个正整数，它的值小于或等于链表的长度。如果节点总数不是`k`的整数倍，那么请将最后剩余的节点保持原有顺序。

**你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。**

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode *head){
        ListNode* pre;
        ListNode* tmp;
        ListNode* cur = head;
        while(cur){
            tmp = cur->next;
            cur->next = pre;
            pre = cur;
            cur = tmp;
        }
        return pre;
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        // 构造头节点
        ListNode* dummy = new ListNode(-1);
        dummy->next = head;

        ListNode* end = dummy;
        ListNode* pre = dummy;

        int cnt = 0;
        while(end && end->next){
            // end是每k个的最后一个，从每一段的前一个开始向后，即idx[0] - 1
            // pre是每k个的第一个的前一个，更新后就是上一个end
            end = end->next;
            cnt += 1;

            if (cnt == k){
                cnt = 0;

                // next保存下一段的开头
                ListNode *next = end->next;
                // 这一段的开头是start
                ListNode* start = pre->next;
                // 这一段的结尾截断
                end->next = nullptr;

                // 反转后，start变为了最后一个
                pre->next = reverseList(start);
                start->next = next;

                // 更新pre，指向当前反转段的段尾
                pre = start;
                // 重新开始end
                end = pre;
            }
        }

        return dummy->next;
    }
};
```

### 61. 旋转链表

给你一个链表的头节点`head`，旋转链表，将链表每个节点向右移动`k`个位置。

示例 1：

```bash
输入：head = [1,2,3,4,5], k = 2
输出：[4,5,1,2,3]
```

```cpp
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if (k == 0 || head == nullptr || head->next == nullptr) {
            return head;
        }
        int n = 1;
        ListNode* iter = head;
        while (iter->next != nullptr) {
            iter = iter->next;
            n++;
        }
        int add = n - k % n;
        if (add == n) {
            return head;
        }
        iter->next = head;
        while (add--) {
            iter = iter->next;
        }
        ListNode* ret = iter->next;
        iter->next = nullptr;
        return ret;
    }
};
```

### 82. 删除排序链表中的重复元素 II

给定一个已排序的链表的头`head`， 删除原始链表中所有重复数字的节点，只留下不同的数字。返回已排序的链表。

示例 1：

```bash
输入：head = [1,2,3,3,4,4,5]
输出：[1,2,5]
```

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (!head) {
            return head;
        }
        
        ListNode* dummy = new ListNode(0, head);

        ListNode* cur = dummy;
        while (cur->next && cur->next->next) {
            if (cur->next->val == cur->next->next->val) {
                int x = cur->next->val;
                while (cur->next && cur->next->val == x) {
                    cur->next = cur->next->next;
                }
            }
            else {
                cur = cur->next;
            }
        }

        return dummy->next;
    }
};
```

### 83. 删除排序链表中的重复元素

给定一个已排序的链表的头`head`， 删除所有重复的元素，使每个元素只出现一次 。返回已排序的链表。

示例 1：

```bash
输入：head = [1,1,2]
输出：[1,2]
```

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* cur = head;
        while(cur){
            while (cur->next && cur->val == cur->next->val){
                cur->next = cur->next->next;
            }

            cur = cur->next;
        }

        return head;

    }
};
```

### 86. 分隔链表

给你一个链表的头节点`head`和一个特定值`x`，请你对链表进行分隔，使得所有**小于**`x` 的节点都出现在**大于或等于**`x`的节点之前。你应当**保留**两个分区中每个节点的初始相对位置。

示例 1：

```bash
输入：head = [1,4,3,2,5,2], x = 3
输出：[1,2,2,4,3,5]
```

```cpp
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode *smlDummy = new ListNode(0), *bigDummy = new ListNode(0);
        ListNode *sml = smlDummy, *big = bigDummy;
        while (head != nullptr) {
            if (head->val < x) {
                sml->next = head;
                sml = sml->next;
            } else {
                big->next = head;
                big = big->next;
            }
            head = head->next;
        }
        sml->next = bigDummy->next;
        big->next = nullptr;
        return smlDummy->next;
    }
};
```

### 92. 反转链表 II

给你单链表的头指针`head`和两个整数`left`和`right`，其中`left <= right`。请你反转从位置`left`到位置`right`的链表节点，返回反转后的链表。

示例 1：

```bash
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

```cpp
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

### 141. 环形链表

给你一个链表的头节点`head`，判断链表中是否有环。如果链表中有某个节点，可以通过连续跟踪`next`指针再次到达，则链表中存在环。为了表示给定链表中的环，评测系统内部使用整数`pos`来表示链表尾连接到链表中的位置（索引从`0`开始）。注意：`pos`不作为参数进行传递 。仅仅是为了标识链表的实际情况。如果链表中存在环，则返回`true`。否则，返回`false`。

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while(fast && fast->next){
            fast = fast->next->next;
            slow = slow->next;
            if(fast == slow){
                return true;
            }
        }

        return false;
    }
};
```

### 142. 环形链表 II

给定一个链表的头节点`head`，返回链表开始入环的第一个节点。 如果链表无环，则返回`null`。如果链表中有某个节点，可以通过连续跟踪`next`指针再次到达，则链表中存在环。为了表示给定链表中的环，评测系统内部使用整数`pos`来表示链表尾连接到链表中的位置（索引从`0`开始）。如果`pos`是`-1`，则在该链表中没有环。注意：`pos`不作为参数进行传递，仅仅是为了标识链表的实际情况。**不允许修改链表。**

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while (true) {
            if (fast == nullptr || fast->next == nullptr) return nullptr;
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) break;
        }
        fast = head;
        while (slow != fast) {
            slow = slow->next;
            fast = fast->next;
        }
        return fast;
    }
};
```

### 148. 排序链表

给你链表的头结点`head`，请将其按**升序**排列并返回排序后的链表。

```cpp
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        return sortList(head, nullptr);
    }

    ListNode* sortList(ListNode* head, ListNode* tail) {
        if (head == nullptr) {
            return head;
        }
        if (head->next == tail) {
            head->next = nullptr;
            return head;
        }
        ListNode* slow = head, *fast = head;
        while (fast != tail) {
            slow = slow->next;
            fast = fast->next;
            if (fast != tail) {
                fast = fast->next;
            }
        }
        ListNode* mid = slow;
        return merge(sortList(head, mid), sortList(mid, tail));
    }

    ListNode* merge(ListNode* head1, ListNode* head2) {
        ListNode* dummyHead = new ListNode(0);
        ListNode* temp = dummyHead, *temp1 = head1, *temp2 = head2;
        while (temp1 != nullptr && temp2 != nullptr) {
            if (temp1->val <= temp2->val) {
                temp->next = temp1;
                temp1 = temp1->next;
            } else {
                temp->next = temp2;
                temp2 = temp2->next;
            }
            temp = temp->next;
        }
        if (temp1 != nullptr) {
            temp->next = temp1;
        } else if (temp2 != nullptr) {
            temp->next = temp2;
        }
        return dummyHead->next;
    }
};
```

### 206. 反转链表

给你单链表的头节点`head`，请你反转链表，并返回反转后的链表。

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr) {
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```

### 328. 奇偶链表

给定单链表的头节点 `head`，将所有索引为奇数的节点和索引为偶数的节点分别组合在一起，然后返回重新排序的列表。第一个节点的索引被认为是**奇数**， 第二个节点的索引为**偶数**，以此类推。

请注意，偶数组和奇数组内部的相对顺序应该与输入时保持一致。你必须在`O(1)`的额外空间复杂度和`O(n)`的时间复杂度下解决这个问题。

示例 1:

```bash
输入: head = [1,2,3,4,5]
输出: [1,3,5,2,4]
```

```cpp
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if (head == nullptr) {
            return head;
        }
        ListNode* evenHead = head->next;
        ListNode* odd = head;
        ListNode* even = evenHead;
        while (even != nullptr && even->next != nullptr) {
            odd->next = even->next;
            odd = odd->next;
            even->next = odd->next;
            even = even->next;
        }
        odd->next = evenHead;
        return head;
    }
};
```

## DFS

### 22. 括号生成

数字`n`代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且**有效的**括号组合。

```cpp
class Solution {
public:
    vector<string> res;
    void dfs(const string& str, int left, int right){
        if(left < 0 || left > right){
            return;
        }
        if(left == 0 && right == 0){
            res.push_back(str);
            return;
        }
        dfs(str + '(', left - 1, right);
        dfs(str + ')', left, right - 1);
    }
    vector<string> generateParenthesis(int n) {
        dfs("", n, n);
        return res;
    }
};
```

### 39. 组合总和

给你一个**无重复元素**的整数数组`candidates`和一个目标整数 `target`，找出`candidates`中可以使数字和为目标数`target`的**所有**不同组合，并以列表形式返回。你可以按**任意顺序**返回这些组合。`candidates`中的**同一个**数字可以无限制重复被选取。如果至少一个数字的被选数量不同，则两种组合是不同的。对于给定的输入，保证和为`target`的不同组合数少于`150`个。

示例 1：

```bash
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]
解释：
2 和 3 可以形成一组候选，2 + 2 + 3 = 7。注意 2 可以使用多次。
7 也是一个候选， 7 = 7。
仅有这两种组合。
```

```cpp
class Solution {
public:
    void dfs(vector<int>& candidates, int target, vector<vector<int>>& ans, vector<int>& combine, int idx){
        if(idx == candidates.size()){
            return;
        }
        if(target == 0){
            ans.push_back(combine);
            return;
        }
        dfs(candidates, target, ans, combine, idx+1);

        if(target - candidates[idx] >= 0){
            combine.push_back(candidates[idx]);
            dfs(candidates, target - candidates[idx], ans, combine, idx);
            combine.pop_back();
        }
    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> ans;
        vector<int> combine;
        dfs(candidates, target, ans, combine, 0);
        return ans;
    }
};
```

### 40. 组合总和 II

给定一个候选人编号的集合`candidates`和一个目标数`target`，找出`candidates`中所有可以使数字和为`target`的组合。`candidates`中的每个数字在每个组合中只能使用**一次**。
注意：解集不能包含重复的组合。

```bash
示例 1:

输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出:
[
    [1,1,6],
    [1,2,5],
    [1,7],
    [2,6]
]
```

```cpp
class Solution {
private:
    vector<pair<int, int>> freq;
    vector<vector<int>> ans;
    vector<int> sequence;

public:
    void dfs(int pos, int rest) {
        if (rest == 0) {
            ans.push_back(sequence);
            return;
        }
        if (pos == freq.size() || rest < freq[pos].first) {
            return;
        }

        dfs(pos + 1, rest);

        int most = min(rest / freq[pos].first, freq[pos].second);
        for (int i = 1; i <= most; ++i) {
            sequence.push_back(freq[pos].first);
            dfs(pos + 1, rest - i * freq[pos].first);
        }
        for (int i = 1; i <= most; ++i) {
            sequence.pop_back();
        }
    }

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        for (int num: candidates) {
            if (freq.empty() || num != freq.back().first) {
                freq.emplace_back(num, 1);
            } else {
                ++freq.back().second;
            }
        }
        dfs(0, target);
        return ans;
    }
};

```

### 46. 全排列

给定一个不含重复数字的数组`nums`，返回其**所有可能的全排列**。你可以**按任意顺序**返回答案。

示例 1：

```bash
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```cpp
class Solution {
public:
    void dfs(vector<vector<int>>& res, vector<int>& nums, int idx, int len){
        if (idx == len){
            res.push_back(nums);
            return;
        }

        for (int i = idx; i < len; i++){
            swap(nums[i], nums[idx]);
            dfs(res, nums, idx + 1, len);
            swap(nums[i], nums[idx]);
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        dfs(res, nums, 0, int(nums.size()));

        return res;

    }
};
```

### 77. 组合

给定两个整数`n`和`k`，返回范围`[1, n]`中所有可能的`k`个数的组合。你可以按**任何顺序**返回答案。

示例 1：

```bash
输入：n = 4, k = 2
输出：
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

```cpp
class Solution {
public:
    vector<int> temp;
    vector<vector<int>> ans;

    void dfs(int cur, int n, int k) {
        // 剪枝：temp 长度加上区间 [cur, n] 的长度小于 k，不可能构造出长度为 k 的 temp
        if (temp.size() + (n - cur + 1) < k) {
            return;
        }
        // 记录合法的答案
        if (temp.size() == k) {
            ans.push_back(temp);
            return;
        }
        // 考虑选择当前位置
        temp.push_back(cur);
        dfs(cur + 1, n, k);
        temp.pop_back();
        // 考虑不选择当前位置
        dfs(cur + 1, n, k);
    }

    vector<vector<int>> combine(int n, int k) {
        dfs(1, n, k);
        return ans;
    }
};
```

### 78. 子集

给你一个整数数组`nums`，数组中的元素**互不相同**。返回该数组所有可能的子集（幂集）。解集**不能**包含重复的子集。你可以按**任意顺序**返回解集。

示例 1：

```bash
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

```cpp
class Solution {
public:
    vector<int> t;
    vector<vector<int>> ans;

    void dfs(int cur, vector<int>& nums) {
        if (cur == nums.size()) {
            ans.push_back(t);
            return;
        }
        t.push_back(nums[cur]);
        dfs(cur + 1, nums);
        t.pop_back();
        dfs(cur + 1, nums);
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        dfs(0, nums);
        return ans;
    }
};
```

位运算：

```cpp
class Solution {
public:
    vector<int> t;
    vector<vector<int>> ans;

    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        for (int mask = 0; mask < (1 << n); ++mask) {
            t.clear();
            for (int i = 0; i < n; ++i) {
                if (mask & (1 << i)) {
                    t.push_back(nums[i]);
                }
            }
            ans.push_back(t);
        }
        return ans;
    }
};
```

### 130. 被围绕的区域

给你一个`m x n`的矩阵`board`，由若干字符`'X'`和`'O'`，找到所有被`'X'`围绕的区域，并将这些区域里所有的`'O'`用`'X'`填充。

示例 1：

```bash
输入：board = [["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]]
输出：[["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]]
解释：被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。
```

```cpp
class Solution {
public:
    int n, m;

    void dfs(vector<vector<char>>& board, int x, int y) {
        if (x < 0 || x >= n || y < 0 || y >= m || board[x][y] != 'O') {
            return;
        }
        board[x][y] = 'A';
        dfs(board, x + 1, y);
        dfs(board, x - 1, y);
        dfs(board, x, y + 1);
        dfs(board, x, y - 1);
    }

    void solve(vector<vector<char>>& board) {
        n = board.size();
        if (n == 0) {
            return;
        }
        m = board[0].size();
        for (int i = 0; i < n; i++) {
            dfs(board, i, 0);
            dfs(board, i, m - 1);
        }
        for (int i = 1; i < m - 1; i++) {
            dfs(board, 0, i);
            dfs(board, n - 1, i);
        }
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (board[i][j] == 'A') {
                    board[i][j] = 'O';
                } else if (board[i][j] == 'O') {
                    board[i][j] = 'X';
                }
            }
        }
    }
};
```

### 200. 岛屿数量

给你一个由`'1'`（陆地和`'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。此外，你可以假设该网格的四条边均被水包围。

示例 1：

```bash
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1
```

```cpp
class Solution {
private:
    void dfs(vector<vector<char>>& grid, int r, int c) {
        int nr = grid.size();
        int nc = grid[0].size();

        grid[r][c] = '0';
        if (r - 1 >= 0 && grid[r-1][c] == '1') dfs(grid, r - 1, c);
        if (r + 1 < nr && grid[r+1][c] == '1') dfs(grid, r + 1, c);
        if (c - 1 >= 0 && grid[r][c-1] == '1') dfs(grid, r, c - 1);
        if (c + 1 < nc && grid[r][c+1] == '1') dfs(grid, r, c + 1);
    }

public:
    int numIslands(vector<vector<char>>& grid) {
        int nr = grid.size();
        if (!nr) return 0;
        int nc = grid[0].size();

        int num_islands = 0;
        for (int r = 0; r < nr; ++r) {
            for (int c = 0; c < nc; ++c) {
                if (grid[r][c] == '1') {
                    ++num_islands;
                    dfs(grid, r, c);
                }
            }
        }

        return num_islands;
    }
};
```

### 207. 课程表

你这个学期必须选修`numCourses`门课程，记为`0`到`numCourses - 1`。在选修某些课程之前需要一些先修课程。先修课程按数组`prerequisites`给出，其中`prerequisites[i] = [ai, bi]`，表示如果要学习课程`ai`则**必须**先学习课程`bi`。

例如，先修课程对`[0, 1]`表示：想要学习课程`0`，你需要先完成课程`1`。
请你判断是否可能完成所有课程的学习？如果可以，返回`true`；否则，返回`false`。

示例 1：

```bash
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。
```

```cpp
class Solution {
public:
    bool dfs(vector<vector<int>>& edges, vector<int>& visited, int idx){
        visited[idx] = 1;
        bool valid;

        // 遍历一个点的所有边
        for (int i = 0; i < edges[idx].size(); i++){
            if (visited[edges[idx][i]] == 0){
                // 如果边的另一个端点没有访问，则继续
                valid = dfs(edges, visited, edges[idx][i]);

                // 向下递归后如果遇到环，则立即返回
                if(valid == false){
                    return false;
                }
            } else if (visited[edges[idx][i]] == 1){
                // 如果访问了，则说明有环
                return false;
            }
        }

        // 标记这个点对应的所有边已结束，下次遇到这个点直接跳过
        visited[idx] = 2;
        return true;
    }
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> edges(numCourses);
        vector<int> visited(numCourses, 0);
        bool valid = true;

        // 生成边
        for (int i = 0; i < prerequisites.size(); i++){
            edges[prerequisites[i][0]].push_back(prerequisites[i][1]);
        }

        for (int i = 0; i < numCourses; i++){
            if(visited[i] == 0){
                valid = dfs(edges, visited, i);
                if(valid == false){
                    break;
                }
            }
        }

        return valid;
    }
};
```

### 210. 课程表 II

现在你总共有 numCourses 门课需要选，记为 0 到 numCourses - 1。给你一个数组 prerequisites ，其中 prerequisites[i] = [ai, bi] ，表示在选修课程 ai 前 必须 先选修 bi 。

例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示：[0,1] 。
返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回 任意一种 就可以了。如果不可能完成所有课程，返回 一个空数组 。

示例 1：

```bash
输入：numCourses = 2, prerequisites = [[1,0]]
输出：[0,1]
解释：总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1] 。
```

示例 2：

```bash
输入：numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
输出：[0,2,1,3]
解释：总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
因此，一个正确的课程顺序是 [0,1,2,3] 。另一个正确的排序是 [0,2,1,3] 。
```

```cpp
class Solution {
private:
    // 存储有向图
    vector<vector<int>> edges;
    // 标记每个节点的状态：0=未搜索，1=搜索中，2=已完成
    vector<int> visited;
    // 用数组来模拟栈，下标 0 为栈底，n-1 为栈顶
    vector<int> result;
    // 判断有向图中是否有环
    bool valid = true;

public:
    void dfs(int u) {
        // 将节点标记为「搜索中」
        visited[u] = 1;
        // 搜索其相邻节点
        // 只要发现有环，立刻停止搜索
        for (int v: edges[u]) {
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
        result.push_back(u);
    }

    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        edges.resize(numCourses);
        visited.resize(numCourses);
        for (const auto& info: prerequisites) {
            edges[info[1]].push_back(info[0]);
        }
        // 每次挑选一个「未搜索」的节点，开始进行深度优先搜索
        for (int i = 0; i < numCourses && valid; ++i) {
            if (!visited[i]) {
                dfs(i);
            }
        }
        if (!valid) {
            return {};
        }
        // 如果没有环，那么就有拓扑排序
        // 注意下标 0 为栈底，因此需要将数组反序输出
        reverse(result.begin(), result.end());
        return result;
    }
};
```

## 动态规划

### 5. 最长回文子串

给你一个字符串 s，找到 s 中最长的回文子串。 如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。
示例 1：

```bash
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        if(n < 2){
            return s;
        }

        int max_len = 1;
        int idx = 0;
        vector<vector<int>> dp(n, vector<int>(n));

        for(int i = 0; i < n; i++){
            dp[i][i] = 1;
        }

        for(int L = 2; L <=n; L++){
            for(int i = 0; i < n; i++){
                int j = L + i -1;
                if(j >= n){
                    break;
                }

                if(s[i] != s[j]){
                    dp[i][j] = 0;
                } else {
                    if (j - i < 3){
                        dp[i][j] = 1;
                    } else {
                        dp[i][j] = dp[i+1][j-1];
                    }
                }

                if(dp[i][j] == 1 && L > max_len){
                    max_len = L;
                    idx = i;
                }
            }
        }

        return s.substr(idx, max_len);
    }
};
```

### 32. 最长有效括号

给你一个只包含 '(' 和 ')' 的字符串，找出最长有效（格式正确且连续）括号子串的长度。
示例 1：

```bash
输入：s = "(()"
输出：2
解释：最长有效括号子串是 "()"
```

```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        int maxans = 0, n = s.length();
        vector<int> dp(n, 0);
        for (int i = 1; i < n; i++) {
            if (s[i] == ')') {
                if (s[i - 1] == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s[i - dp[i - 1] - 1] == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = max(maxans, dp[i]);
            }
        }
        return maxans;
    }
};
```

### 45. 跳跃游戏 II

给定一个长度为 n 的 0 索引整数数组 nums。初始位置为 nums[0]。

每个元素 nums[i] 表示从索引 i 向前跳转的最大长度。换句话说，如果你在 nums[i] 处，你可以跳转到任意 nums[i + j] 处:

0 <= j <= nums[i]
i + j < n
返回到达 nums[n - 1] 的最小跳跃次数。生成的测试用例可以到达 nums[n - 1]。

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();

        if (n == 1){
            return 0;
        }
        vector<int> dp(n, INT_MAX);

        dp[0] = 0;
        for(int i = 0; i < n; i++){
            for (int j = 1; j <= nums[i] && i + j < n; j++){
                dp[i + j] = min(dp[i+j], dp[i] + 1);
            }
        }
        return dp[n-1];
    }
};
```

### 53. 最大子数组和

给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
子数组 是数组中的一个连续部分。

示例 1：

```bash
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6。
```

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        int ret;
        vector<int> dp(n);

        if (n == 1){
            return nums[0];
        }

        dp[0] = nums[0];
        ret = dp[0];
        for (int i = 1; i < n; i++){
            dp[i] = max(dp[i-1] + nums[i], nums[i]);
            ret = max(ret, dp[i]);
        }

        return ret;
    }
};
```

### 55. 跳跃游戏

给你一个非负整数数组 nums ，你最初位于数组的 第一个下标。数组中的每个元素代表你在该位置可以跳跃的最大长度。
判断你是否能够到达最后一个下标，如果可以，返回 true ；否则，返回 false。

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int n = nums.size();
        int max_jump = 0;

        if (n == 1){
            return true;
        }
        for (int i = 0; i < n - 1; i++){
            if (i <= max_jump){
                max_jump = max(max_jump, i + nums[i]);
            }
        }
        return max_jump >= n-1;
    }
};
```

### 62. 不同路径

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
问总共有多少条不同的路径？

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> dp(n);
        dp[0] = 1;
        for (int j = 0; j < m; j ++){
            for (int i = 0; i < n ; i++){
                if (j == 0 && i > 0){
                    dp[i] = 1;
                }
                if (i == 0 && j > 1){
                    dp[i] = 1;
                }
                if(i > 0 && j > 0){
                    dp[i] = dp[i] + dp[i-1];
                }
            }
        }
        return dp[n-1];
    }
};
```

### 63. 不同路径 II

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish”）。
现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？
网格中的障碍物和空位置分别用 1 和 0 来表示。

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int n = obstacleGrid.size(), m = obstacleGrid.at(0).size();
        vector <int> f(m);

        f[0] = (obstacleGrid[0][0] == 0);
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                if (obstacleGrid[i][j] == 1) {
                    f[j] = 0;
                    continue;
                }
                if (j - 1 >= 0 && obstacleGrid[i][j - 1] == 0) {
                    f[j] += f[j - 1];
                }
            }
        }

        return f.back();
    }
};
```

### 64. 最小路径和

给定一个包含非负整数的 m x n 网格 grid ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
说明：每次只能向下或者向右移动一步。

示例 1：

```bash
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。
```

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<int> dp(n);
        for (int i = 0; i < m; i++){
            for (int j = 0; j < n; j++){
                if (i == 0){
                    if(j == 0){
                        dp[j] = grid[0][0];
                    } else{
                        dp[j] = dp[j-1] + grid[i][j];
                    }
                } else {
                    if (j == 0){
                        dp[j] += grid[i][0];
                    } else {
                        dp[j] = min(dp[j], dp[j-1]) + grid[i][j];
                    }
                }
            }
        }
        return dp[n-1];
    }
};
```

### 72. 编辑距离

给你两个单词 word1 和 word2， 请返回将 word1 转换成 word2 所使用的最少操作数。

你可以对一个单词进行如下三种操作：

插入一个字符
删除一个字符
替换一个字符

示例 1：

```bash
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

示例 2：

```bash
输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.length(), m = word2.length();

        if (m * n == 0){
            return m + n;
        }

        vector<vector<int>> dp(n+1, vector<int>(m+1));
        for(int i = 0; i <= n; i++){
            dp[i][0] = i;
        }
        for(int j = 0; j <= m; j++){
            dp[0][j] = j;
        }
        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= m; j++){
                int ans1 = dp[i][j-1] + 1;
                int ans2 = dp[i-1][j] + 1;
                int ans3 = dp[i-1][j-1];
                if (word1[i-1] != word2[j-1]){
                    ans3 +=1;
                }
                dp[i][j] = min(ans1, min(ans2, ans3));
            }
        }
        return dp[n][m];
    }
};
```

### 79. 单词搜索

给定一个 m x n 二维字符网格 board 和一个字符串单词 word。如果 word 存在于网格中，返回 true ；否则，返回 false。
单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

示例 1：

```bash
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

```cpp
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        rows = board.size();
        cols = board[0].size();
        for(int i = 0; i < rows; i++) {
            for(int j = 0; j < cols; j++) {
                if (dfs(board, word, i, j, 0)) return true;
            }
        }
        return false;
    }
private:
    int rows, cols;
    bool dfs(vector<vector<char>>& board, string word, int i, int j, int k) {
        if (i >= rows || i < 0 || j >= cols || j < 0 || board[i][j] != word[k]) return false;
        if (k == word.size() - 1) return true;
        board[i][j] = '\0';
        bool res = dfs(board, word, i + 1, j, k + 1) || dfs(board, word, i - 1, j, k + 1) || 
                      dfs(board, word, i, j + 1, k + 1) || dfs(board, word, i , j - 1, k + 1);
        board[i][j] = word[k];
        return res;
    }
};
```

### 97. 交错字符串

给定三个字符串 s1、s2、s3，请你帮忙验证 s3 是否是由 s1 和 s2 交错 组成的。
两个字符串 s 和 t 交错 的定义与过程如下，其中每个字符串都会被分割成若干 非空 子字符串：

```bash
s = s1 + s2 + ... + sn
t = t1 + t2 + ... + tm
|n - m| <= 1
交错 是 s1 + t1 + s2 + t2 + s3 + t3 + ... 或者 t1 + s1 + t2 + s2 + t3 + s3 + ...
注意：a + b 意味着字符串 a 和 b 连接。
```

示例 1：

```bash
输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
输出：true
```

```cpp
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        auto f = vector < vector <int> > (s1.size() + 1, vector <int> (s2.size() + 1, false));

        int n = s1.size(), m = s2.size(), t = s3.size();

        if (n + m != t) {
            return false;
        }

        f[0][0] = true;
        for (int i = 0; i <= n; ++i) {
            for (int j = 0; j <= m; ++j) {
                int p = i + j - 1;
                if (i > 0) {
                    f[i][j] |= (f[i - 1][j] && s1[i - 1] == s3[p]);
                }
                if (j > 0) {
                    f[i][j] |= (f[i][j - 1] && s2[j - 1] == s3[p]);
                }
            }
        }

        return f[n][m];
    }
};
```

### 122. 买卖股票的最佳时机 II

给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。
在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。
返回 你能获得的 最大 利润 。

示例 1：

```bash
输入：prices = [7,1,5,3,6,4]
输出：7
解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
     总利润为 4 + 3 = 7 。
```

定义状态`dp[i][0]`表示第`i`天交易完后手里没有股票的最大利润，`dp[i][1]`表示第`i`天交易完后手里持有一支股票的最大利润（`i`从`0`开始）。

考虑`dp[i][0]`的转移方程，如果这一天交易完后手里没有股票，那么可能的转移状态为前一天已经没有股票，即`dp[i−1][0]`，或者前一天结束的时候手里持有一支股票，即`dp[i−1][1]`，这时候我们要将其卖出，并获得`prices[i]`的收益。因此为了收益最大化，我们列出如下的转移方程：

```bash
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
```

再来考虑`dp[i][1]`，按照同样的方式考虑转移状态，那么可能的转移状态为前一天已经持有一支股票，即`dp[i−1][1]`，或者前一天结束时还没有股票，即`dp[i−1][0]`，这时候我们要将其买入，并减少`prices[i]`的收益。可以列出如下的转移方程：

```bash
dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int dp[n][2];
        dp[0][0] = 0, dp[0][1] = -prices[0];
        for (int i = 1; i < n; ++i) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```

### 123. 买卖股票的最佳时机 III

给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:

```bash
输入：prices = [3,3,5,0,0,3,1,4]
输出：6
解释：在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int buy1 = -prices[0], sell1 = 0;
        int buy2 = -prices[0], sell2 = 0;
        for (int i = 1; i < n; ++i) {
            buy1 = max(buy1, -prices[i]);
            sell1 = max(sell1, buy1 + prices[i]);
            buy2 = max(buy2, sell1 - prices[i]);
            sell2 = max(sell2, buy2 + prices[i]);
        }
        return sell2;
    }
};
```

### 188. 买卖股票的最佳时机 IV

给你一个整数数组 prices 和一个整数 k ，其中 prices[i] 是某支给定的股票在第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。也就是说，你最多可以买 k 次，卖 k 次。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1：

```bash
输入：k = 2, prices = [2,4,1]
输出：2
解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
```

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        k = min<int>(k, prices.size() / 2) + 1;
        vector buy(k, INT_MIN), sel(k, 0);
        for (int i : prices) {
            for (int j = 1; j < k; j++) {
                buy[j] = max(buy[j], sel[j - 1] - i);
                sel[j] = max(sel[j], buy[j] + i);
            }
        }
        return sel.back();
    }
};
```

### 309. 买卖股票的最佳时机含冷冻期

给定一个整数数组prices，其中第  prices[i] 表示第 i 天的股票价格 。​
设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:

```bash
输入: prices = [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int n = prices.size();
        // f[i][0]: 手上持有股票的最大收益
        // f[i][1]: 手上不持有股票，并且处于冷冻期中的累计最大收益
        // f[i][2]: 手上不持有股票，并且不在冷冻期中的累计最大收益
        vector<vector<int>> f(n, vector<int>(3));
        f[0][0] = -prices[0];
        for (int i = 1; i < n; ++i) {
            f[i][0] = max(f[i - 1][0], f[i - 1][2] - prices[i]);
            f[i][1] = f[i - 1][0] + prices[i];
            f[i][2] = max(f[i - 1][1], f[i - 1][2]);
        }
        return max(f[n - 1][1], f[n - 1][2]);
    }
};
```

### 714. 买卖股票的最佳时机含手续费

给定一个整数数组 prices，其中 prices[i]表示第 i 天的股票价格 ；整数 fee 代表了交易股票的手续费用。
你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。
注意：这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

示例 1：

```bash
输入：prices = [1, 3, 2, 8, 4, 9], fee = 2
输出：8
解释：能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(2));
        dp[0][0] = 0, dp[0][1] = -prices[0];
        for (int i = 1; i < n; ++i) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i] - fee);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```

### 198. 打家劫舍

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

示例 1：

```bash
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        int ret = 0;
        vector<int> dp(n);

        if(n == 1){
            return nums[0];
        }

        if(n == 2){
            return max(nums[0], nums[1]);
        }
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for(int i = 2; i < n; i++){
            dp[i] = max(dp[i-1], dp[i-2] + nums[i]);
            ret = max(ret, dp[i]);
        }

        return ret;
    }
};
```

### 213. 打家劫舍 II

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

示例 1：

```bash
输入：nums = [2,3,2]
输出：3
解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。
```

```cpp
class Solution {
public:
    int robRange(vector<int>& nums, int start, int end) {
        int first = nums[start], second = max(nums[start], nums[start + 1]);
        for (int i = start + 2; i <= end; i++) {
            int temp = second;
            second = max(first + nums[i], second);
            first = temp;
        }
        return second;
    }

    int rob(vector<int>& nums) {
        int length = nums.size();
        if (length == 1) {
            return nums[0];
        } else if (length == 2) {
            return max(nums[0], nums[1]);
        }
        return max(robRange(nums, 0, length - 2), robRange(nums, 1, length - 1));
    }
};
```

### 221. 最大正方形

在一个由 '0' 和 '1' 组成的二维矩阵内，找到只包含 '1' 的最大正方形，并返回其面积。

示例 1：

```bash
输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
输出：4
```

```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        vector<vector<int>> dp(n, vector<int>(m, 0));
        int ret = 0;
        for (int i = 0; i < n; i++){
            if (matrix[i][0] == '1'){
                dp[i][0] = 1;
                ret = 1;
            }
        }
        for(int i = 0; i < m; i++){
            if (matrix[0][i] == '1'){
                dp[0][i] = 1;
                ret = 1;
            }
        }
        for (int i = 1; i < n; i++){
            for(int j = 1; j < m; j++){
                if(matrix[i][j] == '1'){
                    dp[i][j] = min(min(dp[i-1][j], dp[i-1][j-1]), dp[i][j-1]) + 1;
                    ret = max(ret, dp[i][j]);
                }
            }
        }

        return ret * ret;
    }
};
```

### 279. 完全平方数

给你一个整数 n ，返回 和为 n 的完全平方数的最少数量 。
完全平方数 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，1、4、9 和 16 都是完全平方数，而 3 和 11 不是。

示例 1：

```bash
输入：n = 12
输出：3 
解释：12 = 4 + 4 + 4
```

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n+1);
        for (int i = 1; i <= n ;i++){
            int q = sqrt(i);
            if(q*q == i){
                dp[i] = 1;
            }else{
                dp[i] = i;
                for (int j = 1; j <= q; j++){
                    dp[i] = min(dp[i], dp[j*j] + dp[i - j*j]);
                }
            }
        }
        return dp[n];
    }
};
```

### 300. 最长递增子序列

给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

示例 1：

```bash
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = (int)nums.size();
        if (n == 0) {
            return 0;
        }
        vector<int> dp(n, 0);
        for (int i = 0; i < n; ++i) {
            dp[i] = 1;
            for (int j = 0; j < i; ++j) {
                if (nums[j] < nums[i]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        return *max_element(dp.begin(), dp.end());
    }
};
```

## 二叉树

```cpp
//Definition for a binary tree node.
 struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```

### 144. 二叉树的前序遍历

给你二叉树的根节点 root ，返回它节点值的 前序 遍历

```cpp
class Solution {
public:
    void frontOrder(TreeNode* root, vector<int>& res){
        if (!root){
            return;
        }
        res.push_back(root->val);
        frontOrder(root->left, res);
        frontOrder(root->right, res);
    }
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        frontOrder(root, res);
        return res;
    }
};
```

### 94. 二叉树的中序遍历

给定一个二叉树的根节点 root ，返回 它的 中序 遍历 。

```cpp
class Solution {
public:
    void inorder(TreeNode* root, vector<int>& res){
        if (!root){
            return;
        }
        inorder(root->left, res);
        res.push_back(root->val);
        inorder(root->right, res);
    }
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        inorder(root, res);
        return res;
    }
};
```

### 145. 二叉树的后序遍历

给你一棵二叉树的根节点 root ，返回其节点值的 后序遍历 。

```cpp
class Solution {
public:
    void postOrder(TreeNode* root, vector<int>& res){
        if (!root){
            return;
        }
        postOrder(root->left, res);
        postOrder(root->right, res);
        res.push_back(root->val);
    }
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        postOrder(root, res);
        return res;
    }
};
```

### 95. 不同的二叉搜索树 II

给你一个整数 n ，请你生成并返回所有由 n 个节点组成且节点值从 1 到 n 互不相同的不同 二叉搜索树 。可以按 任意顺序 返回答案。

示例 1：

```bash
输入：n = 3
输出：[[1,null,2,null,3],[1,null,3,2],[2,1,3],[3,1,null,null,2],[3,2,null,1]]
```

```cpp
class Solution {
public:
    vector<TreeNode*> generateTrees(int start, int end) {
        if (start > end) {
            return { nullptr };
        }
        vector<TreeNode*> allTrees;
        // 枚举可行根节点
        for (int i = start; i <= end; i++) {
            // 获得所有可行的左子树集合
            vector<TreeNode*> leftTrees = generateTrees(start, i - 1);

            // 获得所有可行的右子树集合
            vector<TreeNode*> rightTrees = generateTrees(i + 1, end);

            // 从左子树集合中选出一棵左子树，从右子树集合中选出一棵右子树，拼接到根节点上
            for (auto& left : leftTrees) {
                for (auto& right : rightTrees) {
                    TreeNode* currTree = new TreeNode(i);
                    currTree->left = left;
                    currTree->right = right;
                    allTrees.emplace_back(currTree);
                }
            }
        }
        return allTrees;
    }

    vector<TreeNode*> generateTrees(int n) {
        if (!n) {
            return {};
        }
        return generateTrees(1, n);
    }
};
```

### 96. 不同的二叉搜索树

给你一个整数 n ，求恰由 n 个节点组成且节点值从 1 到 n 互不相同的 二叉搜索树 有多少种？返回满足题意的二叉搜索树的种数。

```cpp
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

### 98. 验证二叉搜索树

给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。

有效 二叉搜索树定义如下：

节点的左子树只包含 小于 当前节点的数。
节点的右子树只包含 大于 当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

```cpp
class Solution {
public:
    bool helper(TreeNode* root, long long lower, long long upper) {
        if (root == nullptr) {
            return true;
        }
        if (root -> val <= lower || root -> val >= upper) {
            return false;
        }
        return helper(root -> left, lower, root -> val) && helper(root -> right, root -> val, upper);
    }
    bool isValidBST(TreeNode* root) {
        return helper(root, LONG_MIN, LONG_MAX);
    }
};
```

### 99. 恢复二叉搜索树

给你二叉搜索树的根节点 root ，该树中的 恰好 两个节点的值被错误地交换。请在不改变其结构的情况下，恢复这棵树 。

示例 1：

```bash
输入：root = [1,3,null,null,2]
输出：[3,1,null,null,2]
解释：3 不能是 1 的左孩子，因为 3 > 1 。交换 1 和 3 使二叉搜索树有效。
```

```cpp
class Solution {
public:
    void inorder(TreeNode* root, vector<int>& nums) {
        if (root == nullptr) {
            return;
        }
        inorder(root->left, nums);
        nums.push_back(root->val);
        inorder(root->right, nums);
    }

    pair<int,int> findTwoSwapped(vector<int>& nums) {
        int n = nums.size();
        int index1 = -1, index2 = -1;
        for (int i = 0; i < n - 1; ++i) {
            if (nums[i + 1] < nums[i]) {
                index2 = i + 1;
                if (index1 == -1) {
                    index1 = i;
                } else {
                    break;
                }
            }
        }
        int x = nums[index1], y = nums[index2];
        return {x, y};
    }
    
    void recover(TreeNode* r, int count, int x, int y) {
        if (r != nullptr) {
            if (r->val == x || r->val == y) {
                r->val = r->val == x ? y : x;
                if (--count == 0) {
                    return;
                }
            }
            recover(r->left, count, x, y);
            recover(r->right, count, x, y);
        }
    }

    void recoverTree(TreeNode* root) {
        vector<int> nums;
        inorder(root, nums);
        pair<int,int> swapped= findTwoSwapped(nums);
        recover(root, 2, swapped.first, swapped.second);
    }
};
```

### 100. 相同的树

给你两棵二叉树的根节点 p 和 q ，编写一个函数来检验这两棵树是否相同。
如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        bool res1, res2;
        if(!p && !q){
            return true;
        }
        if((!p && q) || (p && !q)){
            return false;
        } else{
            if (p->val != q->val){
                return false;
            }else{
                res1 = isSameTree(p->left, q->left);
                res2 = isSameTree(p->right, q->right);
            }
        }
        return res1 && res2;
    }
};
```

### 101. 对称二叉树

给你一个二叉树的根节点 root ， 检查它是否轴对称。

```cpp
class Solution {
public:
    bool judge(TreeNode* left, TreeNode* right){
        if (!left && !right){
            return true;
        }else if(left && right){
            return (left->val == right->val) && judge(left->left, right->right) && judge(left->right, right->left);
        }
        return false;
    }
    bool isSymmetric(TreeNode* root) {
        if(!root){
            return false;
        }else{
            return judge(root->left, root->right);
        }
    }
};
```

### 102. 二叉树的层序遍历

给你二叉树的根节点 root ，返回其节点值的 层序遍历 。 （即逐层地，从左到右访问所有节点）。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ret;
        if(!root){
            return ret;
        }

        queue <TreeNode*> q;
        q.push(root);
        while(!q.empty()){
            int cur_size = q.size();
            vector<int> lst;

            for (int i = 1; i <= cur_size; i++){
                TreeNode* node = q.front();
                q.pop();
                lst.push_back(node->val);
                if(node->left){
                    q.push(node->left);
                }

                if(node->right){
                    q.push(node->right);
                }
            }

            ret.push_back(lst);
        }

        return ret;
    }
};
```

### 104. 二叉树的最大深度

给定一个二叉树 root ，返回其最大深度。
二叉树的 最大深度 是指从根节点到最远叶子节点的最长路径上的节点数。

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

### 105. 从前序与中序遍历序列构造二叉树

给定两个整数数组 preorder 和 inorder ，其中 preorder 是二叉树的先序遍历， inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

示例 1:

```bash
输入: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出: [3,9,20,null,null,15,7]
```

```cpp
class Solution {
private:
    unordered_map<int, int> index;

public:
    TreeNode* myBuildTree(const vector<int>& preorder, const vector<int>& inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {
        if (preorder_left > preorder_right) {
            return nullptr;
        }
        
        // 前序遍历中的第一个节点就是根节点
        int preorder_root = preorder_left;
        // 在中序遍历中定位根节点
        int inorder_root = index[preorder[preorder_root]];
        
        // 先把根节点建立出来
        TreeNode* root = new TreeNode(preorder[preorder_root]);
        // 得到左子树中的节点数目
        int size_left_subtree = inorder_root - inorder_left;
        // 递归地构造左子树，并连接到根节点
        // 先序遍历中「从 左边界+1 开始的 size_left_subtree」个元素就对应了中序遍历中「从 左边界 开始到 根节点定位-1」的元素
        root->left = myBuildTree(preorder, inorder, preorder_left + 1, preorder_left + size_left_subtree, inorder_left, inorder_root - 1);
        // 递归地构造右子树，并连接到根节点
        // 先序遍历中「从 左边界+1+左子树节点数目 开始到 右边界」的元素就对应了中序遍历中「从 根节点定位+1 到 右边界」的元素
        root->right = myBuildTree(preorder, inorder, preorder_left + size_left_subtree + 1, preorder_right, inorder_root + 1, inorder_right);
        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = preorder.size();
        // 构造哈希映射，帮助我们快速定位根节点
        for (int i = 0; i < n; ++i) {
            index[inorder[i]] = i;
        }
        return myBuildTree(preorder, inorder, 0, n - 1, 0, n - 1);
    }
};
```

### 106. 从中序与后序遍历序列构造二叉树

给定两个整数数组 inorder 和 postorder ，其中 inorder 是二叉树的中序遍历， postorder 是同一棵树的后序遍历，请你构造并返回这颗 二叉树 。

```cpp
class Solution {
    int post_idx;
    unordered_map<int, int> idx_map;
public:
    TreeNode* helper(int in_left, int in_right, vector<int>& inorder, vector<int>& postorder){
        // 如果这里没有节点构造二叉树了，就结束
        if (in_left > in_right) {
            return nullptr;
        }

        // 选择 post_idx 位置的元素作为当前子树根节点
        int root_val = postorder[post_idx];
        TreeNode* root = new TreeNode(root_val);

        // 根据 root 所在位置分成左右两棵子树
        int index = idx_map[root_val];

        // 下标减一
        post_idx--;
        // 构造右子树
        root->right = helper(index + 1, in_right, inorder, postorder);
        // 构造左子树
        root->left = helper(in_left, index - 1, inorder, postorder);
        return root;
    }
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        // 从后序遍历的最后一个元素开始
        post_idx = (int)postorder.size() - 1;

        // 建立（元素，下标）键值对的哈希表
        int idx = 0;
        for (auto& val : inorder) {
            idx_map[val] = idx++;
        }
        return helper(0, (int)inorder.size() - 1, inorder, postorder);
    }
};
```

### 107. 二叉树的层序遍历 II

给你二叉树的根节点 root ，返回其节点值 自底向上的层序遍历 。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

```cpp
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> ret;
        if(!root){
            return ret;
        }

        queue <TreeNode*> q;
        q.push(root);
        while(!q.empty()){
            int cur_size = q.size();
            vector<int> lst;

            for (int i = 1; i <= cur_size; i++){
                TreeNode* node = q.front();
                q.pop();
                lst.push_back(node->val);
                if(node->left){
                    q.push(node->left);
                }

                if(node->right){
                    q.push(node->right);
                }
            }

            ret.push_back(lst);
        }
        
        reverse(ret.begin(), ret.end());
        return ret;
    }
};
```

### 110. 平衡二叉树

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：
一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1 。

```cpp
class Solution {
public:
    int height(TreeNode* root){
        if(!root){
            return 0;
        }
        int left_height = height(root->left);
        int right_height = height(root->right);

        if(left_height == -1 || right_height == -1 || abs(left_height - right_height) > 1 ){
            return -1;
        }
        return max(left_height, right_height) + 1;
    }
    bool isBalanced(TreeNode* root) {
        return height(root) >= 0;
    }
};
```

### 111. 二叉树的最小深度

给定一个二叉树，找出其最小深度。
最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
说明：叶子节点是指没有子节点的节点。

```cpp
class Solution {
public:
    int minDepth(TreeNode *root) {
        if (root == nullptr) {
            return 0;
        }

        if (root->left == nullptr && root->right == nullptr) {
            return 1;
        }

        int min_depth = INT_MAX;
        if (root->left != nullptr) {
            min_depth = min(minDepth(root->left), min_depth);
        }
        if (root->right != nullptr) {
            min_depth = min(minDepth(root->right), min_depth);
        }

        return min_depth + 1;
    }
};
```

### 112. 路径总和

给你二叉树的根节点 root 和一个表示目标和的整数 targetSum 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 targetSum 。如果存在，返回 true ；否则，返回 false 。
**叶子节点 是指没有子节点的节点。**

示例 1：

```bash
      5
   4     8
 11    13 4
7  2        1
输入：root = [5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22
输出：true
解释：等于目标和的根节点到叶节点路径如上图所示。
```

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode *root, int sum) {
        if (root == nullptr) {
            return false;
        }
        if (root->left == nullptr && root->right == nullptr) {
            return sum == root->val;
        }
        return hasPathSum(root->left, sum - root->val) ||
               hasPathSum(root->right, sum - root->val);
    }
};
```

### 113. 路径总和 II

给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。
**叶子节点 是指没有子节点的节点。**

示例 1：

```bash
      5
   4      8
 11    13   4
7  2       5 1
输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
输出：[[5,4,11,2],[5,8,4,5]]
```

```cpp
class Solution {
public:

    void dfs(TreeNode* root, int targetSum, vector<vector<int>>& res, vector<int>& path){
        if (!root){
            return;
        }

        path.push_back(root->val);

        if (targetSum == root->val && !root->left && !root->right){
            res.push_back(path);
        } else{
            dfs(root->left, targetSum - root->val, res, path);
            dfs(root->right, targetSum - root->val, res, path);
        }

        path.pop_back();
    }
    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        vector<vector<int>> res;
        vector<int> path;
        dfs(root, targetSum, res, path);

        return res;
    }
};
```

### 114. 二叉树展开为链表

给你二叉树的根结点 root ，请你将它展开为一个单链表：

展开后的单链表应该同样使用 TreeNode ，其中 right 子指针指向链表中下一个结点，而左子指针始终为 null 。
展开后的单链表应该与二叉树 先序遍历 顺序相同。

示例 1：

```bash
输入：root = [1,2,5,3,4,null,6]
输出：[1,null,2,null,3,null,4,null,5,null,6]
```

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        while(root){
            if (root->left){
                TreeNode* tmp = root->left;
                while(tmp->right){
                    tmp = tmp->right;
                }

                tmp->right = root->right;
                root->right = root->left;
                root->left = nullptr;
                root = root->right;
            } else {
                root = root->right;
            }
        }
    }
};
```

### 124. 二叉树中的最大路径和

二叉树中的 路径 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点。
路径和 是路径中各节点值的总和。
给你一个二叉树的根节点 root ，返回其 最大路径和 。

```cpp
class Solution {
private:
    int maxSum = INT_MIN;

public:
    int maxGain(TreeNode* node) {
        if (node == nullptr) {
            return 0;
        }
        
        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int leftGain = max(maxGain(node->left), 0);
        int rightGain = max(maxGain(node->right), 0);

        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int priceNewpath = node->val + leftGain + rightGain;

        // 更新答案
        maxSum = max(maxSum, priceNewpath);

        // 返回节点的最大贡献值
        return node->val + max(leftGain, rightGain);
    }

    int maxPathSum(TreeNode* root) {
        maxGain(root);
        return maxSum;
    }
};
```

### 337. 打家劫舍 III

小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为 root 。
除了 root 之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 两个直接相连的房子在同一天晚上被打劫 ，房屋将自动报警。
给定二叉树的 root 。返回 在不触动警报的情况下 ，小偷能够盗取的最高金额 。

示例 1:

```bash
输入: root = [3,2,3,null,3,null,1]
输出: 7 
解释: 小偷一晚能够盗取的最高金额 3 + 3 + 1 = 7
```

```cpp
class Solution {
public:
    map <TreeNode*, int> cur, not_cur;

    void dfs(TreeNode* node){
        if(!node){
            return;
        }

        dfs(node->left);
        dfs(node->right);

        cur[node] = node->val + not_cur[node->left] + not_cur[node->right];
        not_cur[node] = max(cur[node->left], not_cur[node->left]) + max(cur[node->right], not_cur[node->right]);
    }
    int rob(TreeNode* root) {
        dfs(root);
        return max(cur[root], not_cur[root]);

    }
};
```

## 字符串

### 28. 找出字符串中第一个匹配项的下标（KMP）

给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回  -1。

```cpp
class Solution {
public:
    int strStr(string s, string p) {
        int n = s.size(), m = p.size();
        if(m == 0) return 0;
        //设置哨兵
        s.insert(s.begin(),' ');
        p.insert(p.begin(),' ');
        vector<int> next(m + 1);
        //预处理next数组
        for(int i = 2, j = 0; i <= m; i++){
            while(j and p[i] != p[j + 1]) j = next[j];
            if(p[i] == p[j + 1]) j++;
            next[i] = j;
        }
        //匹配过程
        for(int i = 1, j = 0; i <= n; i++){
            while(j and s[i] != p[j + 1]) j = next[j];
            if(s[i] == p[j + 1]) j++;
            if(j == m) return i - m;
        }
        return -1;
    }
};

```

### 43. 字符串相乘

给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。
注意：不能使用任何内置的 BigInteger 库或直接将输入转换为整数。

```cpp
class Solution {
public:
    string multiply(string num1, string num2) {
        if ((num1.size() == 1 && num1[0] == '0') || (num2.size() == 1 && num2[0] == '0')){
            return "0";
        }
        int n = num1.size(), m = num2.size();
        vector<int> ans(n * m + 1, 0);
        string long_num, short_num;

        if (n > m){
            long_num = num1;
            short_num = num2;
        } else {
            long_num = num2;
            short_num = num1;
        }
        n = long_num.size();
        m = short_num.size();
        for (int i = m - 1; i >= 0; i--){
            for (int j = n - 1; j >= 0; j--){
                int cur = (long_num[j] - '0') * (short_num[i] - '0');
                ans[m-1-i + n-1-j] += cur;
            }
        }

        string ret;
        int add = 0;
        for (int i = 0; i < m * n + 1; i++){
            ret += (ans[i] + add) % 10 + '0';
            add = (ans[i] + add) / 10;
        }

        reverse(ret.begin(), ret.end());

        string res;
        for (int i = 0; i < ret.size(); i++){
            if (ret[i] != '0'){
                for (int j = i; j < ret.size(); j++){
                    res += ret[j];
                }
                break;
            }
            
        }
        return res;
    }
};
```

### 58. 最后一个单词的长度

给你一个字符串 s，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中 最后一个 单词的长度。
单词 是指仅由字母组成、不包含任何空格字符的最大子字符串。

```cpp
class Solution {
public:
    int lengthOfLastWord(string s) {
        int n = s.size();
        string ret = "";
        int idx = 0;
        for (int i = n - 1; i >= 0; i--){
            if (s[i] != ' '){
                idx = i;
                break;
            }
        }
        for (int i = idx; i >= 0; i --){
            if (s[i] == ' '){
                break;
            }
            ret += s[i];
        }
        return ret.size();

    }
};
```

### 71. 简化路径

给你一个字符串 path ，表示指向某一文件或目录的 Unix 风格 绝对路径 （以 '/' 开头），请你将其转化为更加简洁的规范路径。

在 Unix 风格的文件系统中，一个点（.）表示当前目录本身；此外，两个点 （..） 表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。任意多个连续的斜杠（即，'//'）都被视为单个斜杠 '/'。 对于此问题，任何其他格式的点（例如，'...'）均被视为文件/目录名称。

请注意，返回的 规范路径 必须遵循下述格式：

始终以斜杠 '/' 开头。
两个目录名之间必须只有一个斜杠 '/'。
最后一个目录名（如果存在）不能 以 '/' 结尾。
此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 '.' 或 '..'）。
返回简化后得到的 规范路径。

示例 1：

```bash
输入：path = "/home/"
输出："/home"
解释：注意，最后一个目录名后面没有斜杠。
```

示例 2：

```bash
输入：path = "/../"
输出："/"
解释：从根目录向上一级是不可行的，因为根目录是你可以到达的最高级。
```

示例 3：

```bash
输入：path = "/home//foo/"
输出："/home/foo"
解释：在规范路径中，多个连续斜杠需要用一个斜杠替换。
```

示例 4：

```bash
输入：path = "/a/./b/../../c/"
输出："/c"
```

```cpp
class Solution {
public:
    string simplifyPath(string path) {
        int n = path.size();
        string ret = "", cur = "";
        vector<string> ans, strs;

        if (n == 1) return "/";

        //截断输入
        for (int i = 1; i < n; i++){
            if (path[i] != '/'){
                cur += path[i];
            } else {
                strs.push_back(cur);
                cur = "";
            }
        }

        //注意最后一个子串
        if (cur.size() > 0) strs.push_back(cur);

        for (int i = 0; i < strs.size(); i++){
            // 处理 '..'
            if (strs[i].size() == 2 && strs[i] == ".."){
                if(ans.size() > 0){
                    ans.pop_back();
                }
            } else if (strs[i] == "" || strs[i] == "."){
                // 连续的'//'会出现空子串，'.'应当忽略
                continue;
            } else {
                ans.push_back(strs[i]);
            }
        }

        // 没有有效的子串则返回'/'
        if(ans.size() == 0) return "/";

        for (int i = 0; i < ans.size(); i++){
            ret += "/" + ans[i];
        }

        return ret;
    }
};
```

### 93. 复原 IP 地址

有效 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 有效 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效 IP 地址。
给定一个只包含数字的字符串 s ，用以表示一个 IP 地址，返回所有可能的有效 IP 地址，这些地址可以通过在 s 中插入 '.' 来形成。你 不能 重新排序或删除 s 中的任何数字。你可以按 任何 顺序返回答案。

示例 1：

```bash
输入：s = "25525511135"
输出：["255.255.11.135","255.255.111.35"]
```

```cpp
class Solution {
public:
    void dfs(string s, vector<string>& res, string& ip, int count){
        int n = s.size();
        if(count == 0){
            if (n > 0){
                if (n == 1 || (n >= 1  && n <= 3 && (s[0] != '0') && stoi(s) <= 255)){
                    ip += s;
                    res.push_back(ip);
                }
            }
            return;
        }
        if (n == 0){
            return;
        }
        
        string next_ip;
        next_ip = ip + s[0] + '.';
        dfs(s.substr(1, n - 1), res, next_ip, count - 1);

        if(s[0] != '0'){
            if (n >= 2){
                next_ip = ip + s.substr(0, 2) + '.';
                dfs(s.substr(2, n - 2), res, next_ip, count - 1);
            }
            if (n >= 3){
                if (stoi(s.substr(0, 3)) <= 255){
                    next_ip = ip + s.substr(0, 3) + '.';
                    dfs(s.substr(3, n - 3), res, next_ip, count - 1);
                }
            }
        }
    }

    vector<string> restoreIpAddresses(string s) {
        vector<string> res;
        string ip;
        dfs(s, res, ip, 3);

        return res;
    }
};
```

## 数学

### 50. Pow(x, n)

实现 pow(x, n) ，即计算 x 的整数 n 次幂函数（即，xn ）。

```cpp
class Solution {
public:
    double quickMul(double x, long long n) {
        double ans = 1.0;
        double tmp = x;
        while(n > 0){
            if (n % 2 == 1){
                ans *= x;
            }

            x *= x;
            n /= 2;
        }
        return ans;
    }
    double myPow(double x, int n) {
        long long N = n;
        return N >= 0 ? quickMul(x, N) : 1.0 / quickMul(x, -N);
    }
};
```

### 136. 只出现一次的数字

给你一个 非空 整数数组 nums ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
你必须设计并实现线性时间复杂度的算法来解决此问题，且该算法只使用常量额外空间。

示例 1 ：

```bash
输入：nums = [2,2,1]
输出：1
```

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ans = nums[0];
        for (int i = 1; i < nums.size(); i++){
            ans ^= nums[i];
        }

        return ans;
    }
};
```

### 172. 阶乘后的零

给定一个整数 n ，返回 n! 结果中尾随零的数量。
提示 `n! = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1`

```cpp
class Solution {
public:
    int trailingZeroes(int n) {
        int ans = 0;
        while (n) {
            n /= 5;
            ans += n;
        }
        return ans;
    }
};
```

### 365. 水壶问题

有两个水壶，容量分别为 jug1Capacity 和 jug2Capacity 升。水的供应是无限的。确定是否有可能使用这两个壶准确得到 targetCapacity 升。
如果可以得到 targetCapacity 升水，最后请用以上水壶中的一或两个来盛放取得的 targetCapacity 升水。

你可以：

装满任意一个水壶
清空任意一个水壶
从一个水壶向另外一个水壶倒水，直到装满或者倒空

示例 1:

```bash
输入: jug1Capacity = 3, jug2Capacity = 5, targetCapacity = 4
输出: true
解释：来自著名的 "Die Hard"
```

```cpp
class Solution {
public:
    bool canMeasureWater(int x, int y, int z) {
        if (x + y < z) {
            return false;
        }
        if (x == 0 || y == 0) {
            return z == 0 || x + y == z;
        }
        return z % gcd(x, y) == 0;
    }
};
```

### 461. 汉明距离

两个整数之间的 汉明距离 指的是这两个数字对应二进制位不同的位置的数目。
给你两个整数 x 和 y，计算并返回它们之间的汉明距离。

示例 1：

```bash
输入：x = 1, y = 4
输出：2
解释：
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑
上面的箭头指出了对应二进制位不同的位置。
```

```cpp
class Solution {
public:
    int hammingDistance(int x, int y) {
        int z = x ^ y;
        int ret = 0;

        while(z > 0){
            if (z % 2 == 1){
                ret += 1;
            }

            z /= 2;
        }

        return ret;
    }
};
```

## 数组

### 48. 旋转图像

给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。
你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < (n + 1) / 2; ++j) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][i];
                matrix[n - j - 1][i] = matrix[n - i - 1][n - j - 1];
                matrix[n - i - 1][n - j - 1] = matrix[j][n - i - 1];
                matrix[j][n - i - 1] = temp;
            }
        }
    }
};
```

### 54. 螺旋矩阵

给你一个 m 行 n 列的矩阵 matrix ，请按照 顺时针螺旋顺序 ，返回矩阵中的所有元素。

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector <int> ans;
        if(matrix.empty()) return ans; //若数组为空，直接返回答案
        int u = 0; //赋值上下左右边界
        int d = matrix.size() - 1;
        int l = 0;
        int r = matrix[0].size() - 1;
        while(true)
        {
            for(int i = l; i <= r; ++i) ans.push_back(matrix[u][i]); //向右移动直到最右
            if(++ u > d) break; //重新设定上边界，若上边界大于下边界，则遍历遍历完成，下同
            for(int i = u; i <= d; ++i) ans.push_back(matrix[i][r]); //向下
            if(-- r < l) break; //重新设定有边界
            for(int i = r; i >= l; --i) ans.push_back(matrix[d][i]); //向左
            if(-- d < u) break; //重新设定下边界
            for(int i = d; i >= u; --i) ans.push_back(matrix[i][l]); //向上
            if(++ l > r) break; //重新设定左边界
        }
        return ans;
    }
};
```

### 56. 合并区间

以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi]。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。

示例 1：

```bash
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() == 0) {
            return {};
        }
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> merged;
        for (int i = 0; i < intervals.size(); ++i) {
            int L = intervals[i][0], R = intervals[i][1];
            if (!merged.size() || merged.back()[1] < L) {
                merged.push_back({L, R});
            }
            else {
                merged.back()[1] = max(merged.back()[1], R);
            }
        }
        return merged;
    }
};
```

### 66. 加一

给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。
最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
你可以假设除了整数 0 之外，这个整数不会以零开头。

示例 1：

```bash
输入：digits = [1,2,3]
输出：[1,2,4]
解释：输入数组表示数字 123。
```

```cpp
class Solution {
public:
    vector<int> plusOne(vector<int>& digits) {
        reverse(digits.begin(), digits.end());
        int n = digits.size();
        int c = 0;
        for (int i = 0; i < n; i ++){
            if (i == 0){
                digits[0] += 1;
            }
            int cur = (digits[i] + c) % 10;
            c = (digits[i] + c)/ 10;
            digits[i] = cur;
        }
        if (c > 0){
            digits.push_back(c);
        }
        reverse(digits.begin(), digits.end());
        return digits;
    }
};
```

### 85. 最大矩形

给定一个仅包含 0 和 1 、大小为 rows x cols 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

示例 1：

```bash
输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
输出：6
解释：最大矩形如上图所示。
```

```cpp
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        int m = matrix.size();
        if (m == 0) {
            return 0;
        }
        int n = matrix[0].size();
        vector<vector<int>> left(m, vector<int>(n, 0));

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == '1') {
                    left[i][j] = (j == 0 ? 0: left[i][j - 1]) + 1;
                }
            }
        }

        int ret = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == '0') {
                    continue;
                }
                int width = left[i][j];
                int area = width;
                for (int k = i - 1; k >= 0; k--) {
                    width = min(width, left[k][j]);
                    area = max(area, (i - k + 1) * width);
                }
                ret = max(ret, area);
            }
        }
        return ret;
    }
};
```

### 209. 长度最小的子数组

给定一个含有 n 个正整数的数组和一个正整数 target 。
找出该数组中满足其总和大于等于 target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

示例 1：

```bash
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();
        int ret = n + 1;
        int sum = 0;
        int left = 0, right = 0;
        while(right < n){
            sum += nums[right];
            while(left <= right && sum >= target){
                ret = min(ret, right - left + 1);
                sum -= nums[left];
                left ++;
            }

            right++;
        }

        if(ret == n + 1){
            return 0;
        }

        return ret;
    }
};
```

## 栈

### 20. 有效的括号

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
每个右括号都有一个对应的相同类型的左括号。

```cpp
class Solution {
public:
    bool isValid(string s) {
        int n = s.size();
        stack<char> st;
        for (int i = 0; i < n; i++){
            if (!st.empty() && ((st.top() == '{' && s[i] == '}') || (st.top() == '(' && s[i] == ')') || (st.top() == '[' && s[i] == ']'))){
                st.pop();
            } else {
                st.push(s[i]);
            }
        }

        return st.empty();
    }
};
```

### 232. 用栈实现队列

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：

实现 MyQueue 类：

void push(int x) 将元素 x 推到队列的末尾
int pop() 从队列的开头移除并返回元素
int peek() 返回队列开头的元素
boolean empty() 如果队列为空，返回 true ；否则，返回 false
说明：

你 只能 使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。

```cpp
class MyQueue {
private:
    stack<int> inStack, outStack;

    void in2out() {
        while (!inStack.empty()) {
            outStack.push(inStack.top());
            inStack.pop();
        }
    }

public:
    MyQueue() {}

    void push(int x) {
        inStack.push(x);
    }

    int pop() {
        if (outStack.empty()) {
            in2out();
        }
        int x = outStack.top();
        outStack.pop();
        return x;
    }

    int peek() {
        if (outStack.empty()) {
            in2out();
        }
        return outStack.top();
    }

    bool empty() {
        return inStack.empty() && outStack.empty();
    }
};
```

## 双指针

### 11. 盛最多水的容器

给定一个长度为 n 的整数数组 height。有 n 条垂线，第 i 条线的两个端点是 (i, 0) 和 (i, height[i])。
找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
返回容器可以储存的最大水量。

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ret = 0;
        int n = height.size();
        int left = 0, right = n - 1;
        while (left < right){
            ret = max(ret, min(height[right], height[left]) * (right - left));
            if (height[right] > height[left]){
                left ++;
            } else {
                right --;
            }
        }
        return ret;
    }
};
```

### 26. 删除有序数组中的重复项

给你一个 非严格递增排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致。然后返回 nums 中唯一元素的个数。

考虑 nums 的唯一元素的数量为 k ，你需要做以下事情确保你的题解可以被通过：

更改数组 nums ，使 nums 的前 k 个元素包含唯一元素，并按照它们最初在 nums 中出现的顺序排列。nums 的其余元素与 nums 的大小不重要。
返回 k。

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n = nums.size();
        if (n == 0) {
            return 0;
        }
        int fast = 1, slow = 1;
        while (fast < n) {
            if (nums[fast] != nums[fast - 1]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
};
```

### 27. 移除元素

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。
不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。
元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int n = nums.size();
        int left = 0;
        for (int right = 0; right < n; right++) {
            if (nums[right] != val) {
                nums[left] = nums[right];
                left++;
            }
        }
        return left;
    }
};
```

### 31. 下一个排列

整数数组的一个 排列  就是将其所有成员以序列或线性顺序排列。

例如，arr = [1,2,3] ，以下这些都可以视作 arr 的排列：[1,2,3]、[1,3,2]、[3,1,2]、[2,3,1]。
整数数组的 下一个排列 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 下一个排列 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

例如，arr = [1,2,3] 的下一个排列是 [1,3,2]。
类似地，arr = [2,3,1] 的下一个排列是 [3,1,2]。
而 arr = [3,2,1] 的下一个排列是 [1,2,3] ，因为 [3,2,1] 不存在一个字典序更大的排列。
给你一个整数数组 nums ，找出 nums 的下一个排列。

必须 原地 修改，只允许使用额外常数空间。

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int i = nums.size() - 2;
        while (i >= 0 && nums[i] >= nums[i+1]){
            i--;
        }

        if (i >= 0){
            int j = nums.size() - 1;
            while(j > i && nums[i] >= nums[j]){
                j--;
            }
            swap(nums[i], nums[j]);
        }
        reverse(nums.begin() + i + 1, nums.end());
    }
};
```

### 42. 接雨水

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

示例 1：

```bash
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）
```

```cpp
class Solution {
public:
    int cal_water(vector<int>& height, int left, int right){
        int ret = min(height[left], height[right]) * (right - left - 1);
        int sum = 0;
        for (int i = left + 1; i <= right - 1; i++){
            sum += height[i];
        }
        return ret - sum;

    }
    int trap(vector<int>& height) {
        int n = height.size();
        int ret = 0;
        int r = 1;
        int max_h = height[0], max_h_idx = 0;

        if (n == 1){
            return 0;
        }

        while(max_h_idx <= r && r < n){
            if (height[r] >= max_h){
                ret += cal_water(height, max_h_idx, r);
                max_h = height[r];
                max_h_idx = r;
            }

            r++;
        }

        max_h = height[n-1];
        max_h_idx = n-1;
        r = n-2;
        while(r <= max_h_idx && r >= 0){
            if (height[r] > max_h){
                ret += cal_water(height, r, max_h_idx);
                max_h = height[r];
                max_h_idx = r;
            }

            r--;
        }

        return ret;
    }
};
```

### 75. 颜色分类

给定一个包含红色、白色和蓝色、共 n 个元素的数组 nums ，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。
我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。
必须在不使用库内置的 sort 函数的情况下解决这个问题。

示例 1：

```bash
输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]
```

```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int n = nums.size();

        int red = 0, blue = n - 1;
        for (int i = 0; i < n; i ++){
            if (nums[i] == 0){
                swap(nums[red], nums[i]);
                red ++;
            }
        }

        for (int i = n - 1; i >= 0; i --){
            if (nums[i] == 2){
                swap(nums[blue], nums[i]);
                blue --;
            }
        }
    }
};
```

### 80. 删除有序数组中的重复项 II

给你一个有序数组 nums ，请你 原地 删除重复出现的元素，使得出现次数超过两次的元素只出现两次 ，返回删除后数组的新长度。
不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n = nums.size();
        if (n <= 2) {
            return n;
        }
        int slow = 2, fast = 2;
        while (fast < n) {
            if (nums[slow - 2] != nums[fast]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
};
```

### 88. 合并两个有序数组

给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。
请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。
注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。

示例 1：

```bash
输入：nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
输出：[1,2,2,3,5,6]
解释：需要合并 [1,2,3] 和 [2,5,6] 。
合并结果是 [1,2,2,3,5,6] ，其中斜体加粗标注的为 nums1 中的元素。
```

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int p1 = m - 1, p2 = n - 1;
        int tail = m + n - 1;
        int cur;
        while (p1 >= 0 || p2 >= 0) {
            if (p1 == -1) {
                cur = nums2[p2--];
            } else if (p2 == -1) {
                cur = nums1[p1--];
            } else if (nums1[p1] > nums2[p2]) {
                cur = nums1[p1--];
            } else {
                cur = nums2[p2--];
            }
            nums1[tail--] = cur;
        }
    }
};
```

### 283. 移动零

给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
请注意 ，必须在不复制数组的情况下原地对数组进行操作。

示例 1:

```bash
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n = nums.size();
        int idx = 0;
        for (int i = 0; i < n; i++){
            if (nums[i] != 0){
                swap(nums[i], nums[idx]);
                idx += 1;
            }
        }
    }
};
```

### 345. 反转字符串中的元音字母

给你一个字符串 s ，仅反转字符串中的所有元音字母，并返回结果字符串。
元音字母包括 'a'、'e'、'i'、'o'、'u'，且可能以大小写两种形式出现不止一次。

```cpp
class Solution {
public:
    string reverseVowels(string s) {
        int n = s.size();
        if(n == 1){
            return s;
        }

        set<char> st = {'a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U'};
        int left = 0, right = n - 1;
        while(left < right){
            if (st.find(s[left]) == st.end()){
                left ++;
            }

            if (st.find(s[right]) == st.end()){
                right --;
            }

            if (st.find(s[left]) != st.end() && st.find(s[right]) != st.end()){
                swap(s[left], s[right]);
                left ++;
                right --;
            }
        }

        return s;
    }
};
```

### 455. 分发饼干

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。
对每个孩子 i，都有一个胃口值 g[i]，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j，都有一个尺寸 s[j] 。如果 s[j] >= g[i]，我们可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。

```cpp
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());
        int m = g.size(), n = s.size();
        int count = 0;
        for (int i = 0, j = 0; i < m && j < n; i++, j++) {
            while (j < n && g[i] > s[j]) {
                j++;
            }
            if (j < n) {
                count++;
            }
        }
        return count;
    }
};
```

## 哈希表

### 1. 两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
你可以按任意顺序返回答案。

示例 1：

```bash
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回[0, 1]。
```

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hashtable;
        for (int i = 0; i < nums.size(); i++) {
            auto it = hashtable.find(target - nums[i]);
            if (it != hashtable.end()){
                return {it->second, i};
            }
            hashtable[nums[i]] = i;
        }
        return {};
    }
};
```

### 3. 无重复字符的最长子串

给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:

输入: s = "abcabcbb"
输出: 3
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if(s.size() == 0) return 0;
        unordered_set<char> lookup;
        int maxStr = 0;
        int left = 0;
        for(int i = 0; i < s.size(); i++){
            while (lookup.find(s[i]) != lookup.end()){
                lookup.erase(s[left]);
                left ++;
            }
            maxStr = max(maxStr,i-left+1);
            lookup.insert(s[i]);
        }
        return maxStr;
        
    }
};
```

### 76. 最小覆盖子串

给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

注意：

对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
如果 s 中存在这样的子串，我们保证它是唯一的答案。

示例 1：

```bash
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

```cpp
class Solution {
public:
    unordered_map <char, int> ori, cnt;

    bool check() {
        for (const auto &p: ori) {
            if (cnt[p.first] < p.second) {
                return false;
            }
        }
        return true;
    }

    string minWindow(string s, string t) {
        for (const auto &c: t) {
            ++ori[c];
        }

        int l = 0, r = -1;
        int len = INT_MAX, ansL = -1, ansR = -1;

        while (r < int(s.size())) {
            if (ori.find(s[++r]) != ori.end()) {
                ++cnt[s[r]];
            }
            while (check() && l <= r) {
                if (r - l + 1 < len) {
                    len = r - l + 1;
                    ansL = l;
                }
                if (ori.find(s[l]) != ori.end()) {
                    --cnt[s[l]];
                }
                ++l;
            }
        }

        return ansL == -1 ? string() : s.substr(ansL, len);
    }
};
```

### 128. 最长连续序列

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
请你设计并实现时间复杂度为 O(n) 的算法解决此问题。

示例 1：

```bash
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        int ret = 0;
        unordered_set<int> st;
        int n = nums.size();
        for (int i = 0; i < n; i++){
            if (st.find(nums[i]) == st.end()){
                st.insert(nums[i]);
            }
        }

        int cur_len = 0;
        for (int i = 0; i < n; i ++){
            cur_len = 0;
            if (st.find(nums[i] - 1) == st.end()){
                int start = nums[i];
                while(st.find(start) != st.end()){
                    cur_len ++;
                    start ++;
                }
                ret = max(ret, cur_len);
            }
        }

        return ret;
    }
};
```

### 202. 快乐数

编写一个算法来判断一个数 n 是不是快乐数。

「快乐数」 定义为：

对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。
如果这个过程 结果为 1，那么这个数就是快乐数。
如果 n 是 快乐数 就返回 true ；不是，则返回 false 。

```cpp
class Solution {
public:
    int cal(int n){
        int ret = 0;
        int k = 0;
        while(n != 0){
            k = n % 10;
            n = (n - k)/10;
            ret += k * k;
        }
        return ret;
    }
    bool isHappy(int n) {
        set<int> nums;
        int tmp = cal(n);
        if (tmp == 1){
            return true;
        }

        nums.insert(tmp);
        while(tmp != 1){
            tmp = cal(tmp);
            if (nums.find(tmp) == nums.end()){
                nums.insert(tmp);
            } else {
                return false;
            }
        }
        return true;
        
    }
};
```

## 二分

### 74. 搜索二维矩阵

给你一个满足下述两条属性的 m x n 整数矩阵：

每行中的整数从左到右按非严格递增顺序排列。
每行的第一个整数大于前一行的最后一个整数。
给你一个整数 target ，如果 target 在矩阵中，返回 true ；否则，返回 false。

```bash
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int m = matrix.size(), n = matrix[0].size();
        int low = 0, high = m * n - 1;
        while (low <= high) {
            int mid = (high - low) / 2 + low;
            int x = matrix[mid / n][mid % n];
            if (x < target) {
                low = mid + 1;
            } else if (x > target) {
                high = mid - 1;
            } else {
                return true;
            }
        }
        return false;
    }
};
```

### 162. 寻找峰值

峰值元素是指其值严格大于左右相邻值的元素。
给你一个整数数组 nums，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。
你可以假设 nums[-1] = nums[n] = -∞ 。
你必须实现时间复杂度为 O(log n) 的算法来解决此问题。

```cpp
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int n = nums.size();
        int idx = rand() % n;

        // 辅助函数，输入下标 i，返回一个二元组 (0/1, nums[i])
        // 方便处理 nums[-1] 以及 nums[n] 的边界情况
        auto get = [&](int i) -> pair<int, int> {
            if (i == -1 || i == n) {
                return {0, 0};
            }
            return {1, nums[i]};
        };

        while (!(get(idx - 1) < get(idx) && get(idx) > get(idx + 1))) {
            if (get(idx) < get(idx + 1)) {
                idx += 1;
            }
            else {
                idx -= 1;
            }
        }
        
        return idx;
    }
};
```

## 贪心

### 122. 买卖股票的最佳时机 II

给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。
在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。
返回 你能获得的 最大 利润 。

示例 1：

```bash
输入：prices = [7,1,5,3,6,4]
输出：7
解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
     总利润为 4 + 3 = 7 。
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {   
        int ans = 0;
        int n = prices.size();
        for (int i = 1; i < n; ++i) {
            ans += max(0, prices[i] - prices[i - 1]);
        }
        return ans;
    }
};
```
