# [448. 找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/)

```
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        for(const auto & num : nums)
        {
            int pos = std::abs(num) -1;
            if(nums[pos] > 0)
                nums[pos] = - nums[pos];
        }

        std::vector<int> res;
        for(int i = 0;i < nums.size();i++)
        {
            if(nums[i] > 0)
            {
                res.push_back(i+1);
            }
        }

        return res;
    }
};
```

# [48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)

```
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        //@ 水平翻转
        for(int i = 0;i < n /2;i++)
        {
            for(int j = 0;j < n;j++)
            {
                std::swap(matrix[i][j],matrix[n-i-1][j]);
            }
        }

        //@ 主对角线翻转
        for(int i = 0;i < n;i++)
        {
            for(int j = 0;j < i;j++)
            {
                std::swap(matrix[i][j],matrix[j][i]);
            }
        }
    }
};
```

```
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int temp = 0,n = matrix.size()-1;
        for(int row = 0;row <= n/2;row++)
        {
            for(int col = row;col < n-row;col++)
            {
                temp = matrix[col][n-row];
                matrix[col][n-row] = matrix[row][col];
                matrix[row][col] = matrix[n-col][row];
                matrix[n-col][row] = matrix[n-row][n-col];
                matrix[n-row][n-col] = temp;
            }
        }
    }
};
```

# [27. 移除元素](https://leetcode-cn.com/problems/remove-element/)

```
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int index = 0;
        for(int i = 0;i < nums.size();++i)
        {
            if(nums[i] != val)
                nums[index++] = nums[i];
        }
        return index;
    }
};
```

# [26. 删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.size() <= 1)
            return nums.size();
        int index = 1;
        for(int i = 1;i < nums.size();i++)
        {
            if(nums[index-1] != nums[i])
                nums[index++] = nums[i];
        }
        return index;
    }
};
```

# [80. 删除有序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

```
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.size() <= 2)
            return nums.size();
        
        int index = 2;
        for(int i = 2;i < nums.size();i++)
        {
            if(nums[i] != nums[index-2])
                nums[index++] = nums[i];
        }
        return index;
    }
};
```

# [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

```
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        std::unordered_map<int,int> hash;
        for(int i = 0;i < nums.size();i++)
        {
            if(hash.find(target-nums[i]) != hash.end())
                return {hash[target-nums[i]],i};
            hash[nums[i]] = i;
        }
        return {};
    }
};
```

# [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

```
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n = nums.size();
        if(n < 3)
            return {};
        
        std::sort(nums.begin(),nums.end());
        std::vector<std::vector<int>> res;
        for(int i = 0;i < n-2;i++)
        {
            if(nums[i] > 0)
                break;
            if(i > 0 && nums[i] == nums[i-1])
                continue;
            int left = i+1,right = n-1;
            int target = -nums[i];
            while(left < right)
            {
                int sum = nums[left] + nums[right];
                if(sum == target)
                {
                    std::vector<int> tmp{nums[i],nums[left],nums[right]};
                    res.push_back(tmp);
                    left++,right--;

                    while(left < right && nums[left] == nums[left-1])
                        left++;
                    while(left < right && nums[right] == nums[right+1])
                        right--;
                }
                else if(sum > target)
                    right--;
                else
                    left++;
            }
        }
        return res;
    }
};
```

# [566. 重塑矩阵](https://leetcode-cn.com/problems/reshape-the-matrix/)

```
class Solution {
public:
    vector<vector<int>> matrixReshape(vector<vector<int>>& mat, int r, int c) {
        int rows = mat.size(),cols = mat[0].size();
        if(rows * cols != r * c)
            return mat;
        
        std::vector<std::vector<int>> res(r,std::vector<int>(c,0));
        for(int i = 0;i < r*c;i++)
        {
            res[i/c][i%c] = mat[i/cols][i%cols];
        }
        return res;
    }
};
```

# [56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

```
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
		if(intervals.empty())
			return {};
		
		//@ 默认按照各个位置上元素的字典序依次比较大小
		sort(intervals.begin(),intervals.end());
		vector<vector<int>> res;
		for (int i = 0; i < intervals.size(); ++i) {
			int L = intervals[i][0],R = intervals[i][1];
			if(res.empty() || res.back()[1] < L)
				res.push_back({L,R});
			else
				res.back()[1] = max(res.back()[1],R);
		}
		return res;
    }
};
```

# [435. 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)

```
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
		if(intervals.empty())
			return 0;
		int res = 0;
		sort(intervals.begin(),intervals.end());
		int last = 0; //@ 上一个被保留的区间
		for(int i = 1;i < intervals.size();i++)
		{
			if(intervals[i][0] < intervals[last][1])
			{
				//@ 区间重叠，需要删除一个区间
				res++;
				//@ 被删除的区间选择区间末尾比较的，从而尽可能的避免重叠
				if(intervals[i][1] < intervals[last][1])
					last = i;
			}
			else
				last = i;
		}
		return res;
    }
};
```

# [452. 用最少数量的箭引爆气球](https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/)

```
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        if (points.empty()) 
			return 0;
        sort(points.begin(), points.end());
        int res = 1; //@ 先来一支箭
        int last = points[0][1]; //@ 上一只箭覆盖到的最远范围
        for (int i = 1; i < points.size(); ++i) 
        {
            if (points[i][0] <= last)
                last = min(last, points[i][1]);
            else 
            {
                ++res;
                last = points[i][1];
            }
        }
        return res;
    }
};
```

# [57. 插入区间](https://leetcode-cn.com/problems/insert-interval/)

```
class Solution {
public:
    vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
		intervals.push_back(newInterval);
		sort(intervals.begin(),intervals.end());
		vector<vector<int>> res;
		for(int i = 0;i < intervals.size();i++)
		{
			int L = intervals[i][0],R = intervals[i][1];
			if(res.empty() || res.back()[1] < L)
				res.push_back({L,R});
			else
				res.back()[1] = max(res.back()[1],R);			
		}		
		return res;
    }
};
```

