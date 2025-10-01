<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>🚇 名古屋市営地下鉄 乗換案内（超完全版）</title>
  <style>
    body {
      font-family: Arial, "Hiragino Kaku Gothic ProN", "メイリオ", sans-serif;
      margin: 20px;
      background: #f9f9f9;
    }
    h1 {
      color: #0066cc;
    }
    .form-box {
      background: #fff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      max-width: 500px;
    }
    label {
      font-weight: bold;
    }
    select, button {
      width: 100%;
      padding: 8px;
      margin: 10px 0;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    button {
      background: #0066cc;
      color: white;
      font-size: 16px;
      cursor: pointer;
    }
    button:hover {
      background: #004999;
    }
    .map {
      margin-top: 30px;
      text-align: center;
    }
    .map img {
      max-width: 90%;
      border: 1px solid #ccc;
      border-radius: 10px;
    }
    footer {
      text-align: right;
      margin-top: 20px;
      font-size: 10px;
      color: #555;
    }
  </style>
</head>
<body>

  <h1>🚇 名古屋市営地下鉄 乗換案内（超完全版）</h1>

  <div class="form-box">
    <form>
      <label for="origin">出発駅:</label>
      <select id="origin" name="origin">
        <option value="">駅を選択</option>
        <!-- 東山線 -->
        <optgroup label="東山線">
          <option>高畑</option><option>八田</option><option>岩塚</option><option>中村公園</option>
          <option>中村日赤</option><option>本陣</option><option>亀島</option><option>名古屋</option>
          <option>伏見</option><option>栄</option><option>新栄町</option><option>千種</option>
          <option>今池</option><option>池下</option><option>覚王山</option><option>本山</option>
          <option>東山公園</option><option>星ヶ丘</option><option>一社</option><option>上社</option>
          <option>本郷</option><option>藤が丘</option>
        </optgroup>

        <!-- 名城線 -->
        <optgroup label="名城線">
          <option>大曽根</option><option>ナゴヤドーム前矢田</option><option>砂田橋</option><option>茶屋ヶ坂</option>
          <option>自由ヶ丘</option><option>本山</option><option>名古屋大学</option><option>八事日赤</option>
          <option>八事</option><option>総合リハビリセンター</option><option>瑞穂運動場東</option>
          <option>新瑞橋</option><option>妙音通</option><option>堀田</option><option>伝馬町</option>
          <option>熱田神宮伝馬町</option><option>神宮西</option><option>西高蔵</option><option>金山</option>
          <option>東別院</option><option>上前津</option><option>矢場町</option><option>栄</option>
          <option>久屋大通</option><option>市役所</option><option>名城公園</option><option>黒川</option>
          <option>志賀本通</option><option>平安通</option><option>大曽根</option>
        </optgroup>

        <!-- 名港線 -->
        <optgroup label="名港線">
          <option>金山</option><option>日比野</option><option>六番町</option><option>東海通</option>
          <option>港区役所</option><option>築地口</option><option>名古屋港</option>
        </optgroup>

        <!-- 鶴舞線 -->
        <optgroup label="鶴舞線">
          <option>上小田井</option><option>庄内緑地公園</option><option>庄内通</option><option>浄心</option>
          <option>浅間町</option><option>丸の内</option><option>伏見</option><option>大須観音</option>
          <option>上前津</option><option>鶴舞</option><option>荒畑</option><option>御器所</option>
          <option>川名</option><option>いりなか</option><option>八事</option><option>塩釜口</option>
          <option>植田</option><option>原</option><option>平針</option><option>赤池</option>
        </optgroup>

        <!-- 桜通線 -->
        <optgroup label="桜通線">
          <option>中村区役所</option><option>名古屋</option><option>国際センター</option><option>丸の内</option>
          <option>久屋大通</option><option>高岳</option><option>車道</option><option>今池</option>
          <option>吹上</option><option>御器所</option><option>桜山</option><option>瑞穂区役所</option>
          <option>瑞穂運動場西</option><option>新瑞橋</option><option>桜本町</option><option>鶴里</option>
          <option>野並</option><option>鳴子北</option><option>相生山</option><option>神沢</option>
          <option>徳重</option>
        </optgroup>

        <!-- 上飯田線 -->
        <optgroup label="上飯田線">
          <option>上飯田</option><option>平安通</option>
        </optgroup>
      </select>

      <label for="destination">到着駅:</label>
      <select id="destination" name="destination">
        <!-- 上と同じ駅リストをコピー -->
        <!-- 実運用ではJSで共通リスト化するのがスマート -->
      </select>

      <button type="submit">検索</button>
    </form>
  </div>

  <div class="map">
    <h2>路線図</h2>
    <img src="https://upload.wikimedia.org/wikipedia/commons/5/56/Nagoya_subway_map_ja.png" alt="名古屋市営地下鉄路線図">
  </div>

  <footer>
    作者：ゆいきち
  </footer>

</body>
</html>
