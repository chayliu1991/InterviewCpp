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

# [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

```
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return true;
        
        std::vector<int> values;
        ListNode* curr = head;
        while(curr)
        {
            values.push_back(curr->val);
            curr = curr->next;
        }

        for(int i = 0,j = values.size()-1;i < j;i++,j--)
        {
            if(values[i] != values[j])
                return false;
        }
        return true;
    }
};
```

```
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return true;
        
        ListNode* left_end = find_end_half(head);
        ListNode* right_rev = reverse(left_end->next);
        ListNode* p1 = head,*p2 = right_rev;
        bool res = true;
        while(res && (p1 && p2))
        {
            if(p1->val != p2->val)
                res = false;
            
            p1 = p1->next;
            p2 = p2->next;
        }
        left_end->next = reverse(right_rev);
        return res;
    }

    ListNode* reverse(ListNode* head)
    {
        if(head == nullptr || head->next == nullptr)
            return head;
        ListNode*  new_head = reverse(head->next);
        head->next->next = head;
        head->next = nullptr;
        return new_head;
    }
    
    ListNode* find_end_half(ListNode* head)
    {
        ListNode *slow = head,*fast = head;
        while(fast->next && fast->next->next)
        {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }
};
```

# [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

```
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode* curr = head;
        while(curr->next)
        {
            if(curr->val == curr->next->val)
                curr->next = curr->next->next;
            else
                curr = curr->next;
        }
        return head;
    }
};
```

# [328. 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/)

```
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode *odd_head = head,*odd_tail = head;
        ListNode *even_head = head->next,*even_tail = even_head;
        
        for(ListNode* curr = head->next->next;curr != nullptr;)
        {
            odd_tail = odd_tail->next = curr;
            curr = curr->next;

            if(curr)
            {
                even_tail = even_tail->next = curr;
                curr = curr->next;
            }
        }
        odd_tail->next = even_head;
        even_tail->next = nullptr;
        return odd_head;
    }
};
```

# [19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

```
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* fast = head;
        while(n--)
        {
            if(fast == nullptr)
                return nullptr;
            fast = fast->next;
        }

        if(fast == nullptr)
            return head->next;

        ListNode* slow = head;
        while(fast->next)
        {
            slow = slow->next;
            fast = fast->next;
        }

        slow->next = slow->next->next;
        return head;
    }
};
```

# [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

```
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        
        ListNode* slow = head,*fast = head->next;
        while(fast && fast->next)
        {
            slow = slow->next;
            fast = fast->next->next;
        } 

        ListNode* mid = slow->next;
        slow->next = nullptr;
        return merge(sortList(head),sortList(mid));
    }

     ListNode* merge(ListNode* l1, ListNode* l2)
     {
         ListNode node(-1);
         ListNode* dummy = &node,*curr = dummy;
         while(l1 && l2)
         {
            if(l1->val > l2->val)
                std::swap(l1,l2);
            curr->next = l1;
            l1 = l1->next;
            curr = curr->next;
         }

         curr->next = l1 ? l1 : l2;
         return dummy->next;
     }
};
```







