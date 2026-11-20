# -.get
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0b0e14">
<title>Автохлам</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; -webkit-tap-highlight-color:transparent; }
  html, body { width:100%; height:100%; overflow:hidden; background:#0b0e14; font-family:Arial, sans-serif; touch-action:none; }
  #game { position:fixed; inset:0; width:100%; height:100%; display:block; }
  :root { --ui:1; }

  #hud { position:fixed; top:calc(10px*var(--ui)); left:calc(12px*var(--ui)); right:calc(12px*var(--ui));
    display:flex; justify-content:space-between; align-items:flex-start; pointer-events:none; z-index:5; }
  .hudbox { background:rgba(10,14,20,.72); border-radius:12px; padding:calc(7px*var(--ui)) calc(12px*var(--ui));
    color:#fff; font-size:calc(16px*var(--ui)); font-weight:bold; }
  #hudRight { display:flex; flex-direction:column; gap:6px; align-items:flex-end; }
  #speedBox { font-size:calc(20px*var(--ui)); }
  #hpWrap { width:calc(130px*var(--ui)); height:calc(12px*var(--ui)); background:rgba(255,255,255,.15);
    border-radius:8px; overflow:hidden; margin-top:2px; }
  #hpFill { height:100%; width:100%; background:#2ecc71; border-radius:8px; transition:width .2s; }
  #fuelWrap { width:calc(130px*var(--ui)); height:calc(8px*var(--ui)); background:rgba(255,255,255,.15);
    border-radius:8px; overflow:hidden; margin-top:2px; }
  #fuelFill { height:100%; width:100%; background:#3498db; }

  #btns { position:fixed; top:calc(58px*var(--ui)); right:calc(10px*var(--ui)); display:flex; flex-direction:column; gap:8px; z-index:6; }
  .sbtn { width:calc(46px*var(--ui)); height:calc(46px*var(--ui)); border:none; border-radius:12px;
    background:rgba(20,26,38,.85); color:#fff; font-size:calc(20px*var(--ui)); cursor:pointer; }

  #ctrls { position:fixed; bottom:calc(14px*var(--ui)); left:calc(12px*var(--ui)); right:calc(12px*var(--ui));
    display:flex; justify-content:space-between; align-items:flex-end; z-index:6; pointer-events:none; }
  #ctrls > div { display:flex; gap:calc(12px*var(--ui)); pointer-events:auto; }
  .ctl { width:calc(66px*var(--ui)); height:calc(66px*var(--ui)); border:none; border-radius:50%;
    background:rgba(30,40,58,.88); color:#fff; font-size:calc(26px*var(--ui)); cursor:pointer;
    user-select:none; touch-action:none; }
  .ctl:active { background:#3d5afe; }
  #btnNitro { background:rgba(120,40,200,.9); font-size:calc(18px*var(--ui)); }

  .overlay { position:fixed; inset:0; background:rgba(5,8,14,.82); display:none;
    justify-content:center; align-items:center; z-index:20; }
  .panel { background:#141a26; border-radius:18px; padding:calc(18px*var(--ui)); max-width:calc(420px*var(--ui));
    width:min(92vw, 480px); max-height:86vh; overflow:hidden; display:flex; flex-direction:column; color:#eef2f7; }
  .panel h2 { margin-bottom:10px; }
  .panel p, .panel .hint { font-size:calc(14px*var(--ui)); color:#aeb9c9; margin-bottom:10px; }
  .panel button { background:#2f6fed; color:#fff; border:none; border-radius:10px; padding:calc(10px*var(--ui)) calc(14px*var(--ui));
    font-size:calc(15px*var(--ui)); cursor:pointer; font-weight:bold; }
  .panel button:disabled, .panel button.noB { background:#3a4657; color:#8b96a5; }
  .panel button.danger { background:#c0392b; }
  .panel button.deadB { background:#8a4b12; }
  .btnrow { display:flex; gap:10px; margin-top:8px; }
  .btnrow button { flex:1; }
  .rowline { display:flex; justify-content:space-between; align-items:center; margin-bottom:10px; }
  .rowline span { font-weight:bold; font-size:calc(17px*var(--ui)); }
  .mini { padding:calc(6px*var(--ui)) calc(10px*var(--ui)) !important; }
  .scroll { overflow-y:auto; flex:1; margin-top:6px; }
  .carcard { display:flex; align-items:center; gap:10px; background:#1b2334; border-radius:12px;
    padding:calc(10px*var(--ui)); margin-bottom:8px; }
  .carcard button { white-space:nowrap; }
  .cinfo { flex:1; font-size:calc(14px*var(--ui)); }
  .stats { font-size:calc(12px*var(--ui)); color:#9fb0c5; }
  .swatch { width:calc(34px*var(--ui)); height:calc(46px*var(--ui)); border-radius:8px; flex-shrink:0; }
  .optgrp { display:flex; gap:8px; margin-bottom:12px; flex-wrap:wrap; }
  .opt { flex:1; min-width:80px; background:#232c40 !important; }
  .opt.on { background:#2f6fed !important; outline:2px solid #7ea6ff; }
  #admPass { width:100%; padding:calc(10px*var(--ui)); border-radius:10px; border:1px solid #33415c;
    background:#0d1220; color:#fff; font-size:calc(15px*var(--ui)); margin-bottom:8px; }
  #startOverlay .panel { text-align:center; align-items:center; }
  #startOverlay h1 { font-size:calc(30px*var(--ui)); margin-bottom:8px; }
  #startOverlay .ctlhint { font-size:calc(13px*var(--ui)); color:#aeb9c9; margin-bottom:14px; line-height:1.6; }
  #startOverlay button { font-size:calc(18px*var(--ui)); padding:calc(12px*var(--ui)) calc(34px*var(--ui)); }
</style>
</head>
<body>
<canvas id="game"></canvas>

<div id="hud">
  <div class="hudbox"><span id="hudCoins">🪙 0</span></div>
  <div id="hudRight">
    <div class="hudbox" id="speedBox"><span id="hudSpeed">0 км/ч</span></div>
    <div class="hudbox"><span id="hudKm">0 км</span></div>
    <div class="hudbox"><div id="hpWrap"><div id="hpFill"></div></div></div>
    <div class="hudbox"><div id="fuelWrap"><div id="fuelFill"></div></div></div>
  </div>
</div>

<div id="btns">
  <button class="sbtn" id="btnGarage" title="Гараж">🔧</button>
  <button class="sbtn" id="btnSettings" title="Настройки графики">⚙️</button>
  <button class="sbtn" id="btnFull" title="На весь экран">🖥️</button>
</div>

<div id="ctrls">
  <div>
    <button class="ctl" id="btnLeft">⬅</button>
    <button class="ctl" id="btnRight">➡</button>
  </div>
  <div>
    <button class="ctl" id="btnBrake">🛑</button>
    <button class="ctl" id="btnNitro">NITRO</button>
    <button class="ctl" id="btnGas">⛽</button>
  </div>
</div>

<div class="overlay" id="startOverlay">
  <div class="panel">
    <h1 id="startTitle">АВТОХЛАМ</h1>
    <div class="ctlhint">
      📱 Кнопки внизу или просто ведите пальцем по экрану<br>
      💻 Стрелки / WASD, G — гараж<br>
      🪙 Собирайте монеты, не разбивайте машину!
    </div>
    <button id="btnPlay">▶ Играть</button>
  </div>
</div>

<div class="overlay" id="garageOverlay"><div class="panel"></div></div>
<div class="overlay" id="settingsOverlay"><div class="panel"></div></div>
<div class="overlay" id="adminOverlay"><div class="panel"></div></div>
<div class="overlay" id="brokenOverlay"><div class="panel"></div></div>

<script>

'use strict';
const IS_BETA = false;
const SAVE_KEY = IS_BETA ? 'carGameBeta_v3' : 'carGameOriginal_v3';
const ADMIN_PASS = 'admin123';

const CARS = [
 {id:'vaz', name:'Жигули ВАЗ-2101', price:0, max:150, acc:55, hp:100, rep:40, c1:'#b03a2e', c2:'#7b241c'},
 {id:'oka', name:'Ока', price:120, max:130, acc:45, hp:80, rep:25, c1:'#d5d8dc', c2:'#99a3a4'},
 {id:'moskvich', name:'Москвич-412', price:250, max:160, acc:60, hp:110, rep:35, c1:'#2e4053', c2:'#1b2631'},
 {id:'pickup', name:'Пикап', price:300, max:150, acc:50, hp:150, rep:45, c1:'#5d6d7e', c2:'#34495e'},
 {id:'niva', name:'Нива', price:400, max:155, acc:55, hp:140, rep:45, c1:'#1e8449', c2:'#145a32'},
 {id:'volga', name:'Волга ГАЗ-24', price:550, max:170, acc:60, hp:120, rep:50, c1:'#eaecee', c2:'#aab7b8'},
 {id:'uaz', name:'УАЗ Патриот', price:700, max:160, acc:55, hp:170, rep:55, c1:'#212f3d', c2:'#0f1923'},
 {id:'sport', name:'Спорткар', price:800, max:220, acc:95, hp:80, rep:90, c1:'#c0392b', c2:'#641e16'},
 {id:'vesta', name:'Лада Веста', price:850, max:195, acc:80, hp:110, rep:65, c1:'#2471a3', c2:'#1a5276'},
 {id:'bmw', name:'BMW M3', price:1200, max:240, acc:100, hp:100, rep:80, c1:'#154360', c2:'#0b2b45'},
 {id:'merc', name:'Mercedes S-класс', price:1600, max:230, acc:90, hp:140, rep:95, c1:'#7f8c8d', c2:'#424949'},
 {id:'magnit', name:'Суперкар «Молния»', price:2500, max:290, acc:130, hp:95, rep:120, c1:'#f1c40f', c2:'#b7950b'}
];

let S = {
  coins: 0,
  owned: ['vaz'],
  hpMap: {vaz: 100},
  cur: 'vaz',
  best: 0,
  settings: {quality: 'auto', ui: 'normal'}
};

function save(){ try{ localStorage.setItem(SAVE_KEY, JSON.stringify(S)); }catch(e){} }
function load(){
  try{
    const d = JSON.parse(localStorage.getItem(SAVE_KEY));
    if(d && typeof d === 'object'){
      S = Object.assign(S, d);
      if(!S.settings) S.settings = {quality:'auto', ui:'normal'};
    }
  }catch(e){}
}
load();

// ---------- канвас и качество графики ----------
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
let W = 0, H = 0, dpr = 1, renderScale = 1, SCL = 1;

function resolveQuality(){
  let q = S.settings.quality;
  if(q === 'auto'){
    const smallScreen = Math.min(screen.width, screen.height) < 520;
    const weakCpu = (navigator.hardwareConcurrency || 4) <= 4;
    q = (smallScreen || weakCpu) ? 'medium' : 'high';
  }
  renderScale = q === 'low' ? 0.5 : (q === 'medium' ? 0.75 : 1);
}
function applyUi(){
  const ui = S.settings.ui === 'small' ? 0.85 : (S.settings.ui === 'big' ? 1.3 : 1);
  document.documentElement.style.setProperty('--ui', ui);
}
function resize(){
  W = window.innerWidth; H = window.innerHeight;
  resolveQuality();
  dpr = Math.min(window.devicePixelRatio || 1, 2) * renderScale;
  canvas.width = Math.round(W * dpr);
  canvas.height = Math.round(H * dpr);
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  SCL = Math.min(H / 800, W / 450);
}
window.addEventListener('resize', resize);
window.addEventListener('orientationchange', function(){ setTimeout(resize, 200); });
applyUi();
resize();

// ---------- состояние игры ----------
let paused = true;
let kmh = 0;
let playerX = 0;
let dist = 0;
let kmFloat = 0;
let hp = S.hpMap[S.cur] !== undefined ? S.hpMap[S.cur] : 100;
let dents = [];
let invuln = 0;
let broken = false;
let shake = 0;
let spawnT = 1.2;
let obstacles = [];
let coins = [];
let sparks = [];
let fuel = 100;
let nitroHeld = false;
let last = 0;
let worldOff = 0;

const keys = {};
let gasHeld = false, brakeHeld = false, leftHeld = false, rightHeld = false;
let touchSteerX = null;

function curCar(){ return CARS.find(c => c.id === S.cur) || CARS[0]; }
function hpMax(){ return curCar().hp; }
function pxPerKmh(){ return H * 0.0034; }

function startRun(){
  kmh = 0; dist = 0; kmFloat = 0; dents = []; invuln = 0; broken = false;
  spawnT = 1.2; obstacles = []; coins = []; sparks = []; fuel = 100; shake = 0;
  hp = S.hpMap[S.cur] !== undefined ? S.hpMap[S.cur] : hpMax();
  paused = false;
  hideAll();
}

// ---------- ввод ----------
document.addEventListener('keydown', function(e){
  keys[e.code] = true;
  if(e.code === 'KeyG'){ if(garageOverlay.style.display === 'flex') hideAll(); else openGarage(); }
  if(e.code === 'Escape') hideAll();
});
document.addEventListener('keyup', function(e){ keys[e.code] = false; });

function bindHold(el, set){
  el.addEventListener('pointerdown', function(e){ e.preventDefault(); set(true); });
  el.addEventListener('pointerup', function(e){ set(false); });
  el.addEventListener('pointerleave', function(e){ set(false); });
  el.addEventListener('pointercancel', function(e){ set(false); });
}
bindHold(document.getElementById('btnLeft'), function(v){ leftHeld = v; });
bindHold(document.getElementById('btnRight'), function(v){ rightHeld = v; });
bindHold(document.getElementById('btnGas'), function(v){ gasHeld = v; });
bindHold(document.getElementById('btnBrake'), function(v){ brakeHeld = v; });
if(IS_BETA) bindHold(document.getElementById('btnNitro'), function(v){ nitroHeld = v; });

canvas.addEventListener('pointerdown', function(e){ touchSteerX = e.clientX; });
canvas.addEventListener('pointermove', function(e){ if(touchSteerX !== null) touchSteerX = e.clientX; });
window.addEventListener('pointerup', function(){ touchSteerX = null; });

// ---------- спавн ----------
function roadRect(){
  const roadW = Math.min(W * 0.86, 640 * SCL);
  return {x: (W - roadW) / 2, w: roadW, laneW: roadW / 4};
}
function carSize(){ const h = H * 0.13; return {w: h * 0.52, h: h}; }

function spawnObstacle(){
  const r = roadRect(), cs = carSize();
  const lane = Math.floor(Math.random() * 4);
  const x = r.x + r.laneW * lane + r.laneW / 2;
  const roll = Math.random();
  if(roll < 0.30){
    coins.push({x: x, y: -60, r: 15 * SCL});
  } else if(roll < 0.55){
    obstacles.push({type: 'cone', x: x, y: -60, kmh: 0, w: 22 * SCL, h: 26 * SCL});
  } else if(roll < 0.93){
    const oncoming = Math.random() < 0.3;
    const okmh = oncoming ? 60 + Math.random() * 40 : 50 + Math.random() * 50;
    const colors = ['#8e44ad','#16a085','#d35400','#2c3e50','#c0392b','#27ae60'];
    obstacles.push({type: 'car', x: x, y: -140, kmh: okmh, oncoming: oncoming,
      w: cs.w * (0.9 + Math.random() * 0.2), h: cs.h, c1: colors[Math.floor(Math.random() * colors.length)]});
  } else {
    const side = Math.random() < 0.5 ? 1 : 2;
    obstacles.push({type: 'barrier', x: r.x + r.laneW * side + r.laneW / 2, y: -60,
      kmh: 0, w: r.laneW * 0.9, h: 20 * SCL});
  }
}

// ---------- обновление ----------
function update(dt){
  const car = curCar();
  let gas = gasHeld || keys['ArrowUp'] || keys['KeyW'];
  let brake = brakeHeld || keys['ArrowDown'] || keys['KeyS'];
  let steer = (rightHeld || keys['ArrowRight'] || keys['KeyD'] ? 1 : 0) - (leftHeld || keys['ArrowLeft'] || keys['KeyA'] ? 1 : 0);

  let effMax = car.max;
  if(IS_BETA && nitroHeld && fuel > 0 && !broken && kmh > 10){
    effMax = car.max * 1.5;
    fuel = Math.max(0, fuel - 28 * dt);
  } else {
    fuel = Math.min(100, fuel + 6 * dt);
  }

  if(!broken){
    if(gas) kmh += car.acc * dt * (kmh < effMax ? 1 : 0);
    else kmh -= 30 * dt;
    if(brake) kmh -= 140 * dt;
    kmh = Math.max(0, Math.min(kmh, effMax));
  } else {
    kmh -= 200 * dt; if(kmh < 0) kmh = 0;
  }

  // руление
  const cs = carSize(), r = roadRect();
  if(touchSteerX !== null){
    const dx = touchSteerX - playerX;
    if(Math.abs(dx) > 4) playerX += Math.sign(dx) * Math.min(Math.abs(dx), W * 0.9 * dt);
  } else if(steer !== 0){
    playerX += steer * W * (kmh > 5 ? 0.75 : 0.2) * dt;
  }
  const halfW = cs.w / 2 + 4;
  playerX = Math.max(r.x + halfW, Math.min(r.x + r.w - halfW, playerX));

  const pp = kmh * pxPerKmh();
  dist += pp * dt;
  worldOff += pp * dt;
  kmFloat += (kmh / 3600) * dt * 60; // условный масштаб времени
  if(kmFloat >= 1){ kmFloat -= 1; S.coins += 5; save(); }

  // спавн
  spawnT -= dt * (0.6 + kmh / 110);
  if(spawnT <= 0){ spawnT = 0.85 + Math.random() * 0.9; spawnObstacle(); }

  // движение объектов
  const cs2 = carSize();
  const pbox = {x: playerX - cs2.w / 2, y: H - cs2.h - H * 0.09, w: cs2.w, h: cs2.h};
  for(let i = obstacles.length - 1; i >= 0; i--){
    const o = obstacles[i];
    const rel = o.oncoming ? (kmh + o.kmh) : (kmh - o.kmh);
    o.y += rel * pxPerKmh() * dt;
    if(o.type === 'car' && !o.oncoming && o.y < -100){ obstacles.splice(i, 1); continue; }
    if(o.y > H + 150){ obstacles.splice(i, 1); continue; }
    if(kmh > 3 && invuln <= 0){
      const box = {x: o.x - o.w / 2, y: o.y - o.h / 2, w: o.w, h: o.h};
      if(pbox.x < box.x + box.w && pbox.x + pbox.w > box.x &&
         pbox.y < box.y + box.h && pbox.y + pbox.h > box.y){
        const relK = Math.abs(kmh - (o.oncoming ? -o.kmh : o.kmh));
        const dmg = o.type === 'cone' ? relK * 0.25 + 2 : relK * 0.45 + 4;
        hp = Math.max(0, hp - dmg);
        S.hpMap[S.cur] = Math.round(hp);
        invuln = 0.9;
        shake = 0.35;
        if(dents.length < 8) dents.push({fx: Math.random() * 0.8 + 0.1, fy: Math.random() * 0.8 + 0.1, r: 0.12 + Math.random() * 0.1});
        for(let s = 0; s < 8; s++) sparks.push({x: o.x, y: o.y, vx: (Math.random()-0.5)*260, vy: (Math.random()-0.5)*260, life: 0.4});
        obstacles.splice(i, 1);
        if(hp <= 0 && !broken){ broken = true; openBroken(); }
        save();
      }
    }
  }

  for(let i = coins.length - 1; i >= 0; i--){
    const c = coins[i];
    c.y += pp * dt;
    if(c.y > H + 60){ coins.splice(i, 1); continue; }
    const dx = c.x - playerX, dy = c.y - (pbox.y + pbox.h / 2);
    if(dx * dx + dy * dy < (c.r + cs2.w / 2) * (c.r + cs2.w / 2)){
      S.coins += 10; coins.splice(i, 1); save();
    }
  }

  for(let i = sparks.length - 1; i >= 0; i--){
    const s = sparks[i];
    s.x += s.vx * dt; s.y += s.vy * dt; s.life -= dt;
    if(s.life <= 0) sparks.splice(i, 1);
  }

  if(invuln > 0) invuln -= dt;
  if(shake > 0) shake -= dt;

  if(kmFloat > S.best) S.best = kmFloat;
}

// ---------- отрисовка ----------
function rr(x, y, w, h, rad){
  ctx.beginPath();
  ctx.moveTo(x + rad, y);
  ctx.arcTo(x + w, y, x + w, y + h, rad);
  ctx.arcTo(x + w, y + h, x, y + h, rad);
  ctx.arcTo(x, y + h, x, y, rad);
  ctx.arcTo(x, y, x + w, y, rad);
  ctx.closePath();
}

function drawCar(cx, cy, w, h, c1, c2, flip, dnt){
  ctx.save();
  ctx.translate(cx, cy);
  if(flip) ctx.rotate(Math.PI);
  const x = -w / 2, y = -h / 2;
  // колёса
  ctx.fillStyle = '#111';
  ctx.fillRect(x - 3, y + h * 0.14, 7, h * 0.2);
  ctx.fillRect(x + w - 4, y + h * 0.14, 7, h * 0.2);
  ctx.fillRect(x - 3, y + h * 0.66, 7, h * 0.2);
  ctx.fillRect(x + w - 4, y + h * 0.66, 7, h * 0.2);
  // корпус
  ctx.fillStyle = c1;
  rr(x, y, w, h, w * 0.18); ctx.fill();
  ctx.fillStyle = c2;
  rr(x + w * 0.1, y + h * 0.06, w * 0.8, h * 0.88, w * 0.14); ctx.fill();
  // крыша
  ctx.fillStyle = 'rgba(20,30,45,0.85)';
  rr(x + w * 0.16, y + h * 0.3, w * 0.68, h * 0.26, w * 0.1); ctx.fill();
  ctx.fillStyle = 'rgba(160,220,255,0.55)';
  rr(x + w * 0.18, y + h * 0.08, w * 0.64, h * 0.18, w * 0.08); ctx.fill();
  // фары
  ctx.fillStyle = '#ffe082';
  ctx.fillRect(x + w * 0.08, y + h * 0.9, w * 0.16, h * 0.05);
  ctx.fillRect(x + w * 0.76, y + h * 0.9, w * 0.16, h * 0.05);
  ctx.fillStyle = '#e53935';
  ctx.fillRect(x + w * 0.08, y + h * 0.03, w * 0.16, h * 0.04);
  ctx.fillRect(x + w * 0.76, y + h * 0.03, w * 0.16, h * 0.04);
  // вмятины
  if(dnt && dnt.length){
    ctx.fillStyle = 'rgba(0,0,0,0.45)';
    for(const d of dnt){
      ctx.beginPath();
      ctx.arc(x + w * d.fx, y + h * d.fy, w * d.r, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  ctx.restore();
}

function draw(){
  ctx.save();
  if(shake > 0){
    ctx.translate((Math.random()-0.5) * 8 * shake, (Math.random()-0.5) * 8 * shake);
  }
  const r = roadRect();
  // трава
  ctx.fillStyle = '#1d4a22';
  ctx.fillRect(0, 0, r.x, H);
  ctx.fillRect(r.x + r.w, 0, W - r.x - r.w, H);
  // деревья-полоски на траве
  ctx.fillStyle = '#153a1a';
  const stripe = 90 * SCL;
  const off = worldOff % stripe;
  for(let y = -stripe; y < H + stripe; y += stripe){
    const yy = y + off;
    ctx.fillRect(r.x * 0.25, yy, r.x * 0.5, 34 * SCL);
    ctx.fillRect(r.x + r.w + r.x * 0.25, yy + stripe / 2, r.x * 0.5, 34 * SCL);
  }
  // дорога
  ctx.fillStyle = '#2f3542';
  ctx.fillRect(r.x, 0, r.w, H);
  // отбойники по краям
  const blockH = 46 * SCL, offB = worldOff % (blockH * 2);
  for(let y = -blockH * 2; y < H + blockH * 2; y += blockH * 2){
    const yy = y + offB;
    ctx.fillStyle = '#e74c3c';
    ctx.fillRect(r.x - 6 * SCL, yy, 6 * SCL, blockH);
    ctx.fillRect(r.x + r.w, yy + blockH, 6 * SCL, blockH);
    ctx.fillStyle = '#ecf0f1';
    ctx.fillRect(r.x - 6 * SCL, yy + blockH, 6 * SCL, blockH);
    ctx.fillRect(r.x + r.w, yy, 6 * SCL, blockH);
  }
  // разметка
  ctx.strokeStyle = 'rgba(255,255,255,0.75)';
  ctx.lineWidth = 4 * SCL;
  ctx.setLineDash([44 * SCL, 36 * SCL]);
  ctx.lineDashOffset = -(worldOff % 80 * SCL);
  for(let i = 1; i < 4; i++){
    ctx.beginPath();
    ctx.moveTo(r.x + r.laneW * i, 0);
    ctx.lineTo(r.x + r.laneW * i, H);
    ctx.stroke();
  }
  ctx.setLineDash([]);
  // монеты
  for(const c of coins){
    ctx.fillStyle = '#f1c40f';
    ctx.beginPath(); ctx.arc(c.x, c.y, c.r, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#b7950b';
    ctx.beginPath(); ctx.arc(c.x, c.y, c.r * 0.6, 0, Math.PI * 2); ctx.fill();
  }
  // препятствия
  for(const o of obstacles){
    if(o.type === 'cone'){
      ctx.fillStyle = '#e67e22';
      ctx.beginPath();
      ctx.moveTo(o.x, o.y - o.h / 2);
      ctx.lineTo(o.x - o.w / 2, o.y + o.h / 2);
      ctx.lineTo(o.x + o.w / 2, o.y + o.h / 2);
      ctx.closePath(); ctx.fill();
      ctx.fillStyle = '#ecf0f1';
      ctx.fillRect(o.x - o.w * 0.3, o.y - o.h * 0.05, o.w * 0.6, o.h * 0.14);
    } else if(o.type === 'barrier'){
      ctx.fillStyle = '#f1c40f';
      rr(o.x - o.w / 2, o.y - o.h / 2, o.w, o.h, 4); ctx.fill();
      ctx.fillStyle = '#2c3e50';
      for(let bx = 0; bx < o.w - 10; bx += 26){
        ctx.fillRect(o.x - o.w / 2 + bx, o.y - o.h / 2, 13, o.h);
      }
    } else {
      drawCar(o.x, o.y, o.w, o.h, o.c1, 'rgba(0,0,0,0.35)', !!o.oncoming, null);
    }
  }
  // игрок
  const car = curCar(), cs = carSize();
  const py = H - cs.h - H * 0.09;
  if(!(invuln > 0 && Math.floor(invuln * 12) % 2 === 0)){
    drawCar(playerX, py + cs.h / 2, cs.w, cs.h, car.c1, car.c2, false, dents);
    if(IS_BETA && nitroHeld && fuel > 0 && kmh > 10){
      ctx.fillStyle = 'rgba(255,140,0,0.8)';
      ctx.beginPath();
      ctx.moveTo(playerX - cs.w * 0.2, py + cs.h);
      ctx.lineTo(playerX + cs.w * 0.2, py + cs.h);
      ctx.lineTo(playerX, py + cs.h + 26 * SCL * (0.7 + Math.random() * 0.6));
      ctx.closePath(); ctx.fill();
    }
  }
  // искры
  ctx.fillStyle = '#ffd54f';
  for(const s of sparks){
    ctx.globalAlpha = Math.max(0, s.life * 2.2);
    ctx.fillRect(s.x - 2, s.y - 2, 4, 4);
  }
  ctx.globalAlpha = 1;
  ctx.restore();
}

// ---------- HUD ----------
const elCoins = document.getElementById('hudCoins');
const elKm = document.getElementById('hudKm');
const elSpeed = document.getElementById('hudSpeed');
const elHp = document.getElementById('hpFill');
const elFuel = document.getElementById('fuelFill');
function hud(){
  elCoins.textContent = '🪙 ' + S.coins;
  elKm.textContent = Math.floor(kmFloat) + ' км';
  elSpeed.textContent = Math.round(kmh) + ' км/ч';
  elHp.style.width = (hp / hpMax() * 100) + '%';
  elHp.style.background = hp / hpMax() > 0.5 ? '#2ecc71' : (hp / hpMax() > 0.25 ? '#f39c12' : '#e74c3c');
  if(IS_BETA) elFuel.style.width = fuel + '%';
}

// ---------- цикл ----------
function loop(t){
  const dt = Math.min((t - last) / 1000, 0.05);
  last = t;
  if(!paused) update(dt);
  draw();
  hud();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

// ---------- оверлеи ----------
const startOverlay = document.getElementById('startOverlay');
const garageOverlay = document.getElementById('garageOverlay');
const settingsOverlay = document.getElementById('settingsOverlay');
const adminOverlay = document.getElementById('adminOverlay');
const brokenOverlay = document.getElementById('brokenOverlay');

function hideAll(){
  startOverlay.style.display = 'none';
  garageOverlay.style.display = 'none';
  settingsOverlay.style.display = 'none';
  adminOverlay.style.display = 'none';
  brokenOverlay.style.display = 'none';
  if(kmh === 0 && !broken && dist === 0) return; // до первого старта
  paused = false;
}
function only(ov){
  hideAll0();
  ov.style.display = 'flex';
  paused = true;
}
function hideAll0(){
  [startOverlay, garageOverlay, settingsOverlay, adminOverlay, brokenOverlay].forEach(function(o){ o.style.display = 'none'; });
}

document.getElementById('btnPlay').addEventListener('click', startRun);
document.getElementById('btnGarage').addEventListener('click', openGarage);
document.getElementById('btnSettings').addEventListener('click', openSettings);
document.getElementById('btnFull').addEventListener('click', function(){
  if(document.fullscreenElement) document.exitFullscreen();
  else document.documentElement.requestFullscreen().catch(function(){});
});

// гараж
function dots(n){ let s = ''; for(let i = 0; i < 5; i++) s += i < n ? '●' : '○'; return s; }
function openGarage(){
  only(garageOverlay);
  const cur = curCar();
  const repairCost = Math.ceil((1 - hp / hpMax()) * cur.rep);
  let html = '<h2>🔧 Гараж</h2><div class="rowline"><span>🪙 ' + S.coins + '</span>';
  if(!IS_BETA) html += '<button class="mini" id="adminKey">🔑</button>';
  html += '<button class="mini" id="garClose">✕</button></div>';
  html += '<div class="carcard"><div><b>' + cur.name + '</b> (выбрана)<br>Состояние: ' + Math.round(hp) + '/' + hpMax() + '</div>';
  html += '<button id="btnRepair" ' + (repairCost <= 0 ? 'disabled' : '') + '>Починить 🪙' + repairCost + '</button></div>';
  html += '<div class="scroll">';
  for(const c of CARS){
    if(c.id === S.cur) continue;
    const owned = S.owned.indexOf(c.id) !== -1;
    const chp = S.hpMap[c.id] !== undefined ? S.hpMap[c.id] : c.hp;
    html += '<div class="carcard"><div class="swatch" style="background:' + c.c1 + '"></div>';
    html += '<div class="cinfo"><b>' + c.name + '</b><br><span class="stats">Скорость ' + dots(Math.round((c.max-130)/32)) + ' · Прочность ' + dots(Math.round((c.hp-80)/18)) + '<br>Ремонт 🪙' + c.rep + '</span></div>';
    if(owned){
      html += '<button data-sel="' + c.id + '" ' + (chp <= 0 ? 'class="deadB"' : '') + '>' + (chp <= 0 ? 'Разбита' : 'Выбрать') + '</button>';
    } else {
      html += '<button data-buy="' + c.id + '" ' + (S.coins < c.price ? 'class="noB"' : '') + '>🪙' + c.price + '</button>';
    }
    html += '</div>';
  }
  html += '</div>';
  garageOverlay.querySelector('.panel').innerHTML = html;
  garageOverlay.querySelector('#garClose').addEventListener('click', hideAll);
  const ak = garageOverlay.querySelector('#adminKey');
  if(ak) ak.addEventListener('click', function(){ openAdmin(); });
  garageOverlay.querySelector('#btnRepair').addEventListener('click', function(){
    if(S.coins >= repairCost && repairCost > 0){
      S.coins -= repairCost;
      hp = hpMax(); S.hpMap[S.cur] = hpMax(); dents = []; broken = false;
      save(); openGarage();
    }
  });
  garageOverlay.querySelectorAll('[data-buy]').forEach(function(b){
    b.addEventListener('click', function(){
      const c = CARS.find(function(x){ return x.id === b.getAttribute('data-buy'); });
      if(S.coins >= c.price){
        S.coins -= c.price;
        S.owned.push(c.id);
        if(S.hpMap[c.id] === undefined) S.hpMap[c.id] = c.hp;
        save(); openGarage();
      }
    });
  });
  garageOverlay.querySelectorAll('[data-sel]').forEach(function(b){
    b.addEventListener('click', function(){
      S.cur = b.getAttribute('data-sel');
      save(); startRun();
    });
  });
}

// сломанная машина
function openBroken(){
  only(brokenOverlay);
  const cur = curCar();
  const cost = cur.rep;
  brokenOverlay.querySelector('.panel').innerHTML =
    '<h2>💥 Машина разбита!</h2><p>Ремонт обойдётся в 🪙' + cost + '.<br>У вас: 🪙' + S.coins + '</p>' +
    '<div class="btnrow"><button id="fixNow" ' + (S.coins < cost ? 'class="noB"' : '') + '>Починить 🪙' + cost + '</button>' +
    '<button id="goGar">В гараж</button></div>';
  const f = brokenOverlay.querySelector('#fixNow');
  f.addEventListener('click', function(){
    if(S.coins >= cost){
      S.coins -= cost; hp = hpMax(); S.hpMap[S.cur] = hpMax(); dents = []; broken = false;
      save(); startRun();
    }
  });
  brokenOverlay.querySelector('#goGar').addEventListener('click', function(){ broken = false; openGarage(); });
}

// настройки графики
function openSettings(){
  only(settingsOverlay);
  const q = S.settings.quality, u = S.settings.ui;
  settingsOverlay.querySelector('.panel').innerHTML =
    '<h2>⚙️ Графика</h2>' +
    '<p class="hint">Качество картинки — если игра тормозит, поставьте «Низкое». «Авто» само подберёт под ваше устройство.</p>' +
    '<div class="optgrp">' +
    [['auto','Авто'],['low','Низкое'],['medium','Среднее'],['high','Высокое']].map(function(o){
      return '<button class="opt' + (q === o[0] ? ' on' : '') + '" data-q="' + o[0] + '">' + o[1] + '</button>';
    }).join('') + '</div>' +
    '<p class="hint">Размер интерфейса — кнопки и надписи.</p>' +
    '<div class="optgrp">' +
    [['small','Мелкий'],['normal','Обычный'],['big','Крупный']].map(function(o){
      return '<button class="opt' + (u === o[0] ? ' on' : '') + '" data-u="' + o[0] + '">' + o[1] + '</button>';
    }).join('') + '</div>' +
    '<div class="btnrow"><button id="setClose">Готово</button></div>';
  settingsOverlay.querySelectorAll('[data-q]').forEach(function(b){
    b.addEventListener('click', function(){
      S.settings.quality = b.getAttribute('data-q');
      save(); resize(); openSettings();
    });
  });
  settingsOverlay.querySelectorAll('[data-u]').forEach(function(b){
    b.addEventListener('click', function(){
      S.settings.ui = b.getAttribute('data-u');
      save(); applyUi(); openSettings();
    });
  });
  settingsOverlay.querySelector('#setClose').addEventListener('click', hideAll);
}

// админ-панель (только оригинал)
function openAdmin(){
  only(adminOverlay);
  adminOverlay.querySelector('.panel').innerHTML =
    '<h2>🔑 Админ-панель</h2>' +
    '<p class="hint">Введите пароль администратора:</p>' +
    '<input type="password" id="admPass" placeholder="Пароль">' +
    '<div id="admTools" style="display:none">' +
    '<div class="btnrow"><button id="admCoins">🪙 +1000 монет</button></div>' +
    '<div class="btnrow"><button id="admFix">🔧 Починить машину</button></div>' +
    '<div class="btnrow"><button id="admAll">🚙 Открыть все машины</button></div>' +
    '<div class="btnrow"><button id="admWipe" class="danger">❌ УДАЛИТЬ ВЕСЬ ПРОГРЕСС</button></div>' +
    '</div>' +
    '<div class="btnrow"><button id="admClose">Закрыть</button></div>';
  adminOverlay.querySelector('#admClose').addEventListener('click', hideAll);
  adminOverlay.querySelector('#admPass').addEventListener('input', function(){
    if(adminOverlay.querySelector('#admPass').value === ADMIN_PASS){
      adminOverlay.querySelector('#admTools').style.display = 'block';
      adminOverlay.querySelector('#admPass').style.display = 'none';
    }
  });
  adminOverlay.querySelector('#admCoins').addEventListener('click', function(){ S.coins += 1000; save(); hud(); });
  adminOverlay.querySelector('#admFix').addEventListener('click', function(){
    hp = hpMax(); S.hpMap[S.cur] = hpMax(); dents = []; broken = false; save();
  });
  adminOverlay.querySelector('#admAll').addEventListener('click', function(){
    S.owned = CARS.map(function(c){ return c.id; });
    CARS.forEach(function(c){ if(S.hpMap[c.id] === undefined) S.hpMap[c.id] = c.hp; });
    save();
  });
  adminOverlay.querySelector('#admWipe').addEventListener('click', function(){
    if(confirm('Точно удалить ВЕСЬ прогресс? Это нельзя отменить!')){
      try{ localStorage.removeItem(SAVE_KEY); }catch(e){}
      location.reload();
    }
  });
}

// старт
document.getElementById('startTitle').textContent = IS_BETA ? 'АВТОХЛАМ — BETA 💨' : 'АВТОХЛАМ';
if(!IS_BETA) document.getElementById('btnNitro').style.display = 'none';
if(!IS_BETA) document.getElementById('fuelWrap').style.display = 'none';
startOverlay.style.display = 'flex';
paused = true;

</script>
</body>
</





















..







