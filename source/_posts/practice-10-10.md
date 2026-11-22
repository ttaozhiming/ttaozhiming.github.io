---
title: 杂题选做（10.10-10.19）
# password: muyiio
# abstract: 有东西被加密了, 请输入密码查看。
# message: 有东西被加密了, 请输入密码查看。
# wrong_pass_message: 这个密码不太对捏๑_๑
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-10-26 10:07:40
categories: 题解
tags:
  - 杂题选做
---

做题记录。

<!-- MORE -->

## ARC074D

### 题目描述：

给定一个$H×W$的网格图，`o` 是可以踩踏的点，`.` 是不可踩踏的点。

现有一人在 `S` 处，向 `T` 移动，若此人现在在 $(i,j)$ 上，那么下一步他可以移动到 $(i,k),(k\in[1,W])$ 或 $(k,j),(k\in[1,H])$ 上。

问最少需要将多少个 `o` 改成 `.`，可以使这个人无法从 `S` 到达 `T`，输出最少需要更改的数目；如果无论如何都不能使这个人无法从 `S` 到 `T`，则输出 $-1$。

对有 $100\%$ 的数据，保证：$2\le H$，$W\le 100$。

### 题目分析：

考虑暴力建图，因为一眼就是网络流的模型，每个点向每行每列连他所能到达的，然后还需要一个拆点的操作，每条边的流量为 $1$，暴力建图即可。

交完直接过了，但是大约估计 $10^6$ 的边数，`dinic` 竟然过了，很牛啊。

考虑怎么优化，显然每行没列完全可以建一个点，然后每个点和这个点连双向边即可，这样边数就来到了 $n^2$。

### 代码：

<details>
  <summary> 点击查看代码 </summary>

  ```cpp
const int INF=1e9+7;
const int N=4e6+7;

int n,m,S,T,f[N],d[N],cur[N];
int cnt=1,head[N],q[N];
int a[107][107];
char s[107];
PII s2[2];

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
    if (d[v]==d[u]+1&&f[i])
    {
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

int get(int x,int y){
    return (x-1)*m+y;
}

signed main(){
    scanf("%d%d",&n,&m);
    int sum=n*m+20;
    for (int i=1;i<=n;i++){
        scanf("%s",s+1);
        for (int j=1;j<=m;j++){
            if (s[j]=='S') S=get(i,j),a[i][j]=3,s2[0]={i,j};
            else if (s[j]=='T') T=get(i,j),a[i][j]=4,s2[1]={i,j};
            else if (s[j]=='.') a[i][j]=1,add(get(i,j),get(i,j)+sum,1);
            else if (s[j]=='o') a[i][j]=2,add(get(i,j),get(i,j)+sum,1);
        }
    }
    for (int i=1;i<=n;i++){
        for (int j=1;j<=m;j++){
            if (a[i][j]==1) continue;
            for (int z=1;z<=m;z++){// 同一行
                if (z==j) continue;
                if (a[i][z]!=1){
                    add(a[i][j]==3?get(i,j):get(i,j)+sum,get(i,z),1);
                }
            }
            for (int z=1;z<=n;z++){// 同一列
                if (z==i) continue;
                if (a[z][j]!=1){
                    add(a[i][j]==3?get(i,j):get(i,j)+sum,get(z,j),1);
                }
            }
        }
    }
    if (s2[0].fi==s2[1].fi||s2[0].se==s2[1].se) puts("-1");
    else printf("%d\n",dinic());
    return 0;
}
  ```

</details>

## ABC235G

### 题目描述：

有三种不同颜色的球，分别有 $A,B,C$ 个。（相同颜色的球之间不区分）

将球放入 $N$ 个不同的盒子中，要求：

- 每个盒子至少放了一个球

- 每个盒子不能存在两个相同颜色的球

- 可以不放完所有的球

求放置方案数对 $998244353$ 取模的结果。

对于 $100\%$ 的数据，保证：$1\le N\le 5 \times 10^6$，$0\le A,B,C\le N$。

