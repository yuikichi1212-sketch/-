<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>ぷよぷよ完全版</title>
  <style>
    body {
      background: #222;
      color: #fff;
      font-family: sans-serif;
      text-align: center;
    }
    canvas {
      background: #000;
      display: inline-block;
      border: 2px solid #fff;
    }
    #info {
      display: inline-block;
      vertical-align: top;
      margin-left: 20px;
      text-align: left;
    }
    #info h2 {
      margin-top: 0;
    }
    .next {
      width: 80px;
      height: 80px;
      background: #111;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 10px;
    }
    .next canvas {
      background: #000;
      border: 1px solid #fff;
    }
  </style>
</head>
<body>
  <h1>ぷよぷよ風ゲーム</h1>
  <div>
    <canvas id="puyo" width="240" height="480"></canvas>
    <div id="info">
      <h2>スコア: <span id="score">0</span></h2>
      <div class="next">
        <canvas id="next" width="80" height="80"></canvas>
      </div>
      <p>← → 移動　↑ 回転　↓ 落下</p>
    </div>
  </div>
  <script>
    const COLS = 6, ROWS = 12, SIZE = 40;
    const COLORS = ['#000', '#f00', '#0f0', '#00f', '#ff0', '#f0f'];
    const canvas = document.getElementById('puyo');
    const ctx = canvas.getContext('2d');
    const nextCanvas = document.getElementById('next');
    const nextCtx = nextCanvas.getContext('2d');
    const scoreDisplay = document.getElementById('score');

    let field = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
    let score = 0;

    function randColor() {
      return Math.floor(Math.random() * 5) + 1;
    }

    function createPair() {
      return {
        blocks: [
          { x: 2, y: 0, color: nextColor },
          { x: 2, y: -1, color: randColor() }
        ],
        rotation: 0
      };
    }

    function drawBlock(ctx, x, y, color, size) {
      if (color === 0) return;
      ctx.fillStyle = COLORS[color];
      ctx.fillRect(x * size, y * size, size - 2, size - 2);
    }

    function drawField() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let y = 0; y < ROWS; y++) {
        for (let x = 0; x < COLS; x++) {
          drawBlock(ctx, x, y, field[y][x], SIZE);
        }
      }
      pair.blocks.forEach(b => drawBlock(ctx, b.x, b.y, b.color, SIZE));
    }

    function drawNext() {
      nextCtx.clearRect(0, 0, 80, 80);
      drawBlock(nextCtx, 1, 1, nextColor, 20);
    }

    function canMove(dx, dy) {
      return pair.blocks.every(b => {
        let nx = b.x + dx, ny = b.y + dy;
        return nx >= 0 && nx < COLS && ny < ROWS && (ny < 0 || field[ny][nx] === 0);
      });
    }

    function move(dx, dy) {
      if (canMove(dx, dy)) {
        pair.blocks.forEach(b => {
          b.x += dx;
          b.y += dy;
        });
      } else if (dy === 1) {
        place();
        processMatches(() => {
          spawn();
        });
      }
    }

    function rotate() {
      let [a, b] = pair.blocks;
      let dx = b.x - a.x;
      let dy = b.y - a.y;
      let rx = -dy, ry = dx;
      let nx = a.x + rx, ny = a.y + ry;
      if (nx >= 0 && nx < COLS && ny < ROWS && (ny < 0 || field[ny][nx] === 0)) {
        b.x = nx;
        b.y = ny;
      }
    }

    function place() {
      pair.blocks.forEach(b => {
        if (b.y >= 0) field[b.y][b.x] = b.color;
      });
    }

    function spawn() {
      pair = createPair();
      nextColor = randColor();
      drawNext();
      if (!canMove(0, 0)) {
        alert("ゲームオーバー！");
        field = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
        score = 0;
        scoreDisplay.textContent = score;
      }
    }

    function processMatches(callback) {
      let chain = 0;
      function checkAndClear() {
        let visited = Array.from({ length: ROWS }, () => Array(COLS).fill(false));
        let toClear = [];

        function dfs(x, y, color, group) {
          if (x < 0 || x >= COLS || y < 0 || y >= ROWS) return;
          if (visited[y][x] || field[y][x] !== color) return;
          visited[y][x] = true;
          group.push({ x, y });
          dfs(x + 1, y, color, group);
          dfs(x - 1, y, color, group);
          dfs(x, y + 1, color, group);
          dfs(x, y - 1, color, group);
        }

        for (let y = 0; y < ROWS; y++) {
          for (let x = 0; x < COLS; x++) {
            if (field[y][x] !== 0 && !visited[y][x]) {
              let group = [];
              dfs(x, y, field[y][x], group);
              if (group.length >= 4) toClear.push(...group);
            }
          }
        }

        toClear.forEach(({ x, y }) => field[y][x] = 0);
        if (toClear.length > 0) {
          score += toClear.length * 10 * (chain + 1);
          scoreDisplay.textContent = score;
          applyGravity();
          chain++;
          setTimeout(checkAndClear, 300);
        } else {
          callback();
        }
      }
      checkAndClear();
    }

    function applyGravity() {
      for (let x = 0; x < COLS; x++) {
        for (let y = ROWS - 1; y >= 0; y--) {
          if (field[y][x] === 0) {
            for (let k = y - 1; k >= 0; k--) {
              if (field[k][x] !== 0) {
                field[y][x] = field[k][x];
                field[k][x] = 0;
                break;
              }
            }
          }
        }
      }
    }

    document.addEventListener('keydown', e => {
      if (e.key === 'ArrowLeft') move(-1, 0);
      if (e.key === 'ArrowRight') move(1, 0);
      if (e.key === 'ArrowDown') move(0, 1);
      if (e.key === 'ArrowUp') rotate();
    });

    let pair, nextColor = randColor();
    spawn();
    drawNext();

    function update() {
      move(0, 1);
      drawField();
    }

    setInterval(update, 500);
  </script>
</body>
</html>
