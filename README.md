<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>ゆいコードー — 動画編集</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
  :root{--bg:#0f1720;--panel:#0b1220;--muted:#9aa6b2;--accent:#18a0fb;--danger:#ff6b6b}
  *{box-sizing:border-box}
  html,body{height:100%;margin:0;font-family:'Noto Sans JP',system-ui,-apple-system,'Hiragino Kaku Gothic ProN','Meiryo',sans-serif;background:var(--bg);color:#e6eef6}
  header{height:64px;display:flex;align-items:center;padding:12px 20px;border-bottom:1px solid rgba(255,255,255,0.04);gap:12px}
  header h1{font-size:18px;margin:0;font-weight:600}
  .top-controls{margin-left:auto;display:flex;gap:8px;align-items:center}
  button{background:transparent;color:inherit;border:1px solid rgba(255,255,255,0.06);padding:8px 10px;border-radius:6px;cursor:pointer}
  .main{display:flex;height:calc(100vh - 64px)}
  .left{width:340px;padding:16px;border-right:1px solid rgba(255,255,255,0.03);overflow:auto}
  .center{flex:1;display:flex;flex-direction:column;gap:12px;padding:16px;overflow:hidden}
  .right{width:360px;padding:16px;border-left:1px solid rgba(255,255,255,0.03);overflow:auto}

  /* preview */
  .preview{background:#061021;border-radius:8px;padding:12px;display:flex;flex-direction:column;align-items:center;gap:8px}
  canvas#previewCanvas{background:black;border-radius:6px;max-width:100%;box-shadow:0 6px 20px rgba(0,0,0,0.6)}
  .transport{display:flex;gap:8px;align-items:center}
  .timeline{background:#071425;border-radius:8px;padding:10px;display:flex;flex-direction:column;gap:8px}
  .timeline-track{height:64px;background:rgba(255,255,255,0.02);border-radius:6px;position:relative;overflow:hidden}
  .clip{position:absolute;left:8px;top:8px;height:48px;background:linear-gradient(180deg,#12313f,#0d2730);border:1px solid rgba(255,255,255,0.05);border-radius:6px;padding:6px;color:#e6eef6;cursor:grab;display:flex;align-items:center;gap:8px}
  .clip .label{font-size:13px}
  .ruler{height:24px;color:var(--muted);font-size:12px;padding:6px}

  input[type=range]{width:100%}
  label{font-size:13px;color:var(--muted)}
  .filelist{display:flex;flex-direction:column;gap:6px}
  .file-entry{display:flex;gap:8px;align-items:center;border:1px solid rgba(255,255,255,0.03);padding:8px;border-radius:6px}
  .ghost{opacity:0.5}
  .small{font-size:13px;color:var(--muted)}
  .danger{background:var(--danger);color:#111}
  .ok{background:var(--accent);color:#041226}
  textarea{width:100%;min-height:80px;background:transparent;border:1px solid rgba(255,255,255,0.04);color:inherit;padding:8px;border-radius:6px}
  .prop{display:flex;flex-direction:column;gap:8px}
  .prop .row{display:flex;gap:8px;align-items:center}
  footer{position:fixed;left:16px;bottom:16px;color:var(--muted);font-size:13px}

  /* responsive small */
  @media(max-width:1000px){ .left{display:none} .right{display:none} }
</style>
</head>
<body>
<header>
  <h1>ゆいコードー — 動画エディタ</h1>
  <div class="top-controls">
    <button id="newProj">新規</button>
    <button id="saveProj">保存</button>
    <button id="loadProj">読み込み</button>
    <button id="exportBtn">書き出し</button>
  </div>
</header>

<div class="main">
  <aside class="left">
    <div style="margin-bottom:12px">
      <label class="small">メディアをアップロード（動画/音声）</label>
      <input id="fileInput" type="file" accept="video/*,audio/*" multiple>
    </div>
    <div style="margin-bottom:12px">
      <label class="small">タイムラインに追加</label>
      <div class="filelist" id="fileList"></div>
    </div>

    <div class="panel">
      <h3 style="margin:0 0 8px 0">クリップ操作</h3>
      <div class="prop">
        <div class="row"><label>開始（秒）</label><input id="clipStart" type="number" step="0.1" value="0"></div>
        <div class="row"><label>終了（秒）</label><input id="clipEnd" type="number" step="0.1" value="1"></div>
        <div class="row"><button id="applyTrim">トリム適用</button><button id="splitClip">分割</button></div>
      </div>
    </div>

    <div class="panel">
      <h3 style="margin:0 0 8px 0">オーバーレイ</h3>
      <div class="prop">
        <label>テキスト</label>
        <input id="overlayText" placeholder="テロップを入力">
        <div class="row"><label>開始（秒）</label><input id="overlayStart" type="number" step="0.1" value="0"></div>
        <div class="row"><label>終了（秒）</label><input id="overlayEnd" type="number" step="0.1" value="2"></div>
        <div class="row"><button id="addOverlay">追加</button></div>
      </div>
    </div>

  </aside>

  <section class="center">
    <div class="preview">
      <canvas id="previewCanvas" width="960" height="540"></canvas>
      <div class="transport">
        <button id="playBtn">再生</button>
        <button id="pauseBtn">一時停止</button>
        <label class="small">再生率 <input id="playbackRate" type="range" min="0.25" max="2" step="0.25" value="1"></label>
        <label class="small">ボリューム <input id="masterVolume" type="range" min="0" max="2" step="0.01" value="1"></label>
        <div class="small" id="timecode">0.00 / 0.00</div>
      </div>
    </div>

    <div class="timeline">
      <div class="ruler" id="ruler">Timeline (秒)</div>
      <div class="timeline-track" id="videoTrack"></div>
      <div class="small">ドラッグで配置、クリックで選択。クリップは連続シーケンスが基本です。</div>
    </div>
  </section>

  <aside class="right">
    <div class="panel">
      <h3 style="margin:0">プロジェクト</h3>
      <div class="small">合計長さ: <span id="totalDuration">0.00</span> 秒</div>
      <div style="height:8px"></div>
      <div class="small">選択クリップ: <span id="selectedClipLabel">なし</span></div>
      <div style="height:8px"></div>
      <div class="small">オーバーレイ数: <span id="overlayCount">0</span></div>
      <div style="height:8px"></div>
      <div><button id="undoBtn">元に戻す</button><button id="redoBtn">やり直す</button></div>
    </div>

    <div class="panel">
      <h3 style="margin:0">出力設定</h3>
      <div class="small">解像度</div>
      <select id="resolution"><option value="16:9">16:9 (1920x1080)</option><option value="9:16">9:16 (1080x1920)</option><option value="1:1">1:1 (1080x1080)</option></select>
      <div style="height:8px"></div>
      <button id="captureThumb">サムネイル生成</button>
    </div>
  </aside>
</div>
<footer>注意: このアプリはブラウザで動作します。長い動画や高解像度の書き出しは多くのメモリと時間を要します。</footer>

<script>
// ゆいコードー — シンプル動画エディタ（完全クライアント）
// 実装の方針：タイムラインはシーケンシャル（非重複）クリップを想定。
// プレビューは canvas に各クリップの video 要素を時刻に合わせて描画。
// 書き出しは canvas.captureStream + AudioContext を使って MediaRecorder で録る方式。

const fileInput = document.getElementById('fileInput');
const fileList = document.getElementById('fileList');
const videoTrack = document.getElementById('videoTrack');
const previewCanvas = document.getElementById('previewCanvas');
const ctx = previewCanvas.getContext('2d');
const playBtn = document.getElementById('playBtn');
const pauseBtn = document.getElementById('pauseBtn');
const timecode = document.getElementById('timecode');
const playbackRate = document.getElementById('playbackRate');
const masterVolume = document.getElementById('masterVolume');
const totalDurationEl = document.getElementById('totalDuration');
const selectedClipLabel = document.getElementById('selectedClipLabel');
const overlayCount = document.getElementById('overlayCount');
const exportBtn = document.getElementById('exportBtn');

let mediaPool = []; // {id, file, type, url, kind: 'video'|'audio'}
let timeline = []; // sequence of clips: {id, mediaId, start, end, pos} pos = seconds from start
let overlays = []; // {id, text, start, end, x,y,font,size,color}
let selectedClip = null;
let isPlaying = false; let playStartWall = 0; let playOffset = 0; // offset in timeline seconds
let rafId = null;
let history = []; let histIndex = -1;

// Audio context for mixing
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
const masterGain = audioCtx.createGain(); masterGain.gain.value = 1; masterGain.connect(audioCtx.destination);
masterVolume.addEventListener('input', ()=>{ masterGain.gain.value = parseFloat(masterVolume.value); });

function pushHistory(){ const snap = JSON.stringify({mediaPool,timeline,overlays}); if(histIndex < history.length-1) history = history.slice(0,histIndex+1); history.push(snap); if(history.length>200) history.shift(); histIndex = history.length-1; }
function undo(){ if(histIndex>0){ histIndex--; const obj = JSON.parse(history[histIndex]); mediaPool = obj.mediaPool; timeline = obj.timeline; overlays = obj.overlays; rebuildUI(); }}
function redo(){ if(histIndex < history.length-1){ histIndex++; const obj = JSON.parse(history[histIndex]); mediaPool = obj.mediaPool; timeline = obj.timeline; overlays = obj.overlays; rebuildUI(); }}

fileInput.addEventListener('change', e=>{
  const files = Array.from(e.target.files);
  files.forEach(f=>{
    const id = Date.now().toString(36)+Math.random().toString(36).slice(2,6);
    const url = URL.createObjectURL(f);
    const kind = f.type.startsWith('video')? 'video' : (f.type.startsWith('audio')? 'audio':'other');
    mediaPool.push({id,file:f,url,kind});
  });
  rebuildMediaList(); pushHistory();
});

function rebuildMediaList(){ fileList.innerHTML=''; mediaPool.forEach(m=>{
  const div = document.createElement('div'); div.className='file-entry';
  const lbl = document.createElement('div'); lbl.textContent = m.file.name; lbl.style.flex='1';
  const btn = document.createElement('button'); btn.textContent='追加'; btn.addEventListener('click', ()=> addToTimeline(m.id));
  const del = document.createElement('button'); del.textContent='削除'; del.className='danger'; del.addEventListener('click', ()=>{ mediaPool = mediaPool.filter(x=>x.id!==m.id); rebuildMediaList(); pushHistory(); });
  div.appendChild(lbl); div.appendChild(btn); div.appendChild(del); fileList.appendChild(div);
}); }

function addToTimeline(mediaId){ const m = mediaPool.find(x=>x.id===mediaId); if(!m) return; // default: place at end
  const pos = timeline.length? Math.max(...timeline.map(c=>c.pos + (c.end-c.start))) : 0;
  // need duration: for video, load metadata
  const temp = document.createElement(m.kind==='video'? 'video':'audio'); temp.preload='metadata'; temp.src = m.url; temp.onloadedmetadata = ()=>{
    const dur = temp.duration || 1;
    const clip = {id:Date.now().toString(36)+Math.random().toString(36).slice(2,6), mediaId, start:0, end:Math.min(dur, Math.max(1,dur)), pos};
    timeline.push(clip); pushHistory(); rebuildUI();
  };
}

function rebuildUI(){ // timeline track
  videoTrack.innerHTML=''; let cursor=0; timeline.sort((a,b)=>a.pos-b.pos);
  timeline.forEach((c,i)=>{
    const m = mediaPool.find(x=>x.id===c.mediaId);
    const w = Math.max(30, (c.end-c.start)*40); // px per second approx
    const div = document.createElement('div'); div.className='clip'; div.style.left = (c.pos*40 + 8) + 'px'; div.style.width = w+'px'; div.dataset.id = c.id;
    div.innerHTML = `<div class="label">${m.file.name.split('.')[0]} (${(c.end-c.start).toFixed(2)}s)</div>`;
    div.addEventListener('mousedown', clipMouseDown);
    div.addEventListener('click', (e)=>{ e.stopPropagation(); selectClip(c.id); });
    videoTrack.appendChild(div);
    cursor = Math.max(cursor, c.pos + (c.end-c.start));
  });
  totalDurationEl.textContent = cursor.toFixed(2);
  overlayCount.textContent = overlays.length;
  updateSelectedLabel();
}

function selectClip(id){ selectedClip = timeline.find(x=>x.id===id); updateSelectedLabel();
  if(selectedClip){ document.getElementById('clipStart').value = selectedClip.start; document.getElementById('clipEnd').value = selectedClip.end; }
}
function updateSelectedLabel(){ selectedClipLabel.textContent = selectedClip? (mediaPool.find(m=>m.id===selectedClip.mediaId).file.name + ' ['+selectedClip.start.toFixed(2)+'-'+selectedClip.end.toFixed(2)+']') : 'なし'; }

// clip drag in timeline
let dragState = null;
function clipMouseDown(e){ const el = e.currentTarget; const id = el.dataset.id; dragState = {id, startX: e.clientX, origLeft: parseFloat(el.style.left)}; function onMove(ev){ if(!dragState) return; const dx = ev.clientX - dragState.startX; el.style.left = Math.max(8, dragState.origLeft + dx) + 'px'; }
  function onUp(ev){ if(!dragState) return; const dx = ev.clientX - dragState.startX; const seconds = Math.round((dragState.origLeft + dx - 8)/40 * 100)/100; const clip = timeline.find(c=>c.id===dragState.id); if(clip){ clip.pos = Math.max(0, seconds); pushHistory(); rebuildUI(); }
    document.removeEventListener('mousemove', onMove); document.removeEventListener('mouseup', onUp); dragState = null; }
  document.addEventListener('mousemove', onMove); document.addEventListener('mouseup', onUp);
}

// trim / split
document.getElementById('applyTrim').addEventListener('click', ()=>{
  if(!selectedClip) return alert('クリップを選択してください'); const s = parseFloat(document.getElementById('clipStart').value)||0; const e = parseFloat(document.getElementById('clipEnd').value)||0; if(e<=s) return alert('終了は開始より大きくしてください'); selectedClip.start = s; selectedClip.end = e; pushHistory(); rebuildUI();
});

document.getElementById('splitClip').addEventListener('click', ()=>{
  if(!selectedClip) return alert('クリップを選択してください'); const at = parseFloat(document.getElementById('clipStart').value)||0; if(at<=selectedClip.start || at>=selectedClip.end) return alert('分割位置が範囲外です');
  const idx = timeline.indexOf(selectedClip);
  const a = Object.assign({}, selectedClip, {end: at});
  const b = Object.assign({}, selectedClip, {start: at, id:Date.now().toString(36)+Math.random().toString(36).slice(2,6)});
  timeline.splice(idx,1,a,b); pushHistory(); rebuildUI();
});

// overlays
document.getElementById('addOverlay').addEventListener('click', ()=>{
  const text = document.getElementById('overlayText').value.trim(); if(!text) return alert('テキストを入力してください'); const s = parseFloat(document.getElementById('overlayStart').value)||0; const e = parseFloat(document.getElementById('overlayEnd').value)||0; if(e<=s) return alert('終了は開始より大きくしてください'); const ov = {id:Date.now().toString(36), text, start:s, end:e, x:50, y:50, font:'24px sans-serif', color:'#ffffff'}; overlays.push(ov); pushHistory(); rebuildUI(); });

// preview engine
function getTotalDuration(){ if(!timeline.length) return 0; return Math.max(...timeline.map(c=>c.pos + (c.end-c.start))); }

function drawFrame(currentTime){ // clear
  ctx.fillStyle = '#000'; ctx.fillRect(0,0,previewCanvas.width, previewCanvas.height);
  // find which clip is playing at currentTime
  for(const clip of timeline){ const clipStart = clip.pos; const clipEnd = clip.pos + (clip.end-clip.start); if(currentTime >= clipStart && currentTime < clipEnd){ const media = mediaPool.find(m=>m.id===clip.mediaId); if(media && media.kind==='video'){
        if(!media._video){ media._video = document.createElement('video'); media._video.muted = true; media._video.src = media.url; media._video.crossOrigin='anonymous'; }
        const v = media._video; const tInClip = clip.start + (currentTime - clipStart);
        if(Math.abs(v.currentTime - tInClip) > 0.2) v.currentTime = tInClip; // seek if desynced
        if(v.readyState >= 2){ try{ ctx.drawImage(v,0,0,previewCanvas.width, previewCanvas.height); }catch(err){} }
      }
  }
  // overlays
  ctx.save(); overlays.forEach(ov=>{ if(currentTime>=ov.start && currentTime<=ov.end){ ctx.font = ov.font; ctx.fillStyle = ov.color; ctx.fillText(ov.text, ov.x, ov.y); }}); ctx.restore();
}

function play(){ if(isPlaying) return; if(audioCtx.state==='suspended') audioCtx.resume(); const total = getTotalDuration(); if(total<=0) return alert('タイムラインが空です'); isPlaying = true; playStartWall = performance.now(); playOffset = playOffset || 0;
  function loop(){ const elapsed = (performance.now() - playStartWall)/1000 * parseFloat(playbackRate.value) + playOffset; if(elapsed >= total){ stop(); return; } drawFrame(elapsed); timecode.textContent = elapsed.toFixed(2) + ' / ' + total.toFixed(2); rafId = requestAnimationFrame(loop); }
  rafId = requestAnimationFrame(loop);
}
function stop(){ if(!isPlaying) return; isPlaying=false; if(rafId) cancelAnimationFrame(rafId); rafId=null; playOffset=0; }
function pause(){ if(!isPlaying) return; isPlaying=false; if(rafId) cancelAnimationFrame(rafId); rafId=null; // compute offset
  // approximate offset via timecode
  const parts = timecode.textContent.split(' / ')[0]; playOffset = parseFloat(parts) || 0;
}
playBtn.addEventListener('click', ()=> play()); pauseBtn.addEventListener('click', ()=> pause());

// export: capture canvas + mixed audio
exportBtn.addEventListener('click', async ()=>{
  const total = getTotalDuration(); if(total<=0) return alert('タイムラインが空です');
  // prepare audio: create sources for each clip sequentially played into AudioContext destination
  const dest = audioCtx.createMediaStreamDestination(); const combinedStream = previewCanvas.captureStream(30);
  // connect masterGain to dest as well to capture volume-controlled audio
  masterGain.disconnect(); masterGain.connect(audioCtx.destination); masterGain.connect(dest);
  // for each clip, create an HTMLMediaElement and schedule play via setTimeout when recording
  const mediaElements = [];
  for(const clip of timeline){ const media = mediaPool.find(m=>m.id===clip.mediaId); if(media.kind==='video' || media.kind==='audio'){
      const el = document.createElement(media.kind==='video'?'video':'audio'); el.src = media.url; el.crossOrigin='anonymous'; el.preload='auto'; el.muted = false; el.volume = 1.0; mediaElements.push({el, clip});
    }}
  // start recording
  const outStream = new MediaStream(); combinedStream.getVideoTracks().forEach(t=> outStream.addTrack(t)); dest.stream.getAudioTracks().forEach(t=> outStream.addTrack(t));
  const recorder = new MediaRecorder(outStream, {mimeType:'video/webm;codecs=vp9,opus'});
  const chunks = [];
  recorder.ondataavailable = e=>{ if(e.data.size) chunks.push(e.data); };
  recorder.onstop = ()=>{
    const blob = new Blob(chunks, {type:'video/webm'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = (document.getElementById('projName').value||'yui-kodo') + '.webm'; a.click(); // reconnect masterGain
    masterGain.disconnect(); masterGain.connect(audioCtx.destination);
  };

  recorder.start();
  // play timeline sequentially: play mediaElements at correct offsets
  const startTime = audioCtx.currentTime; // now
  for(const me of mediaElements){ const {el, clip} = me; // connect element to audioCtx
    const src = audioCtx.createMediaElementSource(el); src.connect(masterGain);
    // schedule play using setTimeout to align with real time
    const wait = (clip.pos - 0) * 1000; // since we start immediately at timeline 0
    setTimeout(()=>{ el.currentTime = clip.start; el.playbackRate = parseFloat(playbackRate.value); el.play(); }, Math.max(0, Math.round(wait)));
    // stop after duration
    setTimeout(()=>{ try{ el.pause(); el.currentTime = 0; }catch(e){} }, Math.max(0, Math.round(wait + (clip.end-clip.start)*1000)));
  }

  // drive canvas drawing and stop recorder after total
  let recStart = performance.now(); function drive(){ const elapsed = ((performance.now() - recStart)/1000) * parseFloat(playbackRate.value); drawFrame(elapsed); if(elapsed < total) requestAnimationFrame(drive); }
  drive();
  setTimeout(()=>{ recorder.stop(); }, (total / parseFloat(playbackRate.value))*1000 + 500);
});

// simple project save/load
document.getElementById('saveProj').addEventListener('click', ()=>{
  const data = {mediaPool: mediaPool.map(m=>({id:m.id,name:m.file.name,type:m.file.type})), timeline, overlays, meta:{name:document.getElementById('projName').value||'yui-kodo'}};
  const blob = new Blob([JSON.stringify(data)], {type:'application/json'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = (document.getElementById('projName').value||'yui-kodo') + '.ykproj'; a.click();
});

document.getElementById('loadProj').addEventListener('click', ()=>{
  const input = document.createElement('input'); input.type='file'; input.accept='.ykproj,.json'; input.onchange = e=>{ const f = e.target.files[0]; if(!f) return; const r = new FileReader(); r.onload = ()=>{ try{ const obj = JSON.parse(r.result); // NOTE: media files must be re-imported manually due to browser security. We'll load metadata only.
      timeline = obj.timeline || []; overlays = obj.overlays || []; document.getElementById('projName').value = obj.meta?.name||''; pushHistory(); rebuildUI(); alert('プロジェクト読み込み: メディアは再アップロードが必要です'); }catch(err){ alert('読み込みエラー'); } }; r.readAsText(f); }; input.click();
});

// new project
document.getElementById('newProj').addEventListener('click', ()=>{ if(!confirm('新規プロジェクトを作成しますか？現在の作業は失われます。')) return; mediaPool=[]; timeline=[]; overlays=[]; selectedClip=null; rebuildUI(); pushHistory(); });

// basic UI helpers
function rebuildUI(){ rebuildMediaList(); rebuildUITracks(); }
function rebuildUITracks(){ rebuildMediaList(); rebuildUITimeline(); }
function rebuildUITimeline(){ // re-create timeline track
  videoTrack.innerHTML=''; rebuildUI(); }

// helper for initial state
pushHistory();

</script>
</body>
</html>
