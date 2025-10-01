<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ゆいきちナビ 超完全版</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css">
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.min.js"></script>
  <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.css">
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
    }
    #map {
      width: 100%;
      height: 100%;
    }
    #controls {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(255,255,255,0.9);
      padding: 8px;
      border-radius: 8px;
      z-index: 9999;
    }
    #info {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0,0,0,0.6);
      color: white;
      padding: 6px;
      border-radius: 6px;
      font-size: 14px;
      z-index: 9999;
    }
  </style>
</head>
<body>
  <div id="controls">
    <input type="text" id="destination" placeholder="目的地を入力">
    <button id="searchBtn">ナビ開始</button>
    <button id="stopBtn">ナビ停止</button>
  </div>
  <div id="map"></div>
  <div id="info">準備中...</div>

  <script>
    let map, routingControl, userMarker;
    let compassHeading = 0;
    let navigating = false;
    let turnMarkers = [];
    let pointerMarker = null;

    // 地図の初期化
    map = L.map("map", {zoomControl: true}).setView([35.1709, 136.8815], 16);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      maxZoom: 19,
      attribution: "&copy; OpenStreetMap contributors"
    }).addTo(map);

    // 現在地を取得
    navigator.geolocation.getCurrentPosition((pos) => {
      const latlng = [pos.coords.latitude, pos.coords.longitude];
      map.setView(latlng, 16);

      userMarker = L.marker(latlng).addTo(map).bindPopup("現在地").openPopup();
    }, (err) => {
      console.error("現在地取得失敗:", err);
    }, {enableHighAccuracy: true});
        // ==========================================================
    // Part 2: ルート検索・ナビ機能
    // ==========================================================

    const searchBtn = document.getElementById("searchBtn");
    const stopBtn = document.getElementById("stopBtn");
    const infoBox = document.getElementById("info");

    // 音声読み上げ（簡易）
    function speak(text) {
      if ('speechSynthesis' in window) {
        const utter = new SpeechSynthesisUtterance(text);
        utter.lang = "ja-JP";
        speechSynthesis.speak(utter);
      }
    }

    // ナビ開始
    searchBtn.addEventListener("click", async () => {
      if (!userMarker) {
        alert("現在地が取得できませんでした。");
        return;
      }
      const destination = document.getElementById("destination").value;
      if (!destination) {
        alert("目的地を入力してください。");
        return;
      }

      // 目的地を座標に変換（Nominatim API）
      const res = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(destination)}`);
      const data = await res.json();
      if (data.length === 0) {
        alert("目的地が見つかりません。");
        return;
      }
      const destLat = parseFloat(data[0].lat);
      const destLon = parseFloat(data[0].lon);

      // ルート表示
      if (routingControl) {
        map.removeControl(routingControl);
      }
      routingControl = L.Routing.control({
        waypoints: [
          userMarker.getLatLng(),
          L.latLng(destLat, destLon)
        ],
        lineOptions: {
          styles: [{color: "blue", weight: 6}]
        },
        createMarker: () => null,
        addWaypoints: false,
        routeWhileDragging: false
      }).addTo(map);

      navigating = true;
      infoBox.innerText = "ナビ開始: " + destination;
      speak("目的地までのルートを開始します");
    });

    // ナビ停止
    stopBtn.addEventListener("click", () => {
      if (routingControl) {
        map.removeControl(routingControl);
        routingControl = null;
      }
      navigating = false;
      infoBox.innerText = "ナビ停止中";
      speak("ナビを停止しました");
    });
    // ==========================================================
    // Part 3: 曲がり角マーカーと音声案内
    // ==========================================================

    function clearTurnMarkers() {
      turnMarkers.forEach(m => map.removeLayer(m));
      turnMarkers = [];
    }

    // ルート完了時にステップ情報を取得
    map.on("routeselected", (e) => {
      clearTurnMarkers();

      const route = e.route;
      if (!route || !route.instructions) return;

      route.instructions.forEach((inst, i) => {
        const latlng = inst.latLng;
        const text = inst.text;

        // マーカー作成
        const marker = L.marker(latlng, {
          title: text
        }).addTo(map);

        marker.bindPopup(text);
        turnMarkers.push(marker);

        // 音声案内
        setTimeout(() => {
          speak(text);
          infoBox.innerText = text;
        }, i * 4000); // 順番に読み上げる（仮）
      });
    });

    // ナビ停止時にマーカー削除
    stopBtn.addEventListener("click", () => {
      clearTurnMarkers();
    });
    // ==========================================================
    // Part 4: コンパス・地図回転・ポインター
    // ==========================================================

    // ポインターアイコン
    const pointerIcon = L.divIcon({
      className: "pointer-icon",
      html: "▲",
      iconSize: [30, 30]
    });

    // ポインターを現在地に設置
    function updatePointer(lat, lon, heading) {
      if (!pointerMarker) {
        pointerMarker = L.marker([lat, lon], {icon: pointerIcon}).addTo(map);
      } else {
        pointerMarker.setLatLng([lat, lon]);
      }

      // 方角を適用
      if (pointerMarker._icon) {
        pointerMarker._icon.style.transform =
          `rotate(${heading}deg) translate(-50%, -50%)`;
      }
    }

    // 現在地を常に真ん中に
    function keepUserCentered(lat, lon) {
      if (navigating) {
        map.setView([lat, lon], map.getZoom(), {animate: false});
      }
    }

    // コンパスイベント（デバイスの向き）
    window.addEventListener("deviceorientationabsolute", (event) => {
      if (event.alpha !== null) {
        compassHeading = 360 - event.alpha; // 方角
      }
    }, true);

    // 位置情報の監視（更新時に地図回転＆ポインター更新）
    navigator.geolocation.watchPosition((pos) => {
      const lat = pos.coords.latitude;
      const lon = pos.coords.longitude;

      // 現在地マーカー更新
      if (!userMarker) {
        userMarker = L.marker([lat, lon]).addTo(map).bindPopup("現在地");
      } else {
        userMarker.setLatLng([lat, lon]);
      }

      // ポインター更新
      updatePointer(lat, lon, compassHeading);

      // 地図回転（ナビ中のみ）
      if (navigating) {
        map.setBearing?.(compassHeading); // Leafletに方位回転プラグインを使う場合
      }

      keepUserCentered(lat, lon);
    }, (err) => {
      console.error("現在地更新エラー:", err);
    }, {enableHighAccuracy: true});

    // スタイル調整
    const style = document.createElement("style");
    style.innerHTML = `
      .pointer-icon {
        font-size: 28px;
        color: red;
        text-shadow: 0 0 3px white;
      }
    `;
    document.head.appendChild(style);
  </script>
</body>
</html>
