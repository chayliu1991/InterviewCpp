# [07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

```
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.size() != inorder.size() || preorder.empty())
            return nullptr;
        TreeNode* root = new TreeNode(preorder[0]);
        
        auto it = std::find(inorder.begin(),inorder.end(),preorder[0]);
        auto Linorder = std::vector<int>(inorder.begin(),it);
        auto Rinorder = std::vector<int>(it+1,inorder.end());
        auto Lsize = Linorder.size();
        auto RSize = Rinorder.size();
        auto Lpreorder = std::vector<int>(preorder.begin()+1,preorder.begin()+1+Lsize);
        auto Rpreorder = std::vector<int>(preorder.begin()+Lsize+1,preorder.end());
        
        root->left = buildTree(Lpreorder,Linorder);
        root->right = buildTree(Rpreorder,Rinorder);
        
        return root;
    }
};
```

# [33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

```
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        return dfs(postorder,0, postorder.size() - 1);
    }

    bool dfs(std::vector<int>& post,int left,int right)
    {
        if(left >= right)
            return true;
        int root = post[right];
        int j = 0;
        while(j < right && post[j] < root)
            j++;
        
        for(int i = j;i < right;i++)
        {
            if(post[i] < root)
                return false;
        }
        return dfs(post,left,j-1) && dfs(post,j,right-1);
    }
};
```



# [37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

```
class Codec {
public:

    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        std::ostringstream out;
		std::queue<TreeNode*> q;
		q.push(root);
		while(!q.empty())
		{
			auto node = q.front();
            q.pop();
			if(node != nullptr)
			{				
				out << node->val<<" ";
				q.push(node->left);
				q.push(node->right);
			}
			else
				out <<"null ";		
		}
		return out.str();
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        std::istringstream inPut(data);
		std::string val;
		std::vector<TreeNode*> vec;
		while(inPut >> val)
		{
			if(val == "null")
				vec.push_back(nullptr);
			else 
				vec.push_back(new TreeNode(stoi(val)));
		}
		
		//@ i每往后移动一位，j移动两位，j始终是当前i的左子下标
		int j = 1;
		for(int i = 0;j < vec.size();i++)
		{
			if(vec[i] == nullptr)
				continue;
            if (j < vec.size()) 
			    vec[i]->left = vec[j++];
			if(j < vec.size())
				vec[i]->right = vec[j++];
		}
		return vec.front();
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

# [68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
		if(root == nullptr || p == nullptr || q == nullptr)
			return nullptr;
		
		if(root->val > p->val && root->val > q->val)
			return lowestCommonAncestor(root->left,p,q);
		if(root->val < p->val && root->val < q->val)
			return lowestCommonAncestor(root->right,p,q);
		return root;
    }
};

```

# [ 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || p == root || q == root)
            return root;
        auto left = lowestCommonAncestor(root->left,p,q);
        auto right = lowestCommonAncestor(root->right,p,q);

        if(left && right)
            return root;
        return left ? left : right;
    }
};
```

