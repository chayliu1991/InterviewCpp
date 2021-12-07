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

