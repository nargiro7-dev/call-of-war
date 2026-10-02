const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const overlay = document.getElementById('overlay');
const missionOverlay = document.getElementById('missionOverlay');
const startButton = document.getElementById('startButton');
const nextMissionButton = document.getElementById('nextMissionButton');

const missionLabel = document.getElementById('mission-label');
const missionName = document.getElementById('mission-name');
const statusEl = document.getElementById('status');
const healthEl = document.getElementById('health');
const ammoEl = document.getElementById('ammo');

const missionTitle = document.getElementById('missionTitle');
const missionText = document.getElementById('missionText');

const WORLD_W = 16;
const WORLD_H = 16;
const PLAYER_SPEED = 2.2;
const STRAFE_SPEED = 1.9;
const TURN_SPEED = 2.25;
const FOV = Math.PI / 3;
const MAX_DEPTH = 18;
const CROSSHAIR_SIZE = 18;

const keys = {};
let mouseDown = false;
let fireCooldown = 0;
let lastTime = 0;
let active = false;
let gameOver = false;
let missionIndex = 0;

const MISSION_DATA = [
  {
    title: 'Dawn at Anzac Cove',
    brief: 'Take the beach and reach the ridge.',
    start: { x: 2.5, y: 2.5, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 6.7, y: 3.2, hp: 1, alive: true },
      { x: 9.3, y: 7.1, hp: 1, alive: true },
      { x: 11.6, y: 10.2, hp: 1, alive: true }
    ],
    ammo: 12,
    objective: 'Clear the beach position.'
  },
  {
    title: 'Shrapnel Valley',
    brief: 'Push inland and secure the gully.',
    start: { x: 2.4, y: 12.2, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111101101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 6.2, y: 4.7, hp: 1, alive: true },
      { x: 9.8, y: 6.4, hp: 1, alive: true },
      { x: 12.6, y: 8.4, hp: 1, alive: true },
      { x: 8.0, y: 11.5, hp: 1, alive: true },
      { x: 13.3, y: 2.6, hp: 1, alive: true }
    ],
    ammo: 14,
    objective: 'Clear the valley and hold the ridge.'
  },
  {
    title: 'Lone Pine',
    brief: 'Fight through the trench line.',
    start: { x: 2.5, y: 13.5, a: 0 },
    map: [
      '1111111111111111',
      '1010000100000101',
      '1011110101110101',
      '1000010000010001',
      '1011011101011101',
      '1001000000010001',
      '1011011111011101',
      '1000000100000001',
      '1011110111101101',
      '1000001000010001',
      '1011111001011101',
      '1000010000000001',
      '1011010101111101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 5.5, y: 3.8, hp: 2, alive: true },
      { x: 9.1, y: 5.6, hp: 2, alive: true },
      { x: 7.8, y: 9.6, hp: 2, alive: true },
      { x: 11.5, y: 11.4, hp: 2, alive: true }
    ],
    ammo: 14,
    objective: 'Break through the trench line.'
  },
  {
    title: 'Night Attack on the Sari Bair Range',
    brief: 'Move under cover of darkness.',
    start: { x: 1.6, y: 12.8, a: 0 },
    map: [
      '1111111111111111',
      '1000000010000001',
      '1011111010111101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 6.8, y: 3.0, hp: 2, alive: true },
      { x: 10.5, y: 4.2, hp: 2, alive: true },
      { x: 8.4, y: 8.1, hp: 2, alive: true },
      { x: 12.6, y: 9.2, hp: 2, alive: true },
      { x: 14.0, y: 12.0, hp: 2, alive: true }
    ],
    ammo: 16,
    objective: 'Reach the ravine and drive forward.'
  },
  {
    title: 'The Nek',
    brief: 'Charge across the exposed ground.',
    start: { x: 2.2, y: 13.2, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 5.0, y: 3.4, hp: 2, alive: true },
      { x: 7.8, y: 5.1, hp: 2, alive: true },
      { x: 10.7, y: 7.8, hp: 2, alive: true },
      { x: 12.7, y: 5.3, hp: 2, alive: true },
      { x: 9.3, y: 11.5, hp: 2, alive: true },
      { x: 13.5, y: 12.5, hp: 2, alive: true }
    ],
    ammo: 18,
    objective: 'Break through the line and survive the assault.'
  },
  {
    title: 'Holding the Hill',
    brief: 'Defend the dug-in position.',
    start: { x: 2.2, y: 12.8, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 7.3, y: 2.8, hp: 2, alive: true },
      { x: 10.8, y: 4.1, hp: 2, alive: true },
      { x: 12.0, y: 7.3, hp: 2, alive: true },
      { x: 8.9, y: 10.8, hp: 2, alive: true },
      { x: 5.6, y: 12.1, hp: 2, alive: true }
    ],
    ammo: 18,
    objective: 'Hold the hill against the counterattack.'
  },
  {
    title: 'The Khaki Trenches',
    brief: 'Raid the enemy trench system.',
    start: { x: 2.4, y: 11.8, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 4.6, y: 4.0, hp: 2, alive: true },
      { x: 8.1, y: 5.5, hp: 2, alive: true },
      { x: 11.9, y: 7.0, hp: 2, alive: true },
      { x: 7.6, y: 10.4, hp: 2, alive: true },
      { x: 12.7, y: 11.7, hp: 2, alive: true },
      { x: 14.0, y: 3.2, hp: 2, alive: true }
    ],
    ammo: 18,
    objective: 'Infiltrate and destroy the supply line.'
  },
  {
    title: 'The Guns of Gallipoli',
    brief: 'Break through the artillery line.',
    start: { x: 2.0, y: 13.0, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 5.5, y: 4.0, hp: 3, alive: true },
      { x: 9.6, y: 4.4, hp: 3, alive: true },
      { x: 11.4, y: 8.1, hp: 3, alive: true },
      { x: 8.4, y: 12.0, hp: 3, alive: true },
      { x: 13.2, y: 11.4, hp: 3, alive: true }
    ],
    ammo: 20,
    objective: 'Storm the artillery positions and hold the ground.'
  },
  {
    title: 'The Final Approach',
    brief: 'Push toward the coast.',
    start: { x: 2.4, y: 12.2, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 6.0, y: 3.7, hp: 3, alive: true },
      { x: 8.5, y: 5.3, hp: 3, alive: true },
      { x: 11.3, y: 6.7, hp: 3, alive: true },
      { x: 9.8, y: 9.6, hp: 3, alive: true },
      { x: 12.9, y: 12.3, hp: 3, alive: true },
      { x: 5.5, y: 11.2, hp: 3, alive: true }
    ],
    ammo: 22,
    objective: 'Drive the final approach and hold the line.'
  },
  {
    title: 'The Last Boat',
    brief: 'The campaign ends in silence and survival.',
    start: { x: 2.2, y: 13.5, a: 0 },
    map: [
      '1111111111111111',
      '1000000000000001',
      '1011110111110101',
      '1000010000100001',
      '1011010101101101',
      '1001000100100001',
      '1011010111101101',
      '1000000000100001',
      '1011110110101101',
      '1000000100000001',
      '1011110101111101',
      '1000010000010001',
      '1011011101011101',
      '1000000000000001',
      '1111111111111111'
    ],
    enemies: [
      { x: 5.2, y: 3.0, hp: 3, alive: true },
      { x: 8.1, y: 4.7, hp: 3, alive: true },
      { x: 10.8, y: 5.4, hp: 3, alive: true },
      { x: 13.0, y: 8.1, hp: 3, alive: true },
      { x: 8.7, y: 10.2, hp: 3, alive: true },
      { x: 5.3, y: 12.0, hp: 3, alive: true },
      { x: 12.2, y: 12.5, hp: 3, alive: true }
    ],
    ammo: 24,
    objective: 'Hold the beach long enough to escape.'
  }
];

