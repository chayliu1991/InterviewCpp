# [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

```
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode* new_head = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;
        return new_head;
    }
};
```

```
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode* prev = nullptr,*curr = head;
        while(curr)
        {
            ListNode* tmp = curr->next;
            curr->next = prev;
            prev = curr;
            curr = tmp;
        }
        return prev;
    }
};
```

# [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

```
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode dummy(-1);
        ListNode* curr = &dummy;
        while(list1 && list2)
        {
            if(list1->val <= list2->val)
            {
                curr->next = list1;
                list1 = list1->next;
            }
            else
            {
                curr->next = list2;
                list2 = list2->next;
            }
            curr = curr->next;
        }
        curr->next = list1 ? list1 : list2;
        return dummy.next;
    }
};
```

# [24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

```
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode dummy(-1);
        dummy.next = head;
        ListNode* prev = &dummy,*curr = head;
        ListNode* left = &dummy,*right = head;
        while(curr && curr->next)
        {
            left = curr->next;
            right = curr->next->next;

            curr->next->next = curr;
            curr->next = right;
            prev->next = left;

            prev = curr;
            curr = right;
        }
        return dummy.next;
    }
};
```

# [160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

```
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode *pA = headA,*pB = headB;
        while(pA != pB)
        {
            pA = (pA == nullptr ? headB : pA->next);
            pB = (pB == nullptr ? headA : pB->next);
        }
        return pA;
    }
};
```

