<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Space Dodger</title>
<style>
  :root{
    --bg:#04060a;
    --ui:#e6f7ff;
    --accent:#5eead4;
    --danger:#ff7b7b;
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,Arial,sans-serif;background:linear-gradient(180deg,#00121a 0%, var(--bg) 70%);color:var(--ui)}
  #game-wrap{display:flex;flex-direction:column;align-items:center;justify-content:center;height:100%;gap:12px;padding:20px;box-sizing:border-box}
  canvas{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.00));border-radius:12px;box-shadow:0 8px 30px rgba(0,0,0,0.6);max-width:100%;height:auto}
  .hud{display:flex;gap:10px;align-items:center;width:100%;max-width:700px;justify-content:space-between}
  .panel{background:rgba(255,255,255,0.03);padding:8px 12px;border-radius:10px;font-size:14px}
  .controls{display:flex;gap:8px;align-items:center}
  button{background:transparent;border:1px solid rgba(255,255,255,0.08);color:var(--ui);padding:6px 10px;border-radius:8px;cursor:pointer}
  button:hover{border-color:rgba(255,255,255,0.15)}
  .big{font-weight:700;font-size:18px}
  .center{display:flex;flex-direction:column;align-items:center;gap:8px;margin-top:8px}
  .overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none}
  .message{pointer-events:auto;background:rgba(2,6,10,0.9);padding:20px;border-radius:12px;color:var(--ui);text-align:center;min-width:280px;box-shadow:0 12px 40px rgba(0,0,0,0.7)}
  .message h2{margin:0 0 8px 0}
  .small{font-size:13px;color:rgba(230,247,255,0.8)}
  footer{position:fixed;left:12px;bottom:12px;font-size:12px;color:rgba(230,247,255,0.6)}
  @media (max-width:520px){
    .hud{flex-direction:column;gap:6px}
    canvas{width:100%}
  }
</style>
</head>
<body>
<div id="game-wrap">
  <div class="hud">
    <div class="panel big">Score: <span id="score">0</span></div>
    <div class="controls">
      <button id="startBtn">Start</button>
      <button id="pauseBtn">Pause</button>
      <button id="resetBtn">Reset</button>
    </div>
    <div class="panel">High: <span id="high">0</span></div>
  </div>

  <canvas id="game" width="700" height="420"></canvas>

  <div class="center small">
    <div>Controls: ← → or A / D — Touch: drag left/right</div>
    <div class="small">Dodge asteroids. Survive to increase score!</div>
  </div>
</div>

<footer>Space Dodger — simple HTML5 canvas game</footer>

<div id="overlay" class="overlay" aria-hidden="true">
  <div id="message" class="message" style="display:none">
    <h2 id="msgTitle">Paused</h2>
    <div id="msgBody" class="small">Press Start to play.</div>
    <div style="margin-top:12px">
      <button id="msgStart">Start</button>
      <button id="msgResume">Resume</button>
    </div>
  </div>
</div>

<script>
/*
  Space Dodger
  - Single-file game
  - Keyboard controls: left/right or A/D
  - Touch: drag on canvas
  - Local high score in localStorage
  - Save as "space-dodger.html" and open in browser
*/

