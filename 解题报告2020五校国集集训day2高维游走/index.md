# [解题报告][2020五校国集集训Day2]高维游走


# Statement
[传送门](https://acm.nflsoj.com/problem/798)

# Solution

## 问题转化
由于是求 $\bmod 2$ 意义下的方案数, 所以我们考虑先用卢卡斯定理来简化问题.

假设我们先枚举出每一维走的步数, 设向量 $w$, 它的第 $i$ 维元素 $w_i$ 表示在第 0 阶段中在第 $i$ 维走的步数. 那么满足向量 $w$ 的游走方案为.

$$
\binom{t_0}{w_1, w_2, \cdots, w_m} \prod_{i = 1}^m \binom{t_i}{w_i}
$$

展开后也就是

$$
\binom{t_0}{w_1} \binom{t_0 - w_1}{w_2} \binom{t_0 - w_1 - w_2 - \cdots - w_{m - 1}}{w_m} \prod_{i = 1}^m \binom{t_i}{w_i}
$$

根据卢卡斯定理 $\binom{n}{m} \equiv \binom{\lfloor n / 2 \rfloor}{\lfloor m / 2 \rfloor} \binom{n \bmod 2}{m \bmod 2} \pmod 2$, 如果我们要使上述式子为 1, 则要满足

1. 对于所有 $i \in [1,m]$, 满足 $w_i$ 按位或 $t_i$ 的结果等于 $t_i$, 这里我们称为 $w_i \subseteq t_i$.
2. 对于所有 $i \in [1, m]$, 满足 $w_i \subseteq t_0 - \sum_{j = 1}^{i - 1} w_j$, 直观上来说, 就是 $w_{1 \cdots m}$ 构成了 $t_0$ 的一个**不完全划分** .

那么我们可以通过一个类似数位DP的过程来计算满足这个条件的游走方案数.

---

## DP

首先很容易想到一个 naive 的 DP: 设 $f_{i,v}$ 表示进行到二进制第 $i$ 为, 得到的总代价为 $v$ 的游走方案在模2意义下的结果. 

转移的话只需枚举每一位取 $1\sim m$ 中的哪一维 (或者不选) 即可, 复杂度为 $O(tm\log t_0)$.

---

## 优化


注意到每一步游走的代价最多只有 $m$, 而 $m \le 10$, 所以对于当前的 $i$, 合法的代价 $v$ 不会大出 $2^i$ 太多. 
并且假设我们把 $v$ 按二进制位考虑, $v$ 中小于 $2^{i + 1}$ 的部分已经固定了, 在之后的转移中都不会改变, 所以我们可以认为只有 $v$ 中大于等于 $2^{i + 1}$ 的部分才是对转移有用的.

那么我们可以考虑把 $v$ 表示成 $v' + q2^{i + 1} (v' \le 2^{i + 1})$ 的形式. 然后因为 $v \le \sum_{j = 1}^i m2^j < m2^{i+1}$, 所以 $q \in [0, m)$.

然后重新设一个DP状态: $f_{i, v', q}' = f_{i,v}$, 转移是类似的.

然后由于后面的转移只与 $q$ 有关, 所以对于一对 $(v_1, v_2)$, 若满足对于所有 $q \in [0, m)$, 都有 $f_{i, v_1,q} = f_{i, v_2, q}$, 那么它们后面的转移方向都是一样的, 所以我们可以考虑把这两个状态合并起来以简化复杂度.

所以我们新设一个DP: $g_{i, S}$ 表示进行到第 $i$ 位, 对于所有 $q \in [0, m)$ 满足 $f_{i,v,q} = S_q$ 的 $v$ 的数量. 其中 $S_q$ 表示 $S$ 的二进制表示下的第 $q$ 位数字.

转移的话, 假设当前得出了 $g_{i - 1, S}$, 我们枚举 $a,b$, 表示 $S$ 中的第 $a$ 位以及当前选择第 $b$ 维进行游走. 那么总代价就是

$$
v' + (a + b)2^{i} = v' + r2^i + ((a + b) >> 1))2^{i + 1} = v' + r2^i + c2^{i + 1}
$$

我们设两个变量 $T_0, T_1$, 若 $r = 0$, 则在 $T_0$ 的第 $c$ 位异或上 1, 否则就在 $T_1$ 的第 $c$ 位异或上 1. 最后转移到 $g_{i, T_0}$ 和 $g_{i, T_1}$ 即可.

这样复杂度为 $O(2^m m^2 \log t_0) \approx 3 \times 10^6$, 再乘上 $200$ 组测试数据, $\approx 6 \times 10^8$, 1s 的时限内显然过不了.

再继续考虑优化转移过程, 我们发现枚举 $a,b$ 后, 得到的 $r$ 只有 $0,1$ 两种可能, 而 $c = (a + b) >> 1 = \lfloor (a + b) / 2 \rfloor$ 也只有两种可能 : $\lfloor a / 2 \rfloor + \lfloor b / 2 \rfloor$ 或 $\lfloor a / 2 \rfloor + \lfloor b / 2 \rfloor + 1$. 只需要把 $a,b$ 分奇偶讨论一下就行了.

所以我们可以设两个变量 $S_0, S_1$, 分别把 $S$ 中每个 $a$ 为偶和 $a$ 为奇的情况记录下来, 然后枚举每个 $b$, 按奇偶讨论转移即可.

复杂度为 $O(2^m m \log t_0) \approx 3 \times 10^5$. 在 $200$ 组数据的情况下可以通过.

## Code
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = 10 + 7;
const int __ = (1 << 10) + 7;
const int L = 30;

int m, t[_];
long long g[__], h[__];

void Solve() {
  cin >> m;
  for (int i = 0; i <= m; ++i) scanf("%d", &t[i]);
  memset(g, 0, sizeof g), memset(h, 0, sizeof h);
  g[1] = 1;
  for (int i = 0; i <= L; ++i) {
    for (int s = 1; s < (1 << m); ++s) {
      int s0 = 0, s1 = 0, t0 = 0, t1 = 0;
      for (int a = 0; a < m; ++a) (a & 1 ? s1 : s0) |= (s >> a & 1) << (a / 2);
      for (int b = 0; b <= m; ++b) {
        if (b and !((t[b] & t[0]) >> i & 1ll)) continue;
        if (b & 1) t0 ^= s1 << (b / 2 + 1), t1 ^= s0 << (b / 2);
        else t0 ^= s0 << (b / 2), t1 ^= s1 << (b / 2);
      }
      h[t0] += g[s], h[t1] += g[s];
    }
    memcpy(g, h, sizeof h), memset(h, 0, sizeof h);
  }

  long long ans = 0;
  for (int i = 1; i < (1 << m); ++i)
    for (int j = 0; j < m; ++j)
      if (i >> j & 1) ans += g[i];
  cout << ans << endl;
}

int main() {
  int T; cin >> T;
  while (T--) Solve();
  return 0;
}
```

