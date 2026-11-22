---
title: 杂题选做（10.1-10.8）
# password: muyiio
# abstract: 有东西被加密了, 请输入密码查看。
# message: 有东西被加密了, 请输入密码查看。
# wrong_pass_message: 这个密码不太对捏๑_๑
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-10-10 18:33:40
categories: 题解
tags:
  - 杂题选做
---

好多 ATC 的题...

<!-- MORE -->

## AGC029E

### 题目描述：

高桥君的公司里有 $n$ 个房间，形成一棵树的结构。某一次他在第 $r$ 个房间里迷路了，他想回到第 $1$ 个房间。为了回到 $1$ 号房间，他会做以下操作：

- 设 $S$ 是一些点的集合，一开始令 $S=\{r\}$.
- 他会选择 $S$ 之外的，与 $S$ 中的点相连的，编号最小的点 $x$，然后把它加入 $S$
- 若 $1\in S$ 则停止操作，否则重复操作。

设 $c_r=|S|-1$，要求 $c_2,c_3,\dots,c_n$。

$n\le 2\times 10^5$。

### 题目分析：

考虑一个 $x$，他在走到根的过程中会不会经过一个 $y$。设 $z$ 为 $x$ 和 $y$ 的 `lca`，那么经过 $y$ 的条件就是，$z$ 到根的路径上最大值（不包括 $z$）要大于 $y$ 到 $z$ 路径上的最大值。

考虑 $y$ 到根路径上的最大值 $mx$，$mx$ 以下都可以作为 $y$，考虑 $mx$ 可不可以作为 $y$，那么在记录一个次大值即可。

发现如果 $mx$ 能作为 $y$，出现，那么 $mx$ 子树内的所有点都可以作为 $y$，搞个树上差分即可。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=3e5+7;

int n,c[N],son[N],dep[N],fa[N];
vector<int> G[N];

void dfs(int u,int mx,int rmx){
    dep[u]=dep[fa[u]]+1;
    if (u>mx){
        swap(rmx,mx);
        mx=u;
    }
    else if (u>rmx) rmx=u;
    if (u>1) c[dep[rmx]<dep[mx]?mx:son[mx]]++;
    for (auto v:G[u]){
        if (v==fa[u]) continue;
        fa[v]=u;son[u]=v;
        dfs(v,mx,rmx);
    }
}

void dfs2(int u){
    c[u]+=c[fa[u]];
    for (auto v:G[u]){
        if (v==fa[u]) continue;
        dfs2(v);
    }
}

signed main(){
    scanf("%d",&n);
    for (int i=1;i<n;i++){
        int u=read(),v=read();
        G[u].p_b(v);G[v].p_b(u);
    }
    dfs(1,0,0);
    dfs2(1);
    for (int i=2;i<=n;i++) printf("%d ",c[i]);
    return 0;
}
  ```

</details>

## ABC150F

### 题目描述：

给定两个长度为 $n$ 的序列 $a=\{a_0,a_1,\cdots,a_{n-1}\}$ 和 $b=\{b_0,b_1,\cdots,b_{n-1}\}$，输出所有有序数对 $(k,x)$ ，满足：  

1. $0\leq k<n$ 且 $x\geq 0$。
2. 序列 $a'=b$，其中 $a'_i = a_{i+k\bmod n}\operatorname{xor} x\ (0\leq i<n)$，“$\operatorname{xor}$”表示按位异或。

输出所有满足条件有序对 $(k,x)$，每对占一行。如果没有满足条件的有序对，输出为空。

$1\leq n\leq 2\times 10^5$，$0\leq a_i,b_i<2^{30}$。

### 题目分析：

假如说我们选 $a_1$ 和 $b_1$ 匹配，那么 $x$ 的值就已经固定了，为 $a_1\oplus b_1$，你发现如果想符合题意，就需要让 $a_1\oplus b_1=a_2\oplus b_2 = ...$，移一下项也就是 $a_1 \oplus a_2=b_1\oplus b_2=...$，也就是说我们可以先把 $a$ 两两异或起来，$b$ 两两异或起来，然后让 $a$ 旋转，使得有一个时间 $a$ 和 $b$ 完全相等，就说明找到题目要求的 $(k,x)$ 了。

把 $a$ 复制一遍然后和 $b$ 做 `kmp` 即可。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=2e5+7;

int n;
int a[N],b[N],nxt[N];
int c[N<<1],d[N];

void kmp(){
	for (int i=2,j=0;i<=n;i++)
	{
		while(j&&d[i]!=d[j+1]) j=nxt[j];
		if (d[i]==d[j+1]) j++;
		nxt[i]=j;
	}
}

signed main(){
    scanf("%d",&n);
    for (int i=1;i<=n;i++) scanf("%d",&a[i]);
    for (int i=1;i<=n;i++) scanf("%d",&b[i]);
    for (int i=1;i<=n;i++){
        c[i]=a[i]^a[i%n+1];
        d[i]=b[i]^b[i%n+1];
    }
    kmp();
    for (int i=1;i<=n;i++) c[n+i]=c[i];
    for (int i=1,j=0;i<2*n;i++)
	{
		while(j&&c[i]!=d[j+1]) j=nxt[j];
		if (c[i]==d[j+1]) j++;
		if (j==n) {
            printf("%d %d\n",i-n,a[1+i-n]^b[1]);
            j=nxt[j];
        }
	}
    return 0;
}
  ```

