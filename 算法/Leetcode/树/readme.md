

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

# [103. 二叉树的锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

```
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if(root == nullptr)
            return {};
        vector<vector<int>> res;
        stack<TreeNode*> s1,s2;
        s1.push(root);
        bool l2r = true;
        vector<int> level;
        while(!s1.empty() || !s2.empty())
        {
            TreeNode* node = nullptr;
            if(l2r)
            {
                while(!s1.empty())
                {
                    node = s1.top();
                    s1.pop();
                    level.push_back(node->val);
                    if(node->left)
                        s2.push(node->left);
                    if(node->right)
                        s2.push(node->right);
                }
                l2r = false;
            }
            else
            {
                while(!s2.empty())
                {
                    node = s2.top();
                    s2.pop();
                    level.push_back(node->val);
                    if(node->right)
                        s1.push(node->right);
                    if(node->left)
                        s1.push(node->left);
                }   
                l2r = true;
            }
            res.push_back(move(level));                                   
        }
        return res;
    }
};
```

# [513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

```
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        if (nullptr == root)
            return -1;  

        int res = root->val;
        queue<TreeNode*> que;
        que.push(root);
        while (!que.empty())
        {
            res = que.front()->val;
            int len = que.size();             
            for(int i=0;i<len;++i)
            {
                if (que.front()->left)
                    que.push(que.front()->left);
                if (que.front()->right)
                    que.push(que.front()->right);
                que.pop();
            }
        }
        return res;
    }
};
```


# [538. 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

```
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        if(root == nullptr)
            return nullptr;
        
        stack<TreeNode*> stk;
        auto p = root;
        int sum = 0;
        while(true)
        {
            if(p)
            {
                stk.push(p);
                p = p->right;
            }
            else if(!stk.empty())
            {
                p = stk.top();
                stk.pop();
                p->val += sum;
                sum = p->val;
                p = p->left;
            }
            else
                break;
        }
        return root;
    }
};
```

# [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

```
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
		vector<int>  res;
		if(root == nullptr)
			return res;
		queue<TreeNode*> q;
		q.push(root);
		while(!q.empty())
		{
			int size = q.size();
			for(int i = 0;i < size;++i)
			{
				auto t = q.front();
				q.pop();
				if(t->left)
					q.push(t->left);
				if(t->right)
					q.push(t->right);
				if(i == size -1)
					res.push_back(t->val);
			}
		}
		return res;
    }
};
```

# [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

```
class Solution {
public:
    long long prev = LLONG_MIN;
    bool validBST(TreeNode* root)
    {
        if(root == nullptr)
            return true;
        if(!validBST(root->left))
            return false;
        if(root->val <= prev)
            return false;
        prev = root->val;
        return validBST(root->right);
    }

    bool isValidBST(TreeNode* root) {
        return validBST(root);
    }
};
```

# [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

递归解法：

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

非递归解法：

```
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == nullptr)
            return 0;
        int res = 0;
        queue<TreeNode* > q;
        q.push(root);
        while(!q.empty())
        {
            res ++;
            int size = q.size();
            for(int i=0;i<size;i++)
            {
                TreeNode* node = q.front();
                q.pop();
                if(node->left)
                    q.push(node->left);
                if(node->right)
                    q.push(node->right);
            }
        }
        return res;
    }
};
```

# [110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

```
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        if(root == nullptr)
            return true;
        int left = _depth(root->left);
        int right = _depth(root->right);
        if(abs(left-right) > 1)
            return false;
        return isBalanced(root->left) && isBalanced(root->right);
    }

    int _depth(TreeNode* root)
    {
        if(root == nullptr)
            return 0;  
        return  max(_depth(root->left),_depth(root->right)) + 1;
    }
};
```

# [543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

