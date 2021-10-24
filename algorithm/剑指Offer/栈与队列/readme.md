# [09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

**思路**

- 一个入栈，一个出栈

```
class CQueue {
public:
    CQueue() {

    }
    
    void appendTail(int value) {
        s_in.push(value);
    }
    
    int deleteHead() {
        if(s_in.empty() && s_out.empty())
            return -1;
        
        if(s_out.empty())
        {
            while(!s_in.empty())
			{
				s_out.push(s_in.top());
				s_in.pop();
			}
        }
		
		auto res = s_out.top();
		s_out.pop();
		return res;
    }

    stack<int> s_in,s_out;
};
```

# [19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

```
class Solution {
public:
    bool isMatch(string s, string p) {
        if(p.empty())
            return s.empty();
        
        auto matched = [&](){
            return s[0] == p[0] || p[0] == '.';
        };

        if(p[1] == '*')
            return isMatch(s,p.substr(2)) || (!s.empty() && matched() && isMatch(s.substr(1),p));
        else
            return !s.empty() && matched() && (isMatch(s.substr(1),p.substr(1)));
    }
};
```

# [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

```
class Solution {
public:
    bool isValid(string s) {
        std::unordered_map<char,char> pairs =
        {
            {')','('},
            {']','['},
            {'}','{'}
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

# [30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

思路：

- 一个数据栈，一个最小栈

```
class MinStack {
public:
    /** initialize your data structure here. */
    MinStack() {

    }
    
    void push(int x) {
        if(s_data.empty())
        {
            s_min.push(x);
        }
        else
        {
            if(s_min.top() <= x)
                s_min.push(s_min.top());
            else
                s_min.push(x);
        }

        s_data.push(x);
    }
    
    void pop() {
        s_min.pop();
        s_data.pop();
    }
    
    int top() {
        return s_data.top();
    }
    
    int min() {
        return s_min.top();
    }

    stack<int> s_min,s_data;
};

```

# [ 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

```
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
		if(pushed.size() != popped.size())
			return false;
		std::stack<int> s;
		int n = popped.size();
        int j = 0;
		for(int i = 0;i < n;i++)
		{
			s.push(pushed[i]);
			while(!s.empty() && j < n && s.top() == popped[j])
			{
				s.pop();
				j++;
			}			
		}
		return s.empty();        
    }
};
```

# [59 - I. 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

```
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if(nums.empty())
            return {};

        std::priority_queue<std::pair<int,int>> pq;
        for(int i = 0; i < k;i++)
            pq.emplace(nums[i],i);
        
        vector<int> res;
        res.push_back(pq.top().first);

        for(int i = k;i < nums.size();i++)
        {
            pq.emplace(nums[i],i);
            while(i - pq.top().second >= k)
                pq.pop();
            res.push_back(pq.top().first);
        }

        return res;
    }
};
```

```
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if(nums.empty())
            return {};

        std::deque<int> dq;
        for (int i = 0; i < k; ++i) 
        {
            while (!dq.empty() && nums[i] >= nums[dq.back()]) 
                dq.pop_back();
            
            dq.push_back(i);
        }

        std::vector<int> res = {nums[dq.front()]};
        for (int i = k; i < nums.size(); ++i) 
        {
            while (!dq.empty() && nums[i] >= nums[dq.back()])
                dq.pop_back();
            
            dq.push_back(i);
            while (i - dq.front() >= k) 
                dq.pop_front();

            res.push_back(nums[dq.front()]);
        }
        return res;
    }
};
```

# [59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

```
class MaxQueue {
public:
    MaxQueue() {

    }
    
    int max_value() {
        if(dq.empty())
            return -1;
        return dq.front();
    }
    
    void push_back(int value) {
        data.push(value);
        
        while(!dq.empty() && dq.back() <= value)
            dq.pop_back();
        
        dq.push_back(value);
    }
    
    int pop_front() {
        if(data.empty())
            return -1;
        
        int res = data.front();
        data.pop();
        while(!dq.empty() && dq.front() == res)
            dq.pop_front();
        return res;
    }

    std::deque<int> dq;
    std::queue<int> data;
};
```

