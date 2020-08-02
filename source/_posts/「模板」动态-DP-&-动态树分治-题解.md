---
title: 【模板】动态 DP & 动态树分治 题解
date: 2020-08-02 14:56:40
tags:
- 动态规划
- 动态动态规划
- 树链剖分
- 线段树
- 矩阵乘法
- 广义矩阵乘法
categories:
- [动态规划, 动态动态规划]
- [动态规划, 树形动态规划]
- [数据结构, 树链剖分]
- [数据结构, 线段树]
- [矩阵乘法, 广义矩阵乘法]
mathjax: true
---

蒟蒻太菜了，今天才学会 DDP。

# 题目描述

[Link](https://www.luogu.com.cn/problem/solution/P4719)

<!--more-->

# 题解

如果没有修改操作，那么就是一个入门组水平的树形 DP，设 $dp_{u, 0/1}$ 表示当 $u$ 选或不选时的最小花费。

然后考虑修改操作，naive 的想法是每次修改后暴力跑一遍树形 DP，这样的时间复杂度是 $O(nm)$ 的。

但是在每次修改的过程中很多值是不变的，会改变的值只有它到父亲的一条链上的点。

那就变成了一个链上的问题，考虑树剖。

那么原来的转移方程：

$dp_{u, 0} = \sum\limits_{v \in son(u)} \max\{dp_{v, 0}, dp_{v, 1}\}$

$dp_{u, 1} = \sum\limits_{v \in son(u)} dp_{v, 0} + a_u$

要对于轻重链分开讨论一下。

让 $ldp_{u, 0/1}$ 表示不考虑重儿子的情况，$dp_{u, 0/1}$ 表示考虑重儿子的情况。

为什么要分开讨论呢？

你树剖当然分开讨论不然要树剖干什么。

然后得到新的转移方程：

$ldp_{u, 0} = \sum\limits_{v \in son(u)} \max\{dp_{v, 0}, dp_{v, 1}\}[v \not = heavyson(u)]$

$ldp_{u, 1} = dp_{u, 1} = \sum\limits_{v \in son(u)} dp_{v, 0}[v \not = heavyson(u)] + a_u$

$dp$ 的转移和上面是一样的。

树上问题转换成了链上问题，怎么维护相邻的值？

很自然想到线段树（动态 DP 常用方法），那我们要维护一个有结合律的东西。

回到上面的转移方程，想想有什么长得像，DP 转移还可以用什么方式表示。

下面这个矩阵运算顺序和上面的方程是一样的，但是我们所用的并不是一般的矩阵乘法那样的运算，而是取 $\max$。

$$
\begin{bmatrix}
  dp_{v, 0} \\
  dp_{v, 1}
\end{bmatrix}
\times
\begin{bmatrix}
  ldp_{u, 0} & ldp_{u, 0} \\
  ldp_{u, 1} & -\infty
\end{bmatrix}
=
\begin{bmatrix}
  dp_{u, 0} \\
  dp_{u, 1}
\end{bmatrix}
$$

那么我们重新定义一下矩阵乘法，令 $C_{i, j} = \max\{a_{i, k} + b_{k, j}\}$。

然后展开就和上面的式子一样了。

所以我们用线段树维护上面那样的矩阵，但我好像不太会构造这东西的单位矩阵qwq，所以改了下线段树的写法。

对于一次修改操作，我们从需要修改的节点 $u$ 开始一路向上跳，对于一整条重链我们直接在线段树上统计答案，如果是轻链则暴力乘上当前节点的信息。

因为从点 $u$ 开始到根的重链不超过 $\log n$ 条，轻链长度为 $1$，所以时间复杂度是 $O(n \log^2 n)$。

下次试试看 LCT 来写。

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>

const int N = 1e5 + 10;
const int INF = 0x3f3f3f3f;

int n, m;
int a[N];
struct Edge {
  int v, nxt;

  Edge() : v(0), nxt(0) {}
  Edge(int _v, int _nxt) : v(_v), nxt(_nxt) {}
} e[N << 1];
int head[N], edge_cnt;

void AddEdge(int u, int v) {
  e[++edge_cnt] = Edge(v, head[u]);
  head[u] = edge_cnt;
}

// Heavy Light

int sze[N], son[N], in[N], ou[N], idx, tp[N], fa[N], bel[N];

void Dfs1(int u, int fa) {
  ::fa[u] = fa, sze[u] = 1;
  for (int i = head[u]; i; i = e[i].nxt) {
    int v = e[i].v;
    if (v == fa)
      continue;
    Dfs1(v, u);
    sze[u] += sze[v];
    if (sze[v] > sze[son[u]])
      son[u] = v;
  }
}

void Dfs2(int u, int fa, int tp) {
  ::tp[u] = tp, in[u] = ++idx, ou[tp] = idx, bel[idx] = u;
  if (son[u])
    Dfs2(son[u], u, tp);
  for (int i = head[u]; i; i = e[i].nxt) {
    int v = e[i].v;
    if (v == fa || v == son[u])
      continue;
    Dfs2(v, u, v);
  }
}

struct Matrix {
  int a[2][2];

  Matrix() { memset(a, 0, sizeof(a)); }

  void Init() { memset(a, -0x3f, sizeof(a)); }

  Matrix operator*(const Matrix& rhs) {
    Matrix res;
    res.Init();
    for (int i = 0; i <= 1; ++i)
      for (int j = 0; j <= 1; ++j)
        for (int k = 0; k <= 1; ++k)
          res.a[i][j] = std::max(res.a[i][j], a[i][k] + rhs.a[k][j]);
    return res;
  }

  void Print() { printf("%d %d %d %d\n", a[0][0], a[0][1], a[1][0], a[1][1]); }
};

int dp[N][2], ldp[N][2];  // both, only light
Matrix mat[N];

struct SegmentTree {
#define lc (rt << 1)
#define rc (rt << 1 | 1)
#define ls lc, l, mid
#define rs rc, mid + 1, r

  Matrix t[N << 2];

  void PushUp(int rt) { t[rt] = t[lc] * t[rc]; }

  void Build(int rt, int l, int r) {
    if (l == r) {
      mat[bel[l]].a[0][0] = ldp[bel[l]][0],
      mat[bel[l]].a[0][1] = ldp[bel[l]][0],
      mat[bel[l]].a[1][0] = ldp[bel[l]][1], mat[bel[l]].a[1][1] = -INF;
      t[rt] = mat[bel[l]];
      // printf("%d: ", rt), t[rt].Print();
      return;
    }
    int mid = (l + r) >> 1;
    Build(ls), Build(rs);
    PushUp(rt);
  }

  void Update(int rt, int l, int r, int p) {
    if (l == r)
      return t[rt] = mat[bel[l]], void();
    int mid = (l + r) >> 1;
    if (p <= mid)
      Update(ls, p);
    else
      Update(rs, p);
    PushUp(rt);
  }

  Matrix Query(int rt, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr)
      return t[rt];
    // printf("%d %d %d %d %d\n", rt, l, r, ql, qr);
    int mid = (l + r) >> 1;
    if (qr <= mid)
      return Query(ls, ql, qr);
    if (ql > mid)
      return Query(rs, ql, qr);
    return Query(ls, ql, qr) * Query(rs, ql, qr);
  }
} st;

