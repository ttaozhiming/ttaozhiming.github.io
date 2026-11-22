---
title: 字符串小练习
categories: 题解
tags:
  - AC自动机
  - SAM
date: 2023-09-14 15:18:10
---

一些 ACAM 和 SAM 的题

<!-- MORE -->

## AC 自动机

### P2414

#### 题目描述：

阿狸喜欢收藏各种稀奇古怪的东西，最近他淘到一台老式的打字机。打字机上只有 $28$ 个按键，分别印有 $26$ 个小写英文字母和 `B`、`P` 两个字母。经阿狸研究发现，这个打字机是这样工作的：

* 输入小写字母，打字机的一个凹槽中会加入这个字母(这个字母加在凹槽的最后)。
* 按一下印有 `B` 的按键，打字机凹槽中最后一个字母会消失。
* 按一下印有 `P` 的按键，打字机会在纸上打印出凹槽中现有的所有字母并换行，但凹槽中的字母不会消失。

例如，阿狸输入 `aPaPBbP`，纸上被打印的字符如下：

```
a
aa
ab
```

我们把纸上打印出来的字符串从 $1$ 开始顺序编号，一直到 $n$。打字机有一个非常有趣的功能，在打字机中暗藏一个带数字的小键盘，在小键盘上输入两个数 $(x,y)$（其中 $1\leq x,y\leq n$），打字机会显示第 $x$ 个打印的字符串在第 $y$ 个打印的字符串中出现了多少次。

阿狸发现了这个功能以后很兴奋，他想写个程序完成同样的功能，你能帮助他么？

对于 $100\%$ 的数据，$1\leq n\leq 10^5$，$1\leq m\leq10^5$，第一行总长度 $\leq 10^5$。

#### 题目分析：

首先一个 $t$ 在 $s$ 中出现过，相当于在 $s$ 的某一个前缀的 `fail` 上出现过。

第一想法是把 $t$ 的终止节点打上标记，然后对于每一个 $s$ 的前缀节点，去找他的祖先中是否有 $t$。

但是发现这个复杂度有点炸裂，所以转化一下角度，上面的操作相当于给 $s$ 的每个前缀打上标记，去找 $t$ 的子树中有多少个标记。

这样的话搞一个 `dfs` 序，然后单点加区间查询就好了，但是这样的话复杂度还是很寄。

考虑把询问离线下来，对于每一次 $t$ 在 $s$ 中出现了多少次，用一个 `vector` 把 $t$ 记在 $s$ 中。

因为这个题有一个很好的性质，就是这个 `Trie` 是一条链，所以可以直接从前往后枚举，如果当前点为 $s$ 串的结尾，那么去枚举他的询问 $t$，查询子树和即可。

总复杂度为 $O(n\log n)$ 的。

#### 代码：

