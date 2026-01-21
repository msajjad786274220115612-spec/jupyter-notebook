# jupyter-notebook

Jupyter Notebook virtualenv install for OS X Python3 from brew

Specific to OS X using virtualenv and brew installed Python 3.
- `/usr/local/bin/python3`

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>MegaDemo Shooter + File Download</title>
<style>
body{margin:0;overflow:hidden;background:#000;color:#fff;font-family:sans-serif;}
#score{position:absolute;top:10px;left:10px;font-size:22px;z-index:1000;}
#gameOver{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:36px;color:#f00;display:none;text-align:center;z-index:1000;}
canvas{display:block;background:#111;}
#restartTip{font-size:18px;color:#fff;display:none;}
#fileBoxContainer{
    position:absolute;
    top:10px;
    right:10px;
    z-index:1001;
    background:#222;
    padding:5px;
    border-radius:5px;
}
#fileLink{padding:5px;font-size:16px;}
#fileActions{margin-top:5px;font-size:14px;color:#0f0;}
#fileShowBtn{padding:5px 10px;font-size:16px;cursor:pointer;}
</style>
</head>
<body>

<div id="score">Score: 0 | Level: 1</div>
<div id="gameOver">GAME OVER 🔥<br><span id="restartTip">Tap or Press R to Restart</span></div>

<!-- File Download Box -->
<div id="fileBoxContainer">
  <input type="text" id="fileLink" placeholder="فائل کا لنک ڈالیں">
  <button id="fileShowBtn" onclick="processFile()">Show</button>
  <div id="fileActions"></div>
</div>

<canvas id="gameCanvas"></canvas>

<script>
// ====== Canvas Setup ======
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
function resizeCanvas(){canvas.width=window.innerWidth; canvas.height=window.innerHeight;}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

// ====== Sprites & Sounds (Base64) ======
const playerImg = new Image();
playerImg.src = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAABl0RVh0Q3JlYXRpb24gVGltZQAwOS8yMi8xOBl3ebwAAAC9SURBVHja7NaxDcAwDADQ-o3/2U02tEIUaxVAYDp0HtB7JOtPAPAEDABAwAEDAAQMABAwAEDAAQMABAwAEDAAQMBBOAJSu0uWBUApLxG+7wAAAABJRU5ErkJggg==";

const enemyImg = new Image();
enemyImg.src = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAABl0RVh0Q3JlYXRpb24gVGltZQAwOS8yMi8xOBl3ebwAAACBSURBVHja7NaxDcAwDADQ-o3/2U02tEIUaxVWAYDp0HtB7JOtPAJDAAEDAAQMABAwAEDAAQMABAwAEDAAQMABAwAEDAAQMBP4B1hcgBAgAEAwAEDAAQMABAwAEDAAQMABAwEDg7AAVtS2c2+slnAAAAAElFTkSuQmCC";

const shootSound = new Audio("data:audio/wav;base64,UklGRhQAAABXQVZFZm10IBAAAAABAAEAQB8AAIA+AAACABAAZGF0YQgAAAAA");
const hitSound   = new Audio("data:audio/wav;base64,UklGRgAAAABXQVZFZm10IBAAAAABAAEAQB8AAIA+AAACABAAZGF0YQAAAAA=");
const overSound  = new Audio("data:audio/wav;base64,UklGRhQAAABXQVZFZm10IBAAAAABAAEAQB8AAIA+AAACABAAZGF0YQgAAAAA");

// ====== Game Variables ======
let player = {x:180,y:580,w:40,h:40,speed:6};
let bullets = [], enemies = [];
let score=0, level=1, gameOver=false;
let touchX=null;

// ====== Spawn Enemies ======
function spawnEnemy(){
    if(gameOver) return;
    let x=Math.random()*(canvas.width-30);
    enemies.push({x:x,y:-40,w:32,h:32,speed:2+level*0.3});
    setTimeout(spawnEnemy, Math.max(300,1200 - level*50));
}
spawnEnemy();

// ====== Input Handling ======
document.addEventListener("keydown",e=>{
    if(gameOver && e.key.toLowerCase()=="r"){ resetGame(); }
    if(gameOver) return;
    if(e.key=="ArrowLeft") player.x -= player.speed;
    if(e.key=="ArrowRight") player.x += player.speed;
    if(e.key==" ") { shootSound.currentTime=0; shootSound.play(); bullets.push({x:player.x+14,y:player.y-10,w:8,h:16}); }
});
canvas.addEventListener("touchstart",e=>{ touchX=e.touches[0].clientX; });
canvas.addEventListener("touchmove",e=>{
    let dx=e.touches[0].clientX - touchX;
    player.x += dx; touchX=e.touches[0].clientX;
});
canvas.addEventListener("touchend",e=>{
    touchX=null; shootSound.currentTime=0; shootSound.play();
    bullets.push({x:player.x+14,y:player.y-10,w:8,h:16});
});

// ====== Collision Detection ======
function collides(a,b){return a.x < b.x+b.w && a.x+a.w > b.x && a.y < b.y+b.h && a.y+a.h > b.y;}

// ====== Game Loop ======
function update(){
    if(gameOver) return;
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.drawImage(playerImg,player.x,player.y,player.w,player.h);

    bullets.forEach((b,i)=>{
        ctx.fillStyle="yellow"; ctx.fillRect(b.x,b.y,b.w,b.h);
        b.y-=8;
        if(b.y < -20) bullets.splice(i,1);
    });

    enemies.forEach((en,i)=>{
        ctx.drawImage(enemyImg,en.x,en.y,en.w,en.h);
        en.y += en.speed;

        if(collides(player,en)){
            gameOver = true; overSound.play();
            document.getElementById("gameOver").style.display="block";
            document.getElementById("restartTip").style.display="block";
        }

        bullets.forEach((b,bi)=>{
            if(collides(b,en)){
                hitSound.play();
                bullets.splice(bi,1);
                enemies.splice(i,1);
                score+=10;
                if(score % 50 == 0) level++;
            }
        });
    });

    document.getElementById("score").innerText = `Score: ${score} | Level: ${level}`;
    requestAnimationFrame(update);
}
update();

// ====== Reset Game ======
function resetGame(){
    bullets=[]; enemies=[]; score=0; level=1; gameOver=false;
    document.getElementById("gameOver").style.display="none";
    spawnEnemy();
}

// ====== File Download/Open ======
function processFile(){
    const link=document.getElementById('fileLink').value.trim();
    if(!link){ alert('براہ کرم لنک ڈالیں'); return; }
    const fileName=link.split('/').pop();
    const fileType=fileName.split('.').pop();
    document.getElementById('fileActions').innerHTML=`
        <p>File: ${fileName}</p>
        <p>Type: ${fileType}</p>
        <button onclick="downloadFile('${link}')">Download ⬇️</button>
        <button onclick="openFile('${link}')">Open 📂</button>
    `;
}
function downloadFile(url){
    const a=document.createElement('a');
    a.href=url;
    a.download=url.split('/').pop();
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
}
function openFile(url){ window.open(url,'_blank'); }

</script>
</body>
</html>
