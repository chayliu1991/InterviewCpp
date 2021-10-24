# [ 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

**思路**

- `std::verse`

```
class Solution {
public:
    vector<int> reversePrint(ListNode* head) {
        std::vector<int> res;
        while(head != nullptr)
        {
            res.push_back(head->val);
            head = head->next;
        }
        std::reverse(res.begin(),res.end());
        return res;
    }
};
```


# [23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

```
class Solution {
public:
    
    struct comp{
        bool operator()(ListNode* l1,ListNode* l2)
        {
            return l1->val > l2->val;
        }
    };

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        std::priority_queue<ListNode*,std::vector<ListNode*>,comp> pq;
        for(const auto& l : lists)
        {
            if(l)
                pq.push(l);
        }
        
        ListNode node(-1);
        ListNode* dummy = &node,*curr = dummy;
        while(!pq.empty())
        { 
            ListNode* tmp = pq.top();
            pq.pop();
            curr->next = tmp;
            curr = curr->next;
            if(tmp->next)
                pq.push(tmp->next); 
        }
        return dummy->next;
    }
};
```

# [24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

```
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head->next == nullptr)
            return head;
        ListNode* res = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;
        return res; 
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

# [25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

```
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        if(l1 == nullptr)
            return l2;
        if(l2 == nullptr)
            return l1;
        
        ListNode* res = (l1->val <= l2->val) ? l1 : l2;
        ListNode* curr = res;
        ListNode* p1 = (res == l1) ? l1->next : l1;
        ListNode* p2 = (res == l2) ? l2->next : l2;

        while(p1 && p2)
        {
            if(p1->val <= p2->val)
            {
                curr->next = p1;
                p1 = p1->next;
            }
            else
            {
                curr->next = p2;
                p2 = p2->next;
            }
            curr = curr->next;
        } 
        if(p1)
            curr->next = p1;
        if(p2)
            curr->next = p2;
        return res;
    }
};
```

# [35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

```
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(head == nullptr)
            return nullptr;
        
        std::unordered_map<Node*,Node*> hash;
        Node* curr = head;
        while(curr)
        {
            hash[curr] = new Node(curr->val);
            curr = curr->next;
        }

        curr = head;
        while(curr)
        {
            hash[curr]->next = hash[curr->next];
            hash[curr]->random = hash[curr->random];
            curr = curr->next;
        }
        return hash[head];
    }
};
```

```
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(head == nullptr)
            return head;

        Node* curr = head;
        while(curr)
        {
            Node* node = new Node(curr->val);
            node->next = curr->next;
            curr->next = node;            
            curr = node->next;
        }

        curr = head;
        while(curr)
        {
            if(curr->random != nullptr)
                curr->next->random = curr->random->next;
            curr = curr->next->next;
        }

        curr = head->next;
        Node* prev = head;
        Node* res = head->next;
        while(curr->next != nullptr)
        {
            prev->next = prev->next->next;
            curr->next = curr->next->next;
            prev = prev->next;
            curr = curr->next;
        }
        prev->next = nullptr;
        return res;
    }
};
```

# [52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

```
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(!headA || !headB)
            return nullptr;
        ListNode* h1 = headA,*h2 = headB;
        while(h1 != h2)
        {
            h1 = (h1 != nullptr) ? h1->next : headB;
            h2 = (h2 != nullptr) ? h2->next : headA; 
        }
        return h1;
    }
};
```

```
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        std::unordered_set<ListNode *> s;
        ListNode*hA = headA,*hB = headB;
        while(hA)
        {
            s.insert(hA);
            hA = hA->next;
        }

        while(hB)
        {
            if(s.find(hB) != s.end())
                return hB;
            hB = hB->next;
        }
        return nullptr;
    }
};
```

