# Count Member Mentions

给定成员 ID 列表和多条消息，统计每个成员被提及的**消息数**。消息开头或空格后的 `@` 开始一组提及，组内 ID 用逗号分隔，到空格或消息末尾结束。忽略未知 ID，保留零次的成员。按次数降序、同次数按 ID 字典序升序，返回 `id=count`。

## 思路

先给每个成员建一个值为零的计数。扫描每条消息时，用 `i` 指向当前 ID 的开头，用 `j` 找逗号或空格。遇到逗号就取出 `message.substr(i, j - i)`，然后把 `i` 移到逗号后面。扫描完一组后，再取出最后一个 ID。每条消息单独用一个集合去重，扫完后把集合里的已知成员各加一次。

## 这次踩的坑

1. **同一条消息里，同一个 ID 只计一次。** `"@id1,id2 @id1"` 中，`id1` 和 `id2` 各加一次。集合要放在消息循环里，否则下一条消息里的 `@id1` 也会被去掉。
2. **先判断 `@` 前面的边界。** 用 `i == 0 || message[i - 1] == ' '`，这样开头的 `@` 不会访问下标 `-1`，`"appro@ch"` 中的 `@` 也不会开始一组提及。
3. **注意 `substr` 的第二个参数是长度。** 取 `[i, j)` 要写 `substr(i, j - i)`。逗号处只能取到前面的 ID，所以循环结束后还要取一次。比如 `"@id1,id2"` 里的 `id2` 会在循环结束时取出。
4. **两根下标足够，但要知道它们停在哪里。** 一组扫完时，`j` 位于分隔空格或字符串末尾。赋值 `i = j` 后，外层的 `++i` 只会跳过这个空格，`"@id1 @id2"` 中的第二个 `@` 仍会被检查。内层条件先写 `j < n`，再访问 `message[j]`。
5. **用 `count` 判断是不是成员。** 对未知 ID 直接写 `++counts[id]` 会插入新键。排序时先比较次数，再用字符串比较 ID，因此同分时 `id10` 排在 `id2` 前面。排完序再拼接 `=次数`。
6. **编译和本地调试。** `vector<sting>` 应该写成 `vector<string>`，`to_string(counts[id])` 要闭合右括号。自测时把 `int main()` 写在类的 `};` 后，用 `Solution{}.mentionCounts(members, messages)` 调用。普通的 `cin >> members` 不能整组读取 `vector<string>`，固定样例可以直接用初始化列表。原题平台提供入口时只提交 `Solution`。

## C++17

```cpp
#include <algorithm>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>

using namespace std;

class Solution {
public:
    vector<string> mentionCounts(vector<string>& members, vector<string>& messages) {
        unordered_map<string, int> counts;
        for (const auto& id : members) {
            counts[id] = 0;
        }

        for (const auto& message : messages) {
            unordered_set<string> mentioned;
            const size_t n = message.size();

            for (size_t i = 0; i < n; ++i) {
                if (message[i] == '@' && (i == 0 || message[i - 1] == ' ')) {
                    ++i;
                    size_t j = i;
                    while (j < n && message[j] != ' ') {
                        if (message[j] == ',') {
                            mentioned.insert(message.substr(i, j - i));
                            i = j + 1;
                        }
                        ++j;
                    }
                    mentioned.insert(message.substr(i, j - i));
                    i = j;
                }
            }

            for (const auto& id : mentioned) {
                if (count.count(id)) count[id]++;
            }
        }

        vector<string> answer = members;
        sort(answer.begin(), answer.end(), [&](const string& a, const string& b) {
            if (counts.count(a) != counts.count(b)) {
                return counts.count(a) > counts.count(b);
            }
            return a < b;
        });

        for (auto& id : answer) {
            id += "=" + to_string(counts.at(id));
        }
        return answer;
    }
};
```

## 测试

| 成员 | 消息 | 预期结果 |
| --- | --- | --- |
| `{"id123", "id234", "id7", "id321"}` | `{"Hey @id123,id321 review this PR please! @id123 what do you think about this approach?", "Hey @id7 nice appro@ch! Great job! @id800 what do you think?", "@id123,id321 thx!"}` | `{"id123=2", "id321=2", "id7=1", "id234=0"}` |
| `{"id2", "id10", "id3"}` | `{"@id2,id10 @id2", "abc@id3 @unknown"}` | `{"id10=1", "id2=1", "id3=0"}` |
| `{"id2", "id1"}` | `{}` | `{"id1=0", "id2=0"}` |
| `{"id1", "id2"}` | `{"@id1  @id2", "@id2"}` | `{"id2=2", "id1=1"}` |

## 复杂度

设 `M` 为所有消息的总字符数，`n` 为成员数，`L` 为最长成员 ID 的长度，`S` 为最长消息的长度，`Q` 为消息数。

- **期望时间 `O(M + Q + nL log(n + 1) + R)`**，其中 `R` 为输出字符数。截取和哈希处理的 ID 总长度受消息总长度约束。排序做 `O(n log n)` 次比较，每次哈希查找和字符串比较最多处理 `O(L)` 个字符。
- **额外空间 `O(nL + S)`**，不计返回值。计数表存所有成员，去重集合每次只存当前消息中的 ID。
