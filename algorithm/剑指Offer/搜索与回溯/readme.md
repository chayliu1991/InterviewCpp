# [12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

```
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(word.empty() || board.empty() || board[0].empty()) 
            return false;
        for(int i=0; i<board.size(); ++i)
        {
            for(int j=0; j<board[0].size(); ++j)
            {
                if(dfs(board, word, i, j, 0)) 
					return true;
            }
        }
        return false;
    }
	
    bool dfs(vector<vector<char>>& board, string& word, int i, int j, int curr)
    {
        if(i < 0 || i >= board.size() || j < 0 || j >= board[0].size() || board[i][j] != word[curr]) 
			return false;
        if(curr == word.length() - 1) 
			return true;
        char temp = board[i][j]; 
        board[i][j] = '\0';
        if(dfs(board,word,i-1,j,curr+1) || dfs(board,word,i+1,j,curr+1) || dfs(board,word,i,j-1,curr+1) || dfs(board,word,i,j+1,curr+1))
            return true;

		board[i][j] = temp; 
        return false;
    }
};
```

# [26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

- 递归

```
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if(A == nullptr || B == nullptr)
            return false;
        return AhasB(A,B) || isSubStructure(A->left,B) || isSubStructure(A->right,B);
    }

    bool AhasB(TreeNode* a,TreeNode* b)
    {
        if(b == nullptr)
            return true;
        if(a == nullptr)
            return false;
        if(a->val != b->val)
            return false;
        
        return AhasB(a->left,b->left) && AhasB(a->right,b->right);
    }
};
```

- bfs+dfs

```
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if(A == nullptr || B == nullptr)
            return false;
        
        std::queue<TreeNode*> q;
        q.push(A);
        while(!q.empty())
        {
            int n = q.size();
            while(n--)
            {
                TreeNode* node = q.front();
                q.pop();
                if(node->val == B->val && dfs(node,B))
                    return true;
                if(node->left)
                    q.push(node->left);
                if(node->right)
                    q.push(node->right);
            }            
        }
        return false;
    }

    bool dfs(TreeNode* a,TreeNode* b)
    {
        if(b == nullptr)
            return true;
        if(a == nullptr)
            return false;
        if(a->val != b->val)
            return false;
        return dfs(a->left,b->left) && dfs(a->right,b->right);
    }
};
```

# [27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

- 递归

```
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(root == nullptr)
            return nullptr;
        
        TreeNode* left = mirrorTree(root->left);
        TreeNode* right = mirrorTree(root->right);

        root->left = right;
        root->right = left;
        return root;
    }
};
```

- dfs

```
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(root == nullptr)
            return nullptr;
        dfs(root);
        return root;
    }

    void dfs(TreeNode* root)
    {
        if(root == nullptr || (root->left == nullptr && root->right == nullptr))
            return;
        
        std::swap(root->left,root->right);
        dfs(root->left);
        dfs(root->right);
    }
};
```

- bfs

```
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(root == nullptr)
            return nullptr;
        bfs(root);
        return root;
    }

    void bfs(TreeNode* root)
    {
        if(root == nullptr || (root->left == nullptr && root->right == nullptr))
            return;
        
        std::queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
            TreeNode* node = q.front();
            q.pop();
            std::swap(node->left,node->right);

            if(node->left)
                q.push(node->left);
            if(node->right)
                q.push(node->right);
        }
    }
};
```

# [28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

- bfs

```
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == nullptr)
            return true;
        
        std::queue<TreeNode*> q;
        q.push(root->left);
        q.push(root->right);
        while(!q.empty())
        {
            if(q.size() & 0x01)
                return false;
            
            TreeNode* A = q.front();q.pop();
            TreeNode* B = q.front();q.pop();
            if(A == nullptr && B== nullptr)
                continue;
            if((A == nullptr && B != nullptr) || (A != nullptr && B == nullptr))
                return false;
            if(A->val != B->val)
                return false;
            q.push(A->left);
            q.push(B->right);
            q.push(A->right);
            q.push(B->left);
        }
        return true;
    }
};
```

- dfs

```
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == nullptr)
            return true;
        return dfs(root->left,root->right);
    }

    bool dfs(TreeNode* a,TreeNode* b)
    {
        if(a == nullptr && b == nullptr)
            return true;
        if((a == nullptr && b != nullptr) || (a != nullptr && b == nullptr))
            return false;
        if(a->val != b->val)
            return false;
        return dfs(a->left,b->right) && dfs(a->right,b->left);
    }
};
```

