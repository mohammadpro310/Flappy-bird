<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird</title>
  <style>
    body {
      margin: 0;
      background: #70c5ce;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    canvas {
      background: #70c5ce;
      display: block;
      border: 2px solid #000;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const WIDTH = canvas.width;
    const HEIGHT = canvas.height;

    // Bird properties
    const bird = {
      x: 80,
      y: HEIGHT / 2,
      radius: 15,
      velocity: 0,
      gravity: 0.5,
      lift: -10
    };

    // Pipes
    const pipeWidth = 60;
    const pipeGap = 150;
    let pipes = [];

    // Score
    let score = 0;
    let gameOver = false;

    // Create pipes periodically
    function createPipe() {
      const topHeight = Math.random() * (HEIGHT - pipeGap - 100) + 50;
      pipes.push({
        x: WIDTH,
        topHeight: topHeight,
        bottomY: topHeight + pipeGap,
        passed: false
      });
    }

    setInterval(() => {
      if (!gameOver) createPipe();
    }, 1500);

    // Handle user input
    window.addEventListener('keydown', e => {
      if (e.code === 'Space') {
        if (!gameOver) bird.velocity = bird.lift;
        else resetGame();
      }
    });

    // Touch support for mobile
    window.addEventListener('touchstart', e => {
      if (!gameOver) bird.velocity = bird.lift;
      else resetGame();
      e.preventDefault();
    }, { passive: false });

    function resetGame() {
      bird.y = HEIGHT / 2;
      bird.velocity = 0;
      pipes = [];
      score = 0;
      gameOver = false;
    }

    function update() {
      if (gameOver) return;

      bird.velocity += bird.gravity;
      bird.y += bird.velocity;

      // Ground and ceiling collision
      if (bird.y + bird.radius > HEIGHT || bird.y - bird.radius < 0) {
        gameOver = true;
      }

      // Move pipes
      for (let i = 0; i < pipes.length; i++) {
        pipes[i].x -= 3;

        // Check for passing pipe for scoring
        if (!pipes[i].passed && pipes[i].x + pipeWidth < bird.x) {
          pipes[i].passed = true;
          score++;
        }

        // Collision detection with pipes
        if (
          bird.x + bird.radius > pipes[i].x &&
          bird.x - bird.radius < pipes[i].x + pipeWidth &&
          (bird.y - bird.radius < pipes[i].topHeight ||
           bird.y + bird.radius > pipes[i].bottomY)
        ) {
          gameOver = true;
        }
      }

      // Remove off-screen pipes
      pipes = pipes.filter(pipe => pipe.x + pipeWidth > 0);
    }

    function draw() {
      ctx.clearRect(0, 0, WIDTH, HEIGHT);

      // Draw bird
      ctx.fillStyle = 'yellow';
      ctx.beginPath();
      ctx.arc(bird.x, bird.y, bird.radius, 0, Math.PI * 2);
      ctx.fill();

      // Draw pipes
      ctx.fillStyle = 'green';
      pipes.forEach(pipe => {
        // Top pipe
        ctx.fillRect(pipe.x, 0, pipeWidth, pipe.topHeight);
        // Bottom pipe
        ctx.fillRect(pipe.x, pipe.bottomY, pipeWidth, HEIGHT - pipe.bottomY);
      });

      // Draw score
      ctx.fillStyle = 'white';
      ctx.font = '30px Arial';
      ctx.fillText(`Score: ${score}`, 10, 50);

      // Game over text
      if (gameOver) {
        ctx.fillStyle = 'red';
        ctx.font = '50px Arial';
        ctx.fillText('Game Over!', WIDTH / 2 - 130, HEIGHT / 2);
        ctx.font = '20px Arial';
        ctx.fillText('Press Space or Tap to Restart', WIDTH / 2 - 130, HEIGHT / 2 + 40);
      }
    }

    function loop() {
      update();
      draw();
      requestAnimationFrame(loop);
    }

    loop();
  </script>
</body>
</html>