```cpp
int n,m,ch[N][27],rt=1;
int pos[N],pre[N],tot=1,fail[N];
int sz[N],dfn[N],idx,tr[N],ans[N];
vector<int> G[N];
vector<PII> v[N];
char s[N];

void build(){
    queue<int> q;
    for (int i=0;i<26;i++){
        if (ch[rt][i]) fail[ch[rt][i]]=rt,q.push(ch[rt][i]);
        else ch[rt][i]=rt;
    }
    while(!q.empty()){
        int u=q.front();q.pop();
        for (int i=0;i<26;i++){
            if (ch[u][i]) fail[ch[u][i]]=ch[fail[u]][i],q.push(ch[u][i]);
            else ch[u][i]=ch[fail[u]][i];
        }
    }
}

void dfs(int u,int fa){
    sz[u]=1;dfn[u]=++idx;
    for (auto v:G[u]){
        if (v==fa) continue;
        dfs(v,u);
        sz[u]+=sz[v];
    }
}

void add(int x,int k){
    if (!x) return;
    for (;x<N;x+=lowbit(x)) tr[x]+=k;
}

int query(int x){
    if (x<=0) return 0;
    int res=0;
    for (;x;x-=lowbit(x)) res+=tr[x];
    return res;
}

signed main(){
    scanf("%s",s+1);
    n=strlen(s+1);
    int now=rt,cnt=0;
    for (int i=1;i<=n;i++){
        if (s[i]=='P') pos[++cnt]=now;
        else if (s[i]=='B') now=pre[now];
        else {
            int c=s[i]-'a';
            if (!ch[now][c]) ch[now][c]=++tot;
            pre[ch[now][c]]=now;
            now=ch[now][c];
        }
    }
    build();
    for (int i=2;i<=tot;i++) G[fail[i]].p_b(i);
    dfs(1,0);scanf("%d",&m);
    for (int i=1;i<=m;i++){
        int x=read(),y=read();
        v[y].p_b(m_p(x,i));
    }
    now=rt,cnt=0;
    for (int i=1;i<=n;i++){
        if (s[i]=='P'){
            cnt++;
            for (auto s:v[cnt]){
                int x=pos[s.fi];
                ans[s.se]=query(dfn[x]+sz[x]-1)-query(dfn[x]-1);
            }
        }
        else if (s[i]=='B') {
            add(dfn[now],-1);
            now=pre[now];
        }
        else {
            now=ch[now][s[i]-'a'];
            add(dfn[now],1);
        }
    }
    for (int i=1;i<=m;i++) printf("%d\n",ans[i]);
    return 0;
}
```

### CF1202E

#### 题目描述：

你有一个字符串 $t$ 和 $n$ 个字符串 $s_1,s_2,...,s_n$,所有字符串都只含有小写英文字母。

令 $f(t,s)$ 为 $s$ 在 $t$ 中的出现次数，如 $f('aaabacaa','aa')=3$,$f('ababa','aba')=2$.

请计算 $\sum_{i=1}^n\sum_{j=1}^nf(t,s_i+s_j)$,其中 $s+t$ 代表 $s$ 和 $t$ 连接起来。

注意如果存在两对整数 $i_1,j_1,i_2,j_2$,使 $s_{i_1}+s_{j_1}=s_{i_2}+s_{j_2}$,你需要把 $f(t,s_{i_1}+s_{j_1})$ 和 $f(t,s_{i_2}+s_{j_2})$ 加在答案里。

对于 $100\%$ 的数据：$1\le |t|\le 2\cdot 10^5$，$1\le n\le 2\cdot 10^5$，$1\le |s_i|\le 2\cdot 10^5$。

#### 题目分析：

既然 $s_i+s_j$ 在 $t$ 出现过，相当于有一个分割点 $i$，使得前一半的后缀中有 $s_i$，后一半的前缀中有 $s_j$，当然这个前缀可以通过把字符串反转然后成为后缀。

那么问题就变成了，对于每一个分割点 $i$，他的后缀中出现了多少个 $s$。

设 $f_i$ 表示 $1\sim i$ 这个字符串中后缀出现过多少个 $s$，其实就是 $i$ 这个点所对应的 `fail` 中有多少个点被标记过。

设 $g_i$ 表示 $i+1\sim n$ 这个字符串中前缀出现过多少个 $s$，把字符串反转之后和 $f$ 一样做即可。

那么答案即为：
$$
\sum_{i=1}^{n-1} f[i]\times g[n-i]
$$
复杂度 $O(n)$。

#### 代码：

