# [ 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

```
class Solution {
public:
    string replaceSpace(string s) {
        std::string res;
        for(const auto& c : s)
        {
            if(c == ' ')
                res += "%20";
            else
                res += c;
        }
        return res;
    }
};
```

# [38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

```
class Solution {
public:
    inline bool occur(string &s, int left, int right) 
    {
        for(int i=left; i<right; ++i) 
        {
            if(s[i]==s[right])
                return true;
        }
        return false;
    }

    void dfs(string &s, int id, vector<string> &result) {
        if(id==s.size()) 
            result.push_back(s);
        else 
        {
            for(int i=id; i<s.size(); ++i) 
            {
                if(!occur(s, id, i)) 
                {
                    swap(s[id], s[i]);
                    dfs(s, id+1, result);
                    swap(s[id], s[i]);
                }
            }
        }
    }

    vector<string> permutation(string s) 
    {
        vector<string> res;
        dfs(s, 0, res);
        return res;
    }
};
```


# [50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

```
class Solution {
public:
    char firstUniqChar(string s) {
        std::unordered_map<int,int> hash;
        for(const auto c : s)
            hash[c] ++;
        for(const auto c: s)
        {
            if(hash[c] == 1)
                return c;
        }
        return ' ';
    }
};
```

# [58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

```
class Solution {
public:
    string reverseWords(string s) {
        if(s.empty())
            return "";
        
        std::string res;
        int len = 0;
        for(int i = s.length() - 1; i >= 0;i--)
        {
            if(s[i] == ' ' && len)
            {
                res += s.substr(i+1,len) + ' ';
                len = 0;
                continue;
            }

            if(s[i] != ' ')
                len++;
        }

        if(len)
            res += (s.substr(0,len));
        
        while(res.back() == ' ')
            res.pop_back();
        return res;
    }
};
```

# [58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

```
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        std::reverse(s.begin(),s.begin()+n);
        std::reverse(s.begin()+n,s.end());
        std::reverse(s.begin(),s.end());
        return s;
    }
};
```

```
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        s += s;
        return s.substr(n,s.length()/2);
    }
};
```

# [67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

```
class Solution {
public:
    int strToInt(string str) {
		if(str.empty())
			return 0;
			
		int i = 0;
		while(str[i] == ' ')
			i++;
		int sign = 1;
		if(str[i] == '-')
			sign = -1;
		if(str[i] == '-' || str[i] == '+') 
			i++;
		long res = 0;
		for(;i < str.size() && isdigit(str[i]);i++)
		{
			res = res * 10 + str[i] - '0';
			if(res > INT_MAX && sign == 1)
				return INT_MAX;
			if(res > INT_MAX && sign == -1)
				return INT_MIN;
		}
		return (int)(res * sign);
    }
};
```

