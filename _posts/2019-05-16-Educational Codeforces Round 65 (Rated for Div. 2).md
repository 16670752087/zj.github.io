---
title: Educational Codeforces Round 65 (Rated for Div. 2)
categories:
  - ACM
  - 题解
---
最后一分钟提交不上去，不然E就过了呜呜呜。
## [Telephone Number](https://vjudge.net/problem/CodeForces-1167A)
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 127;
char s[N];
int t, n;
int main()
{
	for (scanf("%d", &t); t--;)
	{
		scanf("%d%s", &n, s);
		printf(find(s, s + n, '8') > s + n - 11 ? "NO\n" : "YES\n");
	}
}
```
## [Lost Numbers](https://vjudge.net/problem/CodeForces-1167B)
暴力check一下即可。
```cpp
#include <bits/stdc++.h>
using namespace std;
int a[6] = {4, 8, 15, 16, 23, 42}, b[6];
int main()
{
	for (int i = 0; i < 4; ++i)
	{
		printf("? %d %d\n", i + 1, i + 2), fflush(stdout);
		scanf("%d", &b[i]);
	}
	do
		if (a[0] * a[1] == b[0] && a[1] * a[2] == b[1] && a[2] * a[3] == b[2] && a[3] * a[4] == b[3])
			return printf("! %d %d %d %d %d %d", a[0], a[1], a[2], a[3], a[4], a[5]), 0;
	while (next_permutation(a, a + 6));
}
```
## [News Distribution](https://vjudge.net/problem/CodeForces-1167C)
每个group里的人都是连通的，最后求一下每个人所在连通块的大小即可。使用并查集实现。
```cpp
#include <bits/stdc++.h>
using namespace std;
struct UnionfindSet : vector<int>
{
	vector<int> siz;
	UnionfindSet(int n) : vector<int>(n), siz(n, 1)
	{
		for (int i = 0; i < n; ++i)
			at(i) = i;
	}
	void merge(int u, int w)
	{
		if (w = ask(w), u = ask(u), w != u)
			siz[at(w) = u] += siz[w];
	}
	int ask(int u) { return at(u) != u ? at(u) = ask(at(u)) : u; }
};
int main()
{
	int n, m;
	scanf("%d%d", &n, &m);
	UnionfindSet ufs(n + 1);
	for (int i = 0, w, u, k; i < m; ++i)
		if (scanf("%d", &k), k)
			for (scanf("%d", &u); --k; ufs.merge(u, w))
				scanf("%d", &w);
	for (int i = 1; i <= n; ++i)
		printf("%d ", ufs.siz[ufs.ask(i)]);
}
```
## [Bicolored RBS](https://vjudge.net/problem/CodeForces-1167D)
二分一下即可。
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 9;
char s[N], ans[N];
int n;
int work(int m)
{
	for (int i = 0, d[2] = {0, 0}; i < n; ++i)
	{
		if (s[i] == '(')
		{
			ans[i] = '0' + (d[0] > d[1]);
			if (++d[d[0] > d[1]] > m)
				return 0;
		}
		else
			ans[i] = '0' + (d[0] < d[1]), --d[d[0] < d[1]];
	}
	return 1;
}
int bs(int b, int e)
{
	if (e - b < 2)
		return e;
	int m = b + e >> 1;
	return work(m) ? bs(b, m) : bs(m, e);
}
int main()
{
	scanf("%d%s", &n, s);
	work(bs(0, n));
	printf("%s", ans);
}
```
## [Range Deleting](https://vjudge.net/problem/CodeForces-1167E)
手速慢了一丢丢，最后一分钟没交上去，赛后马上ac…😔

枚举$1,2,\ldots,x$里的所有上界$r$，找到最大的下界$l$使得$[l,r]$是符合要求，把$l$加入答案即可。比赛的时候这个下界是二分去找的，然而结束之后我仔细想了一下，这个下界$l$是随着$r$单调的，因此可以直接用双指针去维护，于是得到了下面这个线性的做法。

