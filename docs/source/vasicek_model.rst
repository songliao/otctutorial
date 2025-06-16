==============
Vasicek 模型
==============

Vasicek模型用以下随机微分方程来表述随机变量 :math:`X_t` 的演化过程：

.. math:: 
    dX_t = \lambda(\mu - X_t){dt} + \sigma dW_t
    :label: eq:tsa_1

其中，:math:`\lambda` 是均值回归速度，:math:`\mu` 是长期均值，:math:`\sigma` 是波动率，:math:`dW_t` 是标准布朗运动。

上述偏微分方程的解是：

.. math:: 
    X_t = X_0 e^{-\lambda t} + \mu(1 - e^{-\lambda t}) + \sqrt{\frac{\sigma^2}{2\lambda}(1-e^{-2\lambda t})}\epsilon
    :label: eq:tsa_2

其中，:math:`\epsilon \in N(0, 1)` 是标准正态分布随机数。

.. note:: 
    我们可以用上式来模拟Vasicek过程的路径。

模型参数拟合
---------------------------------

我们可以根据 :eq:`eq:tsa_2` 的形式，对给定的具有均值回归特性的时间序列，采用最小二乘法（Ordinary Least Square Method）拟合得到相关模型参数 :math:`\lambda`, :math:`\mu`, :math:`\sigma`。

对于以 :math:`\Delta t` 等时间间隔采样的时间序列 :math:`X_i`, 先通过线性拟合得到，

.. math:: 
    X_{i+1} = a X_i +b + \epsilon_i
    :label: eq:tsa_3

那么，对比 :eq:`eq:tsa_2` 和 :eq:`eq:tsa_3`，我们有

.. math:: 
    a = e^{-\lambda \Delta t}, \quad b = \mu(1 - e^{-\lambda \Delta t}), \quad \sigma(\epsilon) = \sqrt{\frac{\sigma^2}{2\lambda}(1-e^{-2\lambda \Delta t})}
    :label: eq:tsa_4

或者

.. math:: 
    \lambda = -\frac{\ln a}{\Delta t}, \quad \mu = \frac{b}{1 - a}, \quad \sigma = \sigma(\epsilon)\sqrt{\frac{-2\ln{a}}{{\Delta t(1-a^2)}}}
    :label: eq:tsa_5

其中，:math:`\sigma(\epsilon)` 是式 :eq:`eq:tsa_3` 中残差的标准差。

本节主要参考 [1]_ .




参考文献
-----------
.. [1] Wikipeida: Vasicek_model https://en.wikipedia.org/wiki/Vasicek_model