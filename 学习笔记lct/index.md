# [学习笔记]LCT


## 简介

用于解决「动态树」问题。即对一棵树进行加边、删边操作，保证操作后仍是一棵树 （或一棵森林），并维护树链、子树信息。

具体的方式是将原树 / 森林划分成若干条链，并用一棵 Splay 维护一条链，不同的 Splay 之间按照原树结构连边。

特点是好维护链信息，不好维护子树信息。

</br>

## 变量定义

```cpp
int fa[_], ch[_][2], val[_], sum[_];
bool tag[_];
```

`fa[x]`：x 在 Splay 上的父亲。对于 Splay 的根节点来说，就是该棵 Splay 中最浅的节点在原树中的父亲。

`ch[x][0/1]`：x 在 Splay 中的左 / 右儿子。

`val[x]`：x 的点权。

`sum[x]`：x 在 Splay 中的子树权值和。

`tag[x]`：x 的子树是否要交换左右儿子。（用于 `MakeRoot` 函数）

</br>

## 宏定义
```cpp
#define get(x) (x == ch[fa[x]][1])
#define isroot(x) (x != ch[fa[x]][0] and x != ch[fa[x]][1])
```

`get(x)`：x 是 fa[x] 的左 / 右儿子。

`isroot(x)`：x 是否是它所在 Splay 的根节点。

</br>

## 主要函数

### pushup & pushdown

`pushup(x)`：从 x 的子节点更新 x 的信息。

`pushdown(x)`：下放 x 的 tag。

---

### update

```cpp
void update(int x) {
  if (!isroot(x)) update(fa[x]);
  pushdown(x);
}
```

将 Splay 中 x 的所有祖先的 tag 从上往下下放。用于 Splay 操作之前。

---

### Rotate

```cpp
void Rotate(int x) {
  assert(fa[x]);
  int y = fa[x], z = fa[y], k = get(x), t = get(y);
  if (!isroot(y)) ch[z][t] = x;
  ch[y][k] = ch[x][!k], fa[ch[x][!k]] = y;
  ch[x][!k] = y, fa[y] = x;
  fa[x] = z;
  pushup(y), pushup(x);
}
```

和 Splay 的 Rotate 差不多，唯一的区别就是这句话

```cpp
if (!isroot(y)) ch[z][t] = x;
```

要写在 `fa[y]= x;` 前面。

--- 

### Splay

```cpp
void Splay(int x) {
  update(x);
  while (!isroot(x)) {
    if (!isroot(fa[x]) and get(fa[x]) == get(x)) Rotate(fa[x]);
    Rotate(x);
  }
}
```

就是普通的 Splay。记得要先 `update(x)`。

---

### Access

```cpp
int Access(int x) {
  int p = 0;
  while (x) Splay(x), ch[x][1] = p, pushup(x), p = x, x = fa[x];
  return p;
}
```

将 x 到原树上根节点的这条链放到同一棵 Splay 里。

p 表示 x 最后一次经过的 Splay 的根节点，也就是原树的根节点所在的 Splay 的根节点。

要注意 `pushup(x)`。

---

### MakeRoot

```cpp
void MakeRoot(int x) {
  x = Access(x);
  swap(ch[x][0], ch[x][1]), tag[x] ^= 1;
}
```

将 x 变为原树的根。附加效果是使得 x 和它在原树中的子节点不在同一棵 Splay 中。

先将 x 和根放到一棵 Splay 中，然后将 x 变为原树的根就相当于将这条链上的点按照深度从深到浅重新排序，那么也就相当于把这棵 Splay 中的点交换左右儿子。

---

### Split

```cpp
void Split(int x, int y) {
  MakeRoot(x), Access(y), Splay(y);
}
```

将原树上 x 到 y 的链单独抠出来。

具体操作就是先将 x 变为原树的根，然后把 x 到 y 的链单独放到一棵 Splay 上。

最后是否要 `Splay(y)` 因具体情况而定。

---

### Find

```cpp
int Find(int x) {
  x = Access(x);
  while (ch[x][0]) pushdown(x), x = ch[x][0];
  Splay(x);
  return x;
}
```

找 x 所在原树的根节点 rt。可用于判断连通性。

先把 x 和 rt 放到同一棵 Splay 上，然后从这棵 Splay 的根节点开始一直往左跳，找到的最浅的节点即为 rt。

最后记得 `Splay(x)` 以保证 Splay 的复杂度。

---

### Link

