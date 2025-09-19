<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>3Dマイクラ風サンプル</title>
<style>
  body { margin:0; overflow:hidden; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>
<script>
// シーン
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

// カメラ
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(10,15,20);

// レンダラー
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 光
const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff,0.5));

// 16x16の地面
const geo = new THREE.BoxGeometry(1,1,1);
const mat = new THREE.MeshStandardMaterial({color:0x228B22});
for(let x=0;x<16;x++){
  for(let z=0;z<16;z++){
    const cube = new THREE.Mesh(geo,mat);
    cube.position.set(x,0,z);
    scene.add(cube);
  }
}

// カメラ操作（マウスで回せる）
const controls = new THREE.OrbitControls(camera, renderer.domElement);

// リサイズ対応
window.addEventListener("resize",()=>{
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// 描画ループ
function animate(){
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
