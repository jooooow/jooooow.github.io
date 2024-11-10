---
title: "Ewald Summation和PME法计算静电力"
layout: post
---

<script async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" id="MathJax-script"></script>
<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'],['\$', '\$']]
  }
};
</script>

Ewald Summation法常被用来计算存在多个点电荷的电场，最终的结果可以直接看[公式16,17](#summary)。
详细推导过程看[这里](../doc/ewald_derivation.pdf)。

首先定义三维空间中一点的坐标$\vec{z}$为：

$$
\vec{z}=(x,y,z)=(r,\theta,\phi)
$$

其中$ (x,y,z)$为笛卡尔坐标系下的坐标，$(r,\theta,\phi)$为球面坐标系下的坐标。对于存在$N$个点电荷的各边长为$L$的元胞，将其复制无限个副本环绕在周围可以满足周期边界条件，此时空间中任意一点$\vec{z}$的电势可以表示为：

$$
\Phi(\vec{z})=\frac{1}{4 \pi \epsilon_0} \sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} \frac{q_j}{|\vec{z}-\vec{z}_j+nL|} \tag{1}
$$

对于任意的点电荷$i$，在计算其电势时不包含自身，因此空间中任意一点$\vec{z}$受到的除电荷$i$之外的所有电荷施加的电势可以表示为：

$$
\Phi_{[i]}(\vec{z})=\frac{1}{4 \pi \epsilon_0} \sum_{n=-\infty}^{\infty} \sum_{j=1(\ne i)}^{N} \frac{q_j}{|\vec{z}-\vec{z}_j+nL|} \tag{2}
$$

这个$\Phi_{[i]}(\vec{z})$才是我们真正想求出的电势。然而，直接计算的话是一个$O(N^2)$的问题。Ewald的做法是使用假想的电荷密度分解电场，然后计算电荷与假想电场的相互作用。电荷$j$产生的空间中任意一点$\vec{z}$对应的电荷密度可以表示为

$$
\rho_j(\vec{z})=q_j \delta (\vec{z}-\vec{z}_j) \tag{3}
$$

Ewald的核心在于通过假想的高斯分布将3式分解成两部分，即：

$$\begin{align}
\rho_j(\vec{z})&=q_j \delta (\vec{z}-\vec{z}_j)\\
&=\underbrace{q_j (\delta (\vec{z}-\vec{z}_j)-G(\vec{z}-\vec{z}_j))}_{\rho_j^s(\vec{z})}+\underbrace{q_j G(\vec{z}-\vec{z}_j)}_{\rho_j^l(\vec{z})}
 \tag{4}
\end{align}$$

其中$G(\vec{z})$定义为

$$\begin{align}
G(\vec{z})&=\frac{\alpha ^ 3}{\pi ^ {\frac{3}{2}}} e^{-\alpha^2 r^2}  \\
\alpha &= \frac{1}{\sqrt{2} \sigma} \tag{5}
\end{align}$$

$\sigma$为正态分布的标准差，控制着收敛速度。  
$\rho_j^s$对应的是短程作用（实空间快速衰减），$\rho_j^l$对应的是长程作用（倒空间快速衰减）。
<br>
接着来计算这两个假想的电荷密度所产生的电势，首先处理长程作用产生的电势。电荷密度和电势通过泊松方程相关联：

$$
\Delta \Phi(\vec{z}) = -\frac{\rho(\vec{z})}{\epsilon_0} \tag{6}
$$

将5式带入泊松方程可得：

$$\begin{align}
\Delta \Phi(\vec{z}) &= -\frac{G(\vec{z})}{\epsilon_0} \\
\frac{1}{r} \frac{\partial^2}{\partial r^2}(r\Phi(\vec{z}) ) &= -\frac{\alpha ^ 3}{\pi ^ {\frac{3}{2}} \epsilon_0} e^{-\alpha^2 r^2} \\
\frac{\partial^2}{\partial r^2}(r\Phi(\vec{z}) ) &= -\frac{\alpha ^ 3}{\pi ^ {\frac{3}{2}} \epsilon_0} re^{-\alpha^2 r^2} \\
\frac{\partial}{\partial r}(r\Phi(\vec{z}) ) &= \frac{\alpha}{2 \pi ^ {\frac{3}{2}} \epsilon_0} e^{-\alpha^2 r^2} + C_1 \\
r\Phi(\vec{z}) &= \frac{\alpha}{2 \pi ^ {\frac{3}{2}} \epsilon_0} \int_{0}^{r} e^{-\alpha^2 r^2} dr + C_1 r + C_2 \\
\Phi(\vec{z}) &= \frac{\alpha}{2 \pi ^ {\frac{3}{2}} \epsilon_0 r} \int_{0}^{r} e^{-\alpha^2 r^2} dr + C_1 + \frac{C_2}{r} \\
 &= \frac{1}{2  \pi ^ {\frac{3}{2}} \epsilon_0 r} \int_{0}^{r} e^{-\alpha^2 r^2} d(\alpha r) + C_1 + \frac{C_2}{r} \\
 &= \frac{erf(\alpha r)}{4  \pi \epsilon_0 r}  + C_1 + \frac{C_2}{r} \\
