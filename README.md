<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>シンプルタイマー</title>
<style>
  :root {
    --bg: #f7f7f7; --card: #fff; --text: #111; --muted: #666; --accent: #0b84ff;
  }
  [data-theme="dark"] {
    --bg: #0b0b0d; --card: #111214; --text: #e9eef6; --muted: #9aa3b2; --accent: #4aa3ff;
  }
  * { box-sizing: border-box; }
  html,body { height: 100%; }
  body {
    margin: 0; font-family: "Inter", system-ui, -apple-system, "Noto Sans JP", sans-serif;
    background: var(--bg); color: var(--text); display: flex; align-items: center; justify-content: center;
  }
  .card {
    width: 100%; max-width: 500px; background: var(--card);
    border-radius: 12px; box-shadow: 0 6px 24px rgba(2,6,23,0.08);
    padding: 28px; text-align: center;
  }
  h1 { margin: 0 0 8px; }
  .timer-display { font-size: 56px; font-weight: bold; margin: 20px 0; }
  .row { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; }
  input { width: 80px; padding: 8px; border-radius: 8px; border: 1px solid #ccc; text-align: center; }
  .controls { display: flex; justify-content: center; gap: 10px; flex-wrap: wrap; }
  .controls button {
    padding: 10px 14px; border-radius: 10px; border: none; cursor: pointer;
    background: var(--accent); color: #fff;
  }
  .controls button.secondary { background: transparent; color: var(--accent); border: 1px solid var(--accent); }
</style>
</head>
<body data-theme="light">
  <main class="card">
    <h1>シンプルタイマー（ループ版）</h1>
    <div class="row">
      <div>
        <label>時間</label>
        <input id="hours" type="number" min="0" max="99" value="0">
      </div>
      <div>
        <label>分</label>
        <input id="minutes" type="number" min="0" max="59" value="15">
      </div>
      <div>
        <label>秒</label>
        <input id="seconds" type="number" min="0" max="59" value="0">
      </div>
    </div>

    <div class="timer-display" id="display">00:15:00</div>

    <div class="controls">
      <button id="start">開始</button>
      <button id="pause" class="secondary">一時停止</button>
      <button id="reset" class="secondary">リセット</button>
      <button id="theme-toggle" class="secondary">ダークモード</button>
    </div>
  </main>

<script>
(function(){
  const body = document.body;
  const display = document.getElementById('display');
  const startBtn = document.getElementById('start');
  const pauseBtn = document.getElementById('pause');
  const resetBtn = document.getElementById('reset');
  const themeToggle = document.getElementById('theme-toggle');
  const hoursInput = document.getElementById('hours');
  const minutesInput = document.getElementById('minutes');
  const secondsInput = document.getElementById('seconds');

  let totalSeconds = 0, remaining = 0, timerId = null, running = false;

  function fmt(s){
    s = Math.max(0, Math.floor(s));
    const h = String(Math.floor(s/3600)).padStart(2,'0');
    const m = String(Math.floor((s%3600)/60)).padStart(2,'0');
    const sec = String(s%60).padStart(2,'0');
    return `${h}:${m}:${sec}`;
  }

  function updateFromInputs(){
    totalSeconds = (+hoursInput.value||0)*3600 + (+minutesInput.value||0)*60 + (+secondsInput.value||0);
    remaining = totalSeconds;
    refresh();
  }

  function refresh(){
    display.textContent = fmt(remaining);
    document.title = `残り ${fmt(remaining)}`;
  }

  hoursInput.onchange = minutesInput.onchange = secondsInput.onchange = updateFromInputs;

  function tick(){
    remaining--;
    if(remaining <= 0){
      remaining = 0;
      stop();
      playAlarm(() => {
        remaining = totalSeconds;
        start();
      });
    }
    refresh();
  }

  function start(){
    if(running) return;
    running = true;
    startBtn.textContent = '実行中';
    timerId = setInterval(tick,1000);
  }

  function stop(){
    running = false;
    startBtn.textContent = '開始';
    clearInterval(timerId);
  }

  function pause(){
    if(!running) return;
    running = false;
    startBtn.textContent = '再開';
    clearInterval(timerId);
  }

  function reset(){
    stop();
    updateFromInputs();
    refresh();
    document.title = 'シンプルタイマー';
  }

  startBtn.onclick = start;
  pauseBtn.onclick = pause;
  resetBtn.onclick = reset;

  function playAlarm(onEnd){
    let count = 0;
    function beep(){
      const ctx = new (window.AudioContext || window.webkitAudioContext)();
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine'; osc.frequency.value = 880;
      osc.connect(gain); gain.connect(ctx.destination);
      gain.gain.setValueAtTime(0.001, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.3, ctx.currentTime + 0.05);
      osc.start();
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.3);
      setTimeout(()=>{osc.stop(); ctx.close();},400);
      count++;
      if(count<3) setTimeout(beep,1000);
      else setTimeout(()=>onEnd&&onEnd(),1500);
    }
    beep();
  }

  function setTheme(t){
    body.dataset.theme = t;
    themeToggle.textContent = (t==='dark')?'ライトモード':'ダークモード';
  }
  themeToggle.onclick = ()=>{
    const t = body.dataset.theme==='light'?'dark':'light';
    setTheme(t);
  };

  updateFromInputs();
})();
</script>
</body>
</html>
