# [14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```
class Solution {
public:
    int cuttingRope(int n) {
        if(n == 2)
            return 1;
        if(n == 3)
            return 2;
        
        int res = 1;
        while(n > 4)
        {
            res *= 3;
            n -= 3;
        }
        return n * res;
    }
};
```

# [14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

```
class Solution {
public:
    int cuttingRope(int n) {
        if(n <= 3)
            return n - 1;
        if(n == 4)
            return 4;
        long res = 1;
        while(n > 4)
        {
            res *= 3;
            res %= 1000000007;
            n -= 3;
        }
        return (res * n) % 1000000007;
    }
};
```

# [39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

```
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        std::sort(nums.begin(),nums.end());
        return nums[nums.size()/2];
    }
};
```

# [43. 1～n 整数中 1 出现的次数](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

```
class Solution {
public:
    int countDigitOne(int n) {
        int res = 0;
        long i = 1;
        while(n / i)
        {
            long high = n / (10 * i);
            long curr = (n / i) % 10;
            long low = n - (n / i) * i;

            if(curr == 0)
                res += high * i;
            else if(curr == 1)
                res += high * i + low + 1;
            else
                res += high * i + i;
            
            i *= 10;
        }
        return res;
    }
};
```

# [44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

```
class Solution {
public:
    int findNthDigit(int n) {
        long k = n;
        for(int i = 1;;i++)
        {
            if(i * pow(10,i) > k)
            {
                return std::to_string(k/i)[k%i] - '0';
            }
            k += pow(10,i);
        }
    }
};
```

# [57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

```
class Solution {
public:
    vector<vector<int>> findContinuousSequence(int target) {
        int left = 1,right = 1;
        vector<vector<int>>  res;
		int sum = 0;
        while(left <= target/2)
        {
            if(sum < target)
			{
				sum += right;
				right++;		
			}
			else if(sum > target)
			{
				sum -= left;
				left++;	
			}
			else
			{
				vector<int> v;
				for(int i = left;i < right;i++)
					v.push_back(i);
				res.push_back(v);	
				sum -= left;
				left ++;	
			}			
        }
        return res;
    }
};
```

# [ 62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

```
class Solution {
public:
    int lastRemaining(int n, int m) {
        if(n == 1)
            return 0;
        int res;
        for(int i = 2;i <= n;i++)
        {
            res = (res + m) % i;
        }
        return res;
    }
};
```

# [66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

```
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        int n = a.size();
        vector<int> res(n,1);
        for(int i = 1;i < n;i++)
            res[i] = res[i-1] * a[i-1];
        int tmp = 1;
        for(int i = n-2;i >= 0;i--)
        {
            tmp *= a[i+1];
            res[i] *= tmp;
        }
        return res;
    }
};
```