# 245001


<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Underwater Fish Buttons</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  min-height: 100vh;
  overflow: hidden;
  font-family: sans-serif;
  color: white;
  background:
    radial-gradient(ellipse at 50% 0%, #39d9ee55, transparent 55%),
    linear-gradient(180deg, #087eac, #03456e 55%, #01243f);
}

#ocean {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.light {
  position: fixed;
  top: -30%;
  left: 15%;
  width: 70%;
  height: 90%;
  background: linear-gradient(
    115deg, transparent 25%, #9df7ff18 40%,
    transparent 55%, #b6faff12 65%, transparent 80%
  );
  filter: blur(20px);
  animation: lightMove 8s ease-in-out infinite alternate;
  pointer-events: none;
}

@keyframes lightMove {
  from { transform: translateX(-8%) rotate(-5deg); }
  to   { transform: translateX(8%) rotate(5deg); }
}

.seaweed {
  position: fixed;
  bottom: -10px;
  font-size: 90px;
  opacity: .45;
  transform-origin: bottom;
  animation: sway 4s ease-in-out infinite alternate;
  pointer-events: none;
}

.seaweed.left { left: 2%; }
.seaweed.right { right: 3%; animation-delay: -2s; }

@keyframes sway {
  from { transform: rotate(-8deg); }
  to   { transform: rotate(8deg); }
}

main {
  position: relative;
  z-index: 2;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

h1 {
  margin: 0 0 12px;
  font-size: clamp(26px, 4vw, 42px);
  text-shadow: 0 3px 18px #001c35;
  letter-spacing: .08em;
}

.subtitle {
  margin: 0 0 45px;
  color: #c9f8ff;
  text-align: center;
}

.buttons {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 22px;
  width: 100%;
  max-width: 950px;
}

.fish-button {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  width: min(250px, 100%);
  min-height: 165px;
  border: 1px solid #b9f8ff85;
  border-radius: 22px;
  color: white;
  cursor: pointer;
  background: linear-gradient(145deg, #50e5ee42, #0063a866);
  box-shadow: 0 8px 30px #001d3d66,
              inset 0 0 25px #a8faff16;
  backdrop-filter: blur(8px);
  transition: transform .25s, box-shadow .25s;
}

.fish-button:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 35px #48e7ff45,
              inset 0 0 25px #a8faff24;
}

.fish-button:active {
  transform: scale(.96);
}

.fish-button .icon {
  display: block;
  font-size: 42px;
  margin-bottom: 12px;
  transition: transform .3s;
}

.fish-button:hover .icon {
  transform: scale(1.15);
}

.fish-button strong {
  display: block;
  font-size: 19px;
  margin-bottom: 8px;
}

.fish-button small {
  color: #c9f8ff;
  font-size: 12px;
}

.status {
  margin-top: 35px;
  color: #c9f8ff;
  font-size: 14px;
  text-align: center;
}

#reset {
  margin-top: 15px;
  border: 1px solid #c5faff77;
  background: #00395766;
  color: white;
  padding: 9px 20px;
  border-radius: 30px;
  cursor: pointer;
}

#reset:hover { background: #1383a9; }

@media (max-width: 650px) {
  main { justify-content: flex-start; padding-top: 8vh; }
  .subtitle { margin-bottom: 25px; }
  .buttons { gap: 12px; }
  .fish-button { width: min(100%, 340px); min-height: 125px; }
  .fish-button .icon { font-size: 30px; margin-bottom: 5px; }
  .seaweed { font-size: 55px; }
}
</style>
</head>

<body>
<canvas id="ocean"></canvas>
<div class="light"></div>
<div class="seaweed left">🌿</div>
<div class="seaweed right">🌿</div>

<main>
  <h1>UNDER THE SEA</h1>
  <p class="subtitle">ボタンを押して、魚たちを呼んでみよう。</p>

  <div class="buttons">
    <button class="fish-button" data-mode="rush">
      <span class="icon">🐠</span>
      <strong>フィッシュラッシュ</strong>
      <small>魚たちが一斉に集まる！</small>
    </button>

    <button class="fish-button" data-mode="feed">
      <span class="icon">🫧</span>
      <strong>餌やりタイム</strong>
      <small>泡と一緒に魚が集まる</small>
    </button>

    <button class="fish-button" data-mode="vortex">
      <span class="icon">🐟</span>
      <strong>神秘の大群</strong>
      <small>光の中で魚たちが舞う</small>
    </button>
  </div>

  <div class="status" id="status">
    海の中は静かです……
  </div>

  <button id="reset">海を静かにする</button>
</main>

<script>
const canvas = document.getElementById("ocean");
const ctx = canvas.getContext("2d");
const status = document.getElementById("status");