const state = {
  player: { x: 2.5, y: 2.5, angle: 0, health: 100 },
  enemies: [],
  map: [],
  ammo: 12,
  status: 'Hold the line',
  missionComplete: false,
  startTimer: 0
};

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

function isWall(x, y) {
  const gridX = Math.floor(x);
  const gridY = Math.floor(y);
  if (gridX < 0 || gridY < 0 || gridX >= WORLD_W || gridY >= WORLD_H) return true;
  return state.map[gridY][gridX] === 1;
}

function parseMap(layout) {
  return layout.map(row => row.split('').map(ch => ch === '1' ? 1 : 0));
}

function resetMission(index) {
  const mission = MISSION_DATA[index];
  state.map = parseMap(mission.map);
  state.player.x = mission.start.x;
  state.player.y = mission.start.y;
  state.player.angle = mission.start.a;
  state.player.health = 100;
  state.ammo = mission.ammo;
  state.status = mission.objective;
  state.missionComplete = false;
  state.enemies = mission.enemies.map(enemy => ({ ...enemy }));

  missionLabel.textContent = `Mission ${index + 1}`;
  missionName.textContent = mission.title;
  statusEl.textContent = mission.objective;
  healthEl.textContent = `Health: ${state.player.health}`;
  ammoEl.textContent = `Ammo: ${state.ammo}`;
}

