<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memo</title>
    <style>
        :root {
            --glass-bg: rgba(255, 255, 255, 0.35);
            --glass-border: rgba(255, 255, 255, 0.6);
            --glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.1);
            --text-main: #2d3748;
            --text-sub: #718096;
            --accent: #667eea;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            margin: 0;
            min-height: 100vh;
            background: linear-gradient(135deg, #fdfbfb 0%, #ebedee 100%);
            display: flex;
            justify-content: center;
            padding: 40px 20px;
            box-sizing: border-box;
            position: relative;
            overflow-x: hidden;
        }

        /* 背景のオーブ（飾り） */
        .orb {
            position: fixed;
            border-radius: 50%;
            filter: blur(60px);
            z-index: -1;
            animation: float 10s infinite ease-in-out;
        }
        .orb-1 { width: 400px; height: 400px; background: #a1c4fd; top: -100px; left: -100px; }
        .orb-2 { width: 300px; height: 300px; background: #c2e9fb; bottom: -50px; right: -50px; animation-delay: -5s; }
        .orb-3 { width: 200px; height: 200px; background: #ffecd2; top: 40%; left: 60%; animation-duration: 15s; }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-20px); }
        }

        /* --- メインコンテナ --- */
        .container {
            width: 100%;
            max-width: 900px;
            display: flex;
            flex-direction: column;
            gap: 30px;
        }

        /* --- 入力エリア --- */
        .input-box {
            background: var(--glass-bg);
            backdrop-filter: blur(12px);
            border: 1px solid var(--glass-border);
            border-radius: 20px;
            padding: 20px;
            box-shadow: var(--glass-shadow);
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        input, textarea {
            border: none;
            background: rgba(255, 255, 255, 0.5);
            padding: 12px 15px;
            border-radius: 12px;
            font-size: 16px;
            color: var(--text-main);
            outline: none;
            font-family: inherit;
            transition: 0.2s;
        }
        input::placeholder, textarea::placeholder { color: #a0aec0; }
        input:focus, textarea:focus { background: rgba(255, 255, 255, 0.8); box-shadow: 0 0 0 2px var(--accent); }
        
        textarea { resize: vertical; min-height: 60px; }

        .btn-add {
            align-self: flex-end;
            background: var(--accent);
            color: white;
            border: none;
            padding: 10px 24px;
            border-radius: 12px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
        }
        .btn-add:hover { transform: translateY(-2px); box-shadow: 0 6px 16px rgba(102, 126, 234, 0.4); }

        /* --- メモグリッド --- */
        .memo-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
        }

        .memo-card {
            background: rgba(255, 255, 255, 0.45);
            backdrop-filter: blur(8px);
            border: 1px solid var(--glass-border);
            border-radius: 16px;
            padding: 20px;
            box-shadow: var(--glass-shadow);
            transition: all 0.3s ease;
            position: relative;
            display: flex;
            flex-direction: column;
            animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popIn { from { opacity: 0; transform: scale(0.9); } to { opacity: 1; transform: scale(1); } }

        .memo-card:hover {
            transform: translateY(-5px);
            background: rgba(255, 255, 255, 0.65);
            box-shadow: 0 12px 40px 0 rgba(31, 38, 135, 0.15);
        }

        .memo-date { font-size: 11px; color: var(--text-sub); margin-bottom: 5px; }
        .memo-title { font-size: 18px; font-weight: bold; color: var(--text-main); margin: 0 0 8px 0; }
        .memo-body { font-size: 14px; color: #4a5568; line-height: 1.6; white-space: pre-wrap; flex-grow: 1; }

        .memo-actions {
            margin-top: 15px;
            display: flex;
            justify-content: flex-end;
            gap: 10px;
            opacity: 0.6;
            transition: 0.2s;
        }
        .memo-card:hover .memo-actions { opacity: 1; }

        .icon-btn {
            background: none;
            border: none;
            cursor: pointer;
            font-size: 16px;
            padding: 5px;
            border-radius: 5px;
            transition: 0.2s;
        }
        .icon-btn:hover { background: rgba(0,0,0,0.05); }
        .delete-btn { color: #fc8181; }
        .delete-btn:hover { background: #fff5f5; color: #e53e3e; }
        .copy-btn { color: #718096; }

    </style>
</head>
<body>

<div class="orb orb-1"></div>
<div class="orb orb-2"></div>
<div class="orb orb-3"></div>

<div class="container">
    
    <div class="input-box">
        <input type="text" id="titleInput" placeholder="タイトル (例: 買い物リスト)">
        <textarea id="bodyInput" placeholder="メモしたい内容を入力..."></textarea>
        <button class="btn-add" onclick="addMemo()">＋ メモを追加</button>
    </div>

    <div class="memo-grid" id="memoGrid">
        </div>

</div>

<script>
    // 起動時にロード
    document.addEventListener('DOMContentLoaded', loadMemos);

    function addMemo() {
        const titleIn = document.getElementById('titleInput');
        const bodyIn = document.getElementById('bodyInput');

        const title = titleIn.value.trim();
        const body = bodyIn.value.trim();

        if (!title && !body) return alert("何か書いてください！");

        const memo = {
            id: Date.now(),
            title: title || "無題のメモ",
            body: body,
            date: new Date().toLocaleString('ja-JP')
        };

        saveToStorage(memo);
        renderMemo(memo); // 画面に追加

        // 入力をクリア
        titleIn.value = "";
        bodyIn.value = "";
        titleIn.focus();
    }

    function renderMemo(memo) {
        const grid = document.getElementById('memoGrid');
        
        const card = document.createElement('div');
        card.className = 'memo-card';
        card.id = `memo-${memo.id}`;
        card.innerHTML = `
            <div class="memo-date">${memo.date}</div>
            <h3 class="memo-title">${memo.title}</h3>
            <div class="memo-body">${memo.body}</div>
            <div class="memo-actions">
                <button class="icon-btn copy-btn" onclick="copyMemo('${memo.body.replace(/\n/g, '\\n')}')" title="コピー">📋</button>
                <button class="icon-btn delete-btn" onclick="deleteMemo(${memo.id})" title="削除">🗑</button>
            </div>
        `;

        grid.prepend(card); // 新しいものを先頭に
    }

    function deleteMemo(id) {
        if (!confirm("このメモを削除しますか？")) return;

        // アニメーション付きで消す
        const card = document.getElementById(`memo-${id}`);
        card.style.transform = "scale(0.8)";
        card.style.opacity = "0";
        
        setTimeout(() => {
            card.remove();
            removeFromStorage(id);
        }, 300);
    }

    function copyMemo(text) {
        navigator.clipboard.writeText(text).then(() => {
            alert("クリップボードにコピーしました！");
        });
    }

    // --- LocalStorage 管理 ---

    function saveToStorage(memo) {
        const memos = JSON.parse(localStorage.getItem('glass_memos') || "[]");
        memos.push(memo);
        localStorage.setItem('glass_memos', JSON.stringify(memos));
    }

    function removeFromStorage(id) {
        let memos = JSON.parse(localStorage.getItem('glass_memos') || "[]");
        memos = memos.filter(m => m.id !== id);
        localStorage.setItem('glass_memos', JSON.stringify(memos));
    }

    function loadMemos() {
        const memos = JSON.parse(localStorage.getItem('glass_memos') || "[]");
        // 最新順にするために逆順でループ
        // ※ saveToStorageでpush(末尾追加)しているので、表示時は後ろから取り出すと新しい順になる
        for (let i = memos.length - 1; i >= 0; i--) {
            renderMemo(memos[i]);
        }
    }
</script>

</body>
</html>
