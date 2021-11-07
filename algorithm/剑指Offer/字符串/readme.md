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

# [20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

```
class Solution {
public:
    bool isNumber(string s) {
		while(!s.empty() && s[0] == ' ')
			s.erase(0,1);
		while(!s.empty() && s[s.length()-1] == ' ')
			s.erase(s.length()-1,1);
		
		if(s.empty())
			return false;
		
		bool dot = false,exp = false,num = false;
		int last = s.length()-1;
		for(int i =0;i<s.length();i++)
		{
			switch(s[i])
			{
				case ' ':
					return false;
				case '+':
				case '-':
					if((i > 0 && s[i-1] != 'E' && s[i-1] != 'e') || (i == last))
						return false;
					break;
				case 'E':
				case 'e':
					if(exp || i == last || !num)
						return false;
					exp = true;
                    break;
				case '.':
					if(dot || exp || (i == last && !num))
						return false;
					dot = true;
					break;
				default:
					if(s[i] >= '0' && s[i] <= '9')
						num = true;
					else
						return false;
					break;
			}			
		}		
		return true;
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
        
        if(i == str.length())
            return 0;

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

