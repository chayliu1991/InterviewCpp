# [225. 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

```
class MyStack {
public:
    MyStack() {

    }
    
    void push(int x) {
        data.push(x);
    }
    
    int pop() {
        std::queue<int> tmp;
        while(data.size() > 1)
        {
            tmp.push(data.front());
            data.pop();
        }
        int res = data.front();
        data.pop();
        data = std::move(tmp);
        return res;
    }
    
    int top() {
        int res = pop();
        push(res);
        return res;
    }
    
    bool empty() {
        return data.empty();
    }
    
private:
    std::queue<int> data;
};
```

# [232. 用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

```
class MyQueue {
public:
    MyQueue() {

    }
    
    void push(int x) {
        s_in.push(x);
    }
    
    int pop() {
        if(s_out.empty())
        {
            while(!s_in.empty())
            {
                s_out.push(s_in.top());
                s_in.pop();
            }
        }

        int res = s_out.top();
        s_out.pop();
        return res;
    }
    
    int peek() {
        if(s_out.empty())
        {
            while(!s_in.empty())
            {
                s_out.push(s_in.top());
                s_in.pop();
            }
        }

        int res = s_out.top();
        return res;
    }
    
    bool empty() {
        return s_in.empty() && s_out.empty();
    }

    std::stack<int> s_in;
    std::stack<int> s_out;
};
```

# [155. 最小栈](https://leetcode-cn.com/problems/min-stack/)

```
class MinStack {
public:
    MinStack() {

    }
    
    void push(int val) {
        s_data.push(val);
        if(s_min.empty() || s_min.top() > val)
            s_min.push(val);
        else
            s_min.push(s_min.top());

    }
    
    void pop() {
        s_data.pop();
        s_min.pop();
    }
    
    int top() {
        return s_data.top();
    }
    
    int getMin() {
        return s_min.top();
    }
    
private:
    std::stack<int> s_data;
    std::stack<int> s_min;
};
```

# [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

```
class Solution {
public:
    bool isValid(string s) {
        std::unordered_map<char,char> pairs ={
            {'}','{'},
            {']','['},
            {')','('},
        };

        std::stack<char> sk;
        for(const auto c : s)
        {
            if(pairs.find(c) != pairs.end())
            {
                if(sk.empty() || sk.top() != pairs[c])
                    return false;
                sk.pop();
            }         
            else
                sk.push(c);
        }
        return sk.empty() ? true : false;
    }
};
```

# [739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

```
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        std::vector<int> res(n);
        std::stack<int> s;
        for(int i = 0;i < n;++i)
        {
            while(!s.empty() && temperatures[i] > temperatures[s.top()])
            {
                int index = s.top();
                res[index] = i - index;
                s.pop();
            }
            s.push(i);
        }
        return res;
    }
};
```

# [23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

```
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        struct compare{
            bool operator() (const ListNode* l1,const ListNode* l2)
            {
                return l1->val > l2->val;
            }
        };

        std::priority_queue<ListNode*,std::vector<ListNode*>,compare> pq;
        for(const auto list : lists)
        {
            if(list)
                pq.push(list);
        }

        ListNode dummy(-1);
        ListNode *p = &dummy,*curr = &dummy;
        while(!pq.empty())
        {
            ListNode* tmp = pq.top();
            pq.pop();
            curr->next = tmp;
            curr = curr->next;
            if(tmp->next)
                pq.push(tmp->next);
        }
        return dummy.next;
    }
};
```

# [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

```
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if(k <= 0)
            return {};
        
        std::vector<int> res;
        std::deque<int> dq;
        for(int i = 0;i < k;i++)
        {
            while(!dq.empty() && nums[dq.back()] <= nums[i])
                dq.pop_back();
            dq.push_back(i);
        }
        res.emplace_back(nums[dq.front()]);

        for(int i = k;i < nums.size();i++)
        {
            while(!dq.empty() && i - dq.front() >= k)
                dq.pop_front();
             while(!dq.empty() && nums[dq.back()] <= nums[i])
                dq.pop_back();
            dq.push_back(i);
            res.emplace_back(nums[dq.front()]);
        }
        return res;
    }
};
```

# [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

```
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int res = 0;
        std::stack<int> s;
        heights.insert(heights.begin(),0);
        heights.push_back(0);
        for(int i = 0;i < heights.size();i++)
        {
            while(!s.empty() && heights[i] < heights[s.top()])
            {
                int curr = s.top();
                s.pop();
                res = std::max(res,heights[curr] * (i - s.top() -1));
            }
            s.push(i);
        }
        return res;
    }
};
```



