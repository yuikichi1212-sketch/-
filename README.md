<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ゆいきちコーダー</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Kaushan+Script&display=swap" rel="stylesheet">
  <style>
    :root{
      --bg: linear-gradient(180deg,#f3f6ff,#fbfdff);
      --panel-bg: rgba(255,255,255,0.6);
      --muted:#6b7280;
      --accent:#0f172a;
      --glass-border: rgba(255,255,255,0.65);
      --shadow: 0 10px 30px rgba(10,20,40,0.06);
      --ui-w: 340px;
    }
    *{box-sizing:border-box}
    html,body{height:100%; margin:0; font-family:'Noto Sans JP',system-ui,-apple-system,"Hiragino Kaku Gothic ProN","Meiryo",sans-serif; background:var(--bg); color:var(--accent)}

    .wrap{max-width:1320px;margin:18px auto;display:grid;grid-template-columns:1fr var(--ui-w);gap:18px;padding:12px}
    header{display:flex;justify-content:space-between;align-items:center;gap:12px}
    h1{margin:0;font-size:18px}
    .sub{color:var(--muted);font-size:13px}

    /* Canvas area */
    .stage-panel{background:var(--panel-bg);border-radius:12px;padding:12px;border:1px solid var(--glass-border);box-shadow:var(--shadow);display:flex;flex-direction:column;align-items:center}
    .stage-toolbar{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:8px}
    .btn{background:transparent;border:1px solid rgba(15,23,42,0.07);padding:8px 10px;border-radius:10px;cursor:pointer;font-weight:700}
    .btn:hover{transform:translateY(-2px)}

    .canvas-wrap{background:linear-gradient(180deg,#fff,#f7fbff);padding:14px;border-radius:10px}
    canvas{display:block;border-radius:8px;background:transparent}

    /* Right panel compact toolbox */
    .tools{position:relative}
    .panel{background:var(--panel-bg);border-radius:12px;padding:12px;border:1px solid var(--glass-border);box-shadow:var(--shadow);display:flex;flex-direction:column;gap:10px}
    .section{border-radius:8px;padding:8px;background:rgba(255,255,255,0.35);}
    label{font-size:13px;color:var(--muted);display:block;margin-bottom:6px}
    input[type=text], input[type=number], select {width:100%;padding:8px;border-radius:8px;border:1px solid rgba(0,0,0,0.06)}
    input[type=color]{height:36px;border-radius:6px}

    .row{display:flex;gap:8px;align-items:center}
    .muted{color:var(--muted);font-size:13px}
    .texts-list{max-height:180px;overflow:auto}
    .text-item{padding:8px;border-radius:8px;border:1px dashed rgba(0,0,0,0.04);margin-bottom:8px;display:flex;justify-content:space-between;align-items:center}

    .compact{font-size:13px;padding:6px}
    .controls-row{display:flex;gap:6px}
    .divider{height:1px;background:rgba(0,0,0,0.04);margin:6px 0;border-radius:2px}

    /* resize handles visual (not DOM) - handled in canvas drawing */

    footer{margin-top:10px;text-align:center;color:var(--muted)}

    @media(max-width:1100px){.wrap{grid-template-columns:1fr}.tools{order:-1}}
  </style>
</head>
<body>
  <div class="wrap">
    <div>
      <header>
        <div>
          <h1>ゆいきちコーダー</h1>
          <div </div>
        </div>
        <div>
          <button id="downloadAll" class="btn">エクスポート</button>
        </div>
      </header>

      <div class="stage-panel">
        <div class="stage-toolbar">
          <input id="file" type="file" accept="image/*" class="btn compact">
          <button id="addImage" class="btn compact">画像追加(レイヤ)</button>
          <button id="addText" class="btn compact">テキスト追加</button>
          <button id="addRect" class="btn compact">四角</button>
          <button id="addCircle" class="btn compact">円</button>
          <button id="addTriangle" class="btn compact">三角</button>
          <button id="groupCenter" class="btn compact">中心揃え</button>
          <button id="snapCenter" class="btn compact">キャンバス中央へ</button>
          <button id="undo" class="btn compact">Undo</button>
          <button id="redo" class="btn compact">Redo</button>
          <select id="presetSize" class="btn compact">
            <option value="1280x720">1280×720 (YouTube)</option>
            <option value="1920x1080">1920×1080 (FullHD)</option>
            <option value="720x1280">720×1280 (縦)</option>
            <option value="1080x1080">1080×1080 (Instagram)</option>
          </select>
        </div>

        <div class="canvas-wrap">
          <canvas id="canvas" width="1280" height="720"></canvas>
        </div>
      </div>

      <footer>操作ヒント：選択 → ドラッグで移動 / 角をドラッグでリサイズ / 右クリックで回転ハンドル / Deleteで削除</footer>
    </div>

    <aside class="tools">
      <div class="panel">
        <div class="section">
          <label>選択オブジェクト</label>
          <div class="row">
            <div id="selType" class="muted">—</div>
            <div style="flex:1"></div>
            <button id="bringFront" class="btn compact">前へ</button>
            <button id="sendBack" class="btn compact">下へ</button>
          </div>
        </div>

        <div class="section">
          <label>テキストプロパティ</label>
          <input id="txtContent" type="text" placeholder="選択中のテキストを編集">
          <div class="row" style="margin-top:6px">
            <select id="txtFont"><option value="Noto Sans JP">Noto Sans JP</option><option value="sans-serif">sans-serif</option><option value="serif">serif</option><option value="Kaushan Script">手書き</option></select>
            <input id="txtSize" type="number" value="64" style="width:80px">
            <input id="txtColor" type="color" value="#ffffff">
          </div>
          <div class="row" style="margin-top:6px">
            <label><input id="txtBold" type="checkbox"> 太字</label>
            <label><input id="txtShadow" type="checkbox"> シャドウ</label>
            <label><input id="txtStroke" type="checkbox"> 袋文字(縁取り)</label>
          </div>
        </div>

        <div class="section">
          <label>図形 / 画像プロパティ</label>
          <div class="row">
            <label style="flex:1">塗り色: <input id="fillColor" type="color" value="#ff3b30"></label>
          </div>
          <div class="row" style="margin-top:6px">
            <label style="flex:1">枠線: <input id="strokeColor" type="color" value="#000000"></label>
            <input id="strokeWidth" type="number" value="4" style="width:80px">
          </div>
        </div>

        <div class="section">
          <label>配置 / サイズ</label>
          <div class="row">
            <label style="flex:1">X: <input id="propX" type="number" value="0"></label>
            <label style="flex:1">Y: <input id="propY" type="number" value="0"></label>
          </div>
          <div class="row" style="margin-top:6px">
            <label style="flex:1">幅: <input id="propW" type="number" value="200"></label>
            <label style="flex:1">高: <input id="propH" type="number" value="100"></label>
          </div>
          <div style="margin-top:6px" class="controls-row">
            <button id="alignLeft" class="btn compact">左揃え</button>
            <button id="alignCenter" class="btn compact">中央揃え</button>
            <button id="alignRight" class="btn compact">右揃え</button>
          </div>
        </div>

        <div class="section">
          <label>スナップ / ガイド</label>
          <div class="row">
            <label><input id="snapToCenter" type="checkbox" checked> 中央に吸着</label>
          </div>
          <div style="margin-top:6px" class="controls-row">
            <button id="distributeH" class="btn compact">均等配置H</button>
            <button id="distributeV" class="btn compact">均等配置V</button>
          </div>
        </div>

        <div class="section">
          <label>その他</label>
          <div class="row"><button id="exportPng" class="btn compact">PNG出力</button><button id="exportProject" class="btn compact">作業保存</button></div>
          <div class="row" style="margin-top:6px"><button id="loadProject" class="btn compact">作業読み込み</button><button id="clearAll" class="btn compact">全消去</button></div>
        </div>
      </div>
    </aside>
  </div>

  <script>
    // Object model: objects = [{id, type:'image'|'text'|'rect'|'circle'|'triangle', x,y,w,h,rotation,flipX,flipY,props:{...}}]
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    // Elements
    const fileInput = document.getElementById('file');
    const addImageBtn = document.getElementById('addImage');
    const addTextBtn = document.getElementById('addText');
    const addRectBtn = document.getElementById('addRect');
    const addCircleBtn = document.getElementById('addCircle');
    const addTriangleBtn = document.getElementById('addTriangle');
    const presetSize = document.getElementById('presetSize');
    const downloadAll = document.getElementById('downloadAll');

    const selType = document.getElementById('selType');
    const txtContent = document.getElementById('txtContent');
    const txtFont = document.getElementById('txtFont');
    const txtSize = document.getElementById('txtSize');
    const txtColor = document.getElementById('txtColor');
    const txtBold = document.getElementById('txtBold');
    const txtShadow = document.getElementById('txtShadow');
    const txtStroke = document.getElementById('txtStroke');

    const fillColor = document.getElementById('fillColor');
    const strokeColor = document.getElementById('strokeColor');
    const strokeWidth = document.getElementById('strokeWidth');

    const propX = document.getElementById('propX');
    const propY = document.getElementById('propY');
    const propW = document.getElementById('propW');
    const propH = document.getElementById('propH');

    const bringFront = document.getElementById('bringFront');
    const sendBack = document.getElementById('sendBack');
    const alignLeft = document.getElementById('alignLeft');
    const alignCenter = document.getElementById('alignCenter');
    const alignRight = document.getElementById('alignRight');
    const snapToCenter = document.getElementById('snapToCenter');
    const groupCenter = document.getElementById('groupCenter');
    const snapCenterBtn = document.getElementById('snapCenter');
    const distributeH = document.getElementById('distributeH');
    const distributeV = document.getElementById('distributeV');

    const undoBtn = document.getElementById('undo');
    const redoBtn = document.getElementById('redo');
    const exportPng = document.getElementById('exportPng');
    const exportProject = document.getElementById('exportProject');
    const loadProject = document.getElementById('loadProject');
    const clearAll = document.getElementById('clearAll');

    // State
    let objects = [];
    let selectedId = null;
    let dragging = false; let dragOffset = {x:0,y:0};
    let resizing = false; let resizeHandle = null; let rotateMode=false; let rotateStartAngle=0;

    // history
    let history = []; let histIndex = -1; const MAX_HISTORY=80;
    function pushHistory(){ const snap = JSON.stringify({objects, canvasSize:{w:canvas.width,h:canvas.height}}); if(histIndex < history.length-1) history = history.slice(0,histIndex+1); history.push(snap); if(history.length>MAX_HISTORY) history.shift(); histIndex = history.length-1; updateUndoRedo(); }
    function restoreHistory(idx){ if(idx<0 || idx>=history.length) return; const snap = JSON.parse(history[idx]); objects = snap.objects; canvas.width = snap.canvasSize.w; canvas.height = snap.canvasSize.h; histIndex = idx; selectedId=null; draw(); updateUndoRedo(); }
    function updateUndoRedo(){ undoBtn.disabled = histIndex<=0; redoBtn.disabled = histIndex>=history.length-1; }

    undoBtn.addEventListener('click', ()=>{ if(histIndex>0) restoreHistory(histIndex-1); });
    redoBtn.addEventListener('click', ()=>{ if(histIndex < history.length-1) restoreHistory(histIndex+1); });

    // Helpers
    function uid(){ return Date.now().toString(36) + Math.random().toString(36).slice(2,6); }
    function getObjectById(id){ return objects.find(o=>o.id===id); }

    // Add basic objects
    function addRect(){ const o={id:uid(), type:'rect', x:100,y:80,w:400,h:200, rotation:0, props:{fill:fillColor.value, stroke:strokeColor.value, strokeWidth:parseInt(strokeWidth.value)||4}}; objects.push(o); selectedId=o.id; pushHistory(); draw(); }
    function addCircle(){ const o={id:uid(), type:'circle', x:200,y:120,w:220,h:220, rotation:0, props:{fill:fillColor.value, stroke:strokeColor.value, strokeWidth:parseInt(strokeWidth.value)||4}}; objects.push(o); selectedId=o.id; pushHistory(); draw(); }
    function addTriangle(){ const o={id:uid(), type:'triangle', x:160,y:100,w:320,h:260, rotation:0, props:{fill:fillColor.value, stroke:strokeColor.value, strokeWidth:parseInt(strokeWidth.value)||4}}; objects.push(o); selectedId=o.id; pushHistory(); draw(); }
    function addText(){ const text = prompt('テキストを入力'); if(!text) return; const o={id:uid(), type:'text', x:120,y:120,w:300,h:100, rotation:0, props:{text, font:txtFont.value, size:parseInt(txtSize.value)||64, color:txtColor.value, bold:txtBold.checked, shadow:txtShadow.checked, stroke:txtStroke.checked}}; objects.push(o); selectedId=o.id; pushHistory(); draw(); }

    // Add image (as object) from file input
    fileInput.addEventListener('change', e=>{ const f = e.target.files[0]; if(!f) return; const reader = new FileReader(); reader.onload = ()=>{ const img = new Image(); img.onload = ()=>{ const o = {id:uid(), type:'image', x:canvas.width/2 - img.width/4, y:canvas.height/2 - img.height/4, w: img.width/2, h: img.height/2, rotation:0, props:{image:img}}; objects.push(o); selectedId=o.id; pushHistory(); draw(); }; img.src = reader.result; }; reader.readAsDataURL(f); });

    addImageBtn.addEventListener('click', ()=> fileInput.click());
    addTextBtn.addEventListener('click', addText);
    addRectBtn.addEventListener('click', addRect);
    addCircleBtn.addEventListener('click', addCircle);
    addTriangleBtn.addEventListener('click', addTriangle);

    // Draw loop
    function draw(){ ctx.clearRect(0,0,canvas.width,canvas.height);
      // background
      ctx.save(); ctx.fillStyle = '#00000000'; ctx.fillRect(0,0,canvas.width,canvas.height); ctx.restore();
      // draw objects in order
      for(const o of objects){ drawObject(o); }
      // draw selection handles
      if(selectedId){ const s = getObjectById(selectedId); if(s) drawSelectionHandles(s); }
    }

    function drawObject(o){ ctx.save(); ctx.translate(o.x + o.w/2, o.y + o.h/2); ctx.rotate((o.rotation||0)*Math.PI/180); ctx.translate(-o.w/2, -o.h/2);
      if(o.type==='rect'){ ctx.fillStyle = o.props.fill; ctx.fillRect(0,0,o.w,o.h); if(o.props.strokeWidth){ ctx.lineWidth=o.props.strokeWidth; ctx.strokeStyle=o.props.stroke; ctx.strokeRect(0,0,o.w,o.h); } }
      if(o.type==='circle'){ ctx.beginPath(); ctx.arc(o.w/2, o.h/2, Math.min(o.w,o.h)/2, 0, Math.PI*2); ctx.closePath(); ctx.fillStyle=o.props.fill; ctx.fill(); if(o.props.strokeWidth){ ctx.lineWidth=o.props.strokeWidth; ctx.strokeStyle=o.props.stroke; ctx.stroke(); } }
      if(o.type==='triangle'){ ctx.beginPath(); ctx.moveTo(o.w/2,0); ctx.lineTo(o.w, o.h); ctx.lineTo(0,o.h); ctx.closePath(); ctx.fillStyle=o.props.fill; ctx.fill(); if(o.props.strokeWidth){ ctx.lineWidth=o.props.strokeWidth; ctx.strokeStyle=o.props.stroke; ctx.stroke(); } }
      if(o.type==='image'){ if(o.props.image){ ctx.drawImage(o.props.image, 0, 0, o.w, o.h); } }
      if(o.type==='text'){ const p=o.props; ctx.font = `${p.bold?900:400} ${p.size}px ${p.font}`; ctx.textBaseline='top'; ctx.fillStyle=p.color; if(p.shadow){ ctx.fillStyle='rgba(0,0,0,0.45)'; ctx.fillText(p.text, 4, 4); ctx.fillStyle=p.color; }
        if(p.stroke){ ctx.lineWidth = Math.max(2, Math.floor(p.size/12)); ctx.strokeStyle = 'black'; ctx.strokeText(p.text, 0,0); }
        ctx.fillText(p.text, 0,0); // update w/h measurement
        const m = ctx.measureText(p.text); // override width
        // don't auto-change w/h here to preserve manual sizing
      }
      ctx.restore(); }

    function drawSelectionHandles(o){ ctx.save(); ctx.strokeStyle='rgba(255,255,255,0.95)'; ctx.lineWidth=2; const x=o.x,y=o.y,w=o.w,h=o.h; // draw rect around
      // compute rotated corners
      // draw bounding box (unrotated for simplicity)
      ctx.strokeStyle='rgba(255,255,255,0.9)'; ctx.setLineDash([6,4]); ctx.strokeRect(x-6,y-6,w+12,h+12); ctx.setLineDash([]);
      // draw handles
      const handles = [{x:x,y:y},{x:x+w,y:y},{x:x+w,y:y+h},{x:x,y:y+h}]; ctx.fillStyle='white'; for(const hpos of handles){ ctx.beginPath(); ctx.arc(hpos.x, hpos.y, 8,0,Math.PI*2); ctx.fill(); ctx.strokeStyle='rgba(0,0,0,0.08)'; ctx.stroke(); }
      // rotation handle
      ctx.beginPath(); ctx.arc(x + w/2, y - 24, 6,0,Math.PI*2); ctx.fill(); ctx.stroke(); ctx.restore(); }

    // Hit test
    function hitTest(pos){ // return topmost object under pos
      for(let i=objects.length-1;i>=0;i--){ const o=objects[i]; // axis-aligned check (rotation supported approximate)
        if(pos.x >= o.x && pos.x <= o.x+o.w && pos.y >= o.y && pos.y <= o.y+o.h){ return o; } }
      return null; }

    function getHandleAtPos(o,pos){ // approx corners
      const corners = [{x:o.x,y:o.y,name:'nw'},{x:o.x+o.w,y:o.y,name:'ne'},{x:o.x+o.w,y:o.y+o.h,name:'se'},{x:o.x,y:o.y+o.h,name:'sw'}]; for(const c of corners){ const dx=pos.x-c.x, dy=pos.y-c.y; if(Math.sqrt(dx*dx+dy*dy) < 12) return c.name; }
      // rotation handle
      const rx=o.x+o.w/2, ry=o.y-24; const d=Math.hypot(pos.x-rx,pos.y-ry); if(d<10) return 'rotate'; return null; }

    // Mouse events
    canvas.addEventListener('mousedown', e=>{
      const pos = getMousePos(canvas,e);
      const obj = hitTest(pos);
      if(obj){ selectedId = obj.id; const handle = getHandleAtPos(obj,pos); if(handle){ resizing=true; resizeHandle=handle; } else { dragging=true; dragOffset.x = pos.x - obj.x; dragOffset.y = pos.y - obj.y; }
        updatePropertiesUI(); draw(); }
      else { selectedId=null; draw(); updatePropertiesUI(); }
    });

    window.addEventListener('mousemove', e=>{
      const pos = getMousePos(canvas,e);
      if(dragging && selectedId){ const o=getObjectById(selectedId); o.x = pos.x - dragOffset.x; o.y = pos.y - dragOffset.y; propX.value = Math.round(o.x); propY.value = Math.round(o.y); draw(); }
      if(resizing && selectedId){ const o=getObjectById(selectedId); const p=pos; if(resizeHandle==='se'){ o.w = Math.max(20, p.x - o.x); o.h = Math.max(20, p.y - o.y); propW.value = Math.round(o.w); propH.value = Math.round(o.h); } if(resizeHandle==='nw'){ const nx = p.x; const ny = p.y; const oldR = o.x + o.w; const oldB = o.y + o.h; o.x = Math.min(nx, oldR-20); o.y = Math.min(ny, oldB-20); o.w = Math.max(20, oldR - o.x); o.h = Math.max(20, oldB - o.y); propX.value = Math.round(o.x); propY.value = Math.round(o.y); propW.value=Math.round(o.w); propH.value=Math.round(o.h);} if(resizeHandle==='ne'){ o.y = Math.min(p.y, o.y+o.h-20); o.w = Math.max(20, p.x - o.x); o.h = Math.max(20, o.y + o.h - o.y); propW.value=Math.round(o.w); propH.value=Math.round(o.h);} if(resizeHandle==='sw'){ o.x = Math.min(p.x, o.x+o.w-20); o.w = Math.max(20, o.x + o.w - o.x); o.h = Math.max(20, p.y - o.y); propX.value=Math.round(o.x); propW.value=Math.round(o.w); propH.value=Math.round(o.h);} draw(); }
    });

    window.addEventListener('mouseup', e=>{ if(dragging || resizing){ pushHistory(); } dragging=false; resizing=false; resizeHandle=null; });

    // Keyboard shortcuts
    window.addEventListener('keydown', e=>{
      const meta = e.ctrlKey || e.metaKey;
      if(meta && e.key.toLowerCase()==='z'){ e.preventDefault(); if(e.shiftKey) { if(histIndex < history.length-1) restoreHistory(histIndex+1); } else { if(histIndex>0) restoreHistory(histIndex-1); } }
      if(meta && e.key.toLowerCase()==='y'){ e.preventDefault(); if(histIndex < history.length-1) restoreHistory(histIndex+1); }
      if(e.key === 'Delete' || e.key==='Backspace'){ if(selectedId){ objects = objects.filter(o=>o.id!==selectedId); selectedId=null; pushHistory(); draw(); } }
      // arrow nudges
      if(['ArrowLeft','ArrowRight','ArrowUp','ArrowDown'].includes(e.key)){ const step = e.shiftKey?10:1; const o=getObjectById(selectedId); if(o){ if(e.key==='ArrowLeft') o.x-=step; if(e.key==='ArrowRight') o.x+=step; if(e.key==='ArrowUp') o.y-=step; if(e.key==='ArrowDown') o.y+=step; pushHistory(); draw(); } }
    });

    // Property UI bindings
    function updatePropertiesUI(){ const o=getObjectById(selectedId); if(o){ selType.textContent = o.type; propX.value = Math.round(o.x); propY.value = Math.round(o.y); propW.value = Math.round(o.w); propH.value = Math.round(o.h); if(o.type==='text'){ txtContent.value = o.props.text; txtFont.value = o.props.font || 'Noto Sans JP'; txtSize.value = o.props.size||64; txtColor.value = o.props.color||'#ffffff'; txtBold.checked = !!o.props.bold; txtShadow.checked = !!o.props.shadow; txtStroke.checked = !!o.props.stroke; } else { txtContent.value=''; } } else { selType.textContent='—'; txtContent.value=''; }
    }

    // props input listeners
    txtContent.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.text = txtContent.value; pushHistory(); draw(); }});
    txtFont.addEventListener('change', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.font = txtFont.value; pushHistory(); draw(); }});
    txtSize.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.size = parseInt(txtSize.value)||32; pushHistory(); draw(); }});
    txtColor.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.color = txtColor.value; pushHistory(); draw(); }});
    txtBold.addEventListener('change', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.bold = txtBold.checked; pushHistory(); draw(); }});
    txtShadow.addEventListener('change', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.shadow = txtShadow.checked; pushHistory(); draw(); }});
    txtStroke.addEventListener('change', ()=>{ const o=getObjectById(selectedId); if(o && o.type==='text'){ o.props.stroke = txtStroke.checked; pushHistory(); draw(); }});

    fillColor.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && (o.type==='rect' || o.type==='circle' || o.type==='triangle')){ o.props.fill = fillColor.value; pushHistory(); draw(); }});
    strokeColor.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && (o.type!=='text')){ o.props.stroke = strokeColor.value; pushHistory(); draw(); }});
    strokeWidth.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o && (o.type!=='text')){ o.props.strokeWidth = parseInt(strokeWidth.value)||1; pushHistory(); draw(); }});

    propX.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o){ o.x = parseInt(propX.value)||0; draw(); }});
    propY.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o){ o.y = parseInt(propY.value)||0; draw(); }});
    propW.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o){ o.w = Math.max(4, parseInt(propW.value)||4); draw(); }});
    propH.addEventListener('input', ()=>{ const o=getObjectById(selectedId); if(o){ o.h = Math.max(4, parseInt(propH.value)||4); draw(); }});

    bringFront.addEventListener('click', ()=>{ if(!selectedId) return; const idx=objects.findIndex(o=>o.id===selectedId); if(idx>=0){ const [x]=objects.splice(idx,1); objects.push(x); pushHistory(); draw(); }});
    sendBack.addEventListener('click', ()=>{ if(!selectedId) return; const idx=objects.findIndex(o=>o.id===selectedId); if(idx>=0){ const [x]=objects.splice(idx,1); objects.unshift(x); pushHistory(); draw(); }});

    alignLeft.addEventListener('click', ()=>{ if(!selectedId) return; const o=getObjectById(selectedId); o.x = 20; pushHistory(); draw(); });
    alignCenter.addEventListener('click', ()=>{ if(!selectedId) return; const o=getObjectById(selectedId); o.x = (canvas.width - o.w)/2; pushHistory(); draw(); });
    alignRight.addEventListener('click', ()=>{ if(!selectedId) return; const o=getObjectById(selectedId); o.x = canvas.width - o.w - 20; pushHistory(); draw(); });

    groupCenter.addEventListener('click', ()=>{ if(objects.length===0) return; const avgX = objects.reduce((s,o)=>s+(o.x+o.w/2),0)/objects.length; const avgY = objects.reduce((s,o)=>s+(o.y+o.h/2),0)/objects.length; const cx = canvas.width/2, cy = canvas.height/2; const dx = cx-avgX, dy=cy-avgY; for(const o of objects){ o.x += dx; o.y += dy; } pushHistory(); draw(); });
    snapCenterBtn.addEventListener('click', ()=>{ if(!selectedId) return; const o=getObjectById(selectedId); o.x = (canvas.width - o.w)/2; o.y = (canvas.height - o.h)/2; pushHistory(); draw(); });

    distributeH.addEventListener('click', ()=>{ if(objects.length<2) return; const left = Math.min(...objects.map(o=>o.x)); const right = Math.max(...objects.map(o=>o.x+o.w)); const gap = (right-left)/ (objects.length-1); objects.sort((a,b)=>a.x-b.x); for(let i=0;i<objects.length;i++){ objects[i].x = left + gap*i; } pushHistory(); draw(); });
    distributeV.addEventListener('click', ()=>{ if(objects.length<2) return; const top = Math.min(...objects.map(o=>o.y)); const bottom = Math.max(...objects.map(o=>o.y+o.h)); const gap = (bottom-top)/ (objects.length-1); objects.sort((a,b)=>a.y-b.y); for(let i=0;i<objects.length;i++){ objects[i].y = top + gap*i; } pushHistory(); draw(); });

    // Clear / export / project
    clearAll.addEventListener('click', ()=>{ if(!confirm('全て消しますか？')) return; objects=[]; selectedId=null; pushHistory(); draw(); });
    exportPng.addEventListener('click', ()=>{ exportToPNG(); });
    downloadAll.addEventListener('click', ()=>{ exportToPNG(); });

    exportProject.addEventListener('click', ()=>{ const json = JSON.stringify({objects,canvas:{w:canvas.width,h:canvas.height}}); const blob = new Blob([json],{type:'application/json'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'project.json'; a.click(); });
    loadProject.addEventListener('click', ()=>{ const input = document.createElement('input'); input.type='file'; input.accept='.json'; input.onchange = e=>{ const f = e.target.files[0]; if(!f) return; const r = new FileReader(); r.onload = ()=>{ try{ const obj = JSON.parse(r.result); objects = obj.objects || []; canvas.width = obj.canvas.w || canvas.width; canvas.height = obj.canvas.h || canvas.height; pushHistory(); draw(); }catch(err){ alert('読み込み失敗'); } }; r.readAsText(f); }; input.click(); });

    // presets
    presetSize.addEventListener('change', ()=>{ const v = presetSize.value.split('x'); canvas.width = parseInt(v[0]); canvas.height = parseInt(v[1]); pushHistory(); draw(); });

    // export util
    function exportToPNG(){ const out = document.createElement('canvas'); out.width = canvas.width; out.height = canvas.height; const octx = out.getContext('2d'); // draw bg transparent
      octx.fillStyle = '#ffffff00'; octx.fillRect(0,0,out.width,out.height); // draw objects
      for(const o of objects){ // reuse draw logic but adapted
        octx.save(); octx.translate(o.x + o.w/2, o.y + o.h/2); octx.rotate((o.rotation||0)*Math.PI/180); octx.translate(-o.w/2, -o.h/2);
        if(o.type==='rect'){ octx.fillStyle=o.props.fill; octx.fillRect(0,0,o.w,o.h); if(o.props.strokeWidth){ octx.lineWidth=o.props.strokeWidth; octx.strokeStyle=o.props.stroke; octx.strokeRect(0,0,o.w,o.h); } }
        if(o.type==='circle'){ octx.beginPath(); octx.arc(o.w/2,o.h/2,Math.min(o.w,o.h)/2,0,Math.PI*2); octx.closePath(); octx.fillStyle=o.props.fill; octx.fill(); if(o.props.strokeWidth){ octx.lineWidth=o.props.strokeWidth; octx.strokeStyle=o.props.stroke; octx.stroke(); } }
        if(o.type==='triangle'){ octx.beginPath(); octx.moveTo(o.w/2,0); octx.lineTo(o.w,o.h); octx.lineTo(0,o.h); octx.closePath(); octx.fillStyle=o.props.fill; octx.fill(); if(o.props.strokeWidth){ octx.lineWidth=o.props.strokeWidth; octx.strokeStyle=o.props.stroke; octx.stroke(); } }
        if(o.type==='image'){ if(o.props.image) octx.drawImage(o.props.image,0,0,o.w,o.h); }
        if(o.type==='text'){ const p=o.props; octx.font=`${p.bold?900:400} ${p.size}px ${p.font}`; octx.textBaseline='top'; if(p.shadow){ octx.fillStyle='rgba(0,0,0,0.45)'; octx.fillText(p.text, 4,4); } if(p.stroke){ octx.lineWidth = Math.max(2,Math.floor(p.size/12)); octx.strokeStyle='black'; octx.strokeText(p.text,0,0); } octx.fillStyle = p.color; octx.fillText(p.text,0,0); }
        octx.restore(); }
      const a = document.createElement('a'); a.href = out.toDataURL('image/png'); a.download = 'thumbnail.png'; a.click(); }

    // Utilities
    function getMousePos(canvas,evt){ const rect = canvas.getBoundingClientRect(); return { x: (evt.clientX - rect.left)*(canvas.width/rect.width), y: (evt.clientY - rect.top)*(canvas.height/rect.height) }; }

    // init
    pushHistory(); draw();

    // save final state on unload
    window.addEventListener('beforeunload', ()=>{ localStorage.setItem('yui-editor-last', JSON.stringify({objects, canvas:{w:canvas.width,h:canvas.height}})); });
    // load last session
    const last = localStorage.getItem('yui-editor-last'); if(last){ try{ const o = JSON.parse(last); if(o.objects) { objects = o.objects; canvas.width=o.canvas.w; canvas.height=o.canvas.h; pushHistory(); draw(); } }catch(e){} }

  </script>
</body>
</html>
