# [07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

```
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.size() != inorder.size() || preorder.empty())
            return nullptr;
        
        TreeNode* root = new TreeNode(preorder[0]);
        auto it = std::find(inorder.begin(),inorder.end(),preorder[0]);

        auto Lin = std::vector<int>(inorder.begin(),it);
        auto Rin = std::vector<int>(it+1,inorder.end());

        auto size = Lin.size();

        auto Lpre = std::vector<int>(preorder.begin()+1,preorder.begin()+1+size);
        auto Rpre = std::vector<int>(preorder.begin()+1+size,preorder.end());

        root->left = buildTree(Lpre,Lin);
        root->right = buildTree(Rpre,Rin);

        return root;
    }
};
```

# [16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

```
class Solution {
public:
    double myPow(double x, int n) {
        long exp = n;
        if(exp < 0)
            return 1.0 / quick_pow(x,-exp);
        
        return quick_pow(x,n);
    }

    double quick_pow(double x,long n)
    {
        if(n == 0)
            return 1.0;
        if(n == 1)
            return x;
        
        double t = quick_pow(x,n >>1);
        double res = t * t;

        return (n & 1) ? res * x : res;
    }
};
```

# [17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

```
class Solution {
public:
    vector<int> printNumbers(int n) {
        std::vector<int> res;
        std::string str(n,'0');
        back_trace(res,str,0,n);
        return res;
    }

    void back_trace(std::vector<int>& res,std::string& str,int idx,int n)
    {
        if(idx == n)
        {
            int val = std::stoi(str);
            if(val)
                res.push_back(val);
            return;
        }

        for(char i = '0';i <= '9';i++)
        {
            str[idx] = i;
            back_trace(res,str,idx + 1,n);
        }
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

# [51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

```
class Solution {
public:
    int reversePairs(vector<int>& nums) {
        std::vector<int> tmp(nums.size());
        return merge_sort(0, nums.size() - 1, nums, tmp);
    }
private:
    int merge_sort(int left, int right, std::vector<int>& nums, std::vector<int>& tmp) 
	{
        if (left >= right) 
			return 0;
        int m = (left + right) / 2;
        int res = merge_sort(left, m, nums, tmp) + merge_sort(m + 1, right, nums, tmp);
        int i = left, j = m + 1;
        for (int k = left; k <= right; k++)
            tmp[k] = nums[k];
        for (int k = left; k <= right; k++) 
        {
            if (i == m + 1)
                nums[k] = tmp[j++];
            else if (j == right + 1 || tmp[i] <= tmp[j])
                nums[k] = tmp[i++];
            else {
                nums[k] = tmp[j++];
                res += m - i + 1;
            }
        }
        return res;
    }
};
```

