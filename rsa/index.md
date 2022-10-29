# RSA算法的数学原理及证明


### 概述
&emsp;&emsp;RSA是一种广泛使用的非对称加密算法，由由罗纳德·李维斯特（Ron Rivest）、阿迪·萨莫尔（Adi Shamir）和伦纳德·阿德曼（Leonard Adleman）在1977年一起提出的，RSA就是他们三人姓氏头字母组成的。

&emsp;&emsp;对极大整数做因式分解的难度决定了RSA算法的可靠性，目前被破解的最长的RSA密钥为829位。只要密钥的长度足够长，用RSA加密的信息可以认为是无法破解的。下面将介绍与RSA算法相关的数学概念，RSA的算法流程及正确性证明。

### 与RSA算法相关的数学概念

#### 互质
&emsp;&emsp;如果两个正整数除了1之外，没有别的公因数，就称这两个数互质。关于互质，有以下几个推论：

- 任意两个质数构成互质关系；
- 一个数是质数，另一个数只要不是它的倍数，则这两个数互质；
- 如果两个数中，较大的那个数是质数，则这两个数互质；
- $1$和任意自然数互质；
- $p$是大于$1$的整数，则$p$和$p-1$互质；
- $p$是大于$1$的奇数，则$p$和$p-2$互质。

#### 欧拉函数

{{< admonition type=tip title="算术基本定理" open=true >}}
任意一个大于$1$的自然数，要么本身是质数，要么可以写成两个或以上质数的积，而且这些质因子按大小排列之后，写法仅有一种方式。
{{< /admonition >}}

在数论中，对正整数$n$，欧拉函数$\varphi(n)$是小于等于$n$的正整数中与$n$互质的数的数目。

\begin{equation}
\varphi(n)= p_1^{k_1-1}p_2^{k_2-1}\cdots p_r{k_r-1}(p_1-1)(p_2-1)\cdots(p_r-1)
\end{equation}

$p_1，p_2 \dots p_r$是$n$的质因子，该函数也可等价的写成
\begin{equation}
\varphi(n)=n(1-\frac{1}{p_1})(1-\frac{1}{p_2})\cdots(1-\frac{1}{p_r})
\end{equation}

#### 欧拉定理

&emsp;&emsp;欧拉定理是RSA算法的核心，理解了这个定理，就理解了RSA。欧拉定理指
如果两个正整数$a$和$n$互质，$\varphi(n)$是$n$的欧拉函数，则
\begin{equation}
a^{\varphi(n)} \equiv 1 \ (mod \ n)
\end{equation}
&emsp;&emsp;欧拉定理有一个特殊情况，假设正整数$a$与质数$p$互质，因为质数$p$的欧拉函数$\varphi(n)$等于$p-1$，则欧拉定理可以写成
\begin{equation}
a^{p-1} \equiv 1 \ (mod \ n)
\end{equation}
这就是著名的费马小定理，它是欧拉函数的特例。

#### 模反元素

&emsp;&emsp;如果两个正整数$a$和$n$互质，那么一定可以找到整数$b$，使得$ab-1$被$n$整除，这是$b$就叫做$a$的模反元素。
欧拉定理可以证明模反元素必然存在
\begin{equation}
a^{\varphi(n)}=a \times a^{\varphi(n)-1} \equiv 1 \ (mod \ n)
\end{equation}
可以看到，$a^{\varphi(n)-1}$就是$a$的模反元素。

### RSA密钥生成步骤

1. 随机选择两个不相等的质数$p$和$q$。
假设$p=61，q=53$
2. 计算$p$和$q$的乘积$n$，$n = pq = 61 \times 53 = 3233$
3. 计算$n$的欧拉函数$\varphi(n)$，根据公式$\varphi(n)=(p-1)(q-1)=3120$
{{< admonition type=note title="欧拉函数性质" open=true >}}
- 如果$p$为质数，则$\varphi(p) = p-1$;
- 如果$n$可以分解成两个互质的整数$p$和$q$之积，则$\varphi(n)=\varphi(p)\varphi(q)$
{{< /admonition >}}
4. 在$1$到$\varphi(n)$之间随机选择一个与$\varphi(n)$互质的整数$e$
5. 计算$e$对于$\varphi(n)$的模反元素$d$。
\begin{equation}
ed \equiv 1 \ (mod \ \varphi(n))
\end{equation}
上式等价于
\begin{equation}
ed - 1 = k\varphi(n)
\end{equation}
于是，找到模反元素$d$，实质上就是对下面这个二元一次方程求解
\begin{equation}
ex+\varphi(n)y = 1
\end{equation}
此方程可以用<a href="https://zh.wikipedia.org/wiki/%E6%89%A9%E5%B1%95%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E7%AE%97%E6%B3%95">扩展欧几里得算法</a>扩展求解，通过计算，得到$d=2753$。
6. 将$n$和$e$封装成公钥，$n$和$d$封装成私钥。

