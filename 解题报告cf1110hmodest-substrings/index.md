# [解题报告][CF1110H]Modest Substrings


## Statement

[传送门](https://www.luogu.com.cn/problem/CF1110H)

定义一个字符集为 $\{0, 1, 2, \cdots, 9\}$ 的字符串 $S$ 的价值为：$S$ 的所有无前缀0的且所代表的数字属于区间 $[L, R]$ 内的子串 $T$ 的个数。

给定 $S$ 的长度 $n$ 以及区间 $[L, R]$，求价值最大且字典序最小的字符串 $S$。

$1 \le L \le R \le 10^{800}, 1 \le n \ le 2000$。

## Solution

首先考虑若 $R - L$ 很小的时候怎么做。

我们可以把 $[L, R]$ 之间的所有数字插入 Trie 树中，并建立 AC 自动机。

然后设 DP 状态 $f[i][u]$ 表示当前填到 $S$ 的第 $i$ 位，且在 AC 自动机上匹配到节点 $u$ 时的最大价值。

对于构造方案，我们可以对每个 DP 状态找它的前缀。

具体来说，设 $maxf[i][u]$ 表示从 $f[i][u]$ 状态开始转移，最终能够得到的最大值。

然后我们从前往后构造，假设当前的状态为 $(i, u)$，那么我们取最小的满足 $f[i + 1][ch[u][c]]$ 是从 $f[i, u]$ 转移过去的 且 $maxf[i + 1][ch[u][c]] = \mathrm{ans}$ 且 的 $c$ 进行转移即可。

---

现在进一步考虑 $R - L$ 更大的时候怎么做。

我们会发现，当 $[L, R]$ 中的数很多时，Trie 树上会出现很多子树是“满十叉树”的节点。

对于这些节点，只要从它开始往下走一定距离，就一定会得到一些贡献，与走的方向（也就是在 $S$ 中所填的数）无关。

那么我们可以考虑在一到达这些点的时候就先把在它子树中能得到的贡献算进答案中，然后就不需要遍历它的子树了。这样就减少了 Trie 树中的节点数。

于是我们设 $g[u][j]$ 表示从 Trie 树上的节点 $u$ 开始，走 $j$ 步后一定能得到的价值。

现在我们考虑该怎么设这个 $g$ 数组。

我们按照「 $L$ 和 $R$ 的位数是否相同」进行分类讨论。

---

### First. $L$ 和 $R$ 位数不同

这时，对于位数在 $[|L| + 1, |R| - 1]$ 中的所有数，只要没有前导0并且走了 $j$ 步后，就一定能得到一个位数为 $j$ 的数。

那么，我们对节点 1（也就是根节点）的所有0儿子 $v = ch[1][1 \sim 9]$，将 $g[v][|L| \sim |R| - 2]$ 设为 1。（因为从根节点到 $v$ 就已经填了一位了，所以剩下就只用填 $|L| \sim |R| - 2$ 位。）

然后对于位数为 $|L|$ 的所有数，只要它的字典序大于等于 $L$，则它可以贡献到答案中。

在 Trie 树上插入 $L$ 的过程中，我们设当前插入到 $L_i = c$，且到达 Trie 树上的节点 $u$，那么我们将 $g[ch[u][c + 1 \sim 9]][|L| - i]$ 设为 1。

然后设插入 $L$ 后最终到达的节点为 $x$，将 $g[x][0]$ 设为 1。

对于位数为 $|R|$ 的所有数，做类似操作就行了。

### Second. $L$ 和 $R$ 的位数相同

我们设 $i$ 为 $L, R$ 第一次不相等的位置，设它在 Trie 树上对应节点 $u$，将 $g[ch[u][L[i] + 1 \sim R[i] - 1]][|L| - i]$ 设为 1。

剩下的位置与第一种情况中位数为 $|L|$ 的数类似。

---

处理完 $g$ 数组后，在找 $fail$ 的时候把 $g[fail[u]]$ 也加到 $g[u]$ 上。

$f$ 的转移即为 $f[i][u] + \sum_{j = 0}^{n - i - 1} g[ch[u][c]][j] \rightarrow f[i + 1][ch[u][c]]$。可以前缀和优化一下。

时间复杂度为 $O(10|R|n) \approx 1.6 \times 10^7$。

## Code
```cpp
#include <cassert>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <queue>

using namespace std;

const int _ = 2e3 + 7;
const int __ = 2e4 + 7;

typedef long long ll;

int n, l1, l2, ch[__][10], fail[__], tot = 1;
ll f[_][__], maxf[_][__], g[__][_];
char L[_], R[_], S[_];
queue<int> q;

void Ins(char S[], int ty) {
  int u = 1, len = strlen(S + 1); bool flag = l1 == l2;
  for (int i = 1; i <= len; ++i) {
    int c = S[i] - '0';
    if (!ch[u][c]) ch[u][c] = ++tot;
    if (!ty) {
      int lim = flag ? R[i] - '0' - 1 : 9;
      for (int j = c + 1; j <= lim; ++j) {
        if (!ch[u][j]) ch[u][j] = ++tot;
        ++g[ch[u][j]][len - i];
      }
    }
    else {
      int lim = flag ? 9 : 0;
      for (int j = c - 1; j >= lim; --j) {
        if (!j and i == 1) continue;
        if (!ch[u][j]) ch[u][j] = ++tot;
        ++g[ch[u][j]][len - i];
      }
    }
    u = ch[u][c];
    if (L[i] != R[i]) flag = 0;
  }
  if (!ty or !flag) ++g[u][0];
}

void GetFail() {
  for (int i = 0; i < 10; ++i) ch[0][i] = 1;
  q.push(1);
  while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int i = 0; i < 10; ++i)
      if (ch[u][i]) fail[ch[u][i]] = ch[fail[u]][i], q.push(ch[u][i]);
      else ch[u][i] = ch[fail[u]][i];
    for (int i = 0; i <= n; ++i) g[u][i] += g[fail[u]][i];
  }
  for (int u = 1; u <= tot; ++u)
    for (int i = 1; i <= n; ++i) g[u][i] += g[u][i - 1];
}

int main() {
  scanf("%s%s%d", L + 1, R + 1, &n);
  l1 = strlen(L + 1), l2 = strlen(R + 1);
  Ins(L, 0), Ins(R, 1);
  for (int i = 1; i < 10; ++i) {
    if (!ch[1][i]) ch[1][i] = ++tot;
    for (int j = l1; j < l2 - 1; ++j) ++g[ch[1][i]][j];
  }
  GetFail();

  memset(f, -0x3f, sizeof f);
  f[0][1] = 0;
  for (int i = 0; i < n; ++i) {
    for (int j = 1; j <= tot; ++j) {
      if (f[i][j] < 0) continue;
      for (int k = 0; k < 10; ++k)
        f[i + 1][ch[j][k]] = max(f[i + 1][ch[j][k]], f[i][j] + g[ch[j][k]][n - i - 1]);
    }
  }

  ll ans = 0;
  for (int j = 1; j <= tot; ++j) ans = max(ans, f[n][j]), maxf[n][j] = f[n][j];

  for (int i = n - 1; ~i; --i)
    for (int j = 1; j <= tot; ++j)
      for (int k = 0; k < 10; ++k)
        if (f[i + 1][ch[j][k]] == f[i][j] + g[ch[j][k]][n - i - 1])
          maxf[i][j] = max(maxf[i][j], maxf[i + 1][ch[j][k]]);

  assert(maxf[0][1] == ans);
  int u = 1;
  for (int i = 0; i < n; ++i)
    for (int k = 0; k < 10; ++k)
      if (maxf[i + 1][ch[u][k]] == ans and f[i + 1][ch[u][k]] == f[i][u] + g[ch[u][k]][n - i - 1]) {
        S[i + 1] = k + '0', u = ch[u][k];
        break;
      }

  printf("%lld\n%s\n", ans, S + 1);
  return 0;
}
```