function showMissionText(title, text) {
  missionTitle.textContent = title;
  missionText.textContent = text;
  missionOverlay.classList.remove('hidden');
  missionOverlay.classList.add('visible');
  active = false;
}

function hideMissionText() {
  missionOverlay.classList.add('hidden');
  missionOverlay.classList.remove('visible');
  active = true;
}

function completeMission() {
  if (state.missionComplete) return;
  state.missionComplete = true;

  if (missionIndex >= MISSION_DATA.length - 1) {
    statusEl.textContent = 'Campaign complete';
    showMissionText('The Campaign Ends', 'John Wilson survives Gallipoli. The coast remains only a memory, but the men who fell are never forgotten.');
    nextMissionButton.textContent = 'Play Again';
    return;
  }

  const currentMission = MISSION_DATA[missionIndex];
  const nextTitle = `Mission ${missionIndex + 2}`;
  const nextText = `${MISSION_DATA[missionIndex + 1].title}: ${MISSION_DATA[missionIndex + 1].brief}`;
  showMissionText(nextTitle, nextText);
  nextMissionButton.textContent = 'Continue';
}

function startGame() {
  missionIndex = 0;
  resetMission(missionIndex);
  overlay.classList.add('hidden');
  overlay.classList.remove('visible');
  hideMissionText();
  showMissionText(`Mission ${missionIndex + 1}`, `${MISSION_DATA[missionIndex].title}: ${MISSION_DATA[missionIndex].brief}`);
  nextMissionButton.textContent = 'Continue';
  gameOver = false;
}

function startMissionFromOverlay() {
  const mission = MISSION_DATA[missionIndex];
  hideMissionText();
  if (missionIndex === 0 && !active) {
    active = true;
  }
}

function handleInput() {
  if (!active || gameOver) return;

  let moveX = 0;
  let moveY = 0;

  if (keys['w'] || keys['arrowup']) {
    moveX += Math.cos(state.player.angle) * PLAYER_SPEED;
    moveY += Math.sin(state.player.angle) * PLAYER_SPEED;
  }
  if (keys['s'] || keys['arrowdown']) {
    moveX -= Math.cos(state.player.angle) * PLAYER_SPEED;
    moveY -= Math.sin(state.player.angle) * PLAYER_SPEED;
  }
  if (keys['a']) {
    moveX += Math.cos(state.player.angle + Math.PI / 2) * STRAFE_SPEED;
    moveY += Math.sin(state.player.angle + Math.PI / 2) * STRAFE_SPEED;
  }
  if (keys['d']) {
    moveX += Math.cos(state.player.angle - Math.PI / 2) * STRAFE_SPEED;
    moveY += Math.sin(state.player.angle - Math.PI / 2) * STRAFE_SPEED;
  }
  if (keys['q']) {
    state.player.angle -= TURN_SPEED * 0.016;
  }
  if (keys['e']) {
    state.player.angle += TURN_SPEED * 0.016;
  }
  if (keys['arrowleft']) {
    state.player.angle -= TURN_SPEED * 0.016;
  }
  if (keys['arrowright']) {
    state.player.angle += TURN_SPEED * 0.016;
  }

  const moveLen = Math.hypot(moveX, moveY);
  if (moveLen > 0) {
    const nx = moveX / moveLen;
    const ny = moveY / moveLen;
    const tryX = state.player.x + nx * 0.1;
    const tryY = state.player.y + ny * 0.1;

    if (!isWall(tryX, state.player.y)) state.player.x = tryX;
    if (!isWall(state.player.x, tryY)) state.player.y = tryY;
  }

  if (fireCooldown > 0) fireCooldown -= 0.016;

  if (mouseDown && fireCooldown <= 0 && state.ammo > 0) {
    fireWeapon();
  }
}

