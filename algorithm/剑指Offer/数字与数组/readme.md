

# [17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

```
class Solution {
public:
    vector<int> printNumbers(int n) {
        std::vector<int> res;
        int max = pow(10,n);
        for(int i = 1;i < max;i++)
            res.push_back(i);
        return res;
    }
};
```



# [29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if(matrix.empty() || matrix[0].empty())
            return {};
        int rows = matrix.size(),cols = matrix[0].size();
        std::vector<std::vector<bool>> visited(rows,std::vector<bool>(cols,false));

        std::vector<int> res;
        int dx[4] = {0,1,0,-1},dy[4] = {1,0,-1,0}; 
        int x = 0,y = 0,d = 0;
        for(int i = 0;i < rows*cols;i++)
        {
            res.push_back(matrix[x][y]);
            visited[x][y] = true;
            int a = x + dx[d],b = y + dy[d];
            if(a < 0 || a>= rows || b < 0 || b >= cols || visited[a][b])
            {
                d = (d + 1) % 4;
                 a = x + dx[d],b = y + dy[d];
            }
            x = a,y = b;
        }

        return res;
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
		long i = 1; //@ 遍历的位数
		while(n / i)
		{
			long high = n / (10 * i);  //@ 高位数值 
			long curr = (n / i) % 10;  //@ 当前位数值
			long low = n - (n / i) * i; //@ 低位数值
			if(curr == 0)
				res += high * i;
			else if(curr == 1)
				res += high * i + low + 1;
			else
				res += high * i + i;
			i *= 10;  //@ 位数增加
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
        int i = 1;
        while(n > 0.9 * pow(10,i) * i)
        {
            n -= 0.9 * pow(10,i) * i;
            i++;
        }

        std::string res = std::to_string(pow(10,i-1) + (n -1) / i);
        return res[(n-1)%i] - '0';
    }
};
```





# [49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)

```
class Solution {
public:
    int nthUglyNumber(int n) {
        std::vector<int> dp(n,1);
        int i = 0,j = 0,k = 0;
        for(int m = 1;m < n ;m++)
        {
            int n2 = dp[i] * 2,n3 = dp[j] * 3,n5 = dp[k] * 5;
            dp[m] = min(n2,min(n3,n5));
            if(dp[m] == n2)
                i++;
            if(dp[m] == n3)
                j++;
            if(dp[m] == n5)
                k++;
        }
        return dp.back();
    }
};
```

# [51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

```
class Solution {
public:
    int reversePairs(vector<int>& nums) {
        vector<int> tmp(nums.size());
        return mergeSort(0, nums.size() - 1, nums, tmp);
    }
private:
    int mergeSort(int l, int r, vector<int>& nums, vector<int>& tmp) {
        // 终止条件
        if (l >= r) return 0;
        // 递归划分
        int m = (l + r) / 2;
        int res = mergeSort(l, m, nums, tmp) + mergeSort(m + 1, r, nums, tmp);
        // 合并阶段
        int i = l, j = m + 1;
        for (int k = l; k <= r; k++)
            tmp[k] = nums[k];
        for (int k = l; k <= r; k++) {
            if (i == m + 1)
                nums[k] = tmp[j++];
            else if (j == r + 1 || tmp[i] <= tmp[j])
                nums[k] = tmp[i++];
            else {
                nums[k] = tmp[j++];
                res += m - i + 1; // 统计逆序对
            }
        }
        return res;
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


# [66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

```
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        int size = a.size();
        std::vector<int> res(size,1);
        for(int i = 1;i < size;i++)
            res[i] = res[i-1] * a[i-1];

        int tmp = 1;
        for(int i = size-2;i >= 0;i--)
        {
            tmp *= a[i+1];
            res[i] *= tmp;
        }
        return res;
    }
};
```

