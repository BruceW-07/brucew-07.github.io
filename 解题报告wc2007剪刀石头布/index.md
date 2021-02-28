# [解题报告][WC2007]剪刀石头布


# Statement

[传送门](https://www.luogu.com.cn/problem/P4249)

给定一个 $n$ 个点的竞赛图，有一些边已经确定方向，要求你为剩下的边定向，并使得图中三元环数量尽量多。

输出最多的三元环数量以及一种定向方案。

$n \le 100$。

# Solution

一个三元组 $(i, j, k)$ （$i,j,k$ 互不相同）不构成三元环当且仅当存在一个点的入度或出度等于2。

那么三元环个数可以表示为

$$
\mathrm{num} = \binom{n}{3} - \sum_{i = 1}^n \binom{indeg_i}{2}
$$

要是三元环数量尽量多，我们感性上就是要使每个点的入度尽量的少。

而为边 $(u,v)$ 定向的过程就相当于把 $u$ 的入度加一或把 $v$ 的入度加一。

而假设一个点 $i$ 当前的入度为 $indeg_i$，那么它的入度加一后，它贡献的代价的变化量为 $\binom{indeg_i + 1}{2} - \binom{indeg_i}{2}$。

所以我们可以这样连边：

1. 对于每一条未定向的边 $j$，从 $S$ 向 $n + j$ 连一条容量为 $1$，费用为 $0$ 的边；

2. 对每一条未定向的边 $j = (u, v)$，从 $n + j$ 分别向 $u,v$ 连一条容量为 $1$，费用为 $0$ 的边；

3. 对每个点 $i$，设它初始时的入度为 $indeg_i$，从 $i$ 向 $T$ 分别连 $n - 1 - indeg_i$ 条容量为 $1$， 费用为 $\binom{k}{2} - \binom{k - 1}{2} (k \in [indeg_i + 1, n - 1])$ 的边。

在这张图上跑最小费用最大流。设最小费用为 $\mathrm{cost}$，则最终答案为

$$
\mathrm{num} = \binom{n}{3} - \sum_{i = 1}^n \binom{indeg_i}{2} - \mathrm{cost}
$$

$n \le 3$ 的情况可能需要特判一下。

构造方案只需要对于每一条未定向的边 $(u,v)$ 看它连向 $u$ 的边满流还是连向 $v$ 的边满流即可。

复杂度为 $O(费用流(n^2, n^2))$。

# Code
```cpp
#include <cassert>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <queue>

#define mkp make_pair
#define fi first
#define se second

using namespace std;

const int _ = 2e4 + 7;
const int __ = 1e5 + 7;
const int inf = 1e9;
const int S = 2e4 + 1, T = 2e4 + 2;

int n, ind[_], a[107][107], cnt, ans;
int dis[_], dep[_], lst[_], cur[_], nxt[__], to[__], cap[__], wgt[__], tot = 1;
bool b[_];
queue<int> q;
pair<int, int> e[_];

void Add(int x, int y, int c, int w) {
  nxt[++tot] = lst[x], to[tot] = y, cap[tot] = c, wgt[tot] = w, lst[x] = tot;
  nxt[++tot] = lst[y], to[tot] = x, cap[tot] = 0, wgt[tot] = -w, lst[y] = tot;
}

int C2(int n) { return n ? 1ll * n * (n - 1) / 2 : 0; }

bool SPFA() {
  memset(dis, 0x3f, sizeof dis);
  memset(dep, 0, sizeof dep);
  dis[S] = 0, dep[S] = 1, q.push(S);
  while (!q.empty()) {
    int u = q.front(); q.pop(); b[u] = 0;
    for (int i = lst[u]; i; i = nxt[i]) {
      int v = to[i];
      if (dis[v] <= dis[u] + wgt[i] or !cap[i]) continue;
      dis[v] = dis[u] + wgt[i], dep[v] = dep[u] + 1;
      if (!b[v]) q.push(v), b[v] = 1;
    }
  }
  return dis[T] != dis[0];
}

int Extend(int u, int flow) {
  if (u == T) return flow;
  int res = flow;
  for (int &i = cur[u]; i; i = nxt[i]) {
    int v = to[i];
    if (dis[v] != dis[u] + wgt[i] or dep[v] != dep[u] + 1 or !cap[i]) continue;
    int cst = Extend(v, min(cap[i], res));
    cap[i] -= cst, cap[i ^ 1] += cst, res -= cst;
    if (!res) break;
  }
  return flow - res;
}

void Dinic() {
  int flow;
  while (SPFA()) {
    memcpy(cur, lst, sizeof lst);
    do {
      flow = Extend(S, inf);
      ans -= dis[T] * flow;
    } while (flow);
  }

}

int main() {
  cin >> n; ans = 1ll * n * (n - 1) * (n - 2) / 6;
  for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= n; ++j) scanf("%d", &a[i][j]);
  for (int i = 1; i <= n; ++i)
    for (int j = i + 1; j <= n; ++j)
      if (a[i][j] == 0) ++ind[i];
      else if (a[i][j] == 1) ++ind[j];
      else {
        e[++cnt] = mkp(i, j);
        Add(S, cnt + n, 1, 0);
        Add(cnt + n, i, 1, 0);
        Add(cnt + n, j, 1, 0);
      }
  for (int i = 1; i <= n; ++i) {
    ans -= C2(ind[i]);
    for (int j = ind[i] + 1; j < n; ++j) Add(i, T, 1, C2(j) - C2(j - 1));
  }

  Dinic();

  cout << (n < 3 ? 0 : ans) << endl;
  for (int u = 1; u <= cnt; ++u)
    for (int i = lst[n + u]; i; i = nxt[i]) {
      int v = to[i];
      if (v == e[u].fi and !cap[i]) { a[e[u].fi][e[u].se] = 0; break; }
      if (v == e[u].se and !cap[i]) { a[e[u].fi][e[u].se] = 1; break; }
    }
  for (int i = 1; i <= n; ++i)
    for (int j = 1; j < i; ++j) a[i][j] = 1 - a[j][i];
  for (int i = 1; i <= n; ++i) {
    for (int j = 1; j <= n; ++j) printf("%d ", a[i][j]);
    putchar('\n');
  }
  return 0;
}
```

