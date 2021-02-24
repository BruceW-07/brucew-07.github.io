# [解题报告][bzoj3728][bzoj3728]Zarowki


* Statement
[[https://darkbzoj.tk/problem/3728][题面]]

\(n\) 个灯泡和 \(n\) 个房间匹配, 只有灯泡权值比房间权值大才能匹配, 可以将 \(k\) 个灯泡换成权值任意的灯泡, 求存在完美匹配的灯泡总权值的最小值.

* Solution

首先直觉上肯定是要使灯泡和房间的权值之差越小越好.

那么可以发现一个性质, 如果我们要换灯泡, 则一定会将其权值设为某个房间的权值. 

那么我们实际上只要找 \(n - k\) 个匹配, 然后每个匹配的代价为灯泡和房间的权值之差, 最后答案再加上所有房间的权值之和即可.

考虑怎么建图. 要注意用到 "只有灯泡权值比房间权值大才能匹配" 这个性质/条件.

把所有权值(包括灯泡和房间)离散化, 然后从小往大连容量为 \(\inf\), 权值为 \(w_{i + 1} - w_i\) 的边, 然后从 \(S\) 连向房间对应的权值, 从灯泡对应的权值连向 \(T\), 那么在这张图上跑费用流即可得到答案.

发现这里所有边的方向都是一致的, 所以反边是没有用的, 所以我们只需要用堆维护一下权值相邻的房间和灯泡中权值之差的最小值, 每次把最小值的一对灯泡和房间取出来并删除.

这个显然删除操作可以用 \(\mathrm{set}\) 维护, 但是时限只有 1s, 在 darkbzoj 上会 T. 

而实际上把 "删除" 看做 "区间合并", 然后用并查集维护就行了. (这个并查集的细节最好先想清楚.)

* Code
#+begin_src C++
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <iostream>
#include <queue>
#include <set>

#define ins insert
#define ers erase
#define mkp make_pair
#define fi first
#define se second

using namespace std;

typedef long long ll;

const int _ = 1e6 + 7;

int n, K, fa[_], sz[_];
pair<int, int> a[_];
ll ans;
priority_queue<pair<int, int>> h;

int gi() {
  int x = 0; char c = getchar();
  while (!isdigit(c)) c = getchar();
  while (isdigit(c)) x = (x << 3) + (x << 1) + c - '0', c = getchar();
  return x;
}

int Find(int x) { return fa[x] == x ? x : fa[x] = Find(fa[x]); }

int main() {
  cin >> n >> K;
  for (int i = 1; i <= n; ++i) a[i] = mkp(gi(), 1);
  for (int i = 1, w; i <= n; ++i) w = gi(), ans += w, a[n + i] = mkp(w, 0);
  
  sort(a + 1, a + 2 * n + 1);
  a[0].se = 1, a[2 * n + 1].se = 0;
  for (int i = 1; i <= 2 * n + 1; ++i) {
    fa[i] = i, sz[i] = 1;
    if (a[i].se == 0 and a[i + 1].se == 1) h.push(mkp(a[i].fi - a[i + 1].fi, i));
  }

  for (int cnt = 1; cnt <= n - K; ++cnt) {
    if (h.empty()) { puts("NIE"); exit(0); }
    auto t = h.top(); h.pop();
    ans += -t.fi;
    int x = t.se, y = Find(x + 1);
    assert(a[x].se == 0 and a[y].se == 1);
    assert(x >= 1 and y <= 2 * n);
    int tx = x - sz[x], ty = Find(y + 1);
    if (a[tx].se == 0 and a[ty].se == 1) h.push(mkp(a[tx].fi - a[ty].fi, tx));
    fa[x] = y, sz[y] += sz[x];
    fa[y] = ty, sz[ty] += sz[y];
  }

  cout << ans << endl;
  return 0;
}
#+end_src

