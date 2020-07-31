---
title: Task Supercomputer 题解
date: 2020-07-31 11:08:20
tags: 
- 动态规划
- 斜率优化
categories:
- [动态规划，斜率优化]
mathjax: true
---

好难啊qwq，蒟蒻完全不会斜率优化。

# 题目大意

[Link](https://www.luogu.com.cn/problem/P3571)

[原题面](https://szkopul.edu.pl/problemset/problem/e9ycK_efBDBt4aPs-QeqYpwR/site/?key=statement#)

# 题解

$n, q <= 10^6$ 提示离线，我们肯定无法做到对于每个 $k$ 去 $O(1)$ 在线询问。

<!-- ## 引理1： 对于前 $i$ 层我们最少需要 $i$ 次

首先证明不可能少于 $i$ 次，这个很显然，每次必须先要选父亲。

然后证明 $i$ 次一定可行， -->

最优方案肯定是先用 $x$ 步解决前 $x$ 层，然后剩下部分每次能选满 $k$ 个。

显然无法用少于 $x$ 步解决前 $x$ 层因为每次要先选择父亲节点。

那为什么 $x$ 步一定可以选完呢？

[看这里](https://www.luogu.com.cn/blog/league/solution-p3571)

然后设计状态和转移：
- $dp_i$ 表示当 $k = i$ 时最小需要的次数，$s_i$ 表示深度大于 $i$ 的节点数量；
- $dp_i = \max\{j + \lceil \frac{s_j}{i}\rceil\}$

至于为什么是 $\max$ 不是 $\min$，在所有的 $j$ 中充斥着大量不可行的方案，只有一种是合法的，因为我们要保证后面的每次都取满 $i$，这样的 $i$ 显然只有一个。取另外的会导致偏小。

形式化来讲，就是对于 $j, k, j > k$，可以做到用 $j$ 次访问完前 $j$ 层和用 $k$ 次访问完前 $k$ 层。那么：如果我们无法做到用 $j - k$ 次完成 $j$ 到 $k$ 层的访问，那么：

$\lceil \frac{s_k - s_j}{i}\rceil > j - k$

随便搞搞得

$k + \lceil \frac{s_k}{i}\rceil > j + \lceil \frac{s_j}{i} \rceil$

我们要的是 $k$，所以用 $\max$。

直接转移是 $O(n^2)$ 的，

注意到转移方程里有 $i$ 和 $j$，想到斜率优化。

$dp_i = \max\{j + \lceil \frac{s_j}{i}\rceil\}$

对于 $j$ 和 $k$，有：

$\begin{aligned}
j + \lceil \frac{s_j}{i}\rceil &> k + \lceil \frac{s_k}{i} \rceil\\
ij + s_j &> ik + s_k\\
i(j - k) &> s_k - s_j\\
i &< \frac{s_j - s_k}{j - k}
\end{aligned}$

然后就可以斜率优化了。

```cpp
double Slope(int x, int y) {
  return 1.0 * (s[x] - s[y]) / (s - y);
}
```

完整代码：

```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <deque>
#include <vector>

const int N = 1e6 + 10;

int que[N];
int dp[N];
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

int dep[N], cnt[N], s[N];

void Dfs(int u, int fa) {
  dep[u] = dep[fa] + 1;
  for (int i = head[u]; i; i = e[i].nxt) {
    int v = e[i].v;
    if (v == fa)
      continue;
    Dfs(v, u);
  }
}

double Slope(int x, int y) {
  return 1.0 * (s[x] - s[y]) / (x - y);
}

int main() {
  int n, m;
  scanf("%d%d", &n, &m);
  int maxque = 0;
  for (int i = 1; i <= m; ++i)
    scanf("%d", &que[i]), maxque = std::max(maxque, que[i]);
  for (int i = 2, u, v; i <= n; ++i) {
    scanf("%d", &u), v = i;
    AddEdge(u, v), AddEdge(v, u);
  }
  Dfs(1, 0);
  int maxdep = 0;
  for (int i = 1; i <= n; ++i)
    ++cnt[dep[i]], maxdep = std::max(maxdep, dep[i]);
  for (int i = maxdep; i >= 0; --i)
    s[i] = s[i + 1] + cnt[i + 1];
  std::vector<int> q;
  q.resize(N);
  int hd = 0, tl = 0;
  for (int i = 1; i <= maxdep; ++i) {
    while (hd < tl && Slope(q[tl - 1], q[tl]) <= Slope(q[tl], i))
      --tl;
    q[++tl] = i;
  }
  for (int i = 1; i <= maxque; ++i) {
    while (hd < tl && i * q[hd] + s[q[hd]] <= i * q[hd + 1] + s[q[hd + 1]])
      ++hd;
    dp[i] = q[hd] + ceil(1.0 * s[q[hd]] / i);
  }
  for (int i = 1; i <= m; ++i)
    printf("%d ", dp[que[i]]);
  printf("\n");
  return 0;
}
```

不过有两个玄学问题qwq

一个是为什么这一题 Slope 那里返回值为 int 也能过，还有一个是为什么我写的 deque 挂了。

```cpp
std::deque<int> q;
for (int i = 1; i <= maxdep; ++i) {
  while (q.size() >= 2 &&
         Slope(q[q.size() - 2], q[q.size() - 1] <= Slope(q[q.size() - 1], i)))
    q.pop_back();
  q.push_back(i);
}
for (int i = 1; i <= maxque; ++i) {
  while (q.size() >= 2 && i * q[0] + s[q[0]] <= i * q[1] + s[q[1]])
  q.pop_front();
  dp[i] = q[0] + ceil(1.0 * s[q[0]] / i);
}
```
