<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>Three.js 地面テスト</title>
<style>
body { margin:0; overflow:hidden; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
</head>
<body>
<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(8, 8, 12); // カメラを少し上から見る位置に

let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 光
let light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff,0.4));

// 16x16 地面ブロック
let geo = new THREE.BoxGeometry(1,1,1);
let mat = new THREE.MeshStandardMaterial({color:0x228B22});

for(let x=0;x<16;x++){
  for(let z=0;z<16;z++){
    let cube = new THREE.Mesh(geo, mat);
    cube.position.set(x,0,z);
    scene.add(cube);
  }
}

// カメラを地面に向ける
camera.lookAt(8,0,8);

function animate(){
  requestAnimationFrame(animate);
  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>

