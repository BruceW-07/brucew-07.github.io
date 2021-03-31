# [解题报告][CF1007E]Mini Metro


## Statement

[传送门](https://codeforces.com/problemset/problem/1007/E)

有 $n$ 个车站，从 $1$ 到 $n$ 编号，车站 $i$ 初始有 $a_i$ 个人。 

在每个小时结束的前几分钟，车站 $i$ 会新增 $b_i$ 个人。

玩家有无限辆容量为 $k$ 的火车。

玩家在每个小时的中间 (也就是 $\mathrm{30min, 1h30min, 2h30min...}$) 可以让任意辆火车从始发站 $1$ 按照 $1, 2, 3, \cdots n$ 的顺序驶向终点站 $n$。 ($x$ 辆同时发动的火车可看做一辆容量 $x \times k$ 的火车。)

每辆火车会尽可能的载上它经过站点的所有人。也就是说，如果一辆火车搭载了站点 $i$ 的人，则在它经过后，站点 $1 \sim i - 1$ 的人数都为 $0$。

若某个时刻站点 $i$ 的人数超过 $c_i$，则该站台会发生暴乱。

求在保证 $t$ 个小时内所有站台都不发生暴乱情况下，玩家派出的火车的最小数量。

## Solution

考虑从前往后把站台一个个加进来，在此基础上进行 DP。 

为了方便描述，我们在第 $n + 1$ 位加上一个人数无限多的站台，并设 $sa$ 为 $a$ 的前缀和，$sb$ 为 $b$ 的前缀和。

设 DP 状态

1. $f_{i, s, z}$ 表示考虑到前 $i$ 个站台，初始人数为 $z \cdot a_i (z \in \{ 0, 1 \})$，能撑 $s$ 轮，且每辆火车都<font color=red>载满 $k$ 个人</font>的最小代价。若无解则为 $\infty$。

2. $g_{i, s, z}$ 表示在上述条件下满足第 $s$ 轮后站台 $1 \sim i - 1$ 的人数都为 $0$ 的最小代价。若无解则为 $\infty$。


「<font color=red>载满 $k$ 个人</font>」就是保证进行的所有操作不会影响到第 $i + 1$ 个站台；而 $g_{i, s, z}$ 就相当于在满足 $f_{i, s, z}$ 的情况下多派几辆火车，使得接下来能够直接访问站台 $i$。

易得初始值 $f_{0, 0, 1} = 0$，答案为 $f_{n, z, 1}$。

转移的话我们分类讨论在第 $s$ 小时之前是否到过站台 $i$。 

- 在 $s$ 小时之前未到过站台 $i$。

   那么此时我们需要保证在 $s$ 小时内站台 $i$ 不会发生暴乱，并且 $f_{i - 1, s, z} \not = \infty$。
   
   有转移 

   $$
   \begin{aligned}
   f_{i, s, z} &\leftarrow f_{i - 1, s, z}  \\\\ 
   g_{i, s, z} &\leftarrow num = \left\lceil \frac{z \cdot sa_{i - 1} + s \cdot sb_{i - 1}}{k} \right\rceil
   \end{aligned}
   $$
   
   而对于 $g$ 的转移还有条件 $k \cdot num \le z \cdot sa_i + s \cdot b_i$，即保证不会影响到站台 $i + 1$。
   
- 上一次在 $r$ 小时到达了站台 $i$。

   那么我们的转移可以分为三个阶段。
   
   1. 在 $r$ 小时到达了站台 $i$。
   2. 在 $r$ 小时再额外派 $x$ 辆火车，使得站台 $i$ 在接下来 $s - r$ 个小时不会发生暴乱。
   3. 维护前 $i - 1$ 个站台在 $s - r$ 个小时内不会发生暴乱，且前 $i - 1$ 个站台的初始人数都为 $0$。
   
   对于第 1 阶段, 因为到达了站台 $i$，所以前 $i - 1$ 的站台的人数一定都为 $0$，因此代价即为 $g_{i, r, z}$。
   
   对于第 2 阶段，我们可以得到此时站台 $i$ 的人数 $m = z \cdot sa_i + r \cdot sb_i - k \cdot g_{i, r, z}$，因此可以得到 $x = \max(0, \left\lceil \frac{m + (s - r) \cdot b_i - c_i}{k} \right\rceil)$。此时还要判断一下若 $k \cdot x > m$，则不能进行转移。
   
   对于第 3 阶段，我们需要对 $f, g$ 分别讨论
   
   - 对于 $f$，只需要保证这前 $i - 1$ 个站台在初始值为 $0$ 的情况下在 $s - r$ 个小时内不发生暴乱，即为 $f_{i - 1, s - r, 0}$。
   - 对于 $g$，我们在保证不发生暴乱的前提下需要把前 $i - 1$ 个站台清空，则代价为 $num = \left\lceil \frac{(s - r) \cdot sb_{i - 1}}{k} \right\rceil$。转移的条件与第一种情况中 $g$ 的转移类似， 即 $f_{i - 1, s - r, 0} \not = \infty$ 且 $k \cdot num \le m - k \cdot x + (s - r) \cdot b_i$。
   
   所以总的转移即为
   
   $$
   \begin{aligned}
   f_{i, s, z} &\leftarrow g_{i, r, z} + x + f_{i - 1, s - r, 0}  \\\\ 
   g_{i, s, z} &\leftarrow g_{i, r, z} + x + \left\lceil \frac{(s - r) \cdot sb_{i - 1}}{k} \right\rceil
   \end{aligned}
   $$

总时间复杂度为 $O(nt^2)$。

注意要开 $long\ long$。

## Code
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

typedef long long ll;

const int _ = 200 + 7;
const ll inf = 1e15;

int n, T, K;
ll a[_], b[_], c[_], sa[_], sb[_], d[_][_][2], g[_][_][2];

ll Ceil(ll x, ll y) { return x % y ? x / y + 1 : x / y; }

void upd(ll &x, ll y) { x = min(x, y); }

int main() {
  cin >> n >> T >> K;
  for (int i = 1; i <= n; ++i) {
    cin >> a[i] >> b[i] >> c[i];
    sa[i] = sa[i - 1] + a[i];
    sb[i] = sb[i - 1] + b[i];
  }
  a[++n] = inf, c[n] = inf;
  sa[n] = sa[n - 1] + a[n];

  for (int p = 1; p <= n; ++p)
    for (int s = 0; s <= T; ++s)
      for (int z = 0; z <= 1; ++z) 
        d[p][s][z] = g[p][s][z] = inf;

  for (int p = 1; p <= n; ++p)
    for (int s = 0; s <= T; ++s)
      for (int z = 0; z <= 1; ++z) {
        if (z * a[p] + s * b[p] <= c[p] and d[p - 1][s][z] != inf) {
          upd(d[p][s][z], d[p - 1][s][z]);
          ll num = Ceil(z * sa[p - 1] + s * sb[p - 1], K);
          if (num * K <= z * sa[p] + s * sb[p]) upd(g[p][s][z], num);
        }
        for (int r = 0; r < s; ++r)
          if (g[p][r][z] != inf and d[p - 1][s - r][0] != inf) {
            ll m = z * sa[p] + r * sb[p] - K * g[p][r][z];
            ll x = Ceil(max(0ll, m + (s - r) * b[p] - c[p]), K);
            if (K * x <= m) {
              upd(d[p][s][z], g[p][r][z] + x + d[p - 1][s - r][0]);
              ll num = Ceil((s - r) * sb[p - 1], K);
              if (num * K <= m - K * x + (s - r) * sb[p]) upd(g[p][s][z], g[p][r][z] + x + num);
            }
          }
      }

  cout << d[n][T][1] << endl;
  return 0;
}
```