```cpp
int m,f1[N],f2[N];
char t[N],s[N];
 
struct ACAM{
    int val[N],fail[N],ch[N][27],rt=1,cnt=1;
    void insert(char *s){
        int now=rt,n=strlen(s+1);
        for (int i=1;i<=n;i++){
            int c=s[i]-'a';
            if (!ch[now][c]) ch[now][c]=++cnt;
            now=ch[now][c];
        }
        val[now]++;
    }
    void build(){
        queue<int> q;
        for (int i=0;i<26;i++){
            if (ch[rt][i]) {
                fail[ch[rt][i]]=rt;
                val[ch[rt][i]]+=val[rt];
                q.push(ch[rt][i]);
            }
            else ch[rt][i]=rt;
        }
        while(!q.empty()){
            int now=q.front();q.pop();
            for (int i=0;i<26;i++){
                if (ch[now][i]){
                    fail[ch[now][i]]=ch[fail[now]][i];
                    val[ch[now][i]]+=val[fail[ch[now][i]]];
                    q.push(ch[now][i]);
                }
                else ch[now][i]=ch[fail[now]][i];
            }
        }
    }
    void init(char *s,int f[]){
        int now=rt,n=strlen(s+1);
        for (int i=1;i<=n;i++){
            int c=s[i]-'a';
            while (now&&!ch[now][c]) now=fail[now];
            if (ch[now][c]) now=ch[now][c];
            if (!now) now=rt;
            f[i]=val[now];
        }
    }
}AC,RAC;
 
signed main(){
    scanf("%s%lld",s+1,&m);
    int n=strlen(s+1);
    for (int i=1;i<=m;i++){
        scanf("%s",t+1);
        int len=strlen(t+1);AC.insert(t);
        reverse(t+1,t+len+1);RAC.insert(t);
    }
    AC.build(),RAC.build();
    AC.init(s,f1);reverse(s+1,s+n+1);RAC.init(s,f2);
    int ans=0;
    for (int i=1;i<n;i++){
        ans+=f1[i]*f2[n-i];
    }
    printf("%lld",ans);
    return 0;
}
```

### P3041

#### 题目描述：

Bessie 在玩一款游戏，该游戏只有三个技能键 `A`，`B`，`C` 可用，但这些键可用形成 $n$ 种特定的组合技。第 $i$ 个组合技用一个字符串 $s_i$ 表示。

Bessie 会输入一个长度为 $k$ 的字符串 $t$，而一个组合技每在 $t$ 中出现一次，Bessie 就会获得一分。$s_i$ 在 $t$ 中出现一次指的是 $s_i$ 是 $t$ 从某个位置起的连续子串。如果 $s_i$ 从 $t$ 的多个位置起都是连续子串，那么算作 $s_i$ 出现了多次。

若 Bessie 输入了恰好 $k$ 个字符，则她最多能获得多少分？

对于全部的测试点，保证：

- $1 \leq n \leq 20$，$1 \leq k \leq 10^3$。
- $1 \leq |s_i| \leq 15$。其中 $|s_i|$ 表示字符串 $s_i$ 的长度。
- $s$ 中只含大写字母 `A`，`B`，`C`。

#### 题目分析：

考虑每次加入一个字符，能产生的贡献，就是当前点的 `fail` 祖先中标记点的个数。

这里的标记点指的是这个点是一个串的尾结点。

那么设 $dp_{i,j}$ 表示现在填到了第 $i$ 位，最后在 AC 自动机的 $j$ 状态点，能取到的最大分数。

转移即为 $dp_{i+1,z}=max(dp_{i,j}+val_z)$，$val_x$ 为 $x$ 这个节点的 `fail` 祖先中标记点的个数，$z$ 为加入一个字符后转移的点。

答案即为：$\max\limits_{i=2}^{cnt}dp_{n,i}$（$cnt$ 为 AC 自动机中的节点个数，从 $2$ 开始是因为不算根节点）。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=3e3+7;

int dp[N][N],ch[N][27],fail[N],val[N];
int n,k,rt=1,cnt=1;
char s[N];

void insert(char *s){
    int n=strlen(s+1),now=rt;
    for (int i=1;i<=n;i++){
        int c=s[i]-'A';
        if (!ch[now][c]) ch[now][c]=++cnt;
        now=ch[now][c];
    }
    val[now]++;
}

void build(){
    queue<int> q;
    for (int i=0;i<3;i++){
        if (ch[rt][i]){
            fail[ch[rt][i]]=rt;
            val[ch[rt][i]]+=val[rt];
            q.push(ch[rt][i]);
        }
        else ch[rt][i]=rt;
    }
    while(!q.empty()){
        int now=q.front();q.pop();
        for (int i=0;i<3;i++){
            if (ch[now][i]){
                fail[ch[now][i]]=ch[fail[now]][i];
                val[ch[now][i]]+=val[fail[ch[now][i]]];
                q.push(ch[now][i]);
            }
            else ch[now][i]=ch[fail[now]][i];
        }
    }
}

