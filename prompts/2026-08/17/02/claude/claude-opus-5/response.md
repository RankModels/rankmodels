I found the original Flappy Bird sprite rips (`samuelcust/flappy-bird-assets` for the atlas sprites + audio, and `nebez/floppybird` for the score panel, medals and small font), both served over jsDelivr with CORS enabled. Here's the complete game:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<meta name="theme-color" content="#4ec0ca">
<title>Flappy Bird</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{height:100%;overflow:hidden;background:#4ec0ca;
    font-family:"Courier New",monospace;
    -webkit-user-select:none;user-select:none;touch-action:none}
  body{display:flex;align-items:center;justify-content:center;
    transition:background-color .6s ease}
  body::after{content:"";position:fixed;inset:0;pointer-events:none;
    background:radial-gradient(ellipse at center,rgba(0,0,0,0) 45%,rgba(0,0,0,.35) 100%)}
  #stage{position:relative;line-height:0;filter:drop-shadow(0 12px 30px rgba(0,0,0,.45))}
  canvas{display:block;image-rendering:pixelated;image-rendering:crisp-edges;
    -ms-interpolation-mode:nearest-neighbor;border-radius:2px}
  #credit{position:fixed;bottom:6px;left:0;right:0;text-align:center;
    font-size:10px;letter-spacing:.5px;color:rgba(255,255,255,.75);
    text-shadow:0 1px 2px rgba(0,0,0,.5);z-index:5;padding:0 8px}
  #credit a{color:inherit}
  #warn{position:fixed;top:8px;left:0;right:0;text-align:center;font-size:11px;
    color:#fff;text-shadow:0 1px 2px rgba(0,0,0,.6);display:none;z-index:6}
  #loader{position:fixed;inset:0;background:#4ec0ca;display:flex;flex-direction:column;
    align-items:center;justify-content:center;gap:14px;z-index:10;
    transition:opacity .45s ease}
  #loader.hide{opacity:0;pointer-events:none}
  #loader h1{font-size:15px;letter-spacing:4px;color:#fff;font-weight:bold;
    text-shadow:2px 2px 0 #543847}
  #bar{width:180px;height:10px;border:2px solid #543847;background:rgba(255,255,255,.35);
    border-radius:2px;overflow:hidden}
  #fill{height:100%;width:0%;background:#f8e14c;transition:width .2s ease}
</style>
</head>
<body>

<div id="stage"><canvas id="game" width="288" height="512"></canvas></div>
<div id="warn"></div>
<p id="credit">Original game, concept &amp; art by Dong Nguyen (.Gears) &middot; tap / click / space to flap</p>

<div id="loader">
  <h1>LOADING</h1>
  <div id="bar"><div id="fill"></div></div>
</div>

