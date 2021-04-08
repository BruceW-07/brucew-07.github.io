# [学习笔记]博弈论


## Nim 游戏

#### 定义

有 $n$ 堆石子, 每堆有 $a_i$ 个石子, 每次从一堆中取任意个石子, 无法操作者败

#### 结论

先手必胜的条件:  $\bigoplus_{i = 1}^n a_i \not = 0$, 

---
</br>

## SG 函数

#### 定义

设一个局面为 $A$, 它的后继状态集合为 $E_A$, 则

$$
SG(A) = 
\begin{cases}
0, &E_A = \emptyset  \\\\ 
mex(E_A) &E_A \not = \emptyset
\end{cases}
$$

其中 $mex(E_A)$ 表示集合 $E_A$ 中没有出现的最小非负整数.

#### 性质

设两个相互独立的游戏为 $A,B$, 游戏 $A+B$ 为每次在 $A$ **或** $B$ 中进行一次操作的游戏, 则
$$
SG(A+B) = SG(A) \oplus SG(B)
$$
其中 $\oplus$ 表示二进制异或.

##### 扩展

$$
SG(A_1 + A_2 + \cdots + A_k) = SG(A_1) \oplus SG(A_2) \oplus \cdots \oplus SG(A_k)
$$

---
</br>

## Anti 游戏

#### 定义 

无法操作的一方获胜

---

### Anti-Nim

#### 结论

先手必胜的条件:

1. 石子的异或和 $\not = 0$, 且存在一堆石子个数 $> 1$.
2. 石子的异或和 $=0$, 且所有石子个数 $\le 1$.

---

### Anti 游戏

#### 结论

设 $A = A_1 + A_2 + \cdots + A_k$, 则先手必胜情况为

1. $SG(A) \not = 0$, 且 $\exists\ i, SG(A_i) > 1$.

2. $SG(A) = 0$, 且 $\forall\ i, SG(A_i) \le 1$.

**成立条件**: 在 $\forall\ i, SG(A_i) = 0$ 时, 满足其下的任意一条

1. $\forall\ i$, $A_i$ 没有出边 (直接结束游戏).
2. $\exists\ i$, $A_i$ 能转移到 $SG = 1$ 的情况 (转移到一个必败态).

---
</br>

## Bash 博弈

#### 定义

 同 Nim游戏, 但每次取的石子不能超过一个常数 $m$.

#### 结论

 $SG(x) = x \bmod{(m + 1)}$. ($x$ 为一堆中的石子个数).

#### 证明

 手玩打表即可

---
</br>

## 阶梯Nim游戏

#### 定义

$n$ 层阶梯上分别有 $1$ 堆石子, 每次可以选择一堆中的任意个石子扔到下一层 (特殊的, 第一层的石子直接被扔出游戏), 无法操作者败.

#### 结论

总游戏的 $SG$ 函数为奇数层 $SG$ 函数的异或和.

#### 证明

感性理解. 若两人都只操作奇数层的石子, 则是一个 Nim 游戏. 若有一个人把偶数层的石子移到奇数层来了,      那么另一个人可以把这些石子移回偶数层, 游戏结果不发生改变.

---
</br>

## 威佐夫博弈 Wythoff Game

#### 定义

有两堆石子, 每次可以从一堆中取任意个石子 或 从两堆中取出相同个数的石子, 不能操作者败

#### 结论

##### 结论一

把状态设为 $(x, y)$, 把所有 $x < y$ 的 $P$ 态按照 $x$ 从小到大排序设为 $(a_i, b_i)$.

打表可以发现

- $a_i$ 为 $a_{1 \sim i-1}, b_{1\sim i-1}$ 中未出现过的非负整数
- $b_i = a_i + i$

##### 结论二

如果令 $\alpha = \frac{1 + \sqrt5}{2}, \beta = \frac{3 + \sqrt5}{2}$, 则 $(a_i, b_i) = (\lfloor\alpha i\rfloor, \lfloor\beta i\rfloor)$.

#### 证明

##### 结论一

反证法

