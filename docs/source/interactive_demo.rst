======================
交互式模型演示
======================

本页面提供几个可直接在浏览器中运行的演示。修改参数后点击 **绘制**，页面会生成网页原生 SVG 图形。

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
              <div id="cir-chart" class="chart-container"></div>
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
              <div id="opt-chart" class="chart-container"></div>
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
      .chart-container {
        border: 1px solid rgba(0,0,0,.08);
        border-radius: .35rem;
        background: #fff;
        padding: .5rem;
      }
      .chart-container svg {
        width: 100%;
        height: auto;
        display: block;
      }
      .chart-line {
        fill: none;
        stroke-width: 2.2;
        stroke-linecap: round;
        stroke-linejoin: round;
      }
      .axis-line { stroke: #d0d7de; stroke-width: 1; }
      .grid-line { stroke: #eaeef2; stroke-width: 1; }
      .axis-text { fill: #57606a; font-size: 12px; font-family: system-ui, sans-serif; }
      .chart-title { fill: #111; font-size: 16px; font-weight: 600; font-family: system-ui, sans-serif; }
      .crosshair-line { stroke: rgba(37, 99, 235, .45); stroke-width: 1; stroke-dasharray: 4 4; }
      .crosshair-dot { fill: #2563eb; stroke: #fff; stroke-width: 1.5; }
      .crosshair-label { fill: #111; font-size: 13px; font-weight: 600; font-family: system-ui, sans-serif; }
      #opt-chart svg { cursor: crosshair; }
    </style>

    <script>
      (() => {
        const NS = 'http://www.w3.org/2000/svg';

        function createSvg(container, width = 920, height = 460, margin = {}) {
          const m = {top: 45, right: 35, bottom: 65, left: 75, ...margin};
          container.replaceChildren();
          const svg = document.createElementNS(NS, 'svg');
          svg.setAttribute('viewBox', `0 0 ${width} ${height}`);
          svg.setAttribute('role', 'img');
          svg.dataset.width = width;
          svg.dataset.height = height;
          svg.dataset.margin = JSON.stringify(m);
          container.appendChild(svg);
          return svg;
        }

        function el(name, attrs = {}, text) {
          const node = document.createElementNS(NS, name);
          for (const [key, value] of Object.entries(attrs)) node.setAttribute(key, value);
          if (text !== undefined) node.textContent = text;
          return node;
        }

        function addCrosshair(svg, points, xScale, yScale, xFormat, yFormat) {
          const width = Number(svg.dataset.width);
          const height = Number(svg.dataset.height);
          const hoverGroup = el('g', {class: 'crosshair', opacity: 0});
          const vLine = el('line', {class: 'crosshair-line'});
          const hLine = el('line', {class: 'crosshair-line'});
          const dot = el('circle', {r: 4.5, class: 'crosshair-dot'});
          const label = el('text', {class: 'crosshair-label'});
          hoverGroup.append(vLine, hLine, dot, label);
          svg.appendChild(hoverGroup);

          svg.addEventListener('mousemove', event => {
            const rect = svg.getBoundingClientRect();
            const x = (event.clientX - rect.left) * width / rect.width;
            const y = (event.clientY - rect.top) * height / rect.height;
            let nearest = points[0];
            for (const point of points) {
              if (Math.abs(xScale(point[0]) - x) < Math.abs(xScale(nearest[0]) - x)) nearest = point;
            }
            const px = xScale(nearest[0]);
            const py = yScale(nearest[1]);
            vLine.setAttribute('x1', px); vLine.setAttribute('x2', px);
            vLine.setAttribute('y1', 45); vLine.setAttribute('y2', height - 65);
            hLine.setAttribute('x1', 75); hLine.setAttribute('x2', width - 35);
            hLine.setAttribute('y1', py); hLine.setAttribute('y2', py);
            dot.setAttribute('cx', px); dot.setAttribute('cy', py);
            label.textContent = `标的 ${xFormat(nearest[0])}，盈亏 ${yFormat(nearest[1])}`;
            const lx = Math.min(Math.max(px + 12, 82), width - 220);
            label.setAttribute('x', lx); label.setAttribute('y', Math.max(py - 12, 62));
            hoverGroup.setAttribute('opacity', 1);
          });

          svg.addEventListener('mouseleave', () => hoverGroup.setAttribute('opacity', 0));
        }

        function drawSvgChart(container, title, series, xLabels, yLabels, color = '#2563eb', crosshair = null) {
          const width = Number(container.firstElementChild.dataset.width);
          const height = Number(container.firstElementChild.dataset.height);
          const margin = JSON.parse(container.firstElementChild.dataset.margin);
          const svg = container.firstElementChild;
          const plotW = width - margin.left - margin.right;
          const plotH = height - margin.top - margin.bottom;

          const xScale = value => margin.left + (value - xLabels.min) / (xLabels.max - xLabels.min) * plotW;
          const yScale = value => margin.top + plotH - (value - yLabels.min) / (yLabels.max - yLabels.min) * plotH;

          for (let i = 0; i <= 5; i++) {
            const yValue = yLabels.min + (yLabels.max - yLabels.min) * i / 5;
            const y = yScale(yValue);
            svg.appendChild(el('line', {x1: margin.left, y1: y, x2: width - margin.right, y2: y, class: 'grid-line'}));
            svg.appendChild(el('text', {x: margin.left - 12, y: y + 4, 'text-anchor': 'end', class: 'axis-text'}, yLabels.format(yValue)));
          }
          for (let i = 0; i <= 5; i++) {
            const xValue = xLabels.min + (xLabels.max - xLabels.min) * i / 5;
            const x = xScale(xValue);
            svg.appendChild(el('text', {x, y: height - margin.bottom + 24, 'text-anchor': 'middle', class: 'axis-text'}, xLabels.format(xValue)));
          }

          svg.appendChild(el('line', {x1: margin.left, y1: height - margin.bottom, x2: width - margin.right, y2: height - margin.bottom, class: 'axis-line'}));
          svg.appendChild(el('line', {x1: margin.left, y1: margin.top, x2: margin.left, y2: height - margin.bottom, class: 'axis-line'}));
          svg.appendChild(el('text', {x: margin.left + 5, y: 28, class: 'chart-title'}, title));

          series.forEach(item => {
            const path = item.points.map((point, idx) =>
              `${idx ? 'L' : 'M'}${xScale(point[0]).toFixed(2)},${yScale(point[1]).toFixed(2)}`
            ).join(' ');
            const pathNode = el('path', {d: path, class: 'chart-line', stroke: item.color || color});
            if (item.opacity) pathNode.setAttribute('opacity', item.opacity);
            svg.appendChild(pathNode);
          });

          if (crosshair) addCrosshair(svg, crosshair.points, xScale, yScale, crosshair.xFormat, crosshair.yFormat);
        }

        function drawCir() {
          const container = document.getElementById('cir-chart');
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
          const palette = ['#2563eb', '#dc2626', '#16a34a', '#ea580c', '#7c3aed'];
          for (let p = 0; p < paths; p++) {
            let x = x0v;
            const points = [[0, x]];
            for (let i = 0; i < n; i++) {
              const z = Math.sqrt(-2 * Math.log(Math.random() + 1e-15)) *
                        Math.cos(2 * Math.PI * Math.random());
              x += lambda * (mu - x) * dt + sigma * Math.sqrt(Math.max(x, 0) * dt) * z;
              x = Math.max(x, 0);
              points.push([(i + 1) * dt, x]);
              vmin = Math.min(vmin, x);
              vmax = Math.max(vmax, x);
            }
            series.push({points, color: palette[p % palette.length], opacity: .78});
          }

          createSvg(container);
          const pad = Math.max((vmax - vmin) * .08, .002);
          drawSvgChart(container, 'CIR 路径模拟', series,
            {min: 0, max: tmax, format: v => v.toFixed(1)},
            {min: Math.max(0, vmin - pad), max: vmax + pad, format: v => v.toFixed(3)});
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
          const xmin = Math.max(0, Math.min(s0, k) * .6);
          const xmax = Math.max(s0, k) * 1.4;
          const points = [];
          let ymin = Infinity, ymax = -Infinity;
          for (let i = 0; i <= 220; i++) {
            const s = xmin + (xmax - xmin) * i / 220;
            let payoff = 0;
            if (direction === 'call') payoff = Math.max(s - k, 0) - premium;
            else if (direction === 'put') payoff = Math.max(k - s, 0) - premium;
            else payoff = -Math.max(s - k, 0) + premium;
            points.push([s, payoff]);
            ymin = Math.min(ymin, payoff);
            ymax = Math.max(ymax, payoff);
          }
          if (ymax - ymin < 1e-9) { ymin -= 1; ymax += 1; }

          const container = document.getElementById('opt-chart');
          createSvg(container);
          const pad = (ymax - ymin) * .08;
          drawSvgChart(container, '期权到期盈亏', [{points}],
            {min: xmin, max: xmax, format: v => v.toFixed(1)},
            {min: ymin - pad, max: ymax + pad, format: v => v.toFixed(2)},
            '#2563eb',
            {points, xFormat: v => v.toFixed(1), yFormat: v => v.toFixed(2)});
        }

        document.getElementById('cir-draw').addEventListener('click', drawCir);
        document.getElementById('opt-draw').addEventListener('click', drawOption);
        document.addEventListener('DOMContentLoaded', drawCir);
      })();
    </script>

.. note::

    图形由浏览器动态生成，输出为 SVG 矢量图，可直接缩放、复制到网页或另存为 `.svg` 文件。
