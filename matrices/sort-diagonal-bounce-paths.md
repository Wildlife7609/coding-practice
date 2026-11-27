# 对角线反弹路径排序

## 题意

从矩阵最左列的每一行各出发一次，初始方向为右上。每一步向右走一列。下一步的行号如果越界，就先把垂直方向反过来，再移动。路径权重是经过的格子之和，包含起点和最后一列的格子。

先按路径权重升序排序，权重相同时按起点值升序排序。本实现返回排序后的起点值。

## 模拟

列号始终递增，所以用 `for` 遍历列，只需要额外维护当前行 `row` 和垂直方向 `dr`。每一列先累加当前格子的值。先检查 `row + dr` 是否越界，再更新行号。每个新起点都重新初始化方向。

例如 3 行 4 列，从 `(0, 0)` 出发，路径为 `(0, 0) → (1, 1) → (2, 2) → (1, 3)`。第一步在上边界反弹，第三步在下边界反弹。如果只有 3 列，到达 `(2, 2)` 就结束了。

## C++17 实现

```cpp
#include<bits/stdc++.h>
using namespace std;

std::vector<int> solution(vector<vector<int>>& matrix) {
    if (matrix.empty() || matrix[0].empty()) {
        return {};
    }
    int n = matrix.size(), m = matrix[0].size();
    vector<pair<long long, int>> paths;
    paths.reserve(n);
    for (int i = 0; i < n; i ++) {
        int dr = (n == 1) ? 0 : -1;
        int r = i;
        long long weight = 0;
        for (int j = 0; j < m; j ++) {
            weight += matrix[r][j];
            if (r + dr < 0 || r + dr == n) {
                dr = -dr;
            }
            r += dr;
        }
        paths.push_back({weight, matrix[i][0]});
    }
    // pair 先比较权重，权重相同再比较起点值。
    std::sort(paths.begin(), paths.end());

    vector<int> ans;
    for (const auto& p : paths) {
        ans.push_back(p.second);
    }
    return ans;
}
```

## 容易踩的坑

- **单行矩阵**。初始化 `dr = 0`，每次只向右移动。用 `dr = -1` 时，即使反转方向，下一行也会越界。
- **只在下一步越界时反弹**。比如从最后一行出发，初始方向朝上，第一步可以直接走。
- **用 `long long` 累加权重**。每个元素能放进 `int`，整条路径的和仍可能超出 `int`。
- **返回前核对题意**。这里取 `path.second`，也就是起点值。如果题目要求起点行号，就同时保存行号，并按题目要求处理并列。

时间复杂度为 `O(R * C + R log R)`，额外空间复杂度为 `O(R)`。
