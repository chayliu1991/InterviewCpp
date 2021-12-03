# [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

```
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        std::unordered_set<int> hash(nums.begin(),nums.end());
        int res = 0;
        for(auto num : nums)
        {
            if(hash.find(num - 1) != hash.end())
                continue;
            int count = 1;
            while(hash.find(++num) != hash.end())
                count++;
            res = std::max(res,count);
        }
        return res;
    }
};
```

