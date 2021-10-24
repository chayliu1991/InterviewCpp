# 590. N叉树的后序遍历

[590. N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)

递归：

```
class Solution {
public:
	vector<int> res;
    vector<int> postorder(Node* root) {
        if(root == nullptr)
			return res;
		for(auto i : root -> children)
            postorder(i);
        res.push_back(root -> val);
        return res;
    }
};
```

迭代：

```
class Solution {
public:
    vector<int> postorder(Node* root) {
        if(root == nullptr)
			return {};
		vector<int> res;
		stack<Node*> stk;
		stk.push(root);
		while(!stk.empty())
		{
			Node* t = stk.top();
			stk.pop();
			res.push_back(t->val);
			for(auto i : t->children)
				stk.push(i);			
		}
		reverse(res.begin(),res.end());
		return res;
    }
};
```

# 589. N叉树的前序遍历

[589. N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)

递归：

```
class Solution {
public:
	vector<int> res;
    vector<int> preorder(Node* root) {
        if(root == nullptr)
			return res;
		res.push_back(root->val);
		for(auto i : root->children)
			preorder(i);
		return res;
    }
};
```

迭代：

```
class Solution {
public:
    vector<int> preorder(Node* root) {
        vector<int> res;
		if(root == nullptr)
			return res;
		stack<Node*> stk;
		stk.push(root);
		while(!stk.empty())
		{
			auto t = stk.top();
			stk.pop();
            res.push_back(t->val);
			for(int i = t->children.size()-1;i>=0;--i)
				stk.push(t->children[i]);
		}
		return res;
    }
};
```

# 559. N叉树的最大深度

[559. N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

递归：

```
class Solution {
public:
    
    int maxDepth(Node* root) {
        if(root == nullptr)   
			return 0;
        int depth = 0;
        for(const auto i : root -> children)
            depth = max(maxDepth(i), depth);            
        return depth + 1;
    }
};
```

DFS：

```
class Solution {
public:
    int maxDepth(Node* root) {
        if(root == nullptr)
			return 0;
		stack<pair<Node*,int>> stk;
		stk.push(make_pair(root,1));
		int depth = 1;
		while(!stk.empty())
		{
			auto t = stk.top();
			stk.pop();
			int dep = t.second;
			for(const auto i : t.first->children)
				stk.push(make_pair(i,dep+1));
			depth = max(depth,dep);		
		}
		return depth;
    }
};
```

BFS：

```
class Solution {
public:
    int maxDepth(Node* root) {
        if(root == nullptr)
			return 0;
		
		queue<Node*> q;
		q.push(root);
		int depth = 0;
		while(!q.empty())
		{
			depth++;
			int size = q.size();
			for(int i = 0;i<size;i++)
			{
				auto t = q.front();
				q.pop();
				for(const auto i : t->children)
					q.push(i);
			}			
		}
		return depth;
    }
};
```

# 429. N叉树的层序遍历

[429. N叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

```
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> res;
		if(root == nullptr)
			return res;
		queue<Node*> q;
		q.push(root);
		while(!q.empty())
		{
			vector<int> level;
			int size = q.size();
			for(int i=0;i<size;i++)
			{
				auto t = q.front();
				q.pop();
				level.push_back(t->val);
				for(const auto i : t->children)
					q.push(i);
			}
			res.push_back(move(level));
		}		
		return res;
    }
};
```

