参考：

1、[Ross-Ning](https://www.bilibili.com/video/BV1AY4y157yL/?spm_id_from=333.1387.upload.video_card.click&vd_source=bf9acd6f8d387d1b0d89b4c9d8e787c8)

2、算法导论[introduction to algorithms]

算法导论-part vii-string matching章节：

32.1-the naive string matching algorithm：暴力求解。

32.2-the rabin-karp algorithm：像hashmap那样给pattern和text的string算出一个value，通过比较value是否相等来匹配。

32.3-string matching with finite automata：利用有限状态自动机(deterministic finite-state automaton，简称DFA)来求解，每读入一个char，利用transition function = δ(current state, char)算出该跳转到哪个state上，algs4里讲的就是这个方法。

32.4-the knuth-morris-pratt algorithm：维护一个next数组(longest prefix suffix，简称LPS，算法导论里叫prefix function = π[q])，每次遇到不匹配的时候，就利用next中的值调整pattern到合适的位置。

算法导论里把finite automata和kmp归类为相似的方法是因为finite automata 和 KMP 都利用了模式串前缀与后缀的重叠信息，但它们不是同一个算法。finite automata 显式构造 DFA 的 transition function δ(q, a)；KMP 则不显式构造完整 DFA，而是用 prefix function π 在失配时快速回退状态。KMP 是 finite automata 思想的一种压缩实现 / 高效实现。

算法导论中的伪代码暗示了求解next数组(search pattern in pattern)和kmp算法(search pattern in text)思路是一致的。在KMP-MATCHER中，i 扫描 text，q 表示 pattern 已匹配长度，在COMPUTE-PREFIX-FUNCTION中，q 扫描 pattern 的右边部分(相当于前面的i)，k 表示 pattern 中当前已匹配的前缀长度(相当于前面的q)。

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

以参考1里的next为例：

next数组：
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
