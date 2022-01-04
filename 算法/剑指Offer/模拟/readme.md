# [29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if(matrix.empty() || matrix[0].empty())
            return {};
        int rows = matrix.size(),cols = matrix[0].size();
        int dx[] = {0,1,0,-1} ,dy[] = {1,0,-1,0};
        int x = 0,y = 0,d = 0;
        std::vector<std::vector<bool>> visited(rows,std::vector<bool>(cols,false));
        std::vector<int> res;
        for(int i = 0;i < rows* cols;i++)
        {
            res.push_back(matrix[x][y]);
            visited[x][y] = true;
            int a = x + dx[d],b = y + dy[d];
            if(a < 0 || a >= rows || b < 0 || b >= cols || visited[a][b] )
            {
                d = (d+1) % 4;
                a = x + dx[d],b = y + dy[d];
            }
            x = a,y = b;
        }
        return res;
    }
};
```

# [31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

```
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        if(popped.size() != pushed.size())
            return false;
        
        std::stack<int> sk;
        int j = 0,n = pushed.size();
        for(int i = 0;i < n;i++)
        {
            sk.push(pushed[i]);
            while(!sk.empty() && j < n && sk.top() == popped[j])
            {
                sk.pop();
                j++;
            }
        }
        return sk.empty();
    }
};
```

