# [376. 摆动序列](https://leetcode-cn.com/problems/wiggle-subsequence/)

```
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        int n = nums.size();
        if(n < 2) return n;

        enum STATE {BEGIN = 0,UP,DOWN};
        STATE state = BEGIN;
        int max_len = 1;
        for(int i=1;i<n;i++)
        {
            switch(state)
            {
                case BEGIN:
                    if(nums[i-1] < nums[i])
                    {
                        state = UP;
                        max_len++;
                    }
                    else if(nums[i-1] > nums[i])
                    {
                        state = DOWN;
                        max_len++;
                    }
                    break;
                case UP:
                    if(nums[i-1] > nums[i])
                    {
                        state = DOWN;
                        max_len++;
                    }
                    break;
                case DOWN:
                if(nums[i-1] < nums[i])
                {
                    state = UP;
                    max_len++;
                }
                break;
            }
        } 
        return max_len;
    }
};
```

# [860. 柠檬水找零](https://leetcode-cn.com/problems/lemonade-change/)

```
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int fives = 0,tens = 0;
        for(const auto bill : bills)
        {
            if(bill == 5)
                fives++;
            else if(bill == 10)
            {
                if(fives > 0)
                    fives--,tens++;   
                else    
                    return false;
            }
            else
            {
                if(fives > 0 && tens > 0)
                    fives--,tens--;
                else if(fives >= 3)
                    fives -= 3;
                else
                    return false;
            }
        }
        return true;
    }
};
```

# [406. 根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height/)

```
class Solution {
public:
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(),people.end(),[](const vector<int>& lhs,const vector<int>& rhs){
            return lhs[0] > rhs[0] || (lhs[0] == rhs[0] && lhs[1] < rhs[1]);
        });

        vector<vector<int>> res;
        for(auto const p : people)
        {
            res.insert(res.begin() + p[1],p);
        }
        return res;
    }
};
```

# [134. 加油站](https://leetcode-cn.com/problems/gas-station/)

```
class Solution {
public:
   int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int total_gas = accumulate(gas.begin(),gas.end(),0);
        int total_cost = accumulate(cost.begin(),cost.end(),0);

        if(total_cost > total_gas) 
            return -1;
        int remain = 0,start = 0;
        for(int i=0;i<gas.size();++i)
        {
            if(remain + gas[i] < cost[i])
            {
                remain = 0;
                start = i + 1;
            }                
            else
                remain += gas[i] - cost[i];
        }

        return start;
    }
};
```

# [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

```
class Solution {
public:
   bool canJump(vector<int>& nums) 
   {
       int longest = 0;
       for(int i=0;i<nums.size();i++)
       {
           if(i > longest)
                return false;
            longest = max(longest,i+nums[i]);
       }
       return true;
   }
};
```

# [392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

```
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int k = 0;
        for(int i=0;i<t.size() && k<s.size();i++)
        {
            if(t[i] == s[k])
                k++;
        }
        return k == s.size();
    }
};
```

# [316. 去除重复字母](https://leetcode-cn.com/problems/remove-duplicate-letters/)

```
class Solution {
public:
    string removeDuplicateLetters(string s) {
		unordered_map<char,int> remain_counter;
		for(const auto str : s)
			remain_counter[str]++;
		
		vector<char> S;
		for(const auto c : s)
		{
			if(find(S.begin(),S.end(),c) == S.end())
			{
				//@ 栈不为空，并且当前的字符小于栈尾元素，并且末尾这个字符后面还有机会出现
				while(!S.empty() && c < S.back() && remain_counter[S.back()] > 0)				
					S.pop_back();	
				S.push_back(c);
			}			
			remain_counter[c]--;
		}		
		return string(S.begin(),S.end());
    }
};
```

# [455. 分发饼干](https://leetcode-cn.com/problems/assign-cookies/)

```
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(),g.end());
        sort(s.begin(),s.end());
        int child = 0 ,cookie = 0;
        while(child < g.size() && cookie < s.size())
        {
            if(g[child] <= s[cookie])
                ++child;
            ++cookie; 
        }
        return child;
    }
};
```

# [135. 分发糖果](https://leetcode-cn.com/problems/candy/)

```
class Solution {
public:
    int candy(vector<int>& ratings) {
        int size = ratings.size();
        if(size < 2) return size;
        vector<int> candies(size,1);
        for(int i=1;i<size;i++)
            if(ratings[i] > ratings[i-1])
                candies[i] = candies[i-1]+1;
        for(int i=size-1;i>0;i--)
            if(ratings[i-1] > ratings[i])
                candies[i-1] = max(candies[i-1],candies[i]+1);        
        return accumulate(candies.begin(),candies.end(),0);
    }
};
```

