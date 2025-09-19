<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>赤い立方体テスト</title>
<style>
  body { margin:0; overflow:hidden; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
</head>
<body>
<script>
// シーン
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // 青空背景

// カメラ
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.z = 5;  // 手前から見る

// レンダラー
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 赤い立方体
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({color: 0xff0000});
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// アニメーション
function animate(){
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
