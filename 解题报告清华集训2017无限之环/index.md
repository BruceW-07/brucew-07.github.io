# [解题报告][清华集训2017]无限之环


# Statement

[传送门](https://www.luogu.com.cn/problem/P4003)

一个 $n \times m$ 的网格，每个网格中可能有一个以下15类水管中的一个。

可以花费代价 1 ，逆时针或顺时针90°旋转一个格子里的水管。

求最小代价使得所有水管形成若干个环。

$ n \times m \le 2000$。

# Solution

~~网络流建图系列之——那些不看题解就完全想不出来的建图。~~

首先由于水管是在网格上面，所以先考虑利用二分图的性质。

在每个格子中央建一个水管；

对于所有白色格子 $i$， 从 $S$ 向 $i$ 连一条容量为接口数量，费用为 0 的边；

对于所有给色格子 $j$， 从 $j$ 向 $T$ 连一条容量为接口数量，费用为 $0$ 的边。

然后对每个格子 $i$ 的 4 个边界分别建一个节点，设为 $k \times n \times m + i$，然后对一条边线两侧的节点连一条容量为 1，费用为 0 的边，方向从白色一侧连向黑色一侧。

然后我们现在考虑对于所有白点，如何从 $i$ 向 $k \times n \times m + i$ 连边。（对于黑点来说就是把所有边反向。）

大致的思路就是：用旋转后的方向占用旋转前的方向的容量。

设 $(c,w)$ 表示容量为 $c$，费用为 $w$ 的边。分类讨论一下：

1. 对于只有一个接口的水管：
   1. 从中央节点向该接口方向连边 $(1, 0)$；
   2. 从中央节点向与该接口相邻的两个方向连边 $(1, 1)$；
   3. 从中央节点向与该接口相对的方向连边 $(1, 2)$。
2. 对于 L 字形的水管：
   1. 从中央节点向两个接口的方向连边 $(1, 0)$;
   2. 分别从两个接口对应的节点向它相对方向连边 $(1, 1)$。
3. 对于 T 字形水管：
   1. 从中央节点向三个接口的方向连边 $(1, 0)$;
   2. 从 $T$ 字顶部横线的那两个方向的节点向没有接口的方向连边 $(1, 1)$；
   3. 从 $T$ 字形底部的节点向没有接口的方向连边 $(1, 2)$。
   
对于剩余的水管，要么是不能旋转，要么是旋转后没有变化，所以只需要从中央节点向有接口的方向连边即可。

对于黑色格子，把所有连边反向即可。

最后在这张图上跑最小费用最大流。

若所有白格子的接口数量总和等于黑格子的接口数量总和，且最大流等于白格子接口数量和，则答案为最小费用；

否则无解，输出 -1 即可。

## Code
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
const int inf = 1;
const int S = 2e4 + 1, T = 2e4 + 2;

int n, m, nm, All[2], MaxFlow, Cst;
int dis[_], dep[_], lst[_], cur[_], nxt[__], to[__], cap[__], wgt[__], tot = 1;
bool b[_];
queue<int> q;

int Num(int i, int j) {
  if (i < 1 or i > n or j < 1 or j > m) return 0;
  return m * (i - 1) + j;
}

int pop_count(int x) { return (x & 1) + (x >> 1 & 1) + (x >> 2 & 1) + (x >> 3 & 1); }

void Add(int x, int y, int c, int w, bool ty = 1) {
  if (!x or !y) return;
  if (!ty) swap(x, y);
  nxt[++tot] = lst[x], to[tot] = y, cap[tot] = c, wgt[tot] = w, lst[x] = tot;
  nxt[++tot] = lst[y], to[tot] = x, cap[tot] = 0, wgt[tot] = -w, lst[y] = tot;
}

bool SPFA() {
  memset(dis, 0x3f, sizeof dis), memset(dep, 0, sizeof dep);
  dis[S] = 0, dep[S] = 1, q.push(S);
  while (!q.empty()) {
    int u = q.front(); q.pop(), b[u] = 0;
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
    int cst = Extend(v, min(res, cap[i]));
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
      MaxFlow += flow;
      Cst += flow * dis[T];
    } while (flow);
  }
}

int main() {
  cin >> n >> m; nm = n * m;
  for (int i = 1, x, id, cnt, ty; i <= n; ++i)
    for (int j = 1; j <= m; ++j) {
      scanf("%d", &x), cnt = pop_count(x), id = Num(i, j), ty = (i + j) & 1;
      if (!cnt) continue;
      All[ty] += cnt;
      if (ty) Add(S, id, cnt, 0);
      else Add(id, T, cnt, 0);
      for (int k = 0; k < 4; ++k)
        if (x >> k & 1) Add(id, (k + 1) * nm + id, 1, 0, ty);
      bool flag = cnt == 2 and (((x >> 0 & 1) and (x >> 2 & 1)) or ((x >> 1 & 1) and (x >> 3 & 1)));
      if (cnt == 1) {
        int t = 0;
        for (int k = 0; k < 4; ++k)
          if (x >> k & 1) t = k;
        for (int k = 0; k < 4; ++k)
          if ((k - t + 4) % 4 == 2) Add(id, (k + 1) * nm + id, 1, 2, ty);
          else if (k != t) Add(id, (k + 1) * nm + id, 1, 1, ty);
      }
      if (cnt == 2 and !flag) {
        for (int k = 0; k < 4; ++k)
          if (x >> k & 1) Add((k + 1) * nm + id, ((k + 2) % 4 + 1) * nm + id, 1, 1, ty);
      }
      else if (cnt == 3) {
        int t = 0;
        for (int k = 0; k < 4; ++k)
          if (!(x >> k & 1)) t = k;
        Add(((t + 2) % 4 + 1) * nm + id, (t + 1) * nm + id, 1, 2, ty);
        Add(((t + 1) % 4 + 1) * nm + id, (t + 1) * nm + id, 1, 1, ty);
        Add(((t + 3) % 4 + 1) * nm + id, (t + 1) * nm + id, 1, 1, ty);
      }

      if (ty) {
        int n1 = Num(i - 1, j), n2 = Num(i, j + 1), n3 = Num(i + 1, j), n4 = Num(i, j - 1);
        if (n1) Add(1 * nm + id, 3 * nm + n1, 1, 0);
        if (n2) Add(2 * nm + id, 4 * nm + n2, 1, 0);
        if (n3) Add(3 * nm + id, 1 * nm + n3, 1, 0);
        if (n4) Add(4 * nm + id, 2 * nm + n4, 1, 0);
      }
    }

  if (All[0] != All[1]) { puts("-1"); exit(0); }
  Dinic();
  printf("%d\n", MaxFlow == All[0] ? Cst : -1);
  return 0;
}
```

