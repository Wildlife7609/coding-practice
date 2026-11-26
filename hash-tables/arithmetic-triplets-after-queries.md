# 每次查询后的等差三元组

[FastPrep 题目](https://www.fastprep.io/problems/1.tiktok-arithmetic-triplets-after-queries)

## 题意

从空集合开始，`+x` 添加一份 `x`，`-x` 删除所有值为 `x` 的元素。每次操作后，统计满足 `a - b = b - c = diff` 的三元组。插入顺序无关，每种副本选择都单独计数。题目保证 `diff > 0`，所以三元组的三个值互不相同。

## 怎么想到只查三种情况

一次查询只改变一个值 `x` 的频次，所以总数的变化全部来自**包含 `x` 的三元组**。固定一份 `x`，在三元组里放好它的位置，另外两个值就能用公差 `d` 算出来。

| `x` 的位置 | 另外两个值 | 一份 `x` 对应的三元组数 |
| --- | --- | --- |
| 最大值 | `x-d`、`x-2d` | `cnt[x-d] * cnt[x-2d]` |
| 中间值 | `x-d`、`x+d` | `cnt[x-d] * cnt[x+d]` |
| 最小值 | `x+d`、`x+2d` | `cnt[x+d] * cnt[x+2d]` |

把三项相加，得到 `ways`。由于 `d > 0`，每个包含 `x` 的三元组只会落在表中的一行。比如另外两个值各有两份和三份，固定一份 `x` 后就有 `2 × 3 = 6` 种选择。

- 添加一份 `x` 时，总数增加 `ways`。
- 删除全部 `x` 时，总数减少 `cnt[x] * ways`，这里用的是删除前的频次。

以 `diff = 1`、`["+4", "+5", "+6", "+4", "+3", "-4"]` 为例，答案是 `[0, 0, 1, 2, 4, 0]`。第二份 `4` 只新增一个 `(6, 5, 4)`。最后删掉两份 `4` 时，每份分别参与 `(6, 5, 4)` 和 `(5, 4, 3)`，共减掉 `2 × 2 = 4`。

## C++17 实现

```cpp
#include <string>
#include <unordered_map>
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> countTripletsAfterQueries(vector<string>& queries, int diff) {
        unordered_map<long long, int> cnt;
        vector<int> ans;
        long long total = 0;
        const long long d = diff;

        auto get = [&](long long x) -> long long {
            auto it = cnt.find(x);
            return it == cnt.end() ? 0 : it->second;
        };

        for (const auto& query : queries) {
            long long x = stoll(query.substr(1));
            long long ways = get(x - d) * get(x - 2 * d)
                           + get(x - d) * get(x + d)
                           + get(x + d) * get(x + 2 * d);

            if (query[0] == '+') {
                ++cnt[x];
                total += ways;
            } else {
                total -= get(x) * ways;
                cnt.erase(x);
            }

            ans.push_back(static_cast<int>(total));
        }
        return ans;
    }
};
```

在题目的数字长度限制下，处理 `q` 次查询的平均时间为 `O(q)`。哈希表额外空间为 `O(k)`，其中 `k` 是集合中同时出现的不同值的最大数量。答案数组使用 `O(q)` 空间。

## 这次遇到的坑

1. **添加时多乘了 `cnt[x]`。** 第一次加入 `6` 时，旧频次是 `0`，会漏掉 `(6, 5, 4)`。如果用更新后的频次，添加第二份 `4` 时又会把已有的三元组重复算进去。
2. **先 `erase` 再读旧频次。** `cnt[x]` 会把刚删掉的键以 `0` 的值重新插入，导致减去的贡献是 `0`。先减贡献，再删除。把频次设为 `0` 也能正确计数。查邻居时用 `find` 可以避免为缺失的值插入零频次的键。
3. **操作符和负号要分开读。** `stoll(query.substr(1))` 解析负数。 `"+-5"` 添加 `-5`，`"--5"` 删除所有 `-5`，`"-5"` 删除所有正数 `5`。
4. **邻居和乘法要用 64 位。** `x + 2 * d` 最大可达 `3 × 10^9`。如果直接相乘的两个频次都是 `int`，可以在最前面乘 `1LL` 来提升类型。本实现的 `get` 返回 `long long`，乘法从第一步就使用 64 位。

## 验证

通过题目两组样例和 `diff = 10^9` 的边界用例，并与逐个枚举副本的暴力解对拍了 2,000 组随机输入。
