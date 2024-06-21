---
title: "关于FMM中的球谐函数的定义"
---

<script async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" id="MathJax-script"></script>
<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'],['\$', '\$']]
  }
};
</script>

[FMM论文](https://www.cambridge.org/core/journals/acta-numerica/article/abs/new-version-of-the-fast-multipole-method-for-the-laplace-equation-in-three-dimensions/8D84CC50463A63C73A5E97A045F16B79)中公式3.15使用的球谐函数定义与[通常](https://en.wikipedia.org/wiki/Spherical_harmonics#Conventions)不太一样，为此记录一下。  
通常的球谐函数定义：

$$
Y_n^m(\theta, \phi) = \sqrt{\frac{(n-m)!}{(n+m)!}}P_n^m(cos \theta) e^{im \phi}, 0 \le m \le n \tag{1}
$$

其中<a href='https://en.wikipedia.org/wiki/Associated_Legendre_polynomials'>伴随勒让德多项式</a>
$P_n^m(x)$定义为<a href='https://en.wikipedia.org/wiki/Legendre_polynomials'>勒让德多项式</a>$P_n(x)$的$n$次导数乘上$x$的$m$次:

$$
P_n^m(x) = (-1)^m (1 -x^2)^{\frac{m}{2}}\frac{d^m}{dx^m}P_n(x), m\ge0  \tag{2}
$$

由于$P_n(x)$是$x$的$n$次函数，所以当$m>n$时$P_n^m(x)=0$。
对于负次的情况可以参考<a href='http://www.jooooow.com/static/pdf/FMM.pdf'>这里</a>，扩充公式(2)：

$$
P_n^{-m}(x) = (-1)^m\frac{(n-m)!}{(n+m)!}P_n^m(x) \tag{3}, 0 \le m \le n
$$

注意到$P_n^m$定义里这个鬼畜的$(-1)^m$，在定义球谐函数时有些时候会将它从$P_n^m$里摘出来放到$Y_n^m$里，导致$P_n^m$递推公式的<a href='https://www.physics.uoguelph.ca/chapter-4-spherical-harmonics'>细微变化</a>，然而依然满足公式(6)的关系。
<br>
基于公式(3)，可以扩充公式(1)到负次的情况：

$$
\begin{align}
Y_n^{-m}(\theta, \phi) &= \sqrt{\frac{(n+m)!}{(n-m)!}}P_n^{-m}(cos \theta) e^{-im \phi} \\
&=\sqrt{\frac{(n+m)!}{(n-m)!}}(-1)^m\frac{(n-m)!}{(n+m)!}P_n^m(cos \theta)  e^{-im \phi}\\
&=(-1)^m\sqrt{\frac{(n-m)!}{(n+m)!}}P_n^m(cos \theta)  e^{-im \phi} , 0 \le m \le n  \tag{4}\\
\end{align}
$$

整理一下公式(1)和(4)：

$$
    Y_n^{m}(\theta, \phi) =
\begin{cases}
    \sqrt{\frac{(n-m)!}{(n+m)!}}P_n^m(cos \theta) e^{im \phi} &, 0 \le m \le n\\
    (-1)^m\sqrt{\frac{(n-|m|)!}{(n+|m|)!}}P_n^{|m|}(cos \theta) e^{im \phi}  &,  -n \le m \lt 0
\end{cases}\
 \tag{5}
$$

注意到此时满足：

$$
Y_n^{-m}(\theta, \phi) =(-1)^m \overline{Y_n^m(\theta, \phi)}, -n \le m \le n  \tag{6}
$$

在分解$\frac{1}{r}$形电势时使用<a href='https://en.wikipedia.org/wiki/Spherical_harmonics#Algebraic_properties'>球谐函数加法定理</a>把勒让德多项式其可以展开为多个球谐函数的积和：

$$
P_n(cos \gamma)=\sum_{m=-n}^{n}Y_n^m(\theta_1, \phi_1)\overline{Y_n^m(\theta_2, \phi_2)} \tag{7}
$$

或者

$$
P_n(cos \gamma)=\sum_{m=-n}^{n} (-1)^m Y_n^m(\theta_1, \phi_1)Y_n^{-m}(\theta_2, \phi_2) \tag{8}
$$

其中$\gamma$为两个向量的夹角。对比可以发现(7)的形式比较简单，只需要求一次$Y_n^m$即可，而(8)就要麻烦一些，因此倾向于用(7)去计算。然而由于(5)的分段定义导致我们需要对$m$的正负进行讨论，将(6)带入(7)可以发现：

$$
\begin{align}
P_n(cos \gamma)&=\sum_{m=-n}^{n}Y_n^m(\theta_1, \phi_1)\overline{Y_n^m(\theta_2, \phi_2)} \\
&=\sum_{m=0}^{n}Y_n^m(\theta_1, \phi_1)\overline{Y_n^m(\theta_2, \phi_2)}+\sum_{m=1}^{n}Y_n^{-m}(\theta_1, \phi_1)\overline{Y_n^{-m}(\theta_2, \phi_2)}\\
&=\sum_{m=0}^{n}Y_n^m(\theta_1, \phi_1)\overline{Y_n^m(\theta_2, \phi_2)}+\sum_{m=1}^{n}(-1)^m\overline{Y_n^{m}(\theta_1, \phi_1)}\overline{(-1)^m\overline{Y_n^{m}(\theta_2, \phi_2)}}\\
&=\sum_{m=0}^{n}Y_n^m(\theta_1, \phi_1)\overline{Y_n^m(\theta_2, \phi_2)}+\sum_{m=1}^{n}\overline{Y_n^{m}(\theta_1, \phi_1)}Y_n^{m}(\theta_2, \phi_2) \tag{9}
\end{align}
$$

公式(9)意味着我们需要对成地累加$Y_n^1, \overline{Y_n^1},Y_n^2, \overline{Y_n^2},\dots$，换句话说，当计算负次的时$Y_n^m$，我们希望正好它等于对应的正次的$Y_n^m$的共轭(即公式(12))，
为此记：

$$
    S_n^{m}(\theta, \phi) =
\begin{cases}
    Y_n^m(\theta, \phi) &, 0 \le m \le n\\
    \overline{Y_n^{|m|}(\theta, \phi)}  &,  -n \le m \lt 0
\end{cases}\
 \tag{10}
$$

于是$S_n^m$定义为：

$$
S_n^m(\theta, \phi) = \sqrt{\frac{(n-|m|)!}{(n+|m|)!}}P_n^{|m|}(cos \theta) e^{im \phi}, -n \le m \le n \tag{11}
$$

此时$S_n^m$满足：

$$
S_n^{-m} = \overline{S_n^m},-n \le m \le n  \tag{12}
$$

公式(11)就是<a href='https://www.cambridge.org/core/journals/acta-numerica/article/abs/new-version-of-the-fast-multipole-method-for-the-laplace-equation-in-three-dimensions/8D84CC50463A63C73A5E97A045F16B79'>FMM论文</a>中使用的定义3.15(论文中没用$S_n^m$这个符号而是直接用的$Y_n^m$符号具有一定误导性)。    
此时公式(7)可以改写为：

$$
P_n(cos \gamma)=\sum_{m=-n}^{n}S_n^m(\theta_1, \phi_1)S_n^{-m}(\theta_2, \phi_2) \tag{13}
$$

这对应着论文中的公式3.16，由于$S_n^m$的定义是连续的，因此计算$P_n$时就不需要讨论了。


#### 参考资料
+ [https://math.libretexts.org/Bookshelves/Differential_Equations/Introduction_to_Partial_Differential_Equations_(Herman)/06%3A_Problems_in_Higher_Dimensions/6.05%3A_Laplaces_Equation_and_Spherical_Symmetry](https://math.libretexts.org/Bookshelves/Differential_Equations/Introduction_to_Partial_Differential_Equations_(Herman)/06%3A_Problems_in_Higher_Dimensions/6.05%3A_Laplaces_Equation_and_Spherical_Symmetry)
+ [https://zhuanlan.zhihu.com/p/153352797](https://zhuanlan.zhihu.com/p/153352797)
+ [https://www.physics.uoguelph.ca/chapter-4-spherical-harmonics](https://www.physics.uoguelph.ca/chapter-4-spherical-harmonics)