</details>

## CF1868C

### 题目描述：

给定一颗 $n$ 个节点的完全二叉树，每个点有权值 $a_i \in [1,m]$，定义从 $i$ 到 $j$ 的路径的权值 $s_{i,j}$ 为路径上的最大点权。

求所有树（$m^n$ 种点权）的 $\sum_{i=1}^n \sum_{j=i}^n s_{i,j}$ 的和，模 $998244353$。

对于 $100\%$ 的数据，保证 $1\le n\le 10^{18}$，$1\le m \le 10^5$。

### 题目分析：

这种让求路径上的最大值的题目，一般有两种做法：

第一种是找到一颗子树，把他的权值任意钦定，求出来子树大小为 $x$ 时，最大值为 $m$ 的路径条数，最后用这样的路径条数乘最大值，就是能产生的贡献了。

第二种是找出路径长度为 $i$ 的路径个数，然后路径上的点要有一个是最大值，其他的任选，只要不超过自己就好了。

这里介绍第一种，一般要设两个数组：$g_i$ 表示子树大小为 $i$ 时，路径是从根开始往下延伸的最大值为 $m$ 的路径条数，$f_i$ 表示子树大小为 $i$ 时，路径任意放置中最大值为 $m$ 的路径条数。

这里用 $g$ 是因为你发现 $f$ 的转移很难，所以用 $g$ 来辅助转移。

但是你发现最大值为 $m$ 非常的难求，但是最大值 $\le m$ 是好做的，我们可以用 $\le m$ 的减去 $\le m-1$ 的，就能求出来等于 $m$ 的答案。

那么 $g$ 的转移也很显而易见，我们设 $ls$ 表示左子树的大小，$rs$ 表示右子树的大小：
$$
g[i]=(g[ls]\times m^{rs}+g[rs]\times m^{ls} + m^{ls+rs})\times m
$$
以及 $f$ 的转移：
$$
f[i]=g[i]+g[ls]\times g[rs]\times m+f[ls]\times m^{rs+1}+f[rs]\times m^{ls+1}
$$
**注意**：

1. 因为是一颗完全二叉树，所以我们的左右子树是完全可以求出来的，可以自己手模一下，就知道要怎么求了。
2. 如果说当前为完美二叉树，那么记忆化一下，能让代码跑的更快。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=1e5+7;
const int mod=998244353;
 
int n,m,ans[N],sum,k;
struct node{
	int f,g,p;
}F[64];
 
