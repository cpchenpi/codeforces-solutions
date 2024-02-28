比赛链接：https://codeforces.com/contest/1918

官解链接：https://codeforces.com/blog/entry/125300

这场比赛官解有几个地方不太清晰，我尽量解释严谨一些。

### [CF1918A. Brick Wall](https://codeforces.com/contest/1918/problem/A)

#### 题意

有 $n \times m$ 的网格，使用 $1 \times k$（$k$ 任意）的砖块填满，砖块可以水平或竖直放置。求水平与竖直砖块数量差的最大值。

#### 题解

每一行最多能填 $\lfloor \dfrac m 2 \rfloor$ 个水平砖块，故水平砖块数量上界是 $n \lfloor \dfrac m 2 \rfloor$。而这个上界能够达到，且同时可以一个竖直砖块都不放，则答案的最大值也是 $n \lfloor \dfrac m 2 \rfloor$。

#### 代码实现

```cpp
void solve() {
    int n, m;
    cin >> n >> m;
    cout << n * (m / 2) << "\n";
}
```

### [CF1918B. Minimize Inversions](https://codeforces.com/contest/1918/problem/B)

#### 题意

有长度均为 $n$ 的排列 $A$ 和 $B$，你可以指定任意两个位置，在 $A$ 和 $B$ 中同时交换这两个位置上的数。让它们逆序对总和达到最小，输出交换后的结果。

#### 题解

提供的操作相当于将 $A$ 和 $B$ 相同位置上的数绑定后，任意重排一对数 $(x, y)$ 组成的数组。下面我们与 A 题类似，证明答案存在下界，并说明这个下界是可以达到的。

考虑在原数组中下标为 $(i, j)$ 的两个数对 $(a_i, b_i)$ 和 $(a_j, b_j)$。若 $[a_i < a_j] = [b_i < b_j]$，则在新数组中它们对逆序对贡献为 $0$ 或 $2$；否则，它们的贡献始终为 $1$，无需考虑。既然 $0$ 和 $2$ 个数的总和是固定的，我们只需要尽量增大 $0$ 的数量，而 $0$ 数量的上界是 $\sum_\limits{i < j} [a_i < a_j] = [b_i < b_j]$，即不存在 $2$ 的时候。

这个上界是可以达到的，只要使所有满足严格二维偏序关系 $x \prec y$ 的两个数对， $x$ 排在 $y$ 之前即可（即偏序图的任意拓扑排序）。这是非常宽松的条件，以 $a_i, b_i, \max(a_i, b_i), \min(a_i, b_i), a_ib_i$ 甚至 $xa_i + yb_i (\forall x, y > 0)$ 等各种函数为 key 排序均可以达到这个目的。

#### 代码实现

```python
for _ in range(int(input())):
    n = int(input())
    a = list(map(int, input().split()))
    b = list(map(int, input().split()))
    sa, sb = zip(*sorted(zip(a, b)))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: max(x[0], x[1])))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: min(x[0], x[1])))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: x[0] + x[1]))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: x[0]))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: x[1]))
    # sa, sb = zip(*sorted(zip(a, b), key=lambda x: x[0] * x[1]))
    print(*sa)
    print(*sb)

```

### [CF1918C. XOR-distance](https://codeforces.com/contest/1918/problem/C)

#### 题意

给定正整数 $a, b, r \le 10^{18}$，求 $\min\limits_{0 \le x \le r} |(a \oplus x) - (b \oplus x)|$。

#### 题解

贪心题。按位考虑：$(a \oplus x) - (b \oplus x) = \sum_\limits{b = 0}^{59} (a_i \oplus x_i) - (b_i \oplus x_i)$。则 $a_i = b_i$ 的位没有贡献，显然要让 $x_i$ 为 $0$。

记 $S$ 为 $a_i \ne b_i$ 的位集合，$X$ 为 $x_i = 1$ 的位集合，$X \subset S$。由于结果带绝对值，$X$ 和 $S - X$ 对应的答案相同；而我们希望 $x$ 尽可能小，因此指定 $x$ 的最高（第 $\max S$）位为 $0$。又因为 $2^{c+1} > \sum_{c=0}^b 2^c$，已经确定最高位的情况下，之后若能够减小答案，能选则选即可。

#### 代码实现

```cpp
void solve() {
    i64 a, b, r;
    cin >> a >> b >> r;
    bool first = true;
    for (i64 m = 1ll << 59; m > 0; m >>= 1) {
        if (!((a ^ b) & m)) continue;
        if (r >= m && !first && abs((a ^ m) - (b ^ m)) < abs(a - b)) {
            a ^= m, b ^= m;
            r -= m;
        }
        if (first) first = false;
    }
    cout << abs(a - b) << "\n";
}
```



