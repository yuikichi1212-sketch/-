<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>パズル回しw</title>
  <style>
    body {
      background-color: #111;
      color: #0f0;
      text-align: center;
      font-family: 'Courier New', monospace;
    }
    canvas {
      background-color: #000;
      border: 4px solid #0f0;
      margin-top: 10px;
    }
    #score, #level, #highscore {
      margin: 5px;
    }
    #startBtn, #restartBtn {
      margin-top: 10px;
      padding: 10px 20px;
      font-size: 16px;
      background: #0f0;
      color: #000;
      border: none;
      cursor: pointer;
    }
    #gameover {
      color: red;
      font-size: 24px;
      display: none;
    }
  </style>
</head>
<body>
  <h1>パズル回しw</h1>
  <canvas id="gameCanvas" width="160" height="320"></canvas>
  <div id="score">スコア: 0</div>
  <div id="level">レベル: 1</div>
  <div id="highscore">ハイスコア: 0</div>
  <div id="gameover"> GAME OVER してて草</div>
  <button id="startBtn">スタート</button>
  <button id="restartBtn" style="display:none;">もう一度あそぶよね？</button>

  <audio id="pop" src="pop.mp3"></audio>
  <audio id="bgm" src="bgm.mp3" loop></audio>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreDisplay = document.getElementById("score");
    const levelDisplay = document.getElementById("level");
    const highscoreDisplay = document.getElementById("highscore");
    const gameoverDisplay = document.getElementById("gameover");
    const startBtn = document.getElementById("startBtn");
    const restartBtn = document.getElementById("restartBtn");
    const popSound = document.getElementById("pop");
    const bgm = document.getElementById("bgm");

    const COLS = 8, ROWS = 16, BLOCK_SIZE = 20;
    const COLORS = ["#0f0", "#f00", "#ff0", "#0ff", "#f0f", "#aaa"];
    const creeperImg = new Image();
    creeperImg.src = "creeper.png";

    let grid, current, score, level, highscore, gameOver, dropInterval;

    function initGame() {
      grid = Array.from({ length: ROWS }, () => Array(COLS).fill(null));
      score = 0;
      level = 1;
      gameOver = false;
      current = createPair();
      updateDisplays();
      gameoverDisplay.style.display = "none";
      restartBtn.style.display = "none";
      bgm.play();
      clearInterval(dropInterval);
      dropInterval = setInterval(drop, 500);
    }

    function updateDisplays() {
      scoreDisplay.textContent = `スコア: ${score}`;
      levelDisplay.textContent = `レベル: ${level}`;
      highscore = Math.max(score, localStorage.getItem("highscore") || 0);
      localStorage.setItem("highscore", highscore);
      highscoreDisplay.textContent = `ハイスコア: ${highscore}`;
    }

    function createPair() {
      return {
        blocks: [
          { x: 3, y: 0, color: COLORS[Math.floor(Math.random() * COLORS.length)] },
          { x: 3, y: 1, color: COLORS[Math.floor(Math.random() * COLORS.length)] }
        ]
      };
    }

    function drawBlock(x, y, color) {
      ctx.drawImage(creeperImg, x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
    }

    function drawGrid() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let y = 0; y < ROWS; y++) {
        for (let x = 0; x < COLS; x++) {
          if (grid[y][x]) drawBlock(x, y, grid[y][x]);
        }
      }
      current.blocks.forEach(b => drawBlock(b.x, b.y, b.color));
    }

    function canMove(dx, dy) {
      return current.blocks.every(b => {
        const nx = b.x + dx, ny = b.y + dy;
        return nx >= 0 && nx < COLS && ny < ROWS && (!grid[ny][nx]);
      });
    }

    function move(dx, dy) {
      if (canMove(dx, dy)) current.blocks.forEach(b => { b.x += dx; b.y += dy; });
    }

    function rotate() {
      const [pivot, sub] = current.blocks;
      const dx = sub.x - pivot.x, dy = sub.y - pivot.y;
      const newX = pivot.x - dy, newY = pivot.y + dx;
      if (newX >= 0 && newX < COLS && newY < ROWS && !grid[newY][newX]) {
        sub.x = newX; sub.y = newY;
      }
    }

    function drop() {
      if (gameOver) return;
      if (canMove(0, 1)) move(0, 1);
      else {
        current.blocks.forEach(b => {
          if (b.y < 0) {
            endGame();
            return;
          }
          grid[b.y][b.x] = b.color;
        });
        clearMatches();
        current = createPair();
      }
      drawGrid();
    }

    function clearMatches() {
      let matched = [];
      for (let y = 0; y < ROWS; y++) {
        for (let x = 0; x < COLS; x++) {
          const color = grid[y][x];
          if (!color) continue;
          const group = [[x, y]];
          const visited = new Set([`${x},${y}`]);
          function dfs(cx, cy) {
            [[1,0],[0,1],[-1,0],[0,-1]].forEach(([dx, dy]) => {
              const nx = cx + dx, ny = cy + dy;
              if (nx >= 0 && nx < COLS && ny >= 0 && ny < ROWS &&
                  grid[ny][nx] === color && !visited.has(`${nx},${ny}`)) {
                visited.add(`${nx},${ny}`);
                group.push([nx, ny]);
                dfs(nx, ny);
              }
            });
          }
          dfs(x, y);
          if (group.length >= 4) matched.push(...group);
        }
      }

      if (matched.length > 0) {
        matched.forEach(([x, y]) => grid[y][x] = null);
        score += matched.length * 10;
        popSound.play();
        updateDisplays();
        setTimeout(applyGravity, 200);
        if (score >= level * 300) {
          level++;
          clearInterval(dropInterval);
          dropInterval = setInterval(drop, Math.max(100, 500 - level * 50));
        }
      }
    }

    function applyGravity() {
      for (let x = 0; x < COLS; x++) {
        for (let y = ROWS - 2; y >= 0; y--) {
          if (grid[y][x] && !grid[y + 1][x]) {
            let ny = y;
            while (ny + 1 < ROWS && !grid[ny + 1][x]) {
              grid[ny + 1][x] = grid[ny][x];
              grid[ny][x] = null;
              ny++;
            }
          }
        }
      }
      clearMatches();
      drawGrid();
    }

    function endGame() {
      gameOver = true;
      clearInterval(dropInterval);
      gameoverDisplay.style.display = "block";
      restartBtn.style.display = "inline-block";
      bgm.pause();
    }

    document.addEventListener("keydown", (e) => {
      if (gameOver) return;
      if (e.key