### 题目分析：

考虑如果没有第一个限制，那么答案就是：
$$
\sum_{i=0}^{A}\binom{n}{i}\times \sum_{i=0}^{B}\binom{n}{i} \times \sum_{i=0}^{C}\binom{n}{i}
$$
考虑加上第第一个限制怎么做，我们设 $g(i)$ 表示 $i$ 个位置不受限制的方案数，$f(i)$ 表示 $i$ 个位置每个位置上至少有一个球的方案数，则：
$$
g(n)=\sum_{i=0}^{n}\binom{n}{i}f(i) \\
g(n)=\sum_{i=0}^{A}\binom{n}{i}\times \sum_{i=0}^{B}\binom{n}{i} \times \sum_{i=0}^{C}\binom{n}{i}
$$
可以二项式反演一下得到：
$$
f(n)=\sum_{i=0}^{n}(-1)^{n-i}\binom{n}{i}g(i)
$$
那么现在问题变成怎么快速求得 $g(i)$，设 $h(i)=\sum_{i=0}^{A}\binom{n}{i}$，因为 $\binom{n}{m}=\binom{n-1}{m}+\binom{n-1}{m-1}$，所以：
$$
h(i+1)=\sum_{j=0}^{A}\binom{i+1}{j}=\sum_{j=0}^{A}\binom{i}{j}+\sum_{j=0}^{A}\binom{i}{j-1}=\sum_{j=0}^{A}\binom{i}{j}+\sum_{j=1}^{A+1}\binom{i}{j-1}-\binom{i}{A}=2\times h(i)-\binom{i}{A}
$$
可以递推预处理得到，复杂度线性。

### 代码：

<details>
  <summary> 点击查看代码 </summary>

  ```cpp
const int INF=1e9+7;
const int N=6e6+7;
const int mod=998244353;

int f[N],g[N],h[N];
int fac[N],ifac[N];
int n,A,B,C;

int qmi(int a,int k){
    int res=1;
    while(k){
        if (k&1) res=res*a%mod;
        a=a*a%mod;k>>=1;
    }return res;
}

int binom(int n,int m)
{
    if (n<m||n<0||m<0) return 0;
    return fac[n]*ifac[m]%mod*ifac[n-m]%mod;
}

signed main(){
    scanf("%lld%lld%lld%lld",&n,&A,&B,&C);
    fac[0]=1;int inv2=qmi(2,mod-2);
    for (int i=1;i<=n;i++) fac[i]=fac[i-1]*i%mod;
    ifac[n]=qmi(fac[n],mod-2);
    for (int i=n-1;i>=0;i--) ifac[i]=ifac[i+1]*(i+1)%mod;
    for (int i=0;i<=A;i++) f[n]=(f[n]+binom(n,i))%mod;
    for (int i=n-1;i>=0;i--) f[i]=(f[i+1]+binom(i,A))%mod*inv2%mod;
    for (int i=0;i<=B;i++) g[n]=(g[n]+binom(n,i))%mod;
    for (int i=n-1;i>=0;i--) g[i]=(g[i+1]+binom(i,B))%mod*inv2%mod;
    for (int i=0;i<=C;i++) h[n]=(h[n]+binom(n,i))%mod;
    for (int i=n-1;i>=0;i--) h[i]=(h[i+1]+binom(i,C))%mod*inv2%mod;
    int ans=0;
    for (int i=0;i<=n;i++)
        ans=(ans+(((n-i)&1)?mod-1:1)*binom(n,i)%mod*f[i]%mod*g[i]%mod*h[i]%mod)%mod;
    printf("%lld\n",ans);
    return 0;
}
  ```

</details>

## AGC037D

### 题目描述：

给定一个 $n\times m$ 的矩阵 $A$ , 保证 $A$ 内的元素为 $1$ 到 $n\times m$ 的排列。

将 $A$ 每一行的元素任意排列得到 $B$。

