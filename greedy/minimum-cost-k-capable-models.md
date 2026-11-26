# 每个 k 的模型组合最小成本

[FastPrep 题目](https://www.fastprep.io/problems/tiktok-minimum-cost-k-capable-models)

## 题意

每个模型有一个正成本。功能标记 `01` 和 `10` 各支持一种功能，`11` 同时支持两种，`00` 两种都不支持。对每个 `k = 1, …, n`，求让两种功能各至少有 `k` 个模型支持的最小成本。无法满足时返回 `-1`。每个 `k` 独立求解，同一组合里每个模型最多选一次。

## 排序后合并两种选择

一个 `11`，或者一对 `01` 和 `10`，都能让两种功能的支持数各增加 1。

把 `01`、`10`、`11` 三组成本分别从小到大排序。将第一个 `01` 和第一个 `10` 配对，再将第二个和第二个配对，以此类推。两边成本都非递减，所以配对成本也是非递减的。接下来像合并两个有序数组一样，每次比较下一对的成本和下一个 `11` 的成本，取较小的。每取一次，把累计成本写为下一个 `k` 的答案。

例如三组成本分别是 `01: [2]`、`10: [3]`、`11: [5, 6]`。配对成本是 `[5]`，合并后得到 `[5, 5, 6]`，所以前三个答案是 `[5, 10, 16]`。

### 为什么贪心成立

对于固定的 `k`，如果选择了 `x < k` 个 `11`，就至少需要 `k - x` 个 `01` 和 `k - x` 个 `10`。因为成本为正，满足要求后可以删掉多余模型。如果选了至少 `k` 个 `11`，则只需保留其中 `k` 个。于是只需要考虑恰好 `x` 个 `11` 加上 `k - x` 对单功能模型，其中 `0 ≤ x ≤ k`。

固定 `x` 时，各组取最便宜的前缀最划算。依次配对得到的前 `k - x` 个成本之和，恰好等于两个单功能组各自前 `k - x` 项的总和。因此答案就是从两个有序成本序列中取最小的 `k` 项之和。合并时已经从每个序列取了一个前缀，对应的模型也不会重复使用。

## C++17 实现

```cpp
#include <algorithm>
#include <cstddef>
#include <string>
#include <unordered_map>
#include <vector>
using namespace std;

class Solution {
public:
    vector<long long> minimumModelCosts(vector<int>& cost,
                                        vector<string>& featureAvailability) {
        unordered_map<string, vector<int>> costs;
        for (size_t i = 0; i < cost.size(); ++i) {
            if (featureAvailability[i] != "00") {
                costs[featureAvailability[i]].push_back(cost[i]);
            }
        }

        auto& a = costs["01"];
        auto& b = costs["10"];
        auto& both = costs["11"];
        sort(a.begin(), a.end());
        sort(b.begin(), b.end());
        sort(both.begin(), both.end());

        vector<long long> ans;
        ans.reserve(cost.size());
        long long total = 0;
        size_t i = 0, j = 0;

        while ((i < a.size() && i < b.size()) || j < both.size()) {
            if (i < a.size() && i < b.size() &&
                (j == both.size() || 1LL * a[i] + b[i] <= both[j])) {
                total += 1LL * a[i] + b[i];
                ++i;
            } else {
                total += both[j];
                ++j;
            }
            ans.push_back(total);
        }

        ans.resize(cost.size(), -1);
        return ans;
    }
};
```

## 这次遇到的坑

1. **`set<int>` 会丢掉同价模型。** 两个成本都是 `5` 的 `11` 可以一起选，得到 `k = 2`、成本 `10`。用 `vector<int>` 保存每份成本，再排序。
2. **比较前先检查配对两边的下标。** 即使 `01` 还有两个可用模型，只要 `10` 已用完，就只能考虑 `11`。先检查两个数组各自的长度，再用 `&&` 的短路行为保护下标。两组总是一起取，所以可以共用一个下标。
3. **累计成本用 `long long`。** 最多可能选 `10^5` 个模型，每个成本最高 `10^9`，总成本可到 `10^14`。

## 复杂度和基数排序

排序花 `O(n log n)` 时间，合并和填充答案花 `O(n)` 时间。分组成本占 `O(n)` 额外空间，答案数组也占 `O(n)` 空间。

在比较模型下，`O(n log n)` 已经最优。假设所有模型都是 `11`，每个答案就是已排序成本的前缀和。将相邻答案相减，就能恢复排好序的成本，因此这题至少和比较排序一样难。

题目给定成本是有界正整数，可以用基数排序将排序和合并的总时间降到 `O(n)`。十进制基数排序先按个位分到 10 个桶，按桶号顺序收回，再依次处理十位、百位，直到最高位。每轮都保留同一桶里的原顺序，这样上一轮排好的低位顺序就能保留下来。

例如 `[53, 12, 25, 11, 42]` 按个位分桶后得到 `[11, 12, 42, 53, 25]`，再按十位分桶得到 `[11, 12, 25, 42, 53]`。在 `cost[i] ≤ 10^9` 的范围内，最多处理 10 位，每轮花 `O(n + 10)` 时间。各组排好后，仍然用同一段贪心合并。