### [CF1918D. Blocking Elements](https://codeforces.com/contest/1918/problem/D)

#### 题意

给定数组 $A$。任选下标集合 $B$，删去数组中对应元素，数组被分为 $|B| + 1$  段 $[l_k, r_k]$，代价为“被删去的数之和 $\sum\limits_{b_i \in B} a_{b_i}$”与“分割后的各子段和 $\sum\limits_{i = l_k}^{r_k} a_i$”中的最大值。求代价的最小值。

#### 题解

看到最小化最大值显然要二分答案。判断答案是否可行可以使用 DP：记 $dp_i$ 为 $i$ 被选中，且每段和均不超过 $s$ 时选中数的最小值。枚举上一个选中的数 $j$，转移方程为 $dp_i = a_i + \min\set{dp_j | {\sum\limits_{j + 1}^{i-1} a_i \le s}}$。为了方便记 $a_{n+1} = 0$，最终判断是否有 $dp_{n + 1} \le s$ 即可。

采用线段树、堆等方式都可以 $O(n \log n)$ 完成这个 DP。但注意合法转移区间的上下界是单调增的，因此最优做法是双指针计算转移区间，并使用单调队列优化 DP。总时间复杂度 $O(n(\log n + \log \max A))$。

#### 代码实现

```cpp
constexpr i64 INF = 1e18;
void solve() {
    int n;
    cin >> n, n++;
    vector<i64> a(n + 1);
    for (int i = 1; i < n; i++) cin >> a[i];
    auto check = [&](i64 s) {
        vector<i64> dp(n + 1, INF);
        dp[0] = 0;
        vector<int> q(n + 1);
        i64 sum = 0;
        int pnt = 0, l = 0, r = 1;
        for (int i = 1; i <= n; i++) {
            while (sum > s) sum -= a[++pnt];
            while (q[l] < pnt) l++;
            dp[i] = a[i] + dp[q[l]];
            while (dp[q[r - 1]] > dp[i]) r--;
            q[r++] = i;
            sum += a[i];
        }
        return dp[n] > s;
    };
    i64 lb = ranges::max(a), rb = accumulate(a.begin(), a.end(), 0ll);
    cout << *ranges::partition_point(views::iota(lb, rb + 1), check) << "\n";
}
```

### [CF1918E. ace5 and Task Order](https://codeforces.com/contest/1918/problem/E)

#### 题意

交互题，你需要猜出一个排列 $A$。交互器有一个你一开始并不知道的数 $x$。每次查询你可以提供一个位置 $i$，交互器会告诉你 $a_i$ 与 $x$ 的大小关系，并让 $x$ 向 $a_i$ 靠近 $1$。$n \le 2000$，限定 $40n$ 次查询。

#### 题解

