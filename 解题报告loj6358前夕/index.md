# [解题报告][loj6358]前夕


* Statement
[[https://loj.ac/p/6358][传送门]]

给定集合 \( \{0, 1, 2, \cdots, 2^n - 1\} \), 求满足按位与后二进制中 \(1\) 的个数为 \(4\) 的倍数的子集 \(T\) 的个数.

\(n \le 10^7\).

70pts subtask: \(n \le 10^5\).


* Solution

设 \(f_i\) 表示 \(\{0, 1, \cdots, 2^i - 1\}\) 中满足按位与后 \(1\) 的个数为 \(0\) 的 *非空* 子集数量, 则有

\[
\mathrm{ans} = 1 + \sum_{i = 0}^{\lfloor \frac{n}{4} \rfloor} \binom{n}{4i} f_{n - 4i}
\]

(其中 \(1\) 表示的是空集.)

\(f_i\) 的计算可以使用容斥. 我们每次钦定若干个位置, 使得按位与后这些位置为 \(1\), 则可以列出式子

\[
f_i = \sum_{j = 0}^i (-1)^j \binom{i}{j} (2^{2^{i - j}} - 1)
\]

把组合数展开后就可以写成卷积的形式, 使用 NTT 就可以做到 \(O(n \log n)\) 的复杂度, 可以通过 \(n \le 10^5\) 的 subtask.

推一下式子, 发现 \(f\) 的计算似乎没有什么更好的方法, 那么我们考虑把 \(f\) 扔回答案计算式中, 再对整个式子考虑优化.

而上面描述的计算式中, 我们发现 \(4i\) 这个东西不太优美, 因为它使得 \(f\) 不连续了, 可能会导致推式子的过程中出现一些问题. 所以我们把它改为 \([4 | i]\), 并考虑使用单位根反演.

原式为

\[
\begin{aligned}
\mathrm{ans} - 1 
&= \sum_{i = 0}^n [4 | i] \binom{n}{i} f_{n - i}  \\
&= \sum_{i = 0}^n [4 | n - i] \binom{n}{i} f_{i}  \\
&= \sum_{i = 0}^n [4 | n - i] \binom{n}{i} \sum_{j = 0}^i (-1)^j \binom{i}{j} (2^{2^{i - j}} - 1)  \\
\end{aligned}
\]

\((2^{2^{i - j}} - 1)\) 这个东西长得过于丑陋, 所以我们考虑把它扔到前面来枚举.

\[
\begin{aligned}
\mathrm{ans} - 1 
&= \sum_{i = 0}^n [4 | n - i] \binom{n}{i} \sum_{j = 0}^i (-1)^{i - j} \binom{i}{j} (2^{2^j} - 1)  \\
&= \sum_{j = 0}^n (2^{2^j} - 1) \sum_{i = j}^n [4 | n - i] (-1)^{i - j} \binom{n}{i} \binom{i}{j}  \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \sum_{i = j}^n [4 | n - i] (-1)^{i - j} \binom{n - j}{i - j}  \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \sum_{i = 0}^{n - j} [4 | n - i - j] (-1)^i \binom{n - j}{i}  \\
\end{aligned}
\]

然后把 \([4 | i + j]\) 用单位根反演换掉,

\[
\begin{aligned}
\mathrm{ans} - 1 
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \sum_{i = 0}^{n - j} (-1)^i \binom{n - j}{i} \frac{1}{4} \sum_{k = 0}^3 \omega_{4}^{k(n - i - j)}   \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \frac{1}{4} \sum_{k = 0}^3 \omega_{4}^{k(n - j)} \sum_{i = 0}^{n - j} (-1)^i \binom{n - j}{i} \omega_{4}^{-ki}   \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \frac{1}{4} \sum_{k = 0}^3 \omega_{4}^{k(n - j)} \sum_{i = 0}^{n - j} \binom{n - j}{i} (- \omega_{4}^{-k})^i   \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \frac{1}{4} \sum_{k = 0}^3 \omega_{4}^{k(n - j)} (1 - \omega_4^{-k})^{n - j} \\
&= \sum_{j = 0}^n \binom{n}{j} (2^{2^j} - 1) \frac{1}{4} \sum_{k = 0}^3 (\omega_{4}^{k} (1 - \omega_4^{-k}))^{n - j} \\
\end{aligned}
\]

后面那坨东西可以预处理, 前面的组合数和 \(2\) 的若干次方可以递推, 所以总复杂度为 \(O(n)\), 可以通过全部数据.

* Code
#+BEGIN_SRC C++
#include <cassert>
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 1e7 + 7;
const int mod = 998244353;

int n, inv[_], pww[4][_], w[4];

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

void pls(int &x, long long y) { x = (x + y) % mod; }

int main() {
  cin >> n;
  inv[1] = 1;
  for (int i = 2; i <= n; ++i) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
  for (int k = 0; k < 4; ++k) w[k] = Pw(Pw(3, (mod - 1) / 4), k);
  for (int k = 0; k < 4; ++k) {
    pww[k][0] = 1, pww[k][1] = 1ll * w[k] * (1 - w[(4 - k) % 4] + mod) % mod;
    for (int i = 2; i <= n; ++i) pww[k][i] = 1ll * pww[k][i - 1] * pww[k][1] % mod;
  }

  int C = 1, pw = 2, ans = 0;
  for (int j = 0; j <= n; ++j) {
    int res = 0;
    for (int k = 0; k < 4; ++k) pls(res, pww[k][n - j]);
    pls(ans, 1ll * C * (pw - 1 + mod) % mod * res % mod);
    C = 1ll * C * (n - j) % mod * inv[j + 1] % mod;
    pw = 1ll * pw * pw % mod;
  }

  cout << (1ll * ans * Pw(4, mod - 2) % mod + 1) % mod << endl;
  return 0;
}
#+END_SRC

* Postscript
这道题我从 14:30 开始推, 然后在托腮的 "题解" 帮助下在 15:30 推完了, 上了个洗手间后把代码写了, 然后没过样例, 然后发现式子推错了, 然后重新推式子, 然后推完了, 然后代码写完了, 然后又没过样例, 然后写了个暴力, 然后发现是组合数递推出错了, 然后就 A 了, 然后已经 17:00 了.

+再论wz做题量这么少的原因.+

