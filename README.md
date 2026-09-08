<!DOCTYPE html>
<html>
<head>
  <title>Craft-Man Simpel</title>
  <style>
    body { background: #111; color: #fff; font-family: sans-serif; text-align: center; }
    canvas { background: #000; border: 2px solid #555; margin-top: 10px; }
  </style>
</head>
<body>

  <h2>CRAFT-MAN</h2>
  <p>Skor: <span id="score">0</span></p>
  <canvas id="game" width="300" height="300"></canvas>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const size = 30;
    let score = 0;

    // Posisi Awal Player (Steve) & Musuh (Creeper)
    let player = { x: 1, y: 1 };
    let creeper = { x: 8, y: 8 };

    // Peta: 1 = Dinding Batu, 0 = XP Orb, 2 = Jalan Kosong
    const map = [
      [1,1,1,1,1,1,1,1,1,1],
      [1,2,0,0,1,0,0,0,0,1],
      [1,0,1,0,1,0,1,1,0,1],
      [1,0,1,0,0,0,0,1,0,1],
      [1,0,1,1,1,1,0,1,0,1],
      [1,0,0,0,0,0,0,0,0,1],
      [1,0,1,1,0,1,1,1,0,1],
      [1,0,0,1,0,0,0,1,0,1],
      [1,0,0,0,0,1,0,0,2,1],
      [1,1,1,1,1,1,1,1,1,1]
    ];

    // Kontrol Pergerakan
    document.addEventListener('keydown', (e) => {
      let nextX = player.x, nextY = player.y;
      if (e.key === 'ArrowUp' || e.key === 'w') nextY--;
      if (e.key === 'ArrowDown' || e.key === 's') nextY++;
      if (e.key === 'ArrowLeft' || e.key === 'a') nextX--;
      if (e.key === 'ArrowRight' || e.key === 'd') nextX++;

      // Cek Tabrakan Dinding
      if (map[nextY][nextX] !== 1) {
        player.x = nextX;
        player.y = nextY;

        // Makan XP Orb
        if (map[player.y][player.x] === 0) {
          map[player.y][player.x] = 2;
          score += 10;
          document.getElementById('score').innerText = score;
        }
      }
      draw();
    });

    // Menggambar Game ke Layar
    function draw() {
      ctx.clearRect(0, 0, 300, 300);

      // Gambar Peta
      for (let r = 0; r < 10; r++) {
        for (let c = 0; c < 10; c++) {
          if (map[r][c] === 1) { ctx.fillStyle = '#555'; ctx.fillRect(c*size, r*size, size, size); } // Dinding Abu-abu
          if (map[r][c] === 0) { ctx.fillStyle = '#5f5'; ctx.fillRect(c*size+12, r*size+12, 6, 6); } // XP Hijau
        }
      }

      // Gambar Player (Biru) & Creeper (Hijau)
      ctx.fillStyle = '#00a'; ctx.fillRect(player.x*size+4, player.y*size+4, 22, 22);
      ctx.fillStyle = '#0f0'; ctx.fillRect(creeper.x*size+4, creeper.y*size+4, 22, 22);
    }

    draw();
  </script>
</body>
</html>
