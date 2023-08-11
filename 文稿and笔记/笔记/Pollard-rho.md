# Pollard-rho

从来背不下来这个东西在写什么，所以来写个笔记。

### Miller-Rabin

Miller-Rabin 基于：

* 费马素性检测：随一个底数 $a$，然后求一下 $a^{p-1}\bmod p$，看是不是 $1$。假的离谱。
* 二次探测定理：对于奇素数 $p$，$a^2\equiv 1\pmod p$ 的两个解是 $a=1$ 和 $a=p-1$。

所以你就可以缝一下。先判一下 $a^{p-1}\bmod p$，如果不是 $1$ 就直接给它扬了，否则测一下 $a^{\frac{p-1}{2}}$，如果不是 $1$ 或者 $p-1$ 也扬了。如果是 $p-1$ 的话，那么这个底数就没啥用了，随下一个底数测试。

但是快速幂多一个 $\log$，所以可以改成平方。也就是说，先给 $p-1$ 把所有 $2$ 全除掉，然后乘上去。注意到如果是素数，那么它的过程是守序的 $x\to n-1\to 1$ 的过程。否则就不是很守序，呈现一个 $x\to 1$，也就是说 $n-1$ 被扬了。

所以你判一下整个过程有没有 $n-1$ 就完了。当然更为守序的是全都是 $1$。

### Pollard-rho

这个东西可以在 $\mathcal{O}(n^{0.25}\log n)$ 的时间里面求出合数 $n$ 的一个非平凡因数。它基于：

* 生日悖论：随 $\sqrt{n}$ 个值域为 $[1,n]$ 的数期望有两个撞一起。

所以如果你造一个伪随机函数 $f(x)=x^2+c$，$c$ 是任意数。对于 $n$ 的其中一个因数 $m$，在模 $m$ 意义下期望 $\sqrt{m}$ 轮两个数会撞在一起，然后会出现一个长度不超过 $m$ 的环。因为对于合数 $n$ 显然存在小于 $\sqrt{n}$ 的 $m$，所以期望 $n^{0.25}$ 轮会出现这样的事情。

那你尝试一下枚举环的长度 $i$。但是有可能两个数撞在一起你却啥也不知道。所以有这样一个检测方式：check 一下 $\gcd(|x_{2i}-x_i|,n)$，如果 $>1$ 那就完成目标了。

Q：那我求 $\gcd(|x_0-x_i|,n)$ 可以吗。

A：你连环都没进去……

Q：但是 $i$ 也不一定进去了啊。

A：因为尾巴长度也是相同级别的所以可以等 $i$ 比尾巴长在 check 也不迟，check 时候的环长只会是真实环长的常数倍。

不过这个东西需要求逝量 $\gcd$，所以可以把 $\gcd$ 打个包，如果前 $2$ 个没出那就接下来每 $4$ 个 check 一遍 gcd，前 $2^i$ 个没出那就接下来 $2^{i+1}$ 个再 check，反正 check 的次数不会超过环长的常数倍。但是太长了也不好，所以要控制一下最大值为 $127$。

```cpp
mt19937_64 rnd(114514);
bool chk(int n) {
    if(n % 2 == 0)return n == 2;
    int r = n - 1, cnt = 0;
    while(~r & 1)r >>= 1, cnt++;
    for(int _ = 1; _ <= 10; _++) {
        auto ksm = [&](int x, int k, int P) {
            int res = 1;
            for(int pw = x; k; (k & 1) ? res = (__int128)res * pw % P : 0, pw = (__int128)pw * pw % P, k >>= 1);
            return res;
        };
        int a = rnd() % (n - 1) + 1, v = ksm(a, r, n);
        if(v == 1)continue;
        for(int j = 0; j <= cnt; j++) {
            if(v == n - 1)break;
            if(j == cnt)return false;
            v = (__int128)v * v % n;
        }
    }
    return true;
}
int pollard(int n) {
    if(n == 1)return 1;
    int c = rnd() % (n - 1) + 1, s = c, t = 0;
    auto f = [&](int x) {return ((__int128)x * x + c) % n;};
    int val = 1, lim = 1, cnt = 0, d;
    while(s != t) {
        val = (__int128)val * abs(s - t) % n;
        if(++cnt == lim) {
            if((d = __gcd(n, val)) > 1)return d;
            cnt = 0; lim = min(127ll, lim << 1);
        }
        s = f(f(s)); t = f(t);
    }
    if((d = __gcd(n, val)) > 1)return d;
    return n;
}
```

