<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>Canvasマイクラ風</title>
<style>
  body { margin:0; background:#87ceeb; }
  canvas { display:block; margin:auto; background:#87ceeb; }
  #info { position:absolute; top:10px; left:10px; font-family:sans-serif; color:white; }
</style>
</head>
<body>
<div id="info">左クリック: 壊す / 右クリック: 置く</div>
<canvas id="game" width="640" height="480"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const rows = 15;
const cols = 20;
const size = 32; // 1ブロックの大きさ
let blocks = [];

// 初期化（地面を生成）
for(let y=0;y<rows;y++){
  blocks[y] = [];
  for(let x=0;x<cols;x++){
    if(y > rows/2) blocks[y][x] = 1; // 草ブロック
    else blocks[y][x] = 0; // 空
  }
}

// 描画関数
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  for(let y=0;y<rows;y++){
    for(let x=0;x<cols;x++){
      if(blocks[y][x]===1){
        ctx.fillStyle="#228B22"; // 緑
        ctx.fillRect(x*size,y*size,size,size);
        ctx.strokeStyle="black";
        ctx.strokeRect(x*size,y*size,size,size);
      }
      if(blocks[y][x]===2){
        ctx.fillStyle="#8B4513"; // 茶色
        ctx.fillRect(x*size,y*size,size,size);
        ctx.strokeStyle="black";
        ctx.strokeRect(x*size,y*size,size,size);
      }
    }
  }
}
draw();

// クリック操作
canvas.addEventListener("mousedown",e=>{
  const rect = canvas.getBoundingClientRect();
  const x = Math.floor((e.clientX-rect.left)/size);
  const y = Math.floor((e.clientY-rect.top)/size);

  if(x<0||x>=cols||y<0||y>=rows) return;

  if(e.button===0){
    // 左クリック: 壊す
    blocks[y][x] = 0;
  } else if(e.button===2){
    // 右クリック: 置く
    if(blocks[y][x]===0) blocks[y][x] = 2;
  }
  draw();
});

// 右クリックメニュー禁止
canvas.addEventListener("contextmenu", e=>e.preventDefault());
</script>
</body>
</html>
