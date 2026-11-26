# Count One-Swap Number Pairs

给定一个正整数数组，统计有多少对下标 `i < j`，使其中一个数交换至多一对数字后等于另一个数。两个数本来相等也算一对。交换后有前导零的结果无效。

## 思路

从左往右扫描，用哈希表记录前面出现过的**原始数字字符串**及其频次。对当前字符串先查原串（零次交换），然后枚举每一对位置并查交换后的字符串。查完再把原串放进哈希表，所以每对下标只在处理后一个元素时计数。交换可以反过来做，因此只需要尝试交换当前的数字。

## 复习时最容易忘的两点

1. **交换 → 查表 → 换回。** `std::swap(s[i], s[j])` 原地交换两个字符。查完后再交换同一对位置，下一次枚举就能从原串开始。比如 `"123"` 先交换 `(0, 1)` 变成 `"213"`，如果不恢复，再交换 `(0, 2)` 就会得到 `"312"`，相当于在原串上做了两次交换。用 `[312, 123]` 可以查出这个问题，预期是 `0`。
2. **相同字符直接跳过。** 看到题目说「至多一次」且「每对下标只算一次」，就问自己：**有没有一次操作会把原串原封不动地返回？** 用最小反例 `[11, 11]` 手算。处理第二个 `11` 时，查原串加了 `1`，交换两个 `'1'` 后查表又加了 `1`。预期是 `1`，漏写 `if (s[i] == s[j]) continue;` 就会得到 `2`。交换不同字符时，结果与原串恰好在两个位置不同，因此结果可以反推出交换的位置，无需额外用集合去重。

哈希表只存普通十进制字符串。`"01"` 查不到 `"1"`，所以前导零和不同长度自然被排除。可以用 `[1, 10]` 检查，预期是 `0`。

## C++17

```cpp
#include <string>
#include <unordered_map>
#include <utility>
#include <vector>

class Solution {
public:
    int countOneSwapPairs(std::vector<int>& numbers) {
        std::unordered_map<std::string, int> count;
        int answer = 0;

        for (int number : numbers) {
            std::string s = std::to_string(number);
            const auto equal = count.find(s);
            if (equal != count.end()) {
                answer += equal->second;
            }

            for (std::size_t i = 0; i < s.size(); ++i) {
                for (std::size_t j = i + 1; j < s.size(); ++j) {
                    if (s[i] == s[j]) {
                        continue;
                    }

                    std::swap(s[i], s[j]);
                    const auto match = count.find(s);
                    if (match != count.end()) {
                        answer += match->second;
                    }
                    std::swap(s[i], s[j]);
                }
            }

            ++count[s];
        }
        return answer;
    }
};
```

## 复杂度

- **期望时间 `O(nL³)`**。每个数尝试 `O(L²)` 对位置，每次字符串查找的哈希计算需要 `O(L)`。本题 `L ≤ 10`，因此按输入元素个数看是期望 `O(n)`。`n ≤ 10⁴` 时最多尝试 `10,000 × 45 = 450,000` 次交换。
- **空间 `O(nL)`**，哈希表最多保存 `n` 个长度至多为 `L` 的字符串。
