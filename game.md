<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Rocket Crash Game with Bottom Panel</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: #ffffff;
    font-family: sans-serif;
  }
  #rocket {
    position: absolute;
    font-size: 50px;
    pointer-events: none;
    z-index: 2;
  }
  #multiplier {
    position: absolute;
    top: 30px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 80px;
    font-weight: bold;
    color: #00ff00;
    text-shadow:
        0 0 2px #000,
        0 0 5px #00ff00,
        0 0 10px #00ff00,
        0 0 20px #00ff00,
        0 0 40px #00ff00,
        0 0 80px #00ff00;
    z-index: 3;
    pointer-events: none;
  }

  /* Bottom panel wrapper */
  #bottomPanel {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    background: rgba(0,0,0,0.7);
    display: flex;
    justify-content: space-around;
    align-items: center;
    padding: 10px 0;
    box-sizing: border-box;
    z-index: 3;
  }

  .panel {
    display: flex;
    flex-direction: column;
    align-items: center;
    font-size: 18px;
    color: #00ff00;
    text-shadow: 0 0 2px #000, 0 0 5px #00ff00;
    margin: 0 10px;
  }

  .panel input {
    width: 80px;
    font-size: 18px;
    text-align: center;
    border-radius: 5px;
    border: 2px solid #00ff00;
    background: #000;
    color: #00ff00;
  }

  #cashoutBtn {
    padding: 15px 50px;
    font-size: 24px;
    font-weight: bold;
    color: #ffffff;
    background: #ff0000;
    border: none;
    border-radius: 10px;
    cursor: pointer;
  }

  #startBtn {
    position: absolute;
    top: 20px;
    left: 20px;
    padding: 12px 24px;
    font-size: 18px;
    cursor: pointer;
    z-index: 4;
  }
</style>
</head>
<body>
<canvas id="canvas"></canvas>
<div id="rocket">🚀</div>
<div id="multiplier">x1.0</div>

<!-- Bottom panel -->
<div id="bottomPanel">
  <div class="panel">
    <div>Bet</div>
    <input type="number" id="betInput" value="1" min="0.01" step="0.01">
  </div>
  <div class="panel">
    <div>Total</div>
    <div id="totalAmount">$0.00</div>
  </div>
  <div class="panel">
    <div>Won</div>
    <div id="wonAmount">$0.00</div>
  </div>
  <button id="cashoutBtn">CASHOUT</button>
</div>

<button id="startBtn">Launch Rocket</button>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const rocket = document.getElementById('rocket');
const multiplierDisplay = document.getElementById('multiplier');
const startBtn = document.getElementById('startBtn');
const betInput = document.getElementById('betInput');
const totalAmountDisplay = document.getElementById('totalAmount');
const wonAmountDisplay = document.getElementById('wonAmount');
const cashoutBtn = document.getElementById('cashoutBtn');

let startTime = 0;
let crashed = false;
let cashout = false;
let crashMultiplier = 0;
let rocketX = 50, rocketY = 50;
let fallSpeed = 0;
let path = [];
let particles = [];
let animationId;

let betAmount = 1;
let totalAmount = 0;

const launchDuration = 5000;

function resetRocket() {
  betAmount = parseFloat(betInput.value) || 1;
  totalAmount = betAmount;
  startTime = performance.now();
  crashed = false;
  cashout = false;
  crashMultiplier = Math.random() * 2 + 1;
  rocketX = 50;
  rocketY = 50;
  rocket.style.left = rocketX + 'px';
  rocket.style.bottom = rocketY + 'px';
  multiplierDisplay.textContent = 'x1.0';
  totalAmountDisplay.textContent = `$${totalAmount.toFixed(2)}`;
  wonAmountDisplay.textContent = `$0.00`;
  path = [{x: rocketX, y: rocketY}];
  fallSpeed = 0;
  particles = [];
}

function launchRocket() {
  resetRocket();
  if (animationId) cancelAnimationFrame(animationId);

  function animate(currentTime) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    let elapsed = currentTime - startTime;
    let progress = Math.min(elapsed / launchDuration, 1);

    if (!crashed && !cashout) {
      rocketX = 50 + progress * (canvas.width - 100);
      rocketY = 50 + Math.pow(progress, 1.5) * (canvas.height - 100);
      path.push({x: rocketX, y: rocketY});
    } else if (crashed) {
      fallSpeed += 2;
      rocketY -= fallSpeed;
      if (rocketY < -50) rocketY = -50;
      path.push({x: rocketX, y: rocketY});
    }

    rocket.style.left = rocketX + 'px';
    rocket.style.bottom = rocketY + 'px';

    // Flame particles
    if (!crashed && path.length > 1) {
      const last = path[path.length - 1];
      const prev = path[path.length - 2] || last;
      let dx = last.x - prev.x;
      let dy = last.y - prev.y;
      let len = Math.sqrt(dx*dx + dy*dy) || 1;
      dx /= len;
      dy /= len;

      const flameCount = 40;
      for (let i = 0; i < flameCount; i++) {
        const speed = Math.random() * 3 + 1;
        particles.push({
          x: rocketX - dx*10 + (Math.random()-0.5)*2,
          y: rocketY - dy*10 + (Math.random()-0.5)*2,
          size: Math.random() * 3 + 2,
          life: 20 + Math.random() * 20,
          vx: -dx * speed + (Math.random()-0.5)*0.5,
          vy: -dy * speed + (Math.random()-0.5)*0.5,
          color: `rgba(255,${Math.floor(Math.random()*150)},0,1)`
        });
      }
    }

    // Update particles
    for (let i = particles.length - 1; i >= 0; i--) {
      const p = particles[i];
      ctx.beginPath();
      ctx.arc(p.x, canvas.height - p.y, p.size, 0, Math.PI*2);
      ctx.fillStyle = p.color;
      ctx.shadowBlur = 10;
      ctx.shadowColor = p.color;
      ctx.fill();
      ctx.shadowBlur = 0;

      p.x += p.vx;
      p.y += p.vy;
      p.size *= 0.95;
      p.life--;
      if (p.life <= 0 || p.size <= 0.5) particles.splice(i,1);
    }

    // Stripe tail
    if (path.length > 1) {
      ctx.beginPath();
      ctx.moveTo(path[0].x, canvas.height - path[0].y);
      for (let i = 1; i < path.length; i++) {
        ctx.lineTo(path[i].x, canvas.height - path[i].y);
      }
      ctx.strokeStyle = crashed ? 'red' : 'green';
      ctx.lineWidth = 6;
      ctx.stroke();
    }

    // Multiplier
    let currentMultiplier = 1 + rocketY / canvas.height * 3;
    multiplierDisplay.textContent = 'x' + currentMultiplier.toFixed(2);
    totalAmount = betAmount * currentMultiplier;
    totalAmountDisplay.textContent = `$${totalAmount.toFixed(2)}`;

    if (currentMultiplier >= crashMultiplier && !crashed && !cashout) crashed = true;
    if (crashed) multiplierDisplay.style.color = '#ff0000';

    animationId = requestAnimationFrame(animate);
  }

  animationId = requestAnimationFrame(animate);
}

startBtn.addEventListener('click', launchRocket);

// Cashout button
cashoutBtn.addEventListener('click', () => {
  if (!crashed && !cashout) {
    cashout = true;
    wonAmountDisplay.textContent = `$${totalAmount.toFixed(2)}`;
    multiplierDisplay.style.color = '#00ffff';
  }
});

window.addEventListener('resize', () => {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
});
</script>
</body>
</html>
