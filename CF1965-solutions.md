比赛链接：https://codeforces.com/contest/1965

官解链接：https://codeforces.com/blog/entry/128914

比较手速的一场，C 与 D 之间出现了较大的 gifficulty gap。

所幸 C 题猜得比较快（虽然证明其实比较难），最终 rank 190，performance 2525，成功压线拿下 Grandmaster。

<a style="color: red">cpchenpi</a>，堂堂上红！

- 一直到自己的目标达成都没有在比赛中拿下过比较难的题，而是靠着手速带来的高表现分上段，说实话自己有点惭愧。

在题解开始前，最后打一个广告，上红名后我建了一个小群（QQ 902592509），欢迎加入讨论包括并不限于算法竞赛（不限于 Codeforces）、电子音乐、猫鼠手游甚至分享生活等一切话题！

### [CF1965A. Everything Nim](https://codeforces.com/contest/1965/problem/A)

#### 题意

有 $n$ 堆石子，第 $i$ 堆有 $a_i$ 颗。Alice 和 Bob 两人轮流，Alice 先手。每一轮玩家从**所有**非空堆中拿走相同数量的石子，不能行动的失败。若两位玩家均使用最优策略，求最终获胜者。

#### 题解

若所有堆均为空，则无法行动，即为必败态。

若最小的一堆石子只有 $1$ 颗，则只有一种行动方式，即从所有石子堆中同时取出一颗石子。

否则，考虑从所有石子堆中取出一颗石子的新状态为 $S'$。

- 若 $S'$ 为必败态，则走到 $S'$ 即可；当前状态 $S$ 为必胜态。

- 否则，$S'$ 的后继状态中必然存在必败态，记为 $S''$。我们发现 $S'$ 的后继状态集合包含在 $S$ 的后继状态集合中！这样，$S$ 的后继集合中存在必败态，$S$ 是必胜态。

故当前状态 $S$ 必为必胜态。（这是 2023 年 ICPC 网络赛中用到的结论！）

那么只要在最小堆大小为 $1$ 时模拟，直至走到确认必胜/必败态即可。可以使用差分快速实现。

#### 代码实现

``` cpp
void solve() {
    int n; cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];
    sort(a.begin(), a.end());
    a.erase(unique(a.begin(), a.end()), a.end());
    n = a.size();
    int round = 0;
    for (int i = n - 1; i > 0; i--) a[i] -= a[i - 1];
    reverse(a.begin(), a.end());
    while (a.size() && a.back() == 1) {
        a.pop_back(), round ^= 1;
    }
    if (a.size() == 0) round ^= 1;
    cout << (round ? "Bob" : "Alice") << '\n';
}
```

### [CF1965B. Missing Subsequence Sum](https://codeforces.com/contest/1965/problem/B)

#### 题意

给定正整数 $n \le 10^6$ 和 $k \le n$。求一个长度 $m \le 25$ 的数组 $a_{1..m}$，使得：

- $\forall x \in [1, n]$，若 $x \ne k$，存在和为 $x$ 的 $a$ 的子序列。

- 不存在和为 $k$ 的 $a$ 的子序列。

#### 题解

这题有很多精妙的构造方法，这里给出我的思路：

- 先构造数列使得 $a$ 的子序列和恰好覆盖 $1 \sim k - 1$；可以使用二进制在内的很多方法，但请注意最后一步的边界。

- 向 $a$ 中加入 $k + 1$，此时 $a$ 的子序列和覆盖 $1 \sim k - 1$ 以及 $k + 1 \sim 2k$。

- 又回到熟悉的二进制！向 $a$ 中加入 $2k, 4k, 8k, \cdots$ 直至 $a$ 的长度到达 $24$。

  - 此时可以确认 $a$ 的子序列和恰好覆盖了 $n$ 以内所有模 $2k$ 不与 $k$ 同余的数！

  - 最后向 $a$ 中添加 $3k$。容易验证我们得到了满足要求的数组。

    - 我们仍然不能表示出 $k$！

    - 但 $\forall t \ge 1$，我们可以表示出 $t(2k) + k = 3k + (t-1) (2k)$，也就是填上了之后所有空缺。

#### 代码实现

``` cpp
void solve() {
    int n, k;
    cin >> n >> k;
    vector<int> ans;
    int t = 0;
    while (t < k - 1) {
        ans.push_back(min(t + 1, k - 1 - t));
        t += ans.back();
    }
    ans.push_back(k + 1);
    t = 2 * k;
    while (ans.size() < 24) {
        ans.push_back(t);
        t *= 2;
    }
    ans.push_back(3 * k);
    cout << ans.size() << '\n';
    for (int x : ans) cout << x << ' ';
    cout << '\n';
}
```

### [CF1965C. Folding Strip](https://codeforces.com/contest/1965/problem/C)

#### 题意

有一个长度为 $n$ 的纸条，上面有 $n$ 个 $0/1$ 数字。可以在任意两数字之间对其任意翻折。

在满足“所有重叠数字相等”的前提下，翻折后最短长度为多少？

#### 题解

一题比较讨厌的猜猜题。正确答案就是“能折则折”，即在所有相邻相同数字之间翻折。

- 首先，容易发现这样是可行的（如果你第一时间无法证明，可以从实现中看出证明），且最终的结果一定是 $0/1$ 间隔串。

- 如何证明最优？若一个结果串中存在相邻相同字符，则我们可以再调用上面的贪心翻折算法，使其变得更短。

  - 注意我其实略去了“翻折能够合并”的说明。从直觉上这是可行的，可以通过异或的方式将多层折叠展开。

可以使用双指针实现。不过稍微推一推结论，甚至有更简洁的实现方法。

#### 代码实现

``` cpp
void solve() {
    int n; cin >> n;
    string s; cin >> s;
    int now = 0, l = 0, r = 0;
    int dir = 1;
    for (int i = 0, j; i < n; i = j) {
        for (j = i + 1; j < n && s[j] != s[j - 1]; j++);
        now += (j - i) * dir;
        l = min(l, now), r = max(r, now);
        dir *= -1;
    }
    cout << r - l << '\n';
}
```

``` cpp
void solve() {
    int n; cin >> n;
    string s; cin >> s;
    int now = 0, l = 0, r = 0;
    for (int i = 0; i < n; i++) {
        now += (i & 1) ^ (s[i] - '0') ? 1 : -1;
        l = min(l, now), r = max(r, now);
    }
    cout << r - l << '\n';
}
```