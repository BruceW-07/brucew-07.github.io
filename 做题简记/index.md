# 做题简记


## 2/17
### [bzoj3728]Zarowki (模拟费用流)
[题面](https://darkbzoj.tk/problem/3728)

**时间**：16:10 ~ 17:00

**标签**：模拟费用流

**难度**：中下

[代码](https://darkbzoj.tk/submission/111179)

---

## 2/26
### [AHOI2009]最小割（网络流、Tarjan）

[题面](https://www.luogu.com.cn/problem/P4126)

**时间**：16:30 ~ 17:30

**标签**：网络流、Tarjan

**难度**：中下（模板）

[解题报告](https://brucew-07.github.io/%E8%A7%A3%E9%A2%98%E6%8A%A5%E5%91%8Aahoi2009%E6%9C%80%E5%B0%8F%E5%89%B2/)

---

## 2/27
### luoguP4311 士兵占领（网络流）

[题面](https://www.luogu.com.cn/problem/P4311)

**时间**：9:30 ~ 10:10

**标签**：网络流，逆向思维

**难度**：中下

[解题报告](https://brucew-07.github.io/%E8%A7%A3%E9%A2%98%E6%8A%A5%E5%91%8Aluogup4311%E5%A3%AB%E5%85%B5%E5%8D%A0%E9%A2%86/)

### [WC2007]剪刀石头布（费用流）

[题面](https://www.luogu.com.cn/problem/P4249)

**时间**：11:00 ~ 11:50 + 14:00 ~ 14:45

**标签**：费用流

**难度**：中

[解题报告](https://brucew-07.github.io/%E8%A7%A3%E9%A2%98%E6%8A%A5%E5%91%8Awc2007%E5%89%AA%E5%88%80%E7%9F%B3%E5%A4%B4%E5%B8%83/)

### [清华集训2017]无限之环（费用流）

[题面](https://www.luogu.com.cn/problem/P4003)

**时间**：15:10 ~ 17;30 + 19:00 ~ 20:30

**标签**：费用流、神仙建图

**难度**：中上

**备注**：这题总的思路不复杂，但是不看题解完全写不出来。

[解题报告](https://brucew-07.github.io/%E8%A7%A3%E9%A2%98%E6%8A%A5%E5%91%8A%E6%B8%85%E5%8D%8E%E9%9B%86%E8%AE%AD2017%E6%97%A0%E9%99%90%E4%B9%8B%E7%8E%AF/)

### CF708D Incorrect Flow（上下界最大流）

[题面](https://www.luogu.com.cn/problem/CF708D)

**时间**：21:30 ~ 22:30s

**标签**：上下界最大流

**难度**：中

**备注**：大概就是个上下界有源汇最大流板子，但做的时候脑子抽了没想出来怎么建图。

[代码](http://codeforces.com/contest/708/submission/108612352)

---

## 3/2

### 「LibreOJ NOI Round #2」签到游戏（GCD、线段树）

[题面](https://loj.ac/p/576)

**时间**：(3/1)22:30 ~ 22:50 + (3/2)14:00 ~ 15:15

**标签**：GCD、线段树

**难度**：中

**备注**：

利用「固定左端点（右端点），区间不同 GCD 个数为 $O(\log)$」这条性质，用线段树维护区间 GCD，然后每次暴力建出固定左端点（右端点）的 $O(\log)$ 个 GCD 不同的区间，然后二分，check 的时候暴力算前缀和即可。

二分 check 的时候要注意一下边界，考虑的时候要仔细讨论。

[代码](https://loj.ac/s/1080594)

### [USACO21JAN] Paint by Letters P

[题面](https://www.luogu.com.cn/problem/P7295)

**时间**：18:40 ~ 19:50

**标签**：平面图欧拉公式

**难度**：中下

**备注**：基本上是欧拉公式板子题，没见过的话就真的做不出来。

[解题报告]()