```
class Solution {
public:
    int _depth(TreeNode* root,int& diameter)
    {
        if(root == nullptr)
            return 0;
        int L = _depth(root->left,diameter);
        int R = _depth(root->right,diameter);
        diameter = max(L+R,diameter);
        return max(L,R) + 1;        
    }

    int diameterOfBinaryTree(TreeNode* root) {
        int diameter = 0;
        _depth(root,diameter);
        return diameter;
    }
};
```

# [230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

中序遍历：

```
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        if(root == nullptr)
            return -1;
		vector<int> res;
		inorder(root,res);
        return res.size() >=k ?  res[k-1] : -1;		 
    }
	
	void inorder(TreeNode* root,vector<int>& res)
	{
		if(root == nullptr)
			return;
		inorder(root->left,res);
		res.push_back(root->val);
		inorder(root->right,res);
	}
};
```

二分法：

```
class Solution {
public:
    int numofNodes(struct TreeNode* root)
    {
        if(root == nullptr) 
            return 0;
        return 1 + numofNodes(root->left) + numofNodes(root->right);
    }

    int kthSmallest(struct TreeNode* root, int k){
        int cnt = numofNodes(root->left);
        if(k <= cnt) 
            return kthSmallest(root->left,k);
        else if(k > cnt+1) 
            return kthSmallest(root->right,k-cnt-1);
        return root->val; // k == cnt+ 1
    }
};
```

# [700. 二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

递归：

```
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
		if(root == nullptr)
			return nullptr;
		if(root->val == val)
			return root;
		else if(root->val > val)
			return searchBST(root->left,val);
		else
			return searchBST(root->right,val);
    }
};
```

迭代：

```
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        while(root && root->val !=  val)
            root->val > val ? root = root->left : root = root->right;
        return root;
    }
};
```

栈迭代：

```
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        stack<TreeNode*> stk;
        if(root)
            stk.push(root);
        while(!stk.empty())
        {
            auto t = stk.top();
            stk.pop();
            if(t->val == val)
                return t;
            else if(t->val > val && t->left)
                stk.push(t->left);
            else if(t->val < val && t->right)  
                stk.push(t->right);
        }
        return nullptr;
    }
};
```

# [897. 递增顺序查找树](https://leetcode-cn.com/problems/increasing-order-search-tree/)

```
class Solution {
public:
	vector<TreeNode*> inorderV;
	void inorder(TreeNode* root)
	{
		if(root == nullptr)
			return;
		
		inorder(root->left);
		inorderV.push_back(root);
		inorder(root->right);
	}
	
    TreeNode* increasingBST(TreeNode* root) {
        if(root == nullptr)
			return nullptr;
		inorder(root);	
		for(int i=0;i<inorderV.size()-1;++i)
		{
			inorderV[i]->left = nullptr;
			inorderV[i]->right = inorderV[i+1];
		}
		inorderV.back()->left = nullptr;
		inorderV.back()->right = nullptr;
        return inorderV.front();
    }
};
```

# [235. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

```
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || p == nullptr || q == nullptr)
			return nullptr;
		if(root->val > p->val && root->val > q->val)
			return lowestCommonAncestor(root->left,p,q);
		else if(root->val < p->val && root->val < q->val)
			return lowestCommonAncestor(root->right,p,q);
		return root;
    }
};
```

# [112. 路径总和](https://leetcode-cn.com/problems/path-sum/)

递归：

```
class Solution {
public:
    bool hasPathSum(TreeNode* root, int sum) {
        if(root == nullptr)
            return false;
        if(root->left == nullptr && root->right == nullptr)
            return sum == root->val;
        return hasPathSum(root->left,sum - root->val) || hasPathSum(root->right,sum - root->val);
    }
};
```

迭代：

