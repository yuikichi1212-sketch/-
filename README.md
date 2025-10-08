<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ゆいきち ドキュメント（Google Docs風）</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700;900&family=Roboto+Slab:wght@400;700&display=swap" rel="stylesheet">
  <style>
    :root{--bg:#f5f7fb;--panel:#ffffff;--muted:#6b7280;--accent:#0b3b66}
    *{box-sizing:border-box}
    body{margin:0;font-family:'Noto Sans JP',system-ui, -apple-system, 'Hiragino Kaku Gothic ProN','Meiryo',sans-serif;background:var(--bg);color:var(--accent);}
    header{display:flex;align-items:center;justify-content:space-between;padding:12px 18px;background:linear-gradient(180deg,#fff,#f7fbff);box-shadow:0 3px 10px rgba(10,20,40,0.05)}
    .brand{display:flex;align-items:center;gap:12px}
    .brand h1{margin:0;font-size:16px}
    .toolbar{display:flex;gap:8px;align-items:center}
    .toolbar select,.toolbar button,.toolbar input[type=color]{padding:8px;border-radius:6px;border:1px solid rgba(0,0,0,0.06);background:white}
    .main{display:flex;gap:16px;padding:18px}
    .sidebar{width:220px}
    .sidebar .panel{background:var(--panel);padding:12px;border-radius:10px;box-shadow:0 6px 18px rgba(10,20,40,0.04)}
    .editor-wrap{flex:1;display:flex;flex-direction:column;align-items:center}
    .doc{width:842px;min-height:1188px;background:white;padding:48px;border-radius:6px;box-shadow:0 6px 20px rgba(10,20,40,0.06);overflow:auto}
    .doc[contenteditable]{outline:none}
    .status{display:flex;gap:10px;align-items:center;margin-top:8px;color:var(--muted);font-size:13px}
    .btn{cursor:pointer}

    /* small helpers */
    .row{display:flex;gap:8px;align-items:center}
    input[type=number]{width:64px}
    .modal{position:fixed;left:0;top:0;right:0;bottom:0;display:flex;align-items:center;justify-content:center;background:rgba(0,0,0,0.35)}
    .modal .card{background:white;padding:16px;border-radius:8px;min-width:320px}

    @media(max-width:1000px){.main{flex-direction:column}.sidebar{width:100%}}
  </style>
</head>
<body>
  <header>
    <div class="brand"><h1>ゆいきち ドキュメント</h1><div style="color:var(--muted);font-size:13px">Word/Google Docs ライク（オフラインで動く）</div></div>

    <div class="toolbar">
      <select id="fontName"><option value="Noto Sans JP">Noto Sans JP</option><option value="Roboto Slab">Roboto Slab</option><option value="serif">Serif</option><option value="monospace">Monospace</option></select>
      <select id="fontSize"><option>10</option><option>11</option><option selected>12</option><option>14</option><option>18</option><option>24</option><option>36</option><option>48</option></select>
      <button id="boldBtn"><b>B</b></button>
      <button id="italicBtn"><i>I</i></button>
      <button id="underlineBtn">U</button>
      <button id="strikeBtn">S</button>
      <button id="headingBtn">H1</button>
      <button id="quoteBtn">❝</button>
      <button id="olBtn">1.</button>
      <button id="ulBtn">•</button>
      <button id="alignLeft">≡</button>
      <button id="alignCenter">⋯</button>
      <button id="alignRight">≣</button>
      <input id="colorBtn" type="color" value="#000000" title="文字色">
      <input id="highlightBtn" type="color" value="#ffff00" title="ハイライト">
      <button id="insertImage">画像挿入</button>
      <button id="insertTable">表</button>
      <button id="findBtn">検索</button>
      <button id="undoBtn">↶</button>
      <button id="redoBtn">↷</button>
      <button id="saveBtn" class="btn">保存</button>
      <button id="downloadHtml" class="btn">HTML保存</button>
      <button id="downloadDoc" class="btn">Word保存(.doc)</button>
      <button id="printBtn" class="btn">印刷</button>
    </div>
  </header>

  <div class="main">
    <aside class="sidebar">
      <div class="panel">
        <h3 style="margin-top:0">スタイル</h3>
        <div class="row"><label>行間</label><input id="lineHeight" type="number" value="1.6" step="0.1"></div>
        <div style="height:10px"></div>
        <h4>テンプレート</h4>
        <div class="row"><button id="tplBlank" class="btn">白紙</button><button id="tplReport" class="btn">報告書</button></div>

        <div style="height:10px"></div>
        <h4>目次 / 協力</h4>
        <div class="row"><button id="generateToc" class="btn">目次生成</button><button id="wordCount" class="btn">文字数</button></div>

        <div style="height:10px"></div>
        <h4>ファイル</h4>
        <div class="row"><button id="loadFile" class="btn">読み込み</button><button id="clearAll" class="btn">全消去</button></div>
      </div>
    </aside>

    <section class="editor-wrap">
      <div id="doc" class="doc" contenteditable="true" spellcheck="true">
        <h1>タイトル</h1>
        <p>ここに本文を書きます。Googleドキュメントのように自由に編集できます。</p>
      </div>
      <div class="status"><div id="statusSave">自動保存: オン</div><div id="statusCount">単語: 0</div><div id="statusSel">選択: 0</div></div>
    </section>
  </div>

  <!-- hidden inputs -->
  <input type="file" id="imageLoader" accept="image/*" style="display:none">

  <!-- find modal -->
  <div id="findModal" class="modal" style="display:none"><div class="card"><h4>検索と置換</h4><div class="row"><input id="findInput" placeholder="検索語"></div><div class="row" style="margin-top:8px"><input id="replaceInput" placeholder="置換語"></div><div style="margin-top:8px" class="row"><button id="doFind" class="btn">検索</button><button id="doReplace" class="btn">置換</button><button id="closeFind" class="btn">閉じる</button></div></div></div>

  <script>
    // Helper: exec command wrapper
    function exec(cmd, val=null){ document.execCommand(cmd, false, val); doc.focus(); }

    const doc = document.getElementById('doc');
    const fontName = document.getElementById('fontName');
    const fontSize = document.getElementById('fontSize');
    const boldBtn = document.getElementById('boldBtn');
    const italicBtn = document.getElementById('italicBtn');
    const underlineBtn = document.getElementById('underlineBtn');
    const strikeBtn = document.getElementById('strikeBtn');
    const headingBtn = document.getElementById('headingBtn');
    const quoteBtn = document.getElementById('quoteBtn');
    const olBtn = document.getElementById('olBtn');
    const ulBtn = document.getElementById('ulBtn');
    const alignLeft = document.getElementById('alignLeft');
    const alignCenter = document.getElementById('alignCenter');
    const alignRight = document.getElementById('alignRight');
    const colorBtn = document.getElementById('colorBtn');
    const highlightBtn = document.getElementById('highlightBtn');
    const insertImage = document.getElementById('insertImage');
    const insertTable = document.getElementById('insertTable');
    const findBtn = document.getElementById('findBtn');
    const undoBtn = document.getElementById('undoBtn');
    const redoBtn = document.getElementById('redoBtn');
    const saveBtn = document.getElementById('saveBtn');
    const downloadHtml = document.getElementById('downloadHtml');
    const downloadDoc = document.getElementById('downloadDoc');
    const printBtn = document.getElementById('printBtn');
    const imageLoader = document.getElementById('imageLoader');

    // basic commands
    boldBtn.onclick = ()=>exec('bold');
    italicBtn.onclick = ()=>exec('italic');
    underlineBtn.onclick = ()=>exec('underline');
    strikeBtn.onclick = ()=>exec('strikeThrough');
    headingBtn.onclick = ()=>exec('formatBlock', '<h1>');
    quoteBtn.onclick = ()=>exec('formatBlock', '<blockquote>');
    olBtn.onclick = ()=>exec('insertOrderedList');
    ulBtn.onclick = ()=>exec('insertUnorderedList');
    alignLeft.onclick = ()=>exec('justifyLeft');
    alignCenter.onclick = ()=>exec('justifyCenter');
    alignRight.onclick = ()=>exec('justifyRight');

    fontName.addEventListener('change', ()=>exec('fontName', fontName.value));
    fontSize.addEventListener('change', ()=>exec('fontSize', fontSize.value));
    colorBtn.addEventListener('change', ()=>exec('foreColor', colorBtn.value));
    highlightBtn.addEventListener('change', ()=>exec('backColor', highlightBtn.value));

    // images
    insertImage.addEventListener('click', ()=> imageLoader.click());
    imageLoader.addEventListener('change', e=>{
      const f = e.target.files[0]; if(!f) return; const reader = new FileReader(); reader.onload = ()=>{ exec('insertImage', reader.result); }; reader.readAsDataURL(f);
    });

    // table insert
    insertTable.addEventListener('click', ()=>{
      const r = parseInt(prompt('行数を入力', '2')||'2',10); const c = parseInt(prompt('列数を入力', '3')||'3',10); if(r>0 && c>0){ let html='<table border="1" style="border-collapse:collapse;width:100%">'; for(let i=0;i<r;i++){ html+='<tr>'; for(let j=0;j<c;j++){ html+='<td style="padding:6px">&nbsp;</td>'; } html+='</tr>'; } html+='</table><p></p>'; exec('insertHTML', html); } });

    // find & replace
    const findModal = document.getElementById('findModal'); const findInput = document.getElementById('findInput'); const replaceInput = document.getElementById('replaceInput'); const doFind = document.getElementById('doFind'); const doReplace = document.getElementById('doReplace'); const closeFind = document.getElementById('closeFind');
    findBtn.addEventListener('click', ()=>{ findModal.style.display='flex'; findInput.focus(); });
    closeFind.addEventListener('click', ()=> findModal.style.display='none');
    doFind.addEventListener('click', ()=>{ const q = findInput.value; if(!q) return alert('検索語を入れて下さい'); const inner = doc.innerHTML; const re = new RegExp(q,'gi'); const newHtml = inner.replace(re, match => `<mark style="background:yellow">${match}</mark>`); doc.innerHTML = newHtml; findModal.style.display='none'; pushAutoSave(); });
    doReplace.addEventListener('click', ()=>{ const q=findInput.value, r=replaceInput.value; if(!q) return alert('検索語を入れて下さい'); const inner = doc.innerHTML; const re = new RegExp(q,'gi'); doc.innerHTML = inner.replace(re, r); findModal.style.display='none'; pushAutoSave(); });

    // undo/redo via execCommand
    undoBtn.addEventListener('click', ()=>exec('undo'));
    redoBtn.addEventListener('click', ()=>exec('redo'));

    // autosave & manual save
    const AUTO_KEY = 'yui-doc-autosave-v1'; function pushAutoSave(){ localStorage.setItem(AUTO_KEY, doc.innerHTML); document.getElementById('statusSave').textContent = '最後保存: ' + new Date().toLocaleTimeString(); }
    saveBtn.addEventListener('click', ()=>{ pushAutoSave(); alert('保存しました（ブラウザのローカルストレージ）。ダウンロードするにはHTML保存を使ってください'); });
    window.addEventListener('beforeunload', ()=> pushAutoSave());
    // load
    const last = localStorage.getItem(AUTO_KEY); if(last) doc.innerHTML = last;

    // word count
    const statusCount = document.getElementById('statusCount'); const statusSel = document.getElementById('statusSel');
    function updateStats(){ const text = doc.innerText || ''; const words = text.trim().split(/\s+/).filter(Boolean).length; statusCount.textContent = '単語: ' + words; const sel = window.getSelection(); statusSel.textContent = '選択: ' + (sel ? sel.toString().length : 0); }
    doc.addEventListener('input', ()=>{ updateStats(); pushAutoSave(); }); doc.addEventListener('keyup', updateStats); doc.addEventListener('mouseup', updateStats); updateStats();

    // download HTML / doc
    downloadHtml.addEventListener('click', ()=>{ const html = `<!doctype html><html><head><meta charset="utf-8"><title>document</title></head><body>${doc.innerHTML}</body></html>`; const blob = new Blob([html],{type:'text/html'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'document.html'; a.click(); });
    downloadDoc.addEventListener('click', ()=>{ // simple .doc (Word will open HTML doc)
      const html = `<!doctype html><html><head><meta charset="utf-8"><title>document</title></head><body>${doc.innerHTML}</body></html>`; const blob = new Blob([html],{type:'application/msword'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'document.doc'; a.click(); });

    // print
    printBtn.addEventListener('click', ()=>{ const w = window.open('', '_blank'); w.document.write('<!doctype html><html><head><meta charset="utf-8"><title>print</title></head><body>' + doc.innerHTML + '</body></html>'); w.document.close(); w.focus(); setTimeout(()=> w.print(), 400); });

    // templates
    document.getElementById('tplBlank').addEventListener('click', ()=>{ if(!confirm('白紙にしますか？')) return; doc.innerHTML = '<h1>タイトル</h1><p>本文をここに入力してください。</p>'; pushAutoSave(); });
    document.getElementById('tplReport').addEventListener('click', ()=>{ doc.innerHTML = '<h1>報告書タイトル</h1><p>作成日：' + new Date().toLocaleDateString() + '</p><h2>概要</h2><p></p><h2>詳細</h2><p></p><h2>結論</h2><p></p>'; pushAutoSave(); });

    // toc generation
    document.getElementById('generateToc').addEventListener('click', ()=>{
      const headings = doc.querySelectorAll('h1,h2,h3'); if(headings.length===0){ alert('見出しがありません'); return; }
      let toc = '<div><h2>目次</h2><ul>'; headings.forEach((h,i)=>{ const id = 'h'+i; h.id = id; toc += `<li><a href="#${id}">${h.innerText}</a></li>`; }); toc += '</ul></div><hr/>'; doc.insertAdjacentHTML('afterbegin', toc); pushAutoSave(); });

    // load file (html or txt)
    document.getElementById('loadFile').addEventListener('click', ()=>{ const f = document.createElement('input'); f.type='file'; f.accept='.html,.htm,.txt'; f.onchange = e=>{ const file = e.target.files[0]; if(!file) return; const r = new FileReader(); r.onload = ()=>{ if(file.name.endsWith('.txt')) doc.textContent = r.result; else doc.innerHTML = r.result; pushAutoSave(); }; r.readAsText(file); }; f.click(); });

    // clear all
    document.getElementById('clearAll').addEventListener('click', ()=>{ if(!confirm('文書を消去しますか？')) return; doc.innerHTML = ''; pushAutoSave(); });

    // keyboard shortcuts for bold etc. also handled by browser but ensure
    window.addEventListener('keydown', e=>{
      const meta = e.ctrlKey || e.metaKey;
      if(meta && e.key.toLowerCase()==='b'){ e.preventDefault(); exec('bold'); }
      if(meta && e.key.toLowerCase()==='i'){ e.preventDefault(); exec('italic'); }
      if(meta && e.key.toLowerCase()==='s'){ e.preventDefault(); pushAutoSave(); alert('保存しました（ローカル）'); }
    });

    // line height control
    document.getElementById('lineHeight').addEventListener('input', e=>{ doc.style.lineHeight = e.target.value; pushAutoSave(); });

    // simple export to markdown (bonus)
    function toMarkdown(){ const text = doc.innerText; const blob = new Blob([text],{type:'text/markdown'}); const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'document.md'; a.click(); }

    // on load stats
    updateStats();
  </script>
</body>
</html>
