📂 VoxelWebGame (GitHub用完全版構造)

├─ index.html       ← ゲーム画面表示
├─ main.js          ← ブラウザ版マイクラ風ゲームロジック
└─ style.css        ← 簡易CSS（画面全体・文字表示）

---

/\* index.html \*/

<!DOCTYPE html>

<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Voxel WebGL Game</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="info">左クリックで壊す / 右クリックで設置</div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r150/three.min.js"></script>
    <script src="main.js"></script>
</body>
</html>

/\* style.css \*/
body { margin:0; overflow\:hidden; }
canvas { display\:block; }
\#info { position\:absolute; top:10px; left:10px; color\:white; font-family\:sans-serif; z-index:1; }

/\* main.js \*/
let scene, camera, renderer, raycaster;
let blocks = {};
let blockSize = 1;
let pointer = new THREE.Vector2();
let controls;

init();
animate();

function init() {
scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

```
camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(8,10,8);

renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

controls = new THREE.PointerLockControls(camera, renderer.domElement);
document.body.addEventListener('click',()=>{controls.lock();});
scene.add(controls.getObject());

raycaster = new THREE.Raycaster();

const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);

const ambient = new THREE.AmbientLight(0xffffff,0.4);
scene.add(ambient);

// 地面生成
let geometry = new THREE.BoxGeometry(blockSize,blockSize,blockSize);
let material = new THREE.MeshStandardMaterial({color:0x228B22});
for(let x=0;x<16;x++){
    for(let z=0;z<16;z++){
        let cube = new THREE.Mesh(geometry, material);
        cube.position.set(x,0,z);
        scene.add(cube);
        blocks[`${x},0,${z}`]=cube;
    }
}

window.addEventListener('resize',onWindowResize);
document.addEventListener('mousedown',onMouseDown);
```

}

function onWindowResize(){
camera.aspect=window\.innerWidth/window\.innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(window\.innerWidth,window\.innerHeight);
}

function onMouseDown(event){
if(!controls.isLocked) return;
pointer.x=0; pointer.y=0;
raycaster.setFromCamera(pointer,camera);
const intersects = raycaster.intersectObjects(Object.values(blocks));
if(intersects.length>0){
const target = intersects\[0];
const pos = target.object.position.clone();
if(event.button===0){ // 左クリック破壊
scene.remove(target.object);
delete blocks\[`${pos.x},${pos.y},${pos.z}`];
} else if(event.button===2){ // 右クリック設置
const normal = target.face.normal;
const newPos = pos.clone().add(normal);
const key = `${newPos.x},${newPos.y},${newPos.z}`;
if(!blocks\[key]){
let geometry = new THREE.BoxGeometry(blockSize,blockSize,blockSize);
let material = new THREE.MeshStandardMaterial({color:0x8B4513});
let cube = new THREE.Mesh(geometry, material);
cube.position.copy(newPos);
scene.add(cube);
blocks\[key]=cube;
}
}
}
}

function animate(){
requestAnimationFrame(animate);
renderer.render(scene,camera);
}
