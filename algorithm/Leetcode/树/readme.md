

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

