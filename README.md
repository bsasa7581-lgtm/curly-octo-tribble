<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0d0d1a">
<title>Neon Runner — играй в браузере</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  html,body{width:100%;height:100%;height:100dvh;overflow:hidden;background:#0d0d1a;font-family:Arial,Helvetica,sans-serif;touch-action:none;user-select:none;-webkit-user-select:none;-webkit-touch-callout:none}
  #wrap{position:fixed;inset:0;display:flex;justify-content:center;align-items:center}
  canvas{display:block;width:100vw!important;height:100vh!important;height:100dvh!important;background:#1e1e32;cursor:pointer}

  #startScreen{position:fixed;inset:0;display:flex;flex-direction:column;justify-content:center;align-items:center;background:radial-gradient(circle at 50% 40%,#1a1030 0%,#050510 70%);color:#fff;z-index:10;cursor:pointer;text-align:center;padding:20px}
  #startScreen h1{color:#00ffc8;font-size:clamp(30px,7vw,60px);text-shadow:0 0 40px rgba(0,255,200,.65);margin-bottom:10px;letter-spacing:3px;font-weight:900}
  #startScreen .sub{color:#9aa;font-size:clamp(13px,2.4vw,18px);margin-bottom:30px;max-width:520px;line-height:1.5}
  #playBtn{padding:18px 52px;background:linear-gradient(90deg,#00ffc8,#00c8a0);color:#002218;font-weight:900;font-size:clamp(18px,3vw,26px);border:none;border-radius:14px;box-shadow:0 0 50px rgba(0,255,200,.55);cursor:pointer;animation:pulse 1.6s ease-in-out infinite;letter-spacing:1px}
  @keyframes pulse{0%,100%{transform:scale(1)}50%{transform:scale(1.07)}}
  #startScreen .warn{margin-top:28px;font-size:12px;color:#666;max-width:420px;line-height:1.45}

  #hud{position:fixed;top:10px;left:14px;color:#00ffc8;font-size:15px;font-weight:bold;z-index:5;pointer-events:none;text-shadow:0 0 12px rgba(0,255,200,.6);line-height:1.35}
  #hud .dim{color:rgba(255,255,255,.45);font-weight:normal;font-size:12px}
  #hud .coin{color:#ffdd44}
  #pauseBtn{position:fixed;top:10px;right:14px;z-index:6;background:rgba(0,0,0,.55);border:1px solid #00ffc8;color:#00ffc8;border-radius:8px;padding:8px 14px;font-size:15px;font-weight:bold;cursor:pointer;backdrop-filter:blur(4px)}
  #pauseBtn:active{transform:scale(.95)}
  #hint{position:fixed;bottom:10px;width:100%;text-align:center;color:rgba(255,255,255,.4);font-size:13px;pointer-events:none;z-index:5;padding:0 10px}
  .hidden{display:none!important}

  #deathScreen{position:fixed;inset:0;display:none;flex-direction:column;justify-content:center;align-items:center;background:rgba(0,0,0,.85);z-index:20;color:#fff;text-align:center;padding:20px}
  #deathScreen.show{display:flex}
  #deathScreen h2{color:#ff5050;font-size:clamp(30px,6vw,52px);text-shadow:0 0 30px rgba(255,80,80,.7);margin-bottom:10px;font-weight:900}
  #deathScreen .stat{color:#aaa;font-size:clamp(15px,2.6vw,19px);margin-bottom:6px}
  #deathScreen .stat b{color:#00ffc8}
  #deathScreen .best{color:#ffdd44;font-size:14px;margin-bottom:26px}
  #deathScreen .btns{display:flex;gap:14px;flex-wrap:wrap;justify-content:center}
  #deathScreen button{padding:15px 34px;border:none;border-radius:12px;font-weight:900;font-size:16px;cursor:pointer;letter-spacing:.5px}
  #retryBtn{background:linear-gradient(90deg,#00ffc8,#00c8a0);color:#002218;box-shadow:0 0 26px rgba(0,255,200,.5)}
  #menuBtn{background:#3c3c5a;color:#fff}
  #deathScreen button:active{transform:scale(.96)}
</style>
</head>
<body>

<div id="startScreen">
  <h1>NEON RUNNER</h1>
  <p class="sub">Прыгай через шипы, собирай монеты, ставь рекорды.<br>Тап по экрану или пробел — прыжок.</p>
  <button id="playBtn" type="button">▶ ИГРАТЬ</button>
  <p class="warn">⚠️ Игра содержит яркие вспышки и быструю анимацию. При эпилепсии или светочувствительности — не играйте.</p>
</div>

<div id="wrap"><canvas id="game"></canvas></div>

<div id="hud" class="hidden">
  <div>Очки: <span id="score">0</span></div>
  <div class="coin">Монеты: <span id="coinCount">0</span></div>
  <div class="dim">Рекорд: <span id="best">0</span></div>
</div>
<button id="pauseBtn" class="hidden" type="button" title="Пауза (P)">⏸</button>
<div id="hint" class="hidden">Тап / Пробел — прыжок · P — пауза · R — рестарт</div>

<div id="deathScreen">
  <h2>💥 Game Over</h2>
  <div class="stat">Очки: <b id="dScore">0</b></div>
  <div class="stat">Монеты: <b id="dCoins">0</b></div>
  <div class="best">Рекорд: <span id="dBest">0</span></div>
  <div class="btns">
    <button id="retryBtn" type="button">▶ Заново</button>
    <button id="menuBtn" type="button">🏠 Меню</button>
  </div>
</div>

<script>
(function(){
"use strict";

const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const startScreen = document.getElementById('startScreen');
const playBtn = document.getElementById('playBtn');
const hud = document.getElementById('hud');
const scoreEl = document.getElementById('score');
const coinCountEl = document.getElementById('coinCount');
const bestEl = document.getElementById('best');
const pauseBtn = document.getElementById('pauseBtn');
const hintEl = document.getElementById('hint');
const deathScreen = document.getElementById('deathScreen');
const dScore = document.getElementById('dScore');
const dCoins = document.getElementById('dCoins');
const dBest = document.getElementById('dBest');

let GROUND_Y = 0;
let running = false, paused = false, started = false, gameOverState = false;
let lastT = 0;
let W = 0, H = 0, dpr = 1;

const player = {
  x: 0, y: 0, w: 40, h: 40,
  vy: 0, onGround: true,
  gravity: 2600, jumpForce: -940,
  rotation: 0
};

let obstacles = [];
let coins = [];
let particles = [];
let score = 0, coinCount = 0;
let best = Number(localStorage.getItem('neonRunnerBest') || 0);
bestEl.textContent = best;

let worldSpeed = 420;
let distance = 0;
let spawnTimer = 0;
let shake = 0;

function resize(){
  dpr = Math.min(window.devicePixelRatio || 1, 2);
  W = window.innerWidth;
  H = window.innerHeight;
  canvas.width = Math.floor(W*dpr);
  canvas.height = Math.floor(H*dpr);
  canvas.style.width = W+'px';
  canvas.style.height = H+'px';
  ctx.setTransform(dpr,0,0,dpr,0,0);

  const groundOffset = Math.max(70, Math.min(H*0.14, 140));
  GROUND_Y = H - groundOffset;

  const scale = Math.min(W/960, H/540);
  const size = Math.max(30, Math.min(52, 44*scale));
  player.w = size;
  player.h = size;
  player.x = Math.max(80, W*0.14);
  if (player.y === 0) player.y = GROUND_Y - player.h;
}

window.addEventListener('resize', ()=>{
  resize();
  if (player.onGround) player.y = GROUND_Y - player.h;
  else if (player.y + player.h > GROUND_Y) player.y = GROUND_Y - player.h;
});
resize();

function resetGame(){
  obstacles = [];
  coins = [];
  particles = [];
  score = 0;
  coinCount = 0;
  distance = 0;
  spawnTimer = 0.6;
  worldSpeed = 420;
  shake = 0;
  paused = false;
  running = true;
  gameOverState = false;
  deathScreen.classList.remove('show');
  player.y = GROUND_Y - player.h;
  player.vy = 0;
  player.onGround = true;
  player.rotation = 0;
  scoreEl.textContent = '0';
  coinCountEl.textContent = '0';
  bestEl.textContent = best;
  pauseBtn.textContent = '⏸';
}

function spawnObstacle(){
  const r = Math.random();
  const base = 40 + Math.random()*30;
  if (r < 0.55){
    obstacles.push({ x: W + 80, y: GROUND_Y - base, w: base, h: base, type:'spike' });
  } else if (r < 0.85){
    const h = 40 + Math.random()*80;
    obstacles.push({ x: W + 80, y: GROUND_Y - h, w: 50 + Math.random()*40, h: h, type:'block' });
  } else {
    obstacles.push({ x: W + 80, y: GROUND_Y - 45, w: 45, h: 45, type:'spike' });
    obstacles.push({ x: W + 80 + 52, y: GROUND_Y - 45, w: 45, h: 45, type:'spike' });
  }
  if (Math.random() < 0.45){
    coins.push({ x: W + 140, y: GROUND_Y - 170 - Math.random()*120, r: 14, collected:false });
  }
}

function jump(){
  if (!running || paused || gameOverState) return;
  if (player.onGround){
    player.vy = player.jumpForce;
    player.onGround = false;
    for (let i=0;i<10;i++){
      particles.push({
        x: player.x + player.w/2, y: player.y + player.h,
        vx: (Math.random()-0.5)*220, vy: Math.random()*160,
        life: 0.4, maxLife:0.4, col:'#00ffc8'
      });
    }
  }
}

function startGame(){
  if (started) return;
  started = true;
  startScreen.classList.add('hidden');
  hud.classList.remove('hidden');
  pauseBtn.classList.remove('hidden');
  hintEl.classList.remove('hidden');
  resetGame();
  lastT = performance.now();
  requestAnimationFrame(loop);
}

function togglePause(){
  if (!running || gameOverState) return;
  paused = !paused;
  pauseBtn.textContent = paused ? '▶' : '⏸';
}

function gameOver(){
  running = false;
  gameOverState = true;
  shake = 14;
  if (score > best){
    best = score;
    localStorage.setItem('neonRunnerBest', best);
  }
  dScore.textContent = score;
  dCoins.textContent = coinCount;
  dBest.textContent = best;
  bestEl.textContent = best;
  deathScreen.classList.add('show');
  pauseBtn.classList.add('hidden');
  hintEl.classList.add('hidden');
}

playBtn.addEventListener('click', (e)=>{ e.stopPropagation(); startGame(); });
startScreen.addEventListener('click', startGame);

canvas.addEventListener('pointerdown', (e)=>{ e.preventDefault(); jump(); });
window.addEventListener('keydown', (e)=>{
  if (e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW'){ e.preventDefault(); jump(); }
  if (e.code === 'KeyP'){ e.preventDefault(); togglePause(); }
  if (e.code === 'KeyR' && started && !gameOverState){ e.preventDefault(); resetGame(); }
});
pauseBtn.addEventListener('click', (e)=>{ e.stopPropagation(); togglePause(); });

document.getElementById('retryBtn').addEventListener('click', ()=>{
  resetGame();
  pauseBtn.classList.remove('hidden');
  hintEl.classList.remove('hidden');
});
document.getElementById('menuBtn').addEventListener('click', ()=>{
  running = false;
  gameOverState = false;
  deathScreen.classList.remove('show');
  startScreen.classList.remove('hidden');
  hud.classList.add('hidden');
  pauseBtn.classList.add('hidden');
  hintEl.classList.add('hidden');
  started = false;
  draw();
});

function update(dt){
  if (paused || gameOverState) return;
  dt = Math.min(dt, 0.033);

  distance += worldSpeed * dt;
  score = Math.floor(distance / 10);

  player.vy += player.gravity * dt;
  player.y += player.vy * dt;
  if (player.y + player.h >= GROUND_Y){
    player.y = GROUND_Y - player.h;
    player.vy = 0;
    if (!player.onGround){
      player.onGround = true;
      player.rotation = 0;
      for (let i=0;i<8;i++){
        particles.push({
          x: player.x + player.w/2, y: GROUND_Y,
          vx: (Math.random()-0.5)*160, vy: -Math.random()*120,
          life: 0.35, maxLife:0.35, col:'#00ffc8'
        });
      }
    }
  } else {
    player.onGround = false;
  }
  if (!player.onGround) player.rotation += dt * 7;

  spawnTimer -= dt;
  if (spawnTimer <= 0){
    spawnObstacle();
    const minGap = Math.max(0.55, 1.15 - Math.min(distance/40000, 0.5));
    spawnTimer = minGap + Math.random()*0.75;
  }

  for (const o of obstacles) o.x -= worldSpeed * dt;
  for (const c of coins) c.x -= worldSpeed * dt;

  obstacles = obstacles.filter(o => o.x + o.w > -80);
  coins = coins.filter(c => c.x + c.r > -80 && !c.collected);

  const px = player.x, py = player.y, pw = player.w, ph = player.h;

  for (const o of obstacles){
    if (o.type === 'spike'){
      const hit = px < o.x + o.w*0.72 && px + pw > o.x + o.w*0.28 &&
                  py + ph > o.y + o.h*0.32 && py < o.y + o.h;
      if (hit){ gameOver(); return; }
    } else {
      if (px < o.x + o.w && px + pw > o.x && py + ph > o.y && py < o.y + o.h){
        if (player.vy > 0 && (py + ph) - player.vy*dt <= o.y + 6){
          player.y = o.y - ph;
          player.vy = 0;
          player.onGround = true;
          player.rotation = 0;
        } else {
          gameOver(); return;
        }
      }
    }
  }

  for (const c of coins){
    const dx = (px + pw/2) - c.x;
    const dy = (py + ph/2) - c.y;
    if (dx*dx + dy*dy < (c.r + 20)*(c.r + 20)){
      c.collected = true;
      coinCount++;
      score += 50;
      coinCountEl.textContent = coinCount;
      for (let i=0;i<12;i++){
        particles.push({
          x: c.x, y: c.y,
          vx: (Math.random()-0.5)*280, vy: (Math.random()-0.5)*280,
          life: 0.5, maxLife:0.5, col:'#ffdd44'
        });
      }
    }
  }

  for (const p of particles){
    p.x += p.vx * dt;
    p.y += p.vy * dt;
    p.vy += 520 * dt;
    p.life -= dt;
  }
  particles = particles.filter(p => p.life > 0);

  worldSpeed = Math.min(860, 420 + distance/40);
  if (shake > 0) shake = Math.max(0, shake - dt*40);

  scoreEl.textContent = score;
}

function draw(){
  ctx.save();
  if (shake > 0){
    ctx.translate((Math.random()-0.5)*shake, (Math.random()-0.5)*shake);
  }

  const g = ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0,'#1e1e32');
  g.addColorStop(1,'#0d0d1a');
  ctx.fillStyle = g;
  ctx.fillRect(-20,-20,W+40,H+40);

  ctx.strokeStyle = 'rgba(0,255,200,0.06)';
  ctx.lineWidth = 1;
  const grid = 80;
  const off = (distance % grid);
  for (let x = -off; x < W; x += grid){
    ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,GROUND_Y); ctx.stroke();
  }
  for (let y = GROUND_Y; y > 0; y -= grid){
    ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(W,y); ctx.stroke();
  }

  ctx.fillStyle = '#14142a';
  ctx.fillRect(0, GROUND_Y, W, H - GROUND_Y);
  ctx.strokeStyle = '#00ffc8';
  ctx.lineWidth = 3;
  ctx.shadowColor = '#00ffc8';
  ctx.shadowBlur = 14;
  ctx.beginPath(); ctx.moveTo(0, GROUND_Y); ctx.lineTo(W, GROUND_Y); ctx.stroke();
  ctx.shadowBlur = 0;

  for (const o of obstacles){
    if (o.type === 'spike'){
      ctx.fillStyle = '#ff4080';
      ctx.shadowColor = '#ff4080';
      ctx.shadowBlur = 16;
      ctx.beginPath();
      ctx.moveTo(o.x + o.w/2, o.y);
      ctx.lineTo(o.x + o.w, o.y + o.h);
      ctx.lineTo(o.x, o.y + o.h);
      ctx.closePath();
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#ff88b0';
      ctx.lineWidth = 2;
      ctx.stroke();
    } else {
      ctx.fillStyle = '#4040a0';
      ctx.shadowColor = '#6060ff';
      ctx.shadowBlur = 14;
      ctx.fillRect(o.x, o.y, o.w, o.h);
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#8080ff';
      ctx.lineWidth = 2;
      ctx.strokeRect(o.x, o.y, o.w, o.h);
    }
  }

  for (const c of coins){
    if (c.collected) continue;
    ctx.fillStyle = '#ffdd44';
    ctx.shadowColor = '#ffdd44';
    ctx.shadowBlur = 18;
    ctx.beginPath(); ctx.arc(c.x, c.y, c.r, 0, Math.PI*2); ctx.fill();
    ctx.shadowBlur = 0;
    ctx.fillStyle = '#fff8c0';
    ctx.beginPath(); ctx.arc(c.x - c.r*0.3, c.y - c.r*0.3, c.r*0.35, 0, Math.PI*2); ctx.fill();
  }

  ctx.save();
  ctx.translate(player.x + player.w/2, player.y + player.h/2);
  if (!player.onGround) ctx.rotate(player.rotation);
  ctx.fillStyle = '#00ffc8';
  ctx.shadowColor = '#00ffc8';
  ctx.shadowBlur = 22;
  ctx.fillRect(-player.w/2, -player.h/2, player.w, player.h);
  ctx.shadowBlur = 0;
  ctx.strokeStyle = '#aaffee';
  ctx.lineWidth = 2;
  ctx.strokeRect(-player.w/2, -player.h/2, player.w, player.h);
  const ew = Math.max(5, player.w*0.18);
  ctx.fillStyle = '#0d0d1a';
  ctx.fillRect(-player.w/2 + player.w*0.18, -player.h/2 + player.h*0.22, ew, ew);
  ctx.fillRect(player.w/2 - player.w*0.18 - ew, -player.h/2 + player.h*0.22, ew, ew);
  ctx.restore();

  for (const p of particles){
    ctx.globalAlpha = Math.max(0, p.life / p.maxLife);
    ctx.fillStyle = p.col;
    ctx.fillRect(p.x - 2, p.y - 2, 4, 4);
  }
  ctx.globalAlpha = 1;

  ctx.restore();

  if (paused && running && !gameOverState){
    ctx.fillStyle = 'rgba(0,0,0,.65)';
    ctx.fillRect(0,0,W,H);
    ctx.fillStyle = '#00ffc8';
    ctx.textAlign = 'center';
    ctx.font = 'bold ' + Math.max(36, Math.min(60, W*0.06)) + 'px Arial';
    ctx.fillText('ПАУЗА', W/2, H/2);
    ctx.font = '16px Arial';
    ctx.fillStyle = '#aaa';
    ctx.fillText('P — продолжить · R — рестарт', W/2, H/2 + 42);
    ctx.textAlign = 'start';
  }
}

function loop(t){
  const dt = (t - lastT) / 1000;
  lastT = t;
  if (running && !paused && !gameOverState) update(dt);
  else if (shake > 0) shake = Math.max(0, shake - dt*40);
  draw();
  requestAnimationFrame(loop);
}

lastT = performance.now();
draw();

})();
</script>
</body>
</html>
