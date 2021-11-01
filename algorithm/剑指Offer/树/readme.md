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

