---
author: rotas
date: 2026-5-9
OJ: Luogu
problem id: P1114
---

[题目跳转](https://www.luogu.com.cn/problem/P1114)

> 题目大意：给定一个长度为 $n$ 的 $01$ 序列，找出满足 $0$ 和 $1$ 个数相等的子区间的长度的最大值
> 数据范围：$1 \leq n \leq 10^5$

考虑前缀和 `sum[j]`，显然 `sum[j]` 表示 `1 ~ j` 中 1 的个数，那么只要能够找到 $i < j$，使得区间长度 $j - i + 1$ 是区间和 `sum[j] - sum[i - 1]` 的两倍即可

```cpp
void solve()
{
    int n; cin >> n;
    vector<int> sum(n + 1, 0);
    for (int i = 1; i <= n; i++) cin >> sum[i];
    for (int i = 2; i <= n; i++) sum[i] += sum[i - 1];
    int ans = 0;
    for (int i = 1; i <= n; i++)
        for (int j = i + 1; j <= n; j++)
            if (j - i + 1 == 2 * (sum[j] - sum[i - 1]))
                ans = max(ans, j - i + 1);
    cout << ans << '\n';
}
```

时间复杂度为 $O(n^2)$，显然无法通过所有的测试点

但是，这个方法仍然可以继续优化

注意到我们要求的是 `ans` 的最大值，在内层循环中，`j` 实际上可以直接从当前 `i + ans - 1` 的值开始遍历（因为这时恰好满足 $j - i + 1 = \text{ans}$）

因此就有下面的代码

```cpp
void solve()
{
    int n; cin >> n;
    vector<int> sum(n + 1, 0);
    for (int i = 1; i <= n; i++) cin >> sum[i];
    for (int i = 2; i <= n; i++) sum[i] += sum[i - 1];
    int ans = 0;
    for (int i = 1; i <= n; i++)
        for (int j = i + ans - 1; j <= n; j++)
            if (j - i + 1 == 2 * (sum[j] - sum[i - 1]))
                ans = max(ans, j - i + 1);
    cout << ans << '\n';
}
```

这个虽然很快了，但是最坏时间复杂度仍然为 $O(n^2)$，因此在部分情况下仍然会 $\text{\color{#052242}{TLE}}$

考虑这样的操作：将所有的 0 改为 -1

那么前缀和数组若满足 `sum[i] = 0`，就能说明前 $i$ 个元素中 1 和 0 的个数相等，同样，如果前缀和数组中某区间 $[i, j]$ 中的元素和 `sum[j] - sum[i - 1]` 为 0，也能说明这个区间中 1 和 0 的个数相等

因此符合题意的区间 $[i, j]$ 只需满足 `sum[j] = sum[i - 1]`

那么题目就转化为了寻找 `sum` 数组中两个相等的元素的距离的最大值

$$
\begin{array}{cccccccccc}
\text{index} & 1 & 2 & 3 & 4 & 5 & 6 & 7 & 8 & 9\\
\text{origin} & \color{blue}{0} & \color{red}{1} & \color{blue}{0} & \color{blue}{0} & \color{blue}{0} & \color{red}{1} & \color{red}{1} & \color{blue}{0} & \color{blue}{0}\\
\text{new} & \color{blue}{-1} & \color{red}{1} & \color{blue}{-1} & \color{blue}{-1} & \color{blue}{-1} & \color{red}{1} & \color{red}{1} & \color{blue}{-1} & \color{blue}{-1}\\
\text{sum} & \color{green}{-1} & 0 & \color{green}{-1} & \color{orange}{-2} & -3 & \color{orange}{-2} & \color{green}{-1} & \color{orange}{-2} & -3
\end{array}
$$

显然找最远的两个 -1，区间长度为 6

但是我们发现初始的两个元素也符合要求，但是无法被计算在内，因此我们需要定义 `sum[0] = 0`

现在，我们只需要在 `sum` 中找到值相同且最远的两个元素即可

可以这样操作：

- 用一个哈希表 `first` 记录每个值第一次出现的位置
- 用一个哈希表 `vis` 记录每个值是否出现过
- 初始化 `vis[0] = true`，`sum[0] = 0`
- 如果某个元素第一次出现，记录 `first`，同时更新 `vis` 状态为 `true`
- 如果某个元素不是第一次出现，用其下标减去其第一次出现的位置的下标得到的值更新 `ans`

这样，只需要 `O(n)` 的复杂度即可完成任务

```cpp
void solve()
{
    int n; cin >> n;
    vector<int> sum(n + 1, 0);
    for (int i = 1; i <= n; i++)
    {
        int a; cin >> a;
        sum[i] = (a == 0 ? -1 : 1);
    }
    for (int i = 2; i <= n; i++) sum[i] += sum[i - 1];
    unordered_map<int, int> first;
    unordered_map<int, bool> vis;
    vis[0] = true;
    int ans = 0;
    for (int i = 1; i <= n; i++)
    {
        if (!vis[sum[i]])
        {
            first[sum[i]] = i;
            vis[sum[i]] = true;
        }
        else
        {
            // i - first[sum[i]] 表示 sum[i] 与 sum[i] 第一次出现的位置的距离
            ans = max(ans, i - first[sum[i]]);
        }
    }
    cout << ans << '\n';
}
```

到这里已经能够顺利 $\text{\textcolor{green}{AC}}$ 了

考虑到哈希表是大常数，因此我们想用数组模拟哈希表，但是 `sum` 中存在负数

这里可以考虑给每个元素加上一个偏移量 `n`（因为最多 $n$ 个元素），那么数据的范围变为 `0 ~ 2n`

同时，由于我们只要 `sum` 当前的元素，因此可以考虑一个整形变量滚动，进而优化空间

最终代码如下

```cpp
void solve()
{
    int n; cin >> n;
    int ans = 0, sum = 0;
    vector<int> first(2 * n + 1, 0);
    vector<bool> vis(2 * n + 1, false);
    vis[n] = true; // 0 + n
    for (int i = 1; i <= n; i++)
    {
        int a; cin >> a;
        sum += (a == 0 ? -1 : 1);
        int tmp = sum + n;
        if (!vis[tmp])
        {
            first[tmp] = i;
            vis[tmp] = true;
        }
        else 
        {
            ans = max(ans, i - first[tmp]);
        }
    }
    cout << ans << '\n';
}
```


