将 $B$ 每一列的元素任意排列得到 $C$。

将 $C$ 每一行的元素任意排列得到 $D$。

要求 $D_{i,j}=(i-1)\times m+j$ , 请输出一组合法的 $B, C$。

$1\leqslant n,m\leqslant 100$。

### 题目分析：

因为题目里没说无解怎么办，所以猜测保证有解。

正着考虑好像很难，那就倒过来思考，从 $D$ 到 $C$，相当于第 $i$ 行的数 $x$，满足 $\left\lceil \frac{x}{m} \right\rceil=i $，我们按照 $\left\lceil \frac{x}{m} \right\rceil$ 给每个数染色，那 $C$ 到 $B$ 就相当于每一列每一个颜色恰好出现一次。

那么现在问题变成了一共有 $n$ 种颜色，让每一行重新排列，使得 $m$ 列中的每一列都满足每一个颜色恰好出现一次。

完全可以用二分图最大匹配来做，相当于要做 $m$ 次匹配，每次匹配一列。

因为每个 $a_{i,j}$ 不同，并且只有 $n\times m$ 种，所以说每一行一定连着 $n$ 种颜色，根据 $Hall$ 定理，可以证明一定有解。

### 代码：

<details>
  <summary> 点击查看代码 </summary>

  ```cpp
const int INF=1e9+7;
const int N=1e5+7;

int cur[N],d[N];
vector<int> v[107][107];
int f[N],head[N],cnt=1,S,T;
int col[107][107],tmp[1007];
int n,m,a[107][107],b[107][107],c[107][107];
struct node{
    int v,nxt;
}e[N];

void add(int u,int v,int w){
    e[++cnt].v=v;e[cnt].nxt=head[u];head[u]=cnt;f[cnt]=w;
    e[++cnt].v=u;e[cnt].nxt=head[v];head[v]=cnt;f[cnt]=0;
}

bool bfs()
{
    queue<int> q;
    memset(d,-1,sizeof d);
    q.push(S),d[S]=0,cur[S]=head[S];
    while(!q.empty()){
        int u=q.front();q.pop();
        for (int i=head[u];i;i=e[i].nxt){
            int v=e[i].v;
            if (d[v]==-1&&f[i]){
                d[v]=d[u]+1;
                cur[v]=head[v];
                if (v==T) return 1;
                q.push(v);
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
            int t=find(v,min(limit-flow,f[i]));
            if (!t) d[v]=-1;
            f[i]-=t,f[i^1]+=t,flow+=t;
        }
    }
    return flow;
}

int dinic(){
    int ans=0;
    while(bfs()) ans+=find(S,INF);
    return ans;
}

void get(int id){
    for (int i=head[S];i;i=e[i].nxt) f[i]=1,f[i^1]=0;
    for (int i=head[T];i;i=e[i].nxt) f[i^1]=1,f[i]=0;
    dinic();
    for (int j=n+1;j<=n+n;j++){
        for (int i=head[j];i;i=e[i].nxt){
            int u=e[i].v;
            if (f[i]&&u!=T){
                f[i]=0;
                b[u][id]=v[u][j-n].back();
                v[u][j-n].pop_back();
            }
        }
    }
}

signed main(){
    scanf("%d%d",&n,&m);
    S=0,T=n*m+1;
    for (int i=1;i<=n;i++){
        for (int j=1;j<=m;j++){
            scanf("%d",&a[i][j]);
            col[i][j]=(a[i][j]+m-1)/m;
            v[i][col[i][j]].p_b(a[i][j]);
            add(i,col[i][j]+n,1);
        }
    }
    for (int i=1;i<=n;i++){
        add(S,i,1);
        add(i+n,T,1);
    }
    for (int i=1;i<=m;i++) get(i);
    for (int i=1;i<=n;i++,puts("")){
        for (int j=1;j<=m;j++){
            printf("%d ",b[i][j]);
        }
    }
    for (int j=1;j<=m;j++){
        for (int i=1;i<=n;i++) tmp[i]=b[i][j];
        sort(tmp+1,tmp+n+1);
        for (int i=1;i<=n;i++) c[i][j]=tmp[i];
    }
    for (int i=1;i<=n;i++,puts("")){
        for (int j=1;j<=m;j++){
            printf("%d ",c[i][j]);
        }
    }
    return 0;
}
  ```

