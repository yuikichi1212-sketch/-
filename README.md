<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Voxel テスト</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
  <!-- Three.js 読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.150.1/build/three.min.js"></script>

  <script>
    // シーン作成
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb); // 空色

    // カメラ
    const camera = new THREE.PerspectiveCamera(
      75, window.innerWidth / window.innerHeight, 0.1, 1000
    );
    camera.position.set(8, 10, 20);

    // レンダラー
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // ライト
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0xffffff, 0.4));

    // 地面（緑の平面）
    const geometry = new THREE.PlaneGeometry(50, 50);
    const material = new THREE.MeshStandardMaterial({ color: 0x228B22 });
    const plane = new THREE.Mesh(geometry, material);
    plane.rotation.x = -Math.PI / 2; // 横倒しにする
    scene.add(plane);

    // レンダリングループ
    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
