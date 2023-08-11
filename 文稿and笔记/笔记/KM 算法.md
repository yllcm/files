# KM 算法

学一遍忘一遍的东西。

KM 算法用于求解二分图最大权完美匹配问题。

Q：如果没有完美匹配，只求最大权怎么办？

A：把剩下的边设为 $0$。

利用线性规划的结论，二分图最大权完美匹配可以转化为最小顶标和问题。也即求出最小的 $\sum a_i+b_i$，使得 $a_i+b_j\geq g_{i,j}$。

结论：最小的 $a_i,b_j$ 一定满足对于匹配中的边 $(i,j)$，一定有 $a_i+b_j=g_{i,j}$，且这个条件是充要的。

充分性：由于 $a_i+b_j\geq g_{i,j}$，且所有 $i,j$ 互不相同，所以将这些不等式加起来可以得到 $\sum a_i+b_j$ 的下界，而上述过程取到了下界。

必要性：以下通过构造证明。

所以问题在于找到一组完美匹配，使得 $a_i,b_i$ 满足上述条件且匹配中 $a_i+b_j=g_{i,j}$。我们将所有这类边构成的子图称为相等子图。

构造的核心思路在于设置合法的初始顶标，依次考虑每个节点 $u$，不断调整顶标向相等子图里面加入边使得存在 $u$ 的增广路。考虑某个状态，二分图上的相等子图中会形成若干条起点为 $u$，终点为左部点的交错路，将交错路构成的集合称作交错树（严格来讲它的结构并不构成树，而是类似寻找增广路过程中的搜索树结构）。那么我们通过下述方法调整顶标：

* 对于交错树中的左部点 $x$ 与不在交错树中的右部点 $y$，求出 $mn=\min_{x,y} a_x+b_y-g_{x,y}$。
* 令所有交错树中的左部点 $x$，执行 $a_x\gets a_x-mn$，右部点 $y$ 执行 $b_y\gets b_y+mn$。

容易发现，交错树中的边仍然满足条件，且不在交错树中的边至少一条边加入相等子图。

考虑这个算法的朴素实现，每次大力找增广路，没找到就求最小边并调整顶标。分析复杂度，上述算法在 $\mathcal{O}(m)$ 的时间内加入一条边，所以总复杂度是 $\mathcal{O}(m^2)=\mathcal{O}(n^4)$ 的。

考虑优化。这个算法的主要寄点就在于每次找到一条边的时候都要重新搜一遍图，能不能让每次搜索加多条边并保证能找到一条增广路呢。

考虑在交错树上跑一个类似 BFS 的过程，初始交错树为 $\{u\}$，每次尝试加入一条边 $(x,y)$，然后利用 $x$ 跑上面的更新过程，尝试加入一边（不一定与 $x$ 相连），不断扩大交错树。假如当前遍历到一个匹配 $(x,y)$，尝试枚举可能加入的点。对于每个右部点，我们维护 $slack_i$ 表示 $\min_{j}a_j+b_i-g_{j,i}$。遍历 $x$ 的时候，尝试使用 $x$ 更新不在交错树中的右部点，即 $slack_i\gets \min(slack_i,a_x+b_i-g_{x,i})$。更新之后，求出 $y'=\text{argmin}_i slack_i$，使用上面的调整顶标方法，将 $y'$ 加入交错树（此时相等子图上 $y'$ 与 $fa_{y'}=\text{argmin}_i a_i+b_y-g_{i,y}$，称其为 $y'$ 交错路上祖先）。如果 $y'$ 没有匹配，则 $y'$ 在交错树上的一条祖先链构成一条增广路，退出 BFS 并增广。否则，将与 $y'$ 匹配的结点 $x'$ 加入交错树（显然满足 $a_{x'}+b_{y'}=g_{x',y'}$），并利用 $x'$ 重复上述更新过程。

由于一次增广至多遍历所有边，所以复杂度为 $\mathcal{O}(nm)=\mathcal{O}(n^3)$。

```cpp
#include<bits/stdc++.h>
#define ll long long
#define ull unsigned long long
#define db double
#define ldb long double
#define pb push_back
#define mp make_pair
#define pii pair<int, int>
#define FR first
#define SE second
#define int long long
using namespace std;
inline int read() {
    int x = 0; bool op = 0;
    char c = getchar();
    while(!isdigit(c))op |= (c == '-'), c = getchar();
    while(isdigit(c))x = (x << 1) + (x << 3) + (c ^ 48), c = getchar();
    return op ? -x : x;
}
const int N = 510;
const int INF = 1e16;
int n, m;
int g[N][N], a[N], b[N], sl[N], pre[N], mat[N], vis[N];
void bfs(int u) {
    for(int i = 1; i <= n; i++)sl[i] = INF;
    for(int i = 1; i <= n; i++)vis[i] = 0;
    int x = mat[0] = u, y = 0;
    while(true) {
        vis[y] = true;
        int mn = INF, pos = 0;
        for(int i = 1; i <= n; i++) {
            if(vis[i])continue;
            int w = a[x] + b[i] - g[x][i];
            if(w < sl[i])sl[i] = w, pre[i] = y;
            if(sl[i] < mn)mn = sl[i], pos = i;
        }
        a[u] -= mn;
        for(int i = 1; i <= n; i++) {
            if(vis[i])b[i] += mn, a[mat[i]] -= mn;
            else sl[i] -= mn;
        }
        if(mat[y = pos] == 0)break;
        x = mat[y];
    }
    while(y)mat[y] = mat[pre[y]], y = pre[y];
    return ;
}
signed main() { 
    freopen("in.txt", "r", stdin);
    freopen("out.txt", "w", stdout);
    n = read(); m = read();
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= n; j++)
            g[i][j] = -INF;
    for(int i = 1; i <= m; i++) {
        int u = read(), v = read(), w = read();
        g[u][v] = max(g[u][v], w);
    }
    for(int i = 1; i <= n; i++) {
        a[i] = -INF;
        for(int j = 1; j <= n; j++)
            a[i] = max(a[i], g[i][j]);
    }
    for(int i = 1; i <= n; i++)bfs(i);
    int ans = 0;
    for(int i = 1; i <= n; i++)ans += a[i] + b[i];
    printf("%lld\n", ans);
    for(int i = 1; i <= n; i++)printf("%lld ", mat[i]); putchar('\n');
    return 0;
}
```