node dp(int x){
	if (!x) return {0,0,1};
	if (x==1) return {m,m,n};
	x++;
	if (__builtin_popcountll(x)==1&&F[(int)log2(x)].p)
		return F[(int)log2(x)];
	x--;
	int b=log2(x),s=(1ll<<(b-1))-1,s2=s+1;
    int lx=s+min(x-((s<<1)+1),s2),rx=x-lx-1;
	node ls=dp(lx),rs=dp(rx),res;
	res.p=ls.p*rs.p%mod;
	res.g=((ls.g*rs.p%mod+ls.p*rs.g%mod)%mod+res.p%mod)%mod*m%mod;
	res.p=res.p*n%mod;
	res.f=(    (res.g+m*ls.g%mod*rs.g%mod)%mod+
			   (ls.f*rs.p%mod+rs.f*ls.p%mod)%mod*n%mod     )%mod;
	x++;
	if (__builtin_popcountll(x)==1) F[(int)log2(x)]=res;
	x--;
	return res;
}
 
signed main(){
	int T=read();
	while(T--){
		scanf("%lld%lld",&k,&n);
		for (m=1;m<=n;m++) {
			for (int i=0;i<64;i++)
				F[i].f=F[i].g=F[i].p=0;
			ans[m]=dp(k).f%mod;
		}
		sum=0;
		for (m=n;m>=1;m--){
			sum=(sum+m*(ans[m]+mod-ans[m-1])%mod)%mod;
		}
		printf("%lld\n",sum);
	}
	return 0;
}
  ```

</details>

## ARC067D

### 题目描述：

有编号从 $1$ 到 $N$ 的 $N$ 家烧烤店，烧烤店在一条线上按照编号顺序排序，第 $i$ 家烧烤店与第 $i+1$ 家烧烤店的距离是 $A_i$ 。  

你有编号从 $1$ 到 $M$ 的 $M$ 张烧烤券，不管是在哪一家烧烤店都可用烧烤券来吃烧烤，在第 $i$ 家烧烤店用烧烤券 $j$ 可以吃到一顿美味度为 $B_{i,j}$ 的烧烤，每一张烧烤券只能使用一次，但是在一家烧烤店你可以使用任意多张烧烤券。  

你想从自己选择的一家烧烤店开始（随意选择一个开始），然后不断地用未使用的烧烤券去另一家烧烤店。你最终的幸福值是所吃所有烧烤的美味度减去所走的总路程。求最大可能的最终幸福值（ $M$ 张券必须用完）。

对于 $100\%$ 的数据，保证：
$2\le N\le 5\times 10^3$，$1\le M\le 200$，$1\le A_i\le 10^9$，$1\le B_{i,j}\le 10^9$

### 题目分析：

本来想上来直接设 $dp_{i,j}$ 表示第 $i$ 张券，在 $j$ 家用，结果发现编号和券的顺序都可以随意排列，所以 $dp$ 假了。

考虑发现一点性质在做。

发现我们走过的肯定是一个区间，并且是从一个端点走向另外一个端点。

我们通过反证法证明，如果是从中间走到左边再走到右边，显然不如直接从左边走到右边更优。

那么问题就变成了：
$$
\max_{[l,r]\in[1,n]} (\sum_{p=1}^m\max_{j=l}^{r}b_{j,p}+\sum_{i=l}^{r-1}a[i])
$$

发现又是区间取 $\max$，考虑每个点对区间的贡献，如果说当前点是区间的最大值，那么肯定会选当前点。

考虑对于每一个点，用个单调栈找到他左边第一个比他大的数和右边第一个比他大的数，那么左端点在 $[l+1,j]$ 和右端点在 $[j,r-1]$ 的区间都会选择当前点，搞一个二维差分即可。

**注意**，如果有相同的最大值，那么我们不关心选的是谁，只要不重复即可，可以给他按坐标编号，那么也就是左边第一个大于等于他的，和右边第一个大于他的，做二维差分即可。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=5e3+7;

int n,b[N][300];
int a[N],m,stk[N],top;
int s[N][N],r[N],l[N];

void add(int x,int y,int xx,int yy,int val){
    s[x][y]+=val;s[xx+1][yy+1]+=val;
    s[x][yy+1]-=val;s[xx+1][y]-=val;
}

signed main(){
    scanf("%lld%lld",&n,&m);
    for (int i=2;i<=n;i++) scanf("%lld",&a[i]),a[i]+=a[i-1];
    for (int i=1;i<=n;i++)
        for (int j=1;j<=m;j++) scanf("%lld",&b[i][j]);
    for (int i=1;i<=m;i++){
        memset(l,0,sizeof l);
        memset(r,0,sizeof r);
        top=0;
        for (int j=1;j<=n;j++){
            while(top&&b[stk[top]][i]<b[j][i]) top--;
            l[j]=stk[top];stk[++top]=j;
        }
        stk[top=1]=n+1;
        for (int j=n;j>=1;j--){
            while(top>1&&b[stk[top]][i]<=b[j][i]) top--;
            r[j]=stk[top];stk[++top]=j;
            add(l[j]+1,j,j,r[j]-1,b[j][i]);
        }
    }
    int ans=0;
    for (int i=1;i<=n;i++){
        for (int j=1;j<=n;j++){
            s[i][j]+=s[i-1][j]+s[i][j-1]-s[i-1][j-1];
            if (i>j) continue;
            ans=max(ans,s[i][j]-a[j]+a[i]);
        }
    }
    printf("%lld\n",ans);
    return 0;
}
  ```

