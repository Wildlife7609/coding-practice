# 统计后缀单词对

## 题意

给定字符串数组 `words`，统计满足 `i < j` 的下标对。如果其中一个单词是另一个单词的后缀，这对下标就计入答案。整个单词也是自己的后缀，因此两个相同的单词可以配对。

数组最多有 100,000 个单词，每个单词长度为 1 到 10，且只含小写英文字母。

## 方法一 · 暴力枚举

枚举每一对下标，比较较短的词和较长的词末尾对应的字符。设单词数为 $n$，最大词长为 $L$，最坏需要 $O(n^2L)$ 时间。仅枚举下标对就可能接近 50 亿次。

## 方法二 · 按长度排序 + 哈希表

一个长度为 10 的词最多只有 10 个非空后缀。比如 `"cba"` 的后缀是 `"cba"`、`"ba"` 和 `"a"`。枚举这些后缀，就能直接去哈希表里查有多少个对应的完整单词。

1. 按单词长度从短到长排序。
2. 取到一个词时，用 `word.substr(start)` 枚举它的每个非空后缀，把该后缀在 `seen` 中的出现次数加入答案。
3. 查完才把当前的完整单词加入 `seen`。

排序后，每对合法单词中较晚处理的词至少和另一个一样长，因此较早处理的词必然是它的某个后缀。这对下标会在枚举到该后缀时恰好被计数一次。原题允许任一方向的后缀关系，每两个不同下标都对应唯一的 `i < j`，因此调整处理顺序不影响总数。

以 `["aa", "a", "aa"]` 为例，排序后按下面的顺序处理。

| 当前单词 | 已出现的匹配单词 | 新增配对数 |
| --- | --- | ---: |
| `"a"` | 无 | 0 |
| 第一个 `"aa"` | 一个 `"a"` | 1 |
| 第二个 `"aa"` | 一个 `"a"` 和一个 `"aa"` | 2 |

### C++17 实现

```cpp
#include <algorithm>
#include <cstddef>
#include <string>
#include <unordered_map>
#include <vector>

class Solution {
public:
    long long countSuffixPairs(std::vector<std::string>& words) {
        std::sort(words.begin(), words.end(),
                  [](const std::string& a, const std::string& b) {
                      return a.size() < b.size();
                  });

        std::unordered_map<std::string, int> seen;
        long long ans = 0;

        for (const std::string& word : words) {
            for (std::size_t start = 0; start < word.size(); ++start) {
                std::string suffix = word.substr(start);
                auto it = seen.find(suffix);
                if (it != seen.end()) {
                    ans += it->second;
                }
            }
            ++seen[word];
        }

        return ans;
    }
};
```

- **时间复杂度**为 $O(n\log n + nL^2)$，按哈希表的平均查找成本计算。每个单词取 $L$ 次以内的后缀，构造和哈希一个后缀最多处理 $L$ 个字符。
- **额外空间复杂度**为 $O(nL)$，主要用来保存不同单词的键。代码会就地排序输入数组。

## 易错点

- **从下标 0 开始枚举。** `word.substr(0)` 是整个单词，漏掉它就会少算相同单词之间的配对。
- **先查次数，再插入当前词。** 当前词也是自己的后缀。如果先插入，就会把当前下标与自己配成一对。
- **答案用 `long long`。** 如果有 100,000 个相同单词，答案是 4,999,950,000，超过 32 位有符号整数的范围。
- **比较器使用 `[]`。** `a` 和 `b` 由 `std::sort` 作为参数传入。函数体只读取这两个字符串的长度，所以捕获列表可以留空。两个参数都用 `const std::string&`。

## 验证

| 输入 | 预期 | 覆盖情况 |
| --- | ---: | --- |
| `["aa", "a", "aa"]` | 3 | 短词在长词后面，且有重复词 |
| `["back", "backdoor", "door", "comeback"]` | 2 | 前缀关系不计入，后缀关系计入 |
| `["a", "b"]` | 0 | 无合法配对 |
| `["abc"]` | 0 | 单个单词 |
| 100,000 个 `"a"` | 4,999,950,000 | 答案超出 32 位整数范围 |
