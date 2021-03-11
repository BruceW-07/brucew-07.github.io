# [解题报告][PKUWC2018]随机游走


## Statement

[传送门](https://loj.ac/p/2542)

给定一棵节点数为 $n$ 的树, 以及根节点 $X$.

有 $Q$ 次询问, 每次给定一个点集 $S$, 求从 $X$ 开始随机游走 (每次随机走向与 $u$ 相连的一个点), 经过 $S$ 内所有点至少一次的期望步数.

$n \le 18, Q \le 5000$.


## Solution

这题可以看做一种套路的模板, 即:「在树 (特殊图) 上随机游走, 第一次到达某个点集的期望步数」. 主要思路是消元.

这种套路求的是 「**第一次**到达点集的期望步数」, 而这题求的是「到点集内**每个节点**至少一次」, 相当于一个是到达该点集的时间最小值, 一个是到达该点集的时间最大值, 所以我们可以先套个 $\min-\max$ 容斥. 即

$$
\max(S) = \sum_{T \subseteq S} (-1)^{|T| - 1} \min(T)
$$

所以现在我们只需要求第一次到达某个点集的期望步数, 再做个高维前缀和就行了.

然后就可以按照套路来做了.

设当前点集为 $S$, $f_{u, S}$ 为: 从 $u$ 开始随机游走, 第一次到达 $S$ 的期望步数. 简写为 $f_u$.

然后就有式子 (设 $E$ 为边集)

$$
f_{u} = \frac{1}{deg_u} \sum_{(u, v) \in E} (f_{v} + 1)
$$

把 $f_{1 \sim n}$ 看做 $n$ 个变量, 对每个 $u$ 可以列出一个式子, 就可以直接高斯消元了, 总复杂度为 $O(2^n n^3)$.

但考虑到树的特殊结构, 我们可以将消元的复杂度做到更优.

考虑节点 $u$, 在对 $u$ 列出的式子中, 除了 $f_{fa_u}$ 以外, 其余的变量都在 $u$ 的子树内.

所以我们可以考虑先把 $u$ 的子树内 (不包括 $u$) 的变量都化为 $af_{u} + b$ 的形式, 然后再带入 $u$ 的式子内, 就可以把 $f_u$ 化为 $af_{fa_u} + b$ 的形式. 然后对 $fa_u$ 再进行同样的过程, 一直往上消元, 最终就可以得到 $f_X$ 的值了.

然后再 $O(n 2^n)$ 求一次高维前缀和, 每次 $O(1)$ 回答即可.

实际上这题的改成 $X$ 不固定 (即对于每个询问给一个不同的 $X$) 也可以做.

只需要在算出上面的 $f_X$ 后, 把它往子树中代入, 然后就可以对每个 $u$ 都求出 $f_u$ 了.

总复杂度为 $O(2^nn + Q)$.

## Code
```cpp
#include <cstdio>
#include <iostream>
#include <vector>

#define pb push_back
#define sz(x) (int)(x).size()
#define mkp make_pair
#define fi first
#define se second

using namespace std;

const int _ = 18 + 7;
const int __ = (1 << 18) + 7;
const int mod = 998244353;

int n, Q, X, g[__];
pair<int, int> f[_];
vector<int> to[_];

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

int popcount(int x) {
  int cnt = 0;
  for (int i = 0; i < n; ++i) cnt += x >> i & 1;
  return cnt;
}

void Dfs(int u, int fa, int s) {
  f[u] = mkp(0, 0);
  if (s >> (u - 1) & 1) return;
  for (int v: to[u]) {
    if (v == fa) continue;
    Dfs(v, u, s);
    f[u].fi = (f[u].fi + f[v].fi) % mod;
    f[u].se = (f[u].se + f[v].se) % mod;
  }
  if (u != X) {
    int Inv = Pw(sz(to[u]) - f[u].fi + mod, mod - 2);
    f[u].fi = Inv;
    f[u].se = 1ll * (f[u].se + sz(to[u])) * Inv % mod;
  }
  else {
    int Inv = Pw(sz(to[u]), mod - 2);
    f[u].fi = 1ll * f[u].fi * Inv % mod;
    f[u].se = 1ll * (f[u].se + sz(to[u])) * Inv % mod;
  }
}

int main() {
  cin >> n >> Q >> X;
  for (int i = 1, x, y; i < n; ++i) {
    cin >> x >> y;
    to[x].pb(y), to[y].pb(x);
  }

  for (int s = 1; s < 1 << n; ++s) {
    Dfs(X, 0, s);
    int tmp = 1ll * f[X].se * Pw(1 - f[X].fi + mod, mod - 2) % mod;
    g[s] = (popcount(s) & 1) ? tmp : (mod - tmp) % mod;
  }

  for (int i = 0; i < n; ++i)
    for (int s = 0; s < 1 << n; ++s)
      if (s >> i & 1) g[s] = (g[s] + g[s ^ (1 << i)]) % mod;

  for (int t = 1, K, s; t <= Q; ++t) {
    cin >> K, s = 0;
    for (int i = 1, x; i <= K; ++i) cin >> x, s |= 1 << (x - 1);
    cout << g[s] << endl;
  }

  return 0;
}
```


