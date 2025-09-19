<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>Three.js 動作確認</title>
<style>
body { margin: 0; overflow: hidden; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
</head>
<body>
<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(3, 3, 5);

let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 光
const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,10,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff,0.4));

// 赤いブロック
let geo = new THREE.BoxGeometry(1,1,1);
let mat = new THREE.MeshStandardMaterial({color:0xff0000});
let cube = new THREE.Mesh(geo, mat);
scene.add(cube);

function animate(){
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
