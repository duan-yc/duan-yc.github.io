---
layout: post
title: "c++刷题"
subtitle: "c++"
author: "DYC"
header-img: "img/post-bg-linux.jpg"
header-mask: 0.3
catalog: true
tags:
  - c++
---

### Leetcode-c++

#### 链表

##### [反转链表](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)

```c++
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
    ListNode* reverseList(ListNode* head) {
        ListNode* reverseList=NULL;
        while(head!=NULL){
            ListNode* temp=head->next;
            head->next=reverseList;
            reverseList=head;
            head=temp;
        }
        return reverseList;
    }
};
```

思路：首先定义一个链表reverseList，作为反转链表的头节点，然后进入循环挨个提取head链表的节点插入到reverseList链表中，插入的过程是首先定义一个链表保存当前head之后的链表，然后让head指向reverseList，然后将head赋给reverseList，最后将head重新指向原来的head.next。

[从尾到头打印链表](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

```c++
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
    vector<int> reversePrint(ListNode* head) {
        stack<int>s;
        while(head!=NULL){
            s.push(head->val);
            head=head->next;
        }
        vector<int>v;
        int size=s.size();
        for(int i=0;i<size;i++){
            v.push_back(s.top());
            s.pop();
        }
        return v;

    }
};
```

思路：定义一个栈，将head链表的数据加入栈中，再挨个取出栈顶元素放入vector中