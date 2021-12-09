# [374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)

```
class Solution {
public:
    int guessNumber(int n) {
        int left = 1,right = n;
        while(left <= right)
        {
            int mid = left + ((right - left) >> 1);
            int res = guess(mid);
            if(res == 0)
                return mid;
            else if(res == -1)
                right = mid - 1;
            else
                left = mid + 1;
        }
        return -1;
    }
};
```

# [69. Sqrt(x)](https://leetcode-cn.com/problems/sqrtx/)

```
class Solution {
public:
    int mySqrt(int x) {
        if(x <= 1)
            return x;
        
        int left = 1,right = x;
        while(left <= right)
        {
            int mid = left + ((right - left) >> 1);
            if(mid > x / mid)
                right = mid - 1;
            else
            {
                if((mid + 1) > x / (mid + 1))
                    return mid;
                else    
                    left = mid + 1;
            }
        }
        return -1;
    }
};
```

# [441. 排列硬币](https://leetcode-cn.com/problems/arranging-coins/)

```
class Solution {
public:
    int arrangeCoins(int n) {
        int left = 0,right = n;
        while(left <= right)
        {
            long mid = left + ((right - left) >> 1);
            long sum = mid * (mid + 1) >> 1;

            if(sum == n)
                return mid;
            else if(sum > n)
                right = mid - 1;
            else
                left = mid + 1;
        }
        return left - 1;
    }
};
```

# [367. 有效的完全平方数](https://leetcode-cn.com/problems/valid-perfect-square/)

```
class Solution {
public:
    bool isPerfectSquare(int num) {
        int left = 0,right = num;
        while(left <= right)
        {
            long mid = left + ((right - left) >> 1);
            long square = mid * mid;
            if(square == num)
                return true;
            else if(square > num)
                right = mid - 1;
            else
                left = mid + 1;
        }
        return false;
    }
};
```

# [278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/)

```
class Solution {
public:
    int firstBadVersion(int n) {
        int left = 1,right = n;
        while(left < right)
        {
            int mid = left + ((right - left) >> 1);
            bool bad = isBadVersion(mid);
            if(bad)
                right = mid;
            else
                left = mid + 1;
        }
        return isBadVersion(right) ? right : -1;
    }
};
```

# [162. 寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

```
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int left = 0,right = nums.size() - 1;
        while(left < right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] > nums[mid + 1])
                right = mid;
            else
                left = mid + 1;
        }
        return left;
    }
};
```

# [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

```
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        if(nums.empty())
            return {-1,-1};
        int left = lower_bound(nums,target);
        int right = upper_bound(nums,target);
        if(left == -1 || right == -1)
            return {-1,-1};
        return {left,right};        
    }

    int lower_bound(vector<int>& nums, int target)
    {
        int left = 0,right = nums.size()-1;
        while(left < right)
        {
            int mid = left + ((right -left) >> 1);
            if(nums[mid] < target)
                left = mid + 1;
            else
                right = mid;
        }
        return nums[left] == target ? left : -1;
    }

    int upper_bound(vector<int>& nums, int target)
    {
        int left = 0,right = nums.size()-1;
        while(left < right)
        {
            int mid = left + ((right -left + 1) >> 1);
            if(nums[mid] > target)
                right = mid - 1;
            else
                left = mid;
        }
        return nums[right] == target ? right : -1;
    }
};
```

# [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

```
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0,right = nums.size()-1;
        while(left <= right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] == target)
                return mid;
                            
            if(nums[left] <= nums[mid])
            {
                if(nums[left] <= target && nums[mid] > target)
                    right = mid - 1;
                else
                    left = mid + 1;
            }
            else
            {
                if(nums[right] >= target && nums[mid] < target)
                    left = mid + 1;
                else
                    right = mid - 1;
            }
        }
        return -1;
    }
};
```

# [81. 搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)

```
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int left = 0,right = nums.size() -1;
        while(left <= right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] == target)
                return true;
            if(nums[left] == nums[mid])
                ++ left;
            else if(nums[mid] <= nums[right])
            {
                if(target > nums[mid] && target <= nums[right])
                    left = mid + 1;
                else
                    right = mid - 1;
            }
            else
            {
                if(target >= nums[left] && target < nums[mid])
                    right = mid - 1;
                else
                    left = mid + 1;
            }
        }
        return false;
    }
};
```

# [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

```
class Solution {
public:
    int findMin(vector<int>& nums) {
        if(nums.empty())
            return -1;

        int left = 0,right = nums.size()-1;
        while(left < right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] > nums[right])
                left = mid + 1;
            else if(nums[mid] < nums[right])
                right = mid;
            else
                right--;
        }
        return nums[left];
    }
};
```

# [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

```
class Solution {
public:
    int findMin(vector<int>& nums) {
        if(nums.empty())
            return -1;

        int left = 0,right = nums.size() - 1;
        while(left < right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] <= nums[right])
                right = mid;
            else
                left = mid + 1;
        }
        return nums[left];
    }
};
```

# [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

```
class Solution {
public:
    int findMin(vector<int>& nums) {
        if(nums.empty())
            return -1;

        int left = 0,right = nums.size()-1;
        while(left < right)
        {
            int mid = left + ((right - left) >> 1);
            if(nums[mid] > nums[right])
                left = mid + 1;
            else if(nums[mid] < nums[right])
                right = mid;
            else
                right--;
        }
        return nums[left];
    }
};
```

# [74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

```
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if(matrix.empty())
            return false;

        int rows = matrix.size(),cols = matrix[0].size();
        int first = 0,last = rows * cols;
        while(first < last)
        {
            int mid = first + ((last - first) >> 1);
            int value = matrix[mid / cols][mid % cols];
            if(value == target)
                return true;
            else if(value < target)
                first = mid + 1;
            else
                last = mid;
        }
        return false;
    }
};
```

# 二分查找总结

## 解题步骤

- 预处理：如果序列未排序，则先进行排序
- 二分查找：
  - 使用循环或递归将中间值元素与目标元素进行比较，将区间划分为两个子区间
  - 然后在符合条件的其中一个子区间内进行寻找，直至循环或递归结束
- 后处理：在循环或递归完成后，需要对剩余区间的元素中确定符合条件的元素

## 解题模板

### 标准二分查找

寻找等于 X 的任意位置：

```
int search(std::vector<int> nums,int target)
{
	int left = 0,right = nums.size() - 1;
	while(left <= right)  //@ 1,终止条件： left > right,即 left = right + 1
	{
		int mid = left + ((right - left) >> 1);
		if(nums[mid] == target)
			return mid;
		else if(nums[mid] > target)
			right = mid - 1;
		else
			left = mid + 1;
	}
	return -1;
}
```

### 寻找左边界

```
int search(std::vector<int> nums,int target)
{
	int left = 0,right = nums.size() - 1;
	while(left < right)  //@ 1,终止条件： left = right
	{
		int mid = left + ((right - left) >> 1);
		else if(nums[mid] >= target)
			right = mid;
		else
			left = mid + 1;
	}
	return nums[left] >= target ? left : -1;
}
```

### 寻找有边界

```
int search(std::vector<int> nums,int target)
{
	int left = 0,right = nums.size() - 1;
	while(left < right)  //@ 1,终止条件： left = right
	{
		//@ 向上取整，在left与right相差为1时，left可以取到right而不会造成死循环
		int mid = left + ((right - left + 1) >> 1);
		else if(nums[mid] <= target)
			left = mid;
		else
			right = mid - 1;
	}
	return nums[right] <= target ? right : -1;
}
```