signed main(){
    memset(dp,-0x3f,sizeof dp);
    scanf("%d%d",&n,&k);
    for (int i=1;i<=n;i++)
        scanf("%s",s+1),insert(s);
    build();
    dp[0][1]=0;
    for (int i=0;i<=k;i++){// 枚举做到第几位了
        for (int j=1;j<=cnt;j++){ // 枚举当前在 AC 自动机的哪个状态上
            for (int z=0;z<3;z++){ //枚举新加入的点是那一个
                int now=j;
                while(now&&!ch[now][z]) now=fail[now];
                if (ch[now][z]) now=ch[now][z];
                if (!now) now=rt;
                dp[i+1][now]=max(dp[i+1][now],dp[i][j]+val[now]);
            }
        }
    }
    int ans=0;
    for (int i=2;i<=cnt;i++) ans=max(ans,dp[k][i]);
    printf("%d\n",ans);
    return 0;
}
```

### P4052

#### 题目描述：

JSOI 交给队员 ZYX 一个任务，编制一个称之为“文本生成器”的电脑软件：该软件的使用者是一些低幼人群，他们现在使用的是 GW 文本生成器 v6 版。

该软件可以随机生成一些文章——总是生成一篇长度固定且完全随机的文章。 也就是说，生成的文章中每个字符都是完全随机的。如果一篇文章中至少包含使用者们了解的一个单词，那么我们说这篇文章是可读的（我们称文章 $s$ 包含单词 $t$，当且仅当单词 $t$ 是文章 $s$ 的子串）。但是，即使按照这样的标准，使用者现在使用的 GW 文本生成器 v6 版所生成的文章也是几乎完全不可读的。ZYX 需要指出 GW 文本生成器 v6 生成的所有文本中，可读文本的数量，以便能够成功获得 v7 更新版。你能帮助他吗？

答案对 $10^4 + 7$ 取模。

对于全部的测试点，保证：

- $1 \leq n \leq 60$，$1 \leq m \leq 100$。
- $1 \leq |s_i| \leq 100$，其中 $|s_i|$ 表示字符串 $s_i$ 的长度。
- $s_i$ 中只含大写英文字母。

#### 题目分析：

考虑能不能接着用上文的思路，如果当前点的 `fail` 祖先中至少有一个被标记过得点，那么就说明这个篇文章可读，但是你会发现一个非常难统计的地方，就是如果一个串两个位置串的后缀都出现过，那就没法去重了。

要用到一个非常典的性质，看到至少一个，可以想到用总数，减去一个都没有被标记过的。

那现在就好统计了，和上面那个题一样，如果当前点的 `val` 被标记过了，说明当前状态点不可选，转移即可。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=6007;
const int mod=1e4+7;

int ch[N][27],n,m,dp[107][N];
int rt=1,cnt=1,fail[N];
bool val[N];
char s[N];

void insert(char *s){
    int now=rt,n=strlen(s+1);
    for (int i=1;i<=n;i++){
        int c=s[i]-'A';
        if (!ch[now][c]) ch[now][c]=++cnt;
        now=ch[now][c];
    }
    val[now]=1;
}

void build(){
    queue<int> q;
    for (int i=0;i<26;i++){
        if (ch[rt][i]){
            fail[ch[rt][i]]=rt;
            val[ch[rt][i]]|=val[rt];
            q.push(ch[rt][i]);
        }
        else ch[rt][i]=rt;
    }
    while(!q.empty()){
        int now=q.front();q.pop();
        for (int i=0;i<26;i++){
            if (ch[now][i]){
                fail[ch[now][i]]=ch[fail[now]][i];
                val[ch[now][i]]|=val[fail[ch[now][i]]];
                q.push(ch[now][i]);
            }
            else ch[now][i]=ch[fail[now]][i];
        }
    }
}

int qmi(int a,int k){
    int res=1;
    for (;k;k>>=1,a=a*a%mod)
        if (k&1) res=res*a%mod;
    return res;
}

signed main(){
    scanf("%lld%lld",&n,&m);
    for (int i=1;i<=n;i++)
        scanf("%s",s+1),insert(s);
    build();
    dp[0][1]=1;
    for (int i=0;i<=m;i++){ //枚举填到第几位了
        for (int j=1;j<=cnt;j++){ //枚举当前在 AC 自动机的状态点
            if (val[j]) continue;
            for (int k=0;k<26;k++){
                int now=j;
                while(now&&!ch[now][k]) now=fail[now];
                if (ch[now][k]) now=ch[now][k];
                if (val[now]) continue;
                dp[i+1][now]=(dp[i+1][now]+dp[i][j])%mod;
            }
        }
    }
    int ans=qmi(26,m)%mod;
    for (int i=1;i<=cnt;i++) ans=((ans-dp[m][i])%mod+mod)%mod;
    printf("%lld\n",ans);
    return 0;
}
```

