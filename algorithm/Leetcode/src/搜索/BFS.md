# 概述

BFS 算法本质上就是从一个图的起点出发开始搜索找到目标终点完成搜索。

BFS 的整个解决分成下边几个步骤：

- 起点入队列
- 以队列非空为循环条件，进行节点扩散（将所有队列节点出队（同时判断出队节点是否为目标节点），获取其邻接结点）
- 判断获取的节点是否已被遍历，未被遍历节点入队

模板：

```
int BFS(start,target){
     Queue q; 
     Set visited: 
     int step = 0; 

     q.add(start);
     visited.add(start);
     while(q not empty) {
         int sz = q.size();

         for(int i =0 ; i < sz; i++) 
		 {
             cur = q.front();
			 q.pop();
             if(cur is target) {
                 return;
             }

             for(Node n:cur.adjs) 
			 {
                 if(n is not int visited) {
                     visitd.add(n);
                     q.add(n);
                 }
             }
         }
     }
 }
```

# [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

````
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        if(grid.empty())
            return 0;
		
		vector<vector<int>> dir{{-1,0},{1,0},{0,1},{0,-1}};    
        int res = 0;

        for(int i = 0;i < grid.size();i++)
        {
            for(int j = 0;j < grid[0].size();j++)
            {
                if(grid[i][j] == '1')
                {
                    grid[i][j] = '0';
                    queue<pair<int,int>> q;
                    q.push({i,j});
                    res++;
                    while(!q.empty())
                    {
                        auto front = q.front();
                        q.pop();
                        int row = front.first,col = front.second;
						for(const auto d : dir)
						{
							int x = row + d[0];
							int y = col + d[1];
							if(x >=0 && x <  grid.size() && y >=0 && y < grid[0].size() && grid[x][y] == '1')
							{
								q.push({x,y});
								grid[x][y] = '0';
							}
						}
                    }
                }             
            }
        }
        return res;
    }
};
````

# [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

```
class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        if(grid.empty())
            return 0;
		
		vector<vector<int>> dir{{-1,0},{1,0},{0,1},{0,-1}};    
        int res = 0;
        for(int i = 0;i < grid.size();i++)
        {
            for(int j = 0;j < grid[0].size();j++)
            {
                int curr = 0;
                if(grid[i][j] == 1)
                {
                    grid[i][j] = 0;
                    queue<pair<int,int>> q;
                    q.push({i,j});
                    curr++;   

                    while(!q.empty())
                    {
                        auto front = q.front();
                        q.pop();
                        int row = front.first,col = front.second;						
						for(const auto d : dir)
						{
							int x = row + d[0];
							int y = col + d[1];
							if(x >=0 && x < grid.size() && y >=0 && y < grid[0].size() && grid[x][y] == 1)
							{
								q.push({x,y});
								grid[x][y] = 0;
								curr++;  
							}
						}                        
                    }  
                }
                res = max(res,curr);
            }
        }
        return res;
    }
};
```

# [542. 01 矩阵](https://leetcode-cn.com/problems/01-matrix/)

```
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        if(matrix.empty() || matrix[0].empty())
            return {};
        
        constexpr int dx[4] = {1,0,-1,0};
        constexpr int dy[4] = {0,1,0,-1};
        int m = matrix.size(),n = matrix[0].size();
        vector<vector<int>> res(m,vector<int>(n,0)),visited(m,vector<int>(n,0));
        queue<pair<int,int>> q;
        for(int i = 0;i < m;i++)
        {
            for(int j = 0;j < n;j++)
            {
                if(matrix[i][j] == 0)
                {
                    q.emplace(i,j);
                    visited[i][j] = 1;
                }
            }
        }

        while(!q.empty())
        {
            auto it= q.front();
            q.pop();
            int row = it.first,col = it.second;
            for(int k = 0;k < 4;k++)
            {
                int x = row + dx[k];
                int y = col + dy[k];
                if(x >= 0 && x < m && y >= 0 && y < n && !visited[x][y])
                {
                    res[x][y] = res[row][col] + 1;
                    q.emplace(x,y);
                    visited[x][y] = 1;
                }
            }
        }
        return res;
    }
};
```

# [133. 克隆图](https://leetcode-cn.com/problems/clone-graph/)

```
class Solution {
public:
    Node* cloneGraph(Node* node) {
        if(node == nullptr)
			return node;
		unordered_map<Node*,Node*> visited;
		queue<Node*> q;
		q.push(node);
		
		visited[node] = new Node(node->val);
		while(!q.empty())
		{
			auto it = q.front();
			q.pop();
			for(const auto & neighbor : it->neighbors)
			{
				if(visited.find(neighbor) == visited.end())
				{
					visited[neighbor] = new Node(neighbor->val);
					q.push(neighbor);
				}
				visited[it]->neighbors.emplace_back(visited[neighbor]);
			}
		}
		return visited[node];
    }
};
```

# [785. 判断二分图](https://leetcode-cn.com/problems/is-graph-bipartite/)

```
class Solution {
public:
    bool isBipartite(vector<vector<int>>& graph) {
		if(graph.empty())
			return true;
		
		//@ -1 表示节点还没有染色，0 表示染成一种颜色，1表示染成另一种颜色
		vector<int> color(graph.size(),-1);
		for(int i = 0;i < graph.size();i++)
		{
			if(color[i] == -1) //@ 该节点尚未被染色
			{
				queue<int> q;
				q.push(i);
				color[i] = 0; //@ 先染成某种颜色
				
				while(!q.empty())
				{
					int node = q.front();
					q.pop();
					int c_neighbor = color[node] == 0 ? 1 : 0;
					for(const auto n : graph[node])
					{
						if(color[n] == -1) //@ 没有被染色
						{
							q.push(n);
							color[n] = c_neighbor;
						}
						else if(color[n] != c_neighbor) //@ 被染色过，但不是正确的颜色
							return false;
					}					
				}
			}			
		}
		return true;
    }
};
```

# [127. 单词接龙](https://leetcode-cn.com/problems/word-ladder/)

```
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {    
        unordered_set<string> dict(wordList.begin(), wordList.end());    
        if (dict.find(endWord) == dict.end()) 
			return 0; 
       
        queue<string> q;
        q.push(beginWord);
        unordered_map<string, int> visited; 
        visited.insert({beginWord, 1});

        while(!q.empty()) {
            string word = q.front();
            q.pop();
            int path = visited[word]; 
            for (int i = 0; i < word.size(); i++) 
            {
                string new_word = word; 
                for (int j = 0 ; j < 26; j++) 
                {
                    new_word[i] = j + 'a';
                    if (new_word == endWord) 
						return path + 1; 

                    if (dict.find(new_word) != dict.end() && visited.find(new_word) == visited.end()) 
                    {
                        
                        visited.insert({new_word, path + 1});
                        q.push(new_word);
                    }
                }
            }
        }
        return 0;
    }
};
```









