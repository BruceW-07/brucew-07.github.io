# [解题报告]烷基计数&烯烃计数&烷烃计数


* Foreword
昨天晚上讲题的时候看到 \(\mathrm{M\color{red}{\_sea}}\) 巨巨在写烷基计数, 正好我之前在搞 Burnside 引理, 于是也找来做了.

由于不清楚难度顺序, 就先开了「烷烃计数」, 想试着往之前[[https://brucew-07.github.io/posts/%E8%A7%A3%E9%A2%98%E6%8A%A5%E5%91%8Ashoi2006%E6%9C%89%E8%89%B2%E5%9B%BE/][有色图]]那个做法上套, 结果想了一个多小时一点思路都没有. (不过话说从这个数据范围应该就看得出做法明显不一样吧...)

后来 \(\mathrm{M\color{red}{\_sea}}\) 给我指了条明路, 让我先去做「烷基计数」. 一语点醒梦中人, 于是就有了这篇解题报告.

+(三倍经验警告)+

* 烷基计数

** Statement
[[https://loj.ac/p/6538][[loj6538]烷基计数 加强版 加强版]]

求碳原子个数为 \(n\) 的烷基的同分异构体的数量.

\(n \le 10^5\).

** Solution
「烷基计数」比「烷烃计数」简单的地方在于, 「烷基计数」是有根的, 那么我们就可以考虑从根开始往下递归.

首先, 考虑怎么处理根的儿子的同构情况. 这里我们直接使用 Burnside 引理, 便把无标号的子节点变成有标号了, 然后再对每个置换算出它所对应的染色方案数量即可 (即不动点方案). 因为这里根只有 \(3\) 个子节点 (子节点为空的话就看做该子树的大小为 \(0\)), 所以总共只有 \(3! = 6\) 中置换, 手动把每种置换的不动点数量算出来即可.

具体来说, 设 \(f_i\) 表示大小为 \(i\) 的烷基的同分异构体的数量, 则有

\[
f_i = \frac{1}{6} (2[3 \mid (i - 1)]f_{\frac{i - 1}{3}} + 3\sum_{j = 0}^{\lfloor \frac{i - 1}{2} \rfloor} f_jf_{i - 1 - 2j} + \sum_{j = 0}^{i - 1} \sum_{k = 0}^{i - 1 - j} f_jf_kf_{i - 1 - j - k})
\]

这样的话就可以得到一个 \(O(n^3)\) 的暴力.

下面考虑使用生成函数优化.

(由于本人才疏学浅并且对生成函数的应用极其不熟练, 所以以下内容借鉴于[[https://cn.bing.com/search?form=MOZLBR&pc=MOZI&q=%E7%83%B7%E5%9F%BA%E8%AE%A1%E6%95%B0][这篇博客]]).
+(怎么想都想不到可以把指数放在 x 上)+

设 \(F(x) = \sum_{i \ge 0} f_ix^i\). 那么上面的式子可以写成

\[
F(x) = 1 + \frac{1}{6}x[2F(x^3) + 3F(x^2)F(x) + F(x)^3]
\]

(这里不要像我一样看到 \(F(x^3)\) 和 \(F(x^2)\) 这两个东西懵逼了, 实际上把它们看做两个另外的函数就可以了.)

这个式子可以分治FFT, 也可以牛顿迭代. 分治 FFT 的做法比较显然, 下面大致描述一下牛顿迭代的做法.

首先设 \(G(F(x)) = 1 + \frac{1}{6}x[2F(x^3) + 3F(x^2)F(x) + F(x)^3] - F(x)\). 

然后按照牛顿迭代的套路, 假设已经求出了 \(F_0(x)\) 满足 \(G(F_0(X)) \equiv 0 \pmod{x^{\frac{n}{2}}}\), 现在要求 \(F(x)\) 满足 \(G(F(x)) \equiv 0 \pmod{x^n}\)

\[
F(x) = F_0(x) - \frac{G(F_0(x))}{G'(F_0(x))}
\]

手动把 \(G'(F_0(x))\) 化一下即可. 实现的时候可以分子分母同乘一个常数, 就可以不用乘那么多逆元了.

时间复杂度为 \(O(n\log n)\).

实现的时候注意清空会导致循环卷积的位置.

** Code
#+begin_src C++
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = (1 << 20) + 7;
const int mod = 998244353;

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

namespace Poly {
  int tot, num[_], pwrt[_], inv[_], a[_], b[_];
  unsigned long long q[_];

  void Init(int n) {
    tot = 1; while (tot < n + n) tot <<= 1;
    inv[1] = 1; for (int i = 2; i <= tot; ++i) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    pwrt[0] = 1, pwrt[1] = Pw(3, (mod - 1) / tot);
    for (int i = 2; i <= tot; ++i) pwrt[i] = 1ll * pwrt[i - 1] * pwrt[1] % mod;
  }

  void NTT(int *f, bool ty, int t = tot) {
    for (int i = 0; i < t; ++i) {
      num[i] = (num[i >> 1] >> 1) | ((i & 1) ? t >> 1 : 0);
      q[i] = f[num[i]];
    }
    for (int len = 2; len <= t; len <<= 1) {
      int gap = len >> 1, d = tot / len;
      for (int i = 0; i < t; i += len)
        for (int j = 0; j < gap; ++j) {
          int tmp = q[i + j + gap] * pwrt[ty ? tot - j * d : j * d] % mod;
          q[i + j + gap] = q[i + j] - tmp + mod;
          q[i + j] = q[i + j] + tmp;
        }
    }
    for (int i = 0; i < t; ++i) f[i] = q[i] * (ty ? inv[t] : 1) % mod;
  }

  void Mul(int *h, int *f, int *g, int t) {
    memcpy(a, f, t << 2), memcpy(b, g, t << 2), memset(h, 0, t << 2);
    NTT(a, 0, t), NTT(b, 0, t);
    for (int i = 0; i < t; ++i) h[i] = 1ll * a[i] * b[i] % mod;
    NTT(h, 1, t);
  }

  void Inv(int *h, int *f, int Len) {
    memset(a, 0, Len << 3), memset(b, 0, Len << 3);
    memcpy(b, f, sizeof b); memset(h, 0, Len << 3);
    h[0] = Pw(b[0], mod - 2), a[0] = b[0], a[1] = b[1];
    for (int len = 2, t = 4; len <= Len; len <<= 1, t <<= 1) {
      NTT(a, 0, t), NTT(h, 0, t);
      for (int i = 0; i < t; ++i) h[i] = 1ll * h[i] * (2 - 1ll * h[i] * a[i] % mod + mod) % mod;
      NTT(h, 1, t);
      memcpy(a, b, t << 2), memset(h + len, 0, len << 2);
    }
  }
}

int n, F[_], H[_], A[_], B[_], T0[_], T1[_], T2[_];

int main() {
  cin >> n; Poly::Init(2 * (n + 1));
  H[0] = A[0] = B[0] = 1;
  for (int len = 2, t = 4; len <= 2 * (n + 1); len <<= 1, t <<= 1) {
    memset(T0 + len, 0, len << 2);
    for (int i = 0; i < len; ++i) T0[i] = 2ll * A[i] % mod;
    Poly::Mul(T1, B, H, t);
    for (int i = 0; i < len; ++i) T0[i] = (T0[i] + 3ll * T1[i] % mod) % mod;
    Poly::Mul(T2, H, H, t), Poly::Mul(T1, T2, H, t);
    for (int i = len - 1; i; --i) T0[i] = (T0[i - 1] + T1[i - 1]) % mod;
    T0[0] = 6;
    for (int i = 0; i < len / 2; ++i) T0[i] = (T0[i] - 6ll * H[i] % mod + mod) % mod;
    for (int i = len - 1; i; --i) T1[i] = 3ll * (B[i - 1] + T2[i - 1]) % mod;
    T1[0] = mod - 6;
    Poly::Inv(T1, T1, len);
    memset(T1 + len, 0, len << 2);
    Poly::Mul(T0, T0, T1, t);
    for (int i = 0; i < len; ++i) H[i] = (H[i] - T0[i] + mod) % mod, A[i] = B[i] = 0;
    for (int i = 0; i < len * 2 / 3; ++i) A[i * 3] = H[i];
    for (int i = 0; i < len; ++i) B[i * 2] = H[i];
  }
  cout << H[n] << endl;
  return 0;
}
#+end_src

* 烯烃计数
** statement
[[https://www.luogu.com.cn/problem/P6597][[luoguP6597]烯烃计数]]

求出碳原子数量为 \(2 \sim n\) 的 *单烯烃* 的同分异构体数量.

\( n \le 10^5 \).

** Solution

容易想到把碳碳双键看做根, 而碳碳双键两边的碳原子所连出去的东西其实就是若干个烷基, 所以先按照上面的方法算出烷基个数.

然后碳碳双键两边的碳原子只有两个儿子, 那么也与上面类似, 对大小为 \(2\) 的置换讨论一下, 列出式子, 然后生成函数乘一下即可. (需要注意这里 \(x^0\) 的系数要为 \(0\), 因为碳碳双键两边必须要接碳原子.)

然后对于这个碳碳双键, 可以把它看做一个点, 然后也是用 Burnside 引理按照上面的做法弄一下就好了.

时间复杂度为 \(O(n \log n)\).

+(其实和烷基计数没什么区别, 不过可以用来水水题量.)+

** Code
#+begin_src C++
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = (1 << 20) + 7;
const int mod = 998244353;

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

namespace Poly {
  int tot, num[_], pwrt[_], inv[_], a[_], b[_];
  unsigned long long q[_];

  void Init(int n) {
    tot = 1; while (tot < n + n) tot <<= 1;
    inv[1] = 1; for (int i = 2; i <= tot; ++i) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    pwrt[0] = 1, pwrt[1] = Pw(3, (mod - 1) / tot);
    for (int i = 2; i <= tot; ++i) pwrt[i] = 1ll * pwrt[i - 1] * pwrt[1] % mod;
  }

  void NTT(int *f, bool ty, int t = tot) {
    for (int i = 0; i < t; ++i) {
      num[i] = (num[i >> 1] >> 1) | ((i & 1) ? t >> 1 : 0);
      q[i] = f[num[i]];
    }
    for (int len = 2; len <= t; len <<= 1) {
      int gap = len >> 1, d = tot / len;
      for (int i = 0; i < t; i += len)
        for (int j = 0; j < gap; ++j) {
          int tmp = q[i + j + gap] * pwrt[ty ? tot - j * d : j * d] % mod;
          q[i + j + gap] = q[i + j] - tmp + mod;
          q[i + j] = q[i + j] + tmp;
        }
    }
    for (int i = 0; i < t; ++i) f[i] = q[i] * (ty ? inv[t] : 1) % mod;
  }

  void Mul(int *h, int *f, int *g, int t) {
    memcpy(a, f, t << 2), memcpy(b, g, t << 2), memset(h, 0, t << 2);
    NTT(a, 0, t), NTT(b, 0, t);
    for (int i = 0; i < t; ++i) h[i] = 1ll * a[i] * b[i] % mod;
    NTT(h, 1, t);
  }

  void Inv(int *h, int *f, int Len) {
    memset(a, 0, Len << 3), memset(b, 0, Len << 3);
    memcpy(b, f, sizeof b); memset(h, 0, Len << 3);
    h[0] = Pw(b[0], mod - 2), a[0] = b[0], a[1] = b[1];
    for (int len = 2, t = 4; len <= Len; len <<= 1, t <<= 1) {
      NTT(a, 0, t), NTT(h, 0, t);
      for (int i = 0; i < t; ++i) h[i] = 1ll * h[i] * (2 - 1ll * h[i] * a[i] % mod + mod) % mod;
      NTT(h, 1, t);
      memcpy(a, b, t << 2), memset(h + len, 0, len << 2);
    }
  }
}

int n, F[_], G[_], H[_], A[_], B[_], T0[_], T1[_], T2[_], inv2 = 499122177;

int main() {
  cin >> n; Poly::Init(2 * (n + 1));
  H[0] = A[0] = B[0] = 1;
  int len, t;
  for (len = 2, t = 4; len <= 2 * (n + 1); len <<= 1, t <<= 1) {
    memset(T0 + len, 0, len << 2);
    for (int i = 0; i < len; ++i) T0[i] = 2ll * A[i] % mod;
    Poly::Mul(T1, B, H, t);
    for (int i = 0; i < len; ++i) T0[i] = (T0[i] + 3ll * T1[i] % mod) % mod;
    Poly::Mul(T2, H, H, t), Poly::Mul(T1, T2, H, t);
    for (int i = len - 1; i; --i) T0[i] = (T0[i - 1] + T1[i - 1]) % mod;
    T0[0] = 6;
    for (int i = 0; i < len / 2; ++i) T0[i] = (T0[i] - 6ll * H[i] % mod + mod) % mod;
    for (int i = len - 1; i; --i) T1[i] = 3ll * (B[i - 1] + T2[i - 1]) % mod;
    T1[0] = mod - 6;
    Poly::Inv(T1, T1, len);
    memset(T1 + len, 0, len << 2);
    Poly::Mul(T0, T0, T1, t);
    for (int i = 0; i < len; ++i) H[i] = (H[i] - T0[i] + mod) % mod, A[i] = B[i] = 0;
    for (int i = 0; i < len * 2 / 3; ++i) A[i * 3] = H[i];
    for (int i = 0; i < len; ++i) B[i * 2] = H[i];
  }

  len >>= 1, t >>= 1;
  Poly::Mul(G, H, H, t);
  memset(G + len, 0, len << 2);
  for (int i = len - 1; i; --i) G[i] = 1ll * inv2 * (B[i - 1] + G[i - 1]) % mod;
  G[0] = 0;

  memset(B, 0, len << 2);
  for (int i = 0; i < len; ++i) B[i * 2] = G[i];
  Poly::Mul(F, G, G, t);
  for (int i = 0; i < len; ++i) F[i] = 1ll * inv2 * (B[i] + F[i]) % mod;

  for (int i = 2; i <= n; ++i) printf("%d\n", F[i]);
  return 0;
}
#+end_src

* 烷烃计数
** Statement
[[https://www.luogu.com.cn/problem/P6598][luoguP6598]烷烃计数]]

求出碳原子数量为 \(n\) 的烷烃的同分异构体数量.

\(n \le 10^5\).

** Solution.
在经历了「烷基计数」和「烯烃计数」的洗礼后, 我突然领悟了 \(\mathrm{M\color{red}{\_sea}}\) +看似漫不经心实际别有深意地+ 说出的一个词: 「重心」.

现在, 烷烃计数难搞的地方只在于它没有根, 那么我们要是能对一个烷烃找到一个唯一的根, 那么问题就迎刃而解了.

而「重心」就是一个经典的选择.

对于 \(n\) 为奇数的情况, 由于重心只会有一个, 所以直接把重心作为根即可.

具体来说, 就是先求出烷基个数, 在最后合并的时候, 根节点有 \(4\) 个儿子, 并且每一个子树的大小都小于 \(\lceil \frac{n}{2} \rceil \). 那么我们对 \(4! = 24\) 个置换讨论一下即可. 当然可能会有点麻烦.

对于 \(n\) 为偶数的情况, 若重心有一个子树的大小为 \( \frac{n}{2} \), 则这时会有两个重心, 这个烷烃就会被统计两次. (可以画个图理解.)

所以我们限制每个子树大小都小于 \( \frac{n}{2} \), 然后按上面 \(n\) 为奇数的情况统计.

而对于有两个重心的情况, 我们可以把两个重心之间那条边看做一个点, 然后对 \(2! = 2\) 个置换讨论一下即可. (跟上面烯烃计数的情况差不多.)

复杂度为 \(O(n \log n)\).

** Code
#+begin_src C++
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = (1 << 20) + 7;
const int mod = 998244353;

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

namespace Poly {
  int tot, num[_], pwrt[_], inv[_], a[_], b[_];
  unsigned long long q[_];

  void Init(int n) {
    tot = 1; while (tot < n + n) tot <<= 1;
    inv[1] = 1; for (int i = 2; i <= tot; ++i) inv[i] = 1ll * inv[mod % i] * (mod - mod / i) % mod;
    pwrt[0] = 1, pwrt[1] = Pw(3, (mod - 1) / tot);
    for (int i = 2; i <= tot; ++i) pwrt[i] = 1ll * pwrt[i - 1] * pwrt[1] % mod;
  }

  void NTT(int *f, bool ty, int t = tot) {
    for (int i = 0; i < t; ++i) {
      num[i] = (num[i >> 1] >> 1) | ((i & 1) ? t >> 1 : 0);
      q[i] = f[num[i]];
    }
    for (int len = 2; len <= t; len <<= 1) {
      int gap = len >> 1, d = tot / len;
      for (int i = 0; i < t; i += len)
        for (int j = 0; j < gap; ++j) {
          int tmp = q[i + j + gap] * pwrt[ty ? tot - j * d : j * d] % mod;
          q[i + j + gap] = q[i + j] - tmp + mod;
          q[i + j] = q[i + j] + tmp;
        }
    }
    for (int i = 0; i < t; ++i) f[i] = q[i] * (ty ? inv[t] : 1) % mod;
  }

  void Mul(int *h, int *f, int *g, int t = tot) {
    memcpy(a, f, t << 2), memcpy(b, g, t << 2), memset(h, 0, t << 2);
    NTT(a, 0, t), NTT(b, 0, t);
    for (int i = 0; i < t; ++i) h[i] = 1ll * a[i] * b[i] % mod;
    NTT(h, 1, t);
  }

  void Inv(int *h, int *f, int Len) {
    memset(a, 0, Len << 3), memset(b, 0, Len << 3);
    memcpy(b, f, sizeof b); memset(h, 0, Len << 3);
    h[0] = Pw(b[0], mod - 2), a[0] = b[0], a[1] = b[1];
    for (int len = 2, t = 4; len <= Len; len <<= 1, t <<= 1) {
      NTT(a, 0, t), NTT(h, 0, t);
      for (int i = 0; i < t; ++i) h[i] = 1ll * h[i] * (2 - 1ll * h[i] * a[i] % mod + mod) % mod;
      NTT(h, 1, t);
      memcpy(a, b, t << 2), memset(h + len, 0, len << 2);
    }
  }
}

int n, m, F[_], G[_], H[_], A[_], B[_], C[_], T0[_], T1[_], T2[_], inv24 = 291154603;

int main() {
  cin >> n; m = (n & 1) ? n / 2 + 1 : n / 2; Poly::Init(2 * (n + 1));
  H[0] = A[0] = B[0] = 1;
  int len, t;
  for (len = 2, t = 4; len <= 2 * (m + 1); len <<= 1, t <<= 1) {
    memset(T0 + len, 0, len << 2);
    for (int i = 0; i < len; ++i) T0[i] = 2ll * A[i] % mod;
    Poly::Mul(T1, B, H, t);
    for (int i = 0; i < len; ++i) T0[i] = (T0[i] + 3ll * T1[i] % mod) % mod;
    Poly::Mul(T2, H, H, t), Poly::Mul(T1, T2, H, t);
    for (int i = len - 1; i; --i) T0[i] = (T0[i - 1] + T1[i - 1]) % mod;
    T0[0] = 6;
    for (int i = 0; i < len / 2; ++i) T0[i] = (T0[i] - 6ll * H[i] % mod + mod) % mod;
    for (int i = len - 1; i; --i) T1[i] = 3ll * (B[i - 1] + T2[i - 1]) % mod;
    T1[0] = mod - 6;
    Poly::Inv(T1, T1, len);
    memset(T1 + len, 0, len << 2);
    Poly::Mul(T0, T0, T1, t);
    for (int i = 0; i < len; ++i) H[i] = (H[i] - T0[i] + mod) % mod, A[i] = B[i] = 0;
    for (int i = 0; i < len * 2 / 3; ++i) A[i * 3] = H[i];
    for (int i = 0; i < len; ++i) B[i * 2] = H[i];
  }

  if (t > Poly::tot) t >>= 1, len >>= 1;
  memset(T0, 0, sizeof T0), memset(A, 0, sizeof A), memset(B, 0, sizeof B), memset(C, 0, sizeof C);
  memcpy(T0, H, m << 2);
  for (int i = 0; i < len / 2; ++i) A[i * 4] = T0[i];
  for (int i = 0; i < len * 2 / 3; ++i) B[i * 3] = T0[i];
  for (int i = 0; i < len; ++i) C[i * 2] = T0[i];
  memset(A + len, 0, len << 2), memset(B + len, 0, len << 2), memset(C + len, 0, len << 2);

  for (int i = 0; i < len; ++i) F[i] = 6ll * A[i] % mod;
  Poly::Mul(T1, B, T0, t);
  for (int i = 0; i < len; ++i) F[i] = (F[i] + 8ll * T1[i] % mod) % mod;
  Poly::Mul(T1, C, C, t);
  for (int i = 0; i < len; ++i) F[i] = (F[i] + 3ll * T1[i] % mod) % mod;
  Poly::Mul(T1, T0, T0, t), memset(T1 + len, 0, len << 2), Poly::Mul(T1, T1, C);
  for (int i = 0; i < len; ++i) F[i] = (F[i] + 6ll * T1[i] % mod) % mod;
  Poly::Mul(T1, T0, T0, t), memset(T1 + len, 0, len << 2), Poly::Mul(T1, T1, T1, t);
  for (int i = 0; i < len; ++i) F[i] = (F[i] + T1[i]) % mod;
  for (int i = len - 1; i; --i) F[i] = 1ll * inv24 * F[i - 1] % mod;
  F[0] = 1;

  if (n & 1) printf("%d\n", F[n]);
  else {
    int ans = 1ll * (H[n / 2] + 1ll * H[n / 2] * H[n / 2] % mod) * Pw(2, mod - 2) % mod;
    printf("%d\n", (ans + F[n]) % mod);
  }
    
  return 0;
}
#+end_src

