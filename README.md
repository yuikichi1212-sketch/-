<script>
(function(){
  const body = document.body;
  const modeCountdown = document.getElementById('mode-countdown');
  const modeStopwatch = document.getElementById('mode-stopwatch');
  const countdownArea = document.getElementById('countdown-area');
  const display = document.getElementById('display');
  const startBtn = document.getElementById('start');
  const pauseBtn = document.getElementById('pause');
  const resetBtn = document.getElementById('reset');
  const hoursInput = document.getElementById('hours');
  const minutesInput = document.getElementById('minutes');
  const secondsInput = document.getElementById('seconds');
  const themeToggle = document.getElementById('theme-toggle');

  let totalSeconds = 0;
  let remaining = 0;
  let timerId = null;
  let running = false;

  function fmtSec(s){
    s = Math.max(0, Math.floor(s));
    const h = Math.floor(s/3600);
    const m = Math.floor((s%3600)/60);
    const sec = s%60;
    return [h,m,sec].map(n=>String(n).padStart(2,'0')).join(':');
  }

  function refreshDisplay(){
    display.textContent = fmtSec(remaining);
    document.title = "残り " + fmtSec(remaining);
  }

  function updateTotalFromInputs(){
    totalSeconds = (+hoursInput.value||0)*3600 + (+minutesInput.value||0)*60 + (+secondsInput.value||0);
    remaining = totalSeconds;
    refreshDisplay();
  }

  hoursInput.addEventListener('change',updateTotalFromInputs);
  minutesInput.addEventListener('change',updateTotalFromInputs);
  secondsInput.addEventListener('change',updateTotalFromInputs);

  function tick(){
    remaining -= 1;
    if(remaining <= 0){
      remaining = 0;
      stopTimer();
      playAlarm(() => {
        remaining = totalSeconds;   // 🔁 自動ループ再開
        startTimer();
      });
    }
    refreshDisplay();
  }

  function startTimer(){
    if(running) return;
    running = true;
    startBtn.textContent = '実行中';
    startBtn.disabled = true;
    timerId = setInterval(tick,1000);
  }

  function stopTimer(){
    running = false;
    startBtn.textContent = '開始';
    startBtn.disabled = false;
    clearInterval(timerId);
    timerId = null;
  }

  function pauseTimer(){
    if(!running) return;
    running = false;
    startBtn.textContent = '再開';
    startBtn.disabled = false;
    if(timerId) clearInterval(timerId);
    timerId = null;
  }

  function reset(){
    stopTimer();
    updateTotalFromInputs();
    refreshDisplay();
    document.title = "シンプルタイマー";
  }

  startBtn.addEventListener('click',startTimer);
  pauseBtn.addEventListener('click',pauseTimer);
  resetBtn.addEventListener('click',reset);

  //  3回ベル音 + コールバックで次のループへ
  function playAlarm(onEnd){
    let count = 0;
    function beep(){
      const ctx = new (window.AudioContext || window.webkitAudioContext)();
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = "sine";
      osc.frequency.value = 880;
      osc.connect(gain);
      gain.connect(ctx.destination);
      gain.gain.setValueAtTime(0.001, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.3, ctx.currentTime + 0.05);
      osc.start();
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.3);
      setTimeout(() => { osc.stop(); ctx.close(); }, 400);
      count++;
      if(count < 3) setTimeout(beep, 1000);
      else setTimeout(() => { if(onEnd) onEnd(); }, 1500);
    }
    beep();
  }

  //  モード切り替え（そのまま）
  function setTheme(t){
    body.setAttribute('data-theme', t);
    themeToggle.textContent = (t==='dark')? 'ライトモード' : 'ダークモード';
  }
  themeToggle.addEventListener('click',()=>{
    const current = body.getAttribute('data-theme')||'light';
    setTheme(current==='light'?'dark':'light');
  });
  setTheme('light');

  updateTotalFromInputs();
})();
</script>