let W, H, dpr;
let fish = [];
let bubbles = [];
let particles = [];
let mode = "idle";
let startTime = 0;
let target = { x: 0, y: 0 };
let lastSpawn = 0;

const fishColors = [
  "#ffcb69", "#ff8f9c", "#72f0d1",
  "#9fbcff", "#fff09c", "#ffb5e8"
];

function resize() {
  dpr = Math.min(window.devicePixelRatio || 1, 2);
  W = window.innerWidth;
  H = window.innerHeight;
  canvas.width = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width = W + "px";
  canvas.style.height = H + "px";
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}
window.addEventListener("resize", resize);
resize();

function rand(min, max) {
  return Math.random() * (max - min) + min;
}

function makeFish(x, y, options = {}) {
  const angle = rand(0, Math.PI * 2);
  return {
    x, y,
    vx: Math.cos(angle) * rand(.3, 1.5),
    vy: Math.sin(angle) * rand(.3, 1.5),
    size: rand(13, 24),
    color: fishColors[Math.floor(rand(0, fishColors.length))],
    phase: rand(0, Math.PI * 2),
    speed: rand(1.1, 2.5),
    orbit: rand(25, 100),
    seed: rand(0, 100),
    mode: options.mode || mode,
    ...options
  };
}

function spawnFish(button, count, kind) {
  const rect = button.getBoundingClientRect();
  target = {
    x: rect.left + rect.width / 2,
    y: rect.top + rect.height / 2
  };

  mode = kind;
  startTime = performance.now();
  lastSpawn = startTime;

  for (let i = 0; i < count; i++) {
    let x, y;

    // 魚を画面の外側から登場させる
    const side = Math.floor(Math.random() * 4);
    if (side === 0) { x = rand(0, W); y = -30; }
    if (side === 1) { x = W + 30; y = rand(0, H); }
    if (side === 2) { x = rand(0, W); y = H + 30; }
    if (side === 3) { x = -30; y = rand(0, H); }

    fish.push(makeFish(x, y, {
      mode: kind,
      delay: rand(0, 1000),
      orbit: rand(35, 115),
      size: rand(14, kind === "vortex" ? 27 : 22)
    }));
  }

  // クリック地点から広がる光の粒
  for (let i = 0; i < 28; i++) {
    particles.push({
      x: target.x,
      y: target.y,
      vx: rand(-4, 4),
      vy: rand(-4, 4),
      life: rand(25, 55),
      maxLife: 55,
      color: kind === "vortex" ? "#b8f7ff" : "#ffe9a6",
      size: rand(2, 5)
    });
  }

  if (kind === "rush") {
    status.textContent = "🐠 魚たちが一斉に集まってきた！";
  } else if (kind === "feed") {
    status.textContent = "🫧 餌の匂いに誘われて魚たちが集合中……";
    for (let i = 0; i < 18; i++) {
      bubbles.push({
        x: target.x + rand(-45, 45),
        y: target.y + rand(-15, 20),
        r: rand(3, 8),
        vy: rand(.5, 1.8),
        life: 170
      });
    }
  } else {
    status.textContent = "✨ 神秘の魚群が渦を巻き始めた……";
  }
}

document.querySelectorAll(".fish-button").forEach(button => {
  button.addEventListener("click", () => {
    spawnFish(
      button,
      button.dataset.mode === "vortex" ? 65 : 38,
      button.dataset.mode
    );
  });
});

document.getElementById("reset").addEventListener("click", () => {
  mode = "idle";
  fish = [];
  particles = [];
  bubbles = [];
  status.textContent = "海の中は静かです……";
});

// 背景の泡
for (let i = 0; i < 45; i++) {
  bubbles.push({
    x: rand(0, W),
    y: rand(0, H),
    r: rand(1, 4),
    vy: rand(.3, 1.1),
    life: Infinity
  });
}

function drawBackground(time) {
  // 海中の漂う光
  for (let i = 0; i < 5; i++) {
    const x = W * (i + 1) / 6 +
      Math.sin(time * .0004 + i) * 65;
    const grad = ctx.createRadialGradient(
      x, 0, 0, x, H * .45, H * .6
    );
    grad.addColorStop(0, "rgba(128,245,255,0.065)");
    grad.addColorStop(1, "rgba(128,245,255,0)");
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, W, H);
  }
}

