---
title: Fib 定理证明
# password: 1307
# abstract: 有东西被加密了, 请输入密码查看。
# message: 有东西被加密了, 请输入密码查看。(号爸 13 楼门牌号)
# wrong_pass_message: 这个密码不太对捏๑_๑
# wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
date: 2023-11-25 11:14:21
categories: 学习笔记
tags:
  - Fib
index_img:
top:
sticky:
---

闲着没事，写个定理证明。

<!-- MORE -->

在 HB 的时候，最后几天用到了这个定理，我也不知道怎么证明，wjc 老哥讲了一遍，我没听太懂。

没想到回到学一下问了下学长，学长给了个证明/se

证明：$f_{n,m}=f_{\gcd(n,m)}$。

在证明这个之前，要先证明另外一个式子：$f_{n}\times f_{m} + f_{n-1}\times f_{m-1} = f_{n+m-1}$。

这里用数学归纳法证明：

假设 $n$ 是个定值，

1. 当 $m=1$ 时，$f_n=f_n$。
2. 当 $m=2$ 时，$f_n+f_{n-1}=f_{n+1}$。

假设当 $m\le k$ 时，式子都成立，那么看看能不能推出 $m=k+1$ 时成立：
$$
f_n\times f_{k+1}+f_{n-1}\times f_{k} \\\\
f_n\times(f_k+f_{k-1}) + f_{n-1}\times(f_{k-1}+f_{k-2}) \\\\
f_n\times f_k+ f_n\times f_{k-1} + f_{n-1}\times f_{k-2}+f_{n-1}\times f_{k-1}\\\\
f_{n+k-1}+f_{n+k-2}=f_{n+k}
$$
归纳可得：$f_{n}\times f_{m} + f_{n-1}\times f_{m-1} = f_{n+m-1}$。

然后观察 $\gcd(f_n,f_m)$ 其实就等于 $\gcd(f_{n-m},m)$，因为：
$$
\gcd(f_n,f_m) =\gcd(f_{(n-m)+m},f_m) \\\\
=\gcd(f_{n-m}\times f_{m+1} + f_{n-m-1}\times f_{m},f_m) \\\\
=\gcd(f_{n-m}\times f_{m+1} ,f_m)
$$
因为 $f_{m+1}$ 和 $f_m$ 互质，所以最后等于 $\gcd(f_{n-m},f_m)$。

发现 $\gcd(f_{n},f_m)=\gcd(f_{n-m},f_m)$ 就是一个 $n$ 和 $m$ 求 $\gcd$ 的过程，所以可以得到 $\gcd(f_n,f_m)=f_{\gcd(n,m)}$。