```
class Solution {
public:
    bool hasPathSum(TreeNode* root, int sum) {
        stack<pair<TreeNode*, int>> stk;
        if(root) stk.push(make_pair(root, root->val));
        while(stk.size()) {
            pair<TreeNode*, int> cur = stk.top(); stk.pop();
            if(!cur.first->left && !cur.first->right && cur.second == sum) return true;
            if(cur.first->left) stk.push(make_pair(cur.first->left, cur.second + cur.first->left->val));
            if(cur.first->right) stk.push(make_pair(cur.first->right, cur.second + cur.first->right->val));
        }
        return false;
    }
};
```

# [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

```
class Solution {
public:
    vector<vector<int>> res;
    void dfs(TreeNode *root,vector<int> &path,int sum)
    {
        if(!root)
            return;
        path.push_back(root->val);
        if(root->val == sum && (root->left == nullptr && root->right == nullptr))
            res.push_back(path);
        dfs(root->left,path,sum - root->val);
        dfs(root->right,path,sum - root->val);
        path.pop_back();
    }

    vector<vector<int>> pathSum(TreeNode* root, int sum) 
    {
        vector<int> path;
        dfs(root,path,sum);
        return res;        
    }
};
```

# [437. 路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

```
class Solution {
public:
    int count(TreeNode* root, int sum)
    {
        if(root == nullptr)
            return 0;
        return (root->val == sum) + count(root->left,sum - root->val) + count(root->right,sum - root->val);
    }

    int pathSum(TreeNode* root, int sum) {
        if(root == nullptr)
            return 0;
        return count(root,sum) + pathSum(root->left,sum) + pathSum(root->right,sum);
    }
};
```

# [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

递归：

```
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(root == nullptr)
            return nullptr;        
        TreeNode* left = invertTree(root->left);
        TreeNode* right = invertTree(root->right);
        root->left = right;
        root->right = left;
        return root;
    }
};
```

非递归：

```
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(root == nullptr)
            return nullptr;
        
        queue<TreeNode*> q;
        q.push(root);
        while(!q.empty())
        {
            TreeNode* curr = q.front();
            q.pop();
            TreeNode* left = curr->left;
            curr->left = curr->right;
            curr->right = left;
            if(curr->left)
                q.push(curr->left);
            if(curr->right)
                q.push(curr->right);    
        }
        return root;
    }
};
```

# [617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

递归：

```
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if(t1 == nullptr)
            return t2;
        if(t2 == nullptr)
            return t1;
        
        t1->val += t2->val;
        t1->left = mergeTrees(t1->left,t2->left);
        t1->right = mergeTrees(t1->right,t2->right);
        return t1;
    }
};
```

栈迭代：

```
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if(t1 == nullptr)
            return t2;
        stack<pair<TreeNode*,TreeNode*>> stk;
        if(t1 && t2)
            stk.push(make_pair(t1,t2));
        
        while(!stk.empty())
        {
            auto cur = stk.top();
            stk.pop();
            cur.first->val += cur.second->val;
            if(cur.second->left)
            {
                if(cur.first->left == nullptr)
                    cur.first->left = new TreeNode(0);  //如果左子树不存在，新建值为0的节点
                stk.push(make_pair(cur.first->left,cur.second->left));
            }
            if(cur.second->right)
            {
                if(cur.first->right == nullptr)
                    cur.first->right = new TreeNode(0);  //如果右子树不存在，新建值为0的节点
                stk.push(make_pair(cur.first->right,cur.second->right));
            }            
        }
        return t1;
    }
};
```

队列迭代：

```
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if(t1 == nullptr)
            return t2;
        if(t2 == nullptr)
            return t1;
        queue<pair<TreeNode*,TreeNode*>> q;
        q.push(make_pair(t1,t2));
        while(!q.empty())
        {
            auto cur = q.front();
            q.pop();
            cur.first->val  += cur.second->val;      
            if(cur.second->left)
            {
                if(cur.first->left == nullptr)
                    cur.first->left = new TreeNode(0);  //如果左子树不存在，新建值为0的节点
                q.push(make_pair(cur.first->left,cur.second->left));
            }
            if(cur.second->right)
            {
                if(cur.first->right == nullptr)
                    cur.first->right = new TreeNode(0);  //如果右子树不存在，新建值为0的节点
                q.push(make_pair(cur.first->right,cur.second->right));
            }   
        }
        return  t1;
    }
};
```

