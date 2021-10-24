# 概述

深度优先搜索（DFS）类似于树的先序遍历。在搜索时会尽可能的沿着一条所有路径进行搜索，直到该条路径上所有节点搜索完成，然后切换到另一条路径上进行搜索，直到图的所有节点全部都被遍历。

深度优先搜索整个过程可以分成如下步骤：

- 判断终止条件
- 对节点进行访问并加入到访问集合中
- 以当前节点的邻接结点为起点，通过递归向更深层次进行搜索

在实现上DFS一般使用递归或栈来实现。

模板：

```
 set visited;
 void DFS(start) {     
     if(shoud be end) //@ 结束条件
         return;     
     visited.add(start);
     //@ 递归向更深次进行遍历
     for(Node n:start.adjs) 
     {
         if(n is not visited)
             DFS(n);   
     }
 }
```

# [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

```
class Solution {
public:
    unordered_map<char,string> dict = {
        {'2', "abc"}, {'3', "def"}, {'4', "ghi"}, {'5', "jkl"}, {'6', "mno"},
        {'7', "pqrs"}, {'8', "tuv"}, {'9', "wxyz"}};

    vector<string> res;
    string tmp;

    void dfs(int index,string digits)
    {
        if(index == digits.size())
        {
            res.push_back(tmp);
            return;
        }
        
        for(int i = 0;i < dict[digits[index]].size();i++)
        {
            tmp.push_back(dict[digits[index]][i]);
            dfs(index+1,digits);
            tmp.pop_back();
        }
    }

    vector<string> letterCombinations(string digits) {
        if(digits.empty())
            return {};
        dfs(0,digits);
        return res;
    }
};
```

# [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

```
class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        if(grid.empty())
            return 0;
        int res  = 0;
        for(int i = 0;i < grid.size();i++)
        {
            for(int j = 0;j < grid[0].size();j++)
            {
                if(grid[i][j] == 1)
                    res = max(res,dfs(grid,i,j));
            }
        }
        return res;
    }

    int dfs(vector<vector<int>>& grid,int i,int j)
    {
        if(i < 0 || j <0 || i >= grid.size() || j >= grid[0].size() || grid[i][j] == 0)
            return 0;

        grid[i][j] = 0;
        int count = 1;
        vector<vector<int>> dir{{1,0},{-1,0},{0,1},{0,-1}};
        for(const auto d : dir)
            count += dfs(grid,i+d[0],j+d[1]);
        return count;
    }
};
```

# [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

```
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        if(grid.empty())
            return 0;
        int res  = 0;
        for(int i = 0;i < grid.size();i++)
        {
            for(int j = 0;j < grid[0].size();j++)
            {
                if(grid[i][j] == '1')
                {
                    res++;
                    dfs(grid,i,j);
                }
            }
        }
        return res;
    }

    void dfs(vector<vector<char>>& grid,int i,int j)
    {
        if(i < 0 || j < 0 || i >= grid.size() || j >= grid[0].size() || grid[i][j] == '0')
            return;
        grid[i][j] = '0';

        vector<vector<int>> dir{{1,0},{-1,0},{0,1},{0,-1}};
        for(const auto d : dir)
            dfs(grid,i+d[0],j+d[1]);
    }
};
```

# [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

```
class Solution {
public:
	void dfs(vector<vector<char>>& board,int i,int j)
	{
		if(i < 0 || j < 0 || i >= board.size() || j >= board[0].size() || board[i][j] != 'O')
			return;	
		
		board[i][j] = 'A';
		vector<vector<int>> dir{{1,0},{-1,0},{0,1},{0,-1}};
		for(const auto d : dir)
			dfs(board,i+d[0],j+d[1]);		
	}

    void solve(vector<vector<char>>& board) {
        if(board.empty())
			return;
		int m = board.size(),n = board[0].size();
		for(int i = 0;i < m;i++)
		{
			dfs(board,i,0);
			dfs(board,i,n-1);
		}
		
		for(int i = 1;i < n -1;i++)
		{
			dfs(board,0,i);
			dfs(board,m-1,i);
		}
		
		for(int i = 0;i < m;i++)
		{
			for(int j = 0;j < n;j++)
			{
				if(board[i][j] == 'A')
					board[i][j] = 'O';
				else if(board[i][j] == 'O')
					board[i][j] = 'X';
			}
		}		
    }
};
```

# [417. 太平洋大西洋水流问题](https://link.zhihu.com/?target=https%3A//leetcode-cn.com/problems/pacific-atlantic-water-flow/)

```
class Solution {
public:
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& matrix)
    {
        if (matrix.empty() || matrix[0].empty())
            return{};

        int m = matrix.size(), n = matrix[0].size();
        vector<vector<bool>> pacific(m, vector<bool>(n, false));
        vector<vector<bool>> atlantic(m, vector<bool>(n, false));

        vector<vector<int>> res;
        for (int i = 0; i < m; i++)
        {
            dfs(matrix, pacific, INT_MIN, i, 0);
            dfs(matrix, atlantic, INT_MIN, i, n - 1);
        }

        for (int i = 0; i < n; i++)
        {
            dfs(matrix, pacific, INT_MIN, 0, i);
            dfs(matrix, atlantic, INT_MIN, m - 1, i);
        }

        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (pacific[i][j] && atlantic[i][j])
                    res.push_back({ i,j });
            }
        }
        return res;
    }

    void dfs(vector<vector<int>>& matrix, vector<vector<bool>>& visited, int pre, int i, int j)
    {
        if (i < 0 || j < 0 || i >= matrix.size() || j >= matrix[0].size() || visited[i][j] || matrix[i][j] < pre)
            return;

        visited[i][j] = true;

        vector<vector<int>> dir{ { 1,0 },{ -1,0 },{ 0,1 },{ 0,-1 } };
        for (const auto d : dir)
            dfs(matrix, visited, matrix[i][j], i + d[0], j + d[1]);
    }	
};
```

# [463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)

```
class Solution {
public:
	int dfs(vector<vector<int>>& grid,int i,int j)
	{
		//@ 从陆地到海洋或者从陆地到边界，周长+1
		if(i < 0 || j <0 || i >= grid.size() || j >= grid[0].size() || grid[i][j] == 0)
			return 1;
		if (grid[i][j] == 2)
			return 0;
		grid[i][j] = 2; //@ 标记为已读
		int res = 0;
		
		vector<vector<int>> dir{{0,1},{0,-1},{1,0},{-1,0}};
		for(const auto d :dir)
			res += dfs(grid,i+d[0],j+d[1]);
		return res;
	}
	
    int islandPerimeter(vector<vector<int>>& grid) {
		if(grid.empty())
			return 0;
		
		int res  = 0;
		for(int i = 0;i < grid.size();i++)
		{
			for(int j = 0;j < grid[0].size();j++)
			{
				if(grid[i][j] == 1)
					res += dfs(grid,i,j);
			}
		}
		return res;
    }
};
```

# [22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

```
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        if(n <= 0)
            return {};
        
        vector<string>  res;
        dfs(res,"",n,0,0);
        return res;
    }

    void dfs(vector<string>& res,string path,int n,int lc,int rc) 
    {
        if(rc > lc || rc > n || lc > n)
            return;
        if(rc == lc && lc == n)
            res.push_back(path);
        dfs(res,path+'(',n,lc+1,rc);
        dfs(res,path+')',n,lc,rc+1);
    }
};
```