```cpp
void Link(int x, int y) {
  if (Find(x) == Find(y)) return;
  MakeRoot(x), Splay(x), fa[x] = y;
}
```

连接边 (x,y)。

先判断连通性。

然后将 x 变为原树的根，接着将它变为该 Splay 中的根，然后单向连接。

---

### Cut

```cpp
void Cut(int x, int y) {

  MakeRoot(x), Access(y), Splay(y);
  if (ch[y][0] != x or ch[x][1]) return;
  fa[x] = ch[y][0] = 0;
  pushup(y);
}
```

断开边 (x,y)。

先判断是否有 (x,y) 这条边。第一行三个函数相当于 `Split(x,y)`，将 (x,y) 这条链抠出来后，若存在比 x 深且比 y 浅的点，说明不存在 (x,y) 这条边。

然后 `ch[y][0] != x` 这句话顺便把不连通的情况给判掉了。

若存在 (x,y) 这条边，则双向断开。

记得 `pushup(y)`。

</br>

## 代码
[[luoguP3690]【模板】Link Cut Tree （动态树）](https://www.luogu.com.cn/problem/P3690)

```cpp
#include <cassert>
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 1e5 + 7;

struct LCT {
#define get(x) (x == ch[fa[x]][1])
#define isroot(x) (x != ch[fa[x]][0] and x != ch[fa[x]][1])

  int fa[_], ch[_][2], val[_], sum[_];
  bool tag[_];

  void pushup(int x) { sum[x] = sum[ch[x][0]] ^ sum[ch[x][1]] ^ val[x]; }

  void pushdown(int x) {
    if (tag[x]) {
      if (ch[x][0]) swap(ch[ch[x][0]][0], ch[ch[x][0]][1]), tag[ch[x][0]] ^= 1;
      if (ch[x][1]) swap(ch[ch[x][1]][0], ch[ch[x][1]][1]), tag[ch[x][1]] ^= 1;
      tag[x] = 0;
    }
  }

  void update(int x) {
    if (!isroot(x)) update(fa[x]);
    pushdown(x);
  }

  void Rotate(int x) {
    assert(fa[x]);
    int y = fa[x], z = fa[y], k = get(x), t = get(y);
    if (!isroot(y)) ch[z][t] = x;
    ch[y][k] = ch[x][!k], fa[ch[x][!k]] = y;
    ch[x][!k] = y, fa[y] = x;
    fa[x] = z;
    pushup(y), pushup(x);
  }

  void Splay(int x) {
    update(x);
    while (!isroot(x)) {
      if (!isroot(fa[x]) and get(fa[x]) == get(x)) Rotate(fa[x]);
      Rotate(x);
    }
  }

  int Access(int x) {
    int p = 0;
    while (x) Splay(x), ch[x][1] = p, pushup(x), p = x, x = fa[x];
    assert(!fa[p]);
    return p;
  }

  void MakeRoot(int x) {
    x = Access(x);
    assert(x), assert(!fa[x]);
    swap(ch[x][0], ch[x][1]), tag[x] ^= 1;
  }
    
  void Split(int x, int y) {
    MakeRoot(x), Access(y), Splay(y);
  }

  int Find(int x) {
    x = Access(x);
    while (ch[x][0]) pushdown(x), x = ch[x][0];
    Splay(x);
    return x;
  }

  void Link(int x, int y) {
    if (Find(x) == Find(y)) return;
    MakeRoot(x), Splay(x), fa[x] = y;
  }
  
  void Cut(int x, int y) {
    MakeRoot(x), Access(y), Splay(y);
    if (ch[y][0] != x or ch[x][1]) return;
    fa[x] = ch[y][0] = 0;
    pushup(y);
  }

  void Modify(int x, int w) {
    Splay(x), val[x] = w, pushup(x);
  }

  int Query(int x, int y) {
    MakeRoot(x);
    y = Access(y);
    return sum[y];
  }
} S;

int n, Q; 

int main() {
  cin >> n >> Q;
  for (int i = 1; i <= n; ++i) scanf("%d", &S.val[i]), S.sum[i] = S.val[i];
  for (int i = 1, ty, x, y; i <= Q; ++i) {
    scanf("%d%d%d", &ty, &x, &y);
    if (ty == 0) printf("%d\n", S.Query(x, y));
    if (ty == 1) S.Link(x, y);
    if (ty == 2) S.Cut(x, y);
    if (ty == 3) S.Modify(x, y);
  }
  return 0;
}
```

