=====================
分散度（相关性）交易
=====================
分散度交易的基本形式为

1，买指数波动率卖成分股个股波动率，或者

2，卖指数波动率买成分股个股波动率。

通过维持投资组合在方向敞口上的中性（delta-neutral），交易指数与成分股波动率之间的差异来获得收益。

指数与成分股波动率之间的关系
------------------------------

我们用 :math:`I` 表示指数，:math:`S_i` 表示成分股，:math:`w_i` 是指数中该成分股的 **份额** 占比，

.. math:: 
    I = \sum_i^N w_i S_i
    :label: eq:8

那么个股在指数中的 **市值** 权重，即

.. math:: 
    p_i = \frac{w_iS_i}{I}
    :label: eq:9

那么我们有

.. math:: 
    \begin{aligned}
    \frac{dI}{I} &= \sum_i^N \frac{w_i dS_i}{I}\\
    &= \sum_i^N \frac{w_i S_i}{I} \frac{dS_i}{S_i}\\
    &= \sum_i^N p_i \frac{dS_i}{S_i}
    \end{aligned}
    :label: eq:10

所以可以得到指数波动率与个股波动率之间的关系为

.. math::
    \begin{aligned} 
    \sigma_I^2 &= \sum_{i,j}p_i p_j \sigma_i \sigma_j \rho_{i,j}\\
    &= \sum_{i}p_i^2\sigma_i^2 + \sum_{i\neq j} p_i p_j \sigma_i \sigma_j \rho_{i,j}
    \end{aligned}
    :label: eq:11

定义平均相关系数为

.. math:: 
    \bar{\rho} = \frac{\sigma_I^2 - \sum_{i} p_i^2 \sigma_i^2}{\sum_{i\neq j} p_i p_j \sigma_i \sigma_j}
    :label: eq:12


.. warning:: 
    注意到 :math:`(\sum_i p_i\sigma_i)^2 = \sum_i p_i^2\sigma_i^2 + \sum_{i\neq j}p_ip_j\sigma_i\sigma_j`, 所以平均相关系数可以用下式来计算
    
    .. math::
       \bar{\rho} = \frac{\sigma_I^2 - \sum_{i} p_i^2 \sigma_i^2}{(\sum_i p_i\sigma_i)^2-\sum_i p_i^2\sigma_i^2}


期权盈亏的另一种分解方式
------------------------------

回忆BSM PDE，

.. math:: 
    \frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2S^2\frac{\partial^2 V}{\partial S^2} + (r- q)S\frac{\partial V}{\partial S} - rV = 0
    :label: eq:13

在本节以下的讨论中，简单起见，我们假定 :math:`r = q = 0`, 根据 :eq:`eq:13` 式，

.. math:: 
    \frac{\partial^2 V}{\partial S^2}= -2\frac{\partial V}{\partial t}\frac{1}{\sigma^2S^2} 
    :label: eq:14


而期权持有者在 :math:`\Delta t` 时间的持仓盈亏为

.. math:: 
    \Delta V \approx \frac{\partial V}{\partial S}\Delta S + \frac{\partial V}{\partial t}\Delta t + \frac{1}{2}\frac{\partial^2 V}{\partial S^2}\Delta S^2
    :label: eq:15

将 :eq:`eq:14` 式 代入到 :eq:`eq:15` 式中可得

.. note:: 
    
    .. math:: 
       \Delta V \approx \frac{\partial V}{\partial S}\Delta S + \frac{\partial V}{\partial t}\Delta t(1- \frac{\Delta S^2}{S^2\sigma^2\Delta t})
       :label: eq:16

通过 :eq:`eq:16` 式可以看到持有期权的盈亏来自Delta的方向性敞口和Theta盈亏。其中 Theta盈亏还取决于实际的日内方差 :math:`\frac{\Delta S^2}{S^2}` 与 定价方差 :math:`\sigma^2\Delta t` 之间的差异。


指数期权的盈亏分解
-----------------------

接下来我们把指数期权的盈亏分解到其各个成分股上。简洁起见，定义
:math:`\Delta_I = \frac{\partial V_I}{\partial I}` 为指数期权的Delta，:math:`\Theta_I = \frac{\partial V_I}{\partial t}` 为指数期权的Theta，


根据

.. math:: 
    \Delta I = \sum_i w_i \Delta S_i 
    :label: eq:17

定义 

.. math:: 
    n_I = \frac{\Delta I}{I\sigma_I\sqrt{\Delta t}}\\
    n_i = \frac{\Delta S_i }{S_i\sigma_i\sqrt{\Delta t}}


