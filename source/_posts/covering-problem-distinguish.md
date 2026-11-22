---
title: 精确/重复覆盖问题区分
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-06-24 16:54:20
categories: 学习笔记
tags:
  - DLX
---

区分一下精准覆盖和重复覆盖问题的区别，和两个算法的实现过程

<!-- MORE -->

用的全都是 DLX （好像只能用 DLX？我也不知道）

没想到一个小小的精确覆盖问题和重复覆盖问题，需要注意的不同点这么重要！



# 精确覆盖问题

## 怎么实现？

1 找到当前没有被删掉的列中 1 的个数最小的一列

2 把这一列删掉，这里删掉是指先把当前的点的左右链表删掉，然后再把这个点所在的列的点的上下链表删掉

3 枚举这一列的每个 $1$，计入答案，并把这一列中有 $1$ 的行中有 $1$ 的列全部删掉

   删除的方法就是把每个 1 的点的**上下**链表给删掉，这样下次遍历到当前列，从上往下遍历的时候就不会把这个点遍历到

4 继续递归

5 恢复 $3$ 操作

6 恢复 $2$ 操作

7 删除答案

8 如果还是找不到答案，说明无解

## 代码实现（注意注释的地方）

```cpp
#include <map>
#include <set>
#include <queue>
#include <cmath>
#include <ctime>
#include <stack>
#include <random>
#include <vector>
#include <cstdio>
#include <sstream>
#include <cstdlib>
#include <iomanip>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <unordered_set>
#include <unordered_map>

#define fi first
#define se second
#define LL long long
#define m_p make_pair
#define p_b emplace_back
#define lowbit(x) x&(-x)
#define PII pair<int,int>
#define all(x) x.begin(),x.end()
using namespace std;

inline int read(){
	int x=0,w=1;
	char ch=getchar();
	for(;ch>'9'||ch<'0';ch=getchar()) if(ch=='-') w=-1;
	for(;ch>='0'&&ch<='9';ch=getchar()) x=x*10+ch-'0';
	return x*w;
}
inline void write(int x){
	if(x<0) putchar('-'),x=-x;
	if(x>9) write(x/10);
	putchar(x%10+'0');
}

const int N=5510+10;
const int INF=1e9+7;

int n,m;
int l[N],r[N],u[N],d[N],s[N],row[N],col[N],cnt;
int ans[N],top;

void init()
{
	for (int i=0;i<=m;i++)
	{
		l[i]=i-1,r[i]=i+1;
		u[i]=d[i]=i;
	}
	l[0]=m,r[m]=0;
	cnt=m+1;
}

void add(int& hh,int& tt,int x,int y)
{
	row[cnt]=x,col[cnt]=y,s[y]++;
	u[cnt]=y,d[cnt]=d[y],u[d[y]]=cnt,d[y]=cnt;
	r[hh]=l[tt]=cnt,r[cnt]=tt,l[cnt]=hh;
	tt=cnt++;
}

void remove(int p)
{
	r[l[p]]=r[p],l[r[p]]=l[p];
	for (int i=d[p];i!=p;i=d[i])
		for (int j=r[i];j!=i;j=r[j])
		{
			s[col[j]]--;
			d[u[j]]=d[j],u[d[j]]=u[j]; // 这里的删除是把上下链表删除
		}
}

void resume(int p)
{
	for (int i=u[p];i!=p;i=u[i])
		for (int j=l[i];j!=i;j=l[j])
		{
			d[u[j]]=j,u[d[j]]=j;
			s[col[j]]++;
		}
	r[l[p]]=p,l[r[p]]=p;
}

bool dfs()
{
	if (!r[0]) return 1;
	int p=r[0];
	for (int i=r[0];i;i=r[i])
		if (s[i]<s[p])
			p=i;
	remove(p);
	for (int i=d[p];i!=p;i=d[i])
	{
		ans[++top]=row[i];
		for (int j=r[i];j!=i;j=r[j]) remove(col[j]);
		if (dfs()) return 1;
		for (int j=l[i];j!=i;j=l[j]) resume(col[j]);
		top--;
	}
	resume(p);
	return 0;
}

int main(){
	cin>>n>>m;
	init();
	for (int i=1;i<=n;i++)
	{
		int hh=cnt,tt=cnt;
		for (int j=1;j<=m;j++)
		{
			int x;
			scanf("%d",&x);
			if (x) add(hh,tt,i,j);
		}
	}
	if (dfs())
		for (int i=1;i<=top;i++) cout<<ans[i]<<" ";
	else cout<<"No Solution!";
	return 0;
}
/*
--to taozhiming--
0.不要犯愚蠢的错误！
1.数组大小是否足够？数组大小是否足够？数组大小是否足够？整数溢出？
2.你的算法有反例吗？
3.时间复杂性？内存使用？精度误差？
4.是否删除调试？
5.考试的时候你是不是要摆烂？
6.题读对了吗
*/
```



# 重复覆盖问题

## 怎么实现？

1 用个 IDA* 来剪枝（具体的我就不说了）

2 找到最少 $1$ 的列

3 枚举当前列的每一个 $1$ 

