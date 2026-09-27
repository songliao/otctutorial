======================
交互式模型演示
======================

本页面提供几个可直接在浏览器中运行的演示。修改参数后点击 **绘制**，页面会在下方生成图形。

.. tab-set::

    .. tab-item:: CIR 路径模拟

        使用 Euler-Maruyama 方法模拟 CIR 过程：

        .. math::
            dX_t = \lambda(\mu - X_t)dt + \sigma\sqrt{X_t}dW_t

        .. raw:: html

            <div class="interactive-demo">
              <div class="demo-controls">
                <label>初始值 X0 <input id="cir-x0" type="number" value="0.03" step="0.001" min="0"></label>
                <label>回归速度 λ <input id="cir-lambda" type="number" value="2.0" step="0.1" min="0"></label>
                <label>长期均值 μ <input id="cir-mu" type="number" value="0.04" step="0.001" min="0"></label>
                <label>波动率 σ <input id="cir-sigma" type="number" value="0.15" step="0.01" min="0"></label>
                <label>路径数 <input id="cir-paths" type="number" value="20" step="1" min="1" max="200"></label>
                <button id="cir-draw" type="button">绘制</button>
              </div>
              <canvas id="cir-canvas" width="900" height="420"></canvas>
            </div>

    .. tab-item:: 期权到期盈亏

        计算并绘制欧式期权到期收益曲线。输入单位为合约倍数 1 的每股盈亏。

        .. raw:: html

            <div class="interactive-demo">
              <div class="demo-controls">
                <label>现价 S0 <input id="opt-s0" type="number" value="100" step="1" min="0"></label>
                <label>行权价 K <input id="opt-k" type="number" value="105" step="1" min="0"></label>
                <label>期权费 <input id="opt-premium" type="number" value="3.0" step="0.1" min="0"></label>
                <label>方向
                  <select id="opt-direction">
                    <option value="call">买看涨</option>
                    <option value="put">买看跌</option>
                    <option value="covered-call">卖看涨备兑</option>
                  </select>
                </label>
                <button id="opt-draw" type="button">绘制</button>
              </div>
              <canvas id="opt-canvas" width="900" height="420"></canvas>
            </div>

