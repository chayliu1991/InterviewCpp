# [10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

- dp

```
class Solution {
public:
    int fib(int n) {
        if(n == 0)
            return 0;
        if(n == 1)
            return 1;
        int left = 0,right = 1;
        for(int i = 2;i <= n;i++)
        {
            int tmp = (left + right) % 1000000007;
            left = right;
            right = tmp;
        }
        return right;
    }
};
```

# [10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

- dp

```
class Solution {
public:
    int numWays(int n) {
        if(n < 0)
            return -1;
        if(n == 0 || n == 1)
            return 1;
        if(n == 2)
            return 2;
        int left = 1,right = 2;
        for(int i = 3;i <= n;i++)
        {
            int tmp = (left + right) % 1000000007;
            left = right;
            right = tmp;
        }
        return right;
    }
};
```

# [42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

```
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if(nums.empty())
            return 0;
        int res = INT_MIN;
        int sum = 0;
        for(int i = 0;i < nums.size();i++)
        {
            if(sum < 0)
                sum = nums[i];
            else
                sum += nums[i];
            
            res = max(res,sum);
        }
        return res;
    }
};
```

# [46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

- dp

```
class Solution {
public:
    int translateNum(int num) {
        std::string str = std::to_string(num);
        int len = str.size();
        if(len < 2) 
            return len;

        std::vector<int> dp(len+1);
        dp[1] = 1;
        dp[0] = 1;
        for(int i = 2;i <= len;i++)
        {
            dp[i] = dp[i-1];
            if(str[i-2] == '1' || (str[i-2] == '2' && str[i-1] <= '5')) //@ 10 ~25
                dp[i] = dp[i-2]+dp[i-1];
        }
        return dp.back();
    }
};
```
- 状态压缩
```
class Solution {
public:
    int translateNum(int num) {
        std::string str = std::to_string(num);
        return back_trace(str,0);
    }

    int back_trace(std::string& s,int i)
    {
        int n = s.length();
        if(i == n)
            return 1;
        if(i == n-1 || s[i] == '0' || s.substr(i,2) > "25")
            return back_trace(s,i+1);
        return back_trace(s,i+1) + back_trace(s,i+2);
    }
};
```

# [47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

- dp

```
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int rows = grid.size(),cols = grid[0].size();
        std::vector<std::vector<int>> dp(rows,std::vector<int>(cols));
        dp[0][0] = grid[0][0];
        for(int i = 1;i < rows;i++)
            dp[i][0] = dp[i-1][0] + grid[i][0];
        for(int j =1;j < cols;j++)
            dp[0][j] = dp[0][j-1] + grid[0][j];
        
        for(int i = 1;i < rows;i++)
        {
            for(int j =1;j < cols;j++)
            {
                dp[i][j] = max(dp[i-1][j],dp[i][j-1]) + grid[i][j];
            }
        }
        return dp.back().back();
    }
};
```
- dp + 状态压缩
```
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        if(grid.empty() || grid[0].empty())
            return 0;
        
        int rows = grid.size(),cols = grid[0].size();
        std::vector<int> dp(cols);
        for(int i = 0;i < rows;i++)
        {
            for(int j = 0;j < cols;j++)
            {
                if(i == 0 && j == 0)
                    dp[j] = grid[0][0];
                else if(i == 0)
                    dp[j] = grid[0][j] + dp[j-1] ;
                else if(j == 0)
                    dp[j] += grid[i][0];
                else 
                    dp[j] = max(dp[j-1],dp[j]) + grid[i][j];
            }
        }
        return dp.back();
    }
};
```

# [48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

```
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        std::vector<int> hash(128,-1);
        int res,left = -1;
        for(int i = 0;i < s.size();i++)
        {
            left = max(left,hash[s[i]]);
            hash[s[i]] = i;
            res = max(res,i-left);
        }
        return res;
    }
};
```

# [63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

- 遍历

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() <= 1)
            return 0;
        int profit = INT_MIN,buy = prices[0];
        for(int i = 0;i < prices.size();i++)
        {
            buy = min(buy,prices[i]);
            profit = max(profit,prices[i] - buy);
        }
        return profit;
    }
};
```

- 单调栈

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() <= 1)
            return 0;
        std::stack<int> s;
        s.push(prices[0]);
        int profit = 0;
        for(int i = 1;i < prices.size();i++)
        {
            if(s.top() > prices[i])
            {
                s.pop();
                s.push(prices[i]);
            }
            else
            {
                profit = max(profit,prices[i] - s.top());
            }
        }
        return profit;
    }
};
```

- dp

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() <= 1)
            return 0;
        int curr = 0,profit = 0;
        for(int i = 1;i < prices.size();i++)
        {
            curr = max(curr,0) + prices[i] - prices[i-1];
            profit = max(profit,curr);
        }
        return profit;
    }
};
```

- dp

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if(n <= 1)
            return 0;
        
        std::vector<std::vector<int>> dp(n,std::vector<int>(2,0));
        dp[0][0] = 0;
        dp[0][1] = -prices[0];

        for(int i = 1;i < n;i++)
        {
            dp[i][0] = max(dp[i-1][0],dp[i-1][1] + prices[i]);
            dp[i][1] = max(dp[i-1][1],-prices[i]);
        }
        return dp[n-1][0];
    }
};
```

