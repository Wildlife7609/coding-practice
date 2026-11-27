# 单调栈 · 找左右两侧第一个更大或更小的数

[练习题](https://www.fastprep.io/problems/tiktok-next-greater-elements-and-gap)

## 右边第一个更大元素和最大下标差

对每个下标 `i`，找右边最近且满足 `nums[j] > nums[i]` 的下标 `j`。找不到就记为 `-1`。返回两行，第一行记录每个位置的答案，第二行只包含最大的 `j - i`，没有匹配时记为 `0`。

双循环可以从 `i + 1` 开始找，遇到第一个更大的数就停。最坏时间为 `O(n²)`，在本题 `n ≤ 2000` 的限制下最多约 200 万次比较。单调栈把时间降为 `O(n)`。

### 当前下标给栈顶找答案

从左往右扫，栈里存的是还没找到答案的下标，对应值从栈底到栈顶非递增。当前值大于栈顶值时，当前位置就是栈顶的答案。由于我们按下标顺序往右扫，这是栈顶遇到的第一个更大的数。写入答案后弹栈，继续检查新的栈顶。

```cpp
#include <algorithm>
#include <stack>
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> nextGreaterIndicesAndMaxGap(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> ans(2);
        ans[0] = vector<int>(n, -1);
        int max_gap = 0;
        stack<int> stk;

        for (int i = 0; i < n; ++i) {
            while (!stk.empty() && nums[stk.top()] < nums[i]) {
                int prev = stk.top();
                ans[0][prev] = i;
                max_gap = max(max_gap, i - prev);
                stk.pop();
            }
            stk.push(i);
        }

        ans[1].push_back(max_gap);
        return ans;
    }
};
```

每个下标最多进栈、出栈各一次。时间 `O(n)`，栈的额外空间 `O(n)`。空输入返回 `[[], [0]]`。

### 这次踩的坑

- **写完答案要弹栈。** 漏写 `stk.pop()` 时，`[1, 2]` 会一直更新同一个下标，`while` 无法结束。
- **按下标写答案。** `vector<int>(n, -1)` 的参数顺序是「个数、初始值」。已经为每个下标分好位置，所以用 `ans[0][prev] = i`，避免用 `push_back(i)` 把答案追加到行尾。
- **严格更大用 `<` 弹出待匹配的栈顶。** 遇到相等元素时继续等。比如 `[2, 2, 3]` 的两个 `2` 都在下标 `2` 找到答案。

## 四种变体 · 栈顶给当前下标找答案

统一记法是**从答案所在的一侧开始扫**。找左边就从左往右扫，找右边就从右往左扫。栈里存已经扫过的候选下标。先弹掉不符合严格大小关系的栈顶，剩下的栈顶就是当前位置的答案，然后把当前位置入栈。

| 要找的下标 | 扫描方向 | 弹出栈顶的条件 | 栈顶留下时 |
| --- | --- | --- | --- |
| 左边第一个严格更大 | 左 → 右 | `nums[stk.top()] <= nums[i]` | `ans[i] = stk.top()` |
| 左边第一个严格更小 | 左 → 右 | `nums[stk.top()] >= nums[i]` | `ans[i] = stk.top()` |
| 右边第一个严格更大 | 右 → 左 | `nums[stk.top()] <= nums[i]` | `ans[i] = stk.top()` |
| 右边第一个严格更小 | 右 → 左 | `nums[stk.top()] >= nums[i]` | `ans[i] = stk.top()` |

例如，找严格更大时，若栈顶的值小于或等于当前值，栈顶可以丢掉。对之后扫到的任何位置，只要旧栈顶够大，当前值也够大，而且当前下标更近。找严格更小时，大小关系反过来。

```cpp
#include <stack>
#include <vector>
using namespace std;

enum class Side { Left, Right };
enum class Relation { Greater, Smaller };

vector<int> nearestIndices(const vector<int>& nums, Side side, Relation relation) {
    int n = nums.size();
    vector<int> ans(n, -1);
    stack<int> stk;

    bool findLeft = side == Side::Left;
    int first = findLeft ? 0 : n - 1;
    int end = findLeft ? n : -1;
    int step = findLeft ? 1 : -1;

    for (int i = first; i != end; i += step) {
        while (!stk.empty()) {
            int candidate = nums[stk.top()];
            bool qualifies = relation == Relation::Greater
                ? candidate > nums[i]
                : candidate < nums[i];
            if (qualifies) {
                break;
            }
            stk.pop();
        }
        if (!stk.empty()) {
            ans[i] = stk.top();
        }
        stk.push(i);
    }

    return ans;
}
```

### 为什么两种写法对相等元素的处理不同

- **当前下标给栈顶找答案**时，栈顶还在等自己的答案。当前值和它相等时，这个问题仍未解决，所以保留栈顶。
- **栈顶给当前下标找答案**时，栈顶是候选答案。相等不符合严格条件，可以弹掉。当前下标随后入栈，给接下来扫到的位置提供一个更近的同值候选。

### 用同一组数据检查四种结果

输入 `[2, 1, 2, 4, 3]`，结果记录下标。

| 变体 | 预期输出 |
| --- | --- |
| 左边第一个更大 | `[-1, 0, -1, -1, 3]` |
| 左边第一个更小 | `[-1, -1, 1, 2, 2]` |
| 右边第一个更大 | `[3, 2, 3, -1, -1]` |
| 右边第一个更小 | `[1, -1, -1, 4, -1]` |

本题的最大下标差为 `3`，来自下标 `0 → 3`。
