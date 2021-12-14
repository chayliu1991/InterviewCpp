

# [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

```
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        preorder(res,root);
        return res;
    }

    void preorder(vector<int>& vec,TreeNode* root)
    {
        if(root == nullptr)
            return;
        
        vec.emplace_back(root->val);
        preorder(vec,root->left);
        preorder(vec,root->right);
    }
};
```

```
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        if(root == nullptr)
            return {};
        
        std::vector<int> res;
        std::stack<TreeNode*> s;
        s.push(root);       
        while(!s.empty())
        {
            TreeNode* curr = s.top();
            s.pop();
            res.emplace_back(curr->val);
            if(curr->right)
                s.push(curr->right);
            if(curr->left)
                 s.push(curr->left);
        }
        return res;
    }
};
```

# [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

```
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        std::vector<int> res;
        inorder(res,root);
        return res;
    }

    void inorder(std::vector<int>& vec,TreeNode* root)
    {
        if(root == nullptr)
            return;
        inorder(vec,root->left);
        vec.emplace_back(root->val);
        inorder(vec,root->right);
    }
};
```

```
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        if(root == nullptr)
            return {};
        
        std::vector<int> res;
        std::stack<TreeNode*> s;
        TreeNode* curr = root;
        while(curr != nullptr || !s.empty())
        {
            if(curr)
            {
                s.push(curr);
                curr = curr->left;
            }
            else
            {
                curr = s.top();
                s.pop();
                res.emplace_back(curr->val);
                curr = curr->right;
            }
        }
        return res;
    }
};
```

# [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

```
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        if(root == nullptr)
            return {};
        std::vector<int> res;
        postorder(res,root);
        return res;
    }

    void postorder(std::vector<int>& vec,TreeNode* root)
    {
        if(root == nullptr)
            return;
        postorder(vec,root->left);
        postorder(vec,root->right);
        vec.emplace_back(root->val);
    }
};
```

```
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        if(root == nullptr)
            return {};
        
        std::vector<int> res;
        std::stack<TreeNode*> s;
        s.push(root);
        while(!s.empty())
        {
            TreeNode* curr = s.top();
            if(curr != nullptr)
            {
                s.push(nullptr);
                if (curr->right) 
                    s.push(curr->right);
                if (curr->left) 
                    s.push(curr->left);
            }
            else
            {
                s.pop();
                curr = s.top();
                s.pop();
                res.push_back(curr->val);
            }
        }
        return res;
    }
};
```

# [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

```
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if(root == nullptr)
            return {};
        std::vector<vector<int>> res;
        std::queue<TreeNode*> q;
        q.push(root);
        std::vector<int> level;
        while(!q.empty())
        {
           size_t n = q.size();
           for(size_t i = 0;i < n;++i)
           {
                auto curr = q.front();
                q.pop();
                level.emplace_back(curr->val);
                if(curr->left)
                    q.push(curr->left);
                if(curr->right)
                    q.push(curr->right); 
           }
           res.emplace_back(std::move(level));             
        }
        return res;
    }
};
```

