# [32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

- 队列+bfs

```
class Solution {
public:
    vector<int> levelOrder(TreeNode* root) {
        if(root == nullptr)
            return {};
        
        vector<int> res;
        std::queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
           TreeNode* node = q.front();
           q.pop();
           res.push_back(node->val);
           if(node->left) 
                q.push(node->left);
           if(node->right)
                q.push(node->right);
        }
        return res;
    }
};
```

# [32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

- 队列+bfs

```
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if(root == nullptr)
            return {};
        
        vector<vector<int>> res;
        std::queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
            int n = q.size();
            vector<int> tmp; 
            while(n--)
            {
               TreeNode* node = q.front();
               q.pop();
               tmp.push_back(node->val);
               if(node->left)
                    q.push(node->left);
               if(node->right)
                    q.push(node->right);
            }
            res.push_back(tmp);
        }
        return res;
    }
};
```

- dfs

```
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        std::vector<std::vector<int>> res;
        dfs(root,res,0);
        return res;
    }

    void dfs(TreeNode* root,std::vector<std::vector<int>>& vec,int level)
    {
        if(root == nullptr)
            return;
        if(level >= vec.size())
            vec.push_back(std::vector<int>{});
        vec[level].push_back(root->val);
        dfs(root->left,vec,level+1);
        dfs(root->right,vec,level+1);
    }
};
```

# [32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

- 队列+bfs

```
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if(root == nullptr)
            return res;

        bool rever = false;
        std::queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
            int n = q.size();
            std::vector<int> tmp;
            while(n--)
            {
                TreeNode* node = q.front();
                q.pop();
                tmp.push_back(node->val);
                if(node->left)
                    q.push(node->left);
                if(node->right)
                    q.push(node->right);
            }
            if(rever)
                std::reverse(tmp.begin(),tmp.end());
            rever = !rever;
            res.push_back(tmp);
        }
        return res;
    }
};
```

# [34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

```
class Solution {
public:
    vector<vector<int>> pathSum(TreeNode* root, int target) {
		if(root == nullptr)
			return {};
		
		vector<vector<int>> res;	
		vector<int> path;
		dfs(root,target,res,path);
		return res;
    }
	
	void dfs(TreeNode* root, int left,vector<vector<int>>& res,vector<int>& path)
	{	
		if(root == nullptr)
			return;
		path.push_back(root->val);
		left -= root->val;
		if(root->left == nullptr && root->right == nullptr && left == 0)
			res.push_back(path);
		dfs(root->left,left,res,path);
		dfs(root->right,left,res,path);
		path.pop_back();
	}
};
```

# [36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

```
class Solution {
public:
	void inorder(Node* root,std::vector<Node*>& res)
	{
		if(root == nullptr)
			return;
		inorder(root->left,res);
		res.push_back(root);
		inorder(root->right,res);
	}
	
    Node* treeToDoublyList(Node* root) {
		if(root == nullptr)
			return nullptr;
		
		std::vector<Node*> vec;
		inorder(root,vec);
		for(int i = 1;i < vec.size();i++)
		{
			vec[i-1]->right = vec[i];
			vec[i]->left = vec[i-1];
		}
		vec.back()->right = vec.front();
		vec.front()->left = vec.back();
		
		return vec.front();
    }
};

```

# [54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

```
class Solution {
public:
    void inorder(TreeNode* root,std::vector<int>& vec)
    {
        if(root == nullptr)
            return;
        inorder(root->left,vec);
        vec.push_back(root->val);
        inorder(root->right,vec);
    } 

    int kthLargest(TreeNode* root, int k) {
        std::vector<int> vec;
        inorder(root,vec);

        if(vec.size() < k)
            return -1;
        std::reverse(vec.begin(),vec.end());
        return vec[k-1];
    }
};
```

# [55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

```
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == nullptr)
            return 0;
        return max(maxDepth(root->left),maxDepth(root->right)) + 1;
    }
};
```

# [55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

```
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        if(root == nullptr)
            return true;
        int ld = height(root->left);
        int rd = height(root->right);

        return abs(ld - rd) <= 1 && isBalanced(root->left) && isBalanced(root->right);
    }

    int height(TreeNode* root)
    {
        if(root == nullptr)
            return 0;
        return max(height(root->left),height(root->right)) + 1;
    }
};
```