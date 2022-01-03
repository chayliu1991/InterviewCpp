# 模板

```
vector<vector<string>> result;
vector<string> path; 

void backtrace(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtrace(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```

# [77. 组合](https://leetcode-cn.com/problems/combinations/)

```
class Solution {
public:
	vector<vector<int>> res;
	vector<int> path;
	
	void backtrace(int n, int k,int curr_index)
	{	
		if(path.size() == k)
		{
			res.push_back(path);
			return;
		}
		
		for(int i = curr_index;i <= n;i++)
		{
			path.push_back(i);
			backtrace(n,k,i+1);
			path.pop_back();
		}
	}
		
    vector<vector<int>> combine(int n, int k) {
		backtrace(n,k,1);
		return res;
    }
};
```

# [78. 子集](https://leetcode-cn.com/problems/subsets/)

```
class Solution {
public:
	vector<vector<int>> res;
	vector<int> path;
	
	void backtrace(vector<int>& nums,int curr_index)
	{
		res.push_back(path);
		for(int i = curr_index;i < nums.size();i++)
		{
			path.push_back(nums[i]);
			backtrace(nums,i+1);
			path.pop_back();
		}		
	}
	
    vector<vector<int>> subsets(vector<int>& nums) {
		backtrace(nums,0);
		return res;
    }
};
```

# [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

```
class Solution {
public:
	vector<vector<int>> res;
	vector<int> path;
	
	void backtrace(vector<int>& candidates, int target,int cur_sum,int cur_index)
	{
		if(cur_sum > target)
			return;
		if(cur_sum == target)
		{
			res.push_back(path);
			return;
		}
		
		for(int i = cur_index;i < candidates.size();i++)
		{			
			cur_sum += candidates[i];
			path.push_back(candidates[i]);
			backtrace(candidates,target,cur_sum,i); //@ 不用i+1了，表示可以重复读取当前的数
			cur_sum -= candidates[i];
			path.pop_back();			
		}
	}
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        backtrace(candidates, target, 0, 0);
        return res;
    }
};
```

# [46. 全排列](https://leetcode-cn.com/problems/permutations/)

```
class Solution {
public:
    vector<vector<int>> result;
    vector<int> path;
	
    void backtracking (vector<int>& nums, vector<bool>& used) {
        if (path.size() == nums.size()) {
            result.push_back(path);
            return;
        }
        for (int i = 0; i < nums.size(); i++) {
            if (used[i] == true) 
				continue; //@ path里已经收录的元素，直接跳过
            used[i] = true;
            path.push_back(nums[i]);
            backtracking(nums, used);
            path.pop_back();
            used[i] = false;
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> used(nums.size(), false);
        backtracking(nums, used);
        return result;
    }
};
```

# [47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)

```
class Solution {
private:
    vector<vector<int>> res;
	vector<int> path;
	
	
    void backtrace (vector<int>& nums, vector<int>& path, vector<bool>& used) {
        if (path.size() == nums.size()) {
            res.push_back(path);
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            // 这里理解used[i - 1]非常重要 
            // used[i - 1] == true，说明同一树支nums[i - 1]使用过 
            // used[i - 1] == false，说明同一树层nums[i - 1]使用过
            // 如果同一树层nums[i - 1]使用过则直接跳过
            if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false) { 
                continue;
            }
            if (used[i] == false) {
                used[i] = true;
                path.push_back(nums[i]);
                backtrace(nums, path, used);
                path.pop_back();
                used[i] = false;
            }
        }
    }

public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<bool> used(nums.size(), false);        
        backtrace(nums, path, used);
        return res;

    }
};
```

# [79. 单词搜索](https://leetcode-cn.com/problems/word-search/)

```
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
		if(board.empty() || board[0].empty())
			return false;
		
		for(int i = 0;i < board.size();i++)
		{
			for(int j = 0;j < board[0].size();j++)
				if(backtrace(board,word,i,j,0))
					return true;
		}
		return false;
    }
	
	bool backtrace(vector<vector<char>>& board, string& word, int i, int j, int k)
	{
		if(board[i][j] != word[k])
			return false;
		if(k == word.size()-1)
			return true;
		
		board[i][j] = '.';
		vector<vector<int>> dir{{0,1},{1,0},{0,-1},{-1,0}};
		for(const auto d : dir)
		{
			int x = i + d[0],y = j + d[1];
			if(x >= 0 && y >= 0 && x < board.size() && y < board[0].size())
				if(backtrace(board,word,x,y,k+1))
					return true;
		}
		board[i][j] = word[k];
		return false;
	}
};
```

# [473. 火柴拼正方形](https://leetcode-cn.com/problems/matchsticks-to-square/)

```
class Solution {
public:
    bool makesquare(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end(), greater<int>()); 
        int sum = accumulate(nums.begin(), nums.end(), 0); 
        if (sum % 4 != 0 || n < 4) 
            return false;  
        vector<bool> vis(n, false);
        for (int i = 0; i < 4; i++)  //@ 四条边都搜索
            if (!backtrace(nums, vis, 0, sum/4)) return false;
        return true;
    }

     bool backtrace(vector<int> &nums, vector<bool> &visited, int curr_index, int target)
     {
        if(target == 0)
            return true;
        for(int i = curr_index;i < nums.size();i++)
        {
            if(nums[i] <= target && !visited[i])
            {
                visited[i] = true;
                if(backtrace(nums, visited, i+1, target-nums[i])) 
                    return true;  //@ 组合成功就返回，不需要继续搜索。
                visited[i] = false;  //@ 如果组合失败,回溯
            }
        }
        return false;
     }
};
```



