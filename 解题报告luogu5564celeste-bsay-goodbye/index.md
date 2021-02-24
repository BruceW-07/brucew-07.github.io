# [解题报告][luogu5564][Celeste B]Say Goodbye


* Statement

[[https://www.luogu.com.cn/problem/P5564][传送门]]

\( n \) 个点, $m$ 种颜色, 颜色为 $i$ 的点有 $a_i$ 个.

求这 $n$ 个点构成的本质不同无标号有序基环树个数. (有序: 子树有序)

两棵基环树本质相同当且仅当通过旋转基环可以使它们相同.

$n \le 2 \times 10^5$.



* Solution

旋转基环就是个环同构问题, 那么直接考虑 Burnside 引理.

那么根据一般套路, 可以列出式子

\[
\begin{aligned}
\mathrm{ans}
&= \sum_{k = 2}^n \frac{1}{k} \sum_{d |k} [\frac{k}{d} | \gcd_{i = 1}^m a_i \wedge \frac{k}{d} | n]g_{\frac{n}{k / d}}(d) \varphi(\frac{k}{d})	\\
\end{aligned}
\]

其中 \(g_{n}(m)\) 为: 环长为 \(m\) , 点数为 \(n\) 的无标号有序基环树个数. 

接下来考虑如何算 \(g_n(m)\).



首先考虑节点数为 \(n\) 的有根无标号有序树个数怎么算. (不考虑颜色.)

把树的欧拉序看做括号序列, 那么一棵树就唯一地对应一个最外层括号数为 \(1\) 的括号序列. (因为要考虑根节点.)

那么方案数也就是第 \(n - 1\) 个卡特兰数.

我们设 \(f_n\) 为所求方案数, \(F(x)\) 为 \(f_i\) 的 \(\mathrm{OGF}\). 假设 \(C(x)\) 为卡特兰数的 \(\mathrm{OGF}\), 则有 \(F = xC\).

再考虑对节点染色. 可以发现染色方案和树的结构是独立的, 所以直接乘上一个染色方案 \(\binom{n}{a_1, a_2, \cdots, a_m}\) 即可.

那么可以得到

\[
g_n(k) = [x^n]F^k \binom{n}{a_1, a_2, \cdots, a_m}
\]



为了描述更加简洁, 我们默认 \(\frac{k}{d} | n\), 并交换 \(\frac{k}{d}\) 和 \(d\).

\[
\begin{aligned}
\mathrm{ans}
&= \sum_{k = 2}^n \frac{1}{k} \sum_{d |k} [d | \gcd_{i = 1}^m a_i]g_{\frac{n}{d}}(\frac{k}{d}) \varphi(d)	\\
&= \sum_{k = 2}^n \frac{1}{k} \sum_{d |k} [d | \gcd_{i = 1}^m a_i][x^{n/d}]F^{k/d}\binom{n/d}{a_{1 \cdots m} /d} \varphi(d)	\\
&= -f_n \binom{n}{a_{1 \cdots m}} + \sum_{d | \gcd_{i = 1}^m a_i} \varphi(d) \binom{n/d}{a_{1 \cdots m} /d} [x^{n / d}]\sum_{t = 1}^{n / d}\frac{F^{t}}{td}	\\
&= -f_n \binom{n}{a_{1 \cdots m}} + \sum_{d | \gcd_{i = 1}^m a_i} \frac{\varphi(d)}{d} \binom{n/d}{a_{1 \cdots m} /d} [x^{n / d}]\sum_{t = 1}^{+\infty}\frac{F^{t}}{t}	\\
\end{aligned}
\]

第二行到第三行是交换枚举顺序, 并用 \(t\) 代替 \(\frac{k}{d}\).

接下来有两种推导方法.

** 一

我们有结论 \(\sum_{i = 1}^{+ \infty} \frac{x^i}{i} = \ln(1 - x)\), 所以原式化为

\[
\mathrm{ans} = -f_n \binom{n}{a_{1 \cdots m}} + \sum_{d | \gcd_{i = 1}^m a_i} \frac{\varphi(d)}{d} \binom{n/d}{a_{1 \cdots m} /d} [x^{n / d}]\sum_{t = 1}^{+\infty}\ln(1 - F)
\]

多项式 \(\ln\) 求一下即可. 时间复杂度为 \(O(n \log n + \sigma(n))\). (\(\sigma(n)\) 表示 \(n\) 的约数和, 大概是 \(O(n)\) 级别的.)

** 二

卡特兰数生成函数的幂有个性质

\[
[x^n] C^m = \binom{2n - m - 1}{n - m} - \binom{2n - m - 1}{n - m - 1}
\]

然后把这个带进 \(F^t\) 即可得到

\[
\mathrm{ans} = -f_n \binom{n}{a_{1 \cdots m}} + \sum_{d | \gcd_{i = 1}^m a_i} \frac{\varphi(d)}{d} \binom{n/d}{a_{1 \cdots m} /d} \sum_{t = 1}^{n / d} \frac{1}{t} \binom{2n / d - k - 1}{n / d - k} \binom{2n / d - k - 1}{n / d - k - 1}
\]

时间复杂度为 \(O(\sigma(n))\).



* Code
#+BEGIN_SRC C++
#include <cassert>
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 4e5 + 7;
const int mod = 998244353;

int n, m, a[_], gcda;
int pri[_], v[_], phi[_], cnt;
int fac[_], ifac[_], inv[_];

int Gcd(int a, int b) { return b ? Gcd(b, a % b) : a; }

void Init() {
  cin >> n >> m; gcda = n;
  for (int i = 1; i <= m; ++i) {
    scanf("%d", &a[i]);
    gcda = Gcd(gcda, a[i]);
  }

  phi[1] = 1;
  for (int i = 2; i <= n; ++i) {
    if (!v[i]) pri[++cnt] = i, v[i] = i, phi[i] = i - 1;
    for (int j = 1; j <= cnt and pri[j] <= v[i] and i * pri[j] <= n; ++j) {
      v[i * pri[j]] = pri[j];
      phi[i * pri[j]] = (i % pri[j]) ? phi[i] * phi[pri[j]] : phi[i] * pri[j];
    }
  }

  fac[0] = ifac[0] = inv[1] = 1;
  for (int i = 1; i <= 2 * n; ++i) {
    if (i != 1) inv[i] = 1ll * inv[mod % i] * (mod - mod / i ) % mod;
    fac[i] = 1ll * fac[i - 1] * i % mod;
    ifac[i] = 1ll * ifac[i - 1] * inv[i] % mod;
  }
}

int Ch(int d) {
  int res = fac[n / d]; assert(!(n % d));
  for (int i = 1; i <= m; ++i) res = 1ll * res * ifac[a[i] / d] % mod, assert(!(a[i] % d));
  return res;
}

int C(int n, int m) { return 1ll * fac[n] * ifac[m] % mod * ifac[n - m] % mod; }

int main() {
  Init();

  int ans = (mod - 1ll * (C(2 * (n - 1), n - 1) - C(2 * (n - 1), n - 2) + mod) * Ch(1) % mod) % mod;
  for (int d = 1; d <= gcda; ++d)
    if (!(gcda % d)) {
      int res = 0;
      for (int k = 1; k <= n / d; ++k)
        res = (res + 1ll * (C(2 * n / d - k - 1, n / d - k) - C(2 * n / d - k - 1, n / d - k - 1) + mod) * inv[k] % mod) % mod;
      ans = (ans + 1ll * Ch(d) * phi[d] % mod * inv[d] % mod * res % mod) % mod;
    }

  cout << ans << endl;
  return 0;
}

#+END_SRC


* Reference

[[https://www.cnblogs.com/PinkRabbit/p/11525881.html][题解  by PinkRabbit]]