# [572. 另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/)

```
class Solution {
public:
    bool isSubtree(TreeNode* s, TreeNode* t) {
        if(s == nullptr)
            return false;
        return isSubtree(s->left,t) || sameTree(s,t) || isSubtree(s->right,t);
    }

    bool sameTree(TreeNode* s, TreeNode* t)
    {
        if(s == nullptr && t == nullptr)
            return true;
        if(s == nullptr || t == nullptr)
            return false;
        if(s->val != t->val)
            return false;
        
        return sameTree(s->left,t->left) && sameTree(s->right,t->right);
    }
};
```

# [671. 二叉树中第二小的节点](https://leetcode-cn.com/problems/second-minimum-node-in-a-binary-tree/)

```
class Solution {
public:
    set<int> s;
    void dfs(TreeNode* root)
    {
        if(root == nullptr)
            return;
        s.insert(root->val);
        dfs(root->left);
        dfs(root->right);
    }

    int findSecondMinimumValue(TreeNode* root) {
        if(root == nullptr)
            return -1;
        
        dfs(root);
        auto it = s.begin();
        if(++it == s.end())
            return -1;
        else
            return *it;
    }
};
```

# [236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

```
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || root == p || root == q)
            return root;
        
        TreeNode* left = lowestCommonAncestor(root->left,p,q);
        TreeNode* right = lowestCommonAncestor(root->right,p,q);
        if(left && right)
            return root;
        return left ? left : right;
    }
};
```

# [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

递归：

```
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == nullptr)
            return true;
        return  _Symmetric(root->left,root->right);
    }

    bool _Symmetric(TreeNode* left,TreeNode* right)
    {
        if(left == nullptr && right == nullptr)
            return true;
        if(left == nullptr || right == nullptr)
            return false;
        if(left->val != right->val)
            return false;
        return _Symmetric(left->left,right->right) && _Symmetric(left->right,right->left);
    }
};
```

非递归：

```
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == nullptr)
            return true;
        queue<TreeNode*> q;
        q.push(root->left);
        q.push(root->right);
        while(!q.empty())
        {
            TreeNode* L = q.front();
            q.pop();
            TreeNode* R = q.front();
            q.pop();
            if(L == nullptr && R == nullptr)
                continue;
            if(L == nullptr || R == nullptr || (L->val != R->val))
                return false;
            q.push(L->left);
            q.push(R->right);
            q.push(L->right);
            q.push(R->left);
        }
        return true;
    }
};
```

# [109. 有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

```

class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        vector<int> v;
        ListNode* curr = head;
        while(curr)
        {
            v.push_back(curr->val);
            curr = curr->next;
        }
        return vec2BST(v);     
    }

    TreeNode* vec2BST(vector<int>& vec)
    {
        if(vec.empty())
            return nullptr;
        if(vec.size() == 1)
            return new TreeNode(vec.front());

        TreeNode* root = new TreeNode(vec[vec.size()/2]);
        vector<int> left(vec.begin(),vec.begin()+vec.size()/2);
        vector<int> right(vec.begin()+vec.size()/2+1,vec.end());
        root->left = vec2BST(left);
        root->right = vec2BST(right);
        return root;
    }
};
```

快慢指针：

```
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        if(head == nullptr)
            return nullptr;
        if(head->next == nullptr)
            return new TreeNode(head->val);
        
        //@ 快慢指针,pre指向slow节点的前一个节点
        ListNode *pre = head,*slow = head,*fast = head;
        while(fast && fast->next)
        {
            pre = slow;
            slow = slow->next;
            fast = fast->next->next;
        }

        pre->next = nullptr; //@ 链表切断,一分为二
        TreeNode* root= new TreeNode(slow->val);  //@ slow 作为中间节点可以作为根
        root->left = sortedListToBST(head); //@ 递归左子树
        root->right = sortedListToBST(slow->next); //@ 递归右子树
        return root;
    }
};
```