### P3311

#### 题目描述：

我们称一个正整数 $x$ 是幸运数，当且仅当它的十进制表示中不包含数字串集合 $s$ 中任意一个元素作为其子串。例如当 $s = \{22, 333, 0233\}$ 时，$233$ 是幸运数，$2333$、$20233$、$3223$ 不是幸运数。给定 $n$ 和 $s$，计算不大于 $n$ 的幸运数个数。

答案对 $10^9 + 7$ 取模。

对于全部的测试点，保证：

$1 \leq n < 10^{1201}$，$1 \leq m \leq 100$，$1 \leq \sum_{i = 1}^m |s_i| \leq 1500$，$\min_{i = 1}^m |s_i| \geq 1$，其中 $|s_i|$ 表示字符串 $s_i$ 的长度。$n$ 没有前导 $0$，但是 $s_i$ 可能有前导 $0$。

#### 题目分析：

发现这个题和上一个题是一样的，唯一的区别就是钦定了所有选出来的数 $\le n$，还有 $s_i$ 可能会出现前缀 $0$，有一个很好的性质就是本来我们就要 $dp$，然后现在钦定 $\le n$，那不就是让我们进行数位 $dp$ 吗，还可以一起处理了前缀 $0$ 的情况。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=2000+7;
const int mod=1e9+7;

int n,m,ch[N][11],fail[N];
int rt=1,cnt=1,dp[N][N][2][2];
char s[N],num[N];
bool val[N];

void insert(char *s){
    int now=rt,n=strlen(s+1);
    for (int i=1;i<=n;i++){
        int c=s[i]-'0';
        if (!ch[now][c]) ch[now][c]=++cnt;
        now=ch[now][c];
    }
    val[now]=1;
}

void build(){
    queue<int> q;
    for (int i=0;i<10;i++){
        if (ch[rt][i]){
            fail[ch[rt][i]]=rt;
            val[ch[rt][i]]|=val[rt];
            q.push(ch[rt][i]);
        }
        else ch[rt][i]=rt;
    }
    while(!q.empty()){
        int now=q.front();q.pop();
        for (int i=0;i<10;i++){
            if (ch[now][i]){
                fail[ch[now][i]]=ch[fail[now]][i];
                val[ch[now][i]]|=val[fail[ch[now][i]]];
                q.push(ch[now][i]);
            }
            else ch[now][i]=ch[fail[now]][i];
        }
    }
}

int dfs(int now,int pos,int limit,int pre_0){
    if (val[pos]) return 0;
    if (now==n+1){
        if (pre_0) return 0;
        return !val[pos];
    }
    if (~dp[now][pos][limit][pre_0]) return dp[now][pos][limit][pre_0];
    int p=limit?num[now]-'0':9,res=0;
    for (int i=0;i<=p;i++){
        if (!i&&pre_0) res=(res+dfs(now+1,pos,i==p&&limit,1))%mod;
        else{
            int u=pos;
            while(u&&!ch[u][i]) u=fail[u];
            if (ch[u][i]) u=ch[u][i];
            if (!u) u=1;
            res=(res+dfs(now+1,u,i==p&&limit,0))%mod;
        }
    }
    dp[now][pos][limit][pre_0]=res;
    return res;
}

