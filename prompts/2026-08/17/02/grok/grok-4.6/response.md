Here’s a complete, endless Flappy Bird in a single `index.html`. Official sprites are copyrighted, so this is an original pixel-perfect recreation of the look, physics, UI, and feel — no ripped assets.

Save this as `index.html` and open it in a browser:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>Flappy Bird</title>
<style>
  @font-face {
    font-family: "FB";
    src: local("Press Start 2P"), local("Courier New");
  }
  html, body {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    background: #1a1a1a;
    overflow: hidden;
    touch-action: none;
    user-select: none;
    -webkit-user-select: none;
    -webkit-tap-highlight-color: transparent;
  }
  #wrap {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  canvas {
    display: block;
    image-rendering: pixelated;
    image-rendering: crisp-edges;
    box-shadow: 0 12px 48px rgba(0,0,0,.55);
  }
</style>
</head>
<body>
<div id="wrap"><canvas id="c"></canvas></div>
<script>
(() => {
  "use strict";

  const W = 288, H = 512, GH = 112;
  const canvas = document.getElementById("c");
  const ctx = canvas.getContext("2d");
  canvas.width = W;
  canvas.height = H;

  function fit() {
    const s = Math.min(window.innerWidth / W, window.innerHeight / H);
    canvas.style.width = (W * s) + "px";
    canvas.style.height = (H * s) + "px";
  }
  window.addEventListener("resize", fit);
  fit();

  const COL = {
    skyDay: "#4EC0CA",
    skyNight: "#04749B",
    cityDay: "#E1F8CB",
    cityNight: "#0A5A6E",
    cityLine: "#543847",
    cloud: "#ECEFDF",
    outline: "#543847",
    pipe: "#73BF2E",
    pipeMid: "#9CE659",
    pipeLite: "#C4F06B",
    pipeDark: "#4B8C16",
    grass: "#DED895",
    dirt: "#DED895",
    dirt2: "#E5D9A3",
    dirt3: "#C4B56A",
    grassTop: "#5EE270",
    grassDark: "#58C04A",
    birdY: "#F8D030",
    birdY2: "#F8B820",
    birdW: "#F85000",
    birdBeak: "#F8A818",
    birdBeak2: "#F87010",
    cheek: "#F87858",
    white: "#FFFFFF",
    black: "#000000",
    panel: "#DED895",
    panelDk: "#C9B56A",
    ready: "#83D92A",
    over: "#F87220",
    medalBr: "#D4782A",
    medalSv: "#C8D0D8",
    medalGd: "#F8D030",
    medalPt: "#E8F0F8"
  };

  function px(g, x, y, w, h, c) {
    g.fillStyle = c;
    g.fillRect(x | 0, y | 0, w, h);
  }

  function outlineRect(g, x, y, w, h, fill, line) {
    px(g, x, y, w, h, fill);
    g.strokeStyle = line;
    g.lineWidth = 2;
    g.strokeRect(x + 1, y + 1, w - 2, h - 2);
  }

  /* ---------- sprites ---------- */
  function makeBird(frame) {
    const s = document.createElement("canvas");
    s.width = 34; s.height = 24;
    const g = s.getContext("2d");
    const O = COL.outline, Y = COL.birdY, Y2 = COL.birdY2, Wng = COL.birdW;
    const Bk = COL.birdBeak, Bk2 = COL.birdBeak2, Wh = COL.white, Bl = COL.black, Ch = COL.cheek;

    // body
    px(g, 8, 8, 18, 12, O);
    px(g, 10, 6, 14, 16, O);
    px(g, 6, 10, 22, 8, O);
    px(g, 9, 9, 16, 10, Y);
    px(g, 11, 7, 12, 14, Y);
    px(g, 7, 11, 20, 6, Y);
    px(g, 11, 16, 10, 3, Y2);
    // belly highlight
    px(g, 10, 14, 8, 4, "#F8E870");
    // cheek
    px(g, 14, 13, 4, 3, Ch);
    // eye
    px(g, 20, 8, 8, 8, O);
    px(g, 21, 9, 6, 6, Wh);
    px(g, 24, 10, 3, 4, Bl);
    px(g, 25, 10, 1, 1, Wh);
    // beak
    px(g, 26, 13, 8, 6, O);
    px(g, 27, 14, 6, 2, Bk);
    px(g, 27, 16, 6, 2, Bk2);
    px(g, 27, 15, 6, 1, O);

    // wing by frame: 0 down, 1 mid, 2 up
    if (frame === 0) {
      px(g, 4, 12, 12, 8, O);
      px(g, 5, 13, 10, 6, Wng);
      px(g, 6, 14, 8, 2, "#FF7830");
    } else if (frame === 1) {
      px(g, 4, 10, 12, 7, O);
      px(g, 5, 11, 10, 5, Wng);
      px(g, 6, 12, 8, 2, "#FF7830");
    } else {
      px(g, 5, 5, 11, 8, O);
      px(g, 6, 6, 9, 6, Wng);
      px(g, 7, 7, 7, 2, "#FF7830");
    }
    return s;
  }

  const birdSpr = [makeBird(0), makeBird(1), makeBird(2)];

  function makePipeCap() {
    const s = document.createElement("canvas");
    s.width = 56; s.height = 26;
    const g = s.getContext("2d");
    px(g, 0, 0, 56, 26, COL.outline);
    px(g, 2, 2, 52, 22, COL.pipe);
    px(g, 4, 3, 8, 20, COL.pipeLite);
    px(g, 12, 3, 10, 20, COL.pipeMid);
    px(g, 44, 3, 8, 20, COL.pipeDark);
    px(g, 4, 3, 48, 3, COL.pipeLite);
    px(g, 4, 20, 48, 3, COL.pipeDark);
    return s;
  }
  function makePipeBody() {
    const s = document.createElement("canvas");
    s.width = 52; s.height = 16;
    const g = s.getContext("2d");
    px(g, 0, 0, 52, 16, COL.outline);
    px(g, 2, 0, 48, 16, COL.pipe);
    px(g, 4, 0, 7, 16, COL.pipeLite);
    px(g, 11, 0, 9, 16, COL.pipeMid);
    px(g, 41, 0, 7, 16, COL.pipeDark);
    return s;
  }
  const pipeCap = makePipeCap();
  const pipeBody = makePipeBody();

  function makeGround() {
    const s = document.createElement("canvas");
    s.width = 24; s.height = GH;
    const g = s.getContext("2d");
    px(g, 0, 0, 24, GH, COL.dirt);
    px(g, 0, 0, 24, 3, COL.outline);
    px(g, 0, 3, 24, 11, COL.grassTop);
    px(g, 0, 11, 24, 4, COL.grassDark);
    px(g, 0, 14, 24, 2, COL.outline);
    for (let i = 0; i < 8; i++) {
      const x = (i * 7 + 3) % 24, y = 22 + (i * 13) % 80;
      px(g, x, y, 3, 2, i % 2 ? COL.dirt3 : "#E8DCB0");
    }
    px(g, 2, 28, 2, 2, COL.dirt3);
    px(g, 14, 44, 3, 2, "#E8DCB0");
    px(g, 8, 62, 2, 2, COL.dirt3);
    px(g, 18, 80, 3, 2, "#E8DCB0");
    return s;
  }
  const groundSpr = makeGround();

  function makeBg(night) {
    const s = document.createElement("canvas");
    s.width = W; s.height = H - GH;
    const g = s.getContext("2d");
    px(g, 0, 0, W, H - GH, night ? COL.skyNight : COL.skyDay);

    if (night) {
      const stars = [[20,30],[40,70],[70,22],[110,55],[150,18],[180,80],[210,36],[250,60],[268,20],[90,90]];
      stars.forEach(([x,y], i) => {
        px(g, x, y, i % 3 === 0 ? 2 : 1, i % 3 === 0 ? 2 : 1, "#F0F8FF");
      });
      // moon
      px(g, 228, 42, 28, 28, "#F4F0C8");
      px(g, 232, 46, 20, 20, night ? COL.skyNight : COL.skyDay);
    } else {
      // clouds
      const clouds = [[18,90,36],[120,50,28],[210,110,32]];
      clouds.forEach(([x,y,r]) => {
        g.fillStyle = COL.cloud;
        g.beginPath();
        g.ellipse(x, y, r, r * 0.45, 0, 0, Math.PI * 2);
        g.ellipse(x + r * 0.6, y + 4, r * 0.7, r * 0.38, 0, 0, Math.PI * 2);
        g.ellipse(x - r * 0.5, y + 5, r * 0.55, r * 0.32, 0, 0, Math.PI * 2);
        g.fill();
      });
    }

    const city = night ? COL.cityNight : COL.cityDay;
    const win = night ? "#D8E070" : "#8EC5C8";
    const baseY = H - GH;
    const blds = [
      [0, 48, 28], [24, 72, 22], [44, 40, 36], [78, 80, 26],
      [102, 52, 30], [130, 90, 24], [152, 44, 34], [184, 70, 28],
      [210, 38, 40], [248, 64, 26], [270, 50, 20]
    ];
    blds.forEach(([x, h, w]) => {
      px(g, x, baseY - h, w, h, city);
      for (let wy = baseY - h + 6; wy < baseY - 8; wy += 10) {
        for (let wx = x + 4; wx < x + w - 4; wx += 8) {
          if ((wx + wy) % 5 !== 0) px(g, wx, wy, 3, 4, win);
        }
      }
    });
    // bush line
    g.fillStyle = night ? "#0E6A4A" : "#5EE270";
    g.beginPath();
    for (let x = 0; x <= W; x += 16) {
      g.arc(x, baseY + 2, 12, Math.PI, 0);
    }
    g.fill();
    return s;
  }
  const bgDay = makeBg(false);
  const bgNight = makeBg(true);

  function makeDigits() {
    // classic 7-segment-ish flappy digits, 12x18 with outline
    const glyphs = {
      "0": ["01110","10001","10001","10001","10001","10001","01110"],
      "1": ["00100","01100","00100","00100","00100","00100","01110"],
      "2": ["01110","10001","00001","00110","01000","10000","11111"],
      "3": ["01110","10001","00001","00110","00001","10001","01110"],
      "4": ["00010","00110","01010","10010","11111","00010","00010"],
      "5": ["11111","10000","11110","00001","00001","10001","01110"],
      "6": ["01110","10000","11110","10001","10001","10001","01110"],
      "7": ["11111","00001","00010","00100","01000","01000","01000"],
      "8": ["01110","10001","10001","01110","10001","10001","01110"],
      "9": ["01110","10001","10001","01111","00001","00001","01110"]
    };
    const out = {};
    for (const [ch, rows] of Object.entries(glyphs)) {
      const s = document.createElement("canvas");
      s.width = 14; s.height = 20;
      const g = s.getContext("2d");
      for (let r = 0; r < 7; r++) {
        for (let c = 0; c < 5; c++) {
          if (rows[r][c] === "1") {
            px(g, c * 2 + 2, r * 2 + 3, 3, 3, COL.outline);
          }
        }
      }
      for (let r = 0; r < 7; r++) {
        for (let c = 0; c < 5; c++) {
          if (rows[r][c] === "1") px(g, c * 2 + 2, r * 2 + 3, 2, 2, COL.white);
        }
      }
      out[ch] = s;
    }
    return out;
  }
  const digits = makeDigits();

  function drawScore(g, n, cx, cy, scale) {
    const str = String(n);
    const dw = 14 * scale, dh = 20 * scale;
    let x = cx - (str.length * dw) / 2;
    for (const ch of str) {
      g.imageSmoothingEnabled = false;
      g.drawImage(digits[ch], x, cy, dw, dh);
      x += dw - 1;
    }
  }

  function makeMedal(type) {
    const s = document.createElement("canvas");
    s.width = 44; s.height = 44;
    const g = s.getContext("2d");
    const fill = type === "bronze" ? COL.medalBr
      : type === "silver" ? COL.medalSv
      : type === "gold" ? COL.medalGd
      : COL.medalPt;
    const dk = type === "bronze" ? "#8A4010"
      : type === "silver" ? "#808890"
      : type === "gold" ? "#C49010"
      : "#90A0B0";
    g.beginPath();
    g.arc(22, 22, 20, 0, Math.PI * 2);
    g.fillStyle = COL.outline;
    g.fill();
    g.beginPath();
    g.arc(22, 22, 17, 0, Math.PI * 2);
    g.fillStyle = fill;
    g.fill();
    g.beginPath();
    g.arc(16, 16, 5, 0, Math.PI * 2);
    g.fillStyle = "rgba(255,255,255,.45)";
    g.fill();
    g.beginPath();
    g.arc(22, 22, 11, 0, Math.PI * 2);
    g.strokeStyle = dk;
    g.lineWidth = 2;
    g.stroke();
    // tiny bird
    px(g, 16, 20, 12, 7, COL.outline);
    px(g, 17, 21, 10, 5, COL.birdY);
    px(g, 24, 21, 3, 3, COL.white);
    px(g, 25, 22, 1, 1, COL.black);
    return s;
  }
  const medals = {
    bronze: makeMedal("bronze"),
    silver: makeMedal("silver"),
    gold: makeMedal("gold"),
    plat: makeMedal("plat")
  };

  /* ---------- audio (synthesized, no assets) ---------- */
  let actx = null;
  function audio() {
    if (!actx) actx = new (window.AudioContext || window.webkitAudioContext)();
    if (actx.state === "suspended") actx.resume();
    return actx;
  }
  function beep(freq, dur, type, vol, slide) {
    try {
      const a = audio();
      const o = a.createOscillator();
      const gn = a.createGain();
      o.type = type || "square";
      o.frequency.setValueAtTime(freq, a.currentTime);
      if (slide) o.frequency.exponentialRampToValueAtTime(Math.max(40, slide), a.currentTime + dur);
      gn.gain.setValueAtTime(vol || 0.08, a.currentTime);
      gn.gain.exponentialRampToValueAtTime(0.001, a.currentTime + dur);
      o.connect(gn); gn.connect(a.destination);
      o.start(); o.stop(a.currentTime + dur);
    } catch (e) {}
  }
  function sfxFlap() { beep(420, 0.09, "square", 0.05, 720); }
  function sfxPoint() { beep(880, 0.08, "square", 0.07); setTimeout(() => beep(1180, 0.1, "square", 0.07), 70); }
  function sfxHit() { beep(180, 0.18, "sawtooth", 0.1, 60); }
  function sfxDie() { beep(400, 0.45, "square", 0.07, 90); }

  /* ---------- game ---------- */
  const GRAVITY = 0.32;
  const FLAP = -5.15;
  const MAX_VY = 8.6;
  const SPEED = 2.15;
  const PIPE_W = 52;
  const PIPE_GAP = 108;
  const PIPE_EVERY = 168;
  const BIRD_X = 64;
  const BW = 34, BH = 24;

  const S = { READY: 0, PLAY: 1, DEAD: 2, OVER: 3 };

  let state = S.READY;
  let birdY = 230, birdVy = 0, birdRot = 0, birdFrame = 0, frameT = 0, bobT = 0;
  let pipes = [];
  let spawnX = 0;
  let score = 0, best = 0;
  try { best = +localStorage.getItem("fb_best") || 0; } catch (e) {}
  let groundX = 0;
  let flash = 0;
  let deadT = 0;
  let night = false;
  let sparkleT = 0;
  let overSlide = 0;

  function medalFor(n) {
    if (n >= 40) return "plat";
    if (n >= 30) return "gold";
    if (n >= 20) return "silver";
    if (n >= 10) return "bronze";
    return null;
  }

  function reset(toReady) {
    state = toReady ? S.READY : S.PLAY;
    birdY = 230;
    birdVy = 0;
    birdRot = 0;
    pipes = [];
    spawnX = W + 20;
    score = 0;
    flash = 0;
    deadT = 0;
    overSlide = 0;
    night = Math.random() < 0.35;
    if (!toReady) {
      birdVy = FLAP;
      sfxFlap();
    }
  }

  function flap() {
    if (state === S.READY) {
      reset(false);
      return;
    }
    if (state === S.PLAY) {
      birdVy = FLAP;
      sfxFlap();
      return;
    }
    if (state === S.OVER && deadT > 70) {
      reset(true);
    }
  }

  function spawnPipe() {
    const min = 46;
    const max = H - GH - PIPE_GAP - 46;
    const top = min + Math.random() * (max - min);
    pipes.push({ x: W + 8, top, scored: false });
  }

  function hitbox() {
    return { x: BIRD_X + 6, y: birdY - BH / 2 + 5, w: 22, h: 14 };
  }

  function collide() {
    const b = hitbox();
    if (b.y + b.h >= H - GH) return true;
    if (b.y < -8) return true;
    for (const p of pipes) {
      const px = p.x, pw = PIPE_W;
      const gapTop = p.top, gapBot = p.top + PIPE_GAP;
      if (b.x + b.w > px && b.x < px + pw) {
        if (b.y < gapTop || b.y + b.h > gapBot) return true;
      }
      // caps are a bit wider
      if (b.x + b.w > px - 2 && b.x < px + pw + 2) {
        if (b.y < gapTop && b.y + b.h > gapTop - 26) return true;
        if (b.y < gapBot + 26 && b.y + b.h > gapBot) return true;
      }
    }
    return false;
  }

  function update() {
    groundX = (groundX - (state === S.DEAD || state === S.OVER ? 0 : SPEED)) % 24;
    bobT++;
    sparkleT++;

    if (state === S.READY) {
      birdY = 230 + Math.sin(bobT * 0.09) * 4;
      birdRot = 0;
      frameT++;
      if (frameT % 6 === 0) birdFrame = (birdFrame + 1) % 3;
      return;
    }

    if (state === S.PLAY || state === S.DEAD) {
      birdVy = Math.min(MAX_VY, birdVy + GRAVITY);
      birdY += birdVy;
      const target = birdVy < 0.4 ? -22 * Math.PI / 180 : Math.min(90, birdVy * 9) * Math.PI / 180;
      birdRot += (target - birdRot) * 0.18;

      if (state === S.PLAY) {
        frameT++;
        if (birdVy < 1.2) {
          if (frameT % 5 === 0) birdFrame = (birdFrame + 1) % 3;
        } else birdFrame = 1;

        spawnX -= SPEED;
        if (spawnX <= 0) {
          spawnPipe();
          spawnX = PIPE_EVERY;
        }
        for (const p of pipes) {
          p.x -= SPEED;
          if (!p.scored && p.x + PIPE_W < BIRD_X) {
            p.scored = true;
            score++;
            sfxPoint();
          }
        }
        pipes = pipes.filter(p => p.x > -70);

        if (collide()) {
          state = S.DEAD;
          flash = 8;
          deadT = 0;
          sfxHit();
          setTimeout(sfxDie, 180);
        }
      } else {
        birdFrame = 1;
        deadT++;
        if (birdY + BH / 2 >= H - GH) {
          birdY = H - GH - BH / 2 + 2;
          birdVy = 0;
          birdRot = 90 * Math.PI / 180;
          if (deadT > 28) {
            state = S.OVER;
            if (score > best) {
              best = score;
              try { localStorage.setItem("fb_best", String(best)); } catch (e) {}
            }
          }
        }
      }
    }

    if (state === S.OVER) {
      deadT++;
      overSlide = Math.min(1, overSlide + 0.07);
    }
    if (flash > 0) flash--;
  }

  function drawPipes(g) {
    for (const p of pipes) {
      const x = Math.round(p.x);
      const gapTop = Math.round(p.top);
      const gapBot = gapTop + PIPE_GAP;
      // top body
      for (let y = gapTop - 26; y > -16; y -= 16) {
        g.drawImage(pipeBody, x, y - 16);
      }
      g.save();
      g.translate(x + 26, gapTop - 13);
      g.scale(1, -1);
      g.drawImage(pipeCap, -28, -13);
      g.restore();
      // bottom
      for (let y = gapBot + 26; y < H - GH + 16; y += 16) {
        g.drawImage(pipeBody, x, y);
      }
      g.drawImage(pipeCap, x - 2, gapBot);
    }
  }

  function drawGround(g) {
    const gx = Math.round(groundX);
    for (let x = gx - 24; x < W + 24; x += 24) {
      g.drawImage(groundSpr, x, H - GH);
    }
  }

  function drawBird(g) {
    g.save();
    g.translate(Math.round(BIRD_X + BW / 2), Math.round(birdY));
    g.rotate(birdRot);
    g.imageSmoothingEnabled = false;
    g.drawImage(birdSpr[birdFrame], -BW / 2, -BH / 2);
    g.restore();
  }

  function roundPanel(g, x, y, w, h) {
    g.fillStyle = COL.outline;
    g.beginPath();
    g.roundRect(x, y, w, h, 8);
    g.fill();
    g.fillStyle = COL.panel;
    g.beginPath();
    g.roundRect(x + 2, y + 2, w - 4, h - 4, 6);
    g.fill();
    g.fillStyle = "rgba(255,255,255,.25)";
    g.fillRect(x + 6, y + 4, w - 12, 6);
  }

  function strokeText(g, text, x, y, size, fill, stroke, w) {
    g.font = `bold ${size}px Arial, Helvetica, sans-serif`;
    g.textAlign = "center";
    g.textBaseline = "middle";
    g.lineJoin = "round";
    g.strokeStyle = stroke;
    g.lineWidth = w;
    g.strokeText(text, x, y);
    g.fillStyle = fill;
    g.fillText(text, x, y);
  }

  function drawLogo(g, y) {
    strokeText(g, "FLAPPY", W / 2, y, 34, "#F8E030", COL.outline, 7);
    strokeText(g, "BIRD", W / 2, y + 32, 34, "#F8E030", COL.outline, 7);
    // orange drop
    g.globalCompositeOperation = "destination-over";
    strokeText(g, "FLAPPY", W / 2 + 2, y + 3, 34, "#F07018", "#F07018", 7);
    strokeText(g, "BIRD", W / 2 + 2, y + 35, 34, "#F07018", "#F07018", 7);
    g.globalCompositeOperation = "source-over";
  }

  function drawReady(g) {
    strokeText(g, "GET READY", W / 2, 130, 28, COL.ready, COL.outline, 6);
    // tap hint
    const tx = W / 2 + 36, ty = 250;
    g.strokeStyle = COL.outline;
    g.lineWidth = 3;
    g.fillStyle = COL.white;
    g.beginPath();
    g.arc(tx, ty, 16, 0, Math.PI * 2);
    g.fill(); g.stroke();
    g.beginPath();
    g.moveTo(tx - 2, ty + 14);
    g.lineTo(tx + 8, ty + 36);
    g.lineTo(tx + 18, ty + 30);
    g.lineTo(tx + 8, ty + 10);
    g.closePath();
    g.fill(); g.stroke();
    // dashed motion lines
    g.setLineDash([4, 3]);
    g.beginPath();
    g.moveTo(tx - 38, ty - 18);
    g.quadraticCurveTo(tx - 10, ty - 48, tx + 4, ty - 18);
    g.stroke();
    g.setLineDash([]);
    strokeText(g, "TAP", W / 2 - 28, 248, 16, COL.white, COL.outline, 4);
  }

  function drawOver(g) {
    const k = overSlide;
    const y0 = 70 + (1 - k) * -40;
    g.globalAlpha = k;
    strokeText(g, "GAME OVER", W / 2, y0, 30, COL.over, COL.outline, 6);

    const px0 = 36, py = 118 + (1 - k) * 30, pw = 216, ph = 126;
    roundPanel(g, px0, py, pw, ph);

    g.font = "bold 13px Arial, Helvetica, sans-serif";
    g.fillStyle = "#F07018";
    g.textAlign = "left";
    g.strokeStyle = COL.outline;
    g.lineWidth = 3;
    g.strokeText("MEDAL", px0 + 16, py + 28);
    g.fillText("MEDAL", px0 + 16, py + 28);
    g.textAlign = "right";
    g.strokeText("SCORE", px0 + pw - 16, py + 28);
    g.fillText("SCORE", px0 + pw - 16, py + 28);
    g.strokeText("BEST", px0 + pw - 16, py + 78);
    g.fillText("BEST", px0 + pw - 16, py + 78);

    drawScore(g, score, px0 + pw - 48, py + 36, 1.15);
    drawScore(g, best, px0 + pw - 48, py + 86, 1.15);

    const md = medalFor(score);
    if (md) {
      g.drawImage(medals[md], px0 + 24, py + 42);
      if (md === "plat" || md === "gold") {
        const sp = (sparkleT % 24) < 12;
        if (sp) {
          g.fillStyle = COL.white;
          const sx = px0 + 28 + (sparkleT * 1.3) % 36;
          const sy = py + 48 + (sparkleT * 0.7) % 28;
          px(g, sx, sy, 3, 1, COL.white);
          px(g, sx + 1, sy - 1, 1, 3, COL.white);
        }
      }
    } else {
      g.fillStyle = "rgba(84,56,71,.15)";
      g.beginPath();
      g.arc(px0 + 46, py + 64, 18, 0, Math.PI * 2);
      g.fill();
    }

    // OK button
    const bx = W / 2 - 40, by = 270 + (1 - k) * 20;
    g.fillStyle = COL.outline;
    g.beginPath(); g.roundRect(bx, by, 80, 30, 5); g.fill();
    g.fillStyle = "#E8D48A";
    g.beginPath(); g.roundRect(bx + 2, by + 2, 76, 26, 4); g.fill();
    g.fillStyle = "#F8F0C0";
    g.fillRect(bx + 4, by + 3, 72, 8);
    strokeText(g, "OK", W / 2, by + 16, 16, "#F8F0D8", COL.outline, 4);

    if (deadT > 90 && Math.floor(deadT / 24) % 2 === 0) {
      strokeText(g, "TAP TO RESTART", W / 2, 330, 12, COL.white, COL.outline, 4);
    }
    g.globalAlpha = 1;
  }

  function render() {
    ctx.imageSmoothingEnabled = false;
    ctx.drawImage(night ? bgNight : bgDay, 0, 0);
    drawPipes(ctx);
    drawGround(ctx);
    drawBird(ctx);

    if (state === S.READY) {
      drawLogo(ctx, 86);
      drawReady(ctx);
    }
    if (state === S.PLAY) {
      drawScore(ctx, score, W / 2, 36, 1.35);
    }
    if (state === S.DEAD) {
      drawScore(ctx, score, W / 2, 36, 1.35);
    }
    if (state === S.OVER) drawOver(ctx);

    if (flash > 0) {
      ctx.fillStyle = `rgba(255,255,255,${flash / 10})`;
      ctx.fillRect(0, 0, W, H);
    }
  }

  let last = 0, acc = 0;
  const STEP = 1000 / 60;
  function loop(t) {
    if (!last) last = t;
    acc += Math.min(48, t - last);
    last = t;
    while (acc >= STEP) { update(); acc -= STEP; }
    render();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  function onInput(e) {
    e.preventDefault();
    audio();
    flap();
  }
  canvas.addEventListener("pointerdown", onInput, { passive: false });
  window.addEventListener("keydown", e => {
    if (["Space", "ArrowUp", "Enter", "KeyW"].includes(e.code)) {
      e.preventDefault();
      audio();
      flap();
    }
  });
  document.addEventListener("touchmove", e => e.preventDefault(), { passive: false });
})();
</script>
</body>
</html>
```

**How to play**
- **Tap / click / space / ↑** to flap
- Fly through the pipes — the run never ends until you hit one
- **Medals:** 10 bronze · 20 silver · 30 gold · 40 platinum
- Best score is saved in the browser
- About 1 in 3 runs is night mode

**Controls & feel**
- Original 288×512 playfield, pixel-scaled to your screen
- Gravity, flap, pipe gap, and death (flash → fall → board) match the classic game
- Wing animation, bird tilt, scrolling ground, and synthesized flap / score / hit sounds

Official Flappy Bird sprites and audio are copyrighted by .GEARS, so those files are not bundled. Everything here is drawn and synthesized in-file so it can ship as one `index.html` and still read as the real game.