那么

.. math:: 
    \begin{aligned}
     n_I^2 &= \frac{\Delta I ^2}{I^2\sigma_I^2\Delta t}\\
     &=\frac{(\sum_i w_i \Delta S_i )^2}{I^2\sigma_I^2\Delta t}\\
     &= \frac{\sum_{i,j}w_i w_j \Delta S_i \Delta S_j}{I^2\sigma_I^2\Delta t}\\
     &= \sum_{i,j} \frac{\Delta S_i}{S_i \sigma_i \sqrt{\Delta t}}\frac{\Delta S_j}{S_j \sigma_j \sqrt{\Delta t}} \frac{w_i S_i w_jS_j\sigma_i\sigma_j}{I^2\sigma_I^2}\\
     &= \frac{1}{\sigma_I^2}\sum_{i,j} p_i p_j \sigma_i \sigma_j n_i n_j 
     \end{aligned}
    :label: eq:18


将 :eq:`eq:18` 式代入到 :eq:`eq:16` 式中可以得到

.. math:: 
    \begin{aligned}
    \Delta V_I &= \Delta_I \sum_i w_i \Delta S_i + \frac{\Theta_I\Delta t}{\sigma_I^2}(\sigma_I^2 - \sum_{i,j}p_ip_j\sigma_i\sigma_jn_in_j)\\
    &= \Delta_I \sum_i w_i \Delta S_i + \frac{\Theta_I\Delta t}{\sigma_I^2}(\sum_{i}p_i^2\sigma_i^2 + \sum_{i\neq j} p_i p_j \sigma_i \sigma_j \rho_{i,j}- \sum_{i,j}p_ip_j\sigma_i\sigma_jn_in_j)\\
    &= \Delta_I \sum_i w_i \Delta S_i + \frac{\Theta_I\Delta t}{\sigma_I^2}(\sum_{i}p_i^2\sigma_i^2(1-n_i^2) + \sum_{i\neq j}p_ip_j\sigma_i \sigma_j(\bar{\rho}-n_in_j))
    \end{aligned}
    :label: eq:19

指数与成分股期权的组合盈亏
-------------------------------

根据 :eq:`eq:16` 式，各成分股期权的盈亏可以写成

.. math:: 
    \Delta V_i = \Delta_i \Delta S_i + \Theta_i \Delta t(1-n_i^2)
    :label: eq:20

.. math:: 
    \begin{aligned}
    \Delta V_I + \sum_i \Delta V_i &= \sum_i (\Delta_I w_i  + \Delta_i)\Delta S_i + \sum_i (\frac{\Theta_I\Delta t}{\sigma_I^2}p_i^2\sigma_i^2 +\Theta_i\Delta t)(1-n_i^2) + \frac{\Theta_I\Delta t}{\sigma_I^2}\sum_{i \neq j}p_ip_j\sigma_i\sigma_j(\bar{\rho}-n_in_j)
    \end{aligned}
    :label: eq:21

从 :eq:`eq:21` 式可以看到，指数与其成分股期权组合的盈亏可以最终分解到指数各个成分股上的方向性盈亏，波动率盈亏以及由平均相关系数所关联的交叉项（非对角项）盈亏。假设个股与指数期权公允定价或者近似公允定价，那么累计波动率盈亏应近似为0（ :math:`1-n_i^2 \approx 0`），方向性近似满足各成分股期权的 :math:`\Delta_i \approx -w_i\Delta_I`,那么整个投资组合的方向性敞口也近似为0。投资组合的盈亏主要取决于交叉项，即
:math:`\frac{\Theta_I\Delta t}{\sigma_I^2}\sum_{i \neq j}p_ip_j\sigma_i\sigma_j(\bar{\rho}-n_in_j)`。盈亏来自于定价平均相关系数 :math:`\bar{\rho}` 与实际相关系数 :math:`n_in_j` 之间的差异。当我们做多相关性时，即实际相关系数趋于走扩（:math:`\bar{\rho} - n_in_j` 为负），可以通过long指数Gamma，short成分股Gamma（ :math:`\Theta_I` 为负）实现套利，反之亦然。


根据 :eq:`eq:21` 式，我们可以对指数与成分股期权组合进行风险分析与盈亏归因，

.. note:: 
    **方向性**

    风险因子，是分配到各个成分股上的方向性敞口，为

    .. math::
        (\Delta_I w_i  + \Delta_i)S_i

    盈亏为

    .. math::
         (\Delta_I w_i  + \Delta_i)\Delta S_i











