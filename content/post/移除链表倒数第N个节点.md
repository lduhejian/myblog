---
title: "移除链表倒数第N个节点"
date: 2020-08-27
draft: false
tags: [链表, c]
description: "移除链表倒数第N个节点  c语言"
---
```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */

struct ListNode* setDummyHead(struct ListNode* l)
{
    struct ListNode* dummyHead = malloc(sizeof(struct ListNode));
    dummyHead->val = 0;
    dummyHead->next = l;
    return dummyHead;
}

struct ListNode* removeNthFromEnd(struct ListNode* head, int n){
    int count = 0;
    head = setDummyHead(head);
    struct ListNode* p = head;
    while(p != NULL)
    {
        count ++;
        p = p->next;
    }
    int index = count - n; // 需要删除元素的前一个位置
    p = head;
    int i = 0;
    while (p != NULL)
    {
        i ++;
        if (i == index) 
        {
            struct ListNode* tmp = p->next;
             p->next = p->next->next;
            free(tmp);
            break;
        }
        p = p->next;
    }
    return head->next;
}
```

参考：  
[leetcode - 19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)