========================= 
Cox–Ingersoll–Ross 模型
=========================

CIR模型（Cox-Ingersoll-Ross模型）是Vasicek模型的一个扩展，适用于建模利率等 **非负** 随机变量。CIR模型的随机微分方程为：

.. math:: 
    dX_t = \lambda(\mu - X_t){dt} + \sigma \sqrt{X_t} dW_t
    :label: eq:cir_1

其中，:math:`\lambda` 是均值回归速度，:math:`\mu` 是长期均值，:math:`\sigma` 是波动率，:math:`dW_t` 是标准布朗运动。

该随机微分方程的闭式解是：

.. math:: 
    X_t = \frac{Y}{2c}
    :label: eq:cir_2 

其中，:math:`c=\frac{2\lambda}{\sigma^2(1-e^{-\lambda t})}`，:math:`Y \in \chi^2(\frac{4\lambda \mu}{\sigma^2}, 2ce^{-\lambda t}X_0 )` 是自由度为 :math:`\frac{4\lambda \mu}{\sigma^2}`, 中心为
:math:`2ce^{-\lambda t}X_0` 的卡方分布的随机数。  

.. note:: 
    通过上述公式，我们可以采用蒙特卡洛模拟方法来生成CIR过程的路径。

其均值和方差分别为：

.. math:: 
    \text{E}[X_t|X_0] = X_0 e^{-\lambda t} + \mu(1-e^{-\lambda t})
    :label: eq:cir_3

.. math::
    \text{Var}[X_t|X_0] = X_0 \frac{\sigma^2 e^{-\lambda t}}{\lambda}(1-e^{-\lambda t}) + \frac{\mu \sigma^2}{2\lambda}(1-e^{-\lambda t})^2
    :label: eq:cir_4

其概率密度函数为： 

.. math:: 
    p(X_t|X_0) = c e^{-u-v}(\frac{v}{u})^{q/2} I_q(2\sqrt{uv})
    :label: eq:cir_5

其中 :math:`u=cX_0 e^{-\lambda t}`, :math:`v=cX_t`, :math:`q=\frac{2\lambda \mu}{\sigma^2}-1`，:math:`I_q` 是第一类修正贝塞尔函数。   

模型参数拟合
---------------------------------

对于一组以 :math:`\Delta t` 等时间间隔采样的时间序列 :math:`X_i`，我们可以通过极大似然估计（Maximum Likelihood Estimation）的方法来拟合CIR模型的参数 :math:`\lambda`, :math:`\mu`, :math:`\sigma`。
假设 :math:`X_i` 是CIR过程的一个样本路径，那么其似然函数为：

.. math:: 
    L(\lambda, \mu, \sigma) = \prod_{i=0}^{N-1} p(X_{i+1}|X_i)
    :label: eq:cir_6

其中 :math:`f(X_{i+1}|X_i)` 是式 :eq:`eq:cir_5` 的概率密度函数。我们可以通过数值优化方法来最大化该似然函数，从而得到模型参数的估计值。



.. note:: 
    在采用数值方法优化似然函数时，一般采用对数似然函数,将连乘转化为求和：

    .. math::
        \ln L(\lambda, \mu, \sigma) = \sum_{i=0}^{N-1} \ln p(X_{i+1}|X_i)
        :label: eq:cir_7


初始值问题
---------------------------------

在采用极大似然估计方法拟合CIR模型时，选择合适的初始值对于数值优化的收敛性速度和结果的准确性至关重要。我们可以采用最小二乘法对CIR模型进行初步拟合，以得到合理的初始值。


将式 :eq:`eq:cir_1` 离散化为：

.. math:: 
    X_{i+1} - X_i = \lambda(\mu - X_i)\Delta t + \sigma \sqrt{X_i \Delta t}\epsilon_i
    :label: eq:cir_8

其中 :math:`\epsilon_i \in N(0, 1)` 是标准正态分布随机数。  

采用最小二乘法估计参数 :math:`\lambda` , :math:`\mu` ,以及 :math:`\sigma`，即求解

.. math:: 
    \min_{\lambda, \mu} \sum_{i=1}^{N-1} (\frac{X_{i+1} - X_i - \lambda(\mu - X_i)\Delta t}{\sqrt{X_i}})^2
    :label: eq:cir_9

而其中 :math:`\sigma` 可以通过残差的标准差来估计.

通过最小二乘法估计 :math:`\lambda` , :math:`\mu` ,以及 :math:`\sigma` 具体求解过程参考附录 :doc:`cir_model_ols_calibration` 。


数据分析示例
---------------------------------
指数的涨跌幅的方差 :math:`v` 可能是一个CIR过程。

