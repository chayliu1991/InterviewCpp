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

