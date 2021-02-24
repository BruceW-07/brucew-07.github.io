# [解题报告][300iq Contest 2 H]Honorable Mention


* Statement
[[http://codeforces.com/gym/102331/problem/H][题面]]

一个长度为 \(n\) 的序列 \( \{ a \} \), 有 \(Q\) 个形如 \((l, r, )\) 的询问, 每次需要回答在区间 \([l,r]\) 内选择 *恰好* \(k\) 个 *不相交区间* 的元素和最大值.

\( n, Q \le 35000, |a_i| \le 35000\).

* Solution

先考虑对于单个询问怎么做.

假设询问区间为 \([1, n]\), 设 DP 状态 \(f_{i, j}\) 为考虑到第 \(i\) 位, 选择了 \(j\) 个区间的最大值. 容易得到转移.

\[
f_{i, j} = \max(f_{i - 1, j}, \max_{k = 0}^{i - 1} f_{k, j - 1} + sum_{i} - sum_{k})
\]

其中 \(\{ sum \}\) 是 \(\{ a \}\) 的前缀和.

加上一个前缀最大值优化, 我们就得到了一个 \(O(n^2Q)\) 的做法.

考虑怎么优化.

如果你见得多或者直觉比较好, 大概可以猜出 \(F(x) = f_{n, x} (x \in [1, n])\) 这个函数可能是凸的.

证明函数凸性的话就考虑是否能建出一个费用流模型, 如果可以, 就表示它一定是凸的. 

#+BEGIN_QUOTE
因为最大费用流的过程中我们总是先拓展费用最大的流, 所以每次拓展的流的费用是递减的, 就相当于函数的变化量 \( \Delta \) 是递减的, 也就表明这函数是个凸函数.
#+END_QUOTE

设 \((u, v, c, w)\) 为一条从 \(u\) 连向 \(v\) 的容量为 \(c\), 费用为 \(w\) 的边.

考虑把每个元素 \(a_i\) 拆成一个入点 \(u_i\) 和出点 \(v_i\), 连接 \((u_i, v_i, 1, a_i)\).

然后 \( \forall i \in [1, n - 1] \), 连接 \((v_i, u_{i + 1}, 1, 0)\); \(\forall i \in [1, n]\), 连接 \((S, u_i, 1, 0), (v_i, T, 1, 0)\).

这样就把费用流模型建好了, 所以函数 \(F(x)\) 是凸的, 所以我们可以用凸优化来优化它的计算.

假设我们当前要求 \((l, r, k)\), 那么我们二分一个斜率 \(mid\), 并找出该直线的在凸包上的切点 \((x, F(x)\), 若 \(x \ge k\), 则增大 \(mid\), 否则减小.

然后找切点就相当于找凸包上距离斜率为 \(mid\) 的直线最近的点, 也就是 \( x \cdot mid - F(x) \) 最小的点, 也就是 \((x, F(x) - x \cdot mid) \) 的最高点.

那么我们考虑怎么求 \(F(x) - x \cdot mid\) 的最大值.

设 \( g_{i} \) 表示考虑到 \(i\) 时的答案, 并且 \(g_i\) 是一个 \(std::pair\), 第一维记的是 \(F(x) - x \cdot mid\), 第二维记的是 \(x\). 那么容易写出转移方程.

\[
g_i = \max(g_{i - 1}, \max_{k = 1}^{i - 1} g_k + (sum_i - sum_k, 1))
\]

同样, 使用前缀和优化的话一次 DP 可以做到 \(O(n)\), 所以总复杂度为 \(O(nQ \log value)\).

+然而这个 DP 和正解好像并没有什么关系.+

想要做到更优的复杂度, 我们可以考虑用一个数据结构来维护凸包. 这里我们使用线段树.

对于线段树上的一个节点 \(k\) 和它所代表的区间 \([l, r]\), 我们维护出以 \([l, r]\) 为选取区间时的 \(F(x), x \in [1, r - l + 1]\).

这个东西还是比较好弄的, 就是在合并子区间的时候用闵可夫斯基和维护一下即可.

#+BEGIN_QUOTE
闵可夫斯基和就相当于利用函数的凸性来加速背包合并的过程.

具体来说就是对两个子区间的凸包维护分别维护一个指针, 每次将指针所指向的两个值之和贡献给父区间的对应位置, 然后将变化量 \(\Delta\) 更大的一个指针往后移动一位.
#+END_QUOTE

然后这里有个要注意的地方是, 如果左子区间选了最右边的一个元素, 右子区间也选了最左边的一个元素, 那么合并的时候可以认为少选择了一个区间.

所以我们对每个节点实际上要维护 4 个凸包, 分别表示左右端点的元素是否选择的情况下的 \(F(x)\).

这样我们就可以 \(O(n \log n)\) 建出来一棵线段树.

对于查询的话, 我们还是按照凸优化的套路.

先把查询区间 \([L, R]\) 在线段树上对应的区间抠出来, 然后二分一个 \(mid\), 在每个区间的凸包上都二分找到斜率为 \(mid\) 的直线与该凸包的切点 (对于 4 个凸包都要找), 然后在对这些切点所对应的值做一个 01 背包, 求出最大值, 那么就找到了区间 \([L, R]\) 内 \(F(x) - x \cdot mid\) 的最大值, 然后接着二分即可.

但是这样的话一次询问的复杂度是 \(O(\log^ n \log value)\) 的 (\( O(\log n)\) 个线段树上的区间, 找切点一个 \(\log\), 凸优化的二分又一个 \(log\)), 无法通过.

优化的话我们考虑使用 *整体二分* 把 在线段树区间的凸包上找切点 的复杂度优化掉.

我们对所有询问一起进行凸优化, 然后在整体二分进行到某一层的时候, 我们按照 \(mid\) 从大到小枚举每个询问 (这个 \(mid\) 指的是它在凸优化过程中二分出来的斜率 \(mid\), 而不是区间中点.), 然后把 在线段树区间的凸包上找切点 的二分改为 维护一个指针每次暴力移动.

然后因为对于同一个凸包来说, 斜率 \(mid\) 越小, 则它在该凸包上的切点位置越靠右 (画图理解), 所以对于线段树上每个区间的凸包, 它的指针的移动是单调的.

也就是说, 对于整体二分中的每一层, 线段树上所有指针的总移动次数是 \(O(n \log n)\) 的, 那么总复杂度就是 \(O(n \log n \log value)\), 可以通过.

实现的时候由于线段树上的凸包需要用 \(std::vector\) 维护, 而这东西的常数又比较大, 所以不要在整体二分的每一层都对线段树上所有节点的凸包都移动, 而是只要访问到一个节点后移动它的凸包上的指针就行了.

还有凸优化二分的时候会出现凸包上三点贡献的情况, 然后按照上述的二分方法, 这时我们会将 \(mid\) 指向这些共线的点中最右边的那个点, 所以我们对于一个询问 \((l, r, k)\), 在所有 \(x \ge k\) 的情况下 (\(x\) 就是切点横坐标) 都要覆盖答案的值. (因为二分进行到越后面, 得到的 \(x\) 就会越接近于 \(k\), 所以最后一个满足 \(x \ge k\) 的 \(x\) 也就是最小的 \(x\), 也就是与 \((k, F(k))\) 共线的 \((x, F(x))\).)

* Code
#+BEGIN_SRC C++
#include <cassert>
#include <cstdio>
#include <ctime>
#include <iostream>
#include <vector>

#define pb push_back
#define sz(x) (int)(x).size()
#define mkp make_pair
#define fi first
#define se second

using namespace std;

typedef long long ll;

const int _ = 35000 + 7;
const int __ = 1e6 + 7;
const ll inf = 1e18;

int n, Q, a[_], tot;
struct QUE { int l, r, k; int ans; } qu[_];
vector<ll> f[__][2][2];
pair<ll, int> g[2];
int pt[__][2][2], sz[__];

void upd(ll &x, ll y) { x = max(x, y); }

namespace SGT {
#define mid ((l + r) >> 1)
#define ls(k) (k << 1)
#define rs(k) (k << 1 | 1)
  
  void Merge(int k, int len1, int len2, int a, int b, int c, int d) {
    int p1 = 1, p2 = 1;
    while (p1 <= len1 and p2 <= len2) {
      ll tmp = f[ls(k)][a][b][p1] + f[rs(k)][c][d][p2];
      upd(f[k][a][d][p1 + p2], tmp);
      if (b and c) upd(f[k][a][d][p1 + p2 - 1], tmp);
      ll d1 = f[ls(k)][a][b][p1 + 1] - f[ls(k)][a][b][p1];
      ll d2 = f[rs(k)][c][d][p2 + 1] - f[rs(k)][c][d][p2];
      if (p1 != len1 and (p2 == len2 or d1 >= d2)) ++p1;
      else ++p2;
    }
  }

  void Build(int k, int l, int r) {
    tot = max(tot, k), sz[k] = r - l + 1;
    for (int i = 0; i < 2; ++i)
      for (int j = 0; j < 2; ++j) {
        f[k][i][j].resize(r - l + 2);
        for (int t = 1; t <= r - l + 1; ++t) f[k][i][j][t] = -inf;
      }
    if (l == r) return (void)(f[k][1][1][1] = a[l]);
    Build(ls(k), l, mid);
    Build(rs(k), mid + 1, r);
    for (int i = 0; i < 2; ++i) {
      for (int j = 1; j <= mid - l + 1; ++j) upd(f[k][i][0][j], max(f[ls(k)][i][0][j], f[ls(k)][i][1][j]));
      for (int j = 1; j <= r - mid; ++j) upd(f[k][0][i][j], max(f[rs(k)][0][i][j], f[rs(k)][1][i][j]));
    }
    for (int a = 0; a < 2; ++a)
      for (int b = 0; b < 2; ++b)
        for (int c = 0; c < 2; ++c)
          for (int d = 0; d < 2; ++d)
            Merge(k, mid - l + 1, r - mid, a, b, c, d);
  }

  void Move(int k, int i, int j, int slope) {
    int p = pt[k][i][j];
    while (p + 1 <= sz[k] and slope <= f[k][i][j][p + 1] - f[k][i][j][p]) ++p;
    pt[k][i][j] = p;
  }

  void Query(int k, int l, int r, int x, int y, int cst) {
    if (l >= x and r <= y) {
      pair<ll, int> t0 = mkp(-1e18, 0), t1 = mkp(-1e18, 0);
      for (int i = 0; i < 2; ++i)
        for (int j = 0; j < 2; ++j) Move(k, i, j, cst);
      for (int i = 0; i < 2; ++i) {
        for (int j = 0; j < 2; ++j) {
          int p0 = pt[k][j][0], p1 = pt[k][j][1];
          ll f0 = f[k][j][0][p0], f1 = f[k][j][1][p1];
          if (i and j) {
            t0 = max(t0, mkp(g[i].fi + f0 - 1ll * (p0 - 1) * cst, g[i].se + p0 - 1));
            t1 = max(t1, mkp(g[i].fi + f1 - 1ll * (p1 - 1) * cst, g[i].se + p1 - 1));
          }
          t0 = max(t0, mkp(f0 - 1ll * p0 * cst, p0));
          t1 = max(t1, mkp(f1 - 1ll * p1 * cst, p1));
          t0 = max(t0, mkp(g[i].fi + f0 - 1ll * p0 * cst, g[i].se + p0));
          t1 = max(t1, mkp(g[i].fi + f1 - 1ll * p1 * cst, g[i].se + p1));
        }
        if (g[i].se) t0 = max(t0, g[i]);
      }
      g[0] = t0, g[1] = t1;
      return;
    }
    if (x <= mid) Query(ls(k), l, mid, x, y, cst);
    if (y > mid) Query(rs(k), mid + 1, r, x, y, cst);
  }

#undef mid
#undef ls
#undef rs
}

pair<pair<int, int>, int> p[_], tmp1[_], tmp2[_], pn[_];

void Calc() {
  int l = -2e9, r = 2e9; int cnt = Q;
  for (int i = 1; i <= Q; ++i) p[i] = mkp(mkp(l, r), i);
  while (cnt) {
    int cntn = 0;
    for (int i = 1; i <= tot; ++i) pt[i][0][0] = pt[i][0][1] = pt[i][1][0] = pt[i][1][1] = 1;
    for (int i = cnt, j = cnt; i; i = j) {
      int t1 = 0, t2 = 0;
      while (j and p[j].fi.fi == p[i].fi.fi) --j;
      ll l = p[i].fi.fi, r = p[i].fi.se, mid = (l + r) >> 1;
      for (int k = i; k > j; --k) {
        int x = p[k].se;
        g[0] = mkp(0, 0), g[1] = mkp(-1e18, 0);
        SGT::Query(1, 1, n, qu[x].l, qu[x].r, mid);
        auto tmp = max(g[0], g[1]);
        if (tmp.se >= qu[x].k) {
          if (r > mid) tmp2[++t2] = mkp(mkp(mid + 1, r), x);
          qu[x].ans = tmp.fi + 1ll * mid * qu[x].k;
        }
        else if (l < mid) tmp1[++t1] = mkp(mkp(l, mid - 1), x);
      }
      for (int k = 1; k <= t2; ++k) pn[++cntn] = tmp2[k];
      for (int k = 1; k <= t1; ++k) pn[++cntn] = tmp1[k];
    }
    cnt = cntn;
    for (int i = 1; i <= cnt; ++i) p[i] = pn[cnt - i + 1];
  }
}

int main() {
  cin >> n >> Q;
  for (int i = 1; i <= n; ++i) scanf("%d", &a[i]);
  for (int i = 1; i <= Q; ++i) scanf("%d%d%d", &qu[i].l, &qu[i].r, &qu[i].k), qu[i].ans = -inf;
  SGT::Build(1, 1, n);
  Calc();
  for (int i = 1; i <= Q; ++i) printf("%d\n", qu[i].ans);
  return 0;
}
#+END_SRC



