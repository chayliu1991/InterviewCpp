# [15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

```
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int res = 0;
        while(n)
        {
            n &= (n-1);
            res++;
        }
        return res;
    }
};
```

# [56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

```
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        int sum = 0;
        for(const auto n : nums)
            sum ^= n;
        
        int mask = 1;
        while((mask & sum) == 0)
            mask <<= 1;

        int n1 = 0,n2 = 0;
        for(const auto n : nums)
        {
            if(mask & n)
                n1 ^= n;
            else
                n2 ^= n;
        }
        return {n1,n2};
    }
};
```

# [56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

```
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        std::unordered_map<int,int> hash;
        for(const auto n : nums)
        {
            hash[n]++;
            if(hash[n] == 3)
                hash.erase(n);
        }
        return hash.begin()->first;
    }
};
```

# [65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

```
class Solution {
public:
    int add(int a, int b) {
        while(b)
        {
            int tmp = a ^ b;
            int carry = (unsigned)(a&b) << 1;
            b = carry;
            a = tmp;
        }
        return a;
    }
};
```

