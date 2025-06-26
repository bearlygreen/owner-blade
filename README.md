<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>Legends of the Five Cities - Owner Version</title>
<style>
body { margin: 0; font-family: Arial, sans-serif; text-align: center; color: #fff; background: #111; }
canvas { display: block; margin: auto; background: #222; border: 2px solid #555; }
h2, p { margin: 8px; }
#adminPanel {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 10px;
  background: rgba(0,0,0,0.8);
  color: #0f0;
  font-size: 14px;
  display: none;
}
</style>
</head>
<body>
<h2>Legends of the Five Cities - Owner Version (James)</h2>
<canvas id="game" width="900" height="600"></canvas>
<p>Move: ⬅️ ➡️ ⬆️ ⬇️ | Attack: Space | Level Up: Defeat Enemy | Owner/Admin Panel: Press 'A'</p>
<div id="adminPanel">
  <strong>Admin Panel</strong><br/>
  H = Heal to Max<br/>
  K = Kill All Enemies<br/>
  L = Level Up
</div>
<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let player = { x: 100, y: 100, w: 30, h: 30, hp: 200, level: 1, speed: 3 };
let keys = {};
let attackCooldown = 0;

const buildings = [
  { x: 200, y: 50, w: 200, h: 100 },
  { x: 500, y: 50, w: 250, h: 150 },
  { x: 100, y: 250, w: 180, h: 150 },
  { x: 400, y: 300, w: 300, h: 200 },
  { x: 750, y: 400, w: 120, h: 150 }
];

const enemies = [];
const ENEMY_COUNT = 6;

for (let i = 0; i < ENEMY_COUNT; i++) {
  enemies.push({
    x: Math.random() * (canvas.width - 30),
    y: Math.random() * (canvas.height - 30),
    w: 30,
    h: 30,
    hp: 100,
    speed: 1 + Math.random() * 1.5,
    directionX: Math.random() < 0.5 ? 1 : -1,
    directionY: Math.random() < 0.5 ? 1 : -1
  });
}

function drawRect(obj, color) {
  ctx.fillStyle = color;
  ctx.fillRect(obj.x, obj.y, obj.w, obj.h);
}
function drawText(text, x, y, color) {
  ctx.fillStyle = color;
  ctx.font = "16px Arial";
  ctx.fillText(text, x, y);
}
function isColliding(a, b) {
  return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
}
function collidesWithBuildings(rect) {
  for (const b of buildings) {
    if (isColliding(rect, b)) return true;
  }
  return false;
}
function movePlayer() {
  let newX = player.x;
  let newY = player.y;

  if (keys["ArrowLeft"]) newX -= player.speed;
  if (keys["ArrowRight"]) newX += player.speed;
  if (keys["ArrowUp"]) newY -= player.speed;
  if (keys["ArrowDown"]) newY += player.speed;

  const newRect = { x: newX, y: newY, w: player.w, h: player.h };
  if (!collidesWithBuildings(newRect) &&
      newX >= 0 && newX + player.w <= canvas.width &&
      newY >= 0 && newY + player.h <= canvas.height) {
    player.x = newX;
    player.y = newY;
  }
}
function moveEnemies() {
  enemies.forEach(enemy => {
    let newX = enemy.x + enemy.speed * enemy.directionX;
    let newY = enemy.y + enemy.speed * enemy.directionY;
    const newRect = { x: newX, y: newY, w: enemy.w, h: enemy.h };
    if (collidesWithBuildings(newRect) || newX < 0 || newX + enemy.w > canvas.width) {
      enemy.directionX *= -1;
    } else {
      enemy.x = newX;
    }
    if (collidesWithBuildings(newRect) || newY < 0 || newY + enemy.h > canvas.height) {
      enemy.directionY *= -1;
    } else {
      enemy.y = newY;
    }
  });
}
function attackEnemies() {
  if (keys[" "] && attackCooldown === 0) {
    enemies.forEach(enemy => {
      if (enemy.hp > 0 && isColliding(player, enemy)) {
        enemy.hp -= 20;
        attackCooldown = 30;

        if (enemy.hp <= 0) {
          player.level++;
          player.hp += 20;
          enemy.x = Math.random() * (canvas.width - enemy.w);
          enemy.y = Math.random() * (canvas.height - enemy.h);
          enemy.hp = 100;
        }
      }
    });
  }
  attackCooldown = Math.max(0, attackCooldown - 1);
}
function drawBuildings() {
  ctx.font = "18px Arial";
  buildings.forEach((b, i) => {
    drawRect(b, "#444");
    ctx.fillStyle = "#ccc";
    const names = [
      "City of Boom",
      "Goo City",
      "Boxing City",
      "Deku City",
      "Zombie Town"
    ];
    ctx.fillText(names[i], b.x + 10, b.y + 30);
  });
}
function drawEnemies() {
  enemies.forEach(enemy => {
    if (enemy.hp > 0) {
      drawRect(enemy, "#f00");
      drawText(`HP: ${enemy.hp}`, enemy.x, enemy.y - 5, "white");
    }
  });
}
function drawPlayerAndUI() {
  drawRect(player, "#0f0");
  drawText(`James (Owner) | HP: ${player.hp}`, player.x, player.y - 10, "white");
  drawText(`Level: ${player.level}`, 10, 20, "white");
}
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBuildings();
  drawPlayerAndUI();
  drawEnemies();
}
function gameLoop() {
  movePlayer();
  moveEnemies();
  attackEnemies();
  draw();
  requestAnimationFrame(gameLoop);
}

// Admin Functions
function adminHeal() {
  player.hp = 9999;
}
function killAllEnemies() {
  enemies.forEach(enemy => enemy.hp = 0);
}
function levelUp() {
  player.level += 1;
  player.hp += 20;
}

// Event Listeners
window.addEventListener("keydown", e => {
  keys[e.key] = true;

  // Admin Panel Toggle
  if (e.key.toLowerCase() === "a") {
    const panel = document.getElementById("adminPanel");
    panel.style.display = panel.style.display === "none" ? "block" : "none";
  }

  // Admin Actions
  if (e.key.toLowerCase() === "h") adminHeal();
  if (e.key.toLowerCase() === "k") killAllEnemies();
  if (e.key.toLowerCase() === "l") levelUp();
});
window.addEventListener("keyup", e => keys[e.key] = false);

// Start Game
gameLoop();
</script>
</body>
</html>
