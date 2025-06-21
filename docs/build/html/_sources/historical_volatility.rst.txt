关于历史波动率
========================================

简单隔日历史波动率计算方法
----------------------------------------

标的价格收盘价时序数据 :math:`P_i`, :math:`i=0,1,...,N`， 其时间间隔为 :math:`\Delta t`，一般取每交易日收盘价，:math:`\Delta t` 即 1/年交易天数，在本例中我们取 :math:`\Delta t= \frac{1}{242}` .

对数收益率 :math:`r_i = \ln\frac{P_i}{P_{i-1}}`, :math:`i = 1,..., N`

以 :math:`m` 日窗口计算历史波动率(方差)序列

.. math:: 
    \sigma_i^2 =\frac{\Delta t}{m} \sum_{j=m(i-1)+1}^{j=mi}r_j^2
    \label{eq:hv_1}


其中 :math:`i=1,..., \lfloor N/m \rfloor`, :math:`\lfloor N/m \rfloor` 表示不超过 :math:`N/m` 的整数。

.. tip:: 
    我们用方差作为我们要研究的随机样本序列，一个显而易见的原因是方差是波动率的平方，平方后，可以有效平抑极端值对均值等统计指标的影响，
    :math:`\sqrt{a^2 + b^2} < a + b, \forall a,b >0`


