用最小二乘法估计CIR模型参数
=================================

Cox–Ingersoll–Ross 模型的离散形式为：

.. math:: 
    X_{i+1} - X_i = \lambda(\mu - X_i)\Delta t + \sigma \sqrt{X_i \Delta t}\epsilon_i
    :label: eq:ols_1

其中 :math:`\epsilon_i \in N(0, 1)` 是标准正态分布随机数。 

最小二乘法估计模型参数即求解下列最小值问题：

.. math:: 
    \min_{\lambda, \mu} \sum_{i=1}^{N-1} (\frac{X_{i+1} - X_i - \lambda(\mu - X_i)\Delta t}{\sqrt{X_i}})^2
    :label: eq:ols_2

而 :math:`\sigma` 可以通过残差的标准差来估计.

目标函数

.. math:: 
    \begin{aligned}
    f &= \sum_{i=1}^{N-1} (\frac{X_{i+1} - X_i - \lambda(\mu - X_i)\Delta t}{\sqrt{X_i}})^2 \\
    &= \sum_{i=1}^{N-1} (\frac{X_{i+1}}{\sqrt{X_i}} - \lambda \mu \Delta t \frac{1}{\sqrt{X_i}} - (1- \lambda \Delta t)\sqrt{X_i})^2 \\
    &=\sum_{i=1}^{N-1} (\frac{X_{i+1}}{\sqrt{X_i}} - a \frac{1}{\sqrt{X_i}} - b\sqrt{X_i})^2
    \end{aligned}
    :label: eq:ols_3

其中 :math:`a = \lambda \mu \Delta t`, :math:`b = 1- \lambda \Delta t`.

对目标函数 :math:`f` 分别对 :math:`a` 和 :math:`b` 求偏导，并令其为零：

.. math:: 
    \frac{\partial f}{\partial a} = -2\sum_{i=1}^{N-1} (\frac{X_{i+1}}{\sqrt{X_i}} - a \frac{1}{\sqrt{X_i}} - b\sqrt{X_i})\frac{1}{\sqrt{X_i}} = 0
    :label: eq:ols_4

.. math::
    \frac{\partial f}{\partial b} = -2\sum_{i=1}^{N-1} (\frac{X_{i+1}}{\sqrt{X_i}} - a \frac{1}{\sqrt{X_i}} - b\sqrt{X_i})\sqrt{X_i} = 0
    :label: eq:ols_5

由式 :eq:`eq:ols_5` 可得：

.. math:: 
    a = \frac{\sum_{i=1}^{N-1}X_{i+1}-b\sum_{i=1}^{N-1}X_i}{N-1}
    :label: eq:ols_6

再把式 :eq:`eq:ols_6` 代入式 :eq:`eq:ols_4`，可得：

.. math:: 
    \begin{aligned}
    \sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i} - a\sum_{i=1}^{N-1}\frac{1}{X_{i}} - b(N-1) &= 0\\
    \sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i} - (\frac{\sum_{i=1}^{N-1}X_{i+1}-b\sum_{i=1}^{N-1}X_i}{N-1})\sum_{i=1}^{N-1}\frac{1}{X_{i}} - b(N-1) &= 0 \\
    (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i} - \sum_{i=1}^{N-1}X_{i+1}\sum_{i=1}^{N-1}\frac{1}{X_{i}} + b\sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i}} - b(N-1)^2 &= 0\\
    \end{aligned}
    :label: eq:ols_7

于是可以解得

.. math:: 
    b = \frac{\sum_{i=1}^{N-1}X_{i+1}\sum_{i=1}^{N-1}\frac{1}{X_{i}} - (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}}{\sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i}} - (N-1)^2}
    :label: eq:ols_8

再将 :eq:`eq:ols_8` 代入式 :eq:`eq:ols_6`，可得

.. math:: 
    \begin{aligned}
    a &= \frac{\sum_{i=1}^{N-1}X_{i+1}-\frac{\sum_{i=1}^{N-1}X_{i+1}\sum_{i=1}^{N-1}\frac{1}{X_{i}} - (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}}{\sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i+1}} - (N-1)^2}\sum_{i=1}^{N-1}X_i}{N-1}\\
       &= \frac{1}{N-1}(\sum_{i=1}^{N-1}{X_{i+1}} + \frac{\sum_{i=1}^{N-1} X_{i+1}\sum_{i=1}^{N-1}{\frac{1}{X_i}}\sum_{i=1}^{N-1}X_i - (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}\sum_{i=1}^{N-1} X_i}{(N-1)^2- \sum_{i=1}^{N-1} X_i\sum_{i=1}^{N-1}{\frac{1}{X_i}}})\\
       & = \frac{(N-1) \sum_{i=1}^{N-1}{X_{i+1}} - \sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}\sum_{i=1}^{N-1} X_i}{(N-1)^2- \sum_{i=1}^{N-1} X_i\sum_{i=1}^{N-1}{\frac{1}{X_i}}}
    \end{aligned}
    :label: eq:ols_9


于是 

.. math:: 
    \begin{aligned}
    \lambda &= \frac{1-b}{\Delta t} \\
     &= \frac{(N-1)^2 -\sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i}} + \sum_{i=1}^{N-1}X_{i+1}\sum_{i=1}^{N-1}\frac{1}{X_{i}} - (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i} }{((N-1)^2- \sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i}})\Delta t} \\
    \end{aligned}
    :label: eq:ols_10


.. math:: 
    \begin{aligned}
    \mu &= \frac{a}{\lambda \Delta t} \\
     &= \frac{(N-1) \sum_{i=1}^{N-1}{X_{i+1}} - \sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}\sum_{i=1}^{N-1} X_i}{(N-1)^2 -\sum_{i=1}^{N-1}X_i\sum_{i=1}^{N-1}\frac{1}{X_{i}} + \sum_{i=1}^{N-1}X_{i+1}\sum_{i=1}^{N-1}\frac{1}{X_{i}} - (N-1)\sum_{i=1}^{N-1} \frac{X_{i+1}}{X_i}}
     \end{aligned}
    :label: eq:ols_11


