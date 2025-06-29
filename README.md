flappy-bird/
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flappy Bird</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="menu">
    <h1>Flappy Bird</h1>
    <button onclick="startGame()">Start</button>
  </div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <div id="gameOverScreen">
    <h2>Game Over</h2>
    <button onclick="startGame()">Restart</button>
  </div>
  <script src="script.js"></script>
</body>
</html>
body {
  margin: 0;
  background: #70c5ce;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  font-family: Arial, sans-serif;
}
canvas {
  display: none;
  border: 2px solid #000;
  background: #fff;
}
#menu, #gameOverScreen {
  position: absolute;
  text-align: center;
}
button {
  padding: 10px 20px;
  font-size: 18px;
  margin-top: 10px;
  cursor: pointer;
}
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const menu = document.getElementById('menu');
const gameOverScreen = document.getElementById('gameOverScreen');

const birdImg = new Image();
birdImg.src = 'assets/bird.png';

const flapSound = new Audio('assets/flap.wav');
const pointSound = new Audio('assets/point.wav');
const hitSound = new Audio('assets/hit.wav');

let bird, pipes, gravity, jump, score, gameOver;

function initGame() {
  bird = { x: 50, y: 150, width: 34, height: 24, velocity: 0 };
  gravity = 0.5;
  jump = -8;
  pipes = [];
  score = 0;
  gameOver = false;
}

function startGame() {
  menu.style.display = 'none';
  gameOverScreen.style.display = 'none';
  canvas.style.display = 'block';
  initGame();
  loop();
}

function drawBird() {
  ctx.drawImage(birdImg, bird.x, bird.y, bird.width, bird.height);
}

function drawPipes() {
  ctx.fillStyle = 'green';
  pipes.forEach(pipe => {
    ctx.fillRect(pipe.x, 0, pipe.width, pipe.top);
    ctx.fillRect(pipe.x, canvas.height - pipe.bottom, pipe.width, pipe.bottom);
  });
}

function update() {
  bird.velocity += gravity;
  bird.y += bird.velocity;

  if (bird.y + bird.height > canvas.height || bird.y < 0) {
    hitSound.play();
    gameOver = true;
  }

  pipes.forEach(pipe => {
    pipe.x -= 2;

    if (
      bird.x < pipe.x + pipe.width &&
      bird.x + bird.width > pipe.x &&
      (bird.y < pipe.top || bird.y + bird.height > canvas.height - pipe.bottom)
    ) {
      hitSound.play();
      gameOver = true;
    }

    if (!pipe.passed && pipe.x + pipe.width < bird.x) {
      score++;
      pipe.passed = true;
      pointSound.play();
    }
  });

  if (pipes.length === 0 || pipes[pipes.length - 1].x < 200) {
    let top = Math.random() * 200 + 50;
    let gap = 120;
    pipes.push({ x: canvas.width, width: 40, top: top, bottom: canvas.height - top - gap, passed: false });
  }

  pipes = pipes.filter(pipe => pipe.x + pipe.width > 0);
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBird();
  drawPipes();
  ctx.fillStyle = 'black';
  ctx.font = '20px Arial';
  ctx.fillText("Score: " + score, 10, 25);
}

function loop() {
  if (!gameOver) {
    update();
    draw();
    requestAnimationFrame(loop);
  } else {
    gameOverScreen.style.display = 'block';
    canvas.style.display = 'none';
  }
}

function flap() {
  if (!gameOver) {
    bird.velocity = jump;
    flapSound.play();
  }
}

document.addEventListener('keydown', flap);
canvas.addEventListener('click', flap); // for mouse/touch support
canvas.addEventListener('touchstart', flap);