signed main(){
    memset(dp,-1,sizeof dp);
    scanf("%s%lld",num+1,&m);n=strlen(num+1);
    for (int i=1;i<=m;i++) scanf("%s",s+1),insert(s);
    build();
    printf("%lld\n",dfs(1,1,1,1));
    return 0;
}
```

## SAM

在讲一些题之前，建议先去看一下[我之前的博客（重点看定义和性质）](https://blog.taozhiming.top/SAM/)，里面有一些 SAM 的性质和构造（当然构造你不想看也可以）。如果是从那边过来的，那下面的理解应该会清晰不少👍🏻。

首先声明，SAM 的构造不是必须要学的，你可以直接背下来板子，因为理解板子好像用处不大（？）。

下面 $3$ 个题所在的 OJ 已经寄了，所以需要选手自己写 `gen` 和 `std` 对拍。

### 重复旋律 5 | P2408

#### 题目描述：

求本质不同的子串的出现个数。

#### 题目分析：

后缀自动机的几个性质（之前博客里有，但是我还是重复一下）：

- 后缀自动机的状态点代表 `endpos` 相同的子串集合，后缀自动机可以接受一个字符串中所有子串。
- 后缀自动机中每个状态点之间的子串不交，这些字符串满足长度连续。
- 后缀自动机的 `fail` 指针指向的是他的后缀中最长的但是 `endpos` 和当前集合不同的第一个串。

由以上性质我们可以求出来一个状态中的连续的长度，即为 $[len_{fail_u}+1,len_{u}]$，所以其中子串的个数也就是 $len_u-(len_{fail_u+1}+1)$。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=2e5+7;

struct node{
    int nxt[27];
    int len,link;
}tr[N];
int cnt[N],lst,tot;
int n;char s[N];

void init(){tr[0].link=-1;}

void insert(int x){
    int p=lst,u=++tot;
    cnt[u]=1;tr[u].len=tr[lst].len+1;
    for (;~p&&!tr[p].nxt[x];p=tr[p].link) tr[p].nxt[x]=u;
    if (~p){
        int q=tr[p].nxt[x];
        if (tr[q].len==tr[p].len+1) tr[u].link=q;
        else{
            int t=++tot;
            copy(tr[q].nxt,tr[q].nxt+26,tr[t].nxt);
            tr[t].link=tr[q].link,tr[t].len=tr[p].len+1;
            for (;~p&&tr[p].nxt[x]==q;p=tr[p].link) tr[p].nxt[x]=t;
            tr[q].link=tr[u].link=t;
        }
    }
    lst=u;
}

signed main(){
    init();
    scanf("%lld%s",&n,s+1);
    for (int i=1;i<=n;i++) insert(s[i]-'a');
    int ans=0;
    for (int i=0;i<=tot;i++) ans+=tr[i].len-tr[tr[i].link].len;
    printf("%lld\n",ans);
    return 0;
}
```

### 重复旋律 6

#### 题目描述：

求出长度为 $1,2,3,...,n$ 的子串中，出现最多的子串次数。

#### 题目分析：

接着分析一些性质：

- 对于不同的状态点，他们内存的字符串也是不交的。
- 对于不同的 `endpos` 集合，只存在包含或者无交两种关系。

那么可以发现，如果我们知道 `endpos` 集合的大小，也就知道了这个子串的出现次数，两者是等价的。

那现在问题转化成了怎么求 `endpos` 集合的大小。

可以发现，在 `fail` 树中，$i$ 出现的位置 $fail_i$ 都会出现，因为 $fail_i$ 是 $i$ 的后缀，所以 $i$ 的 `endpos` 大小就是在 `fail` 树中所有儿子的 `endpos` 集合大小。

初始化非常简单，也就是每个前缀都设为 $1$。

其实上面的过程稍微思考一下就立马能觉得非常正确，并且也十分好记忆。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=1e5+7;

int n,tot,lst,sz[N],mx[N];
char s[N];bool flag[N];
vector<int> G[N];

