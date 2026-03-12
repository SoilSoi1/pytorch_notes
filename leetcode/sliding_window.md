# 滑动窗口
---
## 技巧总结

滑动窗口模版：
```python
while r < len(s):

    # 右扩
    c = s[r]
    r += 1

    update window

    # 左缩
    while window needs shrink:

        update answer

        d = s[l]
        l += 1

        update window
```

differ方法：
```
function solve(s, p):

    n = len(s)
    m = len(p)

    if n < m:
        return []

    diff[alphabet_size] = 0

    # 1 初始化 diff
    for i in 0..m-1:
        diff[s[i]] += 1
        diff[p[i]] -= 1

    # 2 计算 differ
    differ = count(diff[i] != 0)

    if differ == 0:
        record answer

    # 3 滑动窗口
    # 当 diff 穿过 0 时, differ 改变
    for i in 0..n-m-1:

        remove = s[i]
        add = s[i+m]

        update(remove, -1)   # 移出字符
        update(add, +1)      # 加入字符

        if differ == 0:
            record answer
```

---
## 76.最小覆盖字串

```
给定两个字符串 s 和 t，长度分别是 m 和 n，返回 s 中的 最短窗口 子串，使得该子串包含 t 中的每一个字符（包括重复字符）。如果没有这样的子串，返回空字符串 ""。

测试用例保证答案唯一。

示例 1：

输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
示例 2：

输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
示例 3:

输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```

---

对于滑动窗口，最稳妥的方法是维护两个哈希表：

**need**: 记录 $t$ 中每个字符出现的次数

**window**: 记录当前滑动窗口中各字符出现的次数

以及两组关键参数：

**l, r**: 左右边界

**valid**: 记录窗口里达标的**字符种类**数

---

首先初始化参数：  

```python
need = Counter(t)
window = {}
l, r = 0, 0
valid = 0

start = 0
min_len = float('inf') # 无穷，方便后面更新最小长度
```
然后右指针前进：
```python
while r < len(s):
    char_r = s[r] # 记录此刻参数其实是很重要的，尽量别让动态参数参与更改
    r += 1
    
    if char_r in need:
        window[char_r] = window.get(char_r, 0) + 1
        if window[char_r] == need[char_r]:
            valid += 1 # 某种字符够了就加1
```

一旦```valid == len(need)```，说明此时的窗口已经包含所有的 ```t``` 了，所以左指针可以开始收缩。

```python
while valid == len(need):
        if r - l < min_len:
            start = l
            min_len = r - l
        
        char_l = s[l]
        l += 1
        
        # 如果移出的字符是关键字符，得更新状态
        if char_l in need:
            # 在这过程中会有可能valid不再满足need，此时valid减1，是为了跳出while
            if window[char_l] == need[char_l]:
                valid -= 1 

            window[char_l] -= 1
```

返回：

```python
return "" if min_len == float('inf') else s[start : start + min_len]
```
---

## 438.找到字符串中所有字母异位词
```text
给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

 

示例 1:

输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
 示例 2:

输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
 

提示:

1 <= s.length, p.length <= 3 * 104
s 和 p 仅包含小写字母
```

这题和76题很相似，唯一需要注意的是需要限制l和r的相对位置，附上代码：
```python
from collections import Counter
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        need = Counter(p)
        window = {}
        l = 0
        r = 0
        valid = 0

        start = 0
        log = []

        while r < len(s):
            char_r = s[r]
            # r += 1的好处有两点：
            # 主循环while可以在“r < len(s)”的条件下走完；
            # 此时"r - l"正好是 两对应元素 的长度

            # 初始化window哈希表
            if char_r in need:
                window[char_r] = window.get(char_r, 0) + 1
                if window[char_r] == need[char_r]:
                    valid += 1
            r += 1

            if r - l == len(p):
                if valid == len(need):
                    log.append(l)

                char_l = s[l]
                l += 1
                if char_l in need:
                    if window[char_l] == need[char_l]:
                        valid -= 1
                    window[char_l] -= 1
        return log
```

官方给的解法，只用维护diff这个参数：
```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        s_len, p_len = len(s), len(p)

        if s_len < p_len:
            return []

        ans = []
        count = [0] * 26
        for i in range(p_len):
            count[ord(s[i]) - 97] += 1
            count[ord(p[i]) - 97] -= 1

        differ = [c != 0 for c in count].count(True)

        if differ == 0:
            ans.append(0)

        for i in range(s_len - p_len):
            if count[ord(s[i]) - 97] == 1:  # 窗口中字母 s[i] 的数量与字符串 p 中的数量从不同变得相同
                differ -= 1
            elif count[ord(s[i]) - 97] == 0:  # 窗口中字母 s[i] 的数量与字符串 p 中的数量从相同变得不同
                differ += 1
            count[ord(s[i]) - 97] -= 1

            if count[ord(s[i + p_len]) - 97] == -1:  # 窗口中字母 s[i+p_len] 的数量与字符串 p 中的数量从不同变得相同
                differ -= 1
            elif count[ord(s[i + p_len]) - 97] == 0:  # 窗口中字母 s[i+p_len] 的数量与字符串 p 中的数量从相同变得不同
                differ += 1
            count[ord(s[i + p_len]) - 97] += 1
            
            if differ == 0:
                ans.append(i + 1)

        return ans
```
首先，`ord(s[i]) - 97`指的是a-z的ascii编码，a是97以此类推至z是122，所以这样操作下来，列表[0, 1, ..., 25]就分别对应上26个字母。

s中有的就在对应的位置+1，p中有的在对应位置-1，再利用核心代码`differ = [c != 0 for c in count].count(True)`去计算，不同的个数为多少。

最后，在滑动窗口时，左侧的即将被移出窗口，那代表这个字母对应位置即将-1，如果该字母对应的数量为1，那证明此时differ可以-1（不管后面加进来的是什么）。那同理，进行四个条件操作之后，修改count本身即可。