function drawFish(f, time) {
  ctx.save();
  ctx.translate(f.x, f.y);

  // 魚の進行方向を向く
  ctx.rotate(Math.atan2(f.vy, f.vx));

  const s = f.size;
  const wiggle = Math.sin(time * .012 + f.phase) * s * .17;

  // 尾びれ
  ctx.fillStyle = f.color;
  ctx.beginPath();
  ctx.moveTo(-s * .45, 0);
  ctx.lineTo(-s * .9, -s * .4 + wiggle);
  ctx.lineTo(-s * .85, s * .4 + wiggle);
  ctx.closePath();
  ctx.fill();

  // 胴体
  ctx.beginPath();
  ctx.ellipse(0, 0, s * .62, s * .36, 0, 0, Math.PI * 2);
  ctx.fill();

  // 背びれ
  ctx.beginPath();
  ctx.moveTo(-s * .12, -s * .27);
  ctx.lineTo(s * .08, -s * .5);
  ctx.lineTo(s * .3, -s * .25);
  ctx.fill();

  // 目
  ctx.fillStyle = "#07334d";
  ctx.beginPath();
  ctx.arc(s * .36, -s * .06, Math.max(1.5, s * .055), 0, Math.PI * 2);
  ctx.fill();

  ctx.restore();
}

function updateFish(time) {
  const elapsed = time - startTime;

  fish = fish.filter(f => {
    if (f.delay > elapsed) return true;

    let dx = target.x - f.x;
    let dy = target.y - f.y;
    let dist = Math.hypot(dx, dy) || 1;

    if (f.mode === "rush" || f.mode === "feed") {
      const arriveRadius = f.mode === "feed" ? 48 : 25;

      if (dist > arriveRadius) {
        const force = f.mode === "rush" ? .12 : .055;
        f.vx += dx / dist * force * f.speed;
        f.vy += dy / dist * force * f.speed;
      } else {
        // 集まった魚がその場を泳ぐ
        const a = time * .002 + f.phase;
        f.vx += (-Math.sin(a) * .04);
        f.vy += (Math.cos(a) * .04);
      }
    }

    if (f.mode === "vortex") {
      // ターゲットを中心に渦を描く
      const a = Math.atan2(dy, dx);
      const desiredX = target.x + Math.cos(
        a + .045
      ) * Math.max(35, f.orbit);
      const desiredY = target.y + Math.sin(
        a + .045
      ) * Math.max(35, f.orbit);

      f.vx += (desiredX - f.x) * .003;
      f.vy += (desiredY - f.y) * .003;
      f.orbit += Math.sin(time * .001 + f.phase) * .12;
    }

    // 速度制限
    const maxSpeed = f.mode === "rush" ? 3.8 :
                     f.mode === "vortex" ? 3 : 2.1;
    const speed = Math.hypot(f.vx, f.vy);
    if (speed > maxSpeed) {
      f.vx = f.vx / speed * maxSpeed;
      f.vy = f.vy / speed * maxSpeed;
    }

    // 少し漂う自然な動き
    f.vx += Math.sin(time * .001 + f.phase) * .008;
    f.vy += Math.cos(time * .0013 + f.phase) * .008;
    f.x += f.vx;
    f.y += f.vy;

    drawFish(f, time);

    // 画面外へ逃げた魚は除去
    return f.x > -100 && f.x < W + 100 &&
           f.y > -100 && f.y < H + 100;
  });

  // 長時間放置したら群れを少しずつ消す
  if (mode !== "idle" && elapsed > 13000) {
    fish = fish.filter(() => Math.random() > .025);
    if (fish.length === 0) {
      mode = "idle";
      status.textContent = "海の中は静かです……";
    }
  }
}

function updateBubbles() {
  bubbles = bubbles.filter(b => {
    b.y -= b.vy;
    b.x += Math.sin(b.y * .025) * .35;
    if (b.life !== Infinity) b.life--;

    ctx.beginPath();
    ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2);
    ctx.strokeStyle = "rgba(192,247,255,.48)";
    ctx.lineWidth = 1;
    ctx.stroke();

    return b.y > -20 && b.life > 0;
  });
}

function updateParticles() {
  particles = particles.filter(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.vx *= .98;
    p.vy *= .98;
    p.life--;

    ctx.globalAlpha = Math.max(0, p.life / p.maxLife);
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fill();
    ctx.globalAlpha = 1;

    return p.life > 0;
  });
}

function animate(time) {
  ctx.clearRect(0, 0, W, H);
  drawBackground(time);
  updateBubbles();
  updateParticles();

  // しばらく餌やり中は泡を追加
  if (mode === "feed" && time - lastSpawn > 350) {
    lastSpawn = time;
    bubbles.push({
      x: target.x + rand(-35, 35),
      y: target.y + rand(-10, 20),
      r: rand(3, 7),
      vy: rand(.7, 1.7),
      life: 130
    });
  }

  updateFish(time);
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
</script>
</body>
</html>
