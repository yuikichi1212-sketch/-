<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>ブラウザ版マイクラ風</title>
<style>
body { margin:0; overflow:hidden; }
#info { position:absolute; top:10px; left:10px; color:white; font-family:sans-serif; z-index:1; }
</style>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/examples/js/controls/PointerLockControls.js"></script>
</head>
<body>
<div id="info">クリックで視点ロック<br>左クリック: 壊す / 右クリック: 置く</div>
<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(8,8,12);

let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(10,20,10);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff,0.4));

let controls = new THREE.PointerLockControls(camera, renderer.domElement);
document.body.addEventListener('click',()=>controls.lock());
scene.add(controls.getObject());

let blockSize = 1;
let blocks = {};
let geo = new THREE.BoxGeometry(blockSize,blockSize,blockSize);

// 16x16 地面
for(let x=0;x<16;x++){
  for(let z=0;z<16;z++){
    let mat = new THREE.MeshStandardMaterial({color:0x228B22});
    let cube = new THREE.Mesh(geo, mat);
    cube.position.set(x,0,z);
    scene.add(cube);
    blocks[`${x},0,${z}`] = cube;
  }
}

// Raycaster
let raycaster = new THREE.Raycaster();
let pointer = new THREE.Vector2();

document.addEventListener('mousedown',onMouseDown);
document.addEventListener('contextmenu', e=>e.preventDefault()); // 右クリックメニュー無効

function onMouseDown(event){
  if(!controls.isLocked) return;

  pointer.x = 0;
  pointer.y = 0;
  raycaster.setFromCamera(pointer, camera);
  const intersects = raycaster.intersectObjects(Object.values(blocks));

  if(intersects.length>0){
    const target = intersects[0];
    const pos = target.object.position.clone();

    if(event.button===0){
      // 左クリックで壊す
      scene.remove(target.object);
      delete blocks[`${pos.x},${pos.y},${pos.z}`];
    } else if(event.button===2){
      // 右クリックで置く
      const normal = target.face.normal;
      const newPos = pos.clone().add(normal);
      const key = `${newPos.x},${newPos.y},${newPos.z}`;
      if(!blocks[key]){
        let mat = new THREE.MeshStandardMaterial({color:0x8B4513});
        let cube = new THREE.Mesh(geo, mat);
        cube.position.copy(newPos);
        scene.add(cube);
        blocks[key] = cube;
      }
    }
  }
}

window.addEventListener('resize',()=>{camera.aspect=window.innerWidth/window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);});
camera.lookAt(8,0,8);

function animate(){
  requestAnimationFrame(animate);
  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