这题让我联想到了 [CF1856D. More Wrong](https://codeforces.com/problemset/problem/1856/D)，所以很自然向分治去想。若将下标序列 $[1, n]$ 以 $a_i$ 为 key 排序，排序后数组的逆即为我们需要的原排列。而我们学过两种分治排序：归并排序和快速排序。在该交互框架下，我并没有找到一种实现归并排序的有效方法；那么考虑使用快速排序完成。

快速排序只需完成划分操作。首先不断询问主元 $p$ 直到 $a_p = x$。之后，要知道 $a_i$ 与 $a_p$ 的大小关系，只需进行一次查询 $i$；再进行一次查询 $p$ 以将 $x$ 恢复为 $a_p$。根据快速排序相关结论，查询次数显然是 $O(n \log n)$ 的；接下来证明它能够满足 $40 n$ 的限制。

[众所周知](http://202.38.64.11/~xuyun/Performance-Analysis-of-QuickSort.pdf)快速排序的期望比较次数是 $C_n = 2(n + 1) H_n - 4n$，$H_{2000} \le 8.18$，则 $C_n \le 12.36n + 16.36$。注意每次比较要做两次查询，故用于比较的查询次数期望为 $2C_n \le 24.72n + 32.72$。

但我们在划分前还要将 $x$ 暴力移到 $a_p$。对于一个大小为 $n$ 的子问题，若选择的主元为 $k$，我们不妨如下分配贡献：$x$ 从 $0$ 或者 $n + 1$ 移动到 $k$（它们是对称的）以进行划分；划分后先解决左子问题，此时 $x = k - 1$，将 $x$ 移动到 $k$，再解决右子问题。

则基准情况是 $T(1) = 2$，递归式
$$
\begin{aligned}
T(n) &= \mathop{E}\limits_{k \sim U[1, n]} [T(k-1)+T(n-k) + k +2] \\
&= \mathop{E}\limits_{k \sim U[1, n]} [T(k-1)+T(n-k)] + 3 + \dfrac{n - 1}2
\end{aligned}
$$
放大 $T(1) = 3$，则可以解得 $T(n) \le \dfrac 1 2 C_n + 3 n \le \dfrac12 (12.36n + 16.36) + 3 n = 9.18n + 8.18$。

故总查询次数的期望 $\le 24.72n + 32.72 + 9.18 n+8.18 = 33.9n + 40.9$，可以比较好地满足 $40n$ 的限制。

这只是期望的分析，要分析超出限制的概率，可以考虑使用[这篇论文](https://arxiv.org/abs/1006.4063)给出的方差和切比雪夫不等式等工具做大致的估计。所幸这题的官解还提供了一个确定性的解答，解递归式可以得到最坏查询次数小于 $40n$，这样我们至少不用担心这题是假题。

#### 代码实现

```cpp
void solve() {
    int n;
    cin >> n;
    vector<int> a(n + 1), res(n + 1);
    for (int i = 1; i <= n; i++) a[i] = i;
    auto query = [&](int n) {
        cout << "? " << n << endl;
        char c;
        cin >> c;
        if (c == '-') exit(0);
        return c;
    };
    function<void(int, int)> sort = [&](int l, int r) {
        if (l > r) return;
        uniform_int_distribution<int> distrib(l, r);
        int m = distrib(rng), pivot = a[m];
        swap(a[l], a[m]);
        while (query(pivot) != '=')
            ;
        int lt = l, gt = r + 1, i = l + 1;
        while (i < gt) {
            if (a[i] == pivot) i++;
            else {
                if (query(a[i]) == '<') {
                    swap(a[i++], a[++lt]);
                } else {
                    swap(a[i], a[--gt]);
                }
                query(pivot);
            }
        }
        swap(a[l], a[lt]);
        sort(l, lt - 1), sort(gt, r);
    };
    sort(1, n);
    for (int i = 1; i <= n; i++) res[a[i]] = i;
    cout << "! ";
    for (int i = 1; i <= n; i++) cout << res[i] << " ";
    cout << endl;
}
```

### [CF1918F. Caterpillar on a Tree](https://codeforces.com/contest/1918/problem/F)

逆天结论题，由于我自己也不是很清楚就没办法讲了。如果有比官解更简单（~~好证~~）的做法可以提供给我，我会在此处更新。

### [CF1918G. Permutation of Given](https://codeforces.com/contest/1918/problem/G)

#### 题意

你需要输出一个给定长度的非零元素数组 $A$，满足：

- 若将 $A$ 变换为一个新的数组 $B$，将每个元素同时替换为其相邻两元素之和（即 $B_i = A_{i-1}+A_{i+1}$，令 $A_0 = A_{n+1}=0$），$B$ 是 $A$ 的排列。

#### 题解（官解）

核心思想：若有一个合法答案末尾两元素不同，我们可以将其扩展两个元素，并且可以不断扩展下去。
$$
[\cdots, a, b] \to [\cdots, a, b, -b, a-b]
$$
$B_{n+ 2}$ 相比 $B_n$ 减少了末尾的 $a$，但又增加了 $\set{a-b, a, -b}$，补充减少的元素后，这恰好是 $A$ 新增加的两个元素。另外，由于 $a \ne b , b \ne 0$，新增加的元素均不为 $0$；由于 $a \ne 0 , b \ne a - b$，这样的扩展可以不断进行下去。

对 $n = 2$ 和 $n = 7$ 分别有答案 $[1, 2]$ 和 $[1, 2, -3, 2, 4, -5, 2]$。这样对全部偶数或 $> 5$ 的奇数已经足够在线性时间内给出答案，我们只要说明对 $n = 5$ 没有答案即可。反证法：$[a, b, c, d, e]$ 被变换为 $[b, a +c, b+d, c+e, d]$，故 $[a, c, e]$ 是 $[a+c, b+d, c+e]$ 的排列。由于 $a, c \ne 0$，一定有 $a + c = e$，但这样 $c + e = c + a + c$ 一定是 $a, c$ 之一，这又与 $a, c \ne 0$ 矛盾。

#### 代码实现

```cpp
void solve() {
    int n;
    cin >> n;
    if (n == 3 || n == 5) return void(cout << "NO\n");
    cout << "YES\n";
    vector<int> ans;
    if (n % 2) ans = {1, 2, -3, 2, 4, -5, -2};
    else ans = {1, 2};
    while ((int)ans.size() < n) {
        int k = ans.size(), a = ans[k - 2], b = ans[k - 1];
        ans.push_back(-b), ans.push_back(a - b);
    }
    for (int x : ans) cout << x << " ";
}
```