<script>
(function () {
"use strict";

/* ------------------------------------------------------------------
   ASSETS — original Flappy Bird sprite rips served from jsDelivr
------------------------------------------------------------------ */
const A = "https://cdn.jsdelivr.net/gh/samuelcust/flappy-bird-assets@master/";
const B = "https://cdn.jsdelivr.net/gh/nebez/floppybird@gh-pages/assets/";

const IMG_SRC = {
  bgDay:   A + "sprites/background-day.png",
  bgNight: A + "sprites/background-night.png",
  base:    A + "sprites/base.png",
  pipeGreen: A + "sprites/pipe-green.png",
  pipeRed:   A + "sprites/pipe-red.png",
  message:   A + "sprites/message.png",
  scoreboard: B + "scoreboard.png",
  replay:     B + "replay.png",
  medal0: B + "medal_bronze.png",
  medal1: B + "medal_silver.png",
  medal2: B + "medal_gold.png",
  medal3: B + "medal_platinum.png"
};
["yellow", "red", "blue"].forEach(c => {
  ["upflap", "midflap", "downflap"].forEach(f => {
    IMG_SRC[c + "-" + f] = A + "sprites/" + c + "bird-" + f + ".png";
  });
});
for (let i = 0; i < 10; i++) {
  IMG_SRC["big" + i]   = A + "sprites/" + i + ".png";
  IMG_SRC["small" + i] = B + "font_small_" + i + ".png";
}
const SND_SRC = {
  wing:   A + "audio/wing.wav",
  point:  A + "audio/point.wav",
  hit:    A + "audio/hit.wav",
  die:    A + "audio/die.wav",
  swoosh: A + "audio/swoosh.wav"
};

/* fallback sizes/colours so the game stays playable if a file 404s */
const FALLBACK = {
  bgDay: [288, 512, "#4ec0ca"], bgNight: [288, 512, "#008793"],
  base: [336, 112, "#ded895"], pipeGreen: [52, 320, "#74bf2e"],
  pipeRed: [52, 320, "#e04a3f"], message: [184, 267, "rgba(0,0,0,0)"],
  scoreboard: [236, 280, "rgba(0,0,0,.55)"], replay: [115, 70, "#f8b500"],
  _bird: [34, 24, "#f8e14c"], _big: [24, 36, "#ffffff"], _small: [12, 14, "#ffffff"]
};
function fallbackFor(key) {
  let d = FALLBACK[key];
  if (!d) {
    if (key.indexOf("bird") > -1 || key.indexOf("flap") > -1) d = FALLBACK._bird;
    else if (key.indexOf("big") === 0) d = FALLBACK._big;
    else if (key.indexOf("small") === 0) d = FALLBACK._small;
    else d = [44, 44, "rgba(255,255,255,.4)"];
  }
  const c = document.createElement("canvas");
  c.width = d[0]; c.height = d[1];
  const g = c.getContext("2d");
  g.fillStyle = d[2]; g.fillRect(0, 0, d[0], d[1]);
  return c;
}

const IMG = {};
const SND = {};
let assetErrors = 0;

/* ------------------------------------------------------------------
   LOADING
------------------------------------------------------------------ */
const keys = Object.keys(IMG_SRC);
const total = keys.length + Object.keys(SND_SRC).length;
let done = 0;
const fill = document.getElementById("fill");
const step = () => { done++; fill.style.width = Math.round(done / total * 100) + "%"; };

function loadImage(key, url) {
  return new Promise(res => {
    const im = new Image();
    im.onload  = () => { IMG[key] = im; step(); res(); };
    im.onerror = () => { IMG[key] = fallbackFor(key); assetErrors++; step(); res(); };
    im.src = url;
  });
}

/* Web Audio for low-latency, overlapping SFX */
let actx = null;
function loadSound(key, url) {
  return fetch(url).then(r => r.arrayBuffer())
    .then(buf => new Promise((ok, no) => actx.decodeAudioData(buf, ok, no)))
    .then(b => { SND[key] = b; step(); })
    .catch(() => { step(); });
}
function play(key, vol) {
  if (!actx || !SND[key] || actx.state !== "running") return;
  const s = actx.createBufferSource(), g = actx.createGain();
  s.buffer = SND[key];
  g.gain.value = vol === undefined ? 0.55 : vol;
  s.connect(g); g.connect(actx.destination); s.start(0);
}
function unlockAudio() {
  if (actx && actx.state === "suspended") actx.resume();
}

try { actx = new (window.AudioContext || window.webkitAudioContext)(); } catch (e) { actx = null; }

Promise.all(
  keys.map(k => loadImage(k, IMG_SRC[k]))
  .concat(actx ? Object.keys(SND_SRC).map(k => loadSound(k, SND_SRC[k])) : [])
).then(boot);

/* ------------------------------------------------------------------
   CONSTANTS — tuned to the original 288x512 / 30fps physics
------------------------------------------------------------------ */
const W = 288, H = 512;
const BASE_Y  = 400;      // top of the ground
const SPEED   = 120;      // px/s world scroll  (4 px @30fps)
const GRAVITY = 900;      // px/s^2             (1 px/f^2 @30fps)
const FLAP    = -270;     // px/s               (-9 px/f @30fps)
const MAX_FALL = 300;     // px/s               (10 px/f @30fps)
const PIPE_W = 52, PIPE_H = 320, GAP = 100, SPACING = 144;
const BW = 34, BH = 24;
const BIRD_X = 57, BIRD_Y0 = 244;
const WING = [ "-upflap", "-midflap", "-downflap", "-midflap" ];
const MEDAL_AT = [10, 20, 30, 40];

/* ------------------------------------------------------------------
   STATE
------------------------------------------------------------------ */
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d", { alpha: false });

let state = "splash";           // splash | playing | dying | over
let paused = false;
let T = 0;                      // global clock
let groundX = 0, wingT = 0;
let bird = { y: BIRD_Y0, vy: 0, rot: 0 };
let pipes = [];
let score = 0, best = 0, isNewBest = false;
let flash = 0, shake = 0;
let theme = { bg: "bgDay", pipe: "pipeGreen", bird: "yellow", sky: "#4ec0ca" };
let over = { t: 0, count: 0, medalT: -1, uiT: -1, canRestart: false };

try { best = parseInt(localStorage.getItem("flappy_best") || "0", 10) || 0; } catch (e) { best = 0; }

function randomTheme() {
  const night = Math.random() < 0.35;
  theme.bg   = night ? "bgNight" : "bgDay";
  theme.pipe = Math.random() < 0.3 ? "pipeRed" : "pipeGreen";
  theme.bird = ["yellow", "red", "blue"][(Math.random() * 3) | 0];
  theme.sky  = night ? "#008793" : "#4ec0ca";
  document.body.style.backgroundColor = theme.sky;
  const m = document.querySelector('meta[name="theme-color"]');
  if (m) m.setAttribute("content", theme.sky);
}

function reset() {
  randomTheme();
  state = "splash"; paused = false;
  bird.y = BIRD_Y0; bird.vy = 0; bird.rot = 0;
  pipes = []; score = 0; isNewBest = false;
  flash = 0; shake = 0;
  over = { t: 0, count: 0, medalT: -1, uiT: -1, canRestart: false };
}

function spawnPipe(x) {
  pipes.push({ x: x, gapY: Math.round(81 + Math.random() * 143), scored: false });
}

function startGame() {
  state = "playing";
  pipes = [];
  spawnPipe(W + 70);
  bird.vy = FLAP; bird.rot = -25;
  play("wing", 0.5);
}

function flap() {
  bird.vy = FLAP;
  bird.rot = -25;
  play("wing", 0.5);
}

function die(byGround) {
  play("hit", 0.6);
  flash = 0.4; shake = 0.32;
  if (score > best) { best = score; isNewBest = true;
    try { localStorage.setItem("flappy_best", String(best)); } catch (e) {} }
  if (byGround) { bird.vy = 0; toGameOver(); }
  else { state = "dying"; setTimeout(() => play("die", 0.5), 250); }
}

function toGameOver() {
  state = "over";
  over = { t: 0, count: 0, medalT: -1, uiT: -1, canRestart: false };
  play("swoosh", 0.4);
}

/* ------------------------------------------------------------------
   UPDATE
------------------------------------------------------------------ */
function update(dt) {
  T += dt;
  if (flash > 0) flash -= dt;
  if (shake > 0) shake -= dt;

  if (state === "splash") {
    groundX += SPEED * dt;
    wingT += dt;
    bird.y = BIRD_Y0 + Math.sin(T * 5.5) * 5.5;
    bird.rot = 0;

  } else if (state === "playing") {
    groundX += SPEED * dt;
    wingT += dt;

    bird.vy = Math.min(MAX_FALL, bird.vy + GRAVITY * dt);
    bird.y += bird.vy * dt;
    if (bird.y < -60) { bird.y = -60; bird.vy = Math.max(bird.vy, 0); }
    if (bird.vy > 40) bird.rot = Math.min(90, bird.rot + 480 * dt);

    for (let i = 0; i < pipes.length; i++) {
      const p = pipes[i];
      p.x -= SPEED * dt;
      if (!p.scored && p.x + PIPE_W / 2 <= BIRD_X + BW / 2) {
        p.scored = true; score++; play("point", 0.5);
      }
    }
    while (pipes.length && pipes[0].x < -PIPE_W) pipes.shift();
    const last = pipes[pipes.length - 1];
    if (!last || last.x < W + SPACING) spawnPipe(last ? last.x + SPACING : W + SPACING);

    /* collisions (slightly forgiving hitbox, like the original's hitmask) */
    const bx = BIRD_X + 3, by = bird.y + 3, bw = BW - 6, bh = BH - 6;
    if (by + bh >= BASE_Y) { bird.y = BASE_Y - BH; die(true); return; }
    for (let i = 0; i < pipes.length; i++) {
      const p = pipes[i];
      if (bx + bw > p.x && bx < p.x + PIPE_W &&
          (by < p.gapY || by + bh > p.gapY + GAP)) { die(false); return; }
    }

  } else if (state === "dying") {
    bird.vy = Math.min(MAX_FALL + 100, bird.vy + GRAVITY * dt);
    bird.y += bird.vy * dt;
    bird.rot = Math.min(90, bird.rot + 720 * dt);
    if (bird.y + BH >= BASE_Y) { bird.y = BASE_Y - BH; toGameOver(); }

  } else if (state === "over") {
    over.t += dt;
    const slid = over.t >= 0.45;
    if (slid) {
      const perDigit = score > 0 ? Math.min(0.055, 0.75 / score) : 0;
      const elapsed = over.t - 0.45;
      const target = perDigit > 0 ? Math.floor(elapsed / perDigit) : score;
      over.count = Math.min(score, target);
      if (over.count >= score) {
        if (over.medalT < 0) over.medalT = 0; else over.medalT += dt;
        if (over.uiT < 0) over.uiT = 0; else over.uiT += dt;
        if (over.uiT > 0.25) over.canRestart = true;
      }
    }
  }
}

/* ------------------------------------------------------------------
   DRAW
------------------------------------------------------------------ */
const R = Math.round;
function draw(im, x, y) { ctx.drawImage(im, R(x), R(y)); }

function digitsWidth(n, pre, gap) {
  const s = String(n); let w = 0;
  for (let i = 0; i < s.length; i++) w += IMG[pre + s[i]].width + (i ? gap : 0);
  return w;
}
function drawDigits(n, cx, y, pre, gap, align) {
  const s = String(n), w = digitsWidth(n, pre, gap);
  let x = align === "right" ? cx - w : cx - w / 2;
  for (let i = 0; i < s.length; i++) {
    const g = IMG[pre + s[i]];
    draw(g, x, y); x += g.width + gap;
  }
}

function drawBird(x, y, rot, frame) {
  const im = IMG[theme.bird + WING[frame]];
  ctx.save();
  ctx.translate(R(x + BW / 2), R(y + BH / 2));
  ctx.rotate(rot * Math.PI / 180);
  ctx.drawImage(im, -BW / 2, -BH / 2);
  ctx.restore();
}

function drawPipes() {
  const pim = IMG[theme.pipe];
  for (let i = 0; i < pipes.length; i++) {
    const p = pipes[i], x = R(p.x);
    ctx.save();                       /* upper pipe: vertically flipped */
    ctx.translate(x, R(p.gapY));
    ctx.scale(1, -1);
    ctx.drawImage(pim, 0, 0);
    ctx.restore();
    draw(pim, x, p.gapY + GAP);       /* lower pipe */
  }
}

function drawGround() {
  const g = IMG.base, gw = g.width;
  let x = -(groundX % gw);
  while (x < W) { draw(g, x, BASE_Y); x += gw; }
}

/* tiny 3x5 pixel font for the "NEW" badge (the sprite rip has no such asset) */
const GLYPH = {
  N: ["101", "111", "111", "111", "101"],
  E: ["111", "100", "110", "100", "111"],
  W: ["101", "101", "111", "111", "101"]
};
function drawNewBadge(x, y) {
  ctx.fillStyle = "#543847"; ctx.fillRect(x, y, 16, 9);
  ctx.fillStyle = "#eb6d24"; ctx.fillRect(x + 1, y + 1, 14, 7);
  ctx.fillStyle = "#ffffff";
  let ox = x + 2;
  "NEW".split("").forEach(ch => {
    const gl = GLYPH[ch];
    for (let r = 0; r < 5; r++)
      for (let c = 0; c < 3; c++)
        if (gl[r][c] === "1") ctx.fillRect(ox + c, y + 2 + r, 1, 1);
    ox += 4;
  });
}

function easeOutCubic(t) { return 1 - Math.pow(1 - t, 3); }

function drawScoreboard() {
  const sb = IMG.scoreboard;
  const targetY = R((BASE_Y - sb.height) / 2);
  const p = Math.min(1, over.t / 0.45);
  const y = targetY + (H - targetY) * (1 - easeOutCubic(p));
  const x = R((W - sb.width) / 2);
  draw(sb, x, y);

  /* score + best (offsets taken from the original panel layout) */
  drawDigits(over.count, x + 211, y + 105, "small", 2, "right");
  drawDigits(best,       x + 211, y + 147, "small", 2, "right");

  if (isNewBest && over.count >= score) drawNewBadge(x + 193, y + 136);

  /* medal pops in with a little overshoot */
  let mi = -1;
  for (let i = MEDAL_AT.length - 1; i >= 0; i--) if (score >= MEDAL_AT[i]) { mi = i; break; }
  if (mi >= 0 && over.medalT >= 0) {
    const im = IMG["medal" + mi];
    const t = Math.min(1, over.medalT / 0.3);
    const s = 0.4 + 0.6 * easeOutCubic(t) + Math.sin(t * Math.PI) * 0.15;
    ctx.save();
    ctx.globalAlpha = t;
    ctx.translate(x + 32 + im.width / 2, y + 114 + im.height / 2);
    ctx.scale(s, s);
    ctx.drawImage(im, -im.width / 2, -im.height / 2);
    ctx.restore();
  }

  if (over.uiT >= 0) {
    const rp = IMG.replay, a = Math.min(1, over.uiT / 0.25);
    ctx.save();
    ctx.globalAlpha = a;
    draw(rp, x + (sb.width - rp.width) / 2, y + 205 + (1 - a) * 6);
    ctx.restore();
  }
}

function render() {
  ctx.save();
  if (shake > 0 && state !== "splash") {
    const m = shake * 12;
    ctx.translate(R((Math.random() - 0.5) * m), R((Math.random() - 0.5) * m));
  }

  draw(IMG[theme.bg], 0, 0);
  if (state !== "splash") drawPipes();
  drawGround();

  const frame = (state === "dying" || state === "over") ? 1 : (Math.floor(wingT / 0.1) % 4);
  drawBird(BIRD_X, bird.y, bird.rot, frame);

  if (state === "splash") {
    const m = IMG.message;
    draw(m, (W - m.width) / 2, 50);
  } else if (state === "playing" || state === "dying") {
    drawDigits(score, W / 2, 45, "big", 1);
  } else {
    drawScoreboard();
  }

  ctx.restore();

  if (flash > 0) {
    ctx.fillStyle = "rgba(255,255,255," + Math.min(1, flash / 0.4) + ")";
    ctx.fillRect(0, 0, W, H);
  }

  if (paused) {
    ctx.fillStyle = "rgba(0,0,0,.42)";
    ctx.fillRect(0, 0, W, H);
    ctx.fillStyle = "#fff";
    ctx.fillRect(W / 2 - 15, 220, 10, 34);
    ctx.fillRect(W / 2 + 5,  220, 10, 34);
  }
}

/* ------------------------------------------------------------------
   INPUT
------------------------------------------------------------------ */
function onTap() {
  unlockAudio();
  if (paused) { paused = false; return; }
  if (state === "splash") startGame();
  else if (state === "playing") flap();
  else if (state === "over") {
    if (!over.canRestart) {                    /* tap skips the count-up */
      if (over.t > 0.5) { over.count = score; over.t += 5; }
      return;
    }
    play("swoosh", 0.4);
    reset();
  }
}

canvas.addEventListener("mousedown", e => { e.preventDefault(); onTap(); });
canvas.addEventListener("touchstart", e => { e.preventDefault(); onTap(); }, { passive: false });
document.addEventListener("keydown", e => {
  if (e.code === "Space" || e.code === "ArrowUp" || e.code === "Enter" || e.key === " ") {
    e.preventDefault();
    if (!e.repeat) onTap();
  } else if (e.code === "KeyP" || e.code === "Escape") {
    if (state === "playing") paused = !paused;
  }
});
document.addEventListener("contextmenu", e => e.preventDefault());
document.addEventListener("visibilitychange", () => {
  if (document.hidden && state === "playing") paused = true;
});
window.addEventListener("blur", () => { if (state === "playing") paused = true; });

/* ------------------------------------------------------------------
   RESPONSIVE CANVAS (integer backing scale = crisp pixels)
------------------------------------------------------------------ */
function resize() {
  const vw = window.innerWidth, vh = window.innerHeight;
  const scale = Math.min(vw / W, (vh - 24) / H);
  canvas.style.width  = (W * scale) + "px";
  canvas.style.height = (H * scale) + "px";
  const dpr = Math.min(window.devicePixelRatio || 1, 3);
  const s = Math.max(1, Math.min(6, Math.round(scale * dpr)));
  canvas.width  = W * s;
  canvas.height = H * s;
  ctx.setTransform(s, 0, 0, s, 0, 0);
  ctx.imageSmoothingEnabled = false;
}
window.addEventListener("resize", resize);
window.addEventListener("orientationchange", () => setTimeout(resize, 150));

/* ------------------------------------------------------------------
   LOOP
------------------------------------------------------------------ */
const STEP = 1 / 120;
let acc = 0, last = 0;

function frame(now) {
  const t = now / 1000;
  let dt = last ? t - last : 0;
  last = t;
  if (dt > 0.25) dt = 0.25;
  if (!paused) {
    acc += dt;
    while (acc >= STEP) { update(STEP); acc -= STEP; }
  }
  render();
  requestAnimationFrame(frame);
}

function boot() {
  if (assetErrors) {
    const w = document.getElementById("warn");
    w.textContent = assetErrors + " asset(s) failed to load — using placeholders.";
    w.style.display = "block";
  }
  resize();
  reset();
  document.getElementById("loader").classList.add("hide");
  setTimeout(() => { const l = document.getElementById("loader"); if (l) l.remove(); }, 500);
  requestAnimationFrame(frame);
}
})();
</script>
</body>
</html>
```

A few notes on what's inside:

The physics are the original's numbers converted from 30 fps frame units to per-second units, so it feels exactly right: gravity 900 px/s², flap impulse −270 px/s, terminal fall 300 px/s, world scroll 120 px/s, 100 px gap and 144 px pipe spacing. Everything runs on a fixed 120 Hz accumulator so the game plays identically on 60 Hz and 144 Hz displays.

It's endless by design — pipes are spawned and recycled as they leave the screen with no limit, and difficulty stays constant like the real game. Each run randomizes day/night background, green/red pipes, and the yellow/red/blue bird, exactly as the original does.

Polish details include the three-frame wing cycle, velocity-driven rotation that snaps to −25° on flap and tilts to 90° on the dive, the white hit flash and a brief screen shake, the game-over panel sliding up with an animated score count-up, medal pop-in at 10/20/30/40 points, a "NEW" badge on a personal best stored in localStorage, and auto-pause when the tab loses focus (P or Esc to toggle manually). Sounds go through Web Audio for zero-latency overlapping playback and unlock on your first tap.

The canvas renders at the native 288×512 with an integer backing-store multiplier and nearest-neighbor scaling, so pixels stay sharp at any window size or device pixel ratio. If any sprite fails to fetch, it falls back to a coloured placeholder rather than breaking the game.
