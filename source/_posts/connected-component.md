---
title: 联通分量
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-09-04 19:59:20
categories: 学习笔记
tags:
  - 圆方树
  - 联通分量
---

联通分量部分知识

<!-- MORE -->

# 强联通分量

**适用于有向图** 

## 构造过程

假设当前点是 $u$ ，枚举到了一个儿子 $v$。

1. 如果 $v$ 还没有被遍历过 ，就遍历 $v$ ，并更新 $low$。

2. **主要看下第二部分** ：假如 $v$ 已经被遍历过了，并且还在栈里，那说明 $v$ 这个点再往上已经形成了一个强联通分量，但是发现 $u$ 这个点有一条边可以回到 $v$ （返祖边），所以 $u$ 到 $v$ 这一圈也是一个强联通，因为 $u$ 所在的强联通和 $v$ 所在的强联通可以合并成一个，更新一下 $low$。

## 代码：

```cpp
void dfs(int u)
{
    dfn[u]=low[u]=++cnt;
    stk[++top]=u,vis[u]=1;
    for (auto v:G[u])
    {
        if (!dfn[v]) dfs(v),low[u]=min(low[u],low[v]);
        else if (vis[v]) low[u]=min(low[u],dfn[v]); //判断不要更新横叉边
    }
    if (dfn[u]==low[u])
    {
        int y;col++;
        do{
            y=stk[top--];vis[y]=0;id[y]=col;
            ans[col].p_b(y);
        }while(y!=u);
        sort(ans[col].begin(),ans[col].end());
    }
}
```

# 双联通分量

**用于无向图** 

先来点定义：

1. 割点：去掉这个点之后，图的联通分量数会增加。
2. 割边（桥）：去掉这条边之后，图的联通分量数会增加。
3. 点双联通分量：一张图的极大点双联通子图（子图中不含割点）。
4. 变双联通分量：一张图的记大边双联通子图（子图中不含割边）。

双联通分量缩点后会变成树，强联通分量缩点后是一张 DAG ，点双联通分量缩点后其实就是圆方树。

## 点双联通分量

一些性质：

- 任意两点之间的路径上的割点就是两点之间路径的必经点

- 两个点双如果有交，必然只会交于一点且交点一定为割点

- 一个点是割点当且仅当该点同时包含于超过 $1$ 个点双

- 由一条边连接的两个点满足点双连通（通过定义）

- 对于 $𝑛≥3$ 的点双中的任意一点，必然存在经过该点的简单环。

