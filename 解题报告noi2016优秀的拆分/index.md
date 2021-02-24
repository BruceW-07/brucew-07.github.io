# [解题报告] [NOI2016]优秀的拆分


** 题意
若一个拆分方式能把一个串表示为 \(AABB\) 的形式, 则称这个拆分为 "优秀的拆分".

给定一个串 \(S\), 求它所有子串的所有 "优秀的划分" 数量总和.

\(T\) 组数据, \( T \le 10,\ n \le 30000 \)

** 解法
首先可以想到在 \(AA\) 和 \(BB\) 之间的那个位置统计答案, 那么我们就需要对每个位置 \(i\) 算出以它为起点 / 终点的形如 \(AA\) 的串个数.

计算这个要用到一个 "撒点" 的套路.

具体来说就是枚举 \(AA\) 的长度 \(len\), 然后把原串 \(S\) 分为形如 \([1,len],\ [len + 1, 2len], \cdots,  [klen + 1, n] \) 的若干个子串.

然后我们对相邻的子串分别求出它们的 \(LCP\) 和 \(LCS\) (最长公共前缀和最长公共后缀).

然后稍微画一下就可以找出哪些位置可以作为长度为 \(2len\) 的 \(AA\) 串的起点或终点, 对于两个相邻的子串来说, 这些点会构成一个区间, 所以用差分实现一下区间加即可.

对于每个 \(len\), 处理一次的复杂度是 \(O(\frac{n}{len})\) 的, 所以总复杂度为 \( O(\sum_i \frac{n}{i}) = O(n\log n) \).

然后求 \(LCP\) 和 \(LCS\) 用 SA 或者 SAM 即可. (当然二分加哈希也不是不可以, 不过就是多了个 \(\log\).)

** 代码
#+BEGIN_SRC C++
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

typedef long long ll;

const int _ = 1e5 + 7;
const int L = 15;

int n, Log[_], f[2][_][L + 7], num[2][_];
char S[_];

int rk[2][_], sa[_], c[_], t[_], trk[_];

void GetSa(bool ty) {
  for (int i = 0; i <= max(n, (int)'z'); ++i) c[i] = 0;
  for (int i = 0; i <= 2 * n; ++i) rk[ty][i] = 0; // notice that the upper bound here needs to be 2n
  for (int i = 1; i <= n; ++i) rk[ty][i] = S[i], ++c[rk[ty][i]];
  for (int i = 1; i <= 'z'; ++i) c[i] += c[i - 1];
  for (int i = 1; i <= n; ++i) sa[c[rk[ty][i]]--] = i;
  for (int len = 1; len < n; len <<= 1) {
    for (int i = 0; i <= n; ++i) c[i] = 0;
    for (int i = 1; i <= n; ++i) ++c[rk[ty][i]];
    for (int i = 1; i <= n; ++i) c[i] += c[i - 1];
    for (int i = n; i >= 1; --i) if (sa[i] > len) t[c[rk[ty][sa[i] - len]]--] = sa[i] - len;
    for (int i = n - len + 1; i <= n; ++i) t[c[rk[ty][i]]--] = i;
    for (int i = 1; i <= n; ++i) sa[i] = t[i];
    int cnt = 0;
    for (int i = 1; i <= n; ++i) trk[sa[i]] = (i == 1 or rk[ty][sa[i]] != rk[ty][sa[i - 1]] or rk[ty][sa[i] + len] != rk[ty][sa[i - 1] + len]) ? ++cnt : cnt;
    for (int i = 1; i <= n; ++i) rk[ty][i] = trk[i];
  }
}

int hgt[_];

void GetHgt(bool ty) {
  for (int i = 0; i <= n; ++i) hgt[i] = 0;
  for (int i = 1; i <= n; ++i) {
    int j = sa[rk[ty][i] - 1];
    hgt[rk[ty][i]] = max(0, hgt[rk[ty][i - 1]] - 1);
    while (max(i, j) + hgt[rk[ty][i]] <= n and S[i + hgt[rk[ty][i]]] == S[j + hgt[rk[ty][i]]]) ++hgt[rk[ty][i]];
  }
  for (int i = 1; i <= n; ++i) f[ty][i][0] = hgt[i];
  for (int l = 1; l <= L; ++l)
    for (int i = 1; i + (1 << (l - 1)) <= n; ++i) f[ty][i][l] = min(f[ty][i][l - 1], f[ty][i + (1 << (l - 1))][l - 1]);
}

int Query(bool ty, int l, int r) {
  assert(l != r);
  if (ty) l = rk[1][n - l + 1], r = rk[1][n - r + 1];
  else l = rk[0][l], r = rk[0][r];
  if (l > r) swap(l, r); ++l;
  int s = Log[r - l + 1];
  return min(f[ty][l][s], f[ty][r - (1 << s) + 1][s]);
}

int main() {
  for (int i = 1; i <= 30000; ++i) Log[i] = (i == (1 << (Log[i - 1] + 1))) ? Log[i - 1] + 1 : Log[i - 1];

  int T; cin >> T;
  while (T--) {
    scanf("%s", S + 1); n = strlen(S + 1);
    for (int t = 0; t < 2; ++t) {
      GetSa(t), GetHgt(t);
      reverse(S + 1, S + n + 1);
    }

    for (int t = 0; t < 2; ++t)
      for (int i = 1; i <= n; ++i) num[t][i] = 0;
    for (int l = 1; l <= n; ++l)
      for (int i = 1; i + 2 * l - 1 <= n; i += l) {
        int len1 = min(l, Query(1, i + l - 1, i + 2 * l - 1)), len2 = min(l, Query(0, i + l, i + 2 * l));
        int p1 = i + l - len1, p2 = min(i + l + len2 - 1, i + 2 * l - 2);
        if (p1 <= p2 - l + 1) {
          ++num[1][p1], --num[1][p2 - l + 2];
          ++num[0][p1 + 2 * l - 1], --num[0][p2 + l + 1];
        }
      }

    long long ans = 0;
    for (int i = 1; i <= n; ++i) num[0][i] += num[0][i - 1], num[1][i] += num[1][i - 1];
    for (int i = 1; i < n; ++i) ans += 1ll * num[0][i] * num[1][i + 1];
    cout << ans << endl;
  }
  return 0;
}
#+END_SRC