# [958. 二叉树的完全性检验](https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/)

```
class Solution {
public:
    bool isCompleteTree(TreeNode* root) {
        if(root == nullptr)
            return true;
        queue<TreeNode*> q;
        q.push(root);
        bool final = false;
        while(!q.empty())
        {
            auto t = q.front();
            q.pop();
            if(t->left != nullptr)
            {
                if(final)
                    return false;
                q.push(t->left);
            }
            else
                final = true;
            if(t->right != nullptr)
            {
                if(final)
                    return false;
                q.push(t->right);
            }
            else
                final = true;
            
        }
        return true;
    }
};
```

# [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```
class Solution {
public:
    unordered_map<int,int> dict;
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
         if(preorder.empty())
            return nullptr;
        for(int i = 0;i < inorder.size();++i)
            dict[inorder[i]] = i;
        return build(preorder,0,preorder.size()-1,0);
    }

    //@ start:中序遍历的起始位置,end:中序遍历的终止位置,rootIndex:前序遍历根节点的位置
    TreeNode* build(vector<int>& preorder,int start,int end,int rootIndex)
    {
        if(start > end)
            return nullptr;        
        int val = preorder[rootIndex];
        TreeNode* root = new TreeNode(val);
        int index = dict[val],leftLen = index-start-1;

        root->left = build(preorder,start,index - 1,rootIndex+1);
        root->right = build(preorder,index + 1,end,rootIndex+2+leftLen);
        return root;
    }
};
```

# [889. 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

```
class Solution {
public:
    TreeNode* constructFromPrePost(vector<int>& pre, vector<int>& post) {
        int n = post.size();
        if(n == 0) 
            return nullptr;
        TreeNode* root = new TreeNode(pre[0]);
        if(n == 1) 
            return root;
        else
        {
            int L = 0;
            for(int i=0; i<post.size();++i)
                if(post[i]==pre[1])
                    L = i+1;      
            
            //@ 跳过前序遍历的根节点
            vector<int> Lpre(pre.begin()+1, pre.begin()+L+1);
            //@ 前序遍历除了第一个根节点之后是连续的
            vector<int> Rpre(pre.begin()+L+1, pre.end());
            //@ 后序遍历从起始开始,长度为L
            vector<int> Lpost(post.begin(), post.begin()+L);
            //@ 后序遍历最后一个节点是根节点
            vector<int> Rpost(post.begin()+L, post.end()-1);
            root->left = constructFromPrePost(Lpre, Lpost);
            root->right = constructFromPrePost(Rpre, Rpost);
            return root;
        }
    }
};
```

# [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

````
class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        int n = postorder.size();
        if(n == 0)
            return nullptr;
        int val = postorder.back();
        TreeNode* root = new TreeNode(val);
        int L = 0;
        for(int i=0;i < n;++i)
        {
            if(val == inorder[i])
            {
                L = i;
                break;
            }
        }
        //@ 中序遍历从起始开始,长度为L
        vector<int> Lpre(inorder.begin(), inorder.begin()+L);
        //@ 跳过中序遍历中的根节点位置
        vector<int> Rpre(inorder.begin()+L+1, inorder.end());
        //@ 后序遍历从起始开始,长度为L
        vector<int> Lpost(postorder.begin(), postorder.begin()+L);    
        //@ 后序遍历最后一个节点是根节点
        vector<int> Rpost(postorder.begin()+L, postorder.end()-1);
        root->left = buildTree(Lpre, Lpost);
        root->right = buildTree(Rpre, Rpost);
        return root;
    }
};
````

# [590. N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)

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

# [589. N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)

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

# [429. N叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

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

# [559. N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

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