function fireWeapon() {
  if (!active || gameOver) return;

  const shotAngle = state.player.angle;
  let bestTarget = null;
  let bestDistance = Infinity;

  for (const enemy of state.enemies) {
    if (!enemy.alive) continue;
    const dx = enemy.x - state.player.x;
    const dy = enemy.y - state.player.y;
    const dist = Math.hypot(dx, dy);
    const angleToEnemy = Math.atan2(dy, dx);
    const delta = Math.abs(Math.atan2(Math.sin(angleToEnemy - shotAngle), Math.cos(angleToEnemy - shotAngle)));

    if (dist < bestDistance && delta < 0.12 && dist < 7) {
      bestTarget = enemy;
      bestDistance = dist;
    }
  }

  if (bestTarget) {
    bestTarget.hp -= 1;
    if (bestTarget.hp <= 0) {
      bestTarget.alive = false;
      state.status = 'Enemy neutralized';
    }
  } else {
    state.status = 'No target';
  }

  state.ammo = Math.max(0, state.ammo - 1);
  fireCooldown = 0.28;
  ammoEl.textContent = `Ammo: ${state.ammo}`;

  if (state.enemies.every(enemy => !enemy.alive)) {
    completeMission();
  }
}

function updateEnemyAI(dt) {
  for (const enemy of state.enemies) {
    if (!enemy.alive) continue;

    const dx = state.player.x - enemy.x;
    const dy = state.player.y - enemy.y;
    const dist = Math.hypot(dx, dy);

    if (dist < 1.8) {
      const angleToPlayer = Math.atan2(dy, dx);
      const aimError = Math.abs(Math.atan2(Math.sin(angleToPlayer - enemy.dir), Math.cos(angleToPlayer - enemy.dir)));
      if (!enemy.dir) enemy.dir = Math.random() * Math.PI * 2;
      if (dist < 1.5) {
        state.player.health = clamp(state.player.health - 8 * dt, 0, 100);
        healthEl.textContent = `Health: ${Math.max(0, state.player.health)}`;
      }
      if (state.player.health <= 0) {
        gameOver = true;
        active = false;
        statusEl.textContent = 'You fell at Gallipoli';
        showMissionText('Mission Failed', 'The line has fallen. The campaign ends with a soldier who could not make it back home.');
        nextMissionButton.textContent = 'Restart';
      }
    }
  }
}

function drawBackground() {
  const sky = ctx.createLinearGradient(0, 0, 0, canvas.height / 2);
  sky.addColorStop(0, '#8a9aa0');
  sky.addColorStop(1, '#445d62');
  ctx.fillStyle = sky;
  ctx.fillRect(0, 0, canvas.width, canvas.height / 2);

  const ground = ctx.createLinearGradient(0, canvas.height / 2, 0, canvas.height);
  ground.addColorStop(0, '#5d4b36');
  ground.addColorStop(1, '#2d221a');
  ctx.fillStyle = ground;
  ctx.fillRect(0, canvas.height / 2, canvas.width, canvas.height / 2);
}

function castRay(angle) {
  const rayX = state.player.x;
  const rayY = state.player.y;
  let distance = 0;
  while (distance < MAX_DEPTH) {
    const checkX = rayX + Math.cos(angle) * distance;
    const checkY = rayY + Math.sin(angle) * distance;
    if (isWall(checkX, checkY)) {
      return distance;
    }
    distance += 0.05;
  }
  return MAX_DEPTH;
}

