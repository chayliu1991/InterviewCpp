# [167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)

```
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        std::vector<int> res;
        int left = 0,right = numbers.size()-1;
        while(left < right)
        {
            if(numbers[left] + numbers[right] == target)
                return {left+1,right+1};
            else if(numbers[left] + numbers[right] > target)
                right--;
            else
                left++;
        }
        return res;
    }
};
```

# [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

```
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int i = m-1,j = n-1,last = m+n-1;

        //@ 条件应该包含0
		while(i >= 0 && j >= 0)
			nums1[last--] = nums1[i] >= nums2[j] ? nums1[i--] : nums2[j--];

		while(j >= 0)
			nums1[last--] = nums2[j--];
    }
};
```

