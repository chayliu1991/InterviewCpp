# 模板

BFS 算法本质上就是从一个图的起点出发开始搜索找到目标终点完成搜索。

BFS 的整个解决分成下边几个步骤：

- 起点入队列
- 以队列非空为循环条件，进行节点扩散（将所有队列节点出队（同时判断出队节点是否为目标节点），获取其邻接结点）
- 判断获取的节点是否已被遍历，未被遍历节点入队

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