4 删除掉当前列 **注意：这里没有删掉枚举到的点** 

为什么不删掉枚举到的点？**如果删掉了枚举到的点，也就是说 $i$ 这个点的左右都没了，那下面的枚举这么办？所以说不能删掉当前枚举到的点** 

5 计入答案

6 把当前行的 $1$ 所在的列的每个点的**左右**链表删掉

为什么删左右链表？因为当前列已经有一个 $1$ 了，所以说这一列的点不需要选了

为什么不删**上下**链表？**也是因为删掉了上下链表就没法恢复了** 

7 继续递归

8 恢复 $6$ 操作

9 恢复 $4$ 操作

## 代码实现（同样注意注释的地方）

```cpp
#include <map>
#include <set>
#include <queue>
#include <cmath>
#include <ctime>
#include <stack>
#include <random>
#include <vector>
#include <cstdio>
#include <sstream>
#include <cstdlib>
#include <iomanip>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <unordered_set>
#include <unordered_map>

#define fi first
#define se second
#define LL long long
#define m_p make_pair
#define p_b push_back
#define lowbit(x) x&(-x)
#define PII pair<int,int>
#define all(x) x.begin(),x.end()
using namespace std;

inline int read(){
	int x=0,w=1;
	char ch=getchar();
	for(;ch>'9'||ch<'0';ch=getchar()) if(ch=='-') w=-1;
	for(;ch>='0'&&ch<='9';ch=getchar()) x=x*10+ch-'0';
	return x*w;
}
inline void write(int x){
	if(x<0) putchar('-'),x=-x;
	if(x>9) write(x/10);
	putchar(x%10+'0');
}

const int N=10000+10;
const int INF=1e9+7;

int n,m;
int l[N],r[N],u[N],d[N],s[N],row[N],col[N],cnt;
int ans[N],top;
bool vis[N];

void init()
{
	for (int i=0;i<=m;i++)
	{
		l[i]=i-1,r[i]=i+1;
		col[i]=u[i]=d[i]=i;
		s[i]=0;
	}
	l[0]=m,r[m]=0;
	cnt=m+1;
}

void add(int &hh,int &tt,int x,int y)
{
	row[cnt]=x,col[cnt]=y,s[y]++;
	u[cnt]=y,d[cnt]=d[y],u[d[y]]=cnt,d[y]=cnt;
	r[hh]=l[tt]=cnt,l[cnt]=hh,r[cnt]=tt;
	tt=cnt++;
}

int h()
{
	int tot=0;
	memset(vis,0,sizeof vis);
	for (int i=r[0];i;i=r[i])
	{
		if (vis[i]) continue;
		vis[i]=1;
		tot++;
		for (int j=d[i];j!=i;j=d[j])
			for (int k=r[j];k!=j;k=r[k])
				vis[col[k]]=1;
	}
	return tot;
}

void remove(int p)
{
	for (int i=d[p];i!=p;i=d[i])
		l[r[i]]=l[i],r[l[i]]=r[i]; //注意这里是删的是左右链表
}

void resume(int p)
{
	for (int i=u[p];i!=p;i=u[i])
		l[r[i]]=i,r[l[i]]=i;
}

bool dfs(int k,int depth)
{
	if (k+h()>depth) return 0;
	if (!r[0]) return 1;
	int p=r[0];
	for (int i=r[0];i;i=r[i])
		if (s[i]<s[p])
			p=i;
	for (int i=d[p];i!=p;i=d[i])
	{
		remove(i);
		ans[k]=row[i];
		for (int j=r[i];j!=i;j=r[j]) remove(j);
		if (dfs(k+1,depth)) return 1;
		for (int j=l[i];j!=i;j=l[j]) resume(j);
		resume(i);
	}
	return 0;
}

int main(){
	cin>>n>>m;
	init();
	for (int i=1;i<=n;i++)
	{
		int hh=cnt,tt=cnt;
		for (int j=1;j<=m;j++)
		{
			int x;
			scanf("%d",&x);
			if (x) add(hh,tt,i,j);
		}
	}
	int depth=0;
	while(!dfs(0,depth)) depth++;
	printf("%d\n",depth);
	for (int i=0;i<depth;i++) cout<<ans[i]<<" ";
	return 0;
}
/*
--to taozhiming--
0.不要犯愚蠢的错误！
1.数组大小是否足够？数组大小是否足够？数组大小是否足够？整数溢出？
2.你的算法有反例吗？
3.时间复杂性？内存使用？精度误差？
4.是否删除调试？
5.考试的时候你是不是要摆烂？
6.题读对了吗
*/
```

# 现在我们回过头来，想想变化最大的是什么？

就是我们删的是上下链表还是左右链表对吧

删除方式不一样的原因是目的不同：

**精确覆盖问题删除时，是为了删掉某一行，而删掉某一行的方法就是让上面的点直接遍历到下面的点，所以精确覆盖问题的删除是删上下链别**

**重复覆盖问题删掉时，是因为这一列已经有一个 $1$ 了，所以这一列别的有没有 $1$ 已经无所谓了，所以直接把这一列删掉，删除一列的方法同上可得删除左右链就OK**

