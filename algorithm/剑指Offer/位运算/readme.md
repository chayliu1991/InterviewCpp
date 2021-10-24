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

