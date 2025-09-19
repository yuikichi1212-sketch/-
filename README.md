<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Three.js テスト</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
</head>
<body>
<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // 青空

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.z = 5;

let renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 箱（赤）
let geometry = new THREE.BoxGeometry();
let material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
let cube = new THREE.Mesh(geometry, material);
scene.add(cube);

function animate() {
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
