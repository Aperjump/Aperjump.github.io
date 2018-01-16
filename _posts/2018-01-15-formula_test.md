---
layout: post
title: "formula_testing"
description: "Formular"
categories: [demo]
tags: [demo,random]
redirect_from:
  - /2018/01/15/
---

#### 1. problem 1
设$x\sim N(\mu, \Sigma)$,其中$x = \begin{matrix} (x_1 , x_2 ,\cdots,x_p)\end{matrix}'$, $\mu = \begin{matrix}(\mu_1, \mu_2,\cdots, \mu_p)\end{matrix}'$, $\Sigma = \begin{matrix}diag(\sigma_1^2, \sigma_2^2, \cdots, \sigma_p^2)\end{matrix}$,试证$x_1, x_2, \cdots, x_p$相互独立

由性质（6）可知：对于多元正态变量而言，其子向量之间互不相关与相互独立是等价的，所以证明$x_1, x_2, \cdots, x_p$之间相互独立转化为证明$x_1, x_2, \cdots, x_p$之间不相关。
$$Cov(x_i,x_j) = \Sigma_{ij}$$
由于$\Sigma = \begin{matrix}diag(\sigma_1^2, \sigma_2^2, \cdots, \sigma_p^2)\end{matrix}$，对于$i\neq j$，$\Sigma_{ij} = 0$所以$x_1, x_2, \cdots, x_p$之间不相关,故$x_1, x_2, \cdots, x_p$之间相互独立。