理想是美好的，然而双指针的线性做法居然比二分的做法运行时间要长也是醉了。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll N = 1e6 + 9;
ll n, x, y, ans, mi[N], ma[N], mma[N];
int main()
{
	scanf("%lld%lld", &n, &x);
	for (ll i = 1, a; i <= n; ++i)
	{
		scanf("%lld", &a);
		if (!mi[a])
			mi[a] = i;
		ma[a] = i;
	}
	y = x;
	for (ll i = 1; i <= x; ++i)
	{
		mma[i] = mma[i - 1];
		if (ma[i])
			mma[i] = max(mma[i], ma[i]);
		if (y == x && mi[i] && mi[i] < mma[i - 1])
			y = i;
	}
	for (; x; --x)
	{
		while (y > x || mma[y - 1] > n)
			--y;
		ans += y;
		if (ma[x] && ma[x] > n)
			break;
		if (mi[x])
			n = min(n, mi[x]);
	}
	printf("%lld", ans);
}
```
## [Scalar Queries](https://vjudge.net/problem/CodeForces-1167F)
从小到大枚举a，计算其对答案的贡献。

用两个树状数组分别维护已经考虑过的数里下标前k小和前w大的。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 5e5 + 7;
struct Mod
{
	const ll M;
	Mod(ll M) : M(M) {}
	ll qadd(ll a, ll b) const { return a += b, a < M ? a : a - M; }
	ll add(ll a, ll b) const { return qadd((a + b) % M, M); }
} M(1e9 + 7);
struct BaseFenwick : vector<ll>
{
	BaseFenwick(int n) : vector<ll>(n, 0) {}
	void add(int x, ll w)
	{
		for (; x < size(); x += x & -x)
			at(x) = M.qadd(at(x), w);
	}
	ll ask(int x)
	{
		ll ans = 0;
		for (; x; x -= x & -x)
			ans = M.qadd(ans, at(x));
		return ans;
	}
} t[2] = {N, N};
pair<int, int> a[N];
int n, ans;
int main()
{
	scanf("%d", &n);
	for (int i = 0; i < n; ++i)
		scanf("%d", &a[i].first), a[i].second = i;
	sort(a, a + n);
	for (int i = 0, w, k; i < n; ++i)
	{
		t[1].add(k = n - a[i].second, w = a[i].second + 1);
		t[0].add(k, k);
		ans = M.add(ans, a[i].first * M.add(w * t[0].ask(k), k * M.add(t[1].ask(n), -t[1].ask(k))));
	}
	printf("%d", ans);
}
```
## [Low Budget Inception](https://vjudge.net/problem/CodeForces-1167G)
怪长的题面，大意是计算“盗梦空间”中扭曲空间的角度。按题意找到，双指针强力模拟即可。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef double lf;
const lf PI = acos(-1);
const int N = 2e5 + 9;
int n, d, m, x, cur, a[N];
int main()
{
	scanf("%d%d", &n, &d);
	for (int i = 1; i <= n; ++i)
		scanf("%d", &a[i]);
	for (scanf("%d", &m); m--;)
	{
		for (scanf("%d", &x); cur < n && a[cur + 1] < x;)
			++cur;
		lf ans = 0;
		if (a[cur] == x - 1 && a[cur + 1] == x)
			ans = PI;
		else if (a[cur] == x - 1 || a[cur + 1] == x)
			ans = PI / 2;
		else
		{
			if (cur < n)
				ans = max(ans, atan2(1, a[cur + 1] - x));
			if (cur > 0)
				ans = max(ans, atan2(1, x - a[cur] - 1));
			for (int pl = cur, pr = cur + 1;;)
			{
				if (1 > pl || pr > n || a[pr] - a[pl] > min(a[cur + 1] - x, x - a[cur] - 1) * 4 + 20)
					break;
				int dis1 = a[pr] - x, dis2 = x - a[pl] - 1;
				if (abs(dis1 - dis2) < 2)
				{
					ans = max(ans, atan2(1, 2 * max(dis1, dis2)));
					break;
				}
				dis1 < dis2 ? ++pr : --pl;
			}
		}
		printf("%.9lf\n", ans);
	}
}
```