struct node{
    int nxt[27];
    int link,len;
}tr[N];

void init(){tr[0].link=-1;}

void ins(int x){
    int p=lst,u=++tot;
    tr[u].len=tr[lst].len+1;flag[u]=1;
    for (;~p&&!tr[p].nxt[x];p=tr[p].link) tr[p].nxt[x]=u;
    if (~p){
        int q=tr[p].nxt[x];
        if (tr[q].len==tr[p].len+1) tr[u].link=q;
        else{
            int t=++tot;
            copy(tr[q].nxt,tr[q].nxt+26,tr[t].nxt);
            tr[t].len=tr[p].len+1;tr[t].link=tr[q].link;
            for (;~p&&tr[p].nxt[x]==q;p=tr[p].link) tr[p].nxt[x]=t;
            tr[u].link=tr[p].link=t;
        }
    }
    lst=u;
}

void dfs(int u){
    if (flag[u]) sz[u]=1;
    for (auto v:G[u]){
        dfs(v);
        sz[u]+=sz[v];
    }
}

signed main(){
    init();
    scanf("%s",s+1);n=strlen(s+1);
    for (int i=1;i<=n;i++) ins(s[i]-'a');
    for (int i=1;i<=tot;i++) G[tr[i].link].p_b(i);
    dfs(0);
    for (int i=0;i<=tot;i++) mx[tr[i].len]=max(mx[tr[i].len],sz[i]);
    for (int i=n;i>=1;i--) mx[i]=max(mx[i],mx[i+1]);
    for (int i=1;i<=n;i++) printf("%lld\n",mx[i]);
    return 0;
}
```

### 重复旋律 7

#### 题目描述：

给定若干个由数字组成的串，求所有子串所对应的数字的和。

例如 `123` 的结果就是：$1+2+3+12+23+123=164$。

#### 题目分析：

我们知道后缀自动机有以下性质：

- 由字符的转移所构成的图可以看做是一个有向无环图。

那么我们可以考虑在这个有向无环图上 $dp$ 求解。

设 $dp_{i}$ 表示以 $i$ 为终点的子串的数字和。因为后缀自动机可以容纳所有的子串，所以不会又漏解的情况。

转移：
$$
dp[i]\times 10+num[i]\times x \to dp[nxt[i][x]
$$
最后把每一个状态点的答案加起来即为答案。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=1e6+7;

int n,tot,lst,sz[N<<1],mx[N];
char s[N];bool flag[N<<1];
vector<int> G[N<<1];

struct node{
    int nxt[27];
    int link,len;
}tr[N<<1];

void init(){tr[0].link=-1;}

void ins(int x){
    int p=lst,u=++tot;
    tr[u].len=tr[lst].len+1;flag[u]=1;
    for (;~p&&!tr[p].nxt[x];p=tr[p].link) tr[p].nxt[x]=u;
    if (~p){
        int q=tr[p].nxt[x];
        if (tr[q].len==tr[p].len+1) tr[u].link=q;
        else{
            int t=++tot;
            copy(tr[q].nxt,tr[q].nxt+26,tr[t].nxt);
            tr[t].len=tr[p].len+1;tr[t].link=tr[q].link;
            for (;~p&&tr[p].nxt[x]==q;p=tr[p].link) tr[p].nxt[x]=t;
            tr[u].link=tr[q].link=t;
        }
    }
    lst=u;
}

void dfs(int u){
    if (flag[u]) sz[u]=1;
    for (auto v:G[u]){
        dfs(v);
        sz[u]+=sz[v];
    }
}

signed main(){
    init();
    scanf("%s",s+1);n=strlen(s+1);
    for (int i=1;i<=n;i++) ins(s[i]-'a');
    for (int i=1;i<=tot;i++) G[tr[i].link].p_b(i);
    dfs(0);
    for (int i=0;i<=tot;i++) mx[tr[i].len]=max(mx[tr[i].len],sz[i]);
    for (int i=n-1;i>=1;i--) mx[i]=max(mx[i],mx[i+1]);
    for (int i=1;i<=n;i++) printf("%lld\n",mx[i]);
    return 0;
}
```

