# [解题报告][AT5742]Cookie Distribution


* Statement
[[https://www.luogu.com.cn/problem/AT5742][传送门]]

有一个长度为 \(n\) 初始元素全为 \(0\) 的序列 \(P\).

进行 \(K\) 操作, 每次选择 \(a_i\) 个不同的元素将其 \(+1\), 两次不同操作所选择的元素可以相同.

 求 \(K\) 次操作后 \(P\) 中元素乘积的期望乘以总方案数.

\(n \le 1000, K \le 20\).
 
* Solution

乘积算期望比较难搞, 所以首先把它进行转化.

使用 \(a = \binom{a}{1}\) 这个套路, 得到 
\[
\prod_{i = 1}^n p_i = \prod_{i = 1}^n \binom{p_i}{1}
\]
 而 \( \prod_{i = 1}^n \binom{p_i}{1} \) 的组合意义可以解释为: 对于每个 \(i\) 在选择到了 \(i\) 的若干次操作中选择一个操作的方案数.

然后我们把对象转换一下, 假设先枚举每个 \(i\) 所选择的操作 \(q_i\), 并设 \(x_i = \sum_{j = 1}^n [q_j = i]\), 那么满足 \(\{q\}\) 的操作方案数量为 
\[
\prod_{i = 1}^K \binom{n - x_i}{a_i - x_i}
\]

那么假设我们枚举序列 \(\{x\}\), 则答案为

\[
\sum_{\{x\}} \binom{n}{x_1, x_2, \cdots, x_K} \sum_{i = 1}^K \binom{n - x_i}{a_i - x_i}
\]

这个可以使用 DP 计算. 

具体来说, 就是设 \(f_{i,j}\) 表示考虑到第 \(i\) 次操作, 共钦定了 \(j\) 个节点的 \(q\) 的贡献总和. 转移方程为

\[
f_{i, j} = \sum_{k = 0}^{\min(j, a_i)} f_{i - 1, j - k} \binom{n - k}{a_i - k} \frac{1}{k!}
\]

最后答案即为 \(n!\ \times f_{K, n}\). 复杂度为 \(O(n^2K)\).

* Code
#+begin_src C++
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 20 + 7;
const int __ = 1e3 + 7;
const int mod = 1e9 + 7;

int n, K, a[_], fac[__], ifac[__], inv[__], f[_][__];

int C(int n, int m) { return 1ll * fac[n] * ifac[m] % mod * ifac[n - m] % mod; }

int main() {
  cin >> n >> K; 
  for (int i = 1; i <= K; ++i) cin >> a[i];
  fac[0] = ifac[0] = inv[1] = 1;
  for (int i = 1; i <= n; ++i) {
    fac[i] = 1ll * fac[i - 1] * i % mod;
    if (i != 1) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    ifac[i] = 1ll * ifac[i - 1] * inv[i] % mod;
  }

  f[0][0] = 1;
  for (int i = 1; i <= K; ++i)
    for (int j = 0; j <= n; ++j)
      for (int k = 0; k <= min(a[i], j); ++k)
        f[i][j] = (f[i][j] + 1ll * f[i - 1][j - k] % mod * ifac[k] % mod * C(n - k, a[i] - k) % mod) % mod;
                   
  cout << 1ll * f[K][n] * fac[n] % mod << endl;
  return 0;
}
#+end_src