</details>

## CF444C

### 题目描述：

有一个 $n$ 个元素组成的序列，每个元素有两个属性：颜色 $c_i$ 和权值 $w_i$。$c_i$ 初始为 $i$，$w_i$ 初始为 $0$。$m$ 次操作，操作有两种：

1. `1 l r x`：对 $i\in [l,r]$ 的所有 $i$ 进行如下操作：设第 $i$ 个元素 **原来** 的颜色为 $y$，您要把第 $i$ 个元素的颜色改为 $x$，权值 **增加** $|y-x|$。
2. `2 l r`：求 $\displaystyle\sum_{i=l}^r w_i$。

$1\le n,m\le 10^5$，$1\le x\le 10^8$。

### 题目分析：

 考虑线段树暴力做，每次是 $n\log n$ 的，复杂度甚至不如 $n^2$ 的优秀。

但是一个区间如果颜色是一样的，那么直接可以打上标记，否则就暴力修改。

因为每次修改只会修改两个端点，所以复杂度为 $q\log n$。

### 代码：

<details>
  <summary> 点击查看代码 </summary>

  ```cpp
const int INF=1e9+7;
const int N=1e5+7;

int n,Q;
namespace SG{
    struct node{
        int l,r;
        int sum,tag,col;
    }t[N<<2];
    void pushup(int p){
        if (t[ls(p)].col==t[rs(p)].col) t[p].col=t[ls(p)].col;
        else t[p].col=0;
        t[p].sum=t[ls(p)].sum+t[rs(p)].sum;
    }
    void pushdown(int p){
        if (!t[p].tag) return;
        t[ls(p)].sum+=(t[ls(p)].r-t[ls(p)].l+1)*t[p].tag;
        t[rs(p)].sum+=(t[rs(p)].r-t[rs(p)].l+1)*t[p].tag;
        t[ls(p)].col=t[rs(p)].col=t[p].col;
        t[ls(p)].tag+=t[p].tag;t[rs(p)].tag+=t[p].tag;
        t[p].tag=0;
    }
    void build(int l,int r,int p){
        t[p].l=l,t[p].r=r,t[p].sum=0;
        if (l==r) return t[p].col=l,void();
        int mid=(l+r)>>1;
        build(l,mid,ls(p));build(mid+1,r,rs(p));
        pushup(p);
    }
    void modify(int ql,int qr,int l,int r,int p,int x){
        if (ql<=l&&qr>=r&&t[p].col){
            t[p].sum+=(t[p].r-t[p].l+1)*abs(t[p].col-x);
            t[p].tag+=abs(t[p].col-x);t[p].col=x;return;
        }
        int mid=(l+r)>>1;pushdown(p);
        if (ql<=mid) modify(ql,qr,l,mid,ls(p),x);
        if (qr>mid) modify(ql,qr,mid+1,r,rs(p),x);
        pushup(p);
    }
    int query(int ql,int qr,int l,int r,int p){
        if (ql<=l&&qr>=r) return t[p].sum;
        int mid=(l+r)>>1,sum=0;pushdown(p);
        if (ql<=mid) sum+=query(ql,qr,l,mid,ls(p));
        if (qr>mid) sum+=query(ql,qr,mid+1,r,rs(p));
        return sum;
    }
}using namespace SG;

signed main(){
    scanf("%lld%lld",&n,&Q);
    build(1,n,1);
    while(Q--){
        int opt=read(),l=read(),r=read(),x;
        if (opt==1){
            x=read();
            modify(l,r,1,n,1,x);
        }
        else printf("%lld\n",query(l,r,1,n,1));
    }
    return 0;
}
  ```

