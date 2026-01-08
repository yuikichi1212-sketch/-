<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>パズリング自作</title>
  <style>
    body {
      background-color: #1a1a1a;
      color: #00ff00;
      text-align: center;
      font-family: 'Courier New', monospace;
      margin-top: 30px;
    }

    canvas {
      background-color: #000;
      border: 4px solid #0f0;
      margin-top: 10px;
    }

    h1 {
      animation: glow 2s infinite alternate;
    }

    @keyframes glow {
      from { text-shadow: 0 0 5px #0f0; }
      to { text-shadow: 0 0 20px #0f0, 0 0 30px lime; }
    }
  </style>
</head>
<body>
  <h1>パズリング自作</h1>
  <canvas id="gameCanvas" width="160" height="320"></canvas>
  <p>← → 移動｜↑ 回転｜↓ 落下</p>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    const COLS = 8;
    const ROWS = 16;
    const BLOCK_SIZE = 20;
    const COLORS = ["limegreen", "red", "yellow", "cyan", "magenta", "orange"];

    let grid = Array.from({ length: ROWS }, () => Array(COLS).fill(null));

    function getRandomColor() {
      return COLORS[Math.floor(Math.random() * COLORS.length)];
    }

    function createPair() {
      return {
        blocks: [
          { x: 3, y: 0, color: getRandomColor() },
          { x: 3, y: 1, color: getRandomColor() }
        ],
        rotation: 0
      };
    }

    let current = createPair();

    function drawBlock(x, y, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
      ctx.strokeStyle = "#003300";
      ctx.strokeRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
    }

    function drawGrid() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let y = 0; y < ROWS; y++) {
        for (let x = 0; x < COLS; x++) {
          if (grid[y][x]) {
            drawBlock(x, y, grid[y][x]);
          }
        }
      }
      current.blocks.forEach(b => drawBlock(b.x, b.y, b.color));
    }

    function canMove(dx, dy) {
      return current.blocks.every(b => {
        const nx = b.x + dx;
        const ny = b.y + dy;
        return nx >= 0 && nx < COLS && ny < ROWS && (!grid[ny] || !grid[ny][nx]);
      });
    }

    function move(dx, dy) {
      if (canMove(dx, dy)) {
        current.blocks.forEach(b => {
          b.x += dx;
          b.y += dy;
        });
      }
    }

    function rotate() {
      const [pivot, sub] = current.blocks;
      const dx = sub.x - pivot.x;
      const dy = sub.y - pivot.y;
      const newX = pivot.x - dy;
      const newY = pivot.y + dx;
      if (
        newX >= 0 && newX < COLS &&
        newY >= 0 && newY < ROWS &&
        !grid[newY][newX]
      ) {
        sub.x = newX;
        sub.y = newY;
      }
    }

    function drop() {
      if (canMove(0, 1)) {
        move(0, 1);
      } else {
        current.blocks.forEach(b => {
          if (b.y >= 0) grid[b.y][b.x] = b.color;
        });
        current = createPair();
      }
      drawGrid();
    }

    setInterval(drop, 500);

    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft") move(-1, 0);
      if (e.key === "ArrowRight") move(1, 0);
      if (e.key === "ArrowDown") move(0, 1);
      if (e.key === "ArrowUp") rotate();
      drawGrid();
    });
  </script>
</body>
</html>
