# Render a Messenger Window

给定按顺序发送的消息，每条消息包含发送者和正文。按 `userWidth` 贪心换行，单词不能拆开。用户 `1` 左对齐，用户 `2` 右对齐。每行内部宽度为 `width`，外侧加边框。

## 思路

1. 每条消息从空行开始。`i` 指向当前单词开头，`j` 一直走到下一个空格或正文末尾。单词占 `[i, j)`，长度是 `j - i`。取词后用 `i = j + 1` 跳过一个分隔空格。最后一个词会让 `j == n`，下一轮由 `i < n` 结束。
2. 当前行为空时直接放入单词。否则，加上一个空格和整个单词后长度 `<= userWidth` 就追加。放不下就输出当前行，并用 `line = word` 开启新行。
3. 每条消息扫描结束后输出最后一行。每行补 `width - line.size()` 个空格，左对齐时补在右边，右对齐时补在左边。

## 这次踩的坑

1. **先检查下标，再读字符。** `while (j < n && t[j] != ' ')` 也能处理最后一个词。`t` 是 `m[1]`（正文），`m` 是 `[发送者, 正文]`。
2. **换行时保留触发换行的词。** `i` 已经越过当前词。比如 `userWidth = 5` 时，读到 `"Hello how"` 要先输出 `"Hello"`，再让 `line = "how"`。
3. **消息末尾要再输出一次。** 单词 `"u"` 从未触发超宽，只有循环结束后的输出能把它写进结果。
4. **区分两个宽度。** `userWidth` 决定何时换行，`width` 决定如何对齐。边框字符在 `width` 之外，因此每个输出字符串长度都是 `width + 2`，顶底各有 `width` 个星号。
5. **重复字符用 `string(数量, 字符)`。** `string(width, '*')` 生成一排星号，`to_string` 用于把数字转成字符串。

## C++17

```cpp
#include <string>
#include <vector>

using namespace std;

class Solution {
public:
    vector<string> renderMessengerWindow(vector<vector<string>>& messages, int width, int userWidth) {
        vector<string> ans;
        string st = "+" + string(width, '*') + "+";
        ans.push_back(st);
        for (auto& m: messages) {
            string& t = m[1];
            int n = t.size();
            string line;
            for (int i = 0; i < n;) {
                int j = i;
                while (j < n && t[j] != ' ') j ++;
                string word = t.substr(i, j - i);
                i = j + 1;
                if (line.empty()) line = word;
                else if (line.size() + word.size() + 1 <= userWidth) line += ' ' + word;
                else {
                    if (m[0] == "1") ans.push_back('|' + line + string(width - line.size(), ' ') + '|');
                    else ans.push_back('|' + string(width - line.size(), ' ') + line + '|');
                    line = word;
                }
            }
            if (m[0] == "1") ans.push_back('|' + line + string(width - line.size(), ' ') + '|');
            else ans.push_back('|' + string(width - line.size(), ' ') + line + '|');

        }
        ans.push_back(st);
        return ans;
    }
};
```

## 测试

`messages = {{"1", "ab c d"}, {"2", "1234 a"}}`，`width = 6`，`userWidth = 4`。`"ab c"` 刚好达到 `userWidth`，`"d"` 要换行。右侧的 `"1234"` 左边补两个空格。

```text
+******+
|ab c  |
|d     |
|  1234|
|     a|
+******+
```

本地 C++17 通过题目的两个示例、3 个边界用例和 10,000 组随机对拍。边界用例包括 `userWidth = 1` 和 100 条各含 100 个字符的消息。题目保证至少有一条消息且正文非空。

## 复杂度

设 `N` 为正文总字符数，`L` 为输出的消息行数，`W` 为 `width`。

- 时间 `O(N + (L + 2)W)`，扫描正文并构建每条定宽输出，含两行边框。
- 额外空间 `O(W)`，不计返回值。返回内容占 `O((L + 2)W)`。
