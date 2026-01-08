<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>クリーパーパズル</title>
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
  </style>
</head>
<body>
  <h1>💣 クリーパーパズル 💣</h1>
  <canvas id="gameCanvas" width="160" height="320"></canvas>
  <p>← → で移動、下に落ちるよ！</p>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    const COLS = 8;
    const ROWS = 16;
    const BLOCK_SIZE = 20;

    let grid = Array.from({ length: ROWS }, () => Array(COLS).fill(0));

    let current = {
      x: 3,
      y: 0,
      color: "limegreen"
    };

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
      drawBlock(current.x, current.y, current.color);
    }

    function drop() {
      if (current.y + 1 < ROWS && !grid[current.y + 1][current.x]) {
        current.y++;
      } else {
        grid[current.y][current.x] = current.color;
        current = { x: 3, y: 0, color: "limegreen" };
      }
      drawGrid();
    }

    setInterval(drop, 500);

    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft" && current.x > 0) current.x--;
      if (e.key === "ArrowRight" && current.x < COLS - 1) current.x++;
      drawGrid();
    });
  </script>
</body>
</html>
