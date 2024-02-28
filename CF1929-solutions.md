# Codeforces Round 926 (Div. 2) 题解

比赛链接：https://codeforces.com/contest/1929

官解链接：https://codeforces.com/blog/entry/125943

出的很差的一场。

**UPD1**：加入了 E 题使用或卷积的另解。

#### [推歌](https://music.163.com/#/song?id=2106232177)

### [CF1929A. Sasha and the Beautiful Array](https://codeforces.com/contest/1929/problem/A)

任意排列数组 $a_{1..n}$，求 $\sum_{i=2}^n (a_i - a_{i-1})$ 的最大值。

#### 题解

见过最显然的 A 题，奠定了本场题目简单的基调。

众所周知 $\sum_{i=2}^n (a_i - a_{i-1}) = a_n - a_1$，又因为可以任意排列，答案就是 $\max(a) - \min(a)$。

#### 代码实现

``` cpp
void solve() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int &x: a) cin >> x;
    cout << ranges::max(a) - ranges::min(a) << '\n';
}
```

### [CF1929B. Sasha and the Drawing](https://codeforces.com/contest/1929/problem/B)

在一个 $n \times n$ 的正方形网格上，最少要给多少个格子涂色，才能使至少有 $k$ 条对角线上至少有一个格子被染色？

#### 题解

垃圾场特有的猜猜题，但我一开始画图画错了，导致猜错了……

在一个最优方案中，涂色一个格子只会使染色对角线的数目增加 $1$ 或 $2$。且我们永远可以将染色两条对角线的操作提前（因为他不会和任何之前的操作产生冲突！）。那么问题就是如何连续进行尽可能多的染色两条对角线的操作。

我们有一种染色方案，能使得共 $2n$ 次操作中，只有最后两次操作只染色一条对角线，前 $2n-2$ 次操作如下图：

<div align=center><img src="https://img2024.cnblogs.com/blog/3082141/202402/3082141-20240216125128779-715460913.png" alt="image" style="zoom: 50%;" /></div>


最后两次操作分别染色右上角和右下角的两个小方格。

要证明这是最优方案，只要说明不可能使所有操作都染色两条对角线。很简单：左下角和右上角的两个小方格，它们所在的两条对角线均只能通过这一条小方格染色，也就是这两次涂色必然进行；但它们又在同一条反对角线上，因此两次涂色操作必定至少有一次只染色一条对角线。

若所有操作均染色两条对角线，需要的操作次数为 $\lceil \dfrac k 2\rceil$。当 $\lceil \dfrac k 2\rceil \le 2n-2$ 时，这就是答案；否则答案为 $2n-2 + k - 2 * (2n-2) = k + 2 - 2n$。

#### 代码实现

```cpp
void solve() {
    int n, k;
    cin >> n >> k;
    if (k <= 4 * n - 4) {
        cout << (k + 1) / 2 << '\n';
    } else {
        cout << k + 2 - 2 * n << '\n';
    }
}
```

### [CF1929F. Sasha and the Wedding Binary Search Tree](https://codeforces.com/contest/1929/problem/F)

给定一棵 BST，其中部分点的权值不定 $\in [1, C]$。求有多少填充方案。

#### 题解

非常简单的题，甚至完全可以放到 C 题之前……分数虚高是因为被放在最后，而很多人被 D 挡住了，根本没来得及看后两题。

BST 等价于中序遍历是非降序列。对每段连续的 $-1$，它可以取前后两个数（可以在首尾插入两个哨兵）范围内的任意非降序列。$n$ 个数的，取值范围长度为 $m$ 的非降序列的个数个数即为在 $n$ 个数内插入 $m-1$ 个隔板的方案数 $\binom{n + m-1}{n}$。不同段的方案是无关的，根据乘法原理将方案数相乘即可。

使用 $\binom {n}{m} = \dfrac {n(n-1)\cdots (n-m+1)}{m!}$，可以在 $O(m)$ 时间内暴力求组合数。由于连续 $-1$ 段长度的总和不超过 $n$，时间复杂度是正确的。

#### 代码实现

