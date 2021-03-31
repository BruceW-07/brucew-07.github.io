# [学习笔记]Splay


## 简介

Splay 维护平衡的方式就是每访问到一个节点, 都把它旋转到根. 这个把节点 x 旋转到根的过程就叫做 Splay.

</br>

## 维护变量

```cpp
int rt, fa[_], ch[_][2], val[_], num[_], sum[_];
```

`rt`: 根节点.

`fa[u]`: 节点 u 的父节点.

`ch[u][0/1]`: 节点 u 的左右儿子.

`val[u]`: 节点 u 的权值.

`num[u]`: 权值为 `val[u]` 的元素个数.

`sum[u]`: 节点 u 的子树 `num` 之和.


</br>

## 主要操作

### Update
```cpp
void upd(int x) { sum[x] = sum[ch[x][0]] + sum[ch[x][1]] + num[x]; }
```


### Rotate

和 Treap 的 Rotate 没啥区别. 

```cpp
void Rotate(int x) {
  int y = fa[x], t = get(x);
  ch[y][t] = ch[x][!t], fa[ch[x][!t]] = ch[x][!t] ? y : 0;    // modify ch[x][!t]
  fa[x] = fa[y], ch[fa[x]][get(y)] = fa[x] ? x : 0;    // modify x
  ch[x][!t] = y, fa[y] = x;    // modify y
  upd(y), upd(x);
}

```

---

### Splay

设 `get(u) = 0/1` 表示 u 是 fa[u] 的左/右儿子.

为了保证平衡, 若有 `get(u) == get(fa[u])`, 则要先 Rotate(fa[u]) 后再 Rotate(u).

```cpp
void Splay(int x) {
  while (fa[x]) {
    if (fa[fa[x]] and get(fa[x]) == get(x)) Rotate(fa[x]);
    Rotate(x);
  }
  rt = x;
}
```

---

### Insert

一直往下找, 若有 `val[u] == w`, 则 `++num[u]`; 若到了空节点, 则新建节点. 最后 Splay.

```cpp
void Ins(int &u, int f, int w) {
  if (!u) { u = ++tot, fa[u] = f, val[u] = w, num[u] = sum[u] = 1; Splay(u); return; }
  if (val[u] == w) { ++num[u], ++sum[u]; Splay(u); return; }
  Ins(ch[u][val[u] < w], u, w);
}

void Ins(int w) { Ins(rt, 0, w); }
```

多写一个是为了方便调用.

---

### Merge
设合并的两棵树为 A, B, 则需满足 A 的最大值小于 B 的最小值.

若有一棵树为空, 则将另外一棵树的根节点设为根.

否则将 A 中的最大值 Splay 到根, 然后将 B 设为 A 的右子树. (记得更新相关信息).

```cpp
void Merge(int x, int y) {
  if (!x) { rt = y, fa[y] = 0; return; }    //**
  rt = x, fa[x] = 0;
  while (ch[x][1]) x = ch[x][1];
  Splay(x);
  ch[rt][1] = y, fa[y] = rt, upd(rt);    //**
}
```

(`//**` 是容易写错的地方.)

---

### Delete

若有 `num[u] > 1`, 则 `--num[u]`.

否则将 `Splay(u)`, 然后合并 u 的两棵子树.

```cpp
void Del(int w) {
  Find(rt, w); 
  if (num[rt] > 1) --num[rt];
  else Merge(ch[rt][0], ch[rt][1]);
}
```

### 其他

找排名, 找第 k 小 / 大, 找前驱, 找后继. 都和 Treap 差不多.

</br>

## 代码
```cpp
#include <cstdio>
#include <iostream>

using namespace std;

const int _ = 1e5 + 7;

struct SPLAY {
#define get(x) (x == ch[fa[x]][1])

  int rt, fa[_], ch[_][2], num[_], sum[_], val[_], tot;

  void upd(int x) { sum[x] = sum[ch[x][0]] + sum[ch[x][1]] + num[x]; }

  void Rotate(int x) {
    int y = fa[x], t = get(x);
    ch[y][t] = ch[x][!t], fa[ch[x][!t]] = ch[x][!t] ? y : 0;    // modify ch[x][!t]
    fa[x] = fa[y], ch[fa[x]][get(y)] = fa[x] ? x : 0;    // modify x
    ch[x][!t] = y, fa[y] = x;    // modify y
    upd(y), upd(x);
  }

  void Splay(int x) {
    while (fa[x]) {
      if (fa[fa[x]] and get(fa[x]) == get(x)) Rotate(fa[x]);
      Rotate(x);
    }
    rt = x;
  }

  void Ins(int &u, int f, int w) {
    if (!u) { u = ++tot, fa[u] = f, val[u] = w, num[u] = sum[u] = 1; Splay(u); return; }
    if (val[u] == w) { ++num[u], ++sum[u]; Splay(u); return; }
    Ins(ch[u][val[u] < w], u, w);
  }

  void Ins(int w) { Ins(rt, 0, w); }

  void Find(int u, int w) {
    if (val[u] == w) { Splay(u); return; }
    Find(ch[u][val[u] < w], w);
  }

  void Merge(int x, int y) {
    if (!x) { rt = y, fa[y] = 0; return; }    //**
    rt = x, fa[x] = 0;
    while (ch[x][1]) x = ch[x][1];
    Splay(x);
    ch[rt][1] = y, fa[y] = rt, upd(rt);    //**
  }

  void Del(int w) {
    Find(rt, w); 
    if (num[rt] > 1) --num[rt];
    else Merge(ch[rt][0], ch[rt][1]);
  }
    
  int rk(int w) {
    int u = rt, res = 0;
    while (u) {
      if (val[u] == w) return res + sum[ch[u][0]] + 1;
      else if (val[u] < w) res += sum[ch[u][0]] + num[u], u = ch[u][1];
      else u = ch[u][0];
    }
    return res + 1;
  }

  int kth(int res) {
    int u = rt;
    while (res) {
      if (sum[ch[u][0]] < res and sum[ch[u][0]] + num[u] >= res) return val[u];
      else if (res <= sum[ch[u][0]]) u = ch[u][0];
      else res -= sum[ch[u][0]] + num[u], u = ch[u][1];
    }
    return val[u];
  }

  int pre(int w) {
    Ins(w);
    int u = ch[rt][0];
    while (ch[u][1]) u = ch[u][1];
    Del(w);    //**
    return val[u];
  }

  int suf(int w) {
    Ins(w);
    int u = ch[rt][1];
    while (ch[u][0]) u = ch[u][0];
    Del(w);    //**
    return val[u];
  }
} S;

int n;

int main() {
  cin >> n;
  for (int i = 1, ty, x; i <= n; ++i) {
    scanf("%d%d", &ty, &x);
    if (ty == 1) S.Ins(x);
    if (ty == 2) S.Del(x);
    if (ty == 3) printf("%d\n", S.rk(x));
    if (ty == 4) printf("%d\n", S.kth(x));
    if (ty == 5) printf("%d\n", S.pre(x));
    if (ty == 6) printf("%d\n", S.suf(x));
  }
  return 0;
}
```