\end{align}$$

当$r \to \infty$时：

$$\begin{align}
\lim_{r \to \infty} erf(r) &= 1\\
\lim_{r \to \infty} \Phi(r) &= q_j \frac{1}{4  \pi \epsilon_0 r} (库仑定律)
\end{align}$$

可得$C_1=C_2=0$。因此$\rho_j^l(\vec{z})$产生的长程电势$\Phi_j^l(\vec{z})$为：

$$
\Phi_j^l(\vec{z})= q_j \frac{erf(\alpha |\vec{z}-\vec{z}_j|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j|} \tag{7}
$$

相应的，$\rho_j^s(\vec{z})$产生的短程电势$\Phi_j^s(\vec{z})$为：

$$
\Phi_j^s(\vec{z})= q_j \frac{erfc(\alpha |\vec{z}-\vec{z}_j|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j|}  \tag{8}
$$

$erf$和$erfc$分别为误差函数和互补误差函数。
基于公式7和8，公式2中的$\Phi_{[i]}$可以改写为两个部分的电势的和：

$$\begin{align}
\Phi_{[i]} &= \Phi_{[i]}^{s}+\Phi_{[i]}^{l}\\
&= \sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} \Phi_j^s(\vec{z}) + \sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} \Phi_j^l(\vec{z})\\
&= \sum_{n=-\infty}^{\infty} \sum_{j=1(\ne i)}^{N} q_j \frac{erfc(\alpha |\vec{z}-\vec{z}_j+nL|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j+nL|} + \sum_{n=-\infty}^{\infty} \sum_{j=1(\ne i)}^{N} q_j \frac{erf(\alpha |\vec{z}-\vec{z}_j+nL|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j+nL|} \tag{9}
\end{align}$$

由于$\Phi_{[i]}^{s}$在实空间中快速衰减因此在cutoff内直接计算，而$\Phi_{[i]}^{l}$仅在倒空间中快速衰减因此使用傅里叶变换计算。然而由于傅里叶变换时对整体进行变换无法单独忽略某点电荷，因此计算出的电势中会包含电荷和自己的相互作用，这个自相互作用产生的电势$\Phi_{i}^{self}$需要减掉。因此，实际计算时公式9变形为：

$$\begin{align}
\Phi_{[i]} &= \Phi_{[i]}^{s}+\Phi_{}^{l} - \Phi_{i}^{self}\\
&= \sum_{n=-\infty}^{\infty} \sum_{j=1(\ne i)}^{N} q_j \frac{erfc(\alpha |\vec{z}-\vec{z}_j+nL|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j+nL|} + \sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} q_j \frac{erf(\alpha |\vec{z}-\vec{z}_j+nL|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j+nL|}  - \Phi_{i}^{self} \tag{10}
\end{align}$$

注意公式10中的长程电势中已经不包含$j \ne i$，因此可以被转化到傅里叶空间。

对于自相互作用产生的电势，可以通过将$\Phi_{}^{l}(r)$的$r$趋于$0$得到：

$$
\begin{align}
\Phi_{i}^{self} &= \lim_{r \to 0} q_j \frac{erf(\alpha r)}{4 \pi \epsilon_0 r}\\
&= \frac{q_j}{4 \pi \epsilon_0} \lim_{r \to 0} \frac{erf(\alpha r)}{r}\\
&= \frac{q_j}{4 \pi \epsilon_0} \lim_{x \to 0} \frac{2 \alpha}{\sqrt{\pi}} e^{-x^2} (换元+洛必达)\\
&= \frac{\alpha q_j}{2 \pi ^ {\frac{3}{2}} \epsilon_0}  \tag{11}
\end{align}
$$

现在只剩下长程电势$\Phi_{}^{l}$了，需要在傅里叶空间中计算。这里先明确下使用的傅里叶级数的正/反变换分别定义为：

$$
\begin{align}
F(k)&=\int f(x) e^{-ikx} dx\\
f(x)&=\frac{1}{L} \sum F(k) e^{ikx} dk 
\end{align}
$$

公式4中定义$j$产生的长程电荷密度为：

$$
\rho_j^l(\vec{z})=q_j G(\vec{z}-\vec{z}_j)
$$

基于此，所有电荷产生的长程电荷密度定义为：

$$
\rho^l(\vec{z})=\sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} q_j G(\vec{z}-\vec{z}_j + nL) \tag{12}
$$ 

对其进行变换到频域：

$$\begin{align}
\tilde{\rho}^l(\vec{w}) &= \int \sum_{n=-\infty}^{\infty} \sum_{j=1}^{N} q_j G(\vec{z}-\vec{z}_j + nL) e^{-i\vec{w}\cdot \vec{z}} d\vec{z}\\
&= \sum_{j=1}^{N} q_j  \int_{R^3}   G(\vec{z}-\vec{z}_j) e^{-i\vec{w} \cdot \vec{z}} d\vec{z}\\
&= \frac{\alpha^3}{\pi^{\frac{3}{2}}} \sum_{j=1}^{N} q_j  \int_{R^3}   e^{-\alpha^2 |\vec{z}-\vec{z}_j|^2} e^{-i\vec{w} \cdot \vec{z}_j} d\vec{z}\\
&= e^{-\frac{|\vec{w}|^2}{4 \alpha ^ 2}} \sum_{j=1}^{N} q_j e^{-i\vec{w} \cdot \vec{z}_j} \tag{13}
\end{align}$$

公式13表明只需要对全体点电荷做一个离散傅里叶变换再乘以一个系数就可以得到长程电荷分布的频域函数。

接着，对公式6的泊松方程两端进行傅里叶变换，可以得到关联频域的电势和电荷密度的Green函数：

$$
|\vec{w}|^2  \tilde{\Phi}^l(\vec{w}) = \frac{\tilde{\rho}^l(\vec{w})}{\epsilon_0} \tag{14}
$$

将通过公式13计算出的频域电荷密度带入公式14就可以得到电势的频域函数。最后再将其反变换回实空间即可：

$$
\Phi^l(\vec{z}) = \frac{1}{L^3} \sum_{\vec{w} \ne 0} \tilde{\Phi}^l(\vec{w}) e^{i \vec{w} \cdot \vec{z}} \tag{15}
$$


最后总结一下：
<a name="summary"></a>

$$
\begin{align}
&\Phi_{[i]} = \Phi_{[i]}^{s}+\Phi_{}^{l} - \Phi_{i}^{self} \tag{16}\\ 
&\Phi_{[i]}^{s} = \sum_{n=-\infty}^{\infty} \sum_{j=1(\ne i)}^{N} q_j \frac{erfc(\alpha |\vec{z}-\vec{z}_j+nL|)}{4  \pi \epsilon_0 |\vec{z}-\vec{z}_j+nL|} \\
&\Phi_{}^{l} = IDFT(Green(DFT(\rho^l))) \\
&\Phi_{i}^{self} = \frac{\alpha q_j}{2 \pi ^ {\frac{3}{2}} \epsilon_0}
\end{align} 
$$

得到电势之后力的计算就很简单了，分别对每个部分求空间一阶导再乘上$-q_i$即可：

$$
F_i = -q_i \nabla \Phi_{[i]} \tag{17}
$$


Particle Mesh Ewald(PME)和Ewald Summation的不同就是PME将电荷插值到网格上，之后对网格[FFT](https://jooooow.github.io/archive/fft.html)来计算电势/力。

#### 参考资料

1. [http://micro.stanford.edu/mediawiki/images/4/46/Ewald_notes.pdf](http://micro.stanford.edu/mediawiki/images/4/46/Ewald_notes.pdf)
2. [http://staff.ustc.edu.cn/~zqj/posts/Ewald-Summation/](http://staff.ustc.edu.cn/~zqj/posts/Ewald-Summation/)
3. [https://zhuanlan.zhihu.com/p/406664860](https://zhuanlan.zhihu.com/p/406664860)

<script src="https://utteranc.es/client.js"
        repo="jooooow/jooooow.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

