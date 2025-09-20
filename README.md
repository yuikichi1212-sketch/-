<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Minecraft風ゲーム</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #info {
      position: absolute;
      top: 10px; left: 10px;
      color: white; font-family: sans-serif;
      background: rgba(0,0,0,0.5);
      padding: 5px 10px; border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="info">
    クリックで視点ロック<br>
    WASD: 移動 / マウス: 視点<br>
    左クリック: 壊す / 右クリック: 置く
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/examples/js/controls/PointerLockControls.js"></script>
  <script>
    let scene, camera, renderer, controls, raycaster;
    let blocks = {};
    const blockSize = 1;

    init();
    animate();

    function init() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x87ceeb);

      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(8, 5, 8);

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      // 光源
      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(10, 20, 10);
      scene.add(light);
      scene.add(new THREE.AmbientLight(0xffffff, 0.4));

      // コントロール（FPS視点）
      controls = new THREE.PointerLockControls(camera, document.body);
      document.body.addEventListener("click", () => controls.lock());
      scene.add(controls.getObject());

      raycaster = new THREE.Raycaster();

      // 地面
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

      // イベント
      window.addEventListener("resize", onWindowResize);
      document.addEventListener("mousedown", onMouseDown);
      document.addEventListener("contextmenu", e => e.preventDefault());

      // 移動制御
      document.addEventListener("keydown", onKeyDown);
      document.addEventListener("keyup", onKeyUp);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    // --- ブロック操作 ---
    function onMouseDown(event) {
      if (!controls.isLocked) return;
      raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
      const intersects = raycaster.intersectObjects(Object.values(blocks));
      if (intersects.length > 0) {
        const target = intersects[0];
        const pos = target.object.position.clone();

        if (event.button === 0) {
          // 左クリック: 壊す
          scene.remove(target.object);
          delete blocks[`${pos.x},${pos.y},${pos.z}`];
        } else if (event.button === 2) {
          // 右クリック: 置く
          const normal = target.face.normal;
          const newPos = pos.clone().add(normal);
          const key = `${newPos.x},${newPos.y},${newPos.z}`;
          if (!blocks[key]) {
            const mat = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
            const geo = new THREE.BoxGeometry(blockSize, blockSize, blockSize);
            const cube = new THREE.Mesh(geo, mat);
            cube.position.copy(newPos);
            scene.add(cube);
            blocks[key] = cube;
          }
        }
      }
    }

    // --- 移動制御 ---
    const keys = {};
    function onKeyDown(e) { keys[e.code] = true; }
    function onKeyUp(e) { keys[e.code] = false; }

    function movePlayer() {
      if (!controls.isLocked) return;
      const speed = 0.1;
      const dir = new THREE.Vector3();
      if (keys["KeyW"]) dir.z -= 1;
      if (keys["KeyS"]) dir.z += 1;
      if (keys["KeyA"]) dir.x -= 1;
      if (keys["KeyD"]) dir.x += 1;
      dir.normalize();
      controls.moveRight(dir.x * speed);
      controls.moveForward(dir.z * speed);
    }

    // --- アニメーション ---
    function animate() {
      requestAnimationFrame(animate);
      movePlayer();
      renderer.render(scene, camera);
    }
  </script>
</body>
</html>
