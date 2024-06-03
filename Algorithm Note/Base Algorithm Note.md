### 排序算法
[[排序算法]]

### 二分查找

注意：left + right / 2 一般可以换成left + (left + right) / 2 以防止溢出
经典二分查找：
```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int l = 0, r = nums.size() - 1;
        int mid;
        while(l <= r)
        {
            mid = l + (r - l) / 2;
            if(nums[mid] == target) return mid;
            else if(nums[mid] < target) l = mid + 1;
            else r = mid - 1;
        }
        return -1;
    }
};
```

### 并查集
pre[]：记录每个结点的先驱结点
size[]：记录当前结点所属集合的大小
count：记录连通分量的个数

```cpp
//模板
class UnionFind
{
public:
    vector<int> pre;
    vector<int> size;
    int count;
    UnionFind(int n)
    {
        count = n;
        pre.resize(n);
        size.resize(n, 1);
        for (int i = 0; i < n; i++)
        {
            pre[i] = i;
        }
    }
    //查询根节点
    int find(int x)
    {
        if (pre[x] == x)
            return x;
        return pre[x] = find(pre[x]);
    }
	//查询是否在同一集合
    bool isconnect(int x, int y)
    {
        return find(x) == find(y);
    }
    //合并
    bool connect(int x, int y)
    {
        x = find(x);
        y = find(y);
        if (x == y)
            return false;
        if (size[x] < size[y])
            swap(x, y);
        pre[y] = x;
        size[x] += size[y];
        count--;
        return true;
    }
		//断开连接 
    void disconnect(int x)
    {
        pre[x] = x;
    }
};
```

### 先序遍历二叉树
```cpp
void PreOrder(BiNode *bt) { //树的前序遍历
	SqStack s;
	s = InitStack();
	BiNode *p = bt;
	while (p != NULL || StackEmpty(&s) != 1) { //当p为空，栈也为空时退出循环
		while (p != NULL) {
			visit(p->data);//访问根结点
			Push(&s, p); //将指针p的节点压入栈中
			p = p->Lchild; //遍历左子树
		}
		if (StackEmpty(&s) != 1) { //栈不为空
			p = Pop(&s); //根结点出栈,相当于回退
			p = p->rchild; //遍历右子树
		}
	}
	DestroyStack(&s);
}
```
### 中序遍历二叉树
```cpp
/*中序遍历*/
void InOrder2(BiTree T)
{
	SqStack S;                  // 申请一个辅助栈
	InitStack(&S);				// 初始化
	BiTree p = T;				// p为遍历指针
	while (p || !IsEmpty(S))    // 栈不空或p不空时循环
	{
		if (p)					// 一路向左
		{
			Push(&S, p);        // 当前结点入栈
			p = p->lchild;		// 左孩子不空，一直向左走
		}
		else					// 出栈，并转向出栈结点的右子树
		{
			Pop(&S, &p);		// 栈顶元素出栈
			visit(p);			// 访问出栈结点
			p = p->rchild;		// 向右子树走，p赋值为当前结点的右孩子
		}						// 返回while循环继续进入if-else语句
	}
}
```
### 后序遍历
```cpp
/*后序遍历————利用标志*/
struct stack
{
	BiTree t;
	int tag;            // 标志
};						// tag = 0表示左子女被访问，tag = 1表示右字母被访问
void PostOrder3(BiTree T)
{
	struct stack s[Maxsize];
	int top = -1;
	while (T != NULL || top >= 0)
	{
		while (T != NULL)
		{
			s[++top].t = T;
			s[top].tag = 0;
			T = T->lchild;					// 沿左分支向下
		}
		while (top != -1 && s[top].tag == 1)
			visit(s[top--].t);					// 退栈
		if (top != -1)
		{
			s[top].tag = 1;						// 标志访问过右子树被访问
			T = s[top].t->rchild;				// 沿右分支向下遍历
		}
	}
}
```
### 层序遍历
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*>lay;
        vector<vector<int>>ave;
        if(root != nullptr)lay.push(root);
        while(!lay.empty())
        {
            int s = lay.size();
            vector<int> num;
            for(int i = 0; i < s; i++)
            {
                TreeNode* a = lay.front();
                num.push_back(a->val);
                lay.pop();
                if(a->left != nullptr) lay.push(a->left);
                if(a->right != nullptr) lay.push(a->right);
            }
            ave.push_back(num);
        }
        return ave;
    }
};
```

### 滑动窗口
含不重复字符的最长子串
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int length = s.length();
        if(length == 0) return 0;
        unordered_map<char, int>str;
        int p, maxlen = 0;
        int l = 0;
        for(int r = 0; r < length; r++)
        {
            while(str.find(s[r]) != str.end())
            {
                str.erase(s[l]);
                l++;
            }
            maxlen = max(maxlen, r - l + 1);
            str.emplace(s[r], r);
        }
        return maxlen;
    }
};
```
