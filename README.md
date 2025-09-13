<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<title>2D山地骑行游戏</title>
<style>
body { margin:0; overflow:hidden; background:#87ceeb; touch-action:none; }
#score { position:absolute; top:10px; left:10px; font-size:20px; font-family:Arial, sans-serif; color:#fff; z-index:10;}
.controls { position:absolute; bottom:10px; left:50%; transform:translateX(-50%); z-index:10; }
button { font-size:16px; padding:8px 12px; margin:5px; }
.virtual-btn { position:absolute; bottom:60px; width:60px; height:60px; font-size:24px; opacity:0.5; background:#000; color:#fff; border-radius:50%; text-align:center; line-height:60px; user-select:none; }
#leftBtn { left:30px; }
#rightBtn { right:30px; }
canvas { display:block; }
</style>
</head>
<body>
<div id="score">分数: 0</div>
<div class="controls">
  <button onclick="restartGame()">重新开始</button>
  <button onclick="exitGame()">退出游戏</button>
</div>
<div id="leftBtn" class="virtual-btn">◀</div>
<div id="rightBtn" class="virtual-btn">▶</div>

<canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

let dpr = window.devicePixelRatio || 1;

// 设置 Canvas 尺寸并缩放
function resizeCanvas(){
  dpr = window.devicePixelRatio || 1;
  canvas.width = window.innerWidth * dpr;
  canvas.height = window.innerHeight * dpr;
  canvas.style.width = window.innerWidth + 'px';
  canvas.style.height = window.innerHeight + 'px';
  ctx.setTransform(1,0,0,1,0,0); 
  ctx.scale(dpr,dpr);
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

// 玩家小车
let car = {x: window.innerWidth/2, y: window.innerHeight-100, width:40, height:20};
let keys = {left:false, right:false};
let obstacles = [];
let scroll = 0;
let score = 0;
let gameOver = false;

// 键盘控制
document.addEventListener('keydown', e=>{
  if(e.key==='ArrowLeft'||e.key==='a') keys.left=true;
  if(e.key==='ArrowRight'||e.key==='d') keys.right=true;
});
document.addEventListener('keyup', e=>{
  if(e.key==='ArrowLeft'||e.key==='a') keys.left=false;
  if(e.key==='ArrowRight'||e.key==='d') keys.right=false;
});

// 手机触屏滑动
let touchStartX = 0;
canvas.addEventListener('touchstart', e=>{
  touchStartX = e.touches[0].clientX;
});
canvas.addEventListener('touchmove', e=>{
  const dx = e.touches[0].clientX - touchStartX;
  car.x += dx * 0.1;
  if(car.x<20) car.x=20;
  if(car.x>window.innerWidth-20) car.x=window.innerWidth-20;
  touchStartX = e.touches[0].clientX;
},{passive:false});

// 手机虚拟按键
const leftBtn = document.getElementById('leftBtn');
const rightBtn = document.getElementById('rightBtn');
leftBtn.addEventListener('touchstart', ()=>{ keys.left=true; });
leftBtn.addEventListener('touchend', ()=>{ keys.left=false; });
rightBtn.addEventListener('touchstart', ()=>{ keys.right=true; });
rightBtn.addEventListener('touchend', ()=>{ keys.right=false; });

// 生成障碍物
function createObstacle(){
  const x = Math.random()*(window.innerWidth-40)+20;
  obstacles.push({x:x, y:-20, width:30, height:20});
}

// 重新开始
function restartGame(){
  if(confirm("是否重新开始游戏？")){
    obstacles=[];
    car.x = window.innerWidth/2;
    score = 0;
    scroll = 0;
    gameOver = false;
    document.getElementById('score').innerText = "分数: 0";
  }
}

// 退出游戏
function exitGame(){
  if(confirm("是否确认退出游戏？")){
    gameOver=true;
    alert("游戏已退出！");
    window.close();
    setTimeout(()=>{ location.reload(); },200);
  }
}

// 更新游戏
function update(){
  if(gameOver) return;

  if(keys.left) car.x -= 5;
  if(keys.right) car.x += 5;
  if(car.x<20) car.x=20;
  if(car.x>window.innerWidth-20) car.x=window.innerWidth-20;

  scroll += 5;
  if(scroll % 60 === 0) createObstacle();
  score = Math.floor(scroll / 5);
  document.getElementById('score').innerText = "分数: "+score;

  // 移动障碍物
  for(let i=0;i<obstacles.length;i++){
    obstacles[i].y += 5;
    if(obstacles[i].y + obstacles[i].height > car.y &&
       obstacles[i].y < car.y + car.height &&
       obstacles[i].x + obstacles[i].width > car.x - car.width/2 &&
       obstacles[i].x < car.x + car.width/2){
      gameOver = true;
      alert("游戏结束! 得分: "+score);
    }
  }
  obstacles = obstacles.filter(o=>o.y<window.innerHeight+20);
}

// 绘制山路
function drawRoad(){
  ctx.fillStyle="#654321";
  ctx.beginPath();
  ctx.moveTo(0, window.innerHeight);
  for(let i=0;i<window.innerWidth;i+=10){
    const y = window.innerHeight/2 + Math.sin((i+scroll*0.5)/50)*50;
    ctx.lineTo(i, y);
  }
  ctx.lineTo(window.innerWidth, window.innerHeight);
  ctx.closePath();
  ctx.fill();
}

// 绘制游戏
function draw(){
  ctx.clearRect(0,0,window.innerWidth, window.innerHeight);
  drawRoad();
  // 玩家小车
  ctx.fillStyle = "green";
  ctx.fillRect(car.x-car.width/2, car.y, car.width, car.height);
  // 障碍物
  ctx.fillStyle = "red";
  for(let o of obstacles){
    ctx.fillRect(o.x, o.y, o.width, o.height);
  }
}

// 游戏循环
function loop(){
  update();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
