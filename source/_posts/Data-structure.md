---
title: 数据结构
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2026-11-11 21:26:10
categories: 学习笔记
tags:
  - 
---

数据结构详解

<!-- MORE -->

## 栈

**核心思想：先进先出**

可以用数组模拟，也可以直接使用 `STL` 中的 `spnpm exec hexo cleantack`。

### 例题

#### 题意：

给定一个表达式，其中运算符仅包含 +,-,*,/（加 减 乘 整除），可能包含括号，请你求出表达式的最终值。

注意：

- 数据保证给定的表达式合法。
- 题目保证符号 `-` 只作为减号出现，不会作为负号出现，例如，`-1+2,(2+2)*(-(1+1)+2)` 之类表达式均不会出现。
- 题目保证表达式中所有数字均为正整数。
- 题目保证表达式在中间计算过程以及结果中，均不超过 $2^{31}−1$。
- 题目中的整除是指向 $0$ 取整，也就是说对于大于 $0$ 的结果向下取整，例如 `5/3=1`，对于小于 $0$ 的结果向上取整，例如 `5/(1−4)=−1`。

#### 分析：

如果是表达式的后端遍历，直接进行操作即可。

但是中序遍历，就需要栈来把表达式树上的操作符记录下来，左右两边子树都计算完之后再进行合并计算。

#### 代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

stack<int> num;
stack<char> op;

void eval(){
    auto b=num.top();num.pop();
    auto a=num.top();num.pop();
    auto c=op.top();op.pop();
    int x;
    if (c=='+') x=a+b;
    else if (c=='-') x=a-b;
    else if (c=='*') x=a*b;
    else x=a/b;
    num.push(x);
}

signed main(){
    map<char,int> pr{{'+',1},{'-',1},{'*',2},{'/',2}};
    string str;
    cin>>str;
    for (int i=0;i<str.size();i++){
        auto c=str[i];
        if (isdigit(c)){
            int x=0,j=i;
            while(j<str.size()&&isdigit(str[j])){
                x=x*10+str[j++]-'0';
            }
            i=j-1;
            num.push(x);
        }
        else if (c=='(') op.push(c);
        else if (c==')'){
            while(op.top()!='(') eval();
            op.pop();
        }
        else{
            while(op.size()&&op.top()!='('&&pr[op.top()]>=pr[c]) eval();
            op.push(c);
        }
    }
    while(op.size()) eval();
    printf("%d\n",num.top());
    return 0;
}
```

## 单调栈

### 例题

#### 题意：

给定一个长度为 $N$ 的整数数列，输出每个数左边第一个比它小的数，如果不存在则输出 $−1$。

#### 分析：

从暴力枚举出发，对于一个 $i$，从 $i$ 往前枚举第一个小于 $a[i]$ 的数。

考虑优化，显然枚举 $i$ 的这一步是不可能优化的，转向 $j$ 的优化。

寻找性质：$x<y$ 且 $a[x]\ge a[y]$，那么 $x$ 不可能作为答案出现。

因此我们只需要维护一个上升的数组即可。

#### 代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N=1e5+7;

int stk[N],n,a[N];

signed main(){
    scanf("%d",&n);
    for (int i=1;i<=n;i++) scanf("%d",&a[i]);
    int tail=0;
    for (int i=1;i<=n;i++){
        while(stk[tail]>=a[i]) tail--;
        if (!tail) printf("-1 ");
        else printf("%d ",stk[tail]);
        stk[++tail]=a[i];
    }
    return 0;
}
```

## 滑动窗口

### 例题

#### 题意：

给定一个大小为 $n\le 106$ 的数组。

有一个大小为 $k$ 的滑动窗口，它从数组的最左边移动到最右边。

你只能在窗口中看到 $k$ 个数字。

每次滑动窗口向右移动一个位置。

#### 分析：

$\circ$ 先用普通队列的方法，进去一个出去一个，把整个队列枚举一遍求最大和最小值

$\circ$ 考虑优化队列，删去无用元素，比如求最小值时，如果 $i<j,a[i]>a[j]$，那么 $i$ 一定不会作为最小值被输出。

$\circ$ 如此优化，队列出现了单调性，只需要输出队头/尾即可，$O(n)$ 优化为 $O(1)$。

#### 代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N=1e6+7;

int n,k,a[N];
int q[N],hh,tt;

signed main(){
    scanf("%d%d",&n,&k);
    for (int i=1;i<=n;i++) scanf("%d",&a[i]);
    hh=0,tt=-1;
    for (int i=1;i<=n;i++){
        if (hh<=tt&&q[hh]<i-k+1) hh++;
        while(hh<=tt&&a[q[tt]]>a[i]) tt--;
        q[++tt]=i;
        if (i>k-1) printf("%d ",a[q[hh]]);
    }
    printf("\n");
    hh=0,tt=-1;
    for (int i=1;i<=n;i++){
        if (hh<=tt&&q[hh]<i-k+1) hh++;
        while(hh<=tt&&a[q[tt]]<a[i]) tt--;
        q[++tt]=i;
        if (i>k-1) printf("%d ",a[q[hh]]);
    }
    return 0;
}
```

## KMP

### 算法过程

[请看VCR，讲的很不错](https://www.bilibili.com/video/BV1AY4y157yL/?spm_id_from=333.337.search-card.all.click&vd_source=d375755009e3efb9201b1c57645b3f1d)

### 例题：

#### 题意：

给定一个字符串 $S$，以及一个模式串 $P$，所有字符串中只包含大小写英文字母以及阿拉伯数字。

模式串 $P$ 在字符串 $S$ 中多次作为子串出现。

求出模式串 $P$ 在字符串 $S$ 中所有出现的位置的起始下标。

#### 代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N=1e6+7;

int n,m,nxt[N];
char s[N],t[N];

signed main(){
    scanf("%d %s %d %s",&n,t+1,&m,s+1);
    for (int i=2,j=0;i<=n;i++){
        while(j&&t[i]!=t[j+1]) j=nxt[j];
        if (t[i]==t[j+1]) j++;
        nxt[i]=j;
    }
    for (int i=1,j=0;i<=m;i++){
        while(j&&s[i]!=t[j+1]) j=nxt[j];
        if (s[i]==t[j+1]) j++;
        if (j==n){
            printf("%d ",i-j+1);
            j=nxt[j];
        }
    }
    return 0;
}
```