</details>

## CF348C

### 题目描述：

给定一个 $n$ 个数的序列 $a$，$m$ 个下标集合，记 $S_{k}=\{S_{k,i}\}$。
两种操作：

1. `? k`：求集合 $k$ 的和，即求集合 $k$ 所有元素做原数组下标的和。

2. ` + k w`：给集合 $k$ 的所有下标代表的数加 $w$。

**输入格式**：

输入的第一行包括三个整数  $n$，$m$ 和 $q$ $(1\le n,m,q\le 10^5)$ 。

第二行包括 $n$ 个整数 $ a_1, a_2, ..., a_n $，表示数组 $a$。

接下来的 $m$ 行，每行描述了一个集合。第 $k$ 行首先包括一个正整数 $x$，表示集合 $S_k$ 的大小。接下来是  $x$  个正整数，表示集合 $S_k$ 的元素所处的下标。

接下来的 $q$ 行是将要进行的操作。每种操作占一行。第一种操作形如 `? k`，表示询问集合 $S_k$ 中所有元素的和。第二种操作形如 `+ k x`，表示将集合 $S_k$ 中的所有元素加上 $x$。对于每个询问，都有 $(1\le k\le m,|x|\le 10^8)$。

保证所有集合的元素数量和不超过 $10^5$。

### 题目分析：

上来看到要加一个集合，以为可以线段树，但是其实加的是一个下标集合，线段树这类结构只能处理连续区间的操作，但是不可以分散着处理。

考虑根号分治，把集合大小 $\le \sqrt{n}$ 的记为小区间，大于的记为大区间。

每次修改小区间的时候，直接暴力修改，和大区间的交集，直接暴力加即可，复杂度为 $\sqrt{n}$。

对于大区间，可以打上 $tag$ 做即可。

这样复杂度就是 $n\sqrt{n}$ 的了。

**注意**，处理交集的时候，只用处理大区间对于所有区间的交集个数即可，这个复杂度是集合大小 $\times \sqrt{n}$ 的。

### 代码：

<details>
  <summary> 点击查看代码 </summary>


  ```cpp
const int INF=1e9+7;
const int N=1e5+7;

int n,m,q,sz,tot,num[N],cnt;
LL sum[N],tag[N],a[N];
int f[320][N];
vector<int> v[N],b;
bool visb[N],vis[N];

signed main(){
    scanf("%d%d%d",&n,&m,&q);sz=sqrt(n);
    for (int i=1;i<=n;i++) scanf("%lld",&a[i]);
    for (int i=1;i<=m;i++){
        int x=read(),y;
        for (int j=1;j<=x;j++){
            scanf("%d",&y),v[i].p_b(y),sum[i]+=a[y];
        }
        if (x>=sz) b.p_b(i),visb[i]=1,num[i]=++cnt;
    }
    for (auto i:b){
        for (auto j:v[i]) vis[j]=1;
        for (int j=1;j<=m;j++){
            if (j==i) continue;
            for (auto s:v[j])
                if (vis[s]) f[num[i]][j]++;
        }
        for (auto j:v[i]) vis[j]=0;
    }
    string opt;int x,y;
    while(q--){
        cin>>opt;
        if (opt=="+"){
            x=read(),y=read();
            sum[x]+=1ll*(int)v[x].size()*y;
            if (visb[x]) tag[x]+=y;
            else{
                for (auto s:v[x]) a[s]+=y;
                for (auto i:b)
                    if (f[num[i]][x]&&i!=x)
                        sum[i]+=1ll*f[num[i]][x]*y;
            }
        }
        else {
            x=read();LL ans=0;
            if (visb[x]) ans=sum[x];
            else for (auto s:v[x]) ans+=a[s];
            for (auto i:b) if (i!=x) ans+=1ll*f[num[i]][x]*tag[i];
            printf("%lld\n",ans);
        }
    }
    return 0;
}
  ```

</details>

