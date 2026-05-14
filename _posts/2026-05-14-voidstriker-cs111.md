---
layout: post
title: "CS111 Requirements — Void Striker Game Level"
permalink: /voidstriker-cs111
---

# CS111 Requirements — Void Striker Game Level

Evidence of all CS111 learning objectives demonstrated in `GameLevelVoidStriker.js`.

---

## Object-Oriented Programming

### Writing Classes

The file defines two custom classes. `GameLevelVoidStriker` serves as the top-level level class consumed by the game engine, and `VoidStrikerGame` is structured as an IIFE module encapsulating the entire game.

```js
class GameLevelVoidStriker {
  constructor(gameEnv) {
    let width  = gameEnv.innerWidth;
    let height = gameEnv.innerHeight;
    let path   = gameEnv.path;

    // ...

    this.classes = [
      { class: GameEnvBackground, data: image_data_space },
    ];
  }
}
```

### Methods & Parameters

Methods throughout the game accept multiple parameters. `spawnExplosion` takes `x`, `y`, and `color`; `fireDirected` takes directional vector components `sx` and `sy`.

```js
function spawnExplosion(x, y, color) {
  for (let i = 0; i < 18; i++) {
    const angle = rand(0, Math.PI * 2);
    const speed = rand(1, 5);
    particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      r:  rand(1.5, 4),
      alpha: 1,
      color,
    });
  }
}
```

```js
function fireDirected(sx, sy) {
  const len = Math.hypot(sx, sy) || 1;
  const nx = sx / len, ny = sy / len;
  const angle = Math.atan2(ny, nx);
  // ...
}
```

### Instantiation & Objects

Game objects (`ship`, `enemies`, `asteroids`, `boss`, `bullets`, `particles`) are instantiated as plain objects with defined properties inside their respective builder/spawner functions.

```js
function buildShip() {
  ship = {
    x: W / 2, y: H * 0.78,
    w: 28, h: 38,
    speed: SHIP_CHARS[activeChar].speed,
    shootCooldown: 0,
    invincible: 0,
    thrustFlicker: 0,
  };
}
```

```js
boss = {
  x: W / 2,
  y: -80,
  r: 40 + tier * 2,
  hp,
  maxHp: hp,
  speed: 2.7 + tier * 0.45,
  pulse: 0,
  tier,
  palette: BOSS_PALETTES[tier % BOSS_PALETTES.length],
};
```

### Inheritance (Basic)

