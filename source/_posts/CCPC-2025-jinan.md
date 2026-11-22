---
title: CCPC 2025 济南站
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2026-10-8 20:53:10
categories: 题解
tags:
  - ACM
---

CCPC 2025 济南站

<!-- MORE -->

# CCPC 2025 JINAN

### K

#### 题意：

给定 $2n$ 张卡牌，数字为 $1$ 到 $n$，每个数字的卡牌各有 $2$ 次。 $a$ 和 $b$ 各自持 $n$ 张，两人轮流放置卡牌，$a$ 先手。

- 对于当前选手，如果他放置的卡牌号码，在当前牌堆上已经出现过了，那么他获得 $1$ 分，同时把当前牌堆上两个卡牌号码**及其中间的牌**都移除。

已知 $a$ 的出牌顺序，调整 $b$ 的出牌顺序，使得 $a$ 的得分最小化。

$n\le 10^5$

#### 分析：

题目本质是扑克牌中的小猫钓鱼，两张相同的牌可以吃掉中间的所有牌。

观察性质：

- $a$ 手里有若干两张，$b$ 手里就有相同数量的两张。
- 只用让 $a$ 的得分最小化，而不是求 $b$ 的得分最大化，就不用考虑牺牲再得分问题。

所以得出构造方案：

- 在 $a$ 两个相同牌的前面放上 $b$ 的相同牌，这样就可以取消 $a$ 两张牌得分的情况。
- 在此基础并且 $b$ 后手的情况下，$b$ 只需要直接在 $a$ 的单张后面按照 $a$ 的顺序放入即可。

#### 代码：

```cpp
void solve(){
    scanf("%d",&n);
    for (int i=1;i<=n;i++){
        ans[i]=0;
        vis[i]=0;
        mp[i]=0;
    }
    for (int i=1;i<=n;i++){
        scanf("%d",&a[i]);
        vis[a[i]]++;
    }
    vector<int> Ga,Gb;
    for (int i=1;i<=n;i++){
        if (!vis[i]) Gb.p_b(i);
    }
    if (vis[a[1]]==2){
        Ga.p_b(Gb.back());
        Ga.p_b(Gb.back());
        Gb.pop_back();
    }
    for (int i=1;i<=n;i++)
        if (vis[a[i]]==1) Ga.p_b(a[i]);
    int pos=0;
    for (int i=1;i<=n;i++){
        if (vis[a[i]]==2&&a[i]!=a[1]){
            if (!mp[a[i]]){
                mp[a[i]]=Gb[pos++];
            }
            ans[i-1]=mp[a[i]];
        }
    }
    pos=0;
    for (int i=1;i<=n;i++)
        if (!ans[i]) ans[i]=Ga[pos++];
    printf("%d\n",vis[a[1]]==2?1:0);
    for (int i=1;i<=n;i++) printf("%d ",ans[i]);
    printf("\n");
}
```

### L

#### 题意：

给定长度为 $n$ 的数组，执行以下过程：

- 选择当前左边两个数字，取出更大的，作为当前这一轮得分。
- 将更小的那个元素，放置到数组末尾。

有两种操作：

- `C x y`， 交换位置 $x,y$ 的值，$1\le x,y\le n$。
- `A x y`， 查询对于当前数组，第 $x$ 轮到第 $y$ 轮的得分之和，$1\le x\le y<n$。

$n,m\le 10^5$

#### 分析：

既然数据给到 $10^5$，说明操作的查询或者修改一定有一个可以通过数据结构快速的答案，另一个是题目性质自带的复杂度低。

得分之和用数据结构更方便通过数据结构优化（记录每一轮的得分），说明修改操作是可以在 $10^3$ 复杂度内实现的。

观察到，一个数从开头比到结尾，最多 $log_2^n$ 次，因此一个数出现的位置也只有 $log_2^n$ 次，所以只需要记录每次出现的位置，那么修改操作就可以通过 $log_2^n$ 模拟的复杂度完成。

#### 代码：

