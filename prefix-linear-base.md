区间线性基问题，即已知序列 $a_{1..n}$，对任意区间 $[l, r]$ 查询 $a_{l..r}$ 中数组成的异或线性基。

使用下面的一种被称作“前缀线性基”的维护方式，这个问题可以做到 $O(n \log U) - O(\log U)$：

- 对线性基中的每个数 $b_i$，同时维护一个下标 $p_i$，表示它存在的后缀。
- 在插入一个数 $x$ 时（将 $[1, r-1]$ 拓展到 $[1, r]$），按如下方式插入数对 $(x, r)$：
  - 对线性基中每个数 $y$，若当前 bit 存在：
    - 若 $y$ 对应的下标 $p < r$，将 $(y, p)$ 替换为 $(x, r)$，并继续向下插入 $(x \mathbin \oplus y, p)$；
    - 否则，继续向下插入 $(x \mathbin \oplus y, r)$。
  - 若不存在，直接在该位置插入 $(x, r)$。
- 对查询 $[l, r]$，要求的线性基就是前缀 $[1, r]$ 组成的线性基中，所有在查询区间内存在的，即 $p \ge l$ 的数。

时间复杂度显然是 $O(n \log U) - O(\log U)$ 的。

可以看到比起一般的线性基插入，这里只在位长相同时增加了下标的交换操作，并未改变线性基本身的正确性。

下面我们证明它对区间线性基查询的正确性。

### 另一种解法与一些启发

我们不妨首先看一种复杂度略差的维护方式：离线扫描右端点，我们实时维护对所有 $l \le r$，区间 $[l, r]$ 组成的线性基。不难发现它的大小从右向左形成一个上升阶梯形状（且有包含关系），因为这相当于向一个空线性基插入从 $r$ 到 $1$ 的数。

正因为大小越来越大且存在包含关系，插入 $a_r$ 时，线性空间增大越来越难，因此只有一个后缀会真正插入 $a_r$。我们直接从 $r$ 向左，对每个线性基尝试插入 $a_r$，直到第一次失败为止；由于每个点最多被成功插入 $O(\log U)$ 次，失败插入最多共 $n$ 次，总复杂度是 $O(n \log ^2 U + n \log U) = O(n \log^2 U)$ 的，而查询的时间复杂度是 $O(\log U)$。

但我们同时发现，上升的起始位置是有限的，也就是阶梯的 $O(\log U)$ 个上升点右侧。而且，正因为向线性空间内插入数越来越难，我们可以在 $O(\log U)$ 的时间内完成这 $O(\log U)$ 次尝试插入。这样，我们得到了复杂度同为 $O(n \log U) - O(\log U)$ 的一个解法，并且只要记录这些上升点和上升点处使线性空间增大的数就可以方便地转为在线。

事实上，最上面的做法中的 $p$ 就是阶梯的上升点。那么，为什么像一开始那样维护上升点是正确的？

### 正确性证明

首先显然查询得到的线性空间包含于 $a_{l..r}$ 张成的空间，只需要结论“若线性基中存在数对 $(y, p)$，则有 $y \in \langle a_p, \cdots, a_r \rangle$ ”，它很容易使用数学归纳法证明。

还需要证明查询得到的线性空间的大小是正确的。首先注意到一个位置 $p_0$ 对应的向量只有在其变为 $0$ （即已经被 $p > p_0$ 的向量线性表示而不再需要）时才会被删除，否则只会变为与 $p > p_0$ 的向量的线性组合，因此 $p \ge p_0$ 的向量张成的线性空间不会减小。而该增大（即 $a_r \notin \langle a_p, \cdots, a_{r-1} \rangle$）时线性空间也会增大，因为 $(a_r, r)$ **总**会被插入。

### 模板实现

``` cpp
template <class T, size_t B> struct LinearBase {
    array<T, B> base;
    LinearBase() { base.fill(0); }
    void Insert(T x) {
        for (int i = B - 1; i >= 0; i--) {
            if (x >> i & 1) {
                if (!base[i]) {
                    base[i] = x;
                    return;
                } else {
                    x ^= base[i];
                }
            }
        }
    }
    auto QueryMax() -> T {
        T res = 0;
        for (int i = B - 1; i >= 0; i--) {
            res = max(res, res ^ base[i]);
        }
        return res;
    }
    auto Contains(T x) -> bool {
        for (int i = B - 1; i >= 0; i--) {
            if (x >> i & 1) {
                x ^= base[i];
            }
        }
        return x == 0;
    }
    auto operator+(const auto &o) -> LinearBase<T, B> {
        auto res = *this;
        for (int i = B - 1; i >= 0; i--) {
            res.Insert(o.base[i]);
        }
        return res;
    }
};

template <class T, size_t B> struct PrefixLinearBase {
    array<T, B> base;
    array<int, B> pos;
    PrefixLinearBase() {
        base.fill(0);
        pos.fill(-1);
    }
    void InsertWithPos(T x, int p) {
        for (int i = B - 1; i >= 0; i--) {
            if (x >> i & 1) {
                if (!base[i]) {
                    base[i] = x, pos[i] = p;
                    return;
                } else if (pos[i] < p) {
                    swap(base[i], x);
                    swap(pos[i], p);
                }
                x ^= base[i];
            }
        }
    }
    auto QueryRange(int l) -> LinearBase<T, B> {
        LinearBase<T, B> res;
        for (int i = 0; i < static_cast<int>(B); i++) {
            if (pos[i] >= l) {
                res.base[i] = base[i];
            }
        }
        return res;
    }
};
```

