<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Web Memo</title>
    <style>
        :root {
            --bg-color: #1e1e1e;
            --sidebar-bg: #252526;
            --text-color: #d4d4d4;
            --accent-color: #007acc;
            --border-color: #3c3c3c;
        }
        body {
            margin: 0;
            font-family: 'Segoe UI', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            height: 100vh;
        }
        /* サイドバー */
        #sidebar {
            width: 250px;
            background-color: var(--sidebar-bg);
            border-right: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
        }
        .sidebar-header {
            padding: 20px;
            font-weight: bold;
            border-bottom: 1px solid var(--border-color);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        #memo-list {
            flex-grow: 1;
            overflow-y: auto;
            list-style: none;
            padding: 0;
            margin: 0;
        }
        .memo-item {
            padding: 15px 20px;
            cursor: pointer;
            border-bottom: 1px solid var(--border-color);
            font-size: 14px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
        .memo-item:hover { background-color: #2a2d2e; }
        .memo-item.active { background-color: #37373d; border-left: 4px solid var(--accent-color); }

        /* メインエリア */
        #main {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            padding: 20px;
        }
        #memo-title {
            background: transparent;
            border: none;
            color: white;
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 15px;
            outline: none;
        }
        #editor {
            flex-grow: 1;
            background: transparent;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            color: var(--text-color);
            padding: 15px;
            font-size: 16px;
            resize: none;
            outline: none;
            line-height: 1.6;
        }
        .toolbar {
            margin-top: 15px;
            display: flex;
            gap: 10px;
            align-items: center;
        }
        button {
            background: var(--accent-color);
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 13px;
        }
        button.delete { background: #a31515; }
        button:hover { opacity: 0.8; }
        .info { font-size: 12px; color: #888; margin-left: auto; }
    </style>
</head>
<body>

<div id="sidebar">
    <div class="sidebar-header">
        🗒 My Memos
        <button onclick="createNewMemo()">＋</button>
    </div>
    <ul id="memo-list"></ul>
</div>

<div id="main">
    <input type="text" id="memo-title" placeholder="タイトル未設定">
    <textarea id="editor" placeholder="内容を入力してください..."></textarea>
    <div class="toolbar">
        <button onclick="downloadMemo()">⬇ ダウンロード</button>
        <button class="delete" onclick="deleteMemo()">🗑 削除</button>
        <div class="info" id="char-count">0 文字</div>
    </div>
</div>

<script>
    let memos = JSON.parse(localStorage.getItem('web_memos')) || [
        { id: Date.now(), title: '最初のメモ', content: 'ここに内容を書けます。' }
    ];
    let currentMemoId = memos[0].id;

    const listEl = document.getElementById('memo-list');
    const editorEl = document.getElementById('editor');
    const titleEl = document.getElementById('memo-title');
    const charCountEl = document.getElementById('char-count');

    function init() {
        renderList();
        loadMemo(currentMemoId);
    }

    function renderList() {
        listEl.innerHTML = '';
        memos.forEach(memo => {
            const li = document.createElement('li');
            li.className = `memo-item ${memo.id === currentMemoId ? 'active' : ''}`;
            li.innerText = memo.title || '無題のメモ';
            li.onclick = () => loadMemo(memo.id);
            listEl.appendChild(li);
        });
    }

    function loadMemo(id) {
        currentMemoId = id;
        const memo = memos.find(m => m.id === id);
        titleEl.value = memo.title;
        editorEl.value = memo.content;
        updateCharCount();
        renderList();
    }

    function save() {
        const memo = memos.find(m => m.id === currentMemoId);
        if (memo) {
            memo.title = titleEl.value;
            memo.content = editorEl.value;
            localStorage.setItem('web_memos', JSON.stringify(memos));
            renderList();
            updateCharCount();
        }
    }

    function createNewMemo() {
        const newMemo = { id: Date.now(), title: '新しいメモ', content: '' };
        memos.unshift(newMemo);
        loadMemo(newMemo.id);
    }

    function deleteMemo() {
        if (memos.length <= 1) return alert('最後のメモは削除できません');
        if (confirm('このメモを削除しますか？')) {
            memos = memos.filter(m => m.id !== currentMemoId);
            currentMemoId = memos[0].id;
            save();
            loadMemo(currentMemoId);
        }
    }

    function downloadMemo() {
        const blob = new Blob([editorEl.value], { type: 'text/plain' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `${titleEl.value || 'memo'}.txt`;
        a.click();
    }

    function updateCharCount() {
        charCountEl.innerText = `${editorEl.value.length} 文字`;
    }

    titleEl.addEventListener('input', save);
    editorEl.addEventListener('input', save);

    init();
</script>
</body>
</html>