![](https://cdn.luogu.com.cn/upload/image_hosting/kj896bp4.png)

##### 结论二

 可用 Betty 定理推导

![](https://cdn.luogu.com.cn/upload/image_hosting/ah3cyu68.png)

---
</br>

## 翻硬币游戏

#### 定义

![](https://cdn.luogu.com.cn/upload/image_hosting/y293785h.png)

#### 结论

![](https://cdn.luogu.com.cn/upload/image_hosting/yqx6yijh.png)

#### 证明

![](https://cdn.luogu.com.cn/upload/image_hosting/06qjh76o.png)

---
</br>

## 删边游戏

### 树上删边游戏 一

#### 定义

一棵有根树, 每次删除一条边及其子树部分, 不能操作者败.

#### 结论

节点 $u$ 的 $SG$ 等于它所有子节点的 $SG +1$ 的异或和. (叶节点的 $SG = 0$)

#### 证明

感性理解. 考虑一条链的情况, 就相当于一个 Nim游戏, 转移到父亲节点时 $SG + 1$.

---

### 树上删边游戏 二

#### 定义

同 树上删边游戏 一, 但树上挂了若干个环 (即有若干个从树上一个节点出发并回到该节点的环).

#### 结论

长度为奇数的环 $SG = 1$; 长度为偶数的环 $SG = 0$.

#### 证明

长度为奇数的环, 删了一条边后, 剩下两条链奇偶相同, 所以其异或和不可能为奇数, 所以 $SG = 1$.

长度为偶数的环, 删了一条边后, 剩下两条链奇偶不同, 所以其异或和不可能为偶数, 所以 $SG = 0$.

---

###  无向图删边游戏

#### 定义

一个无向图, 每次删除一条边以及与根节点不连通的部分.

#### 结论 (Fusion Principle)

将偶环替换成一个新点, 奇环替换成一个新点连出一条边, 原图中连向环的边全部连向新点, 图的 $SG$ 值不会改变

#### 推广

对于一个边双连通分量, 其 $SG$ 只与其边数的奇偶性有关, 偶数则为 $0$, 奇数则为 $1$.

---
</br>

## 动态减法游戏

### 1-动态减法游戏

#### 定义

![](https://cdn.luogu.com.cn/upload/image_hosting/vbqfzpns.png)

#### 结论

![](https://cdn.luogu.com.cn/upload/image_hosting/tsq3sh6c.png)

---

### 2-动态减法游戏

#### 定义

![](https://cdn.luogu.com.cn/upload/image_hosting/c4ra7t3c.png)

#### 结论

![](https://cdn.luogu.com.cn/upload/image_hosting/ejh2cweo.png)

---

### k-动态减法游戏

#### 定义

![](https://cdn.luogu.com.cn/upload/image_hosting/7asiaihc.png)

#### 结论

![](https://cdn.luogu.com.cn/upload/image_hosting/vy38ifpy.png)

#### 证明

![](https://cdn.luogu.com.cn/upload/image_hosting/ez0dok88.png)

#### 扩展

![](https://cdn.luogu.com.cn/upload/image_hosting/sw1g1ql2.png)

##### 证明

![](https://cdn.luogu.com.cn/upload/image_hosting/ptgfo168.png)

---
</br>

## 匹配在博弈中的应用

#### 定义

一张无向图, 初始时有一个棋子在 $s$ 点, 轮流将棋子移向一个之前没有到达过的点, 无法操作者败.

#### 结论

先手必胜, 当且仅当 $s$ 在该图的所有最大匹配中 (即删去 $s$ 会使该图的最大匹配减小).

#### 证明

##### 充分性

假设一对匹配为 $(s, t)$, 先手将棋子移向 $t$, 若后手能够将棋子移向一个非匹配点 $x$, 则让 $x$ 与 $t$ 匹配不会减小该图匹配数, 即删去 $s$ 后不会使该图的最大匹配减小, 与条件矛盾.

所以后手要么无法操作, 要么只能将棋子移向另一个匹配点 $s'$, 而这种情况下先手一定能将棋子移到 $t'$, 所以先手必胜.

##### 必要性

反证法.

假设 $s$ 不在该图的所有最大匹配中, 那么考虑一个 $s$ 不为匹配点的最大匹配, 先手一定只能将棋子移向一个匹配点 $s'$, 而这种情况下后手一定能将棋子移向 $t'$, 所以后手必胜.

所以 $s$ 一定在该图的所有最大匹配中.

---
</br>

## Every-SG 游戏

#### 定义

有若干个独立的游戏, 每次要移动所有可以移动的棋子, 不能操作者败.

#### 解法

每个单独的游戏的结果已经确定了, 对最终结果有影响的就只有每个游戏结束的时间.

对于先手必胜的游戏, 我们希望它进行的时间可以尽量的长, 而对于先手必败的游戏, 我们希望它尽早结束, 这样我们才能确保最终的胜利.

所以计算出先手必胜的最长时间 和 先手必败的最短时间, 将两者进行比较即可.

---
</br>

## 题目

[CF794E Choosing Carrot](http://codeforces.com/problemset/problem/794/E) 手玩找规律

[[HEOI2014]人人尽说江南好](https://www.luogu.com.cn/problem/P4101) 结论: 最终堆数为 $\lceil \frac{n}{m} \rceil$ 堆. (扩展: Alice 最多能堆 A 个石子, Bob 最多能堆 B 个石子)

[[清华集训2016] Alice 和 Bob 又在玩游戏](https://www.luogu.com.cn/problem/P6665) 加速运算 SG 函数.

---

还没做的题目

[[ZJOI2009]染色游戏](https://www.luogu.com.cn/problem/P2594)

[[HAOI2015]数组游戏](https://www.luogu.com.cn/problem/P3179)

[[HNOI2014]江南乐](https://www.luogu.com.cn/problem/P3235)

[CF494E Sharti](http://codeforces.com/problemset/problem/494/E)

[CF138D World of Darkraft](http://codeforces.com/problemset/problem/138/D)

[[HDU3595]GG and MM](http://acm.hdu.edu.cn/showproblem.php?pid=3595) (Every-SG)

