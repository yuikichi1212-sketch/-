<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Minecraft風ブラウザゲーム</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #info {
      position: absolute; top: 10px; left: 10px;
      color: white; font-family: sans-serif; z-index: 1;
    }
  </style>
</head>
<body>
  <div id="info">クリックで視点ロック<br>WASD移動<br>左クリック: 壊す / 右クリック: 置く</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.150.1/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.150.1/examples/js/controls/PointerLockControls.js"></script>

  <script>
    let scene, camera, renderer, raycaster, controls;
    let blocks = {};
    const blockSize = 1;
    const pointer = new THREE.Vector2();

    init();
    animate();

    function init() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x87ceeb); // 空

      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(8, 15, 25);   // ← 高めの位置
      camera.lookAt(8, 0, 8);           // ← 地面の中央を見る

      renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      // 視点操作
      controls = new THREE.PointerLockControls(camera, document.body);
      document.body.addEventListener('click', () => controls.lock());
      scene.add(controls.getObject());

      raycaster = new THREE.Raycaster();

      // 光源
      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(30, 50, 30);
      scene.add(light);
      scene.add(new THREE.AmbientLight(0xffffff, 0.5));

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
      window.addEventListener('resize', onWindowResize);
      document.addEventListener('mousedown', onMouseDown);
      document.addEventListener('contextmenu', e => e.preventDefault());

      // 移動処理
      const keys = {};
      document.addEventListener('keydown', e => keys[e.code] = true);
      document.addEventListener('keyup', e => keys[e.code] = false);

      function movePlayer() {
        const speed = 0.2;
        if (controls.isLocked) {
          if (keys["KeyW"]) controls.moveForward(speed);
          if (keys["KeyS"]) controls.moveForward(-speed);
          if (keys["KeyA"]) controls.moveRight(-speed);
          if (keys["KeyD"]) controls.moveRight(speed);
        }
        requestAnimationFrame(movePlayer);
      }
      movePlayer();
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function onMouseDown(event) {
      if (!controls.isLocked) return;

      pointer.x = 0;
      pointer.y = 0;
      raycaster.setFromCamera(pointer, camera);
      const intersects = raycaster.intersectObjects(Object.values(blocks));

      if (intersects.length > 0) {
        const target = intersects[0];
        const pos = target.object.position.clone();

        if (event.button === 0) {
          // 壊す
          scene.remove(target.object);
          delete blocks[`${pos.x},${pos.y},${pos.z}`];
        } else if (event.button === 2) {
          // 設置
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
