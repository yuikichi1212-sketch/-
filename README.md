<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>Minecraft風テスト</title>
<style>
  body { margin:0; overflow:hidden; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
</head>
<body>
<script>
// シーン
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // 青空

// カメラ
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(5, 5, 10);

// レンダラー
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 光
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 5);
scene.add(light);

// ブロック作成関数
function createBlock(x, y, z, color=0x00ff00){
  const geometry = new THREE.BoxGeometry(1,1,1);
  const material = new THREE.MeshLambertMaterial({color});
  const cube = new THREE.Mesh(geometry, material);
  cube.position.set(x,y,z);
  scene.add(cube);
  return cube;
}

// 地面（10×10）
for(let i=-5;i<5;i++){
  for(let j=-5;j<5;j++){
    createBlock(i, 0, j, 0x228B22); // 緑
  }
}

// 置きブロック
createBlock(0,1,0,0x8B4513); // 茶色
createBlock(1,1,0,0xff0000); // 赤

// アニメーション
function animate(){
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