(() => {
  // Canvas setup
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d', { alpha: false });
  let W = canvas.width;
  let H = canvas.height;

  // UI
  const scoreEl = document.getElementById('score');
  const highEl = document.getElementById('high');
  const startBtn = document.getElementById('startBtn');
  const pauseBtn = document.getElementById('pauseBtn');
  const resetBtn = document.getElementById('resetBtn');
  const overlay = document.getElementById('overlay');
  const message = document.getElementById('message');
  const msgTitle = document.getElementById('msgTitle');
  const msgBody = document.getElementById('msgBody');
  const msgStart = document.getElementById('msgStart');
  const msgResume = document.getElementById('msgResume');

  // Game state
  let running = false;
  let paused = false;
  let lastTime = 0;
  let score = 0;
  let highScore = Number(localStorage.getItem('sd_high') || 0);
  highEl.textContent = highScore;

  // Player
  const player = {
    w: 44,
    h: 30,
    x: W / 2,
    y: H - 60,
    speed: 420, // pixels/sec
    vx: 0
  };

  // Controls
  const keys = { left:false, right:false };
  let touchDragging = false;

  // Asteroids
  const asteroids = [];
  let spawnTimer = 0;
  let spawnInterval = 0.9; // seconds
  let difficultyTimer = 0;
  let asteroidSpeedBase = 110;

  // Visual helpers
  function clear() {
    ctx.fillStyle = '#02111a';
    ctx.fillRect(0,0,W,H);
  }

  function drawShip(x,y,w,h) {
    // simple triangle ship
    ctx.save();
    ctx.translate(x,y);
    ctx.beginPath();
    ctx.moveTo(0, -h/2);
    ctx.lineTo(w/2, h/2);
    ctx.lineTo(-w/2, h/2);
    ctx.closePath();
    // gradient
    const g = ctx.createLinearGradient(-w/2,-h/2,w/2,h/2);
    g.addColorStop(0,'#7ee7ff');
    g.addColorStop(1,'#5eead4');
    ctx.fillStyle = g;
    ctx.fill();
    // cockpit
    ctx.fillStyle = 'rgba(2,6,10,0.6)';
    ctx.beginPath();
    ctx.ellipse(0,-h*0.05,w*0.22,h*0.22,0,0,Math.PI*2);
    ctx.fill();
    ctx.restore();
  }

  function drawAsteroid(a) {
    ctx.save();
    ctx.translate(a.x,a.y);
    ctx.rotate(a.rot);
    ctx.fillStyle = '#8a6e5a';
    ctx.beginPath();
    // irregular polygon
    const r = a.r;
    const spikes = 7;
    for(let i=0;i<spikes;i++){
      const ang = i/(spikes)*Math.PI*2;
      const rr = r*(0.75 + 0.5 * Math.sin(ang*3 + a.seed));
      ctx.lineTo(Math.cos(ang)*rr, Math.sin(ang)*rr);
    }
    ctx.closePath();
    ctx.fill();
    ctx.restore();
  }

  function spawnAsteroid() {
    const r = rand(16,42);
    const x = rand(r, W - r);
    const speed = asteroidSpeedBase + Math.random()*120 + score*0.8;
    asteroids.push({
      x, y: -r - 10, r,
      vy: speed,
      rot: (Math.random()-0.5)*0.03,
      seed: Math.random()*10
    });
  }

  function rand(a,b){return a + Math.random()*(b-a);}

  // Collisions
  function circleRect(cx,cy,cr,rx,ry,rw,rh){
    // closest point
    const closestX = Math.max(rx, Math.min(cx, rx+rw));
    const closestY = Math.max(ry, Math.min(cy, ry+rh));
    const dx = cx - closestX;
    const dy = cy - closestY;
    return (dx*dx + dy*dy) < (cr*cr);
  }

  // Resize handling
  function onResize(){
    // preserve aspect ratio of canvas by scaling CSS while keeping internal resolution
    const maxW = Math.min(window.innerWidth - 40, 900);
    const scale = Math.min(maxW / canvas.width, 1);
    canvas.style.width = Math.round(canvas.width * scale) + 'px';
  }
  window.addEventListener('resize', onResize);
  onResize();

  // Input
  window.addEventListener('keydown', e => {
    if(e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a'){ keys.left = true; }
    if(e.key === 'ArrowRight' || e.key.toLowerCase() === 'd'){ keys.right = true; }
    if(e.key === ' '){ togglePause(); }
  });
  window.addEventListener('keyup', e => {
    if(e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a'){ keys.left = false; }
    if(e.key === 'ArrowRight' || e.key.toLowerCase() === 'd'){ keys.right = false; }
  });

  // Touch: drag to move
  let lastTouchX = null;
  canvas.addEventListener('touchstart', e => {
    touchDragging = true;
    lastTouchX = e.touches[0].clientX;
  }, {passive:true});
  canvas.addEventListener('touchmove', e => {
    if(!touchDragging) return;
    const t = e.touches[0];
    const rect = canvas.getBoundingClientRect();
    // translate to canvas coordinates
    const canvasX = ((t.clientX - rect.left)/rect.width) * W;
    player.x = clamp(canvasX, player.w/2, W - player.w/2);
    e.preventDefault();
  }, {passive:false});
  canvas.addEventListener('touchend', e => { touchDragging = false; lastTouchX = null; });

  // Buttons
  startBtn.addEventListener('click', startGame);
  pauseBtn.addEventListener('click', togglePause);
  resetBtn.addEventListener('click', resetGame);
  msgStart.addEventListener('click', () => { hideMessage(); startGame(); });
  msgResume.addEventListener('click', () => { hideMessage(); resume(); });

  function showMessage(title, body){
    msgTitle.textContent = title;
    msgBody.textContent = body;
    message.style.display = 'block';
    overlay.style.pointerEvents = 'auto';
  }
  function hideMessage(){
    message.style.display = 'none';
    overlay.style.pointerEvents = 'none';
  }

  // Game loop
  function startGame(){
    if(running) return;
    running = true;
    paused = false;
    score = 0;
    asteroids.length = 0;
    spawnTimer = 0;
    difficultyTimer = 0;
    asteroidSpeedBase = 110;
    player.x = W/2;
    lastTime = performance.now();
    hideMessage();
    requestAnimationFrame(loop);
  }

  function resetGame(){
    running = false;
    paused = false;
    score = 0;
    asteroids.length = 0;
    spawnTimer = 0;
    updateUI();
    showMessage('Reset', 'Game reset. Press Start to play.');
  }

  function gameOver(){
    running = false;
    paused = false;
    if(score > highScore){
      highScore = Math.floor(score);
      localStorage.setItem('sd_high', String(highScore));
      highEl.textContent = highScore;
    }
    showMessage('Game Over', `Score: ${Math.floor(score)}\nPress Start to try again.`);
  }

  function togglePause(){
    if(!running) return;
    if(paused) resume();
    else pause();
  }
  function pause(){
    paused = true;
    showMessage('Paused', 'Game is paused.');
  }
  function resume(){
    if(!running) return;
    paused = false;
    hideMessage();
    lastTime = performance.now();
    requestAnimationFrame(loop);
  }

  function updateUI(){
    scoreEl.textContent = Math.floor(score);
    highEl.textContent = highScore;
  }

  function loop(now){
    if(!running) return;
    const dt = Math.min(0.05, (now - lastTime) / 1000);
    lastTime = now;
    if(!paused){
      step(dt);
      render();
      requestAnimationFrame(loop);
    }
  }

  function step(dt){
    // inputs
    player.vx = 0;
    if(keys.left) player.vx = -player.speed;
    if(keys.right) player.vx = player.speed;
    player.x += player.vx * dt;
    // clamp
    player.x = clamp(player.x, player.w/2, W - player.w/2);

    // spawn asteroids over time
    spawnTimer -= dt;
    if(spawnTimer <= 0){
      spawnAsteroid();
      spawnTimer = spawnInterval * (0.85 + Math.random()*0.3);
    }

    // increase difficulty slowly
    difficultyTimer += dt;
    if(difficultyTimer > 7){
      difficultyTimer = 0;
      asteroidSpeedBase += 8;
      spawnInterval = Math.max(0.35, spawnInterval - 0.06);
    }

    // move asteroids
    for(let i = asteroids.length-1; i >= 0; i--){
      const a = asteroids[i];
      a.y += a.vy * dt;
      a.rot += (a.vy/400)*(Math.random()*0.12 - 0.06);
      // collision: approximate ship as rect
      const shipRect = { x: player.x - player.w/2, y: player.y - player.h/2, w: player.w, h: player.h };
      if(circleRect(a.x, a.y, a.r, shipRect.x, shipRect.y, shipRect.w, shipRect.h)){
        // hit!
        gameOver();
        return;
      }
      // remove off-screen
      if(a.y - a.r > H + 30) {
        asteroids.splice(i,1);
        score += 10; // reward for dodging
      }
    }

    // score increases with time survived
    score += dt * 12;
    updateUI();
  }

  function render(){
    clear();

    // starfield
    drawStars();

    // draw player
    drawShip(player.x, player.y, player.w, player.h);

    // draw asteroids
    for(const a of asteroids) drawAsteroid(a);

    // HUD - subtle
    ctx.fillStyle = 'rgba(255,255,255,0.03)';
    ctx.fillRect(0, H-28, W, 28);
  }

  // Simple starfield
  const stars = Array.from({length:80}, () => ({x:Math.random()*W, y:Math.random()*H, s:Math.random()*1.3+0.2}));
  function drawStars(){
    ctx.fillStyle = '#071821';
    // move some stars slowly
    for(const s of stars){
      s.y += s.s * 0.45;
      if(s.y > H) s.y = -2, s.x = Math.random()*W;
      ctx.fillStyle = `rgba(255,255,255,${0.06 + s.s*0.15})`;
      ctx.fillRect(s.x, s.y, s.s, s.s);
    }
  }

  function clamp(v,a,b){ return Math.max(a, Math.min(b, v)); }

  // Utility: adapt canvas internal resolution to devicePixelRatio for crispness
  function fixHiDPI(){
    const dpr = window.devicePixelRatio || 1;
    const desiredW = canvas.width;
    const desiredH = canvas.height;
    canvas.width = desiredW * dpr;
    canvas.height = desiredH * dpr;
    canvas.style.width = desiredW + 'px';
    canvas.style.height = desiredH + 'px';
    ctx.setTransform(dpr,0,0,dpr,0,0);
    W = desiredW; H = desiredH;
  }
  fixHiDPI();

  // Initialize: show start message
  showMessage('Space Dodger', 'Press Start to play. Use arrow keys or touch to move.');
  // Expose controls for convenience
  updateUI();

  // Keyboard focus workaround
  canvas.tabIndex = 1000;
  canvas.style.outline = 'none';
  canvas.addEventListener('click', ()=>canvas.focus());

  // Prevent scrolling on spacebar or arrow keys when playing
  window.addEventListener('keydown', function(e){
    if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) e.preventDefault();
  }, {passive:false});
})();
</script>
</body>
</html>