void Dfs3(int u) {
  ldp[u][1] = a[u];
  for (int i = head[u]; i; i = e[i].nxt) {
    int v = e[i].v;
    if (v == fa[u] || v == son[u])
      continue;
    Dfs3(v);
    ldp[u][0] += std::max(dp[v][0], dp[v][1]);
    ldp[u][1] += dp[v][0];
  }
  dp[u][0] = ldp[u][0], dp[u][1] = ldp[u][1];
  if (son[u]) {
    Dfs3(son[u]);
    dp[u][0] += std::max(dp[son[u]][0], dp[son[u]][1]);
    dp[u][1] += dp[son[u]][0];
  }
}

void Update(int x, int k) {
  mat[x].a[1][0] += k - a[x];
  a[x] = k;
  while (x) {
    int _x = x;
    x = tp[x];
    Matrix pre = st.Query(1, 1, n, in[x], ou[x]);
    // printf("pre: "), pre.Print();
    st.Update(1, 1, n, in[_x]);
    Matrix now = st.Query(1, 1, n, in[x], ou[x]);
    // printf("now: "), now.Print();
    x = fa[x];
    mat[x].a[0][0] +=
        std::max(now.a[0][0], now.a[1][0]) - std::max(pre.a[0][0], pre.a[1][0]);
    mat[x].a[0][1] = mat[x].a[0][0];
    mat[x].a[1][0] += now.a[0][0] - pre.a[0][0];
    // printf("mat[%d]: ", x), mat[x].Print();
  }
}

int Query() {
  Matrix res = st.Query(1, 1, n, in[1], ou[1]);
  // res.Print();
  return std::max(res.a[0][0], res.a[1][0]);
}

int main() {
  scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; ++i)
    scanf("%d", &a[i]);
  for (int i = 2, u, v; i <= n; ++i) {
    scanf("%d%d", &u, &v);
    AddEdge(u, v), AddEdge(v, u);
  }
  Dfs1(1, 0), Dfs2(1, 0, 1), Dfs3(1);
  st.Build(1, 1, n);
  while (m--) {
    int x, y;
    scanf("%d%d", &x, &y);
    Update(x, y);
    printf("%d\n", Query());
  }
  return 0;
}
```
