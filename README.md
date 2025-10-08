<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ゆいきちコーダー）</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&display=swap" rel="stylesheet">
  <style>
    :root{
      --bgA: #f6f7fb;
      --glass: rgba(255,255,255,0.55);
      --muted:#6b7280;
      --accent:#0f172a;
      --glass-border: rgba(255,255,255,0.6);
      --shadow: 0 8px 30px rgba(12,18,31,0.07);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; font-family: 'Noto Sans JP', system-ui, -apple-system, 'Hiragino Kaku Gothic ProN', 'Meiryo', sans-serif; background: linear-gradient(180deg,#eef2ff 0%, #f8fafc 100%); color:var(--accent); display:flex; flex-direction:column; align-items:center; padding:18px;
    }
    header{width:100%; max-width:1200px; display:flex; align-items:center; justify-content:space-between; gap:12px; margin-bottom:12px}
    header h1{margin:0; font-size:18px}
    header p{margin:0; color:var(--muted); font-size:13px}

    .app{width:100%; max-width:1200px; display:grid; grid-template-columns: 1fr 360px; gap:18px}

    /* Stage */
    .stage-wrap{background:linear-gradient(180deg,#fff,#fbfdff); padding:18px; border-radius:14px; box-shadow:var(--shadow); display:flex; flex-direction:column; align-items:center}
    .stage-inner{background:conic-gradient(from 180deg at 50% 50%, rgba(0,0,0,0.02), transparent); padding:12px; border-radius:10px}
    canvas{background:transparent; border-radius:8px; display:block; box-shadow:0 6px 20px rgba(11,20,40,0.04)}
    .stage-toolbar{display:flex; gap:8px; margin-top:12px; align-items:center}

    /* Controls */
    .panel{background:var(--glass); border-radius:12px; padding:14px; border:1px solid var(--glass-border); backdrop-filter: blur(8px) saturate(120%); box-shadow:var(--shadow);}
    .controls{display:flex; flex-direction:column; gap:12px}
    label{font-size:13px; color:var(--muted); display:block; margin-bottom:6px}
    input[type=text], input[type=number], select, textarea{width:100%; padding:8px; border-radius:8px; border:1px solid rgba(0,0,0,0.06); font-size:14px}
    input[type=color]{padding:4px; height:36px}

    /* buttons: transparent, subtle */
    .btn{background:transparent; border:1px solid rgba(15,23,42,0.08); padding:8px 10px; border-radius:10px; cursor:pointer; color:var(--accent); font-weight:600}
    .btn:hover{transform:translateY(-2px); box-shadow:0 6px 18px rgba(12,18,31,0.06)}
    .btn-primary{background:linear-gradient(180deg, rgba(255,255,255,0.35), rgba(255,255,255,0.15)); border:1px solid rgba(255,255,255,0.6)}
    .small{padding:6px 8px; font-size:13px}

    .row{display:flex; gap:8px; align-items:center}
    .muted{color:var(--muted); font-size:13px}
    .text-item{padding:8px;border-radius:8px;border:1px dashed rgba(0,0,0,0.04);margin-bottom:8px; display:flex; justify-content:space-between; align-items:center}

    .footer{font-size:13px; color:var(--muted); text-align:center; margin-top:10px}

    @media(max-width:1100px){.app{grid-template-columns:1fr;}.stage-wrap{order:2}}
  </style>
</head>
<body>
  <header>
    <h1>ゆいきちコーダー</h1>
    <p>透明感のあるUI・回転/トリミング/縁取り/シャドウ/Undo・Redo など搭載</p>
  </header>

  <div class="app">
    <section class="stage-wrap panel">
      <div class="stage-inner">
        <canvas id="canvas" width="1280" height="720"></canvas>
      </div>

      <div class="stage-toolbar">
        <input id="file" type="file" accept="image/*" class="btn">
        <button id="fit" class="btn">フィット</button>
        <button id="rotateLeft" class="btn">⟲ 回転</button>
        <button id="rotateRight" class="btn">⟳ 回転</button>
        <button id="flipX" class="btn">水平反転</button>
        <button id="flipY" class="btn">垂直反転</button>
        <button id="cropMode" class="btn">トリミング</button>
        <button id="undo" class="btn">Undo</button>
        <button id="redo" class="btn">Redo</button>
        <button id="downloadPng" class="btn btn-primary">PNGで保存</button>
      </div>

      <div style="display:flex; gap:8px; margin-top:8px; align-items:center;">
        <div class="muted">キャンバス: <strong>1280×720</strong></div>
        <div class="muted">背景: <label style="display:inline-block;margin-left:6px"><input id="bgTransparent" type="checkbox"> 透過</label></div>
      </div>
    </section>

    <aside class="panel controls">
      <div>
        <label>画像拡大/縮小</label>
        <input id="scale" type="range" min="0.1" max="3" step="0.01" value="1">
      </div>
      <div>
        <label>位置（X / Y）</label>
        <div class="row">
          <input id="offsetX" type="range" min="-2000" max="2000" step="1" value="0">
          <input id="offsetY" type="range" min="-2000" max="2000" step="1" value="0">
        </div>
      </div>

      <hr>
      <h3 style="margin:0 0 8px 0">テキスト（レイヤー対応）</h3>
      <div class="row">
        <input id="textInput" type="text" placeholder="テキストを入力">
        <button id="addText" class="btn small">追加</button>
      </div>
      <div style="display:flex; gap:8px; margin-top:8px">
        <select id="fontSelect">
          <option value="Noto Sans JP">Noto Sans JP</option>
          <option value="sans-serif">sans-serif</option>
          <option value="serif">serif</option>
          <option value="Brush Script MT, cursive">手書き風</option>
        </select>
        <input id="fontSize" type="number" value="64" min="10" max="400" style="width:96px">
        <input id="fontColor" type="color" value="#ffffff">
      </div>
      <div class="row" style="margin-top:8px">
        <label><input id="bold" type="checkbox"> 太字</label>
        <label><input id="shadow" type="checkbox" checked> シャドウ</label>
        <label><input id="stroke" type="checkbox"> 縁取り</label>
      </div>

      <div style="margin-top:8px">
        <label>レイヤー</label>
        <div id="textsList"></div>
        <div class="row" style="margin-top:8px">
          <button id="bringFront" class="btn small">前面へ</button>
          <button id="sendBack" class="btn small">背面へ</button>
          <button id="deleteText" class="btn small">削除</button>
        </div>
      </div>

      <hr>
      <h3 style="margin:0 0 8px 0">背景</h3>
      <div class="row">
        <label>色: <input id="bgColor" type="color" value="#000000"></label>
        <button id="clearBg" class="btn small">背景クリア</button>
      </div>

      <hr>
      <h3 style="margin:0 0 8px 0">トリミング / エクスポート</h3>
      <div class="row">
        <button id="applyCrop" class="btn small">トリミング適用</button>
        <button id="cancelCrop" class="btn small">キャンセル</button>
      </div>

      <div class="footer">操作: テキストをクリックで選択 → ドラッグで移動。ダブルクリックで編集。</div>
    </aside>
  </div>

  <script>
    // Elements
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const fileInput = document.getElementById('file');
    const scaleRange = document.getElementById('scale');
    const offsetX = document.getElementById('offsetX');
    const offsetY = document.getElementById('offsetY');
    const fitBtn = document.getElementById('fit');
    const rotateLeft = document.getElementById('rotateLeft');
    const rotateRight = document.getElementById('rotateRight');
    const flipX = document.getElementById('flipX');
    const flipY = document.getElementById('flipY');
    const cropModeBtn = document.getElementById('cropMode');
    const downloadPng = document.getElementById('downloadPng');
    const addTextBtn = document.getElementById('addText');
    const textInput = document.getElementById('textInput');
    const fontSelect = document.getElementById('fontSelect');
    const fontSizeInput = document.getElementById('fontSize');
    const fontColorInput = document.getElementById('fontColor');
    const boldCheckbox = document.getElementById('bold');
    const shadowCheckbox = document.getElementById('shadow');
    const strokeCheckbox = document.getElementById('stroke');
    const textsList = document.getElementById('textsList');
    const bringFrontBtn = document.getElementById('bringFront');
    const sendBackBtn = document.getElementById('sendBack');
    const deleteTextBtn = document.getElementById('deleteText');
    const bgColorInput = document.getElementById('bgColor');
    const bgTransparent = document.getElementById('bgTransparent');
    const applyCrop = document.getElementById('applyCrop');
    const cancelCrop = document.getElementById('cancelCrop');
    const undoBtn = document.getElementById('undo');
    const redoBtn = document.getElementById('redo');

    // State
    let img = null;
    let imgState = {scale:1, x:0, y:0, rotation:0, flipX:false, flipY:false};
    let texts = []; // {id, text, x, y, font, size, color, bold, shadow, stroke}
    let selectedId = null;
    let cropMode = false;
    let cropRect = null; // {x,y,w,h}

    // Undo/Redo
    const history = []; let histIndex = -1; const MAX_HISTORY = 40;
    function pushHistory(){
      const snap = JSON.stringify({imgState, texts, bgColor: bgColorInput.value, bgTransparent: bgTransparent.checked});
      // truncate
      if(histIndex < history.length-1) history.splice(histIndex+1);
      history.push(snap); if(history.length>MAX_HISTORY) history.shift(); histIndex = history.length-1; updateUndoRedo();
    }
    function restoreSnapshot(sn){
      const obj = JSON.parse(sn);
      imgState = obj.imgState; texts = obj.texts; bgColorInput.value = obj.bgColor; bgTransparent.checked = obj.bgTransparent; selectedId = null; draw(); updateTextsList(); updateUndoRedo();
    }
    function updateUndoRedo(){ undoBtn.disabled = histIndex<=0; redoBtn.disabled = histIndex>=history.length-1; }

    undoBtn.addEventListener('click', ()=>{ if(histIndex>0) { histIndex--; restoreSnapshot(history[histIndex]); }});
    redoBtn.addEventListener('click', ()=>{ if(histIndex<history.length-1){ histIndex++; restoreSnapshot(history[histIndex]); }});

    // Draw
    function draw(){
      // clear
      ctx.clearRect(0,0,canvas.width,canvas.height);
      // bg
      if(!bgTransparent.checked){ ctx.fillStyle = bgColorInput.value; ctx.fillRect(0,0,canvas.width,canvas.height); }

      // image transform and draw
      if(img){
        ctx.save();
        ctx.translate(canvas.width/2 + imgState.x + parseFloat(offsetX.value||0), canvas.height/2 + imgState.y + parseFloat(offsetY.value||0));
        ctx.rotate(imgState.rotation * Math.PI/180);
        const sx = imgState.flipX ? -1:1; const sy = imgState.flipY? -1:1;
        ctx.scale(sx, sy);
        const iw = img.width * imgState.scale; const ih = img.height * imgState.scale;
        ctx.drawImage(img, -iw/2, -ih/2, iw, ih);
        ctx.restore();
      }

      // crop overlay if mode on
      if(cropMode && cropRect){ ctx.save(); ctx.strokeStyle='rgba(255,255,255,0.9)'; ctx.lineWidth=2; ctx.setLineDash([6,6]); ctx.strokeRect(cropRect.x, cropRect.y, cropRect.w, cropRect.h); ctx.restore(); }

      // texts draw in order
      for(const t of texts){
        ctx.save();
        ctx.font = `${t.bold?900:400} ${t.size}px ${t.font}`;
        ctx.textBaseline='top'; ctx.textAlign='left';
        if(t.shadow){ ctx.fillStyle = 'rgba(0,0,0,0.45)'; ctx.fillText(t.text, t.x+4, t.y+4); }
        if(t.stroke){ ctx.lineWidth = Math.max(2, Math.floor(t.size/12)); ctx.strokeStyle = 'rgba(0,0,0,0.6)'; ctx.strokeText(t.text, t.x, t.y); }
        ctx.fillStyle = t.color; ctx.fillText(t.text, t.x, t.y);
        if(t.id === selectedId){ const m = ctx.measureText(t.text); const w=m.width; const h=t.size; ctx.save(); ctx.strokeStyle='rgba(255,255,255,0.9)'; ctx.lineWidth=2; ctx.strokeRect(t.x-6, t.y-6, w+12, h+12); ctx.restore(); }
        ctx.restore();
      }
    }

    // initial draw
    draw(); pushHistory();

    // File load
    fileInput.addEventListener('change', e=>{
      const f = e.target.files[0]; if(!f) return;
      const reader = new FileReader(); reader.onload = ()=>{
        const tmp = new Image(); tmp.onload = ()=>{ img = tmp; // reset transforms
          imgState = {scale: Math.min(canvas.width/img.width, canvas.height/img.height), x:0, y:0, rotation:0, flipX:false, flipY:false}; scaleRange.value = imgState.scale; offsetX.value=0; offsetY.value=0; pushHistory(); draw(); }
        tmp.src = reader.result;
      }
      reader.readAsDataURL(f);
    });

    // Fit
    fitBtn.addEventListener('click', ()=>{ if(!img) return; imgState.scale = Math.min(canvas.width/img.width, canvas.height/img.height); scaleRange.value = imgState.scale; offsetX.value=0; offsetY.value=0; pushHistory(); draw(); });

    // Scale/offset
    scaleRange.addEventListener('input', ()=>{ imgState.scale = parseFloat(scaleRange.value); draw(); });
    offsetX.addEventListener('input', ()=>{ draw(); });
    offsetY.addEventListener('input', ()=>{ draw(); });

    // Rotation / flip
    rotateLeft.addEventListener('click', ()=>{ imgState.rotation = (imgState.rotation - 90) % 360; pushHistory(); draw(); });
    rotateRight.addEventListener('click', ()=>{ imgState.rotation = (imgState.rotation + 90) % 360; pushHistory(); draw(); });
    flipX.addEventListener('click', ()=>{ imgState.flipX = !imgState.flipX; pushHistory(); draw(); });
    flipY.addEventListener('click', ()=>{ imgState.flipY = !imgState.flipY; pushHistory(); draw(); });

    // Crop mode
    cropModeBtn.addEventListener('click', ()=>{ cropMode = !cropMode; cropRect = null; cropModeBtn.textContent = cropMode ? 'トリミング中（ドラッグ）':'トリミング'; draw(); });
    let draggingCrop = false; let cropStart = null;
    canvas.addEventListener('mousedown', e=>{
      const pos = getMousePos(canvas,e);
      if(cropMode){ draggingCrop = true; cropStart = pos; cropRect = {x:pos.x, y:pos.y, w:0, h:0}; return; }
      // select text
      for(let i=texts.length-1;i>=0;i--){ const t=texts[i]; const m = ctx.measureText(t.text); const w=m.width; const h=t.size; if(pos.x>=t.x-6 && pos.x<=t.x+w+6 && pos.y>=t.y-6 && pos.y<=t.y+h+6){ selectedId = t.id; draggingText=true; dragOffset.x = pos.x - t.x; dragOffset.y = pos.y - t.y; updateTextsList(); draw(); return; } }
      selectedId = null; updateTextsList(); draw();
    });
    window.addEventListener('mousemove', e=>{
      const pos = getMousePos(canvas,e);
      if(draggingCrop && cropStart){ cropRect.x = Math.min(cropStart.x, pos.x); cropRect.y = Math.min(cropStart.y, pos.y); cropRect.w = Math.abs(pos.x - cropStart.x); cropRect.h = Math.abs(pos.y - cropStart.y); draw(); }
      if(draggingText){ const t = texts.find(x=>x.id===selectedId); if(t){ t.x = pos.x - dragOffset.x; t.y = pos.y - dragOffset.y; draw(); }}
    });
    window.addEventListener('mouseup', ()=>{ if(draggingCrop){ draggingCrop=false; } draggingText=false; });

    applyCrop.addEventListener('click', ()=>{ if(!cropRect || !img) return; // create temp canvas and crop
      const tmp = document.createElement('canvas'); tmp.width = cropRect.w; tmp.height = cropRect.h; const tctx = tmp.getContext('2d');
      // draw current canvas crop area
      tctx.drawImage(canvas, cropRect.x, cropRect.y, cropRect.w, cropRect.h, 0,0,cropRect.w,cropRect.h);
      const newImg = new Image(); newImg.onload = ()=>{ img = newImg; imgState = {scale:1, x:0, y:0, rotation:0, flipX:false, flipY:false}; scaleRange.value = 1; offsetX.value=0; offsetY.value=0; cropMode=false; cropRect=null; pushHistory(); draw(); }
      newImg.src = tmp.toDataURL();
    });
    cancelCrop.addEventListener('click', ()=>{ cropMode=false; cropRect=null; draw(); });

    // Text handling
    function addText(text){ const id = Date.now().toString(36); const t = {id, text, x:80, y:80 + texts.length*70, font: fontSelect.value, size: parseInt(fontSizeInput.value,10)||64, color: fontColorInput.value, bold: boldCheckbox.checked, shadow: shadowCheckbox.checked, stroke: strokeCheckbox.checked}; texts.push(t); selectedId=id; updateTextsList(); pushHistory(); draw(); }
    addTextBtn.addEventListener('click', ()=>{ const v=textInput.value.trim(); if(!v) return; addText(v); textInput.value=''; });

    // texts list UI
    function updateTextsList(){ textsList.innerHTML=''; texts.forEach(t=>{ const el = document.createElement('div'); el.className='text-item'; el.innerHTML = `<div style="flex:1"><strong>${escapeHtml(t.text)}</strong><div style='font-size:12px;color:var(--muted)'>${t.font} / ${t.size}px</div></div><div style='display:flex;gap:6px'><button data-id="${t.id}" class='btn small selectBtn'>選択</button><button data-id="${t.id}" class='btn small editBtn'>編集</button></div>`; textsList.appendChild(el); });
      [...textsList.querySelectorAll('.selectBtn')].forEach(b=> b.onclick = ()=>{ selectedId = b.dataset.id; syncSelectedProps(); draw(); });
      [...textsList.querySelectorAll('.editBtn')].forEach(b=> b.onclick = ()=>{ const id=b.dataset.id; const t = texts.find(x=>x.id===id); const newT = prompt('テキストを編集', t.text); if(newT!==null){ t.text=newT; pushHistory(); updateTextsList(); draw(); } });
    }

    function syncSelectedProps(){ const t = texts.find(x=>x.id===selectedId); if(!t) return; fontSizeInput.value = t.size; fontColorInput.value = t.color; boldCheckbox.checked = t.bold; shadowCheckbox.checked = t.shadow; strokeCheckbox.checked = t.stroke; fontSelect.value = t.font; }

    bringFrontBtn.addEventListener('click', ()=>{ if(!selectedId) return; const idx = texts.findIndex(x=>x.id===selectedId); if(idx>=0){ const [x] = texts.splice(idx,1); texts.push(x); pushHistory(); updateTextsList(); draw(); }});
    sendBackBtn.addEventListener('click', ()=>{ if(!selectedId) return; const idx = texts.findIndex(x=>x.id===selectedId); if(idx>=0){ const [x] = texts.splice(idx,1); texts.unshift(x); pushHistory(); updateTextsList(); draw(); }});
    deleteTextBtn.addEventListener('click', ()=>{ if(!selectedId) return; texts = texts.filter(x=>x.id!==selectedId); selectedId=null; pushHistory(); updateTextsList(); draw(); });

    // Text property changes
    fontSizeInput.addEventListener('input', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.size = parseInt(fontSizeInput.value,10)||40; pushHistory(); draw(); }});
    fontColorInput.addEventListener('input', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.color = fontColorInput.value; pushHistory(); draw(); }});
    boldCheckbox.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.bold = boldCheckbox.checked; pushHistory(); draw(); }});
    shadowCheckbox.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.shadow = shadowCheckbox.checked; pushHistory(); draw(); }});
    strokeCheckbox.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.stroke = strokeCheckbox.checked; pushHistory(); draw(); }});
    fontSelect.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.font = fontSelect.value; pushHistory(); draw(); }});

    // Dragging text
    let draggingText=false; let dragOffset={x:0,y:0};
    canvas.addEventListener('dblclick', e=>{ const pos = getMousePos(canvas,e); for(let i=texts.length-1;i>=0;i--){ const t = texts[i]; const m = ctx.measureText(t.text); const w=m.width; const h=t.size; if(pos.x>=t.x-6 && pos.x<=t.x+w+6 && pos.y>=t.y-6 && pos.y<=t.y+h+6){ const newT = prompt('テキスト編集', t.text); if(newT!==null){ t.text=newT; pushHistory(); updateTextsList(); draw(); } return; } } });

    // click to select, mousedown to start drag
    canvas.addEventListener('mousedown', e=>{
      const pos = getMousePos(canvas,e);
      if(cropMode) return; // handled earlier
      for(let i=texts.length-1;i>=0;i--){ const t=texts[i]; const m=ctx.measureText(t.text); const w=m.width; const h=t.size; if(pos.x>=t.x-6 && pos.x<=t.x+w+6 && pos.y>=t.y-6 && pos.y<=t.y+h+6){ selectedId = t.id; draggingText=true; dragOffset.x = pos.x - t.x; dragOffset.y = pos.y - t.y; updateTextsList(); draw(); return; } }
      selectedId = null; updateTextsList(); draw();
    });
    window.addEventListener('mousemove', e=>{ if(draggingText){ const pos = getMousePos(canvas,e); const t = texts.find(x=>x.id===selectedId); if(t){ t.x = pos.x - dragOffset.x; t.y = pos.y - dragOffset.y; draw(); } } });
    window.addEventListener('mouseup', ()=>{ if(draggingText){ draggingText=false; pushHistory(); } });

    // download PNG
    downloadPng.addEventListener('click', ()=>{
      const out = document.createElement('canvas'); out.width = canvas.width; out.height = canvas.height; const octx = out.getContext('2d');
      if(!bgTransparent.checked) { octx.fillStyle = bgColorInput.value; octx.fillRect(0,0,out.width,out.height); }
      // draw image
      if(img){ octx.save(); octx.translate(out.width/2 + imgState.x + parseFloat(offsetX.value||0), out.height/2 + imgState.y + parseFloat(offsetY.value||0)); octx.rotate(imgState.rotation * Math.PI/180); const sx = imgState.flipX ? -1:1; const sy = imgState.flipY? -1:1; octx.scale(sx,sy); const iw = img.width * imgState.scale; const ih = img.height * imgState.scale; octx.drawImage(img, -iw/2, -ih/2, iw, ih); octx.restore(); }
      // draw texts
      for(const t of texts){ octx.save(); octx.font = `${t.bold?900:400} ${t.size}px ${t.font}`; octx.textBaseline='top'; if(t.shadow){ octx.fillStyle = 'rgba(0,0,0,0.45)'; octx.fillText(t.text, t.x+4, t.y+4); } if(t.stroke){ octx.lineWidth = Math.max(2, Math.floor(t.size/12)); octx.strokeStyle = 'rgba(0,0,0,0.6)'; octx.strokeText(t.text, t.x, t.y); } octx.fillStyle = t.color; octx.fillText(t.text, t.x, t.y); octx.restore(); }
      const a = document.createElement('a'); a.href = out.toDataURL('image/png'); a.download = 'thumbnail.png'; a.click();
    });

    // helpers
    function getMousePos(canvas, evt){ const rect = canvas.getBoundingClientRect(); return { x: (evt.clientX - rect.left) * (canvas.width / rect.width), y: (evt.clientY - rect.top) * (canvas.height / rect.height) }; }
    function escapeHtml(s){ return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

    // utility: update texts UI every 300ms for selection highlight
    setInterval(()=>{ updateTextsList(); syncSelectedProps(); }, 300);

  </script>
</body>
</html>