function drawWalls() {
  const numRays = canvas.width;
  for (let i = 0; i < numRays; i += 2) {
    const screenX = i;
    const rayAngle = state.player.angle - FOV / 2 + (i / numRays) * FOV;
    const dist = castRay(rayAngle);
    const wallHeight = (canvas.height / dist) * 1.8;
    const shade = clamp(1 - dist / MAX_DEPTH, 0.12, 1);

    ctx.fillStyle = `rgba(${Math.floor(150 * shade)}, ${Math.floor(110 * shade)}, ${Math.floor(80 * shade)}, 1)`;
    ctx.fillRect(screenX, (canvas.height - wallHeight) / 2, 2, wallHeight);
  }
}

function drawEnemies() {
  const visible = [];

  for (const enemy of state.enemies) {
    if (!enemy.alive) continue;
    const dx = enemy.x - state.player.x;
    const dy = enemy.y - state.player.y;
    const dist = Math.hypot(dx, dy);
    const angleToEnemy = Math.atan2(dy, dx);
    const delta = Math.atan2(Math.sin(angleToEnemy - state.player.angle), Math.cos(angleToEnemy - state.player.angle));

    if (Math.abs(delta) < FOV / 1.5 && dist < MAX_DEPTH) {
      visible.push({ enemy, dist, delta });
    }
  }

  visible.sort((a, b) => b.dist - a.dist);

  for (const { enemy, dist, delta } of visible) {
    const screenX = ((delta + FOV / 2) / FOV) * canvas.width;
    const spriteSize = clamp((canvas.height / dist) * 0.8, 18, 150);
    const x = screenX - spriteSize / 2;
    const y = canvas.height / 2 - spriteSize / 2;

    ctx.fillStyle = '#8b1d1d';
    ctx.fillRect(x, y, spriteSize, spriteSize);

    ctx.fillStyle = '#f1d1b5';
    ctx.fillRect(x + spriteSize * 0.2, y + spriteSize * 0.2, spriteSize * 0.6, spriteSize * 0.6);
  }
}

function drawPlayerReticle() {
  ctx.strokeStyle = 'rgba(255,255,255,0.7)';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(canvas.width / 2 - CROSSHAIR_SIZE, canvas.height / 2);
  ctx.lineTo(canvas.width / 2 + CROSSHAIR_SIZE, canvas.height / 2);
  ctx.moveTo(canvas.width / 2, canvas.height / 2 - CROSSHAIR_SIZE);
  ctx.lineTo(canvas.width / 2, canvas.height / 2 + CROSSHAIR_SIZE);
  ctx.stroke();
}

function drawScene() {
  drawBackground();
  drawWalls();
  drawEnemies();
  drawPlayerReticle();
}

function animate(timestamp) {
  const dt = (timestamp - lastTime) / 1000 || 0.016;
  lastTime = timestamp;

  handleInput();
  updateEnemyAI(dt);
  drawScene();
  requestAnimationFrame(animate);
}

window.addEventListener('keydown', (event) => {
  const key = event.key.toLowerCase();
  keys[key] = true;

  if (key === ' ') {
    event.preventDefault();
    if (active && !gameOver && state.ammo > 0) fireWeapon();
  }
});

window.addEventListener('keyup', (event) => {
  keys[event.key.toLowerCase()] = false;
});

window.addEventListener('mousedown', () => {
  mouseDown = true;
  if (active && !gameOver) fireWeapon();
});

window.addEventListener('mouseup', () => {
  mouseDown = false;
});

nextMissionButton.addEventListener('click', () => {
  if (gameOver) {
    missionIndex = 0;
    startGame();
    return;
  }

  if (missionIndex >= MISSION_DATA.length - 1) {
    missionIndex = 0;
    startGame();
    return;
  }

  missionIndex += 1;
  resetMission(missionIndex);
  hideMissionText();
  active = true;
  statusEl.textContent = MISSION_DATA[missionIndex].objective;
});

startButton.addEventListener('click', () => {
  startGame();
});

resetMission(0);
requestAnimationFrame(animate);
