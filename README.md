<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Minecraft Pac-Man (Craft-Man)</title>
  <style>
    body {
      background-color: #121212;
      color: #ffffff;
      font-family: 'Courier New', Courier, monospace;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      margin: 0;
    }

    h1 {
      margin: 5px;
      color: #55ff55;
      text-shadow: 2px 2px #00aa00;
    }

    .hud {
      display: flex;
      gap: 30px;
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 10px;
      background: #222;
      padding: 10px 20px;
      border: 2px solid #555;
    }

    canvas {
      border: 4px solid #555;
      background-color: #000;
      box-shadow: 0 0 20px rgba(85, 255, 85, 0.2);
    }

    .controls-info {
      margin-top: 10px;
      font-size: 14px;
      color: #aaa;
    }
  </style>
</head>
<body>

  <h1>CRAFT-MAN</h1>
  <div class="hud">
    <div>SCORE: <span id="score" style="color: #ffff55;">0</span></div>
    <div>LIVES: <span id="lives" style="color: #ff5555;">3</span></div>
  </div>

  <canvas id="gameCanvas" width="570" height="570"></canvas>

  <div class="controls-info">
    Gunakan tombol <b>Panah (Arrow Keys)</b> atau <b>WASD</b> untuk bergerak.
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreEl = document.getElementById("score");
    const livesEl = document.getElementById("lives");

    const tileSize = 30;
    const gridCount = 19;

    let score = 0;
    let lives = 3;
    let gameOver = false;
    let powerModeTimer = 0;

    // Map: 1 = Wall (Stone), 0 = XP Orb, 2 = Golden Apple, 3 = Empty
    const map = [
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
      [1,2,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,2,1],
      [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
      [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
      [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
      [1,0,1,1,0,1,0,1,1,1,1,1,0,1,0,1,1,0,1],
      [1,0,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,0,1],
      [1,1,1,1,0,1,1,1,3,1,3,1,1,1,0,1,1,1,1],
      [3,3,3,1,0,1,3,3,3,3,3,3,3,1,0,1,3,3,3],
      [1,1,1,1,0,1,3,1,1,3,1,1,3,1,0,1,1,1,1],
      [3,3,3,3,0,3,3,1,3,3,3,1,3,3,0,3,3,3,3],
      [1,1,1,1,0,1,3,1,1,1,1,1,3,1,0,1,1,1,1],
      [3,3,3,1,0,1,3,3,3,3,3,3,3,1,0,1,3,3,3],
      [1,1,1,1,0,1,0,1,1,1,1,1,0,1,0,1,1,1,1],
      [1,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,1],
      [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
      [1,2,0,1,0,0,0,0,0,3,0,0,0,0,0,1,0,2,1],
      [1,1,0,1,0,1,0,1,1,1,1,1,0,1,0,1,0,1,1],
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
    ];

    // Player (Steve)
    const player = {
      x: 9,
      y: 16,
      dirX: 0,
      dirY: 0,
      nextDirX: 0,
      nextDirY: 0
    };

    // Mobs (Ghosts)
    const mobs = [
      { x: 8, y: 9, color: "creeper", dirX: 1, dirY: 0 },
      { x: 9, y: 9, color: "zombie", dirX: -1, dirY: 0 },
      { x: 10, y: 9, color: "skeleton", dirX: 0, dirY: -1 },
      { x: 9, y: 8, color: "enderman", dirX: 0, dirY: 1 }
    ];

    // Event Listener Keyboard
    window.addEventListener("keydown", (e) => {
      switch (e.key) {
        case "ArrowUp": case "w": case "W":
          player.nextDirX = 0; player.nextDirY = -1; break;
        case "ArrowDown": case "s": case "S":
          player.nextDirX = 0; player.nextDirY = 1; break;
        case "ArrowLeft": case "a": case "A":
          player.nextDirX = -1; player.nextDirY = 0; break;
        case "ArrowRight": case "d": case "D":
          player.nextDirX = 1; player.nextDirY = 0; break;
      }
    });

    function update() {
      if (gameOver) return;

      // Cek apakah pemain bisa berbelok ke arah yang dipencet
      if (canMove(player.x + player.nextDirX, player.y + player.nextDirY)) {
        player.dirX = player.nextDirX;
        player.dirY = player.nextDirY;
      }

      // Gerakkan Steve
      if (canMove(player.x + player.dirX, player.y + player.dirY)) {
        player.x += player.dirX;
        player.y += player.dirY;

        // Tembus batas terowongan (Kiri-Kanan)
        if (player.x < 0) player.x = gridCount - 1;
        if (player.x >= gridCount) player.x = 0;
      }

      // Makan Item
      const currentTile = map[player.y][player.x];
      if (currentTile === 0) { // XP Orb
        map[player.y][player.x] = 3;
        score += 10;
      } else if (currentTile === 2) { // Golden Apple
        map[player.y][player.x] = 3;
        score += 50;
        powerModeTimer = 30; // 30 tick power mode
      }
      scoreEl.innerText = score;

      if (powerModeTimer > 0) powerModeTimer--;

      // Update Mobs
      mobs.forEach(mob => {
        // Gerakan AI Acak Sederhana
        const possibleMoves = [
          { x: 0, y: -1 }, { x: 0, y: 1 },
          { x: -1, y: 0 }, { x: 1, y: 0 }
        ].filter(m => canMove(mob.x + m.x, mob.y + m.y));

        if (possibleMoves.length > 0) {
          const move = possibleMoves[Math.floor(Math.random() * possibleMoves.length)];
          mob.x += move.x;
          mob.y += move.y;
        }

        // Cek Tabrakan Steve & Mob
        if (mob.x === player.x && mob.y === player.y) {
          if (powerModeTimer > 0) {
            // Respawn Mob ke tengah
            mob.x = 9;
            mob.y = 9;
            score += 200;
          } else {
            // Steve Kena
            lives--;
            livesEl.innerText = lives;
            resetPositions();
            if (lives <= 0) {
              gameOver = true;
              alert("Game Over! Total Skor Anda: " + score);
            }
          }
        }
      });
    }

    function canMove(x, y) {
      if (x < 0 || x >= gridCount || y < 0 || y >= gridCount) return true; // Untuk portal
      return map[y][x] !== 1;
    }

    function resetPositions() {
      player.x = 9; player.y = 16;
      player.dirX = 0; player.dirY = 0;
      player.nextDirX = 0; player.nextDirY = 0;
      mobs[0].x = 8; mobs[0].y = 9;
      mobs[1].x = 9; mobs[1].y = 9;
      mobs[2].x = 10; mobs[2].y = 9;
      mobs[3].x = 9; mobs[3].y = 8;
    }

    // DRAW FUNCTIONS
    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Draw Map
      for (let r = 0; r < gridCount; r++) {
        for (let c = 0; c < gridCount; c++) {
          const x = c * tileSize;
          const y = r * tileSize;

          if (map[r][c] === 1) { // Wall Stone
            ctx.fillStyle = "#555555";
            ctx.fillRect(x, y, tileSize, tileSize);
            ctx.strokeStyle = "#333333";
            ctx.strokeRect(x, y, tileSize, tileSize);
          } else if (map[r][c] === 0) { // XP Orb
            ctx.fillStyle = "#55ff55";
            ctx.fillRect(x + 12, y + 12, 6, 6);
          } else if (map[r][c] === 2) { // Golden Apple
            ctx.fillStyle = "#ffaa00";
            ctx.fillRect(x + 8, y + 8, 14, 14);
            ctx.fillStyle = "#55ff55"; // Daun
            ctx.fillRect(x + 12, y + 4, 4, 4);
          }
        }
      }

      // Draw Steve (Player)
      drawSteve(player.x * tileSize, player.y * tileSize);

      // Draw Mobs
      mobs.forEach(mob => {
        if (powerModeTimer > 0) {
          drawMobHead(mob.x * tileSize, mob.y * tileSize, "#0000aa", "scared");
        } else {
          drawMob(mob);
        }
      });
    }

    function drawSteve(x, y) {
      // Kepala Steve
      ctx.fillStyle = "#db8254"; // Kulit
      ctx.fillRect(x + 3, y + 3, 24, 24);
      ctx.fillStyle = "#4a270f"; // Rambut
      ctx.fillRect(x + 3, y + 3, 24, 6);
      ctx.fillStyle = "#ffffff"; // Mata
      ctx.fillRect(x + 5, y + 12, 6, 4);
      ctx.fillRect(x + 19, y + 12, 6, 4);
      ctx.fillStyle = "#0000aa"; // Pupil
      ctx.fillRect(x + 8, y + 12, 3, 4);
      ctx.fillRect(x + 19, y + 12, 3, 4);
    }

    function drawMob(mob) {
      const x = mob.x * tileSize;
      const y = mob.y * tileSize;

      if (mob.color === "creeper") {
        drawMobHead(x, y, "#55ff55", "creeper");
      } else if (mob.color === "zombie") {
        drawMobHead(x, y, "#00aaaa", "zombie");
      } else if (mob.color === "skeleton") {
        drawMobHead(x, y, "#aaaaaa", "skeleton");
      } else if (mob.color === "enderman") {
        drawMobHead(x, y, "#220022", "enderman");
      }
    }

    function drawMobHead(x, y, bgColor, type) {
      ctx.fillStyle = bgColor;
      ctx.fillRect(x + 3, y + 3, 24, 24);

      if (type === "creeper") {
        ctx.fillStyle = "#000000";
        ctx.fillRect(x + 7, y + 8, 5, 5); // Mata Kiri
        ctx.fillRect(x + 18, y + 8, 5, 5); // Mata Kanan
        ctx.fillRect(x + 11, y + 13, 8, 9); // Mulut
      } else if (type === "enderman") {
        ctx.fillStyle = "#ff00ff"; // Mata ungu menyala
        ctx.fillRect(x + 5, y + 13, 6, 3);
        ctx.fillRect(x + 19, y + 13, 6, 3);
      } else if (type === "scared") {
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(x + 7, y + 10, 4, 4);
        ctx.fillRect(x + 19, y + 10, 4, 4);
      } else {
        // Zombie & Skeleton
        ctx.fillStyle = "#000000";
        ctx.fillRect(x + 6, y + 10, 5, 5);
        ctx.fillRect(x + 19, y + 10, 5, 5);
      }
    }

    // Game Loop
    function gameLoop() {
      update();
      draw();
      setTimeout(() => {
        requestAnimationFrame(gameLoop);
      }, 200); // Speed control
    }

    gameLoop();
  </script>
</body>
</html>