`GameLevelVoidStriker` imports and uses `GameEnvBackground`, placing it inside `this.classes` so the game engine can instantiate it via the standard class hierarchy (`GameEnvBackground` extends the engine's base class).

```js
import GameEnvBackground from '@assets/js/GameEnginev1.1/essentials/GameEnvBackground.js';

// Inside constructor:
this.classes = [
  { class: GameEnvBackground, data: image_data_space },
];
```

### Constructor Chaining

The `GameLevelVoidStriker` constructor receives and immediately uses `gameEnv` (passed from the engine), which represents a chained initialization pattern — the engine calls `new GameLevelVoidStriker(gameEnv)` and this constructor delegates further setup to `VoidStrikerGame.init(gameEnv)`.

```js
constructor(gameEnv) {
  let width  = gameEnv.innerWidth;
  let height = gameEnv.innerHeight;
  let path   = gameEnv.path;
  // ...
  setTimeout(() => VoidStrikerGame.init(gameEnv), 200);
}
```

---

## Control Structures

### Iteration

Multiple loop styles are used throughout. `forEach` iterates star layers and particles; a traditional `for` loop iterates the collision detection arrays in reverse to safely splice elements.

```js
// forEach on layer arrays (updateLayers)
layerFarStars.forEach(s => { s.y = (s.y + s.speed) % H; });
layerMidStars.forEach(s => {
  s.y        = (s.y + s.speed) % H;
  s.twinkleT += s.twinkleRate;
});
```

```js
// Reverse for-loop for safe splice during collision checks
for (let bi = bullets.length - 1; bi >= 0; bi--) {
  const b = bullets[bi];
  if (b.life <= 0) continue;
  // ...
}
```

```js
// for-loop for boss tentacle drawing
for (let i = 0; i < 6; i++) {
  const a = (i / 6) * Math.PI * 2 + Math.sin(boss.pulse + i) * 0.2;
  ctx.beginPath();
  ctx.moveTo(Math.cos(a) * r * 0.7, Math.sin(a) * r * 0.7);
  ctx.lineTo(Math.cos(a) * r * 1.5, Math.sin(a) * r * 1.5);
  ctx.stroke();
}
```

### Conditionals

State transitions and game logic are controlled throughout with `if/else`.

```js
if (gameState === 'playing' && !consoleActive) {
  updateShip();
  updateBullets();
  updateEnemies();
  updateBoss();
  updateParticles();
  checkCollisions();
  updateHUD();
  // ...
  if (lives <= 0) {
    gameState = 'dead';
    showDeadScreen();
  }
} else if (gameState === 'playing' && consoleActive) {
  // Game paused — draw last frame frozen
  drawEnemies();
  drawBoss();
  drawBullets();
  drawParticles();
  drawShip();
}
```

### Nested Conditions

Complex game logic uses multi-level conditionals. The collision handler checks invincibility state, then checks bullet-hits, then contact-hits, all with nested conditions:

```js
if (ship.invincible <= 0) {
  for (let bi = enemyBullets.length - 1; bi >= 0; bi--) {
    const b = enemyBullets[bi];
    const dx = ship.x - b.x, dy = ship.y - b.y;
    if (Math.sqrt(dx * dx + dy * dy) < 20) {
      enemyBullets.splice(bi, 1);
      lives--;
      ship.invincible = 90;
      spawnExplosion(ship.x, ship.y, '#00eeff');
      if (lives <= 0) gameState = 'dead';
      break;
    }
  }
}
```

The enemy chaser logic also uses nested conditions to choose between chasing and straight-line movement, then further nests shooting behavior:

```js
if (e.chaser) {
  const dx = ship.x - e.x;
  const dy = ship.y - e.y;
  const dist = Math.hypot(dx, dy) || 1;
  e.chaseAcc = Math.min(e.speed, e.chaseAcc + 0.0006);
  const spd = (e.speed + e.chaseAcc) * worldSpeed;
  e.x += (dx / dist) * spd;
  e.y += (dy / dist) * spd;
  e.angle = Math.atan2(dy, dx);
} else {
  e.y  += e.speed * worldSpeed;
  e.x  += e.vx    * worldSpeed;
  if (e.x < 20 || e.x > W - 20) e.vx *= -1;
}

if (e.shootTimer !== Infinity) {
  e.shootTimer -= worldSpeed;
  if (e.shootTimer <= 0) {
    enemyBullets.push({ x: e.x, y: e.y + e.r, vx: 0, vy: 4 + wave * 0.3, life: 80 });
    e.shootTimer = randI(60, 140);
  }
}
```

---

## Data Types

### Numbers

Position, velocity, size, health, and score are all tracked numerically.

```js
let wave = 1, lives = 3;
let bestKills = 0;
let totalKills = 0;
let worldSpeed = 1;

ship = {
  x: W / 2, y: H * 0.78,
  w: 28, h: 38,
  speed: SHIP_CHARS[activeChar].speed,
  shootCooldown: 0,
  invincible: 0,
};
```

### Strings

Character names, sprite state strings, background scene labels, and HSL color strings are all stored and manipulated.

```js
let gameState = 'title'; // 'title' | 'playing' | 'dead'

const BG_SCENES  = ['nebula', 'deepspace', 'supernova'];

const SHIP_CHARS = [
  { name: 'Striker',  body: '#a0d8ff', cockpit: '#00eeff', thrustRgb: '0,200,255',  bullet: '#00eeff', speed: 4.5 },
  { name: 'Shadow',   body: '#bb99ee', cockpit: '#cc55ff', thrustRgb: '160,0,255',  bullet: '#cc55ff', speed: 5.2 },
  // ...
];

// Dynamic color string from numeric hue
color: `hsl(${(hueBase + randI(-15, 15) + 360) % 360},85%,55%)`
```

### Booleans

Boolean flags control invincibility, shooting cooldown checks, enemy type, and pause state.

```js
let consoleActive = false;

const isChaser = wave >= 2 && Math.random() < Math.min(0.45, 0.08 * wave);

// Boolean used as a guard in collision detection
if (ship.invincible <= 0) { ... }

// Boolean expression controls shooting
if (ship.shootCooldown === 0) {
  let sx = 0, sy = 0;
  if (keys['ArrowLeft'])  sx -= 1;
  if (keys['ArrowRight']) sx += 1;
  if (sx !== 0 || sy !== 0) {
    fireDirected(sx, sy);
    ship.shootCooldown = 12;
  }
}
```

### Arrays

All game object collections are arrays, and `Array.from` generates star and asteroid data.

```js
let bullets = [], enemies = [], asteroids = [], particles = [];
let enemyBullets = [];

layerFarStars = Array.from({ length: 200 }, () => ({
  x:     rand(0, W),
  y:     rand(0, H),
  r:     rand(0.3, 0.9),
  alpha: rand(0.2, 0.5),
  speed: rand(0.08, 0.18),
}));

// Array filter removes dead bullets each frame
bullets = bullets.filter(b => b.life-- > 0);
particles = particles.filter(p => p.alpha > 0.02);
```

### Objects (JSON)

Configuration objects define character skins, boss palettes, and background cloud descriptors.

```js
const SHIP_CHARS = [
  { name: 'Striker',  body: '#a0d8ff', cockpit: '#00eeff', thrustRgb: '0,200,255',  bullet: '#00eeff', speed: 4.5 },
  { name: 'Shadow',   body: '#bb99ee', cockpit: '#cc55ff', thrustRgb: '160,0,255',  bullet: '#cc55ff', speed: 5.2 },
  { name: 'Inferno',  body: '#ffbb77', cockpit: '#ff5500', thrustRgb: '255,100,0',  bullet: '#ff6600', speed: 4.0 },
  { name: 'Nova',     body: '#99ffcc', cockpit: '#00ff99', thrustRgb: '0,255,140',  bullet: '#00ff99', speed: 4.8 },
];

const image_data_space = {
  id:     'VoidStriker-Background',
  src:    '',
  pixels: { height: 570, width: 1025 }
};
```

---

## Operators

### Mathematical

Physics calculations use all four arithmetic operators — gravity accumulates on asteroids, velocity is updated each frame, and distances are computed with `Math.sqrt` / `Math.hypot`.

```js
// Gravity system on asteroids
a.verticalVelocity -= a.gravityAcceleration;
if (-a.verticalVelocity > a.terminalVelocity) {
  a.verticalVelocity = -a.terminalVelocity;
}
a.y += -a.verticalVelocity * worldSpeed;

// Boss pursuit: distance formula + trig
const dx = ship.x - boss.x;
const dy = ship.y - boss.y;
const angle = Math.atan2(dy, dx);
boss.x += Math.cos(angle) * boss.speed;
boss.y += Math.sin(angle) * boss.speed;
```

### String Operations

Template literals are used to build dynamic CSS colors, HUD labels, and HTML content.

```js
// Dynamic RGBA color string
glow.addColorStop(0, `rgba(${p.glow},0.55)`);
fill.style.background = `linear-gradient(90deg, rgba(${p.glow},1), ${p.bodyHi})`;

// HUD text
s.textContent = `KILLS: ${totalKills}`;
w.textContent = `WAVE: ${wave}`;

// HSL color from numeric hue band
color: `hsl(${(hueBase + randI(-15, 15) + 360) % 360},85%,55%)`
```

### Boolean Expressions

Compound `&&` and `||` conditions are used throughout for multi-condition guards.

```js
// Compound AND: game must be playing AND console must be inactive
if (gameState === 'playing' && !consoleActive) { ... }

// Compound OR for movement keys
if (keys['a'] || keys['A']) ship.x -= ship.speed;
if (keys['d'] || keys['D']) ship.x += ship.speed;

// Compound: chaser enemies only appear from wave 2 with probability scaling
const isChaser = wave >= 2 && Math.random() < Math.min(0.45, 0.08 * wave);
```

---

## Input / Output

### Keyboard Input

Arrow keys trigger directional shooting; WASD controls movement; `P` pauses the game. All handled via `window.addEventListener`.

```js
function attachInput() {
  window.addEventListener('keydown', e => {
    if (e.key === 'p' || e.key === 'P') {
      if (gameState !== 'playing') return;
      if (consoleActive) {
        closeConsole();
      } else {
        openConsole();
      }
      return;
    }
    if (!consoleActive) {
      keys[e.key] = true;
    }
    // Prevent default page scroll for game keys
    if (e.key === ' ' || e.key === 'ArrowUp' || e.key === 'ArrowDown' ||
        e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
      e.preventDefault();
    }
  });
  window.addEventListener('keyup', e => { keys[e.key] = false; });
}
```

```js
// Arrow key shooting direction built from pressed keys each frame
let sx = 0, sy = 0;
if (keys['ArrowLeft'])  sx -= 1;
if (keys['ArrowRight']) sx += 1;
if (keys['ArrowUp'])    sy -= 1;
if (keys['ArrowDown'])  sy += 1;
if (sx !== 0 || sy !== 0) {
  fireDirected(sx, sy);
  ship.shootCooldown = 12;
}
```

### Canvas Rendering

All game objects are drawn using the Canvas 2D API inside dedicated `draw*` functions. The main game loop calls `ctx.clearRect` each frame before redrawing.

```js
// Main loop clears and redraws every frame
function loop() {
  ctx.clearRect(0, 0, W, H);
  updateLayers();
  drawLayers();
  // ...
  drawEnemies();
  drawBoss();
  drawBullets();
  drawParticles();
  drawShip();
  frameId = requestAnimationFrame(loop);
}
```

```js
// Ship drawn with Canvas paths, transforms, and gradients
function drawShip() {
  ctx.save();
  ctx.translate(x, y);
  // Thrust flame
  const tg = ctx.createLinearGradient(0, h * 0.4, 0, h * 0.4 + fl);
  tg.addColorStop(0, `rgba(${ch.thrustRgb},0.9)`);
  tg.addColorStop(1, 'transparent');
  ctx.fillStyle = tg;
  ctx.beginPath();
  ctx.moveTo(-w * 0.25, h * 0.35);
  ctx.lineTo(0, h * 0.4 + fl);
  ctx.lineTo(w * 0.25, h * 0.35);
  ctx.fill();
  // Hull
  ctx.fillStyle = ch.body;
  ctx.beginPath();
  ctx.moveTo(0, -h / 2);
  ctx.lineTo(w / 2, h / 2);
  ctx.lineTo(0, h * 0.3);
  ctx.lineTo(-w / 2, h / 2);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```

### GameEnv Configuration

`GameLevelVoidStriker` reads canvas dimensions from `gameEnv` and configures the background image data, then passes the full `gameEnv` to `VoidStrikerGame.init()` for container detection and canvas setup.

```js
constructor(gameEnv) {
  let width  = gameEnv.innerWidth;
  let height = gameEnv.innerHeight;
  let path   = gameEnv.path;

  const image_data_space = {
    id:     'VoidStriker-Background',
    src:    '',
    pixels: { height: 570, width: 1025 }
  };

  setTimeout(() => VoidStrikerGame.init(gameEnv), 200);

  this.classes = [
    { class: GameEnvBackground, data: image_data_space },
  ];
}
```

```js
// Inside init(), the engine's canvas and container are detected
function init(gameEnv) {
  let engineCanvas = (gameEnv && gameEnv.canvas)
    || document.querySelector('canvas[id]')
    || document.querySelector('.game-container canvas')
    || document.querySelector('canvas');

  container = engineCanvas
    ? (engineCanvas.parentElement || document.body)
    : (document.querySelector('.game-container') || document.body);

  const rect = container.getBoundingClientRect();
  W = rect.width  || (gameEnv && gameEnv.innerWidth)  || 800;
  H = rect.height || (gameEnv && gameEnv.innerHeight) || 500;
  // ...
}
```

### API Integration

Kill events are dispatched via a custom `CustomEvent` and could be picked up by a leaderboard integration. The cheat console also demonstrates event-driven output patterns.

```js
// Dispatched on every enemy or boss kill
window.dispatchEvent(new CustomEvent('vs-kills', { detail: { total: totalKills } }));

// Also dispatched when the cheat code is entered
function applyCheat() {
  totalKills = Math.max(totalKills, 30);
  updateHUD();
  window.dispatchEvent(new CustomEvent('vs-kills', { detail: { total: totalKills } }));
}
```

> **Note:** The `vs-kills` event stream is the integration point for a Leaderboard `fetch` POST — a listener can pick up `totalKills` and POST it to the backend API using `async/await`.

---

## Documentation

### Code Comments

Functions throughout the file include inline explanatory comments, particularly around non-obvious physics logic.

```js
// Per-frame gravity loop (lesson: Gravity System):
// 1. Subtract gravity from vertical velocity (pulls asteroid down)
a.verticalVelocity -= a.gravityAcceleration;
// 2. Clamp at terminal velocity so asteroids don't fall infinitely fast
if (-a.verticalVelocity > a.terminalVelocity) {
  a.verticalVelocity = -a.terminalVelocity;
}
// 3. Convert to engine velocity (flip sign): negative verticalVelocity = downward y motion
a.y += -a.verticalVelocity * worldSpeed;
```

```js
// Boss respawn loop: first boss on wave 3, then every 2 waves after each kill.
let bossesDefeated = 0;
let nextBossWave = 3;

// While the Boss Alien is alive, all other moving objects tick at this fraction
// of their normal speed. Restored to 1 the instant the boss dies.
let worldSpeed = 1;
```

```js
// ── Boss Alien (peer lesson: chasing-enemy update() loop) ───────────────
// Adapted from the "ocean" lesson on Math.atan2-based pursuit. Each frame,
// compute the angle from the boss to the ship and step toward the ship along
// (cos, sin). The boss eats 30 hits and counts as 5 kills on death.
```

---

## Debugging

### Console Debugging

Strategic logging opportunity points exist at key state transitions. The `updateHUD()` call at every frame renders live game state to the DOM. Placing `console.log` calls in `checkCollisions`, `updateShip`, or `spawnWave` provides per-frame debugging.

```js
// Example: add console.log inside checkCollisions to trace hit detection
function checkCollisions() {
  for (let bi = bullets.length - 1; bi >= 0; bi--) {
    const b = bullets[bi];
    if (boss) {
      const dx = b.x - boss.x, dy = b.y - boss.y;
      const dist = Math.sqrt(dx * dx + dy * dy);
      // console.log(`Bullet ${bi} → boss dist: ${dist.toFixed(1)}`);
      if (dist < boss.r + 4) {
        boss.hp--;
        // console.log(`Boss hit! HP remaining: ${boss.hp}`);
      }
    }
  }
}
```

### Hit Box Visualization

Collision radii can be visualized by drawing debug circles over each object's collision boundary directly in the `draw*` functions.

```js
// Add to drawEnemies() to visualize collision radius
enemies.forEach(e => {
  // ... existing draw code ...

  // DEBUG: draw hit box
  ctx.save();
  ctx.strokeStyle = 'rgba(255,0,0,0.5)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.arc(e.x, e.y, e.r + 4, 0, Math.PI * 2); // +4 matches collision check
  ctx.stroke();
  ctx.restore();
});
```

The collision check uses radius `e.r + 4` (enemy) and `boss.r + 4` (boss), so debug circles should match those values exactly.

### Source-Level Debugging

Set breakpoints in DevTools at the following high-value lines:

| Function | What to inspect |
|---|---|
| `checkCollisions()` line 954 | `dx`, `dy`, `dist` for bullet–enemy hits |
| `updateBoss()` line 657 | `angle`, `boss.x/y` chasing math |
| `spawnWave()` line 716 | `wave`, `nextBossWave` to verify boss trigger |
| `startGame()` line 1185 | Full reset — confirm all arrays are cleared |

### Network Debugging

The `vs-kills` CustomEvent is the hook for leaderboard API calls. In the Network tab, look for POST requests to your score endpoint triggered when this event fires. Check:

- Request payload contains `{ total: totalKills }`
- Response status is `200`/`201`
- CORS headers are present if calling a remote backend

```js
// Listening for the event and posting to an API (to be implemented):
window.addEventListener('vs-kills', async (e) => {
  try {
    const res = await fetch('/api/leaderboard', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ kills: e.detail.total }),
    });
    const data = await res.json();
    console.log('Score saved:', data);
  } catch (err) {
    console.error('Leaderboard POST failed:', err);
  }
});
```

---

## Testing & Verification

### Gameplay Testing

Key scenarios to test during a live playthrough:

| Test | What to verify |
|---|---|
| Move with WASD | Ship stays within canvas bounds (`Math.max`/`Math.min` clamping) |
| Shoot with Arrow keys | 3-way spread fires; 2-way spread activates during boss |
| Enemy reaches bottom | Enemy removed from array; no crash |
| Wave cleared | `wave++` increments and `spawnWave()` is called |
| Boss spawns wave 3 | `worldSpeed` drops to 0.4, boss health bar appears |
| Boss killed | `worldSpeed` resets to 1, bar hides, `bossesDefeated++` |
| `lives` reaches 0 | `gameState = 'dead'` and `showDeadScreen()` renders |
| `P` key | Pause overlay opens; game loop freezes on last frame |
| Cheat code `teambob` | `totalKills` jumps to 30 minimum |

### Integration Testing

The `vs-kills` CustomEvent is the integration surface. A leaderboard listener can be wired to POST scores to a backend. Verify end-to-end by:

1. Starting a game and killing an enemy — check the Network tab for the POST request
2. Confirming the response payload is valid JSON
3. Retrieving the leaderboard via GET and confirming the score was persisted

### API Error Handling

Wrap any `fetch` calls in `try/catch` and surface errors gracefully.

```js
window.addEventListener('vs-kills', async (e) => {
  try {
    const res = await fetch('/api/leaderboard', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ kills: e.detail.total }),
    });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    console.log('Leaderboard updated:', data);
  } catch (err) {
    console.error('Leaderboard error:', err.message);
    // Optionally: show a non-blocking in-game toast
  }
});
```