#### 2. problem 2
设$x\sim N_{2p}(\mu, \Sigma)$，$x,\mu,\Sigma$的刨分如下：
$$ 
x = \left(
    \begin{array}{cc}
        x_1\\
        x_2
    \end{array}
\right)
\mu = \left(
    \begin{array}{cc}
        \mu_1\\
        \mu_2
    \end{array}
\right)
\Sigma = \left(
    \begin{array}{cc}
        \Sigma_1 & \Sigma_2\\
        \Sigma_2 & \Sigma_1
    \end{array}
\right)
$$
试证$x_1+x_2$和$x_1-x_2$相互独立
令$C_1 = (I_p,0)$, $x_1\sim N_p(C_1\mu, C_1\Sigma C_1')$
同理令$C_2 = (0,I_p)$, $x_2\sim N_p(C_2\mu, C_2\Sigma C_2')$
$$
\Sigma_{x1} = (I_p,0)\left(
    \begin{array}{cc}
        \Sigma_1 & \Sigma_2\\
        \Sigma_2 & \Sigma_1
    \end{array}
\right)(
    \begin{array}{cc}
        I_p\\
        0
    \end{array}) = \Sigma_1
$$
$$
\Sigma_{x2} = (0,I_p)\left(
    \begin{array}{cc}
        \Sigma_1 & \Sigma_2\\
        \Sigma_2 & \Sigma_1
    \end{array}
\right)(
    \begin{array}{cc}
        0\\
        I_p
    \end{array}) = \Sigma_1
$$
所以$x_1+x_2\sim N_p(\mu_1+\mu_2,2\Sigma_1)$ ,$x_1-x_2\sim N_p(\mu_1-\mu_2, 2\Sigma_1)$
设$$y = \left(
    \begin{array}{cc}
        x_1+x_2\\
        x_1-x_2
    \end{array}
\right)
\mu'= \left(
    \begin{array}{cc}
        \mu_1+\mu_2\\
        \mu_2-\mu_2
    \end{array}
\right)
\Sigma' = \left(
    \begin{array}{cc}
        2\Sigma_1 & 0\\
        0 & 2\Sigma_1
    \end{array}
\right)
$$有
$$
f(y) = (2\pi)^{-p}\|\Sigma'\|^{-1/2}exp[-\frac{1}{2}(y-\mu')'\Sigma^{-1}(y-\mu')] \\
f(x_1+x_2) = (2\pi)^{-p/2}\|2\Sigma_1\|^{-1/2}exp[-\frac{1}{2}(x_1+x_2-(\mu_1+\mu_2))'2\Sigma_1^{-1}(x_1+x_2-(\mu_1+\mu_2))]\\
f(x_1-x_2) = (2\pi)^{-p/2}\|2\Sigma_1\|^{-1/2}exp[-\frac{1}{2}(x_1-x_2-(\mu_1-\mu_2))'2\Sigma_1^{-1}(x_1-x_2-(\mu_1-\mu_2))]
$$
由于
$$
-\frac{1}{2}(y-\mu')'\Sigma^{-1}(y-\mu') =-\frac{1}{2}(x_1+x_2-(\mu_1+\mu_2))'2\Sigma_1^{-1}(x_1+x_2-(\mu_1+\mu_2))  -\frac{1}{2}(x_1+x_2-(\mu_1+\mu_2))'2\Sigma_1^{-1}(x_1+x_2-(\mu_1+\mu_2)) \\
\|\Sigma'\| = 4\|\Sigma_1\|
$$
故
$$
f(y) = f(x_1+x_2)f(x_1-x_2)
$$
####3. problem 3
导出统计量$T^2 = n(C\bar{x} - \varphi)'(CSC')^{-1}(C\bar{x}-\varphi)$
由于$x = (x_1,x_2,\cdots,x_n)$取自正态总体$N(\mu, \Sigma)$则$Cx-\varphi\sim N(0, \Sigma)$并且根据Wishart分布的性质，
$\sum(Cx-\varphi)(Cx-\varphi)'\sim W_p(n,C\Sigma C')$
样本方差矩阵具有$(n-1)CSC'\sim W_p(n-1,C\Sigma C')$
根据Hotelling分布的定义，有
$$T^2 = n(Cx-\varphi)(CSC')(Cx-\varphi)'\sim T^2(p, n-1)$$
####4. problem 4
检验甲乙两种品牌之间各个指标是否存在显著的差异

    A <- c(11, 33, 20, 18, 22, 18, 27, 28, 26, 23, 
           15, 31, 27, 18, 22, 18, 21, 23, 18, 16,
           15, 17, 19, 9, 10)
    A <- matrix(A, nrow = 5, ncol = 5)
    B <- c(18, 31, 14, 25, 36, 17, 24, 16, 24, 28,
           20, 31, 17, 31, 24, 18, 26, 20, 26, 26, 
           18, 20, 17, 18, 29)
    B <- matrix(B, nrow = 5, ncol = 5)
    C <- c(1, 0, 0, 0, -1, 1, 0, 0,
           0, -1, 1, 0, 0, 0, -1, 1,
           0, 0, 0, -1)
    C <- matrix(C, nrow = 4, ncol = 5)
    mu_A <- c(20.8, 24.4, 22.6, 19.2, 14)
    mu_B <- c(24.8, 21.8, 24.6, 23.2, 20.4)
    a <- A - mean(A)
    a_1 <- (a[1,] %*% t(a[1,]))
    for (i in 2:5){
      a_1 <- a_1 + (a[i,] %*% t(a[i,]))
    }
    a_1 <- a_1 / 4
    
    b <- B - mean(B)
    b_1 <- (b[1,] %*% t(b[1,]))
    for (i in 2:5){
      b_1 <- b_1 + (b[i,] %*% t(b[i,]))
    }
    b_1 <- b_1 / 4
    S_p <- (4 * a_1 + 4 * b_1) / 8
    T_square <- (5*5/(5+5)) * t(mu_A - mu_B) %*% t(C) %*%
      solve(C %*% S_p %*% t(C)) %*% C %*% (mu_A - mu_B)
    T_alpha <- 4*8 / 5 * qf(0.95, 4, 5+5-5)
最终结果为不拒绝

    > T_square
             [,1]
    [1,] 6.213861
    > T_alpha
    [1] 33.22987