#### 使用例

模板题：[CF1100F Ivan and Burgers](https://www.luogu.com.cn/problem/CF1100F)，已知序列 $a_{1..n}$，查询区间 $a_{l..r}$ 子序列的最大异或和。

只要求出区间异或线性基，在其上查询异或空间最大值即可。

代码实现：

``` cpp
void GraciousMisery() {
    int n;
    cin >> n;
    PrefixLinearBase<int, B> b;
    vector<PrefixLinearBase<int, B>> bases(n + 1);
    for (int i = 1; i <= n; i++) {
        int x;
        cin >> x;
        b.InsertWithPos(x, i);
        bases[i] = b;
    }
    int q;
    cin >> q;
    while (q--) {
        int l, r;
        cin >> l >> r;
        cout << bases[r].QueryRange(l).QueryMax() << '\n';
    }
}
```

搬到树上：[CF1902F Trees and XOR Queries Again](https://www.luogu.com.cn/problem/CF1902F)，已知一棵点带权的树，查询 $u$ 到 $v$ 的路径上，是否存在一个子序列异或和为 $x$，即需要求出任意路径上值的线性基。求出它们的 LCA $w$，可以分别求 $w$ 到 $u$ 和 $w$ 到 $v$ 两个路径的线性基，之后合并它们即可。

现在我们只要求出每个节点到它的某个祖先路径上的线性空间。容易发现此时对每个节点就是以**深度**为下标的序列上的问题，直接套用上面的模板就在 $O(n(\log n + \log U)) - O(\log n +\log U)$ 时间内解决了。最终单次查询复杂度 $O(\log n + \log^2 U)$，瓶颈在线性基合并。

代码实现：

``` cpp
struct BinaryLifting {
    vector<vector<int>> fa;
    vector<int> dep;
    int lg2;
    BinaryLifting(vector<int> &f, vector<int> &dep)
        : dep(dep), lg2(__lg(f.size())) {
        int n = f.size();
        fa.resize(lg2 + 1);
        fa[0] = f;
        for (int b = 1; b <= lg2; b++) {
            fa[b].resize(n);
            for (int i = 0; i < n; i++) {
                fa[b][i] = fa[b - 1][fa[b - 1][i]];
            }
        }
    }
    auto Ancestor(int u, int diff) -> int {
        for (int b = lg2; b >= 0; b--) {
            if (diff >> b & 1) u = fa[b][u];
        }
        return u;
    }
    auto LCA(int u, int v) -> int {
        if (dep[u] < dep[v]) swap(u, v);
        u = Ancestor(u, dep[u] - dep[v]);
        if (u == v) return u;
        for (int b = lg2; b >= 0; b--) {
            if (fa[b][u] != fa[b][v]) {
                u = fa[b][u];
                v = fa[b][v];
            }
        }
        return fa[0][u];
    }
};

constexpr int B = 20;

void GraciousMisery() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }
    vector<vector<int>> adj(n);
    for (int ei = 1; ei < n; ei++) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    vector<int> fa(n), dep(n);
    vector<PrefixLinearBase<int, B>> bases(n);
    function<void(int)> dfs = [&](int u) {
        bases[u].InsertWithPos(a[u], dep[u]);
        for (int v : adj[u]) {
            if (v == fa[u]) continue;
            fa[v] = u;
            dep[v] = dep[u] + 1;
            bases[v] = bases[u];
            dfs(v);
        }
    };
    dfs(0);
    int q;
    cin >> q;
    auto bin_lift = BinaryLifting(fa, dep);
    while (q--) {
        int u, v, x;
        cin >> u >> v >> x;
        u--, v--;
        int w = bin_lift.LCA(u, v);
        auto space = bases[u].QueryRange(dep[w]) + bases[v].QueryRange(dep[w]);
        if (space.Contains(x)) {
            cout << "YES\n";
        } else {
            cout << "NO\n";
        }
    }
}
```

