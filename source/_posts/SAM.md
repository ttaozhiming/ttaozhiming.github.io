---
title: SAM（后缀自动机）
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-08-07 20:13:10
categories: 学习笔记
tags:
  - SAM
---

未完待续...

<!-- MORE -->

## SAM

之前听 yxc 讲的时候没有细致的讲是如何构造的，今天难得听到了 yny 讲的 SAM，真的很细致！（当然我可能写不了这么细致）。



### 定义

对于所有 $endpos$ 集合相同的子串，将其压缩到一个点上（记住定义，一个点指的是 $endpos$ 的集合）。



**endpos** ：一个串的 $endpos$ 是他这个串的右端点所在的位置。

例：$ababa$ 中 $aba$ 的 $endpos$ 为 $3$ 和 $5$。

**link** ：假设现在有两个点，分别是 $A$ 和 $B$ ，使得 $A\subsetneqq B$ ，并且 $B$ 的集合大小最小，则 $link[A]=B$。

**nxt** ：$nxt[x][c]$ 表示在 $x$ 这个点，若在其中的每一个字母后加上 $c$，组成的新字符串所对应的 $endpos$ 集。

**last** ：整个串的 $endpos$ 所对应的点。

**len** ：$len[x]$ 表示 $x$ 这个点中最长的串的长度。

注意：如果一个点没有他的真超集 ，那么他的 $link$ 为 $0$。

### 性质

1. 对于每个点，endpos 所对应的串的长度一定是连续的。

   证明：假设一个串 $C$ ，选两个两个后缀 $A$ 和 $B$ ，使得 $B$ 的长度小于 $A$  ，且 $C$ 和 $A$ 的 $endpos$ 都在同一个点中，因为一个串他的串长越短，他所对应的 $endpos$ 就越多，所以 $C$ 的 $endpos$ 数小于等于 $B$ 的 $endpos$ 数，且 $B$ 的 $endpos$ 数小于等于 $A$ 的 $endpos$ 数，又因为 $A$ 和 $C$ 的 $endpos$ 相同，所以 $B.endpos=A.endpos=C.endpos$ ，所以 $B$ 也在 $A$ 所在的点中。

2. $nxt[x][c]$ 中的每一个串后加上 $c$ 后的 $endpos$ 所对应的一定是同一个点，找到 $x$ 这个点中长度最长的那个串，那么剩余的串一定是他的后缀，假设这个点后面有 $c$ ，那么他的后缀后面也全都有 $c$ 这个点，如果没有，那后缀后面也不会有，所以最后的 $endpos$ 一定是一样的。

3. 可以发现每次跳 $link$ 都相当于跳到了一个串的后缀，假设这个后缀的点在原串的位置为 $x$ ，那么 $1 \sim x-1$ 这些串都和 $1$ 这个串在同一个点中。

4. 同时可以发现，一个点（假设为 $x$ ）中的最短子串长度是 $len[link[x]]-1$。

### 构造

假设现在要新加入一个点 $c$ ，新建一个节点 $u$ ，表示新的长度为 $n+1$ 的串 $endpos$ 所对应的点。

设 $p=last$ （$last$ 上面有定义），我们不停地让 $p$ 跳 $link$ ，现在出现了两种情况：

1. $nxt[p][c]=0$ 也就是说在整个串中没有出现过 串+$c$ 这个子串，那么 $nxt[p][c]$ 可以直接指向 $u$。

2. 如果 $nxt[p][c]!=0$ ，设 $q=nxt[p][c]$ ，当然这里也需要分成两类讨论。

   1. 若 $len[q]=len[p]+1$ ，那么也就是说 $p$ + $c$ 这个点后 和 $q$ 这两个串完全相同，那么 $q$ 这个点的 $endpos$ 就加入了 $n+1$ 这个数，可以发现现在的 $q$ 这个点就是 $u$ 的 $link$ ，因为这个点就是目前 $endpos$ 个数最小，并且是 $u$ 的真超集，如果再跳 $link$ ，那么后缀的长度变短，找到的 $endpos$ 的个数会不降，所以 $link[u]=q$。

   2. 若不等，那么我们可以发现，$q$ 这个点中串长度小于等于 $len[p]+1$ 的串的 $endpos$ 都会新增一个 $n+1$ ，但是长度大于 $len[p]+1$ 的并不会新增 $endpos$ ，所以这里我们就被迫把一个点分裂成两个，一个给 $p$ ，一个给 $q$ 中长度大于 $len[p]+1$ 的串，我们新建一个节点 $t$ 来作为分裂成的第一个节点，

      考虑这里如何转移，假如当前需要找到 $nxt[q][y]$ 的值，那么发现在 $n+1$ 这个位置后面是没有 $y$ 这个字符的，所以说 $n+1$ 这个 $endpos$ 是无法转移到 $n+2$ 这个点的，当我们要找 $nxt[q][y]$ 时，我们的两个点就合并成原先的一个点了，因此 $t$ 这个点的 $nxt$ 要继承 $q$ 的 $nxt$。

      可以发现，原先 $q$ 的 $link$ 变成了 $t$ 的 $link$ ，并且 $len[t]=len[p]+1$。

      但是 $p$ 的每一个后缀的 $endpos$ 都多了一个 $n+1$ ，那么如果有原先的 $nxt[p][c]=q$ ，那么现在多了一个 $endpos$ ，那么 $nxt[p][c]=t$。

      最后 $q$ 和 $u$ 的 $link$ 都是 $t$。

3. 如果 $p$ 在循环中跳到了 $0$ ，说明 $c$ 这个字符就没在原串中出现过，所以 $link[u]$ 应该为 $0$ ，但是发现 $link[u]$ 本来就是 $0$ ，所以可以直接不管。

最后贴上一个代码（对着理解一下）：

```cpp
struct node{
    int nxt[27];
    int len,link;
}tr[N];
int cnt[N],lst,tot;

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
```
