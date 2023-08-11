# PAM

也是写一次忘一次的东西。

回文自动机的一条链接表示往两边添加相同的字符。一条后缀链接表示它的最长回文后缀。和 SAM 一样，这东西也是增量构造的，但是 $lst$ 的意义不太一样，因为不是每个前缀都是回文串。

初始回文树只有两个节点：奇根和偶根，它们的父亲同为奇根。但是那个奇根很假，它只有偶根一个儿子。接下来我们增量构造：

* 对于当前 $lst$，爬父亲直到当前字符 $c$ 和 $s_{i-len_{lst}-1}$ 能匹配为止，假设最后的节点为 $p$。
* 如果 $p$ 没有字符为 $c$ 的链接，那么新建一个节点。此时还需要找它的父亲，这个需要从 $q=fa_p$ 往上爬。
* **找到 $p,q$ 之后**，维护 PAM 信息：$(len_{cur},fa_{cur}),son_{p,c}$。
* 将 $lst$ 置为 $son_{p,c}$。

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
using namespace std;
inline int read() {
    int x = 0; bool op = 0;
    char c = getchar();
    while(!isdigit(c))op |= (c == '-'), c = getchar();
    while(isdigit(c))x = (x << 1) + (x << 3) + (c ^ 48), c = getchar();
    return op ? -x : x;
}
const int N = 5e5 + 10;
int n, tot = 2, lst = 1;
int fa[N], len[N], dep[N], a[N], son[N][26];
char s[N];
void extend(int t, int c) {
    int p = lst;
    while(a[t - len[p] - 1] != c)p = fa[p];
    if(son[p][c] == 0) {
        int cur = ++tot, q = fa[p];
        while(a[t - len[q] - 1] != c)q = fa[q];
        fa[cur] = (son[q][c] ? son[q][c] : 2);
        len[cur] = len[p] + 2;
        dep[cur] = dep[fa[cur]] + 1; son[p][c] = cur;
    }
    lst = son[p][c];
    return ;
}
int main() { 
    scanf("%s", s + 1); n = strlen(s + 1);
    len[1] = -1; len[2] = 0; fa[1] = fa[2] = 1; a[0] = 114514;
    for(int i = 1, lstans = 0; i <= n; i++) {
        int c = ((int)s[i] - 97 + lstans) % 26;
        a[i] = c; extend(i, c);
        printf("%d ", lstans = dep[lst]);
    }
    return 0;
}
```

