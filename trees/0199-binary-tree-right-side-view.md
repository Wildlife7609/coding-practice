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

1. 用堆式下标连接紧凑层序数组。改用一个下标顺序读孩子。
2. 创建根节点后直接用 `q.front()`。先把根节点入队，每次取队首前确保队列有元素。
3. 先用 `*entry` 创建节点，再判断 `entry` 有没有值。以 `[1, 2, null]` 为例，右孩子应保持为空。先检查 `has_value()`，再取出整数并创建节点。

## C++17 实现

建树后，按从左到右的顺序遍历每层，把这一层的最后一个节点加入答案。这里用一个 `vector` 持有节点，并提前预留足够的空间，保证建树时节点的地址保持有效。

```cpp
#include <cstddef>
#include <optional>
#include <queue>
#include <vector>

class Solution {
    struct Node {
        int value;
        Node* left = nullptr;
        Node* right = nullptr;

        explicit Node(int value) : value(value) {}
    };

public:
    std::vector<int> solution(
        const std::vector<std::optional<int>>& levelOrder
    ) {
        if (levelOrder.empty() || !levelOrder.front().has_value()) {
            return {};
        }

        std::vector<Node> nodes;
        nodes.reserve(levelOrder.size());
        nodes.emplace_back(*levelOrder.front());

        Node* root = &nodes.front();
        std::queue<Node*> parents;
        parents.push(root);
        std::size_t next = 1;

        while (!parents.empty() && next < levelOrder.size()) {
            Node* parent = parents.front();
            parents.pop();

            const auto& left = levelOrder[next++];
            if (left.has_value()) {
                nodes.emplace_back(*left);
                parent->left = &nodes.back();
                parents.push(parent->left);
            }

            if (next < levelOrder.size()) {
                const auto& right = levelOrder[next++];
                if (right.has_value()) {
                    nodes.emplace_back(*right);
                    parent->right = &nodes.back();
                    parents.push(parent->right);
                }
            }
        }

        std::vector<int> result;
        std::queue<Node*> pending;
        pending.push(root);

        while (!pending.empty()) {
            const std::size_t levelSize = pending.size();

            for (std::size_t i = 0; i < levelSize; ++i) {
                const Node* node = pending.front();
                pending.pop();

                if (i == levelSize - 1) {
                    result.push_back(node->value);
                }
                if (node->left) {
                    pending.push(node->left);
                }
                if (node->right) {
                    pending.push(node->right);
                }
            }
        }

        return result;
    }
};
```

时间复杂度为 O(n)，空间复杂度为 O(n)，其中 n 是输入数组的长度。

| 输入 | 预期输出 |
| --- | --- |
| `[]` | `[]` |
| `[0]` | `[0]` |
| `[1, 2, null]` | `[1, 2]` |
| `[1, null, 2, 3]` | `[1, 2, 3]` |
| `[1, 2, 3, null, 5, null, 4]` | `[1, 3, 4]` |
