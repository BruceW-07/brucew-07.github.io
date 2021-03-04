# [解题报告][USACO21JAN] Paint by Letters P


## Statement

[传送门](https://www.luogu.com.cn/problem/P7295)

一个 $N \times M$ 的网格，每个格子里有一个大写字母。

$Q$ 次询问，每次询问一个子矩阵里的同字符四连通块数量。

$N, M \le 1000, Q \le 1000$。

## Solution

这题应该不算难题，但是那种没见过就写不出来的题……

心路历程：

1. 一顿乱想
2. 什么都没想出来
3. 自闭
4. 看题解

</br>

首先，把每个格子看做一个点，然后在四连通的同字符点之间连边。

显然这样会形成一个平面图。

然后对于平面图连通块数量我们有一个「欧拉公式」。

{{< admonition quote "欧拉公式" true >}}
对于平面图，设 $V$ 为点数，$E$ 为边数, $F$ 为区域数量（不闭合的区域也算一个区域），$C$ 为连通块数量，有
$$F + V - E = C + 1$$
{{< /admonition >}}

这里我们要求的就是 $C$。

$V$ 可以直接用 $(x_2 - x_1 + 1) \times (y_2 - y_1 +1)$ 算出；

$E$ 可以预处理二维前缀和，然后每次询问的时候再遍历边界，把边界的边删除；

$F$ 稍微要想点方法，我们先对整张图 BFS 一遍，然后对于每个区域，选择一个关键点来表示它。然后我们预处理出关键点数量的二维前缀和。在询问时，边界上所有没有闭合的区域都只能算作一个区域，所以我们要减去边界上没有闭合的且关键点在询问矩阵内的区域，并开个桶记录哪些区域已经被删除了，就不会重复统计了。

复杂度为 $O(NM + Q(M+N))$。

## Code
```cpp
#include <cstdio>
#include <iostream>
#include <queue>

#define mkp make_pair
#define fi first
#define se second

using namespace std;

const int _ = 1e3 + 7;
const int __ = 1e6 + 7;

int N, M, Q, area[_][_], sumE[_][_], sumF[_][_], cnt;
char A[_][_];
pair<int, int> rep[__];
bool vis[_][_], del[__];
queue<pair<int, int>> q;

void Extend(int x, int y) { if (!vis[x][y]) q.push(mkp(x, y)), vis[x][y] = 1, area[x][y] = cnt; }

void Bfs(int sx, int sy) {
  q.push(mkp(sx, sy)), vis[sx][sy] = 1, area[sx][sy] = ++cnt;
  rep[cnt] = mkp(sx, sy), ++sumF[sx][sy];
  while (!q.empty()) {
    int x = q.front().fi, y = q.front().se; q.pop();
    if (x + 1 < N and A[x + 1][y] != A[x + 1][y + 1]) Extend(x + 1, y);
    if (y + 1 < M and A[x][y + 1] != A[x + 1][y + 1]) Extend(x, y + 1);
    if (x > 1 and A[x][y] != A[x][y + 1]) Extend(x - 1, y);
    if (y > 1 and A[x][y] != A[x + 1][y]) Extend(x, y - 1);
  }
}

bool Check(pair<int, int> pt, int x1, int x2, int y1, int y2) {
  return pt.fi >= x1 and pt.fi <= x2 and pt.se >= y1 and pt.se <= y2;
}

int main() {
  cin >> N >> M >> Q;
  for (int i = 1; i <= N; ++i) scanf("%s", A[i] + 1);
  for (int i = 1; i <= N; ++i)
    for (int j = 1; j <= M; ++j)
      if (!vis[i][j]) Bfs(i, j);
  for (int i = 1; i <= N; ++i)
    for (int j = 1; j <= M; ++j) {
      sumE[i][j] = sumE[i - 1][j] + sumE[i][j - 1] - sumE[i - 1][j - 1] + (A[i][j] == A[i - 1][j]) + (A[i][j] == A[i][j - 1]);
      sumF[i][j] = sumF[i][j] + sumF[i - 1][j] + sumF[i][j - 1] - sumF[i - 1][j - 1];
    }

  for (int t = 1, x1, y1, x2, y2; t <= Q; ++t) {
    scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
    int V = (x2 - x1 + 1) * (y2 - y1 + 1);
    int E = sumE[x2][y2] - sumE[x1 - 1][y2] - sumE[x2][y1 - 1] + sumE[x1 - 1][y1 - 1];
    for (int i = x1; i <= x2; ++i)
      if (A[i][y1] == A[i][y1 - 1]) --E;
    for (int j = y1; j <= y2; ++j)
      if (A[x1][j] == A[x1 - 1][j]) --E;
    int F = sumF[x2 - 1][y2 - 1] - sumF[x1 - 1][y2 - 1] - sumF[x2 - 1][y1 - 1] + sumF[x1 - 1][y1 - 1] + 1;
    for (int i = x1; i < x2; ++i) {
      if (A[i][y1] != A[i + 1][y1] and Check(rep[area[i][y1]], x1, x2 - 1, y1, y2 - 1) and !del[area[i][y1]])
        del[area[i][y1]] = 1, --F;
      if (A[i][y2] != A[i + 1][y2] and Check(rep[area[i][y2 - 1]], x1, x2 - 1, y1, y2 - 1) and !del[area[i][y2 - 1]])
        del[area[i][y2 - 1]] = 1, --F;
    }
    
    for (int j = y1; j < y2; ++j) {
      if (A[x1][j] != A[x1][j + 1] and Check(rep[area[x1][j]], x1, x2 - 1, y1, y2 - 1) and !del[area[x1][j]])
        del[area[x1][j]] = 1, --F;
      if (A[x2][j] != A[x2][j + 1] and Check(rep[area[x2 - 1][j]], x1, x2 - 1, y1, y2 - 1) and !del[area[x2 - 1][j]])
        del[area[x2 - 1][j]] = 1, --F;
    }
    for (int i = x1; i < x2; ++i) del[area[i][y1]] = del[area[i][y2 - 1]] = 0;
    for (int j = y1; j < y2; ++j) del[area[x1][j]] = del[area[x2 - 1][j]] = 0;
    printf("%d\n", F + V - E - 1);
  }
  return 0;
}
```

