# Wait Until the Next Shuttle

给定按时间排序的当天班车时刻表和当前时间，找到第一班尚未发车的车。恰好在当前时间发车也可以乘坐。等待时间用两位小时和两位分钟表示。如果当天没有可乘坐的班车，返回 `"-1"`。

## 思路

1. 时间都是固定长度、补零的 `HH:MM`，所以字符串比较的顺序和时间顺序一致。顺序扫描，第一个满足 `current_time <= departure` 的班次就是答案。
2. 分别计算小时差和分钟差。分钟差为负时，从小时差借一个小时，再给分钟差加 60。例如 `14:55 → 15:05` 的差值先得到 1 小时和 −50 分钟，借位后是 `00:10`。
3. 先把小时差和分钟差转为字符串，再给不足两位的部分补零。

## 这次踩的坑

1. **在单个时间字符串上调用 `.substr()`。** `shuttle_times` 是整个 vector。循环里的 `departure` 才是一条班车时间，相当于 `shuttle_times[i]`。
2. **先赋值，再补零。** 如果只在数字小于 10 时给字符串赋值，两位数就会留下空字符串。例如，`05:25` 会变成 `05:`。
3. **拼接变量时去掉引号。** `a + ":" + b` 拼接分钟字符串，写成 `a + ":" + "b"` 会在末尾输出字母 b。
4. **比较时包含等号。** 当前时间与发车时间相同，等待时间为 `00:00`。

## C++17

```cpp
#include <string>
#include <vector>

using namespace std;

class Solution {
public:
    string nextShuttleWait(vector<string>& shuttle_times, string current_time) {
        for (const string& departure : shuttle_times) {
            if (departure < current_time) continue;

            int hours = stoi(departure.substr(0, 2))
                      - stoi(current_time.substr(0, 2));
            int minutes = stoi(departure.substr(3))
                        - stoi(current_time.substr(3));

            if (minutes < 0) {
                --hours;
                minutes += 60;
            }

            string hh = to_string(hours);
            string mm = to_string(minutes);
            if (hours < 10) hh = "0" + hh;
            if (minutes < 10) mm = "0" + mm;
            return hh + ":" + mm;
        }

        return "-1";
    }
};
```

## 测试

| 班车时刻 | 当前时间 | 等待时间 |
| --- | --- | --- |
| `["12:30", "14:00", "19:55"]` | `14:30` | `05:25` |
| `["15:05"]` | `14:55` | `00:10` |
| `["14:00", "14:00", "19:55"]` | `14:00` | `00:00` |
| `["12:34"]` | `00:00` | `12:34` |
| `["19:55"]` | `20:00` | `-1` |
| `[]` | `20:00` | `-1` |

## 复杂度

- 时间 `O(n)`。每个时间字符串的长度固定。
- 额外空间 `O(1)`。
