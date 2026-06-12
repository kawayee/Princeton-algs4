### 参考：

- 1、[Ross-Ning](https://www.bilibili.com/video/BV1AY4y157yL/?spm_id_from=333.1387.upload.video_card.click&vd_source=bf9acd6f8d387d1b0d89b4c9d8e787c8)

- 2、算法导论[introduction to algorithms]

### 算法导论-part vii-string matching章节：

- 32.1-the naive string matching algorithm：暴力求解。

- 32.2-the rabin-karp algorithm：像hashmap那样给pattern和text的string算出一个value，通过比较value是否相等来匹配。

- 32.3-string matching with finite automata：利用有限状态自动机(deterministic finite-state automaton，简称DFA)来求解，每读入一个char，利用transition function = δ(current state, char)算出该跳转到哪个state上，algs4里讲的就是这个方法。

- 32.4-the knuth-morris-pratt algorithm：维护一个next数组(longest prefix suffix，简称LPS，算法导论里叫prefix function = π[q])，每次遇到不匹配的时候，就利用next中的值调整pattern到合适的位置。

- 算法导论里把finite automata和kmp归类为相似的方法是因为finite automata 和 KMP 都利用了模式串前缀与后缀的重叠信息，但它们不是同一个算法。finite automata 显式构造 DFA 的 transition function δ(q, a)；KMP 则不显式构造完整 DFA，而是用 prefix function π 在失配时快速回退状态。KMP 是 finite automata 思想的一种压缩实现 / 高效实现。

- 算法导论中的伪代码暗示了求解next数组(search pattern in pattern)和kmp算法(search pattern in text)思路是一致的。在KMP-MATCHER中，i 扫描 text，q 表示 pattern 已匹配长度，在COMPUTE-PREFIX-FUNCTION中，q 扫描 pattern 的右边部分(相当于前面的i)，k 表示 pattern 中当前已匹配的前缀长度(相当于前面的q)。

```python
KMP-MATCHER(T, P, n, m)
1  π = COMPUTE-PREFIX-FUNCTION(P, m)
2  q = 0                                  // number of characters matched
3  for i = 1 to n                         // scan the text from left to right
4      while q > 0 and P[q + 1] ≠ T[i]
5          q = π[q]                       // next character does not match
6      if P[q + 1] == T[i]
7          q = q + 1                      // next character matches
8      if q == m                          // is all of P matched?
9          print "Pattern occurs with shift" i - m
10         q = π[q]                       // look for the next match

COMPUTE-PREFIX-FUNCTION(P, m)
1  let π[1 : m] be a new array
2  π[1] = 0
3  k = 0
4  for q = 2 to m                          // scan the pattern from left to right
5      while k > 0 and P[k + 1] ≠ P[q]
6          k = π[k]                        // next character does not match
7      if P[k + 1] == P[q]
8          k = k + 1                       // next character matches
9      π[q] = k
10 return π
```
### 举例：

以参考1的next数组为例：

| A | B | A | C | A | B | A | B |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 0 | 0 | 1 | 0 | 1 | 2 | 3 | 2 |

当前状态：
```
          i
          ↓
A B A C A B A B -> text
        A B A C A B A B -> pattern
          ↑
          prefixLen
```
若下一项匹配：
```
            i
            ↓
A B A C A B A B -> text
        A B A C A B A B -> pattern
            ↑
            prefixLen
```
若下一项不匹配：
```
              i
              ↓
A B A C A B A B -> text
        A B A C A B A B -> pattern
              ↑
              prefixLen
查看next[prefixLen-1]=next[3-1]=1，移动prefixLen=1，相当于移动pattern。
======================================================================
              i
              ↓
A B A C A B A B -> text
            A B A C A B A B -> pattern
              ↑
              prefixLen
```
### 代码：
```python
def build_lps(pattern: str) -> list[int]:
    """
    构建 KMP 算法中的 lps 数组。
    lps 的含义：
        lps[i] 表示 pattern[0:i+1] 这个子串中，最长的“相等真前缀和真后缀”的长度。pattern[0:i+1]是python中的切片写法(左闭右开)，等价于数学中的闭区间pattern[0,i]。
        lps[i] 表示 以索引 i 结尾的子串的最长相等真前后缀长度，这个子串包含了从索引 0 开始，一直到索引 i（包含 i）的所有字符。

    例如：
        pattern = "ababaca"
        子串        最长相等前后缀    lps
        "a"         无              0
        "ab"        无              0
        "aba"       "a"             1
        "abab"      "ab"            2
        "ababa"     "aba"           3
        "ababac"    无              0
        "ababaca"   "a"             1

        所以：
        lps = [0, 0, 1, 2, 3, 0, 1]

    时间复杂度：
        O(m)，m 是 pattern 的长度
    """

    m = len(pattern)
    lps = [0] * m

    # length 表示当前已经匹配到的“最长相等前后缀”的长度
    # 也可以理解为：当前准备和 pattern[i] 比较的位置
    length = 0

    # i 从 1 开始，因为 lps[0] 一定是 0
    i = 1

    while i < m:
        # 情况 1：当前字符 pattern[i] 和 pattern[length] 相等
        # 说明最长相等前后缀可以继续延长一位
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1

        else:
            # 情况 2：pattern[i] 和 pattern[length] 不相等
            # 如果 length > 0，说明前面还有可回退的位置。
            # 不能直接把 length 变成 0，而是尝试缩短当前前后缀长度。
            #
            # 为什么是 lps[length - 1]？
            # 因为 pattern[0:length] 这段已经是一个相等前后缀，
            # 现在要在这段里面继续找更短的相等前后缀。
            if length > 0:
                length = lps[length - 1]

            else:
                # 情况 3：length 已经是 0，说明没有任何可保留的前后缀。
                # 当前 i 位置的 lps 就是 0，继续看下一个字符。
                lps[i] = 0
                i += 1
    return lps

def kmp_search(text: str, pattern: str) -> int:
    """
    使用 KMP 算法在 text 中查找 pattern 第一次出现的位置。
    返回值：
        如果找到，返回 pattern 在 text 中第一次出现的起始下标；
        如果没找到，返回 -1。
    核心思想：
        当 text[i] 和 pattern[j] 匹配失败时，
        不让 i 回退，而是根据 lps 数组移动 j。
    时间复杂度：
        O(n + m)
        n 是 text 的长度，m 是 pattern 的长度
    """

    # 按照 Python str.find 的习惯：空模式串默认匹配在位置 0
    if pattern == "":
        return 0

    n = len(text)
    m = len(pattern)

    # 如果模式串比主串还长，肯定匹配不上
    if m > n:
        return -1

    # 先构建模式串的 lps 数组
    lps = build_lps(pattern)

    # i 指向 text 当前比较的位置
    # j 指向 pattern 当前比较的位置
    i = 0
    j = 0

    while i < n:
        # 情况 1：当前字符匹配，两个指针都往后走
        if text[i] == pattern[j]:
            i += 1
            j += 1

            # 如果 j == m，说明整个 pattern 都匹配完了
            if j == m:
                return i - m

        else:
            # 情况 2：当前字符不匹配
            # 如果 j > 0，说明 pattern 前面已经有一部分匹配成功。
            # 此时不移动 i，只让 j 根据 lps 回退。
            if j > 0:
                j = lps[j - 1]

            else:
                # 情况 3：j == 0，说明 pattern 的第一个字符就没匹配上。
                # 此时没有任何可保留的信息，只能移动 i。
                i += 1

    # text 扫描完了还没匹配成功
    return -1

def kmp_search_all(text: str, pattern: str) -> list[int]:
    """
    使用 KMP 算法查找 pattern 在 text 中出现的所有起始位置。

    例如：
        text = "aaaaa"
        pattern = "aa"

        匹配位置是：
        [0, 1, 2, 3]

    注意：
        这里支持重叠匹配。

    时间复杂度：
        O(n + m)
    """

    if pattern == "":
        return list(range(len(text) + 1))

    n = len(text)
    m = len(pattern)

    if m > n:
        return []

    lps = build_lps(pattern)

    result = []

    i = 0
    j = 0

    while i < n:
        if text[i] == pattern[j]:
            i += 1
            j += 1

            if j == m:
                # 找到一次完整匹配
                result.append(i - m)

                # 继续寻找下一次匹配
                #
                # 这里不能直接把 j 设为 0，
                # 因为可能存在重叠匹配。
                #
                # 例如：
                # text    = "aaaaa"
                # pattern = "aa"
                #
                # 匹配完 text[0:2] 后，
                # 下一次匹配可以从 text[1] 开始。
                j = lps[j - 1]

        else:
            if j > 0:
                j = lps[j - 1]
            else:
                i += 1

    return result
```