```cpp
void solve() {
    int n, m;
    cin >> n >> m;
    vector<int> l(n + 1), r(n + 1), val(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> l[i] >> r[i] >> val[i];
    }
    vector<int> a;
    a.push_back(1);
    function<void(int)> dfs = [&](int u) {
        if (l[u] != -1) dfs(l[u]);
        a.push_back(val[u]);
        if (r[u] != -1) dfs(r[u]);
    };
    dfs(1);
    a.push_back(m);
    Z ans = 1;
    auto comb = [&](int n, int m) -> Z {
        Z res = 1;
        for (int i = 1; i <= m; i++) {
            res = res * (n + 1 - i) / i;
        }
        return res;
    };
    for (int i = 1, j; i <= n; i = j) {
        if (a[i] != -1) {
            j = i + 1;
        } else {
            for (j = i; a[j] == -1; j++);
            int len = j - i;
            int num = a[j] - a[i - 1] + 1;
            ans *= comb(len + num - 1, len);
        }
    }
    cout << ans << '\n';
}
```


### [CF1929C. Sasha and the Casino](https://codeforces.com/contest/1929/problem/C)

赔率为 $k$ 的赌场，保证至多连续输 $x$ 次。求 $a$ 元本金能否保证在有限时间内赢得任意金额，即对任意符合条件的输赢情况，$\lim\limits_{t \to \infty}  money= \infty$。

#### 题解

这场比赛最有趣的一题。大家在学概率论时，也许听说过这么一个问题：

若赔率为 $2$，且赌徒采用如下策略：
- 第 $i$ 次下注 $2^{i-1}$ 元（以保证一旦赢钱，总计能赚 $1$ 元钱），且一旦第一次赢钱就停手。

是否代表赌徒一定稳赚不赔？事实上当胜率 $p < \dfrac 1 2$ 时，由于总下注的期望 $\sum_{i=1}^{\infty} (2^i-1) q^{i-1}p = +\infty$，而成本是有限的，在赢得这一元之前，总会把本金赔光。

回到问题本身来。加入了至多连续输 $x$ 次的限制后，这个策略是否是最优的？有以下观察：

- 每次赢钱后的局面没有本质区别。因此，策略只与当前连续输的局数有关。

- 策略需要确保无论输多少（$\le x$）次之后赢，至少能赚一元钱。

- 由于本金是有限的，在这个前提下，花的本金越少越好。

因此这个策略确实是最优的。花的本金的最大值为连续输了 $x$ 次时，共 $x+1$ 次下注的总和，判断是否有这么多钱即可。每次下注的本金是 $\lceil \dfrac {s + 1} {k-1}\rceil = \lfloor \dfrac s {k-1} \rfloor + 1$，其中 $s$ 是已经输掉的金额。

由于金额是指数增长的，时间复杂度为 $O(\min(x, \log a))$。

#### 代码实现

```cpp
void solve() {
    int k, x, a;
    cin >> k >> x >> a;
    int need = 1;
    for (int round = 0; round < x && need <= a; round++) {
        need += need / (k - 1) + 1;
    }
    cout << (need <= a ? "YES" : "NO") << '\n';
}
```

### [CF1929D. Sasha and a Walk in the City](https://codeforces.com/contest/1929/problem/D)

有一棵树，求满足以下条件点集的数量：

- 树上任意一条简单路径至多包含点集中的两个点。

#### 题解

分析满足条件点集的性质，发现其等价于存在一个点，以它为根时，任意节点之间不存在父子关系。在任意定根的树上，它等价于上面两种情况之一：

- 任意两点之间不存在父子关系；

- 或存在一个点是其它所有节点的父亲，但不是它们的 LCA（即是它们 LCA 的祖先）。

考虑使用树形 DP 如何计算。记 $dp_u$ 为 $u$ 的子树中，任意两点之间不存在父子关系的点集的数量（包含空集），由于 $dp_u$ 一定是 $dp_{fa_u}$ 的一部分，这部分贡献可以汇总到树根计入答案。对第二种情况，我们该点直接将贡献计入答案即可。具体流程如下：

- $dp_u = 1 + \prod\limits_{v} dp_v$

- $ans \leftarrow ans + \sum\limits_v (dp_v - 1)$

对第一种情况，若选择 $u$，则不能选择子树中任意其它节点，这部分方案数为 $1$。而不选择 $u$，则所有子树内的方案互不干扰，根据乘法原理将它们乘起来。

对第二种情况，我们必须选择 $u$，而为了防止成为 LCA，只能选择一个子树中的节点，且注意排除空集。

#### 代码实现

```cpp
void solve() {
    int n;
    cin >> n;
    vector<vector<int>> adj(n);
    for (int e = 1; e < n; e++) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    vector<Z> dp(n);
    Z ans = 0;
    function<void(int, int)> dfs = [&](int u, int fa) {
        dp[u] = 1;
        for (int v : adj[u]) {
            if (v == fa) continue;
            dfs(v, u);
            dp[u] *= dp[v];
            ans += dp[v] - 1;
        }
        dp[u] += 1;
    };
    dfs(0, 0);
    cout << ans + dp[0] << '\n';
}
```