</details>

## ARC117D

### 题目描述：

给定一棵 $n$ 个节点的树，要求构造出一个点权序列 $E$，满足以下三个条件：

1.所有 $E_i\ge 1(1\le i\le n)$。

2.对于任意一组 $(i,j)(1 ≤ i < j ≤ N)$，使 $|E_i-E_j|\ge dist(i,j)$，$dist$ 即树上两点距离。

3.使 $E$ 中的最大值最小。

对于 $100\%$ 的数据，保证：$2\le N \le 200000$，$1\le A_i < B_i \le N$。

### 题目分析：

可以发现一个性质，选 $1$ 的点一定在直径的一个端点上，那么我们可以从端点开始思考，发现完全可以按照 `dfn` 序进行标号，对于最长的一条链，肯定是最后加进去。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=3e5+7;

int dep[N],n,ans[N];
int idx=0,fail[N],son[N];
vector<int> G[N];

void dfs(int u,int fa){
    dep[u]=dep[fa]+1;fail[u]=0;
    for (auto v:G[u]){
        if (v==fa) continue;
        dfs(v,u);
        if (fail[v]+1>fail[u]) son[u]=v,fail[u]=fail[v]+1;
    }
}

void dfs1(int u,int fa){
    ans[u]=++idx;
    for (auto v:G[u]){
        if (v==fa||v==son[u]) continue;
        dfs1(v,u);
    }
    if (son[u]) dfs1(son[u],u);
    ++idx;
}

