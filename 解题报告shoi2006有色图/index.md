# [解题报告][SHOI2006]有色图


* Statement
[[https://www.luogu.com.cn/problem/P4128][传送门]]

一张 \(n\) 个点的无向完全图, 每条边可以染上 \(m\) 种颜色之一, 求 \(n!\) 种点置换下本质不同的染色方案.

\(n \le 53 \).

* Solution
看到置换 + 染色, 先考虑一波 Burnside 引理. (实际上这里用的是 Pólya 计数定理.)

需要注意的是这里是对 *边* 染色, 对 *点* 置换, 所以我们要先把 *点置换* 转成 *边置换* 后才能用 Burnside 引理做.

我们把一个点置换用循环的形式表示, 即 \( q_1q_2q_3 \cdots q_k \), 其中 \( q_i \) 表示该置换的第 \( i \) 个循环的大小. 

我们这里把 \(q_i\) 排序后的结果称为该点置换的 *循环表示*, 那么可以推导出符合一个循环表示的点置换的数量为

\[
\mathrm{tms} = n! \prod_{i = 1}^k \frac{1}{q_i} \prod_{i = 1}^n \frac{1}{\#_i}
\]

其中 \(n!\) 是为了 \(n\) 个点分配到每个循环; 

\(\prod_{i = 1}^k \frac{1}{q_i}\) 是因为循环是个圆排列, 所以要除以 \(q_i\); 

\(\#_i\) 表示该置换中大小为 \(i\) 的循环的数量, 因为相同的循环之间应该是无序的, 否则会算重.

接下来考虑怎么算边置换.

首先有一个显然的结论, 「循环表示相同的点置换所对应的边置换相同」. 那么我们只需要考虑对于一个循环表示, 它对应的点置换所对应的边置换有多少个循环. (有点绕.)

首先考虑端点在同一个点循环内的边. 容易发现, 在同一个点循环内, 端点距离相同的边构成了一个边循环, 而一个大小为 \(q_i\) 的点循环内有 \( \lfloor \frac{q_i}{2} \rfloor \) 种端点距离. 所以一个大小为 \(q_i\) 点循环可以贡献 \( \lfloor \frac{q_i}{2} \rfloor \) 个边循环.

再考虑端点在两个不同点循环内的边. 两个大小分别为 \(q_1, q_2\) 的点循环之间有 \(q_1q_2\) 条边, 而一个边循环的长度为 \( lcm(q_1, q_2) = \frac{q_1q_2}{\gcd(q_1, q_2)} \) (画张图可以便于理解), 所以这两个点循环可以贡献 \( \gcd(q_1, q_2) \) 个边循环.

所以一个循环表示为 \(q_1q_2 \cdots q_k\) 的点循环所对应的边置换的循环数量为

\[
\mathrm{num} = \sum_{i = 1}^k \lfloor \frac{q_i}{2} \rfloor + \sum_{i = 1}^k \sum_{j = i + 1}^k \gcd(q_i, q_j)
\]

所以一个循环表示所对应的不动点数量就是

\[
\mathrm{tms} \cdot m^{\mathrm{num}}
\]

所以只要枚举序列 \( \{ q \} \) 然后统计它的 \(\mathrm{tms}\) 和 \(\mathrm{num}\) 即可.

设 \(Q(n)\) 为点数为 \(n\) 的循环表示数量, 即为 \(n\) 的划分数. 暴搜一下, 发现 \(Q(53) = 329931\).

所以时间复杂度为 \( O(Q(n)n^2) \), 然而跑不满 (因为枚举两个点循环那地方不一定是 \(n^2\)). +所以是 O(能过).+
* Code
#+BEGIN_SRC C++
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 53 + 7;
const int __ = 3e3 + 7;

int n, m, mod, fac[_], ifac[_], inv[_], pw[__], ans, p[_], gcd[_][_], tot;

int Gcd(int a, int b) { return b ? Gcd(b, a % b) : a; }

void Dfs(int k, int las, int cnt) {
  if (k == n) {
    int tms = fac[n], num = 0;
    for (int i = 1; i < cnt; ++i) {
      tms = 1ll * tms * inv[p[i]] % mod;
      num += p[i] / 2;
      for (int j = i + 1; j < cnt; ++j) num += gcd[p[i]][p[j]];
    }
    for (int i = 1, j = 1; i < cnt; i = j) {
      while (j < cnt and p[j] == p[i]) ++j;
      tms = 1ll * tms * ifac[j - i] % mod;
    }
    ans = (ans + 1ll * tms * pw[num] % mod) % mod;
    tot = (tot + tms) % mod;
    return;
  }
    
  for (int i = 1; i <= min(las, n - k); ++i) p[cnt] = i, Dfs(k + i, i, cnt + 1);
}

int main() {
  cin >> n >> m >> mod;
  pw[0] = fac[0] = ifac[0] = inv[1] = 1;
  for (int i = 1; i <= n; ++i) {
    if (i != 1) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    fac[i] = 1ll * fac[i - 1] * i % mod;
    ifac[i] = 1ll * ifac[i - 1] * inv[i] % mod;
  }
  for (int i = 1; i <= n * n; ++i) pw[i] = 1ll * pw[i - 1] * m % mod;
  for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= n; ++j) gcd[i][j] = Gcd(i, j);

  Dfs(0, n, 1);

  cout << 1ll * ans * ifac[n] % mod << endl;
  cerr << "tot: " << tot << ' ' << fac[n] << endl;

  return 0;
}
#+END_SRC

