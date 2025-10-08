<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ゆいきち サムネ編集アプリ</title>
  <style>
    :root{
      --bg:#f7f7f8;
      --accent:#ff5c99;
      --muted:#666;
      --panel:#ffffff;
    }
    *{box-sizing:border-box}
    body{
      margin:0; font-family: Inter, system-ui, -apple-system, "Hiragino Kaku Gothic ProN", "メイリオ", Arial, sans-serif; background:var(--bg); color:#222; display:flex; flex-direction:column; min-height:100vh;
    }
    header{padding:18px 20px; display:flex; gap:12px; align-items:center; background:linear-gradient(90deg,#fff 0%, #fff 60%); box-shadow:0 1px 0 rgba(0,0,0,0.05)}
    header h1{font-size:18px;margin:0;color:var(--accent)}
    .container{display:flex; gap:20px; padding:20px; width:100%; max-width:1200px; margin:0 auto; flex:1}

    /* left: editor */
    .editor{flex:1; display:flex; flex-direction:column; align-items:center; gap:12px}
    .stage{background:linear-gradient(180deg,#fff,#fafafa); padding:12px; border-radius:12px; box-shadow:0 6px 20px rgba(0,0,0,0.06);}
    canvas{background:#222; border-radius:8px; display:block;}

    /* right: controls */
    .controls{width:340px; max-width:40%; background:var(--panel); padding:14px; border-radius:12px; box-shadow:0 6px 20px rgba(0,0,0,0.06)}
    .controls h2{margin:0 0 8px 0; font-size:16px; color:var(--muted)}
    .row{display:flex; gap:8px; align-items:center; margin-bottom:10px}
    label{font-size:13px; color:var(--muted)}
    input[type=text], input[type=color], select, input[type=range], input[type=number] {width:100%; padding:8px; border-radius:8px; border:1px solid #e6e6e6; font-size:14px}
    button{appearance:none; border:none; background:var(--accent); color:white; padding:8px 12px; border-radius:8px; cursor:pointer}
    .btn-ghost{background:transparent;color:var(--accent); border:1px solid rgba(0,0,0,0.06)}
    .small{padding:6px 8px;font-size:13px}
    .meta{font-size:12px;color:var(--muted)}
    .thumb{width:48px; height:48px; border-radius:6px; object-fit:cover; border:1px solid #eee}

    /* text list */
    .text-item{padding:8px;border-radius:8px;border:1px dashed #eee;margin-bottom:8px}
    .footer{padding:12px;text-align:center;font-size:13px;color:var(--muted)}

    @media(max-width:900px){.container{flex-direction:column}.controls{width:100%}}
  </style>
</head>
<body>
  <header>
    <h1>🎨 ゆいきち サムネメーカー</h1>
    <div class="meta">GitHub Pagesにそのまま置けます。編集→ダウンロードで完結。</div>
  </header>

  <div class="container">
    <div class="editor">
      <div class="stage">
        <canvas id="canvas" width="1280" height="720"></canvas>
      </div>
      <div style="display:flex; gap:8px; align-items:center;">
        <input id="file" type="file" accept="image/*">
        <button id="fit" class="small btn-ghost">画面に合わせる</button>
        <button id="clear" class="small">画像クリア</button>
        <button id="download" class="small">ダウンロード</button>
      </div>
      <div class="meta">キャンバスサイズ: <span id="sizeLabel">1280×720 (YouTubeサムネ推奨)</span></div>
    </div>

    <aside class="controls">
      <h2>画像調整</h2>
      <div class="row">
        <label style="width:100%;">拡大/縮小: <input id="scale" type="range" min="0.1" max="3" step="0.01" value="1"></label>
      </div>
      <div class="row">
        <label style="width:100%;">X位置: <input id="offsetX" type="range" min="-2000" max="2000" step="1" value="0"></label>
      </div>
      <div class="row">
        <label style="width:100%;">Y位置: <input id="offsetY" type="range" min="-2000" max="2000" step="1" value="0"></label>
      </div>

      <hr>
      <h2>テキスト追加</h2>
      <div class="row">
        <input id="textInput" type="text" placeholder="ここに文字を入力して追加">
        <button id="addText" class="small">追加</button>
      </div>
      <div class="row">
        <label>フォント:
          <select id="fontSelect">
            <option value="48px sans-serif">太ゴシック（デフォ）</option>
            <option value="48px 'Noto Sans JP', sans-serif">日本語サンセリフ</option>
            <option value="48px 'Georgia', serif">セリフ</option>
            <option value="48px 'Brush Script MT', cursive">手書き風</option>
          </select>
        </label>
      </div>
      <div class="row">
        <label style="flex:1">サイズ: <input id="fontSize" type="number" value="48" min="10" max="400"></label>
        <label style="width:110px">色: <input id="fontColor" type="color" value="#ffffff"></label>
      </div>
      <div class="row">
        <label style="flex:1"><input id="bold" type="checkbox"> 太字</label>
        <label style="flex:1"><input id="shadow" type="checkbox" checked> ドロップシャドウ</label>
      </div>

      <hr>
      <h2>テキスト管理</h2>
      <div id="textsList"></div>
      <div style="display:flex; gap:8px; margin-top:8px;">
        <button id="bringFront" class="small">前面へ</button>
        <button id="sendBack" class="small btn-ghost">背面へ</button>
        <button id="deleteText" class="small">削除</button>
      </div>

      <hr>
      <h2>背景</h2>
      <div class="row">
        <label>背景色: <input id="bgColor" type="color" value="#000000"></label>
      </div>
      <div class="row">
        <button id="resetAll" class="small btn-ghost">リセット</button>
      </div>

      <div class="footer">操作：ドラッグでテキスト移動 / テキストをクリックして選択。保存はダウンロード。</div>
    </aside>
  </div>

  <script>
    // 基本セットアップ
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const fileInput = document.getElementById('file');
    const scaleRange = document.getElementById('scale');
    const offsetX = document.getElementById('offsetX');
    const offsetY = document.getElementById('offsetY');
    const fitBtn = document.getElementById('fit');
    const clearBtn = document.getElementById('clear');
    const downloadBtn = document.getElementById('download');
    const textInput = document.getElementById('textInput');
    const addTextBtn = document.getElementById('addText');
    const fontSelect = document.getElementById('fontSelect');
    const fontSize = document.getElementById('fontSize');
    const fontColor = document.getElementById('fontColor');
    const boldCheckbox = document.getElementById('bold');
    const shadowCheckbox = document.getElementById('shadow');
    const textsList = document.getElementById('textsList');
    const bringFrontBtn = document.getElementById('bringFront');
    const sendBackBtn = document.getElementById('sendBack');
    const deleteTextBtn = document.getElementById('deleteText');
    const bgColorInput = document.getElementById('bgColor');
    const resetAllBtn = document.getElementById('resetAll');

    let img = null;
    let imgState = {scale:1, x:0, y:0};
    let texts = []; // {id, text, x, y, font, size, color, bold, shadow, selected}
    let selectedId = null;

    function draw(){
      // 背景
      ctx.fillStyle = bgColorInput.value;
      ctx.fillRect(0,0,canvas.width,canvas.height);

      // 画像
      if(img){
        const s = imgState.scale;
        const iw = img.width * s;
        const ih = img.height * s;
        const cx = canvas.width/2 + parseFloat(offsetX.value || 0) + imgState.x;
        const cy = canvas.height/2 + parseFloat(offsetY.value || 0) + imgState.y;
        ctx.save();
        ctx.translate(cx, cy);
        ctx.drawImage(img, -iw/2, -ih/2, iw, ih);
        ctx.restore();
      }

      // テキスト（描画順 = 配列順）
      for(const t of texts){
        ctx.save();
        ctx.font = `${t.bold? '700':'400'} ${t.size}px ${t.fontFamily}`;
        ctx.textAlign = 'left';
        ctx.textBaseline = 'top';
        if(t.shadow){
          ctx.fillStyle = 'rgba(0,0,0,0.6)';
          ctx.fillText(t.text, t.x+2, t.y+2);
        }
        ctx.fillStyle = t.color;
        ctx.fillText(t.text, t.x, t.y);
        // 選択枠
        if(t.id === selectedId){
          const metrics = ctx.measureText(t.text);
          const w = metrics.width;
          const h = t.size;
          ctx.strokeStyle = '#fff';
          ctx.lineWidth = 2;
          ctx.strokeRect(t.x-6, t.y-6, w+12, h+12);
        }
        ctx.restore();
      }
    }

    // 初回描画
    draw();

    // ファイル読み込み
    fileInput.addEventListener('change', (e)=>{
      const f = e.target.files[0];
      if(!f) return;
      const reader = new FileReader();
      reader.onload = ()=>{
        const tmp = new Image();
        tmp.onload = ()=>{
          img = tmp;
          // fit
          imgState.scale = Math.min(canvas.width/img.width, canvas.height/img.height);
          imgState.x = 0; imgState.y = 0;
          scaleRange.value = imgState.scale;
          offsetX.value = 0; offsetY.value = 0;
          draw();
        }
        tmp.src = reader.result;
      }
      reader.readAsDataURL(f);
    });

    fitBtn.addEventListener('click', ()=>{
      if(!img) return;
      imgState.scale = Math.min(canvas.width/img.width, canvas.height/img.height);
      scaleRange.value = imgState.scale;
      offsetX.value = 0; offsetY.value = 0;
      draw();
    });

    clearBtn.addEventListener('click', ()=>{
      img = null; draw(); fileInput.value = null;
    });

    // scale/offset の反映
    scaleRange.addEventListener('input', ()=>{ imgState.scale = parseFloat(scaleRange.value); draw(); });
    offsetX.addEventListener('input', ()=>{ draw(); });
    offsetY.addEventListener('input', ()=>{ draw(); });

    // テキスト操作
    function addText(text){
      const id = Date.now().toString(36);
      const size = parseInt(fontSize.value,10)||48;
      const fontFamily = (fontSelect.value.split(' ').slice(1).join(' ') || 'sans-serif').replace(/['"]+/g,'') || 'sans-serif';
      const entry = {id, text, x:100, y:100, fontFamily, size, color:fontColor.value, bold:boldCheckbox.checked, shadow:shadowCheckbox.checked};
      texts.push(entry);
      selectedId = id;
      rebuildList(); draw();
    }

    addTextBtn.addEventListener('click', ()=>{
      const v = textInput.value.trim();
      if(!v) return; addText(v); textInput.value='';
    });

    // リスト更新
    function rebuildList(){
      textsList.innerHTML='';
      texts.forEach(t=>{
        const el = document.createElement('div'); el.className='text-item';
        el.innerHTML = `<div style="display:flex;justify-content:space-between;align-items:center"><strong style="font-size:14px">${escapeHtml(t.text)}</strong><div style=\"display:flex;gap:8px\"><button data-id="${t.id}" class=\"selectBtn small\">選択</button><button data-id="${t.id}" class=\"editBtn small btn-ghost\">編集</button></div></div>`;
        textsList.appendChild(el);
      });
      // attach handlers
      [...textsList.querySelectorAll('.selectBtn')].forEach(b=>{
        b.onclick = ()=>{ selectedId = b.dataset.id; draw(); }
      });
      [...textsList.querySelectorAll('.editBtn')].forEach(b=>{
        b.onclick = ()=>{ const id=b.dataset.id; const t=texts.find(x=>x.id===id); const newT=prompt('テキスト編集', t.text); if(newT!==null){ t.text=newT; draw(); rebuildList(); } }
      });
    }

    // 選択操作ボタン
    bringFrontBtn.addEventListener('click', ()=>{
      if(!selectedId) return; const idx=texts.findIndex(t=>t.id===selectedId); if(idx>=0){ const [x]=texts.splice(idx,1); texts.push(x); rebuildList(); draw(); }
    });
    sendBackBtn.addEventListener('click', ()=>{
      if(!selectedId) return; const idx=texts.findIndex(t=>t.id===selectedId); if(idx>=0){ const [x]=texts.splice(idx,1); texts.unshift(x); rebuildList(); draw(); }
    });
    deleteTextBtn.addEventListener('click', ()=>{
      if(!selectedId) return; texts = texts.filter(t=>t.id!==selectedId); selectedId = null; rebuildList(); draw();
    });

    // クリックで選択、ドラッグで移動
    let dragging = false; let dragOffset = {x:0,y:0};
    canvas.addEventListener('mousedown', (e)=>{
      const pos = getMousePos(canvas,e);
      // 逆順で探す（上のテキストを優先）
      for(let i=texts.length-1;i>=0;i--){ const t = texts[i]; const metrics = ctx.measureText(t.text); const w = metrics.width; const h = t.size; if(pos.x >= t.x-6 && pos.x <= t.x + w + 6 && pos.y >= t.y-6 && pos.y <= t.y + h + 6){ selectedId = t.id; dragging = true; dragOffset.x = pos.x - t.x; dragOffset.y = pos.y - t.y; rebuildList(); draw(); return; } }
      selectedId = null; rebuildList(); draw();
    });
    window.addEventListener('mousemove', (e)=>{
      if(!dragging) return; const pos = getMousePos(canvas,e); const t = texts.find(x=>x.id===selectedId); if(t){ t.x = pos.x - dragOffset.x; t.y = pos.y - dragOffset.y; draw(); }
    });
    window.addEventListener('mouseup', ()=>{ dragging=false; });

    // ダブルクリックでテキスト編集
    canvas.addEventListener('dblclick', (e)=>{
      const pos = getMousePos(canvas,e);
      for(let i=texts.length-1;i>=0;i--){ const t = texts[i]; const metrics = ctx.measureText(t.text); const w = metrics.width; const h = t.size; if(pos.x >= t.x-6 && pos.x <= t.x + w + 6 && pos.y >= t.y-6 && pos.y <= t.y + h + 6){ const newT = prompt('テキストを編集', t.text); if(newT!==null){ t.text = newT; rebuildList(); draw(); } return; } }
    });

    function getMousePos(canvas, evt){ const rect = canvas.getBoundingClientRect(); return { x: (evt.clientX - rect.left) * (canvas.width / rect.width), y: (evt.clientY - rect.top) * (canvas.height / rect.height) }; }

    // ユーティリティ
    function escapeHtml(s){ return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

    // 選択中のプロパティ反映
    function syncSelectedProps(){
      const t = texts.find(x=>x.id===selectedId);
      if(!t) return;
      fontSize.value = t.size;
      fontColor.value = t.color;
      boldCheckbox.checked = t.bold;
      shadowCheckbox.checked = t.shadow;
      // font selectは not exact match - ignore
    }

    // プロパティ操作
    fontSize.addEventListener('input', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.size = parseInt(fontSize.value,10)||40; draw(); } });
    fontColor.addEventListener('input', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.color = fontColor.value; draw(); } });
    boldCheckbox.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.bold = boldCheckbox.checked; draw(); } });
    shadowCheckbox.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ t.shadow = shadowCheckbox.checked; draw(); } });
    fontSelect.addEventListener('change', ()=>{ const t = texts.find(x=>x.id===selectedId); if(t){ // parse size from value
      const val = fontSelect.value; const maybeSize = parseInt(val); t.fontFamily = val.split(' ').slice(1).join(' ').replace(/['"]+/g,'') || 'sans-serif'; draw(); } });

    // 選択同期ループ
    setInterval(()=>{ syncSelectedProps(); }, 300);

    // ダウンロード
    downloadBtn.addEventListener('click', ()=>{
      // high-res export: create temp canvas with same pixel size
      const out = document.createElement('canvas'); out.width = canvas.width; out.height = canvas.height; const octx = out.getContext('2d');
      // draw bg
      octx.fillStyle = bgColorInput.value; octx.fillRect(0,0,out.width,out.height);
      // draw image
      if(img){ const s = imgState.scale; const iw = img.width * s; const ih = img.height * s; const cx = out.width/2 + parseFloat(offsetX.value || 0) + imgState.x; const cy = out.height/2 + parseFloat(offsetY.value || 0) + imgState.y; octx.translate(cx, cy); octx.drawImage(img, -iw/2, -ih/2, iw, ih); octx.setTransform(1,0,0,1,0,0); }
      // draw texts
      for(const t of texts){ octx.save(); octx.font = `${t.bold? '700':'400'} ${t.size}px ${t.fontFamily}`; octx.textAlign='left'; octx.textBaseline='top'; if(t.shadow){ octx.fillStyle='rgba(0,0,0,0.6)'; octx.fillText(t.text, t.x+2, t.y+2); } octx.fillStyle=t.color; octx.fillText(t.text, t.x, t.y); octx.restore(); }
      const data = out.toDataURL('image/png');
      const a = document.createElement('a'); a.href = data; a.download = 'thumbnail.png'; a.click();
    });

    // リセット
    resetAllBtn.addEventListener('click', ()=>{ if(!confirm('リセットしますか？（テキストと画像が消えます）')) return; img=null; texts=[]; selectedId=null; fileInput.value=null; scaleRange.value=1; offsetX.value=0; offsetY.value=0; bgColorInput.value='#000000'; draw(); rebuildList(); });

    // helper: keep canvas size ratio controllable (optional)
    // For now standard 1280x720. Could add UI to change size.

  </script>
</body>
</html>
