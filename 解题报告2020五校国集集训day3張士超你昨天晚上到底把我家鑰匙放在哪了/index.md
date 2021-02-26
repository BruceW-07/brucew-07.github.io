# [解题报告][2020五校国集集训Day3]張士超你昨天晚上到底把我家鑰匙放在哪了？


# Statement
[传送门](https://acm.nflsoj.com/contest/64/problem/803)

# Solution

考虑从朴素算法开始一步步推导。

我们考虑枚举做对的题目集合 $T$，并计算满足集合 $T$ 中题目总分大于等于 $n$ 的分数分配方案数。即

$$
\begin{aligned}
\\mathrm{Ans}
&= \sum_{T} \prod_{i \in T}p_i \prod_{i \not \in T} (1 - p_i) 【T 中题目总分大于等于 n 的合法分配方案数】 \\\\ 
&= \sum_{T} p(T) 【T 中题目总分大于等于 n 的合法分配方案数】 \\\\ 
\end{aligned}
$$

而题目中对于“合法”的要求是：第 $i$ 道题目分配的分数不能超过 $a_i$。

给题目分配分数可以看做把“小球放进盒子”的模型，而对于这种“盒子对小球的个数有限制”的模型，我们一般会用容斥来计算方案数。

具体来说，就是枚举盒子的每个子集 $A$，钦定 $A$ 中的盒子超出了限制，然后其他盒子就随便放。

</br>

因此我们的式子可以化为。

$$
\begin{aligned}
\\mathrm{Ans}
&= \sum_{T} p(T) \sum_{A} (-1)^{|A|} 【将 N' 个球放进 M 个盒子，且 T 中的盒子的球总数大于等于 n' 的方案数】\\\\ 
\end{aligned}
$$

其中 $N' = N - \sum_{i \in A} (a_i + 1),\ n' = n - \sum_{i \in A \cap T} (a_i + 1)$。

</br>

现在考虑怎么计算“【】”里描述的那个东西。

这种把球放进盒子里的方案数，我们通常会考虑隔板法。

然后这里为了方便描述，我们设前 $|T|$ 个隔板分别表示集合 $T$ 中的题目。因为球都是一样的，所以这样显然不会影响方案数的计算。

那么为了使得这 $|T|$ 个盒子中球数总和大于等于 $n'$，我们就要让第 $|T|$ 个隔板放在第 $n'$ 个球后面。

所以我们枚举第 $n'$ 个球之前的隔板数量，分别计算隔板放置方案，并求和即可。即

$$
\sum_{t=0}^{|T|-1} \binom{n' + t - 1}{t} \binom{(N' + 1 - n') + (M - 1 - t) - 1}{M - 1 - t} 
= \sum_{t=0}^{|T|-1} \binom{n' + t - 1}{t} \binom{N' - n' + M - 1 - t}{M - 1 - t}
$$

那么答案的计算式就可以进一步化为

$$
\begin{aligned}
\\mathrm{Ans}
&= \sum_{T} p(T) \sum_{A} (-1)^{|A|} \sum_{t=0}^{|T|-1} \binom{n' + t - 1}{t} \binom{N' - n' + M - 1 - t}{M - 1 - t} \\\\ 
\end{aligned}
$$

现在我们可以开始考虑如何计算了。

首先 $0 \sim |T| - 1$ 属于可以接受的枚举范围，而前面的枚举 $T$ 和 $A$ 的操作我们可以考虑用 DP 计算系数。

所以大概思路就是对于每一个 $\sum_{t=0}^{|T|-1}$ 的组合数求和操作，我们计算出它的系数，然后把所有结果累加。

那么我们就要观察这个组合数求和需要知道哪些变量，并根据这些变量来设置 DP 状态。

$\sum_{t=0}^{|T|-1} \binom{n' + t - 1}{t} \binom{N' - n' + M - 1 - t}{M - 1 - t}$ 这个式子中，只有 $M$ 是已知的，所以我们还需要知道 $|T|,N',n'$。

所以我们设 $f_{i,j,k,l}$ 表示考虑到第 $i$ 道题，$|T|=j, N'=k, n'=l$ 的系数总和。转移只需枚举第 $i$ 道题目是否加入 $T$ 集合以及是否加入 $A$ 集合即可。

然后由于 $N'$ 等于 $N$ 减去若干个 $a_i + 1$，$n'$ 等于 $n$ 减去若干个 $a_i + 1$，而题目中有条件：存在 $d$，满足对于所有 $i$ 都有 $d | (a_i+1)$，并且 $N \le 100 \times d$。

所以我们可以把 $k,l$ 的意义改为 $N' = N - k \times d, n' = n - l \times d$。这样 DP 的复杂度就降为 $O(M^2 (\frac{N}{d})^2) \approx 10^8$。

而最后统计答案的复杂度也是 $O(M^2 (\frac{N}{d})^2)$ 的，所以总复杂度也是 $O(M^2 (\frac{N}{d})^2)$ 的。

最后统计答案，组合数求和时，可以需要特判一下 $n' \le 0$ 的情况。

# Summary

这道题的解题逻辑还是比较清晰的，唯一不太想得到的可能就只有【将 N' 个球放进 M 个盒子，且 T 中的盒子的球总数大于等于 n' 的方案数】这个东西。

但问题是推式子的时候比较难冷静下来，而且容易去想一些奇奇怪怪的东西，从而绕很多弯路，而这种推导的时候一旦方向偏了就比较难搞了。

# Code
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = 1e2 + 7;
const int mod = 998244353;

int M, d, N, n, a[_], aa[_], p[_], f[_][_][_], g[_][_][_], inv[_], C1[_], C2[_], ans;

int main() {
  cin >> M >> d >> N >> n;
  for (int i = 1; i <= M; ++i) {
    scanf("%d%d", &a[i], &p[i]);
    aa[i] = (a[i] + 1) / d;
  }

  inv[1] = 1;
  for (int i = 2; i <= M; ++i) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
  
  f[0][0][0] = 1;
  for (int i = 1; i <= M; ++i) {
    for (int j = 0; j <= i; ++j)
      for (int k = 0; k <= N / d; ++k) {
        for (int l = 0; l <= k; ++l) {
          unsigned long long t1 = 0, t2 = 0, t3 = 0, t4 = 0;
          if (j) t1 = 1ll * f[j - 1][k][l] * p[i];
          if (j and l >= aa[i] and k >= aa[i]) t2 = 1ll * f[j - 1][k - aa[i]][l - aa[i]] * p[i];
          if (k >= aa[i]) t3 = 1ll * f[j][k - aa[i]][l] * (1 - p[i] + mod);
          t4 = 1ll * f[j][k][l] * (1 - p[i] + mod);
          g[j][k][l] = ((t1 + t4) % mod + mod - (t2 + t3) % mod) % mod;
        }
      }
    memcpy(f, g, sizeof f), memset(g, 0, sizeof g);
  }
  
  for (int j = 1; j <= M; ++j)
    for (int k = 0; k <= N / d; ++k)
      for (int l = 0; l <= k; ++l) {
        if (l * d > N) continue;
        int NN = N - k * d, nn = max(0, n - l * d);
        if (nn > NN) continue;
        C1[0] = 1, C2[j - 1] = 1;
        for (int t = 1; t <= j - 1; ++t) C1[t] = 1ll * C1[t - 1] * (nn - 1 + t) % mod * inv[t] % mod;
        for (int t = 1; t <= M - 1 - (j - 1); ++t) C2[j - 1] = 1ll * C2[j - 1] * (NN - nn + t) % mod * inv[t] % mod;
        for (int t = j - 2; t >= 0; --t) C2[t] = 1ll * C2[t + 1] * (NN - nn + M - 1 - t) % mod * inv[M - 1 - t] % mod;
        if (nn == 0) { ans = (ans + 1ll * C2[0] * f[j][k][l] % mod) % mod; continue; }
        int res = 0;
        for (int t = 0; t < j; ++t) res = (res + 1ll * C1[t] * C2[t]) % mod;
        ans = (ans + 1ll * res * f[j][k][l] % mod) % mod;
      }
  cout << ans << endl;
  return 0;
}
```

