# 花坛单喷头最大覆盖与最小坐标

## 题意

`flowers` 是一维坐标数组。半径为 `radius` 的喷头放在 `p` 时，覆盖闭区间 `[p - radius, p + radius]`。求能覆盖最多花坛的最小喷头坐标。以下实现假设花坛数组非空、半径非负、喷头可在数轴任意位置放置，且答案可用 `int` 表示。

## 排序与滑动窗口

1. 先按坐标排序。窗口两端的距离至多为 `2 * radius` 时，一个喷头就能覆盖整个窗口。固定右端 `i`，如果窗口太宽，就不断右移左端 `j`。保留下来的窗口是以 `i` 为右端的最长合法窗口。
2. 让喷头的右边界刚好碰到最右花坛，即 `p + radius = flowers[i]`。因此最小可行坐标是 `p = flowers[i] - radius`，此时左边界 `flowers[i] - 2 * radius` 也能覆盖窗口最左边的花坛。
3. 随着 `i` 右移，候选坐标只会增大或保持不变。只有覆盖数量严格增加时才更新答案，覆盖数量相同就保留先找到的坐标。

例如 `flowers = {8, 1, 4, 2}`，`radius = 2`。排序后最佳窗口是 `[1, 2, 4]`，喷头放在 `4 - 2 = 2`，覆盖 `[0, 4]`，共三个花坛。

## C++17 实现

```cpp
#include <algorithm>
#include <vector>

int smallestPosOfMostCovered(std::vector<int>& flowers, int radius) {
    std::sort(flowers.begin(), flowers.end());

    long long best_pos = -1;
    int best_cover = 0;
    int n = static_cast<int>(flowers.size());

    for (int i = 0, j = 0; i < n; ++i) {
        while (1LL * flowers[i] - flowers[j] > 2LL * radius) {
            ++j;
        }

        int cover = i - j + 1;
        if (cover > best_cover) {
            best_cover = cover;
            best_pos = 1LL * flowers[i] - radius;
        }
    }

    return static_cast<int>(best_pos);
}
```

## 容易踩的坑

- **端点算覆盖**。坐标差恰好等于 `2 * radius` 时保留窗口，所以 `while` 用 `>`。
- **半径为零**。窗口只保留同一坐标的花坛。例如 `{8, 4, 1, 4, 1}` 在 `1` 和 `4` 都能覆盖两个花坛，返回较小的 `1`。
- **避免中间计算溢出**。用 `long long` 计算坐标差、覆盖跨度和候选坐标。即使最终答案能放进 `int`，较早的候选坐标也可能超出范围。如果题目允许最终答案超出 `int`，将返回类型改成 `long long`，直接返回 `best_pos`。
- **检查喷头坐标限制**。如果花坛坐标都非负，且喷头只能放在非负坐标，候选位置可以改成 `std::max(0, flowers[i] - radius)`。若题目另有限制，候选位置还须落在窗口允许的范围 `[flowers[i] - radius, flowers[j] + radius]` 内。

排序耗时 `O(n log n)`，窗口扫描耗时 `O(n)`。窗口本身使用 `O(1)` 额外空间，排序还会使用其实现所需的栈空间。