### 重复旋律 8

#### 题目描述：

给定 $n$ 个字符串 $t$ 和一个字符串 $s$，求 $t$ 的所有循环同构在 $s$ 中的出现次数，出现多次则重复计算。

循环同构例子： $csp$ 的循环同构分别是 $csp、spc、pcs$。

#### 题目分析：

因为一个循环同构你去枚举的话，那时间复杂度就裂开了，有一个很好的方法可以有效避免，就是把这个字符复制一遍，只要选取的长度为原串的长度，那么便是原串的一个循环同构。

如果想求一个串在 $s$ 中出现了几次，需要记录 `endpos` 的集合大小。

对于一个串，如果加入一个字符 $t_i$，那么会跳到 $nxt[now][t_i]$，为了求出他的出现次数并且使得当前串的长度仍然大于原串的长度，那么我们可以一直跳 `link`，找到之后直接把 `endpos` 大小加入答案即可，哪怕长度大于原串的长度，但是这个串的出现次数一定和后缀的出现次数一样，所以不用担心统计错了。

**注意**几个细节：

1. 一个状态的 `endpos` 大小只能被统计一次，比如 `aaaa`，他的循环同构全是 `aaaa`，但是只会统计一次答案。
2. 我们在匹配的时候出现当前状态点的 $len\ge$ 原串长度是，要找的循环同构就是当前串的后缀，所以在保证长度 $\ge$ 原串的同时，不断的跳 `link`，使得状态点的 $len$ 在合法的情况下尽可能小，如果不往上跳的话，会有别的地方没有被统计到而导致答案出错。

可以发现时间复杂度是 $O(|t|+|s|)$。

#### 代码：

```cpp
const int INF=1e9+7;
const int N=1e5+7;

int n,sz[N<<1],lst,tot;
char s[N],t[N];
bool flag[N<<1],vis[N<<1];
vector<int> G[N<<1];

struct node{
    int nxt[27];
    int link,len;
}tr[N<<1];

void init(){tr[0].link=-1;}

void ins(int x){
    int p=lst,u=++tot;
    tr[u].len=tr[lst].len+1;flag[u]=1;
    for (;~p&&!tr[p].nxt[x];p=tr[p].link) tr[p].nxt[x]=u;
    if (~p){
        int q=tr[p].nxt[x];
        if (tr[q].len==tr[p].len+1) tr[u].link=q;
        else{
            int t=++tot;tr[t].len=tr[p].len+1;
            copy(tr[q].nxt,tr[q].nxt+26,tr[t].nxt);
            tr[t].link=tr[q].link;
            for (;~p&&tr[p].nxt[x]==p;p=tr[p].link) tr[p].nxt[x]=t;
            tr[q].link=tr[u].link=t;
        }
    }
    lst=u;
}

void dfs(int u){
    if (flag[u]) sz[u]=1;
    for (auto v:G[u]){
        dfs(v);
        sz[u]+=sz[v];
    }
}

signed main(){
    init();
    scanf("%s",s+1);
    n=strlen(s+1);
    for (int i=1;i<=n;i++) ins(s[i]-'a');
    for (int i=0;i<=tot;i++) G[tr[i].link].p_b(i);
    dfs(0);
    scanf("%d",&n);
    while(n--){
        int ans=0;memset(vis,0,sizeof vis);
        scanf("%s",t+1);int m=strlen(t+1);
        for (int i=m+1;i<2*m;i++) t[i]=t[i-m];
        int now=0,len=0;
        for (int i=1;i<2*m;i++){
            int c=t[i]-'a';
            while (now&&!tr[now].nxt[c]){
                now=tr[now].link;len=tr[now].len;
            }
            if (tr[now].nxt[c]) now=tr[now].nxt[c],len++;
            else now=0,len=0;
            if (len>m){
                while(tr[tr[now].link].len>=m)
                    now=tr[now].link,len=tr[now].len;
            }
            if (len>=m&&!vis[now]){ans+=sz[now];vis[now]=1;}
        }
        printf("%d\n",ans);
    }
    return 0;
}
```

