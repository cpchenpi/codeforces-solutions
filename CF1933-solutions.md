比赛链接：https://codeforces.com/contest/1933

官解暂未放出。

质量不错的一场 D3。

以下所有问题解法都是 $O(n)$ 或 $O(n \log n)$ 的（$n$ 为问题规模），且比较明显，不再标注数据范围和时间复杂度。

### [CF1933D. Turtle Tenacity: Continual Mods](https://codeforces.com/contest/1933/problem/D)

#### 题意

将数组 $a_{1..n} \ge 1$ 重排，问是否能使 $((a_1 \mathbin {\rm mod} a_2) \mathbin {\rm mod} a_3) \cdots \mathbin {\rm mod} a_n \ne 0$。

#### 题解

记 $m$ 为原数组最小值 $\min a$。首先若最小值唯一，只要将其放在数组首位，它将始终不变，符合要求。另外，若有一个模 $m$ 不为零的元素，将它和 $m$ 放在数组的前两位，即可造出一个全新的唯一最小值，符合上面的条件。

除此之外总是无解的。证明：若所有元素都是 $m$ 的倍数，则结果也总是 $m$ 的倍数（因为 $(xm) \mathbin {\rm mod} (ym) = (x \mathbin {\rm mod} y ) m$）。而因为至少存在两个 $m$，至少有一个 $m$ 被用做过模数，在这一步答案一定变为 $0$。

有另外一种判定方式，显然与上面是等价的：若 $\gcd$ 的出现次数至少为两次，则说明答案为否。

#### 代码实现

<details>
<summary>C++，gcd 判据</summary>

``` cpp
void solve() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int &x : a) cin >> x;
    int g = a[0];
    for (int x : a) g = gcd(g, x);
    cout << (ranges::count(a, g) <= 1 ? "YES" : "NO") << "\n";
}
```
</details>

<details>
<summary>Python，min 判据</summary>

``` py
for _ in range(int(input())):
    n = int(input())
    a = list(map(int, input().split()))
    m = min(a)
    if a.count(m) == 1 or any(x % m for x in a):
        print("YES")
    else:
        print("NO")
```
</details>

### [CF1933E. Turtle vs. Rabbit Race: Optimal Trainings](https://codeforces.com/contest/1933/problem/E)

#### 题意

给定数组 $a_i$，每次查询给定 $l$ 和 $u$，求满足条件的最小的 $r$：

- 最小化 $\sum_{i=1}^s (u + 1 - i)$，其中 $s = \sum_{i=l}^r$。

#### 题解

我们知道 $\sum_{i=1}^s (u + 1 - i)$ 是一个二次函数，且可以推出其对称轴为 $u + \dfrac 1 2$，因此我们只需要找到离 $u + \dfrac 1 2$ 最近的 $s$。这可以通过二分查找实现，找到对称轴左右两侧第一个位置，比较与对称轴的距离即可。注意边界。

#### 代码实现

<details>
<summary>C++ 代码</summary>

``` cpp
void solve() {
    int n;
    cin >> n;
    vector<i64> a(n), pres(n + 1);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        pres[i + 1] = pres[i] + a[i];
    }
    int q;
    cin >> q;
    while (q--) {
        int l, u;
        cin >> l >> u;
        int r = ranges::upper_bound(pres, u + pres[l - 1]) - pres.begin();
        if (r == l) cout << r << " ";
        else if (r > n) cout << r - 1 << " ";
        else {
            if ((pres[r] - pres[l - 1]) + (pres[r - 1] - pres[l - 1]) <
                2 * u + 1) {
                cout << r << " ";
            } else {
                cout << r - 1 << " ";
            }
        }
    }
    cout << "\n";
}
```
</details>

<details>
<summary>Python 代码</summary>

``` py
from itertools import accumulate
from bisect import bisect_left

readInt = lambda: int(input())
for _ in range(readInt()):
    n = readInt()
    s = [0] + list(accumulate(map(int, input().split())))
    ans = []
    for _ in range(readInt()):
        l, u = map(int, input().split())
        r = bisect_left(s, s[l - 1] + u + 1)
        if r > n:
            ans.append(n)
        elif r == l:
            ans.append(l)
        else:
            if s[r] - s[l - 1] + s[r - 1] - s[l - 1] >= 2 * u + 1:
                ans.append(r - 1)
            else:
                ans.append(r)
    print(*ans)
```
</details>

### [CF1933F. Turtle Mission: Robot and the Earthquake](https://codeforces.com/contest/1933/problem/F)

#### 题意

一个 $n \times m$ 的网格中，某些点存在石头。每一秒内，所有石头会同时向上移动一格，而主角可以选择向上、下、右移动一格。问是否能从 $(1, 1)$ 移动到 $(n, m)$ 点，并且不碰到任何石头。我们认为石头和主角的移动速度均为 $1$，且在非整数时刻相遇也算碰撞。保证最后一列没有石头。所有上下的移动是循环的。

#### 题解

我们切换到石头的坐标系，则相当于每一步停留在原地，向右下角移动一格，或向下移动两格。

下结论：在石头的坐标系走最短路移动至最后一列，之后再上下移动至终点，这个策略总是最优的。证明：在确定石头坐标系中到达最后一列时的位置时，晚到永远不会占便宜：由于最后一列没有石头，我们永远能花晚到的相同时间，走相同的跟随石头移动的路。

因此我们 BFS 求出石头坐标系中每个位置的最短路，并枚举最后一列更新答案即可。

<details>
<summary>C++ 代码</summary>