signed main(){
    scanf("%d",&n);
    for (int i=1;i<n;i++){
        int u=read(),v=read();
        G[u].p_b(v);G[v].p_b(u);
    }
    dfs(1,0);int K=0;
    for (int i=1;i<=n;i++) if (dep[i]>dep[K]) K=i;
    memset(son,0,sizeof son);
    dfs(K,0);dfs1(K,0);
    for (int i=1;i<=n;i++)
        printf("%d ",ans[i]);
    return 0;
}
  ```

</details>

## ABC259G

### 题目描述：

芷萱和诺丝正在玩一个有趣的游戏。

有一个 $H \times W$ 的正方形网格，$i$ 行 $j$ 列的点的权值为 $a_{i,j}$。

+ 芷萱选择 $H$ 行中的任意几行（可能为 $0$），给这些行上的点放上一张红色卡片。

+ 诺丝选择 $W$ 列中的任意几列（可能为 $0$），给这些列上的点放上一张蓝色卡片。

他们计算本游戏分数的方式如下：

如果存在一个网格 $(i,j)$ 满足 $a_{i,j}<0$ 且这个点上同时存在两种颜色的卡片，则游戏结束，分数为 $-10^{100}$ 分，否则，分数为所有放了卡片（不管放了几张，不管放了什么颜色）的网格的权值之和。

取分数的最大值。

对于 $100\%$ 的数据，保证：$1\le H$，$W\le 100$，$-10^9\le A_{i,j}\le 10^9$。

### 题目分析：

上来想直接网络流的，但是发现行和列的限制和平常的不一样，非常难建模。

考虑逆向思维，为了让答案最大，那就用总权值减去不符合的答案。

考虑什么样的行或者列可能可以计入答案，首先行或者列的总和是负数的肯定不管，考虑剩下的怎么连边才能得到我们想要的答案：

1. 首先对于 $a_{i,j}$ 为负数的情况，行和列肯定不能同时选，那么让 $i$ 到 $j$ 连一条流量为 INF 的边，保证这条边不会被割掉。
2. 对于 $a_{i,j}$ 非负的情况，$i$ 连向 $j$ 一条流量为 $a_{i,j}$ 的边。
3. 考虑对于每一行每一列，你考虑从原点连向 $i$ 一条流量为 $i$ 行总和的边，从 $j$ 连向汇点一条流量为 $j$ 列总和的边。

这样之后求最小割就是我们不选的答案。

如果 $a_{i,j}$ 为负，那么肯定选择行或者列作为割边。

如果 $a_{i,j}$ 为正，那么割行就是不选行，割列就是不选列，割 $a_{i,j}$ 都选。

这样就能构造出和题意一模一样的网络流建模了。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=1e5+7;

int n,m,S,T,f[N],d[N],cur[N];
int cnt=1,head[N],q[N],tot;
int c[N],t[N],a[107][107];// c[i] 表示第 i 行，t[i] 表示第 i 列

struct node{
	int v,nxt;
}e[N];

void add(int u,int v,int w){
	e[++cnt].v=v;e[cnt].nxt=head[u];head[u]=cnt;f[cnt]=w;
    e[++cnt].v=u;e[cnt].nxt=head[v];head[v]=cnt;f[cnt]=0;
}

bool bfs(){
	int hh=0,tt=0;
	memset(d,-1,sizeof(d));
	q[0]=S,d[S]=0,cur[S]=head[S];
	while(hh<=tt){
		int u=q[hh++];
		for (int i=head[u];i;i=e[i].nxt){
			int v=e[i].v;
			if (d[v]==-1&&f[i]){
				d[v]=d[u]+1;
				cur[v]=head[v];
				if (v==T) return 1;
				q[++tt]=v;
			}
		}
	}
	return 0;
}

int find(int u,int limit){
	if (u==T) return limit;
	int flow=0;
	for (int i=cur[u];i&&flow<limit;i=e[i].nxt){
		cur[u]=i;
		int v=e[i].v;
		if (d[v]==d[u]+1&&f[i]){
			int t=find(v,min(f[i],limit-flow));
			if (!t) d[v]=-1;
			f[i]-=t,f[i^1]+=t,flow+=t;
		}
	}
	return flow;
}

int dinic(){
	int sm=0;
	while(bfs()) sm+=find(S,INF);
	return sm;
}

signed main(){
    scanf("%lld%lld",&n,&m);
    S=0,T=n+m+1;
    for (int i=1;i<=n;i++){
        for (int j=1;j<=m;j++){
            scanf("%lld",&a[i][j]);
            if (a[i][j]>=0) add(i,j+n,a[i][j]);
            else add(i,j+n,INF);
            c[i]+=a[i][j],t[j]+=a[i][j];
        }
    }
    int ans=0;
    for (int i=1;i<=n;i++){
        if (c[i]>0){
            ans+=c[i];
            add(S,i,c[i]);
        }
    }
    for (int i=1;i<=m;i++){
        if (t[i]>0){
            ans+=t[i];
            add(i+n,T,t[i]);
        }
    }
    printf("%lld\n",ans-dinic());
    return 0;
}
  ```

</details>

