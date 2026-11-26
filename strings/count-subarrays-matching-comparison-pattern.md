# 统计匹配比较模式的子数组

[FastPrep 题目](https://www.fastprep.io/problems/tiktok-count-subarrays-matching-comparison-pattern)

## 题意

给定整数数组 `numbers` 和只包含 `1`、`0`、`-1` 的 `pattern`。这三个值分别表示相邻数字上升、相等和下降。设两者长度分别为 `n` 和 `m`，统计有多少个长度为 `m + 1` 的连续子数组，其 `m` 次相邻比较与模式完全一致。重叠的子数组也分别计数。

## 方法一 · 暴力枚举

枚举窗口的起点 `i`。窗口的最后一个数字下标是 `i + m`，所以循环条件为 `i < n - m`。在每个窗口中，用 `j` 指向相邻两个数字中的后一个，用 `k` 指向对应的模式项。一旦失配就结束当前窗口的检查。

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int countMatchingSubarrays(vector<int>& numbers, vector<int>& pattern) {
        int n = numbers.size(), m = pattern.size();
        int ans = 0;

        for (int i = 0; i < n - m; ++i) {
            bool matches = true;
            for (int j = i + 1, k = 0; j <= i + m; ++j, ++k) {
                if (pattern[k] == 1) {
                    if (numbers[j] <= numbers[j - 1]) {
                        matches = false;
                        break;
                    }
                } else if (pattern[k] == 0) {
                    if (numbers[j] != numbers[j - 1]) {
                        matches = false;
                        break;
                    }
                } else {
                    if (numbers[j] >= numbers[j - 1]) {
                        matches = false;
                        break;
                    }
                }
            }
            if (matches) {
                ++ans;
            }
        }
        return ans;
    }
};
```

最坏时间复杂度为 `O((n - m) · m)`，额外空间为 `O(1)`。当 `numbers` 有 100,000 个递增的数，且 `pattern` 有 50,000 个 `1` 时，每个窗口都要比较到底，总共要检查 25 亿对相邻数字。

## 方法二 · 比较序列 + KMP

把每对相邻数字转成 `1`、`0` 或 `-1`。例如，`[1, 4, 4, 1, 3, 5, 5, 3]` 变成 `[1, 0, -1, 1, 1, 0, -1]`。模式 `[1, 0, -1]` 在比较序列里出现两次，对应原数组中的 `[1, 4, 4, 1]` 和 `[3, 5, 5, 3]`。

### KMP 怎么回退

这份代码在比较序列 `s` 和模式 `p` 的下标 `0` 放了一个占位值，实际内容从下标 `1` 开始。

- `j` 是已经匹配的元素个数。已经匹配了 `p[1..j]`，下一项就和 `p[j + 1]` 比较。
- `nxt[i]` 是 `p[1..i]` 的最长相等前缀和后缀的长度，整段自身不计入。例如，模式 `[1, 1, 0]` 的 `nxt[1..3]` 是 `[0, 1, 0]`。

假设模式是 `[1, 1, 0]`，已经匹配了两个 `1`，当前却又读到一个 `1`。预期的 `0` 对不上，就执行 `j = nxt[2]`，让 `j` 从 `2` 回退到 `1`。之前匹配的最后一个 `1` 可以当作新匹配的开头。当前读到的 `1` 还要再和 `p[2]` 比一次，匹配后 `j` 又回到 `2`。

完整匹配后同样执行 `j = nxt[j]`。例如，比较序列 `[1, 1, 1]` 与模式 `[1, 1]` 匹配两次。第一次匹配结束时保留末尾的 `1`，才能在读到第三个 `1` 时数出第二次匹配。

### C++17 实现

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int countMatchingSubarrays(vector<int>& numbers, vector<int>& pattern) {
        int n = numbers.size(), m = pattern.size();
        int ans = 0;

        // 下标 0 留作占位，实际内容从下标 1 开始。
        vector<int> s = {0};
        vector<int> p = {0};

        for (int i = 1; i < n; ++i) {
            if (numbers[i] > numbers[i - 1]) {
                s.push_back(1);
            } else if (numbers[i] == numbers[i - 1]) {
                s.push_back(0);
            } else {
                s.push_back(-1);
            }
        }

        for (int x : pattern) {
            p.push_back(x);
        }

        // nxt[1] 已初始化为 0，从模式的第二项开始构建。
        vector<int> nxt(m + 1, 0);
        for (int i = 2, j = 0; i <= m; ++i) {
            while (j > 0 && p[i] != p[j + 1]) {
                j = nxt[j];
            }
            if (p[i] == p[j + 1]) {
                ++j;
            }
            nxt[i] = j;
        }

        // s 有 n - 1 个比较结果，从 s[1] 扫到 s[n - 1]。
        for (int i = 1, j = 0; i < n; ++i) {
            while (j > 0 && s[i] != p[j + 1]) {
                j = nxt[j];
            }
            if (s[i] == p[j + 1]) {
                ++j;
            }
            if (j == m) {
                ++ans;
                j = nxt[j];
            }
        }

        return ans;
    }
};
```

**时间复杂度**为 `O(n + m)`。外层循环每轮最多让 `j` 增加一次，而每次回退都会让 `j` 减小，所以回退的总次数也是线性的。**额外空间**为 `O(n + m)`，用于存储 `s`、`p` 和 `nxt`。

## 这次踩的两个坑

1. **把 `nxt[i] = j` 写成了 `nxt[j] = i`。** `i` 表示当前处理的是模式的前多少项，`j` 是要存下来的长度。以 `[1, 1]` 为例，正确的是 `nxt[2] = 1`。
2. **扫描比较序列时用了 `i <= n`。** `n` 个数字只有 `n - 1` 个相邻比较。`s[0]` 加了占位后，最后一个有效下标是 `n - 1`，所以循环条件是 `i < n`。

## 验证

FastPrep 的 3 组样例全部通过。KMP 版和暴力版对拍了 10,000 组随机输入，结果一致。对 100,000 个递增数字和 50,000 个 `1` 的压力用例，KMP 返回了 `50,000`。