``` cpp
constexpr int INF = 1e9 + 7;
void solve() {
    int n, m;
    cin >> n >> m;
    vector<vector<int>> g(n, vector<int>(m));
    for (auto &v : g)
        for (auto &x : v) cin >> x;
    vector<vector<int>> dis(n, vector<int>(m, INF));
    queue<pair<int, int>> q;
    dis[0][0] = 0;
    q.push({0, 0});
    while (!q.empty()) {
        auto [i, j] = q.front();
        q.pop();
        if (j != m - 1 && !g[(i + 1) % n][j + 1]) {
            if (dis[(i + 1) % n][j + 1] == INF) {
                dis[(i + 1) % n][j + 1] = dis[i][j] + 1;
                q.push({(i + 1) % n, j + 1});
            }
        }
        if (!g[(i + 1) % n][j] && !g[(i + 2) % n][j]) {
            if (dis[(i + 2) % n][j] == INF) {
                dis[(i + 2) % n][j] = dis[i][j] + 1;
                q.push({(i + 2) % n, j});
            }
        }
    }
    int ans = INF;
    for (int i = 0; i < n; i++) {
        if (dis[i][m - 1] == INF) continue;
        int t = dis[i][m - 1];
        int accual_pos = ((i - t) % n + n) % n;
        // walk down later
        ans = min(ans, t + (n - 1 - accual_pos));
        // walk up later
        ans = min(ans, t + 1 + accual_pos);
    }
    if (ans == INF) cout << -1 << "\n";
    else cout << ans << "\n";
}
```
</details>

<details>
<summary>Python 代码</summary>

``` py
from collections import deque

INF = 10**9 + 7
for _ in range(int(input())):
    n, m = map(int, input().split())
    g = [list(map(int, input().split())) for _ in range(n)]
    dis = [[INF] * m for _ in range(n)]
    q = deque()
    q.append((0, 0))
    dis[0][0] = 0
    ans = INF
    while q:
        i, j = q.popleft()
        if j != m - 1:
            ni, nj = (i + 1) % n, j + 1
            if not g[ni][nj] and dis[ni][nj] == INF:
                dis[ni][nj] = dis[i][j] + 1
                q.append((ni, nj))
        else:
            pos = (i - dis[i][j]) % n
            ans = min(ans, dis[i][j] + min(pos + 1, n - 1 - pos))
        if not g[(i + 1) % n][j]:
            ni, nj = (i + 2) % n, j
            if not g[ni][nj] and dis[ni][nj] == INF:
                dis[ni][nj] = dis[i][j] + 1
                q.append((ni, nj))
    print(-1 if ans == INF else ans)
```
</details>

### [CF1933G. Turtle Magic: Royal Turtle Shell Pattern](https://codeforces.com/contest/1933/problem/G)

#### 题意

在一个 $n \times m$（$n, m \ge 5$）的平面上放**满**两种棋子，求不存在三子连珠的方案数。

另外会增加多组限制，每个限制规定 $(i, j)$ 位置必须放某一种棋子。每次增加限制后，也输出满足所有限制和前面条件的方案数。

#### 题解

结论：满足条件的棋子摆放本质上只有一种方式，是如下表摆放方式的无限循环：

<table border="1">
    <tr>
        <td>1</td>
        <td>1</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>1</td>
    </tr>
</table>

符合条件的摆放方式即为它的平移和旋转，其中不同的总共有八种。

可以尝试使用类似搜索的方式，通过分类讨论证明这个结论。这里提供一种较为简短的思路。

首先证明一个 $2 \times 2$ 的网格中不存在相同的三角：

<table border="1">
    <tr>
        <td>1</td>
        <td>1</td>
        <td style="color: skyblue">0</td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td>1</td>
        <td style="color: skyblue">1</td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td style="color: skyblue">0</td>
        <td style="color: skyblue">0</td>
        <td style="color: red">?</td>
    </tr>
</table>

其中天蓝色的位置所放的棋子是之后推出的；而最终在红色位置导出矛盾。

那么 $2 \times 2$ 网格中合法的方案本质上只有两种：

<table border="1">
    <tr>
        <td>1</td>
        <td>0</td>
    </tr>
    <tr>
        <td>1</td>
        <td>0</td>
    </tr>
</table>

它已经能确定全平面。

<table border="1">
    <tr>
        <td>1</td>
        <td>0</td>
    </tr>
    <tr>
        <td>0</td>
        <td>1</td>
    </tr>
</table>

在枚举 $(1,3)$ 处的棋子种类后，也能确定全平面。

我们已经得到了我们需要的 $8$ 种排列，证毕。

回到问题本身，一开始输出 $8$，之后每次查询对所有方案各自更新是否满足即可。这个判定可以打表，也有更简短的判定方式，参照代码。

#### 代码实现

<details>
<summary>C++ 代码</summary>

``` cpp
int calc(int i, int j) { return (i % 2) ^ (j % 4 / 2); }

void solve() {
    int n, m, q;
    cin >> n >> m >> q;
    bitset<8> b = ~0;
    cout << 8 << "\n";
    while (q--) {
        int i, j;
        cin >> i >> j;
        string s;
        cin >> s;
        int c = s == "circle";
        for (int d : {0, 1, 2, 3}) {
            if (calc(i, j + d) ^ c) b[d] = 0;
            if (calc(j, i + d) ^ c) b[d + 4] = 0;
        }
        cout << b.count() << "\n";
    }
}
```
</details>

<details>
<summary>Python 代码</summary>

``` py
def calc(x, y):
    return (x + y // 2) & 1

for _ in range(int(input())):
    n, m, q = map(int, input().split())
    a = [1] * 8
    print(8)
    for _ in range(q):
        x, y, s = input().split()
        c = s[0] == "c"
        x, y = int(x), int(y)
        for d in range(4):
            a[d] &= calc(x, y + d) ^ c
            a[d + 4] &= calc(y, x + d) ^ c
        print(sum(a))
```
</details>