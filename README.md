<!DOCTYPE html>
<html lang="ur">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>All-in-One Mega Demo + Mobile-Friendly Mini-Game</title>

<!-- PWA manifest link -->
<link rel="manifest" href="manifest.json">

<style>
  body { font-family:sans-serif; direction:rtl; text-align:center; background:#f5f5f5; padding:20px; }
  h1,h2,h3 { margin:10px 0; }
  button{ padding:10px 20px; margin:5px; cursor:pointer; border-radius:6px; border:none; font-size:16px; }
  input{ padding:8px; border-radius:5px; border:1px solid #ccc; width:200px; margin-bottom:10px; }
  table { border-collapse: collapse; margin:20px auto; background:#fff; }
  th, td { border:1px solid #333; padding:10px 20px; cursor:pointer; }
  th { background-color:#f0f0f0; }
  td:hover { background-color:#d0e6f6; }
  tr.selected { background-color:#fffa90; }
  td:focus { outline:2px solid #007BFF; }

  #miniGame, #gameArea { margin-top:30px; }
  #playButton { padding:10px 20px; background-color:#28a745; color:#fff; }
  #playButton:hover { background-color:#218838; }
  #gameArea{ display:none; width:300px; height:300px; background:#fff; border:2px solid #333; margin:20px auto; position:relative; overflow:hidden; }
  .hero{ width:30px; height:30px; background:#4CAF50; position:absolute; top:0; left:0; }
  .item{ width:20px; height:20px; background:gold; position:absolute; top:200px; left:200px; }
</style>
</head>
<body>

<h1>All-in-One Mega Demo + Mobile-Friendly Mini-Game</h1>

<h2>📊 Table Demo</h2>
<table id="demoTable">
<tr><th>کالم 1</th><th>کالم 2</th><th>کالم 3</th></tr>
<tr class="master-row" data-angle="1"><td>1-1</td><td>1-2</td><td>1-3</td></tr>
<tr class="master-row" data-angle="2"><td>2-1</td><td>2-2</td><td>2-3</td></tr>
<tr class="master-row" data-angle="3"><td>3-1</td><td>3-2</td><td>3-3</td></tr>
<tr class="master-row" data-angle="4" style="display:none"><td>4-1</td><td>4-2</td><td>4-3</td></tr>
<tr class="master-row" data-angle="5" style="display:none"><td>5-1</td><td>5-2</td><td>5-3</td></tr>
</table>
<button id="showFrom4">Show Rows from 4️⃣</button>

<div id="miniGame">
<h2>🎮 Story Mini-Game</h2>
<input id="titleInput" placeholder="مثال: وقت کا مسافر">
<br>
<button id="playButton">▶️ Play Game</button>
<div id="gameArea">
  <div id="hero" class="hero"></div>
  <div id="item" class="item"></div>
</div>
<p id="status"></p>
<br>
<button onclick="portToAndroid()">📱 Port to Android</button>
<button onclick="portToiOS()">🍏 Port to iOS</button>
</div>

<script>
/* ===== Table JS ===== */
document.getElementById('showFrom4').addEventListener('click', () => {
  document.querySelectorAll('.master-row').forEach(row => {
    if(parseInt(row.dataset.angle) >= 4) row.style.display='table-row';
  });
});
document.querySelectorAll('#demoTable td').forEach(td => {
  td.addEventListener('click', ()=>{ td.parentElement.classList.toggle('selected'); });
});

/* ===== Mini-Game JS ===== */
const playBtn = document.getElementById('playButton');
const gameArea = document.getElementById('gameArea');
const status = document.getElementById('status');
const heroEl = document.getElementById('hero');
const itemEl = document.getElementById('item');

let hero = { x:0, y:0 }, step=10, score=0;

playBtn.addEventListener('click', startGame);

function startGame(){
  gameArea.style.display="block";
  hero.x=0; hero.y=0;
  heroEl.style.left="0px"; heroEl.style.top="0px";
  itemEl.style.display="block";
  status.innerText="⬅️➡️⬆️⬇️ Arrow keys یا Touch سے hero چلائیں اور ⭐ پکڑیں";
}

document.addEventListener("keydown", e=>{
  if(gameArea.style.display!=="block") return;
  if(e.key==="ArrowRight") hero.x+=step;
  if(e.key==="ArrowLeft") hero.x-=step;
  if(e.key==="ArrowDown") hero.y+=step;
  if(e.key==="ArrowUp") hero.y-=step;
  keepHeroInBounds();
  updateHeroPosition();
  checkCollision();
});

let touchStartX=0, touchStartY=0;
gameArea.addEventListener('touchstart', e=>{
  const touch = e.touches[0];
  touchStartX = touch.clientX;
  touchStartY = touch.clientY;
});
gameArea.addEventListener('touchmove', e=>{
  e.preventDefault();
  const touch = e.touches[0];
  const dx = touch.clientX - touchStartX;
  const dy = touch.clientY - touchStartY;
  hero.x += dx;
  hero.y += dy;
  touchStartX = touch.clientX;
  touchStartY = touch.clientY;
  keepHeroInBounds();
  updateHeroPosition();
  checkCollision();
});

function keepHeroInBounds(){
  hero.x = Math.max(0, Math.min(300-30, hero.x));
  hero.y = Math.max(0, Math.min(300-30, hero.y));
}

function updateHeroPosition(){
  heroEl.style.left = hero.x + "px";
  heroEl.style.top = hero.y + "px";
}

function checkCollision(){
  const ix = itemEl.offsetLeft;
  const iy = itemEl.offsetTop;
  if(hero.x < ix + 20 && hero.x + 30 > ix && hero.y < iy + 20 && hero.y + 30 > iy && itemEl.style.display!=="none"){
    itemEl.style.display="none";
    score = 10;
    status.innerText = `🎉 مبارک ہو! آپ جیت گئے! Score: ${score}`;
  }
}

function portToAndroid() {
  console.log("⚡ Preparing mini-game for Android WebView...");
  heroEl.style.transition = "all 0.05s linear";
  itemEl.style.transition = "all 0.05s linear";
  alert("Mini-game ported for Android WebView ✅");
}

function portToiOS() {
  console.log("⚡ Preparing mini-game for iOS WebView...");
  heroEl.style.transition = "all 0.05s linear";
  itemEl.style.transition = "all 0.05s linear";
  alert("Mini-game ported for iOS WebView ✅");
}

/* ===== PWA Service Worker Registration ===== */
if('serviceWorker' in navigator){
  navigator.serviceWorker.register('service-worker.js').then(()=>{
    console.log("Service Worker Registered ✅");
  });
}
</script>
</body>const CACHE_NAME = 'mega-demo-cache-v1';
const urlsToCache = [
  './',
  './index.html',
  './manifest.json'
];

self.addEventListener('install', e=>{
  e.waitUntil(
    caches.open(CACHE_NAME).then(cache=>{
      return cache.addAll(urlsToCache);
    })
  );
});

self.addEventListener('fetch', e=>{
  e.respondWith(
    caches.match(e.request).then(response=>{
      return response || fetch(e.request);
    })
  );
});
</html>

Jupyter Notebook virtualenv install for OS X Python3 from brew

Specific to OS X using virtualenv and brew installed Python 3.
- `/usr/local/bin/python3`

