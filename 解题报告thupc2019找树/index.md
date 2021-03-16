# [解题报告][THUPC2019]找树


## Statement

[传送门](https://loj.ac/p/6622)

定义一个二元运算 $\oplus$ 表示对每一个二进制位分别进行**不同**的位运算 (或 / 与 / 异或).

给定一个 $n$ 个点 $m$ 条边的无向图, 每条边有一个 $[0, 2^w)$ 范围内的边权.

求这张图在 $\oplus$ 运算下的最大生成树权值.

$ 1 \le n \le 70, 1 \le m \le 5000, 1 \le w \le 12 $.

## Solution

直接想最大值完全没有思路.

但可以注意到这个 $w$ 很小, 所以我们可以考虑枚举权值, 然后计算该权值的生成树数量.

生成树数量可以用矩阵树定理求; 特定权值的生成树数量在矩阵里套个生成树就行了.

如果边权之间的运算是普通的 $+$ 的话, 直接把矩阵内的数乘换成多项式乘法就行了.

而这里边权之间的运算是一个二进制运算, 所以我们可以考虑用 FWT.

但一般使用 FWT 时, 所有二进制位的运算类型都是相同的, 而这里每个二进制位都有一个独立的运算类型.

但实际上 FWT 也可以处理这种情况, 只要在 FWT / IFWT 时对于每一个二进制位按照它对应的运算进行对应的操作即可. (为什么是对的我也不会证, 姑且就当个结论来记吧.)

所以我们现在得到一个做法: 把矩阵中的数换成生成函数, 然后在生成函数之间做 FWT, 最后查看每一个权值的生成树数量, 取最大的生成树数量非 0 的权值即为答案.

但求行列式 (高斯消元) 时如果直接把所有数乘都换成 FWT, 复杂度是 $O(n^3 2^w w) \approx 10^{10}$, 显然过不了.

NTT 中一个常见的优化思路是把多项式运算换成点值运算. 这里对于 FWT, 我们也同样可以使用"点值" (也就是 FWT 后得到的数组) 运算来优化.

由于 FWT 不用像 NTT 那样考虑循环卷积的问题, 所以我们可以先一次性把所有多项式先 FWT 一遍, 求完行列式后再 IFWT 回来即可.

并且由于点值运算中各个点值之间是相互独立的, 所以我们可以每次让矩阵中所有多项式的同一个点值构成一个新的矩阵, 对这个矩阵单独求行列式, 最后把每个点值求出的行列式的值拼起来, 最后再 IFWT 回去就行了.

时间复杂度 $O(n^3 w^m) \approx 10^9$. 看着非常不能过, 但它就是能过...

托腮课件里说可以用一个较小的模数来优化常数, 但我直接带 $10^9 + 7$ 它也没 T...

## Code
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

const int _ = 70 + 7;
const int __ = (1 << 12) + 7;
const int mod = 1e9 + 7, inv2 = 500000004;

int n, m, W, tot, a[_][_], f[_][_][__], g[__];
char S[_];

void FWT(int *f, bool ty) {
  for (int len = 2, t = 0; len <= tot; len <<= 1, ++t) {
    int gap = len >> 1;
    for (int i = 0; i < tot; i += len)
      for (int j = 0; j < gap; ++j) {
        int w[2] = { f[i + j], f[i + j + gap] };
        if (S[t] == '&') f[i + j] = (0ll + w[0] + (ty ? mod - w[1] : w[1])) % mod;
        else if (S[t] == '|') f[i + j + gap] = (0ll + w[1] + (ty ? mod - w[0] : w[0])) % mod;
        else if (!ty) f[i + j] = (w[0] + w[1]) % mod, f[i + j + gap] = (w[0] - w[1] + mod) % mod;
        else f[i + j] = 1ll * (w[0] + w[1]) * inv2 % mod, f[i + j + gap] = 1ll * (w[0] - w[1] + mod) * inv2 % mod;
      }
  }
}

int Pw(int a, int p) {
  int res = 1;
  while (p) {
    if (p & 1) res = 1ll * res * a % mod;
    a = 1ll * a * a % mod;
    p >>= 1;
  }
  return res;
}

int Det() {
  int res = 1, t = 0;
  for (int i = 1; i < n; ++i) {
    if (!a[i][i]) {
      for (int j = i + 1; j < n; ++j)
        if (a[j][i]) { for (int k = i; k < n; ++k) swap(a[j][k], a[i][k]); break; }
      t ^= 1;
    }
    if (!a[i][i]) return 0;
    int Inv = Pw(a[i][i], mod - 2); res = 1ll * res * a[i][i] % mod;
    for (int j = i + 1; j < n; ++j) {
      int tmp = 1ll * a[j][i] * Inv % mod;
      for (int k = i; k < n; ++k) a[j][k] = (a[j][k] - 1ll * a[i][k] * tmp % mod + mod) % mod;
    }
  }
  return t ? (mod - res) % mod : res;
}

int main() {
  scanf("%d%d%s", &n, &m, S), W = strlen(S), tot = 1 << W;
  for (int i = 1, x, y, w; i <= m; ++i) {
    scanf("%d%d%d", &x, &y, &w);
    ++f[x][x][w], ++f[y][y][w];
    --f[x][y][w], --f[y][x][w];
  }

  for (int i = 1; i < n; ++i)
    for (int j = 1; j < n; ++j) {
      for (int w = 0; w < tot; ++w) f[i][j][w] = (f[i][j][w] + mod) % mod;
      FWT(f[i][j], 0);
    }

  for (int w = 0; w < tot; ++w) {
    for (int i = 1; i < n; ++i)
      for (int j = 1; j < n; ++j) a[i][j] = f[i][j][w];
    g[w] = Det();
  }
  FWT(g, 1);

  for (int w = tot - 1; ~w; --w)
    if (g[w]) { printf("%d\n", w); exit(0); }
  puts("-1");
  return 0;
}
```

