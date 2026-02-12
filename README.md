<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>メモ</title>
    <style>
        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: #f5f5f7;
            display: flex;
            justify-content: center;
            padding: 20px;
        }
        .container {
            width: 100%;
            max-width: 600px;
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h2 {
            color: #333;
            margin-top: 0;
            border-bottom: 2px solid #007aff;
            padding-bottom: 10px;
        }
        textarea {
            width: 100%;
            height: 300px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            font-size: 16px;
            box-sizing: border-box;
            resize: vertical;
            outline: none;
            transition: border-color 0.3s;
        }
        textarea:focus {
            border-color: #007aff;
        }
        .controls {
            margin-top: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        button {
            background-color: #ff3b30;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover {
            background-color: #d32f2f;
        }
        .status {
            font-size: 12px;
            color: #888;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Memo</h2>
    <textarea id="memoArea" placeholder="ここにメモを書いてください..."></textarea>
    
    <div class="controls">
        <span class="status" id="statusMsg">自動保存中...</span>
        <button onclick="clearMemo()">内容を全消去</button>
    </div>
</div>

<script>
    const memoArea = document.getElementById('memoArea');
    const statusMsg = document.getElementById('statusMsg');

    // 1. ページ読み込み時に保存された内容を出す
    window.onload = () => {
        const savedMemo = localStorage.getItem('myMemo');
        if (savedMemo) {
            memoArea.value = savedMemo;
        }
    };

    // 2. 入力されるたびに保存する
    memoArea.addEventListener('input', () => {
        localStorage.setItem('myMemo', memoArea.value);
        statusMsg.innerText = "保存されました ";
        
        // 2秒後にメッセージを戻す
        setTimeout(() => {
            statusMsg.innerText = "自動保存中...";
        }, 2000);
    });

    // 3. 全消去機能
    function clearMemo() {
        if (confirm('本当に全てのメモを消去しますか？')) {
            memoArea.value = '';
            localStorage.removeItem('myMemo');
            statusMsg.innerText = "消去しました";
        }
    }
</script>

</body>
</html>
