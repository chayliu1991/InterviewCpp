# [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

```
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
		map<int,int> dict;
		for(int i = 0;i < nums.size();i++)
		{
			if(dict.find(target - nums[i]) != dict.end())
				return {dict[target - nums[i]],i};
			
			dict[nums[i]] = i;
		}
		return {};
    }
};
```

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

# [146. LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/)

```
class LRUCache {
	int cap_;
    list<pair<int, int>> record_;  //@ pair<key,value>
	using Iter = list<pair<int,int>>::iterator;	
    unordered_map<int, Iter> dict_;   
	
public:
    LRUCache(int capacity):cap_(capacity) {
    }
    
    int get(int key) {
		if(dict_.find(key) == dict_.end())
			return -1;
		int res = dict_[key]->second;
		record_.erase(dict_[key]);
		record_.push_front({key,res});
		dict_[key] = record_.begin();
		return res;
    }
    
    void put(int key, int value) {
		if(dict_.find(key) != dict_.end())		
			record_.erase(dict_[key]);
		
		record_.push_front({key,value});
		dict_[key] = record_.begin();
		
		if(record_.size() > cap_)
		{	
			dict_.erase(record_.back().first);
			record_.pop_back();
		}	
    }
};
```

# [380. 常数时间插入、删除和获取随机元素](https://leetcode-cn.com/problems/insert-delete-getrandom-o1/)

```
class RandomizedSet {
    vector<int> data_;
    unordered_map<int, int> dict_;
	
public:
    RandomizedSet() {}
    bool insert(int val) {
        if (dict_.find(val) != dict_.end()) 
			return false;
        data_.push_back(val);
        dict_[val] = data_.size() - 1;
        return true;
    }
	
    bool remove(int val) {
        if (dict_.find(val) == dict_.end())
			return false;
        int num = data_.back();
        dict_[num] = dict_[val];
        data_[dict_[num]] = num;
        data_.pop_back();
        dict_.erase(val);
        return true;
    }
	
    int getRandom() {
        return data_[rand() % data_.size()];
    }
};
```

# [205. 同构字符串](https://leetcode-cn.com/problems/isomorphic-strings/)

```
class Solution {
public:
    bool isIsomorphic(string s, string t) {
		vector<int> s_index(256,0),t_index(256,0);
		for(int i = 0;i < s.length();i++)
		{
			if(s_index[s[i]] != t_index[t[i]])
				return false;
			s_index[s[i]] = t_index[t[i]] = i + 1;
			
		}
		return true;
    }
};
```

# [347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

```
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int> dict;
        for(const auto num : nums)
            dict[num]++;
        
        using Data = std::pair<int,int>;
        vector<Data> vec(dict.begin(),dict.end());
        sort(vec.begin(),vec.end(),[](const Data& d1,const Data& d2){return d1.second > d2.second;});

        vector<int> res;
        for(int i = 0;i <k;i++)
            res.push_back(vec[i].first);
        return res;
    }
};
```



