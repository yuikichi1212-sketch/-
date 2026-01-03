<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
  <title>ゆいきち計算</title>
  <style>
    :root {
      --primary: #007aff;
      --bg: #f2f2f7;
      --card: #ffffff;
      --text: #1c1c1e;
      --gray: #8e8e93;
      --orange: #ff9500;
      --red: #ff3b30;
      --green: #34c759;
    }
    * { box-sizing: border-box; touch-action: manipulation; }
    body {
      margin: 0; padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: var(--bg); color: var(--text);
      display: flex; flex-direction: column; height: 100vh;
      overflow: hidden;
    }

    /* ヘッダー */
    header {
      text-align: center; padding: 12px;
      background: var(--card); border-bottom: 1px solid #d1d1d6;
      font-weight: 800; font-size: 18px; z-index: 10;
    }

    /* メインコンテンツエリア */
    main {
      flex: 1; overflow-y: auto; position: relative;
    }
    .tab-content {
      display: none; padding: 20px;
      height: 100%;
      flex-direction: column; align-items: center;
    }
    .tab-content.active { display: flex; }

    /* フッターナビ（タブ） */
    nav {
      background: var(--card); border-top: 1px solid #d1d1d6;
      display: flex; justify-content: space-around; padding-bottom: env(safe-area-inset-bottom);
    }
    .nav-btn {
      flex: 1; padding: 12px 0; border: none; background: none;
      font-size: 10px; color: var(--gray); font-weight: 600;
      display: flex; flex-direction: column; align-items: center; gap: 4px;
    }
    .nav-btn.active { color: var(--primary); }
    .nav-icon { font-size: 20px; }

    /* --- 電卓スタイル --- */
    .calc-display {
      width: 100%; max-width: 320px;
      background: #000; color: #fff;
      font-size: 48px; text-align: right; padding: 20px;
      border-radius: 12px; margin-bottom: 10px;
      overflow-x: auto; white-space: nowrap;
    }
    .calc-grid {
      display: grid; grid-template-columns: repeat(4, 1fr);
      gap: 12px; width: 100%; max-width: 320px;
    }
    .c-btn {
      aspect-ratio: 1; border-radius: 50%; border: none;
      font-size: 24px; font-weight: 500; cursor: pointer;
      background: #e5e5ea; color: #000;
      transition: opacity 0.2s;
    }
    .c-btn:active { opacity: 0.7; }
    .c-btn.op { background: var(--orange); color: #fff; }
    .c-btn.zero { grid-column: span 2; aspect-ratio: auto; border-radius: 40px; }
    .c-btn.top { background: #d4d4d2; }

    /* --- ツール共通 --- */
    .tool-card {
      background: var(--card); width: 100%; max-width: 340px;
      padding: 20px; border-radius: 16px; margin-bottom: 20px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05); text-align: center;
    }
    .big-time { font-size: 48px; font-variant-numeric: tabular-nums; font-weight: 300; margin: 10px 0; }
    .ctrl-row { display: flex; gap: 10px; justify-content: center; margin-top: 10px; }
    .t-btn {
      padding: 10px 24px; border-radius: 20px; border: none;
      font-weight: bold; color: #fff; cursor: pointer; font-size: 16px;
    }
    .start { background: var(--green); }
    .stop { background: var(--red); }
    .reset { background: var(--gray); }
    
    input.time-inp {
      width: 60px; font-size: 24px; text-align: center;
      border: 1px solid #ccc; border-radius: 8px; padding: 4px;
    }

    /* --- 通貨スタイル --- */
    .rate-row {
      display: flex; align-items: center; justify-content: space-between;
      margin-bottom: 12px; width: 100%;
    }
    .curr-inp {
      font-size: 24px; width: 140px; padding: 8px;
      text-align: right; border: 1px solid #ddd; border-radius: 8px;
    }
    .label { font-size: 20px; font-weight: bold; }
    .rate-set { font-size: 12px; color: var(--gray); margin-top: 20px; text-align: left; }
    .rate-set input { width: 60px; text-align: right; }

  </style>
</head>
<body>

<header>ゆいきち計算</header>

<main>
  <div id="tab-calc" class="tab-content active">
    <div class="calc-display" id="calc-disp">0</div>
    <div class="calc-grid">
      <button class="c-btn top" onclick="calcClear()">AC</button>
      <button class="c-btn top" onclick="calcInvert()">+/-</button>
      <button class="c-btn top" onclick="calcPercent()">%</button>
      <button class="c-btn op" onclick="calcOp('/')">÷</button>
      
      <button class="c-btn" onclick="calcNum('7')">7</button>
      <button class="c-btn" onclick="calcNum('8')">8</button>
      <button class="c-btn" onclick="calcNum('9')">9</button>
      <button class="c-btn op" onclick="calcOp('*')">×</button>
      
      <button class="c-btn" onclick="calcNum('4')">4</button>
      <button class="c-btn" onclick="calcNum('5')">5</button>
      <button class="c-btn" onclick="calcNum('6')">6</button>
      <button class="c-btn op" onclick="calcOp('-')">-</button>
      
      <button class="c-btn" onclick="calcNum('1')">1</button>
      <button class="c-btn" onclick="calcNum('2')">2</button>
      <button class="c-btn" onclick="calcNum('3')">3</button>
      <button class="c-btn op" onclick="calcOp('+')">+</button>
      
      <button class="c-btn zero" onclick="calcNum('0')">0</button>
      <button class="c-btn" onclick="calcNum('.')">.</button>
      <button class="c-btn op" onclick="calcEq()">=</button>
    </div>
  </div>

  <div id="tab-time" class="tab-content">
    
    <div class="tool-card">
      <h3>ストップウォッチ</h3>
      <div class="big-time" id="sw-disp">00:00.00</div>
      <div class="ctrl-row">
        <button class="t-btn start" onclick="swStart()">開始</button>
        <button class="t-btn stop" onclick="swStop()">停止</button>
        <button class="t-btn reset" onclick="swReset()">復帰</button>
      </div>
    </div>

    <div class="tool-card">
      <h3>タイマー</h3>
      <div style="margin-bottom:10px;">
        <input type="number" id="tm-m" class="time-inp" placeholder="分" value="3"> 分 
        <input type="number" id="tm-s" class="time-inp" placeholder="秒" value="00"> 秒
      </div>
      <div class="big-time" id="tm-disp">03:00</div>
      <div class="ctrl-row">
        <button class="t-btn start" onclick="tmStart()">開始</button>
        <button class="t-btn stop" onclick="tmStop()">一時停止</button>
        <button class="t-btn reset" onclick="tmReset()">リセット</button>
      </div>
    </div>

    <div class="tool-card">
      <h3>アラーム</h3>
      <input type="time" id="al-time" style="font-size:24px; padding:4px;">
      <div class="ctrl-row">
        <button class="t-btn start" id="al-set-btn" onclick="alToggle()">セット</button>
      </div>
      <div id="al-status" style="margin-top:8px; font-size:12px; color:var(--gray)">未設定</div>
    </div>
  </div>

  <div id="tab-money" class="tab-content">
    <div class="tool-card">
      <h3>通貨コンバータ</h3>
      <p style="font-size:12px; color:gray">入力すると自動計算します</p>
      
      <div class="rate-row">
        <span class="label">🇯🇵 JPY (円)</span>
        <input type="number" class="curr-inp" id="inp-jpy" oninput="conv('jpy')">
      </div>
      <div class="rate-row">
        <span class="label">🇺🇸 USD (ドル)</span>
        <input type="number" class="curr-inp" id="inp-usd" oninput="conv('usd')">
      </div>
      <div class="rate-row">
        <span class="label">🇨🇳 CNY (元)</span>
        <input type="number" class="curr-inp" id="inp-cny" oninput="conv('cny')">
      </div>

      <div class="rate-set">
        <h4>レート設定 (1ドル / 1元 あたり)</h4>
        <div>1ドル = <input type="number" id="rate-usd" value="150"> 円</div>
        <div style="margin-top:4px">1元 = <input type="number" id="rate-cny" value="21"> 円</div>
        <button onclick="conv('usd')" style="margin-top:8px;">レート更新</button>
      </div>
    </div>
  </div>
</main>

<nav>
  <button class="nav-btn active" onclick="switchTab('calc', this)">
    <span class="nav-icon">🧮</span><span>電卓</span>
  </button>
  <button class="nav-btn" onclick="switchTab('time', this)">
    <span class="nav-icon">⏱</span><span>ツール</span>
  </button>
  <button class="nav-btn" onclick="switchTab('money', this)">
    <span class="nav-icon">💴</span><span>通貨</span>
  </button>
</nav>

<script>
  // --- タブ切り替え ---
  function switchTab(id, btn) {
    document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
    document.getElementById('tab-' + id).classList.add('active');
    document.querySelectorAll('.nav-btn').forEach(el => el.classList.remove('active'));
    btn.classList.add('active');
  }

  // --- 電卓ロジック ---
  let cVal = '0', cOp = null, cPrev = null, cNew = false;
  const disp = document.getElementById('calc-disp');

  function updateDisp() { disp.textContent = cVal.substring(0, 12); }
  function calcNum(n) {
    if (cNew) { cVal = '0'; cNew = false; }
    if (cVal === '0' && n !== '.') cVal = n;
    else cVal += n;
    updateDisp();
  }
  function calcClear() { cVal = '0'; cOp = null; cPrev = null; updateDisp(); }
  function calcInvert() { cVal = String(parseFloat(cVal) * -1); updateDisp(); }
  function calcPercent() { cVal = String(parseFloat(cVal) / 100); updateDisp(); }
  function calcOp(op) {
    calcEq();
    cPrev = cVal; cOp = op; cNew = true;
  }
  function calcEq() {
    if (!cOp || cPrev === null) return;
    const a = parseFloat(cPrev), b = parseFloat(cVal);
    let res = 0;
    if (cOp === '+') res = a + b;
    if (cOp === '-') res = a - b;
    if (cOp === '*') res = a * b;
    if (cOp === '/') res = a / b;
    cVal = String(res);
    cOp = null; cNew = true;
    updateDisp();
  }

  // --- ストップウォッチ ---
  let swT = 0, swInt = null, swLast = 0;
  const swD = document.getElementById('sw-disp');
  function swFmt(ms) {
    const m = Math.floor(ms / 60000).toString().padStart(2,'0');
    const s = Math.floor((ms % 60000) / 1000).toString().padStart(2,'0');
    const cs = Math.floor((ms % 1000) / 10).toString().padStart(2,'0');
    return `${m}:${s}.${cs}`;
  }
  function swStart() {
    if (swInt) return;
    swLast = Date.now();
    swInt = setInterval(() => {
      const now = Date.now();
      swT += (now - swLast);
      swLast = now;
      swD.textContent = swFmt(swT);
    }, 10);
  }
  function swStop() { clearInterval(swInt); swInt = null; }
  function swReset() { swStop(); swT = 0; swD.textContent = '00:00.00'; }

  // --- タイマー ---
  let tmSec = 0, tmInt = null;
  const tmD = document.getElementById('tm-disp');
  function tmFmt(s) {
    const m = Math.floor(s / 60).toString().padStart(2,'0');
    const sec = (s % 60).toString().padStart(2,'0');
    return `${m}:${sec}`;
  }
  function tmStart() {
    if (tmInt) return;
    if (tmSec === 0) {
      const m = parseInt(document.getElementById('tm-m').value) || 0;
      const s = parseInt(document.getElementById('tm-s').value) || 0;
      tmSec = m * 60 + s;
    }
    if (tmSec <= 0) return;
    tmD.textContent = tmFmt(tmSec);
    tmInt = setInterval(() => {
      tmSec--;
      tmD.textContent = tmFmt(tmSec);
      if (tmSec <= 0) {
        tmStop();
        tmD.textContent = "時間です!";
        alert("タイマー終了！");
        tmSec = 0;
      }
    }, 1000);
  }
  function tmStop() { clearInterval(tmInt); tmInt = null; }
  function tmReset() { tmStop(); tmSec = 0; tmD.textContent = '00:00'; }

  // --- アラーム ---
  let alInt = null, alTarget = null;
  function alToggle() {
    const btn = document.getElementById('al-set-btn');
    const stat = document.getElementById('al-status');
    if (alInt) {
      clearInterval(alInt); alInt = null;
      btn.textContent = "セット"; btn.className = "t-btn start";
      stat.textContent = "未設定";
    } else {
      const v = document.getElementById('al-time').value;
      if (!v) return alert('時間を入力してください');
      btn.textContent = "解除"; btn.className = "t-btn stop";
      stat.textContent = v + " に鳴ります";
      alInt = setInterval(() => {
        const d = new Date();
        const cur = d.getHours().toString().padStart(2,'0') + ':' + d.getMinutes().toString().padStart(2,'0');
        if (cur === v) {
          clearInterval(alInt); alInt = null;
          btn.textContent = "セット"; btn.className = "t-btn start";
          stat.textContent = "時間です！";
          alert("アラーム: " + v + " になりました！");
        }
      }, 1000);
    }
  }

  // --- 通貨換算 ---
  function conv(src) {
    const rUSD = parseFloat(document.getElementById('rate-usd').value) || 150;
    const rCNY = parseFloat(document.getElementById('rate-cny').value) || 21;
    
    const iJPY = document.getElementById('inp-jpy');
    const iUSD = document.getElementById('inp-usd');
    const iCNY = document.getElementById('inp-cny');

    let yen = 0;
    if (src === 'jpy') yen = parseFloat(iJPY.value);
    if (src === 'usd') yen = parseFloat(iUSD.value) * rUSD;
    if (src === 'cny') yen = parseFloat(iCNY.value) * rCNY;

    if (isNaN(yen)) return;

    if (src !== 'jpy') iJPY.value = Math.round(yen);
    if (src !== 'usd') iUSD.value = (yen / rUSD).toFixed(2);
    if (src !== 'cny') iCNY.value = (yen / rCNY).toFixed(2);
  }
</script>
</body>
</html>
