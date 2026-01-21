<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ドキュメントアプリ</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
        }
        #app {
            background: white;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        h1 {
            text-align: center;
        }
        textarea {
            width: 100%;
            height: 300px;
            margin: 10px 0;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 16px;
            resize: none;
        }
        button {
            padding: 10px 20px;
            margin-right: 10px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        .save-btn {
            background-color: #4CAF50;
            color: white;
        }
        .clear-btn {
            background-color: #f44336;
            color: white;
        }
    </style>
</head>
<body>
    <div id="app">
        <h1>ドキュメントアプリ</h1>
        <textarea id="document" placeholder="ここに文書を入力してください..."></textarea>
        <br>
        <button class="save-btn" onclick="saveDocument()">保存</button>
        <button class="clear-btn" onclick="clearDocument()">クリア</button>
    </div>

    <script>
        function saveDocument() {
            const content = document.getElementById('document').value;
            const blob = new Blob([content], { type: 'text/plain' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'document.txt';
            a.click();
            URL.revokeObjectURL(url);
        }

        function clearDocument() {
            document.getElementById('document').value = '';
        }
    </script>
</body>
</html>
