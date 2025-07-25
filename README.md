const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

// Game settings
const PADDLE_WIDTH = 15;
const PADDLE_HEIGHT = 100;
const BALL_RADIUS = 10;
const PLAYER_X = 20;
const AI_X = canvas.width - PADDLE_WIDTH - 20;
const PADDLE_SPEED = 7; // Only used for AI

// State
let playerY = (canvas.height - PADDLE_HEIGHT) / 2;
let aiY = (canvas.height - PADDLE_HEIGHT) / 2;
let ballX = canvas.width / 2;
let ballY = canvas.height / 2;
let ballSpeedX = 6 * (Math.random() > 0.5 ? 1 : -1);
let ballSpeedY = 4 * (Math.random() > 0.5 ? 1 : -1);
let playerScore = 0;
let aiScore = 0;

// Mouse control for player paddle
canvas.addEventListener('mousemove', (e) => {
  const rect = canvas.getBoundingClientRect();
  const scaleY = canvas.height / rect.height;
  let mouseY = (e.clientY - rect.top) * scaleY;
  playerY = mouseY - PADDLE_HEIGHT / 2;
  // Boundaries
  if (playerY < 0) playerY = 0;
  if (playerY > canvas.height - PADDLE_HEIGHT) playerY = canvas.height - PADDLE_HEIGHT;
});

// Drawing functions
function drawRect(x, y, w, h, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x, y, w, h);
}

function drawCircle(x, y, r, color) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x, y, r, 0, Math.PI * 2, false);
  ctx.closePath();
  ctx.fill();
}

function drawText(text, x, y, color) {
  ctx.fillStyle = color;
  ctx.font = "32px Arial";
  ctx.fillText(text, x, y);
}

// Collision detection
function collision(px, py, pw, ph, bx, by, br) {
  // Closest point on paddle to ball
  let closestX = Math.max(px, Math.min(bx, px + pw));
  let closestY = Math.max(py, Math.min(by, py + ph));
  let dx = bx - closestX;
  let dy = by - closestY;
  return (dx * dx + dy * dy) < br * br;
}

// Reset ball to center
function resetBall() {
  ballX = canvas.width / 2;
  ballY = canvas.height / 2;
  ballSpeedX = 6 * (Math.random() > 0.5 ? 1 : -1);
  ballSpeedY = 4 * (Math.random() > 0.5 ? 1 : -1);
}

// Game update
function update() {
  // Move ball
  ballX += ballSpeedX;
  ballY += ballSpeedY;

  // Top/bottom wall collision
  if (ballY - BALL_RADIUS < 0) {
    ballY = BALL_RADIUS;
    ballSpeedY *= -1;
  } else if (ballY + BALL_RADIUS > canvas.height) {
    ballY = canvas.height - BALL_RADIUS;
    ballSpeedY *= -1;
  }

  // Left paddle collision
  if (
    ballSpeedX < 0 &&
    collision(PLAYER_X, playerY, PADDLE_WIDTH, PADDLE_HEIGHT, ballX, ballY, BALL_RADIUS)
  ) {
    ballX = PLAYER_X + PADDLE_WIDTH + BALL_RADIUS;
    ballSpeedX *= -1.05; // Increase speed and reverse
    // Add a bit of randomness
    ballSpeedY += (Math.random() - 0.5) * 2;
  }

  // Right paddle collision (AI)
  if (
    ballSpeedX > 0 &&
    collision(AI_X, aiY, PADDLE_WIDTH, PADDLE_HEIGHT, ballX, ballY, BALL_RADIUS)
  ) {
    ballX = AI_X - BALL_RADIUS;
    ballSpeedX *= -1.05;
    ballSpeedY += (Math.random() - 0.5) * 2;
  }

  // Score left/right
  if (ballX - BALL_RADIUS < 0) {
    aiScore++;
    resetBall();
  } else if (ballX + BALL_RADIUS > canvas.width) {
    playerScore++;
    resetBall();
  }

  // AI paddle movement (simple AI)
  let aiCenter = aiY + PADDLE_HEIGHT / 2;
  if (aiCenter < ballY - 20) {
    aiY += PADDLE_SPEED;
  } else if (aiCenter > ballY + 20) {
    aiY -= PADDLE_SPEED;
  }
  // Boundaries
  if (aiY < 0) aiY = 0;
  if (aiY > canvas.height - PADDLE_HEIGHT) aiY = canvas.height - PADDLE_HEIGHT;
}

// Render everything
function render() {
  // Clear
  drawRect(0, 0, canvas.width, canvas.height, '#222');
  // Net
  for (let i = 15; i < canvas.height; i += 35) {
    drawRect(canvas.width / 2 - 2, i, 4, 20, '#fff');
  }
  // Paddles
  drawRect(PLAYER_X, playerY, PADDLE_WIDTH, PADDLE_HEIGHT, '#fff');
  drawRect(AI_X, aiY, PADDLE_WIDTH, PADDLE_HEIGHT, '#fff');
  // Ball
  drawCircle(ballX, ballY, BALL_RADIUS, '#fff');
  // Score
  drawText(playerScore, canvas.width / 2 - 60, 40, '#fff');
  drawText(aiScore, canvas.width / 2 + 40, 40, '#fff');
}

// Game loop
function gameLoop() {
  update();
  render();
  requestAnimationFrame(gameLoop);
}

// Start game
gameLoop();
