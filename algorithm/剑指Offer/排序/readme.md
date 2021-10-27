# [45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

```
class Solution {
public:
    string minNumber(vector<int>& nums) {
		if(nums.empty())
			return "";
		std::vector<std::string> strs;
		std::transform(nums.begin(), nums.end(), std::back_inserter(strs), [](const int x) {return std::to_string(x); });
		std::sort(strs.begin(),strs.end(),[](const std::string& s1,const std::string& s2){return s1+s2 < s2+s1;});
		std::string res;
		for(const auto& s: strs)
			res += s;
		return res;
    }
};
```

# [ 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

```
class Solution {
public:
    bool isStraight(vector<int>& nums) {
        std::sort(nums.begin(),nums.end());
        int min_index = 0;
        while(nums[min_index] == 0)
            min_index++;
        for(int i = min_index;i < nums.size()-1;i++)
        {
            if(nums[i] == nums[i+1])
                return false;
        }
        return nums[4] - nums[min_index] <= 4;
    }
};
```