.. raw:: html

    <style>
      .interactive-demo {
        margin: 1rem 0 2rem;
        padding: 1rem;
        border: 1px solid rgba(0,0,0,.12);
        border-radius: .5rem;
      }
      .demo-controls {
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
        align-items: end;
        margin-bottom: 1rem;
      }
      .demo-controls label {
        display: grid;
        gap: .25rem;
        font-size: .85rem;
      }
      .demo-controls input,
      .demo-controls select {
        min-width: 110px;
      }
      .demo-controls button {
        padding: .4rem 1rem;
        border-radius: .35rem;
        border: 1px solid var(--color-foreground-border, #888);
        cursor: pointer;
      }
      .interactive-demo canvas {
        width: 100%;
        height: auto;
        max-width: 900px;
        border: 1px solid rgba(0,0,0,.08);
        border-radius: .35rem;
        background: #fff;
      }
    </style>

    <script>
      (() => {
        function setupCanvas(canvas) {
          const ctx = canvas.getContext('2d');
          const w = canvas.width, h = canvas.height;
          ctx.clearRect(0, 0, w, h);
          ctx.fillStyle = '#fff';
          ctx.fillRect(0, 0, w, h);
          return {ctx, w, h};
        }

        function drawAxes(ctx, w, h, margin, xmin, xmax, ymin, ymax) {
          const x0 = margin.left, x1 = w - margin.right;
          const y0 = h - margin.bottom, y1 = margin.top;
          ctx.strokeStyle = '#d0d7de';
          ctx.lineWidth = 1;
          ctx.beginPath();
          ctx.moveTo(x0, y0);
          ctx.lineTo(x1, y0);
          ctx.moveTo(x0, y0);
          ctx.lineTo(x0, y1);
          ctx.stroke();

          ctx.fillStyle = '#57606a';
          ctx.font = '12px system-ui, sans-serif';
          const xTicks = 5, yTicks = 5;
          for (let i = 0; i <= xTicks; i++) {
            const v = xmin + (xmax - xmin) * i / xTicks;
            const x = x0 + (x1 - x0) * i / xTicks;
            ctx.fillText(v.toFixed(2), x - 12, y0 + 18);
          }
          for (let i = 0; i <= yTicks; i++) {
            const v = ymin + (ymax - ymin) * i / yTicks;
            const y = y0 - (y0 - y1) * i / yTicks;
            ctx.fillText(v.toFixed(3), x0 - 45, y + 4);
          }
          return {
            x: value => x0 + (value - xmin) / (xmax - xmin) * (x1 - x0),
            y: value => y0 - (value - ymin) / (ymax - ymin) * (y0 - y1)
          };
        }

        function drawCir() {
          const canvas = document.getElementById('cir-canvas');
          const x0v = Number(document.getElementById('cir-x0').value);
          const lambda = Number(document.getElementById('cir-lambda').value);
          const mu = Number(document.getElementById('cir-mu').value);
          const sigma = Number(document.getElementById('cir-sigma').value);
          const paths = Math.max(1, Math.min(200, Number(document.getElementById('cir-paths').value)));
          if (![x0v, lambda, mu, sigma].every(Number.isFinite) || x0v < 0 || lambda < 0 || sigma < 0) {
            alert('请输入有效的非负参数。');
            return;
          }

          const n = 250, tmax = 2.0, dt = tmax / n;
          const series = [];
          let vmin = Infinity, vmax = -Infinity;
          for (let p = 0; p < paths; p++) {
            let x = x0v;
            const arr = [x];
            for (let i = 0; i < n; i++) {
              const z = Math.sqrt(-2 * Math.log(Math.random() + 1e-15)) *
                        Math.cos(2 * Math.PI * Math.random());
              x += lambda * (mu - x) * dt + sigma * Math.sqrt(Math.max(x, 0) * dt) * z;
              x = Math.max(x, 0);
              arr.push(x);
              vmin = Math.min(vmin, x);
              vmax = Math.max(vmax, x);
            }
            series.push(arr);
          }

          const {ctx, w, h} = setupCanvas(canvas);
          const margin = {left: 60, right: 30, top: 30, bottom: 45};
          const scale = drawAxes(ctx, w, h, margin, 0, tmax,
                                Math.max(0, vmin - 0.002), vmax + 0.002);
          const palette = ['#2563eb', '#dc2626', '#16a34a', '#ea580c', '#7c3aed'];
          series.forEach((arr, idx) => {
            ctx.beginPath();
            ctx.strokeStyle = palette[idx % palette.length];
            ctx.globalAlpha = 0.75;
            arr.forEach((value, i) => {
              const x = scale.x(i * dt), y = scale.y(value);
              i ? ctx.lineTo(x, y) : ctx.moveTo(x, y);
            });
            ctx.stroke();
          });
          ctx.globalAlpha = 1;
          ctx.fillStyle = '#111';
          ctx.font = '14px system-ui, sans-serif';
          ctx.fillText('CIR 路径模拟', margin.left + 5, 20);
        }

        function drawOption() {
          const s0 = Number(document.getElementById('opt-s0').value);
          const k = Number(document.getElementById('opt-k').value);
          const premium = Number(document.getElementById('opt-premium').value);
          const direction = document.getElementById('opt-direction').value;
          if (![s0, k, premium].every(Number.isFinite) || s0 < 0 || k < 0 || premium < 0) {
            alert('请输入有效的非负参数。');
            return;
          }
          const xmin = Math.max(0, Math.min(s0, k) * 0.6);
          const xmax = Math.max(s0, k) * 1.4;
          const points = [];
          let ymin = Infinity, ymax = -Infinity;
          for (let i = 0; i <= 200; i++) {
            const s = xmin + (xmax - xmin) * i / 200;
            let payoff = 0;
            if (direction === 'call') payoff = Math.max(s - k, 0) - premium;
            else if (direction === 'put') payoff = Math.max(k - s, 0) - premium;
            else payoff = -Math.max(s - k, 0) + premium;
            points.push([s, payoff]);
            ymin = Math.min(ymin, payoff);
            ymax = Math.max(ymax, payoff);
          }
          if (ymax - ymin < 1e-9) { ymin -= 1; ymax += 1; }

          const canvas = document.getElementById('opt-canvas');
          const {ctx, w, h} = setupCanvas(canvas);
          const margin = {left: 60, right: 30, top: 30, bottom: 45};
          const scale = drawAxes(ctx, w, h, margin, xmin, xmax, ymin - 0.05 * (ymax - ymin), ymax + 0.05 * (ymax - ymin));
          const yZero = scale.y(0);
          ctx.strokeStyle = '#8c959f';
          ctx.beginPath(); ctx.moveTo(margin.left, yZero); ctx.lineTo(w - margin.right, yZero); ctx.stroke();

          ctx.beginPath();
          ctx.strokeStyle = '#2563eb';
          ctx.lineWidth = 2.5;
          points.forEach(([s, v], i) => {
            const x = scale.x(s), y = scale.y(v);
            i ? ctx.lineTo(x, y) : ctx.moveTo(x, y);
          });
          ctx.stroke();
          ctx.lineWidth = 1;
          ctx.fillStyle = '#111';
          ctx.font = '14px system-ui, sans-serif';
          ctx.fillText('期权到期盈亏', margin.left + 5, 20);
        }

        document.getElementById('cir-draw').addEventListener('click', drawCir);
        document.getElementById('opt-draw').addEventListener('click', drawOption);
        document.addEventListener('DOMContentLoaded', drawCir);
      })();
    </script>

.. note::

    当前页面所有计算均在浏览器本地完成，刷新页面后图形会按默认参数重新绘制。
