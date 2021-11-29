# [461. 汉明距离](https://leetcode-cn.com/problems/hamming-distance/)

```
class Solution {
public:
    int hammingDistance(int x, int y) {
        int res = 0;
        int diff = x ^ y;
        while(diff)
        {
            diff &= (diff-1);
            res ++;
        }
        return res;
    }
};
```

# [190. 颠倒二进制位](https://leetcode-cn.com/problems/reverse-bits/)

```
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        uint32_t res;
        for(uint32_t i = 0;i < 32;i++)
        {
            res = (res << 1) | (n & 1);
            n >>= 1;
        }
        return res;
    }
};
```

# [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

```
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for(const auto& num : nums)
        {
            res ^= num;
        }
        return res;
    }
};
```

# [342. 4的幂](https://leetcode-cn.com/problems/power-of-four/)

```
class Solution {
public:
    bool isPowerOfFour(int n) {
        return n > 0 && !(n & (n - 1)) && !((n - 1) % 3);
    }
};
```

# [318. 最大单词长度乘积](https://leetcode-cn.com/problems/maximum-product-of-word-lengths/)

```
class Solution {
public:
    int convert(const std::string& str)
    {
        int res = 0;
        for(auto c : str)
        {
            res |= 1 << (c - 'a');
        }
        return res;
    } 

    int maxProduct(vector<string>& words) {
        int n = words.size();
        std::vector<int> masks(n,0);
        for(int i = 0;i < n;i++)
        {
            masks[i] = convert(words[i]);
        }

        int res = 0;
        for(int i = 0;i < n;++i)
        {
            for(int j = 0;j < i;++j)
            {
                if((masks[i] & masks[j]) == 0 && words[i].length() * words[j].length() > res)
                {
                    res = words[i].length() * words[j].length();
                }
            }
        }
        return res;
    }
};
```

# [693. 交替位二进制数](https://leetcode-cn.com/problems/binary-number-with-alternating-bits/)

```
class Solution {
public:
    bool hasAlternatingBits(int n) {
        n = n ^ (n >> 1);
        return (n & (long(n) + 1)) == 0;
    }
};
```