- 无向图中没有横叉边 [证明](https://www.luogu.com.cn/paste/ip7gpjpe)

### tarjan 求割点

求割点时候的判断和前面不同，因为只要 $low[v]>=dfn[u]$，那么就可以判定 $u$ 是割点，所以缩点的位置在判断中，别的地方不变。

### 代码：

```cpp
void tarjan(int u,int fa)
{
	dfn[u]=low[u]=++ts;
	stk[++hh]=u;
	int son=0;
	for (int i=head[u];i;i=e[i].nxt)
	{
		int v=e[i].v;
		if (!dfn[v])
		{
			tarjan(v,u);
			low[u]=min(low[u],low[v]);
			if (low[v]>=dfn[u])
			{
				son++;col++;
				while(stk[hh+1]!=v) ans[col].p_b(stk[hh--]);
				ans[col].p_b(u);
			}
		}
		else low[u]=min(low[u],dfn[v]);
	}
	if (fa==0&&son==0) ans[++col].p_b(u);
}
```



## 边双联通分量

一些性质：

- 任意两点的路径上的割边，就是两点的路径的必经边

- 边双具有传递性，即若 A,B 边双联通，B,C 边双联通，则 A,C 边双连通

  这里需要注意点双不存在这样的性质，比如说给一个 $8$ 字形的两个点双，$a$ 为上面的一个点，$b$ 为割点，$c$ 为下面的一个点，就可以发现 $a$ 和 $c$ 并不点双联通

- 对于同一个边双内的两个点 $(𝑢,𝑣)$，必然满足 $(𝑢,𝑣)$ 边双联通，因为每一个边双内没有割边

- 对于点双内的任意一条边 $(u,v)$ ，一定存在一个经过 $(u,v)$ 的环，存在一个点 $u$ ，也一定存在一个经过 $u$ 的环

通过这些可以发现点双比边双的一些性质更强。


### tarjan 求割边

假设当前的节点为 $u$ ，且枚举到了一个儿子 $v$。

1. 如果 $v$ 还没有被遍历过，就去遍历 $v$，如果 $dfn[u]<low[v]$ ，那么说明 $v$ 这个点怎么走返祖边都回不到 $u$ 这个点，则 $i$ 这条边是**桥**。

2. 如果 $v$ 这个点已经被遍历过了，说明 $i$ 这条边一定是返祖边，一定不会是横叉边，直接更新。

   不用 $stk$ 因为 强联通用 $stk$ 是为了防止更新返祖边的时候更新到了横叉边，但是这里是不会出现横叉边的，因此不需要判断。

### 代码：

```cpp
void tarjan(int u,int from)
{
	dfn[u]=low[u]=++ts;
	stk[++top]=u;
	for (int i=head[u];i;i=e[i].nxt)
	{
		int v=e[i].v;
		if (!dfn[v])
		{
			tarjan(v,i);
			low[u]=min(low[u],low[v]);
			if (dfn[u]<low[v]) bri[i]=bri[i^1]=1;//这里 bri 是割边
		}
		else if (i!=(from^1)) low[u]=min(low[u],dfn[v]);
	}
	if (dfn[u]==low[u])
	{
		int y;col++;
		do{
			y=stk[top--],ans[col].p_b(y);
		}while(y!=u);
	}
}
```

# 圆方树

圆方树的本质就是点双联通分量，在圆方树中我们可以很好的保留下来原先图的信息，对于处理仙人掌的问题，一般也使用圆方树。

圆方树的构造其实非常简单，就是对于每一个点双联通分量向一个新建节点连边，原图中的点是圆的，新建的节点是方的。

但是在弹栈的时候不要把割点也给弹了，因为割点可能存在于多个联通分量中。

##性质

- 性质 1：圆点 $x$ 的度数等于包含他的点双个数
- 性质 2：圆方树上圆方点相间
- 性质 3：圆点 $x$ 是叶子当且仅当在原图上为非割点
- 性质 4：在圆方树上删去 $x$ 和在原图中删去 $x$ 连通性相同
- 性质 5：$x,y$ 在圆方树上的简单路径上的圆点就是原图中 $x,y$ 经过的简单路径的点

## 代码：

```cpp
void tarjan(int u){
    dfn[u]=low[u]=++cnt;
    stk[++top]=u;
    for (auto v:G1[u]){
        if (!dfn[v]){
            tarjan(v);
            low[u]=min(low[u],low[v]);
            if (low[v]>=dfn[u]){
                int y;add(++tot,u);
                do{
                    y=stk[top--],add(tot,y);
                }while(y!=v);
            }
        }
        else low[u]=min(low[u],dfn[v]);
    }
}
```

## 例题

### abc318G

[原题链接](https://atcoder.jp/contests/abc318/tasks/abc318_g)

#### 题目分析：

判断路径是否经过某个点，还是在图上的操作，显然想到圆方树。

直接建出圆方树，并在 $A$ 到 $C$ 路径上的圆点和方点连接的圆点中，若有 $B$ 那就 $Yes$。

#### Code：

```cpp
bool flag;
int n,m,A,B,C;
int fa[N],dep[N];
int dfn[N],low[N],cnt,stk[N],top,tot;
vector<int> G1[N],G2[N];

void add(int u,int v){
    G2[u].p_b(v);
    G2[v].p_b(u);
}

void tarjan(int u){
    dfn[u]=low[u]=++cnt;
    stk[++top]=u;
    for (auto v:G1[u]){
        if (!dfn[v]){
            tarjan(v);
            low[u]=min(low[u],low[v]);
            if (low[v]>=dfn[u]){
                int y;add(++tot,u);
                do{
                    y=stk[top--],add(tot,y);
                }while(y!=v);
            }
        }
        else low[u]=min(low[u],dfn[v]);
    }
}

void dfs(int u,int fath){
    fa[u]=fath;dep[u]=dep[fath]+1;
    for (auto v:G2[u]){
        if (v==fath) continue;
        dfs(v,u);
    }
}

void check(int u){
    if (u<=n&&u==B) flag=1;
    else{
        for (auto v:G2[u]){
            if (v==B) flag=1;
        }
    }
}

signed main(){
    scanf("%d%d",&n,&m);tot=n;
    scanf("%d%d%d",&A,&B,&C);
    for (int i=1;i<=m;i++){
        int u=read(),v=read();
        G1[u].p_b(v);
        G1[v].p_b(u);
    }
    tarjan(1);dfs(1,0);flag=0;
    while(A!=C){
        if (dep[A]<dep[C]) swap(A,C);
        check(A);
        A=fa[A];
    }
    check(A);
    if (flag) puts("Yes");
    else puts("No");
    return 0;
}
```

### P4630

#### 题目分析：

假设固定一对 $s,f$ ，那么 $c$ 可以有 $s$ 到 $f$ 路径上点双大小并 $-2$。

因为本题的简单路径为不经过重复点的路径，和连通性有关，所以可以想到圆方树。

因为 $u,v$ 的答案应该是他们路径的点双大小之和。

注意，路径上除了 $u$ 和 $v$ 以外的割点都会被统计两次，并且最后还要减去 $u$ 和 $v$ 作为 $c$ 的贡献，所以每个圆点权值为 $-1$ ，方点权值为点双的大小。

![](https://img2023.cnblogs.com/blog/3186835/202309/3186835-20230906091410906-2067125270.png)

如上图，从左边到右边，方点的权值分别是 $6$ 和 $5$ ，那么答案为 $5+6=11$ ，中间的那个割点被统计了两次，再减去 $u$ 和 $v$ 的贡献，所以答案应该为 $6-1+5-1-1=8$。

设每个点的权值为 $a_p$ 那么答案应该是 $\sum_{u\not=v,u\le n}\sum_{p\in path(u,v)}a_p$ ，枚举 $u,v$ 来统计答案非常炸复杂度，所以考虑用一个 $dfs$ 来计算每一个点会被多少条路径经过，乘上 $a_p$ 即可。

#### Code：

```cpp
LL ans=0;
int n,m,w[N];
int sz[N],dfn[N],low[N],stk[N];
int node,cnt,tot,top;
vector<int> G1[N],G2[N];

void add(int u,int v){
    G2[u].p_b(v);
    G2[v].p_b(u);
}

void dfs(int u,int fath){
    sz[u]=u<=n;
    LL res=0;
    for (auto v:G2[u]){
        if (v==fath) continue;
        dfs(v,u);
        res+=1ll*sz[v]*sz[u];
        sz[u]+=sz[v];
    }
    res+=1ll*sz[u]*(tot-sz[u]);
    ans+=2*res*w[u];
}

void tarjan(int u){
    dfn[u]=low[u]=++cnt;
    stk[++top]=u;tot++;
    for (auto v:G1[u]){
        if (!dfn[v]){
            tarjan(v);
            low[u]=min(low[u],low[v]);
            if (low[v]>=dfn[u]){
                int y;add(++node,u);
                w[node]=1;
                do{
                    y=stk[top--];
                    add(node,y);
                    w[node]++;
                }while(y!=v);
            }
        }
        else low[u]=min(low[u],dfn[v]);
    }
}

signed main(){
    scanf("%d%d",&n,&m);node=n;
    for (int i=1;i<=m;i++){
        int u=read(),v=read();
        G1[u].p_b(v);
        G1[v].p_b(u);
    }
    memset(w,-1,sizeof w);
    for (int i=1;i<=n;i++){
        if (!dfn[i]){
            tarjan(i);dfs(i,0);
            top=tot=0;
        }
    }
    printf("%lld\n",ans);
    return 0;
}
```

### CF487E

#### 题目分析：

看到是求图上的任意一条简单路径，并且是无向连通图，可以想到转移到圆方树上求解。

思考一下如何计算答案，经过一个方点，那么经过的最小值一定是这个点双中最小的数，经过一个圆点直接取 $\min$ 即可，每一个方点用个 `multiset` 存下来一个点双内的值。

查询的复杂度再加一个树剖就能做到 $q\log ^2n$ 了，但是看下修改的复杂度好像不尽人意。

修改一个点，我们要把与他相连的所有方点的 `multiset` 修改一遍，这个复杂度就寄了。

如果只更新当前点的父亲节点呢？

发现只有当两个点的 $LCA$ 为方点的时候，会少计算方点父亲的权值，特判一下就好了。

时间复杂度为 $O((n+q\log n)\log n)$。

#### Code：

```cpp
int n,m,q,w[N],rev[N];
multiset<int> sq[N];

namespace RST{
    int dfn[N],low[N],stk[N],cnt,node,topp;
    vector<int> G1[N],G2[N];
    void add(int u,int v){
        G2[u].p_b(v);
        G2[v].p_b(u);
    }
    void tarjan(int u){
        dfn[u]=low[u]=++cnt;
        stk[++topp]=u;
        for (auto v:G1[u]){
            if (!dfn[v]){
                tarjan(v);
                low[u]=min(low[u],low[v]);
                if (low[v]>=dfn[u]){
                    int y;add(++node,u);
                    do{
                        y=stk[topp--],add(node,y);
                    }while(y!=v);
                }
            }
            else low[u]=min(low[u],dfn[v]);
        }
    }
}using namespace RST;

namespace DS{
    int sz[N],dep[N],fa[N],son[N],top[N];
    void dfs1(int u,int fath){
        sz[u]=1;dep[u]=dep[fath]+1;fa[u]=fath;
        for (auto v:G2[u]){
            if (v==fath) continue;
            dfs1(v,u);sz[u]+=sz[v];
            if (sz[v]>sz[son[u]]) son[u]=v;
            if (u>n) sq[u].insert(w[v]);
        }
    }
    void dfs2(int u,int topp){
        dfn[u]=++cnt;rev[cnt]=u;top[u]=topp;
        if (son[u]) dfs2(son[u],topp);
        for (auto v:G2[u]) {if (v!=fa[u]&&v!=son[u]) dfs2(v,v);}
    }
    struct nde{
        int mn;
    }t[N<<2];
    void pushup(int p){t[p].mn=min(t[ls(p)].mn,t[rs(p)].mn);}
    void build(int l,int r,int p){
        if (l==r) return t[p].mn=w[rev[l]],void();
        int mid=(l+r)>>1;
        build(l,mid,ls(p)),build(mid+1,r,rs(p));
        pushup(p);
    }
    void update(int pos,int l,int r,int p){
        if (l==r) return t[p].mn=w[rev[l]],void();
        int mid=(l+r)>>1;
        if (pos<=mid) update(pos,l,mid,ls(p));
        else update(pos,mid+1,r,rs(p));
        pushup(p);
    }
    int query(int ql,int qr,int l,int r,int p){
        if (ql<=l&&qr>=r) return t[p].mn;
        int mid=(l+r)>>1,ans=INF;
        if (ql<=mid) ans=min(ans,query(ql,qr,l,mid,ls(p)));
        if (qr>mid) ans=min(ans,query(ql,qr,mid+1,r,rs(p)));
        return ans;
    }
}using namespace DS;

signed main(){
    scanf("%d%d%d",&n,&m,&q);node=n;
    for (int i=1;i<=n;i++) scanf("%d",&w[i]);
    for (int i=1;i<=m;i++){
        int u=read(),v=read();
        G1[u].p_b(v);G1[v].p_b(u);
    }
    tarjan(1);dfs1(1,0);cnt=0;dfs2(1,1);
    for (int i=n+1;i<=node;i++) w[i]=*sq[i].begin();
    build(1,node,1);
    FOR(_,1,q){
        char op;cin>>op;
        int x=read(),y=read();
        if (op=='A'){
            int ans=INF;
            while(top[x]!=top[y]){
                if (dep[top[x]]<dep[top[y]]) swap(x,y);
                ans=min(ans,query(dfn[top[x]],dfn[x],1,node,1));
                x=fa[top[x]];
            }
            if (dep[x]>dep[y]) swap(x,y);
            ans=min(ans,query(dfn[x],dfn[y],1,node,1));
            if (x>n) ans=min(ans,w[fa[x]]);
            printf("%d\n",ans);
        }
        else{
            if (x==1) {w[1]=y;update(dfn[1],1,node,1);continue;}
            sq[fa[x]].erase(sq[fa[x]].find(w[x]));
            sq[fa[x]].insert(w[x]=y);
            w[fa[x]]=*sq[fa[x]].begin();
            update(dfn[fa[x]],1,node,1);
            update(dfn[x],1,node,1);
        }
    }
    return 0;
}
```
