# 按元音与辅音数量差排序单词

## 题意

将英文句子按空白分成单词。对每个单词计算 `abs(元音数 - 辅音数)`，先按差值升序，再按原单词的字典序升序。大小写都参与字母计数，字典序比较保留原大小写。标点随单词保留，计数时跳过。

## 思路

- 用 `istringstream >> word` 按空白读取。这样 `bucket,` 和 `don't` 都各自是一个单词。
- 用 `unordered_set::count` 判断是否是元音。其余字符只有在 `isalpha` 为真时才算辅音。
- 将 `{差值, 单词}` 存入 `pair<int, string>`。默认排序先比较第一项，相同再比较第二项。

## C++17 实现

```cpp
#include <algorithm>
#include <cctype>
#include <cstdlib>
#include <sstream>
#include <string>
#include <unordered_set>
#include <utility>
#include <vector>

std::string sortWords(const std::string& input) {
    const std::unordered_set<char> vowels = {
        'a', 'A', 'e', 'E', 'i', 'I', 'o', 'O', 'u', 'U'
    };

    std::istringstream stream(input);
    std::vector<std::pair<int, std::string>> words;
    std::string word;

    while (stream >> word) {
        int vowelCount = 0;
        int consonantCount = 0;
        for (char ch : word) {
            if (vowels.count(ch)) {
                ++vowelCount;
            } else if (std::isalpha(static_cast<unsigned char>(ch))) {
                ++consonantCount;
            }
        }

        words.emplace_back(std::abs(vowelCount - consonantCount), word);
    }

    std::sort(words.begin(), words.end());

    std::string answer;
    for (auto& [k, v]: words) {
        answer += v + ' ';
    }
    if (answer.size()) answer.pop_back();
    return answer;
}
```

## 容易踩的坑

- **计数和下标**。题目要的是两个数量的差。只有在 `[i, j)` 全部是字母时，才有 `consonantCount = j - i - vowelCount`，差值也可写成 `abs(2 * vowelCount - (j - i))`。
- **标点和分词**。用 `while (isalpha(...))` 扫描会把标点当分隔符。按空白分词时，标点留在输出中。例如 `don't` 的得分为 `abs(1 - 3) = 2`。
- **空输入**。拼接完成后，只有 answer 非空时才调用 pop_back()，删除末尾空格。
- **返回类型**。这里沿用 `sortWords` 的字符串返回值，单词间用一个空格连接。如果平台要求 `vector<string>`，就按排序后的顺序把每个 pair 的第二项放入结果数组。

设输入长度为 `N`，单词数为 `W`，最长单词长度为 `L`。读入、计数和拼接耗时 `O(N)`，排序最多进行 `O(W log W)` 次比较，字典序比较最坏耗时 `O(L)`。总时间复杂度为 `O(N + W log W * L)`，额外空间复杂度为 `O(N)`。