### RSA算法的可靠性
公钥$(n，e)$是公开的，只要得知了$d$，就等于私钥泄露，即已知$n$和$e$，是否可能推导出$d$?
{{< admonition type=question title="" open=true >}}
- $ed \equiv 1 \ (mod \ \varphi(n))$。只有知道$e$和$ \varphi(n) $，才能算出$d$。
- $n=pq$，只有将$n$因数分解，才能算出$p$和$q$。
- 如果$n$可以分解成两个互质的整数$p$和$q$之积，则$\varphi(n)=\varphi(p)\varphi(q)$
{{< /admonition >}}
但是，分解质因数是很困难的，目前除了暴力破解没有别的办法，因此RSA算法的可靠性是基于这个困难的数学问题。

### RSA算法的加密与解密
假设要发送的信息是$m$，加密后的信息为$c$，则加密过程为
\begin{equation}
c=m^e \ mod \ n
\end{equation}
{{< admonition type=warning title="" open=true >}}
由数论知识可知，$m \ mod \ n$的值域是$[0，n)$，因此$m$的值域不能超过$pm$的值域，否则将产生碰撞，即$m$必须小于$n$。
{{< /admonition >}}
解密过程为
\begin{equation}
m = c^d \ mod \ n
\end{equation}

### 私钥解密的证明
&emsp;&emsp;证明私钥能解密，即是证明下式：
\begin{equation}
c^d \equiv m \ (mod \ n)
\end{equation}
由加密公式可得：
\begin{equation}
c=m^e-kn
\end{equation}
将其代入解密公式(10)，
\begin{equation}
(m^e-kn)^d \equiv m \ (mod \ n)
\end{equation}
等价于
\begin{equation}
m^{ed} \equiv m \ (mod \ n)
\end{equation}
{{< admonition type=tip title="" open=true >}}
对$(m^e-kn)^d$二项式展开，除第一项外其他项都包含$kn$，都可以被$n$整除
{{< /admonition >}}
由于
\begin{equation}
ed \equiv 1 \ (mod \ \varphi(n))
\end{equation}
所以
\begin{equation}
ed = h\varphi(n)+1
\end{equation}
将上式代入(14)：
\begin{equation}
m^{h\varphi(n)+1} \equiv m \ (mod \ n)
\end{equation}
接下来，分两种情况讨论，
- $m$和$n$互质
根据欧拉定理，
\begin{equation}
m^{\varphi(n)} \equiv 1 \ (mod \ n)
\end{equation}
得到
\begin{equation}
(m^{\varphi(n)})^h \times m \equiv m \ (mod \ n)
\end{equation}
得证。
- $m$和$n$不是互质关系
由于$n=pq$，则$m$必然等于$kp$或$kq$。
  假设$m=kp$，此时$k$和$q$必然互质，根据欧拉定理知：
\begin{equation}
(kp)^{q-1} \equiv 1 \ (mod \ q)
\end{equation}
进一步得到
\begin{equation}
[(kp)^{q-1}]^{h(p-1)} \times kp \equiv kp \ (mod \ q)
\end{equation}
将式(16)带入上式
\begin{equation}
(kp)^{ed} \equiv kp \ (mod \ q)
\end{equation}
即
\begin{equation}
(kp)^{ed} \equiv tq + kp
\end{equation}
这时$t$必然能被$p$整除，即有$t=t^{\prime}p$
{{< admonition type=tip title="" open=true >}}
这是因为上式两边同时除以$p$，可知等式左边必然能够整除，因此右边$t\frac{q}{p}+p$也必然是一个整数，又因为$p$和$q$互质，则$\frac{t}{p}$必然是一个整数，才能保证等式成立。
{{< /admonition >}}
因此上式可写为
\begin{equation}
(kp)^{ed} = t^{\prime}pq+kp
\end{equation}
将$m=kp$，$n=pq$代入得
\begin{equation}
m^{ed}=t^{prime}n+m
\end{equation}
即
\begin{equation}
m^{ed} \equiv m \ (mod \ n)
\end{equation}
得证。

### 参考文献
- [1] [RSA算法原理（一）](https://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html])
- [2] [RSA算法原理（二）](https://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)
- [3] [RSA算法的数学原理与证明](https://cnodejs.org/topic/5bb9c31e15e4fd1923f48d0b)
