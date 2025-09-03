<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ゆいきちAIチャット（オフライン版）</title>
<style>
  body { font-family: Arial, sans-serif; background:#f0f0f0; margin:0; padding:0; }
  #chatContainer { max-width:600px; margin:50px auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
  #messages { max-height:400px; overflow-y:auto; margin-bottom:20px; }
  .message { margin:10px 0; }
  .user { text-align:right; color:blue; }
  .ai { text-align:left; color:green; }
  .video { margin:5px 0; padding:5px; border:1px solid #ccc; border-radius:5px; background:#fafafa; }
</style>
</head>
<body>
<div id="chatContainer">
  <div id="messages"></div>
  <input type="text" id="userInput" placeholder="メッセージを入力..." style="width:80%;">
  <button onclick="sendMessage()">送信</button>
</div>

<script>
// サンプル動画リスト
const videoList = [
  {title:"ゆいきちの面白動画A", tags:["面白い","おすすめ"], url:"#"},
  {title:"ゆいきちの神回B", tags:["神回","おすすめ"], url:"#"},
  {title:"ゆいきちのゲーム実況C", tags:["ゲーム","実況"], url:"#"},
  {title:"ゆいきちのVlogD", tags:["日常","Vlog"], url:"#"},
  {title:"ゆいきちの挑戦動画E", tags:["挑戦","チャレンジ"], url:"#"}
];

const messagesDiv = document.getElementById('messages');
const userInput = document.getElementById('userInput');

function sendMessage() {
  const text = userInput.value.trim();
  if(!text) return;
  appendMessage(text, 'user');
  userInput.value = '';

  // 簡易ルールベースでAI返答を作る
  let aiText = "ゆいきちAIです！おすすめの動画はこちら:";
  const recommended = recommendVideos(text);
  appendMessage(aiText, 'ai');
  recommended.forEach(v => appendVideo(v.title, v.url));
}

function recommendVideos(inputText) {
  // 入力に含まれるキーワードで動画をフィルター
  const keywords = inputText.toLowerCase().split(/\s+/);
  let results = [];

  keywords.forEach(k => {
    videoList.forEach(v => {
      if(v.tags.some(tag => tag.toLowerCase().includes(k)) && !results.includes(v)) {
        results.push(v);
      }
    });
  });

  // もしキーワード一致がなければ最初の3本を返す
  if(results.length === 0) results = videoList.slice(0,3);
  return results;
}

function appendMessage(text, type){
  const div = document.createElement('div');
  div.className = `message ${type}`;
  div.innerText = text;
  messagesDiv.appendChild(div);
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}

function appendVideo(title, url){
  const div = document.createElement('div');
  div.className = 'video';
  div.innerHTML = `<a href="${url}" target="_blank">${title}</a>`;
  messagesDiv.appendChild(div);
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}
</script>
</body>
</html>