```cpp
struct Tree{
    int sum,tag;
}t[N];

void pushup(int p){t[p].sum=t[ls(p)].sum+t[rs(p)].sum;}
void add(int l,int r,int p,int k){t[p].sum+=(r-l+1)*k;t[p].tag+=k;}

void pushdown(int l,int r,int p){
    if (!t[p].tag) return;
    int mid=(l+r)>>1;
    add(l,mid,ls(p),t[p].tag);
    add(mid+1,r,rs(p),t[p].tag);
    t[p].tag=0;
}

void modify(int ql,int qr,int l,int r,int p,int k){
    if (ql<=l&&qr>=r){add(l,r,p,k);return;}
    int mid=(l+r)>>1;pushdown(l,r,p);
    if (ql<=mid) modify(ql,qr,l,mid,ls(p),k);
    if (qr>mid) modify(ql,qr,mid+1,r,rs(p),k);
    pushup(p);
}

int query(int ql,int qr,int l,int r,int p){
    if (ql<=l&&qr>=r){return t[p].sum;}
    int mid=(l+r)>>1,res=0;pushdown(l,r,p);
    if (ql<=mid) res+=query(ql,qr,l,mid,ls(p));
    if (qr>mid) res+=query(ql,qr,mid+1,r,rs(p));
    return res;
}

void deal(int x){//重点：模拟写法
    while(nxt[x]!=-1){
        int y=nxt[x],flag=mx[y];
        mx[y]=max(a[pos[y][0]],a[pos[y][1]]);
        a[y]=min(a[pos[y][0]],a[pos[y][1]]);
        modify(y-n,y-n,1,n-1,1,mx[y]-flag);
        x=y;
    }
}

signed main(){
    scanf("%lld%lld",&n,&m);
    queue<int> q;cnt=n;
    for (int i=1;i<=n;i++) {
        scanf("%lld",&a[i]);
        q.push(i);
    }
    while(q.size()>1){//重点：模拟写法
        int x=q.front();q.pop();
        int y=q.front();q.pop();
        mx[++cnt]=max(a[x],a[y]);
        a[cnt]=min(a[x],a[y]);
        pos[cnt][0]=x;pos[cnt][1]=y;
        nxt[x]=nxt[y]=cnt;
        q.push(cnt);
    }
    nxt[cnt]=-1;
    for (int i=n+1;i<=cnt;i++){
        modify(i-n,i-n,1,n-1,1,mx[i]);
    }
    for (int i=1;i<=m;i++){
        int x,y;
        char s;cin>>s;
        scanf("%lld%lld",&x,&y);
        if (s=='C'){
            swap(a[x],a[y]);
            deal(x);deal(y);
        }
        else {
            printf("%lld\n",query(x,y,1,n-1,1));
        }
    }
```

### F

#### 题意：

给定 $n$ 张牌，第 $i$ 张牌取值范围为 $[li,ri]$，给每张牌选一个值，该组牌的权重为：

- 把这 $n$ 个数划分为若干连续段
- 同一段内元素取值都相同，相邻段的取值不同
- 权重为所有连续段的平方之和

比如一组牌 $【1, 1, 3, 3, 5, 5, 5, 3, 3】$，对应权重为 $2^2 + 2^2 + 3^2 + 2^2 = 21$。

求最小化的权重。

$1\le l\le r\le 10^9$，$n\le 10^6$。

#### 分析：

$n$ 的范围到达 $10^6$，那复杂度接近 $O(n)$。

数列上的问题，很容易想到使用 `dp` 来解决。

那么我们需要考虑那些元素是必须要留下的：

- 当前元素——`val`
- 当前的代价——`len`
- 已经连续的长度（用于递推计算代价）——`res`

但是 $l,r$ 的范围太大，不可能每个可取的值都枚举一遍。

可以发现：当 $r-l+1\ge 3$ 时，不管当前数怎么选，一定可以和左右不同，因此这种点可以直接不考虑。

那么现在需要考虑的每个数只有两种取值。

用正常数组来存储两维一定会爆空间，因此用 `vector` 来存储三个需要留下的元素。

但是最后可能的情况上限可以达到 $2^n$，仍然会 T，考虑优化：

- 按照 `val len res` 的顺序从小到大排序
- 如果 `val` 一样，那么 `len` 和 `res` 越小越好
- 如果 `val` 不一样，存下排序的第一个即可

这样的话，每个 `val` 只会对应一个 `len res`，复杂度就可以控制在 $O(n)$。

#### 代码：

```cpp
vector<array<LL,3>> cur,pre,tmp;
//当前选的数、相同数累计长度、当前总价值

void solve(){
    scanf("%d",&n);
    for (int i=1;i<=n;i++) scanf("%lld",&l[i]);
    for (int i=1;i<=n;i++) scanf("%lld",&r[i]);
    for (int i=1;i<=n;i++){
        if (r[i]-l[i]+1>=3) r[i]=l[i]=-i;
    }
    pre.clear();
    for (int i=l[1];i<=r[1];i++){
        pre.p_b({i,1,1});
    }
    for (int i=2;i<=n;i++){
        tmp.clear();
        for (int j=l[i];j<=r[i];j++){
            for (int k=0;k<pre.size();k++){
                LL val=pre[k][0],len=pre[k][1],res=pre[k][2];
                if (val==j) tmp.p_b({j,len+1,res+len*2+1});
                else tmp.p_b({j,1,res+1});
            }
        }
        sort(tmp.begin(),tmp.end());
        cur.clear();
        array<LL,3> lst={0,0,0};
        for (int k=0;k<tmp.size();k++){
            LL val=tmp[k][0],len=tmp[k][1],res=tmp[k][2];
            if (val!=lst[0]||res<lst[2]){
                cur.p_b({val,len,res});
                lst={val,len,res};
            }
        }
        pre=cur;
    }
    LL ans=1e18;
    for (int i=0;i<pre.size();i++) ans=min(ans,pre[i][2]);
    printf("%lld\n",ans);
}
```





