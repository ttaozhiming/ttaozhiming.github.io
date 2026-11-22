---
title: 欧拉函数
# password: muyiio
# abstract: 这里有东西被加密了，需要输入密码查看哦。
# message: 您好，这里需要密码。
# wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-07-30 20:59:10
categories: 学习笔记
tags:
  - 欧拉函数
---

欧拉筛的推导，和一些欧拉函数的性质

<!-- MORE -->

欧拉函数的几个性质（以下 $p$ 无特殊说明，都为质数）：

1. 若 $gcd(a,b)=1$ ，则 $\varphi(a)\times \varphi(b)$，证明不会，大家记住就行了

2. $n=\sum_{d|n}\varphi(d)$ 

   证明：设 $f(x)$ 表示 $gcd(k,n)=x$ 的数的个数（ $k<=n$ ）

   结论：$n=\sum_{i=1}^nf(i)$

   我们可以分类讨论（保证 $x<=n$）：

   1. $x$ 是 $n$ 的约数，那么 $gcd(x,n)=x$ ，可以有 $1$ 的贡献
   2. $x$ 不是 $n$ 的约数，$x$ 也和 $n$ 不互质，那么一定会被他们俩的最大公约数带来 $1$ 的贡献
   3. $x$ 和 $n$ 互质，那么会被 $gcd(x,n)=1$ 带来 $1$ 的贡献

   综上：每个数 $x$ 都会且只会被统计一次 $1$ 的贡献

   证出结论：$n=\sum_{i=1}^nf(i)$

   现在我们观察上面的式子：$gcd(k,n)=d$ 等价于 $gcd(\frac{k}{d},\frac{n}{d})=1$ 也就等价于 $\varphi(\frac{n}{d})$

   所以上面的结论可以转化成：$n=\sum_{d|n}\varphi(\frac{n}{d})=\sum_{d|n}\varphi(d)$

3. 若 $n=p^k$ ，则 $\varphi(n)=p^k-p^{k-1}$

   可以看出用到了容斥，用总的数量，减去和 $n$ 不互质的数量。

   法一：设 $x$ 为 $p$ 的系数，则 $x$ 的取值范围为 $1,2,3,...,p^{k-1}$ ，所以总共有 $p^{k-1}$ 个与 $n$ 不互质的数

   法二：$p^k/p$ 就是 $p$ 的倍数在 $1-n$ 中出现的个数，即 $p^{k-1}$

4. $\varphi(p^k)=p^{k-1}\times(p-1)$

   这个证明是简单的

   上面已经得出 $\varphi(p^k)=p^k-p^{k-1}$ 我们把 $p^{k-1}$ 提出来

   可以得到 $\varphi(p^k)=p^{k-1}\times(p-1)$

5. $\varphi(n)=n*\prod_{i=1}^c\frac{p_i-1}{p_i}$ （ $c$ 是 $n$ 的质因数个数）

   证明：

   已知 $n=\prod_{i=1}^c \varphi(p_i^{k_i})$ （证明：由 $1$ 性质可知）

   则 $n=\prod_{i=1}^c (p^k-p^{k-1})$ 这次我们提出来 $p^k$ 

   式子就变成 $n=\prod_{i=1}^c(p^k\times(1-\frac{1}{p}))$

了解了这些性质以后，我们就可以愉快的进行欧拉筛了

分类讨论：

1. 若 $p$ 为素数，则 $\varphi(p)=p-1$

2. 发现 $n$ 会被 $p\times n'$ 筛掉

   1. $p$ 是 $n'$ 的最小质因数，$\varphi(n)=p\times \varphi(n')$

      证明：

      $\varphi(n)=n\times\prod_{i=1}^c\frac{p_i-1}{p_i}$

      把 $n$ 分解成 $p\times n'$  可知 $\varphi(n)=p\times n'\times\prod_{i=1}^c\frac{p_i-1}{p_i}$

      把后两项合并得： $\varphi(n)=p\times \varphi(n')$

   2. $p$ 不是 $n'$ 的最小质因数

      则 $\varphi(n)=\varphi(n')\times p$

      即 $\varphi(n)=\varphi(n')\times(p-1)$

到这里，欧拉筛就结束了，花了一晚上时间，一个题没调，但是感觉理解的透彻多了！

给个代码，大家可以理解一下：

```cpp
void init()
{
	phi[1]=1;
	for (int i=2;i<N;i++)
	{
		if (!vis[i]) pr[++cnt]=i,phi[i]=i-1;
		for (int j=1;pr[j]*i<N;j++)
		{
			vis[pr[j]*i]=1;
			if (i%pr[j]==0)
			{
				phi[pr[j]*i]=phi[i]*pr[j];
				break;
			}
			phi[pr[j]*i]=phi[i]*(pr[j]-1);
		}
	}
	for (int i=1;i<N;i++) sum[i]=sum[i-1]+phi[i];
}
```
