CIR波动率期限结构的构建
================================

1. 模型
------

CIR 对**方差**建模（SDE）：

.. math::

    dv_t = \lambda(\mu - v_t)dt + \sigma \sqrt{v_t} dW_t

三个参数从历史波动率数据校准（OLS 闭式解）：

- :math:`\lambda`：均值回归速度
- :math:`\mu`：方差长期中枢
- :math:`\sigma`：方差自身的波动率

校准公式见 :doc:`cir_model_ols_calibration` 公式 (8)-(11)。


2. 方差的期望路径（条件一阶矩）
--------------------------------

CIR 的**条件期望**（conditional expectation，即方差的**一阶矩**）有闭式解：

.. math::

    E[v_t \mid v_0] = \mu + (v_0 - \mu) \cdot e^{-\lambda t}

.. note::

    这是 :math:`E[v_t]`（方差过程的期望值），**不是** :math:`\text{Var}[v_t]`（方差过程自身的条件方差）。两者的区别：

    .. list-table::
       :header-rows: 1

       * - 量
         - 含义
         - 公式
       * - :math:`E[v_t \mid v_0]`
         - 未来方差的期望水平
         - :math:`\mu + (v_0-\mu)e^{-\lambda t}`
       * - :math:`\text{Var}[v_t \mid v_0]`
         - 未来方差的离散程度
         - :math:`v_0\frac{\sigma^2}{\lambda}(e^{-\lambda t}-e^{-2\lambda t}) + \frac{\mu\sigma^2}{2\lambda}(1-e^{-\lambda t})^2`

    前者（Wikipedia 的条件均值）用于**期限结构**——我们关心方差往哪走；后者（Wikipedia 的条件方差）描述这个走向有多不确定。

这是从 :math:`v_0` 出发、以速率 :math:`\lambda` 指数衰减到 :math:`\mu` 的曲线。


3. 期限结构：路径平均方差
---------------------------

期限结构不应只看终端点 :math:`E[v_T]`，而应看整段路径 :math:`[0,T]` 上的平均：

.. math::

    V(T) = E\left[\frac{1}{T}\int_0^{T} v_t dt\right]

交换积分与期望（Fubini），并将条件期望 :math:`E[v_t]` 代入：

.. math::

    \begin{aligned}
    V(T) &= \frac{1}{T}\int_0^{T} E[v_t] dt \\
    &= \frac{1}{T}\int_0^{T} \left[\mu + (v_0 - \mu)e^{-\lambda t}\right] dt \\
    &= \mu + (v_0 - \mu) \cdot \frac{1 - e^{-\lambda T}}{\lambda T}
    \end{aligned}

记衰减因子：

.. math::

    D(T) = \frac{1 - e^{-\lambda T}}{\lambda T}

则：

.. math::

    \boxed{V(T) = \mu + (v_0 - \mu) \cdot D(T)}

:math:`D(T)` 的含义：初始偏离 :math:`(v_0 - \mu)` 在 0 到 :math:`T` 这段路径上的**平均残留比例**。

极限性质：

.. math::

    T \to 0:\; D(T) \to 1,\quad V(T) \to v_0 \qquad （短期完全看当前）

    T \to \infty:\; D(T) \to 0,\quad V(T) \to \mu \qquad （远期只看长期中枢）


4. 波动率期限结构
------------------

对 :math:`V(T)` 取平方根：

.. math::

    \boxed{\sigma(T) = \sqrt{V(T)} = \sqrt{\mu + (v_0 - \mu) \cdot \frac{1 - e^{-\lambda T}}{\lambda T}}}


5. 模型预测起点 :math:`(T=1)`
--------------------------------

若直接从 :math:`T=0` 开始，起点就是原始输入 :math:`v_0`，无法体现模型自身的动态偏离。因此 :math:`T=1` 的起点取模型的一步预测值：

.. math::

    v_1 = v_0 + \lambda(\mu - v_0)\Delta t

其中 :math:`\Delta t = 1/242`（一个交易日）。

期限结构从 :math:`T=1` 开始，:math:`T=1` 取模型的一步预测值作为起点；:math:`T>1` 仍以原始 :math:`v_0` 为入参计算路径平均：

.. math::

    V(T) = \begin{cases}
    v_1 = v_0 + \lambda(\mu - v_0)\Delta t & T = 1 \\
    \mu + (v_0 - \mu) \cdot \dfrac{1 - e^{-\lambda T / 242}}{\lambda \cdot T / 242} & T > 1
    \end{cases}


6. 算法步骤
-----------

::

    输入: 当前波动率 spot_vol, 校准参数 (λ, μ), 期限列表 [T₁, T₂, ...]
    输出: 每个 T 对应的期限结构波动率

    1. v₀ = spot_vol²
    2. dt = 1/242
    3. v₁ = v₀ + λ(μ - v₀)·dt            ← 模型一步预测（仅用于 T=1）
    4. 对每个 T:
       a. 若 T == 1:
            V = v₁
       b. 若 T > 1:
            D = (1 - e^{-λ·T/242}) / (λ·T/242)
            V = μ + (v₀ - μ)·D
       c. σ(T) = √V


7. 与模拟路径的关系
--------------------

期限结构曲线等价于：对大量 CIR 模拟路径，每条算 :math:`\sqrt{\frac{1}{T}\sum v_t}`，然后取所有路径的均值。当路径数 :math:`\to\infty` 时收敛到上述闭式解。