### [CF1929E. Sasha and the Happy Tree Cutting](https://codeforces.com/contest/1929/problem/E)

有一棵树，树上有 $k \le 20$ 条路径。求最少染色多少条边，可以使每条路径上至少有一条边被染色。

#### 题解

考虑将一条边涂色能影响哪些路径，可以发现不同影响的总数是 $O(k)$ 的。证明可以从虚树考虑：建出这 $2k$ 个点的虚树，则虚树中相邻两点连线上所有边的影响相同；而我们知道虚树的边数是 $O(2k) = O(k)$ 的。

我们也可以做更细致的分析：对路径 $(u, v)$，记他们的 LCA 为 $w$，则对该路径造成影响的边是 $(w, v)$ 和 $(w, u)$ 两条自上而下的链上的边。因此边的影响集合向上只在经过一个是某条路径 LCA 的点时才会减小；又因为只要影响集合不减小，将涂色边向上移就不会使解更劣，考虑边的范围可以缩减到所有 $(w, v)$ 和 $(w, u)$ 路径上的第一条边。

之后即可朴素地状压 DP。可以看出这是一个 DAG 上最短路的模型，使用类似最短路的写法。总时间复杂度 $O(n + k 2^k)$。

#### 代码实现

```cpp
constexpr int INF = 1e9 + 7;
void solve() {
    int n;
    cin >> n;
    vector<vector<int>> adj(n);
    for (int e = 1; e < n; e++) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    int k;
    cin >> k;
    vector<int> emsk(n);
    for (int p = 0; p < k; p++) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        emsk[u] ^= 1 << p;
        emsk[v] ^= 1 << p;
    }
    vector<int> buc;
    function<void(int, int)> dfs = [&](int u, int fa) {
        for (int v : adj[u]) {
            if (v == fa) continue;
            dfs(v, u);
            emsk[u] ^= emsk[v];
        }
        for (int v : adj[u]) {
            if (v != fa && (~emsk[u] & emsk[v])) {
                buc.push_back(emsk[v]);
            }
        }
    };
    dfs(0, 0);
    vector<int> dp(1 << k, INF);
    dp[0] = 0;
    for (int msk = 0; msk < (1 << k) - 1; msk++) {
        for (int trans : buc) {
            dp[msk | trans] = min(dp[msk | trans], dp[msk] + 1);
        }
    }
    cout << dp[(1 << k) - 1] << '\n';
}
```

#### UPD：另解

事实上没有边的种类为 $O(k)$ 的性质，这题也可以使用一种暴力至极的方法去做。

考虑令 $g(S) = \sum_{e \in E} [f(e) = S]$，则它与自身 OR 卷积 $t$ 次的结果 $g^{(t)}(S)$ 代表“选出 $t$ 条边，它们影响路径的并集是 $S$ 的方案数（可以重复，考虑顺序）”。注意到答案若存在，显然不超过 $k$（每条选中的边都一定有作用），做卷积一直到全集 $U$ 的方案数不为 $0$ 时，卷积的次数就是答案。时间复杂度 $O(k^2 2^k)$。

注意对一般的问题，中间结果可能会溢出。但由于我们只关心方案数是否为 $0$，每步计算结束后将不为 $0$ 的数全部置为 $1$，即可保证所有数始终在 `int32` 范围内。

#### 代码实现

``` cpp
using Z = int;
vector<Z> fwtOr(const vector<Z> &f, int n, Z mul = 1) {
    vector<Z> res = f;
    for (int k2 = 2, k = 1; k2 <= n; k2 <<= 1, k <<= 1) {
        for (int i = 0; i < n; i += k2) {
            for (int j = 0; j < k; j++) {
                res[i + j + k] += res[i + j] * mul;
            }
        }
    }
    return res;
}

void solve() {
    // use the same emsk
    vector<Z> g(1 << k, 0);
    for (int x : emsk) {
        g[x] = 1;
    }
    vector<Z> conv(g);
    int ans = 1;
    g = fwtOr(g, 1 << k);
    while (!conv[(1 << k) - 1]) {
        conv = fwtOr(conv, 1 << k);
        for (int i = 0; i < (1 << k); i++) {
            conv[i] *= g[i];
        }
        conv = fwtOr(conv, 1 << k, -1);
        for (int &x : conv) {
            if (x) {
                x = 1;
            }
        }
        ans++;
    }
    cout << ans << '\n';
}
```

