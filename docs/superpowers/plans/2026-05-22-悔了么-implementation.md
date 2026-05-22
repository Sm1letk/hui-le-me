# 悔了么 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单页面 HTML 工具，帮用户计算"如果当时买了这只股票现在能赚多少钱"，附带调侃文案。

**Architecture:** 纯单文件 `index.html`，内联 CSS + JavaScript，零依赖。HTML 负责结构，CSS 实现黑底白字响应式布局，JS 处理实时换算、计算逻辑和结果渲染。

**Tech Stack:** 原生 HTML5 / CSS3 / Vanilla JS，无框架，无构建工具。

---

## 文件结构

```
index.html          ← 整个应用，包含 HTML / <style> / <script>
```

---

## Task 1：创建 HTML 结构 + 完整 CSS

**Files:**
- Create: `index.html`

- [ ] **Step 1：创建 `index.html`，写入完整 HTML 结构和 CSS**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>悔了么</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: #0a0a0a;
      color: #fff;
      font-family: 'PingFang SC', 'Helvetica Neue', Arial, sans-serif;
      min-height: 100vh;
      padding: 32px 16px 64px;
    }

    main { max-width: 480px; margin: 0 auto; }

    /* ── Header ── */
    header { text-align: center; margin-bottom: 32px; }
    header h1 { font-size: 38px; font-weight: 900; letter-spacing: 6px; }
    header p  { color: #555; font-size: 13px; margin-top: 8px; letter-spacing: 1px; }

    /* ── Cards ── */
    .card {
      background: #141414;
      border: 1px solid #222;
      border-radius: 16px;
      padding: 20px;
      margin-bottom: 16px;
    }

    /* ── Fields ── */
    .field { margin-bottom: 14px; }
    .field label {
      display: block;
      color: #666;
      font-size: 11px;
      letter-spacing: 1px;
      text-transform: uppercase;
      margin-bottom: 6px;
    }
    .field input {
      width: 100%;
      background: #1e1e1e;
      border: 1px solid #2a2a2a;
      border-radius: 8px;
      padding: 12px 14px;
      color: #fff;
      font-size: 15px;
      font-family: inherit;
      outline: none;
      transition: border-color 0.2s;
      -webkit-appearance: none;
      appearance: none;
    }
    .field input[type="number"]::-webkit-inner-spin-button,
    .field input[type="number"]::-webkit-outer-spin-button { -webkit-appearance: none; }
    .field input::placeholder { color: #444; }
    .field input:focus { border-color: #444; }
    .field input.error { border-color: #ff4444 !important; }

    /* ── Two-column row ── */
    .field-row { display: flex; gap: 12px; }
    .field-row .field { flex: 1; }

    /* ── Two-in-one hint box ── */
    .two-in-one {
      background: #1a1a1a;
      border-radius: 10px;
      padding: 14px;
      margin-bottom: 16px;
    }
    .two-in-one .hint {
      color: #555;
      font-size: 11px;
      letter-spacing: 1px;
      margin-bottom: 10px;
    }
    .two-in-one .field { margin-bottom: 0; }

    /* ── Button ── */
    #calcBtn {
      width: 100%;
      background: #fff;
      color: #000;
      border: none;
      border-radius: 10px;
      padding: 14px;
      font-size: 15px;
      font-weight: 700;
      letter-spacing: 2px;
      cursor: pointer;
      font-family: inherit;
      transition: opacity 0.15s;
    }
    #calcBtn:hover  { opacity: 0.85; }
    #calcBtn:active { opacity: 0.70; }

    /* ── Result card ── */
    #resultCard { animation: fadeIn 0.35s ease; }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to   { opacity: 1; transform: translateY(0);    }
    }

    /* ── Price comparison row ── */
    .price-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding-bottom: 20px;
      margin-bottom: 20px;
      border-bottom: 1px solid #1f1f1f;
    }
    .price-item { text-align: center; }
    .price-label { display: block; color: #555; font-size: 10px; margin-bottom: 4px; }
    .price-value { color: #fff; font-size: 20px; font-weight: 700; }
    .arrow { color: #333; font-size: 18px; }

    /* ── Big missed amount ── */
    .missed-section { text-align: center; margin-bottom: 16px; }
    .missed-label   { color: #555; font-size: 12px; letter-spacing: 2px; margin-bottom: 8px; }
    .missed-amount  { font-size: 44px; font-weight: 900; letter-spacing: -1px; }

    /* ── Humor copy ── */
    .humor-box {
      background: #1a1a1a;
      border-radius: 10px;
      padding: 14px;
      text-align: center;
      color: #666;
      font-size: 13px;
      line-height: 2;
      margin-bottom: 12px;
    }
    .humor-box span { color: #fff; font-weight: 600; }

    /* ── Taunt ── */
    .taunt { text-align: center; color: #333; font-size: 12px; }
  </style>
</head>
<body>
  <main>
    <header>
      <h1>悔了么</h1>
      <p>如果当时买了，现在赚了多少</p>
    </header>

    <section class="card" id="inputCard">
      <div class="field">
        <label>股票名称</label>
        <input type="text" id="stockName" placeholder="例：宁德时代">
      </div>

      <div class="field-row">
        <div class="field">
          <label>当时价格（元）</label>
          <input type="number" id="buyPrice" placeholder="0.00" min="0" step="0.01">
        </div>
        <div class="field">
          <label>现在价格（元）</label>
          <input type="number" id="currentPrice" placeholder="0.00" min="0" step="0.01">
        </div>
      </div>

      <div class="two-in-one">
        <p class="hint">填股数或金额，二选一即可</p>
        <div class="field-row">
          <div class="field">
            <label>打算买多少股</label>
            <input type="number" id="shares" placeholder="股" min="0" step="1">
          </div>
          <div class="field">
            <label>或投入多少钱</label>
            <input type="number" id="amount" placeholder="元" min="0" step="0.01">
          </div>
        </div>
      </div>

      <button id="calcBtn">算一算</button>
    </section>

    <section class="card" id="resultCard" style="display:none">
      <div class="price-row">
        <div class="price-item">
          <span class="price-label">当时</span>
          <span class="price-value" id="resBuyPrice"></span>
        </div>
        <span class="arrow">→</span>
        <div class="price-item">
          <span class="price-label">现在</span>
          <span class="price-value" id="resCurrentPrice"></span>
        </div>
        <div class="price-item">
          <span class="price-label">涨幅</span>
          <span class="price-value" id="resGain"></span>
        </div>
      </div>

      <div class="missed-section">
        <p class="missed-label">你错失了</p>
        <p class="missed-amount" id="resMissed"></p>
      </div>

      <div class="humor-box" id="resHumor"></div>
      <p class="taunt" id="resTaunt"></p>
    </section>
  </main>

  <script>
    /* JS will be added in Task 2 */
  </script>
</body>
</html>
```

- [ ] **Step 2：在浏览器中打开 `index.html`，确认视觉外观**

用浏览器直接打开文件，检查：
- 黑色背景，白色大标题"悔了么"
- 输入卡片结构正确（股票名称、两个价格并排、二选一区域、白色按钮）
- 结果卡片不可见（`display:none`）
- 手机宽度（375px）和桌面宽度下布局均正常

---

## Task 2：添加 JavaScript — 完整逻辑

**Files:**
- Modify: `index.html`（替换 `<script>` 块内容）

- [ ] **Step 1：将 `<script>` 注释替换为完整 JavaScript**

用以下内容替换 `index.html` 中的 `<script>/* JS will be added in Task 2 */</script>`：

```html
  <script>
    // ── DOM refs ──
    const buyPriceInput    = document.getElementById('buyPrice');
    const currentPriceInput = document.getElementById('currentPrice');
    const sharesInput      = document.getElementById('shares');
    const amountInput      = document.getElementById('amount');
    const calcBtn          = document.getElementById('calcBtn');
    const resultCard       = document.getElementById('resultCard');

    // tracks which of the two fields the user last typed into
    let lastEdited = null; // 'shares' | 'amount'

    // ── Number formatting ──
    function fmt(n) {
      return n.toLocaleString('zh-CN', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      });
    }

    // ── Two-in-one auto-sync ──
    function syncFromShares() {
      lastEdited = 'shares';
      const bp = parseFloat(buyPriceInput.value);
      const s  = parseFloat(sharesInput.value);
      amountInput.value = (!isNaN(bp) && bp > 0 && !isNaN(s) && s > 0)
        ? fmt(s * bp)
        : '';
    }

    function syncFromAmount() {
      lastEdited = 'amount';
      const bp = parseFloat(buyPriceInput.value);
      const a  = parseFloat(amountInput.value.replace(/,/g, ''));
      sharesInput.value = (!isNaN(bp) && bp > 0 && !isNaN(a) && a > 0)
        ? Math.floor(a / bp)
        : '';
    }

    sharesInput.addEventListener('input', syncFromShares);
    amountInput.addEventListener('input', syncFromAmount);

    // re-sync when buy price changes
    buyPriceInput.addEventListener('input', () => {
      if (lastEdited === 'shares') syncFromShares();
      else if (lastEdited === 'amount') syncFromAmount();
    });

    // ── Validation ──
    function validate() {
      [buyPriceInput, currentPriceInput, sharesInput, amountInput]
        .forEach(el => el.classList.remove('error'));

      let ok = true;

      if (!buyPriceInput.value || isNaN(parseFloat(buyPriceInput.value))) {
        buyPriceInput.classList.add('error'); ok = false;
      }
      if (!currentPriceInput.value || isNaN(parseFloat(currentPriceInput.value))) {
        currentPriceInput.classList.add('error'); ok = false;
      }

      const s = parseFloat(sharesInput.value);
      const a = parseFloat(amountInput.value.replace(/,/g, ''));
      if (isNaN(s) && isNaN(a)) {
        sharesInput.classList.add('error');
        amountInput.classList.add('error');
        ok = false;
      }

      return ok;
    }

    // ── Calculation (pure) ──
    function calculate(buyPrice, currentPrice, shares) {
      const invested     = shares * buyPrice;
      const currentValue = shares * currentPrice;
      const missed       = currentValue - invested;
      const gainPercent  = ((currentPrice - buyPrice) / buyPrice) * 100;
      return { invested, currentValue, missed, gainPercent };
    }

    // ── Taunt phrases ──
    const TAUNTS = [
      '「当时只差一个勇气」',
      '「纸上谈兵，白纸一张」',
      '「你的钱，在别人口袋里」',
      '「错过的不是股票，是自由」',
      '「下次一定」',
    ];

    // ── Render results ──
    function renderResult(buyPrice, currentPrice, shares, missed, gainPercent) {
      document.getElementById('resBuyPrice').textContent    = '¥' + fmt(buyPrice);
      document.getElementById('resCurrentPrice').textContent = '¥' + fmt(currentPrice);
      document.getElementById('resGain').textContent =
        (gainPercent >= 0 ? '+' : '') + gainPercent.toFixed(2) + '%';
      document.getElementById('resMissed').textContent = '¥' + fmt(missed);

      const luckin  = Math.floor(missed / 30);
      const iphone  = Math.floor(missed / 9999);
      const salary  = Math.floor(missed / 12000);

      document.getElementById('resHumor').innerHTML =
        `相当于 <span>${luckin.toLocaleString()}</span> 杯瑞幸咖啡<br>` +
        `或 <span>${iphone.toLocaleString()}</span> 台 iPhone 16 Pro Max<br>` +
        `或 <span>${salary.toLocaleString()}</span> 个月北京平均工资`;

      document.getElementById('resTaunt').textContent =
        TAUNTS[Math.floor(Math.random() * TAUNTS.length)];

      // re-trigger animation each time
      resultCard.style.display = 'none';
      // eslint-disable-next-line no-unused-expressions
      resultCard.offsetHeight; // force reflow
      resultCard.style.removeProperty('display');
    }

    // ── Main button handler ──
    calcBtn.addEventListener('click', () => {
      if (!validate()) return;

      const buyPrice     = parseFloat(buyPriceInput.value);
      const currentPrice = parseFloat(currentPriceInput.value);

      let shares;
      const rawShares = parseFloat(sharesInput.value);
      const rawAmount = parseFloat(amountInput.value.replace(/,/g, ''));

      if (lastEdited === 'amount' && !isNaN(rawAmount)) {
        shares = rawAmount / buyPrice;
      } else {
        shares = rawShares;
      }

      const { missed, gainPercent } = calculate(buyPrice, currentPrice, shares);
      renderResult(buyPrice, currentPrice, shares, missed, gainPercent);
    });
  </script>
```

- [ ] **Step 2：浏览器中验证以下测试用例**

**用例 A — 正常计算（填股数）：**
- 当时价格：`130`，现在价格：`260`，股数：`1000`
- 期望：错失金额 = `¥130,000.00`，涨幅 = `+100.00%`
- 瑞幸 = `4,333` 杯，iPhone = `13` 台，月薪 = `10` 个月

**用例 B — 填金额自动换算股数：**
- 当时价格：`10`，现在价格：`20`，投入金额：`10000`
- 期望：股数自动换算为 `1000`，错失金额 = `¥10,000.00`，涨幅 = `+100.00%`

**用例 C — 改买入价后金额自动更新：**
- 先填股数 `100`，买入价 `50` → 金额自动显示 `5,000.00`
- 再把买入价改为 `100` → 金额自动更新为 `10,000.00`

**用例 D — 验证校验：**
- 什么都不填直接点"算一算" → 当时价格、现在价格、股数、金额输入框均变红边框
- 填完后重新点击 → 红边框消失，结果正常显示

**用例 E — 多次点击动画重置：**
- 计算一次后，修改价格再次点击"算一算" → 结果卡片重新淡入

---

## Task 3：git init + 提交

**Files:**
- No code changes

- [ ] **Step 1：初始化 git 仓库**

```bash
git init
```

- [ ] **Step 2：添加 `.gitignore`**

```bash
echo ".superpowers/" > .gitignore
```

- [ ] **Step 3：提交**

```bash
git add index.html docs/ .gitignore
git commit -m "feat: 悔了么 初始版本

单文件 HTML 工具，输入股票买入价/现价/数量，
计算错失收益并展示调侃文案。"
```

---

## 规格覆盖核查

| 规格要求 | 对应 Task |
|----------|-----------|
| 单文件 HTML，零依赖 | Task 1 |
| 黑底白字，最大宽 480px 居中 | Task 1 CSS |
| 股票名称输入（仅展示） | Task 1 HTML |
| 当时价格 / 现在价格必填校验 | Task 2 `validate()` |
| 股数 / 金额二选一，实时换算 | Task 2 `syncFromShares` / `syncFromAmount` |
| 两者都填以最后编辑为准 | Task 2 `lastEdited` 变量 |
| 价格对比行（当时→现在→涨幅） | Task 1 HTML + Task 2 `renderResult` |
| 错失金额大字，千分位格式化 | Task 2 `fmt()` + `renderResult` |
| 调侃换算（瑞幸/iPhone/月薪） | Task 2 `renderResult` |
| 随机底部吐槽语 | Task 2 `TAUNTS` |
| 结果卡片 fade-in 动画 | Task 1 CSS `@keyframes fadeIn` + Task 2 reflow trick |
| 校验失败输入框变红 | Task 1 CSS `.error` + Task 2 `validate()` |
| 响应式（手机 + 桌面） | Task 1 CSS（无媒体查询，max-width 天然响应式） |
