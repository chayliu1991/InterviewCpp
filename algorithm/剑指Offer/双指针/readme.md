# [18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

```
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val) {
        if(head == nullptr)
            return head;
        if(head->val == val)
            return head->next;
        
        ListNode* curr = head,* prev = head;
        while(curr)
        {
            if(curr->val == val)
                prev->next = curr->next;
            prev = curr;
            curr = curr->next;
        }
        return head;
    }
};
```

# [21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

```
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int n = nums.size();
        if(n <= 1)
            return nums;
        int left = 0,right = n -1;
        while(left < right)
        {
            if(nums[left] & 0x01 && left < right)
                left++;
            if(!(nums[right] & 0x01) && left < right)
                right--;
            if(left < right)      
                std::swap(nums[left],nums[right]);       
        }
        return nums;
    }
};
```

# [22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

```
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        if(head == nullptr)
            return head;
        ListNode* fast = head,*slow = head;
        while(fast && k--)
            fast = fast->next;
        
        while(fast)
        {
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }
};
```

# [57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

```
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        if(nums.empty())
            return {};
        int left = 0,right = nums.size()-1;
        while(left < right)
        {
            if(nums[left] + nums[right] == target)
                return {nums[left],nums[right]};
            else if(nums[left] + nums[right] > target)
                right--;
            else    
                left++;
        }
        return {};
    }
};
```