# [解题报告][luogu4916]魔力环


* Statement
[[https://www.luogu.com.cn/problem/P4916][传送门]]

一个点数为 \(n\) 的环 (项链), 每个点可以被染成黑色或白色.

染色要求为: 恰好有 \(m\) 个点被染为黑色, 且不存在长度 \(>k\) 的黑色连续串.

考虑旋转同构, 求本质不同的的染色方案数.

\( m \le n \le 10^5, k \le 10^5\).

* Solution
这里旋转就相当于置换, 本质不同的的染色方案就相当于轨道数. 那么使用 Burnside 引理, 把轨道数转化为置换的不动点数量.

这里总共有 \(n\) 个置换, 每个置换都形如

\[
\begin{Bmatrix}
1 & 2     & 3     & \cdots & n - i + 1 & n - i & \cdots & n - 1 & n     \\
i & i + 1 & i + 2 & \cdots & n         & 1     & \cdots & i - 2 & i - 1 
\end{Bmatrix}
\]

那么第 \(i\) 个置换的每个循环的长度为 \( \mathrm{lgt} = \frac{lcm(i, n)}{i} = \frac{n}{\gcd(i, n)} \), 循环的数量为 \( \mathrm{num} = \gcd(i, n) \).

所以 \(n\) 就会被分成 \(\mathrm{lgt}\) 段由 \(\mathrm{num}\) 个属于不同循环的点组成的区间 (是的这里没打错).

那么对这个 \(n\) 元环的染色就相当于对这个 \(\mathrm{num}\) 元环的染色. (因为每个循环中的颜色都必须相同.)

那么我们设 \( g(i, j) \) 为给 \(i\) 元环染上 \(m\) 个黑点的合法方案数 (不考虑循环同构). 

那么我们要求的就是 (这里我们默认 \( \frac{n}{\gcd(i, n)} \mid m \).)

\[
\frac{1}{n} \sum_{i = 1}^n g(\gcd(i, n), \frac{m}{n / \gcd(i, n)})
\]

改为枚举 \(\gcd\) (默认 \( d \mid n \))

\[
\begin{aligned}
\frac{1}{n} \sum_{i = 1}^n g(\gcd(i, n), \frac{m}{n / \gcd(i, n)})
&= \frac{1}{n} \sum_{d = 1}^n g(d, \frac{m}{n / d}) \sum_{i = 1}^{\frac{n}{d}} [\gcd(\frac{n}{d}, i) = 1] \\
&= \frac{1}{n} \sum_{d = 1}^n g(d, \frac{m}{n / d}) \varphi(\frac{n}{d})
\end{aligned}
\]

那么现在考虑怎么求 \(g(d, \frac{m}{n / d})\).

环不好弄, 那么我们枚举最后一段黑色连续段和第一段黑色连续段的长度之和, 断环为链. 然后因为黑点和白点的数量是固定的, 所以我们可以把黑色连续段看做把若干个黑点放在白点之间. 然后每段黑色连续段长度不超过 \(k\) 的方案可以用容斥算出来.

形式化地说, 设 \( h(n, c) \) 为将 \(c\) 个黑点放进 \(n\) 个空位中的方案. 则有

\[
g(d, \frac{m}{n / d}) = \sum_{i = 0}^k (i + 1) h(d - \frac{m}{n / d} - 1, \frac{m}{n / d} - i)
\]

(乘上 \((i + 1)\) 是因为要算最后一段和第一段分别放了多少个黑点.)

\[
h(n, c) = \sum_{i = 0}^{\lfloor\frac{c}{k + 1}\rfloor} (-1)^i \binom{n}{i} \binom{c - i(k + 1) + n - 1}{n - 1}
\]

(最后一个组合数是用 插板法 + 可重组合 计算把 \(c - i(k + 1)\) 个黑点放进 \(n\) 个空位的方案.)

这样每次计算 \(g(d, \frac{m}{n / d})\) 的时间复杂度为 \( O(k \lfloor\frac{\frac{m}{n / d}}{k + 1}\rfloor) = O(d)\), 所以总时间复杂度为 \( O(\sigma(n)) \).

* Code
#+begin_src C++
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 2e5 + 7;
const int mod = 998244353;

int n, m, K, pri[_], phi[_], v[_], cnt, fac[_], ifac[_], inv[_];

void Init() {
  cin >> n >> m >> K;
  if (m == n) { puts("0"); exit(0); }
  phi[1] = 1;
  for (int i = 2; i <= n; ++i) {
    if (!v[i]) pri[++cnt] = i, v[i] = i, phi[i] = i - 1;
    for (int j = 1; j <= cnt and pri[j] <= v[i] and i * pri[j] <= n; ++j) {
      v[i * pri[j]] = pri[j];
      phi[i * pri[j]] = i % pri[j] ? phi[i] * phi[pri[j]] : phi[i] * pri[j];
    }
  }

  fac[0] = ifac[0] = inv[1] = 1;
  for (int i = 1; i <= 2 * n; ++i) {
    fac[i] = 1ll * fac[i - 1] * i % mod;
    if (i != 1) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    ifac[i] = 1ll * ifac[i - 1] * inv[i] % mod;
  }
}

int C(int n, int m) { return n < 0 or m < 0 ? 0 : 1ll * fac[n] * ifac[m] % mod * ifac[n - m] % mod; }

int F(int sum, int num) {
  if (num == 0) return sum == 0;
  int res = 0;
  for (int i = 0; i <= min(sum / (K + 1), num); ++i) {
    int tmp = 1ll * C(num, i) * C(sum - (K + 1) * i + num - 1, num - 1) % mod;
    res = (res + ((i & 1) ? mod - tmp : tmp)) % mod;
  }
  return res;
}

int main() {
  Init();

  int ans = 0;
  for (int d = 1; d <= n; ++d)
    if (!(n % d) and !(m % (n / d))) {
      int t = m / (n / d), res = 0;
      for (int i = 0; i <= min(t, K); ++i) res = (res + 1ll * (i + 1) * F(t - i, d - t - 1) % mod) % mod;
      ans = (ans + 1ll * res * phi[n / d] % mod) % mod;
    }

  cout << 1ll * ans * inv[n] % mod << endl;
  return 0;
}
#+end_src

* Reference

[[https://www.luogu.com.cn/blog/cjyyb/solution-p4916][题解 by yybyyb]]

