# LeetCode 199 · 二叉树右视图

[LeetCode 199](https://leetcode.com/problems/binary-tree-right-side-view/) 直接给 `TreeNode* root`。[PracHub Part 2](https://prachub.com/coding-questions/solve-string-grouping-and-tree-right-view-problems) 给的是紧凑的层序数组，需要先读出父子关系。本文记录 C++17 下的建树和 BFS。

## 从层序数组建树

先把根节点放进队列。每取出一个真实的父节点，就从数组顺序读最多两项，依次作为左、右孩子。空项占一个孩子位置，只有存在的孩子才创建节点并入队。

例如 `[1, null, 2, 3]`：

| 父节点 | 左孩子 | 右孩子 |
| --- | --- | --- |
| 1 | 空 | 2 |
| 2 | 3 | 空 |

右视图是 `[1, 2, 3]`。用 `2*i+1` 会去下标 5 找节点 2 的左孩子，漏掉实际在下标 3 的节点。

## 这次遇到的三个坑

1. 不能用 2*i+1 和 2*i+2 找孩子。 这个数组只给真实节点记录孩子。用队列按顺序取出父节点，每取出一个，就从数组接着读最多两项作为它的左、右孩子。
2. 创建根节点后直接用 `q.front()`。先把根节点入队，每次取队首前确保队列有元素。
3. 先用 `*entry` 创建节点，再判断 `entry` 有没有值。以 `[1, 2, null]` 为例，右孩子应保持为空。先检查 `has_value()`，再取出整数并创建节点。

## C++17 实现

建树后，按从左到右的顺序遍历每层，把这一层的最后一个节点加入答案。下面的示例用 `new` 创建节点，释放节点的两种写法见「内存管理」。

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    struct Node {
        int val;
        Node* left;
        Node* right;
        Node(int x) {
            val = x;
            left = right = nullptr;
        }
    };

    void deleteTree(Node* root) {
        if (root == nullptr) return;
        deleteTree(root->left);
        deleteTree(root->right);
        delete(root);
    }

    vector<int> solution(const vector<optional<int>>& levelOrder) {
        if (levelOrder.empty() || !levelOrder[0].has_value()) return {};
        Node* root = new Node(*levelOrder[0]);
        int n = levelOrder.size();
        queue<Node*> q;
        q.push(root);
        for (int i = 1; i < n; i += 2) {
            Node* cur = q.front();
            q.pop();
            if (levelOrder[i]) {
                cur->left = new Node(*levelOrder[i]);
                q.push(cur->left);
            }
            if (i + 1 < n && levelOrder[i + 1]) {
                cur->right = new Node(*levelOrder[i + 1]);
                q.push(cur->right);
            }
        }

        vector<Node*> curr = {root};
        vector<int> ans;
        while (curr.size()) {
            ans.push_back(curr.back()->val);
            vector<Node*> nxt;
            for (auto& node : curr) {
                if (node->left) nxt.push_back(node->left);
                if (node->right) nxt.push_back(node->right);
            }
            curr = nxt;
        }
        deleteTree(root);
        return ans;
    }
};
```

时间复杂度为 O(n)，空间复杂度为 O(n)，其中 n 是输入数组的长度。

## 内存管理

上面的建树代码用 `new Node(...)` 创建根节点和孩子。`queue<Node*>` 和 `vector<Node*>` 只保存节点地址。函数返回时，这些容器会销毁，但 `new` 创建的节点仍占用内存，因此会发生内存泄漏。树用完后，记得调用 deleteTree(root)，把每个节点都释放掉。

| 输入 | 预期输出 |
| --- | --- |
| `[]` | `[]` |
| `[0]` | `[0]` |
| `[1, 2, null]` | `[1, 2]` |
| `[1, null, 2, 3]` | `[1, 2, 3]` |
| `[1, 2, 3, null, 5, null, 4]` | `[1, 3, 4]` |
