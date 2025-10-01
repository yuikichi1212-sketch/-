<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>名古屋市営地下鉄 乗換案内（超完全版）</title>
  <style>
    body { font-family: sans-serif; margin: 20px; background: #f9f9f9; }
    h1 { color: #1e90ff; }
    .container { display: flex; gap: 30px; }
    .map { flex: 2; background: #fff; border: 1px solid #ccc; padding: 10px; }
    .form { flex: 1; background: #fff; border: 1px solid #ccc; padding: 15px; }
    label { display: block; margin-top: 10px; }
    select, input, button { width: 100%; padding: 5px; margin-top: 5px; }
    .result { margin-top: 20px; background: #eef; padding: 10px; border: 1px solid #99c; }
    footer { text-align: right; font-size: 10px; margin-top: 30px; color: #666; }
    svg { width: 100%; height: auto; }
  </style>
</head>
<body>

<h1>🚇 名古屋市営地下鉄 乗換案内（超完全版）</h1>

<div class="container">
  <!-- 路線図 -->
  <div class="map">
    <h2>路線図</h2>
    <svg viewBox="0 0 600 600">
      <!-- 東山線（黄色） -->
      <line x1="50" y1="100" x2="550" y2="100" stroke="#FFD700" stroke-width="6"/>
      <text x="280" y="80">東山線</text>

      <!-- 鶴舞線（茶色） -->
      <line x1="100" y1="50" x2="100" y2="550" stroke="#8B4513" stroke-width="6"/>
      <text x="110" y="300" transform="rotate(90,110,300)">鶴舞線</text>

      <!-- 桜通線（赤） -->
      <line x1="200" y1="150" x2="500" y2="450" stroke="#FF1493" stroke-width="6"/>
      <text x="400" y="400">桜通線</text>

      <!-- 名城線（紫円環状） -->
      <circle cx="300" cy="300" r="180" stroke="#800080" stroke-width="6" fill="none"/>
      <text x="480" y="300">名城線</text>

      <!-- 名港線（水色、名城線から分岐） -->
      <line x1="400" y1="400" x2="500" y2="550" stroke="#1E90FF" stroke-width="6"/>
      <text x="490" y="530">名港線</text>

      <!-- 上飯田線（緑） -->
      <line x1="300" y1="120" x2="300" y2="50" stroke="#228B22" stroke-width="6"/>
      <text x="310" y="70">上飯田線</text>

      <!-- 駅の例 -->
      <circle cx="300" cy="100" r="5" fill="#000"/>
      <text x="310" y="95">栄</text>

      <circle cx="100" cy="100" r="5" fill="#000"/>
      <text x="70" y="95">名古屋</text>
    </svg>
  </div>

  <!-- 入力フォーム -->
  <div class="form">
    <h2>経路検索</h2>
    <label>出発駅
      <select id="origin">
        <option>名古屋</option>
        <option>栄</option>
        <option>金山</option>
        <option>大曽根</option>
      </select>
    </label>

    <label>到着駅
      <select id="destination">
        <option>栄</option>
        <option>名古屋</option>
        <option>八事</option>
        <option>名古屋港</option>
      </select>
    </label>

    <label>出発時刻
      <input type="time" id="time" value="08:00">
    </label>

    <button onclick="searchRoute()">検索</button>

    <div class="result" id="result">
      経路がここに表示されます。
    </div>
  </div>
</div>

<footer>作者：ゆいきち</footer>

<script>
function searchRoute(){
  let o = document.getElementById("origin").value;
  let d = document.getElementById("destination").value;
  let t = document.getElementById("time").value;
  document.getElementById("result").innerHTML =
    "🔍 " + o + " → " + d + " を " + t + " に出発<br>（※サンプル表示です。実際の時刻表ロジック未搭載）";
}
</script>

</body>
</html>
