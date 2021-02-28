# [解题报告][AHOI2009]最小割


# Statement

[传送门](https://www.luogu.com.cn/problem/P4126)

给定一张 $n$ 个点，$m$ 条边的有向图，并给定源点 $S$ 和汇点 $T$，对每条边进行判断

1. 该边是否可能在最小割中
2. 该边是否一定在最小割中

$n \le 4 \times 10^3, m \le 6 \times 10^4$.

# Solution

首先可以感性理解：原图中的重边可以看做一条边，流量为它们流量的总和。

但在代码编写中并不需要将这些边合并，这里提到这一点只是为了便于描述以及对便于以下证明的理解。

---

**一条有向边 $(u,v)$ 可能在最小割中的充要条件**：

1. 该边满流。
2. 残量网络上不存在一条从 $u$ 到 $v$ 的路径。即残量网络中 $u$ 和 $v$ 不在同一个 $SCC$ 中。

**简证**：

首先可以感性理解，不满流的边显然不在最小割中。（因为感觉这部分比较好理解，所以就没有仔细想理性的证明方法，对想知道证明的读者表示歉意。）

然后「边 $(u,v)$ 可能在最小割中」实际上等价于：把 $(u,v)$ 的原流量 $c$ 减小一个极小值 $\varepsilon$，则新图的最小割 / 最大流会减小。

那么假设 $(u,v)$ 的原流量减小了 $\varepsilon$，如果新图的最大流也减小了 $\varepsilon$，则必定存在一条从 $S$ 到 $u$ 的路径以及一条从 $v$ 到 $T$ 的路径，否则就说明原来这条流量为 $\varepsilon$ 的路径从另外一个地方增广出去了，那么新图的最大流就没有变化，与假设矛盾。

那么现在存在一条从 $S$ 到 $u$ 的路径、一条从 $v$ 到 $T$ 的路径以及一条从 $u$ 到 $v$ 的路径，那么就存在一条从 $S$ 到 $T$ 的增广路，那么新图的最大图就没有变化，与假设矛盾。

于是得证。

---

**一条有向边 $(u,v)$ 一定在最小割中的充要条件**：

1. 该边满流。
2. 残量网络中存在一条从 $S$ 到 $u$ 的路径，且存在一条从 $v$ 到 $T$ 的路径。即 $S$ 和 $u$ 在同一个 $SCC$ 中，且 $v$ 和 $T$ 在同一个 $SCC$ 中。

**关于「条件 2」**：

这里我们不考虑原流量 $c \le 0$ 的情况。

那么若 $(u,v)$ 满流，则不然存在一条从 $u$ 到 $S$ 的反向路径和一条从 $T$ 到 $v$ 的反向路径。

所以就解释了「条件 2」的中第一句话可以导出第二句话的原因。

**简证**：

首先满流依然是可以感性理解的，这里不在赘述。

然后「边 $(u,v)$ 一定在最小割中」等价于：把 $(u,v)$ 的原流量 $c$ 增加一个正数 $\Delta c$，则新图的最小割 / 最大流会增加。

给 $(u,v)$ 的原流量增加一个正数 $\Delta c$，相当于在残量网络上增加一条边 $(u,v,\Delta c)$。

那么如果存在一条从 $S$ 到 $u$ 的路径以及一条从 $v$ 到 $T$ 的路径，那么就存在一条从 $S$ 到 $T$ 的增广路，于是新图的最大流会增加，于是就说明边 $(u,v)$ 一定在最小割中。

得证。

---

于是我们只需要对原图跑一边最大流，然后在残量网络上跑一遍 $Tarjan$ 算法来求出 $SCC$，最后对每条边按照上述的条件判断即可。

复杂度为 $O(Dinic(n, m) + n + m)$。

# Code

注：代码中的注释部分记录的是作者在编写代码时出错过的地方，读者可不必在意。

```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <map>
#include <queue>

#define mkp make_pair
#define fi first
#define se second

using namespace std;

const int _ = 4e3 + 7;
const int __ = 12e4 + 7;
const int inf = 1e9;

int n, m, S, T;
int dis[_], lst[_], cur[_], nxt[__], to[__], cap[__], tot = 1;
int dfn[_], low[_], stk[_], top, scc[_], Dfn, Scc;
bool b[_];
pair<int, int> e[__];
queue<int> q;
map<pair<int, int>, bool> con;

void Add(int x, int y, int c) {
  nxt[++tot] = lst[x], to[tot] = y, cap[tot] = c, lst[x] = tot;
  nxt[++tot] = lst[y], to[tot] = x, cap[tot] = 0, lst[y] = tot;  // cap[tot] = c,
}

bool Bfs() {
  memset(dis, 0, sizeof dis);
  while (!q.empty()) q.pop();
  dis[S] = 1, q.push(S);
  while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int i = lst[u]; i; i = nxt[i]) {
      int v = to[i];
      if (dis[v] or !cap[i]) continue;
      dis[v] = dis[u] + 1, q.push(v);
      if (v == T) return 1;
    }
  }
  return 0;
}

int Extend(int u, int flow) {
  if (u == T) return flow;
  int res = flow;
  for (int &i = cur[u]; i; i = nxt[i]) {
    int v = to[i];
    if (dis[v] != dis[u] + 1 or !cap[i]) continue;
    int cst = Extend(v, min(cap[i], res));
    cap[i] -= cst, cap[i ^ 1] += cst, res -= cst;
    if (!res) break;
  }
  return flow - res;
}

void Dinic() {
  int maxflow = 0, flow = 0;
  while (Bfs()) {
    memcpy(cur, lst, sizeof lst);
    do {
      flow = Extend(S, inf);
      maxflow += flow;
    } while (flow);
  }
  cerr << "maxflow: " << maxflow << endl;
}

void Tarjan(int u, int fa) {
  dfn[u] = low[u] = ++Dfn, stk[++top] = u, b[u] = 1;
  for (int i = lst[u]; i; i = nxt[i]) {
    int v = to[i];
    if (!cap[i]) continue;
    if (!dfn[v]) Tarjan(v, u), low[u] = min(low[u], low[v]);
    else if (b[v]) low[u] = min(low[u], low[v]);
  }
  if (low[u] == dfn[u]) {
    scc[u] = ++Scc, b[u] = 0;
    while (stk[top] != u) scc[stk[top]] = Scc, b[stk[top]] = 0, --top;
    --top;
  }
}

int main() {
  cin >> n >> m >> S >> T;
  for (int i = 1, c; i <= m; ++i) {
    scanf("%d%d%d", &e[i].fi, &e[i].se, &c);
    Add(e[i].fi, e[i].se, c);
  }

  Dinic();
  for (int i = 1; i <= n; ++i)
    if (!dfn[i]) Tarjan(i, 0);

  for (int u = 1; u <= n; ++u)
    for (int i = lst[u]; i; i = nxt[i])
      if (cap[i]) con[mkp(u, to[i])] = 1;

  for (int i = 1; i <= m; ++i)
    printf("%d %d\n", !con[e[i]] and scc[e[i].fi] != scc[e[i].se], !con[e[i]] and scc[e[i].fi] == scc[S] and scc[e[i].se] == scc[T]);  // con[e[i]]
  return 0;
}
```

# Reference
[题解 P4126 【[AHOI2009]最小割】](https://www.luogu.com.cn/blog/zadow/solution-p4126) by 斗神·君莫笑

PPT 【网络流简单入⻔】 by xzz

