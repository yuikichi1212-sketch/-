<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Minecraft風ゲーム - ゆいきち版</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #info {
      position: absolute;
      top: 10px; left: 10px;
      color: white;
      font-family: sans-serif;
      z-index: 1;
      background: rgba(0,0,0,0.5);
      padding: 5px 10px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="info">
    クリックで視点ロック<br>
    WASD: 移動<br>
    左クリック: 壊す / 右クリック: 置く
  </div>

  <!-- Three.js & Controls 読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.150.1/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.150.1/examples/js/controls/PointerLockControls.js"></script>

  <script>
    let scene, camera, renderer, controls, raycaster;
    let blocks = {};
    const blockSize = 1;
    const pointer = new THREE.Vector2(0, 0);

    init();
    animate();

    function init() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x87ceeb); // 空色

      // カメラ
      camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
      camera.position.set(8, 5, 8);

      // レンダラー
      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      // プレイヤー操作
      controls = new THREE.PointerLockControls(camera, renderer.domElement);
      document.body.addEventListener('click', () => { controls.lock(); });
      scene.add(controls.getObject());

      // ライト
      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(10, 20, 10);
      scene.add(light);
      scene.add(new THREE.AmbientLight(0xffffff, 0.4));

      // 地面生成
      const groundGeo = new THREE.BoxGeometry(blockSize, blockSize, blockSize);
      const groundMat = new THREE.MeshStandardMaterial({ color: 0x228B22 });
      for (let x = 0; x < 16; x++) {
        for (let z = 0; z < 16; z++) {
          const cube = new THREE.Mesh(groundGeo, groundMat);
          cube.position.set(x, 0, z);
          scene.add(cube);
          blocks[`${x},0,${z}`] = cube;
        }
      }

      // レイキャスト
      raycaster = new THREE.Raycaster();

      // イベント
      window.addEventListener('resize', onWindowResize);
      document.addEventListener('mousedown', onMouseDown);
      document.addEventListener('contextmenu', e => e.preventDefault()); // 右クリックメニュー無効化
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function onMouseDown(event) {
      if (!controls.isLocked) return;

      raycaster.setFromCamera(pointer, camera);
      const intersects = raycaster.intersectObjects(Object.values(blocks));

      if (intersects.length > 0) {
        const target = intersects[0];
        const pos = target.object.position.clone();

        if (event.button === 0) {
          // 左クリック → 壊す
          scene.remove(target.object);
          delete blocks[`${pos.x},${pos.y},${pos.z}`];
        } else if (event.button === 2) {
          // 右クリック → 設置
          const normal = target.face.normal;
          const newPos = pos.clone().add(normal);
          const key = `${newPos.x},${newPos.y},${newPos.z}`;

          if (!blocks[key]) {
            const cube = new THREE.Mesh(
              new THREE.BoxGeometry(blockSize, blockSize, blockSize),
              new THREE.MeshStandardMaterial({ color: 0x8B4513 })
            );
            cube.position.copy(newPos);
            scene.add(cube);
            blocks[key] = cube;
          }
        }
      }
    }

    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }
  </script>
</body>
</html>
