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

# [76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

```
class Solution {
public:
    string minWindow(string s, string t) {
        if(s.length() < t.length())
            return "";
        
        std::vector<int> needs(128,0),windows(128,0);
        int left = 0,right = 0;
        int match_len = 0,min_len = INT_MAX;
        std::string res;
        for(auto c : t)
            needs[c]++;
        
        while(right < s.length())
        {
            char rc = s[right];
            windows[rc]++;

            if(needs[rc] && windows[rc] <= needs[rc])
                match_len++;
            
            while(left <= right && match_len == t.length())
            {
                if(min_len > right - left + 1)
                {
                    min_len = right - left + 1;
                    res = s.substr(left,min_len);
                }

                char lc = s[left];
                windows[lc]--;
                if(needs[lc] && windows[lc] < needs[lc])
                    match_len--;
                left++;
            }
            ++right;
        }
        return res;
    }
};
```

# [633. 平方数之和](https://leetcode-cn.com/problems/sum-of-square-numbers/)

```
class Solution {
public:
    bool judgeSquareSum(int c) {
        long left = 0,right = sqrt(c);
        while(left <= right)
        {
            long square = left * left + right * right;
            if(square == c)
                return true;
            else if(square > c)
                right--;
            else
                left++;
        }
        return false;
    }
};
```

# [680. 验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/)

```
class Solution {
public:
    bool validPalindrome(string s) {
        int left = 0,right = s.length() -1;
        while(left < right)
        {
            if(s[left] == s[right])
                left++,right--;
            else
                return valid(s,left+1,right) || valid(s,left,right-1);
        }
        return true;
    }

    bool valid(const std::string& str,int i,int j)
    {
        while(i <= j)
        {
            if(str[i++] != str[j--])
                return false;
        }
        return true;
    }
};
```



