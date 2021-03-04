# [解题报告][NFLSOJ529][2020六校联合训练省选2]打卡任务


校内考试，不予外传。

<!--more-->

{{% hugo-encryptor "xxx" %}}

# 这里是你要加密的内容!

这里是你要加密的内容!

**别忘了闭合 `hugo-encryptor` shortcode 标签:**

{{% /hugo-encryptor %}}


## Statement

[传送门](https://acm.nflsoj.com/problem/529)


从大小为 $n$ 的正整数集合 $\\{1, 2, 3, \cdots, n\\}$ 中选出一个大小为 $m$ 的子集 $T$。

求满足 $T$ 中的数之和在模 $n$ 意义下等于 $k$ 的 $T$ 的个数。

答案对 $998244353$ 取模。

$0 \le k < n < 998244353, 1 \le m \le n$。

## Solution

先把答案用二元生成函数的形式表示出来，即

$$
\mathrm{ans} = [y^m][x^k] \prod_{i = 0}^{n - 1} (1 - x^iy) \pmod{(x^n - 1)}
$$

（$ \mod{(x^n - 1)} $即为循环卷积。）

然后由于 $x$ 这一维是循环卷积，所以我们考虑引入单位根，用 DFT、IDFT 的形式来表示答案。

回顾一下 IDFT 的过程。设 $A$ 为多项式，$B$ 为 $A$ 在 $\omega_{n}^{0 \sim n - 1}$ 处的点集，则有

$$

$$

