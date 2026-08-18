Here's a complete, single-file **Flappy Bird** clone. It's endless, responsive, and playable right in the browser.

One note on assets: the original game's sprite files are copyrighted (DotGEARS), so instead of shipping those files I've drawn faithful original artwork directly in Canvas — same iconic look and palette (yellow bird, green pipes, scrolling ground, day sky), with zero external files or dependencies.

Save it as `index.html` and open it in any browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<title>Flappy Bird</title>
<style>
  html, body {
    margin: 0; padding: 0; height: 100%; overflow: hidden;
    background: linear-gradient(#4EC0CA, #8EE7E7);
  }
  body {
    display: flex; align-items: center; justify-content: center;
    font-family: monospace;
    -webkit-tap-highlight-color: transparent;
  }
  canvas {
    display: block;
    touch-action: none;
    user-select: none;
    -webkit-user-select: none;
  }
</style>
</head>
<body>
<canvas id="game"></canvas>
<script>
(function(){
"use strict";

/* ============================== CANVAS ============================== */
var canvas = document.getElementById('game');
var mainCtx = canvas.getContext('2d');

// Low-res render buffer (classic 288x512), upscaled with nearest-neighbour
// for an authentic retro pixel look.
var W = 288, H = 512;
var buf = document.createElement('canvas');
buf.width = W; buf.height = H;
var bctx = buf.getContext('2d');
bctx.lineJoin = 'round';

/* ============================ CONSTANTS ============================= */
var GROUND_H = 112;
var groundY = H - GROUND_H;            // 400
var BIRD_X   = 82;
var R        = 11;                     // forgiving collision radius

var GRAVITY   = 0.38;
var FLAP      = -6.6;
var MAXFALL   = 9.5;
var SPEED     = 2.1;                   // px per 60fps frame
var PIPE_W    = 52;
var GAP       = 120;
var SPAWN_GAP = 214;
var CAP_H     = 26;

/* ============================= PALETTE ============================== */
var SKY_TOP   = '#4EC0CA', SKY_MID = '#74D2D2', SKY_BOT = '#A4E4D8';
var TOWER_C   = '#B7DFC8',  BUSH_C   = '#9AD06B';
var DIRT      = '#E0D98D',  DIRT_DARK= '#C7BC70';
var GRASS     = '#78C83C',  GRASS_LIGHT = '#A7E45C', GRASS_DARK = '#57982F';

var PIPE = { base:'#6FBF35', light:'#9EE567', dark:'#4A941F', outline:'#2B5810' };

var BIRD      = '#F8D12F', BIRD_LIGHT = '#FFE17A';
var WING      = '#FBE89D';
var BEAK_TOP  = '#F5821F', BEAK_BOT  = '#E06A10';
var BIRD_OUT  = 'rgba(60,40,0,0.45)';

/* ============================ PIXEL FONT ============================ */
// Compact 5x7 bitmap font ('#' = filled).
var FONT = {
  'A':['.###.','#...#','#...#','#####','#...#','#...#','#...#'],
  'B':['####.','#...#','#...#','####.','#...#','#...#','####.'],
  'C':['.###.','#...#','#....','#....','#....','#...#','.###.'],
  'D':['####.','#...#','#...#','#...#','#...#','#...#','####.'],
  'E':['#####','#....','#....','####.','#....','#....','#####'],
  'F':['#####','#....','#....','####.','#....','#....','#....'],
  'G':['.###.','#...#','#....','#.###','#...#','#...#','.###.'],
  'H':['#...#','#...#','#...#','#####','#...#','#...#','#...#'],
  'I':['.###.','..#..','..#..','..#..','..#..','..#..','.###.'],
  'J':['..###','...#.','...#.','...#.','...#.','#..#.','.##..'],
  'K':['#...#','#..#.','#.#..','##...','#.#..','#..#.','#...#'],
  'L':['#....','#....','#....','#....','#....','#....','#####'],
  'M':['#...#','##.##','#.#.#','#.#.#','#...#','#...#','#...#'],
  'N':['#...#','##..#','#.#.#','#..##','#...#','#...#','#...#'],
  'O':['.###.','#...#','#...#','#...#','#...#','#...#','.###.'],
  'P':['####.','#...#','#...#','####.','#....','#....','#....'],
  'Q':['.###.','#...#','#...#','#...#','#.#.#','#..#.','.##.#'],
  'R':['####.','#...#','#...#','####.','#.#..','#..#.','#...#'],
  'S':['.####','#....','#....','.###.','....#','....#','####.'],
  'T':['#####','..#..','..#..','..#..','..#..','..#..','..#..'],
  'U':['#...#','#...#','#...#','#...#','#...#','#...#','.###.'],
  'V':['#...#','#...#','#...#','#...#','#...#','.#.#.','..#..'],
  'W':['#...#','#...#','#...#','#.#.#','#.#.#','##.##','#...#'],
  'X':['#...#','#...#','.#.#.','..#..','.#.#.','#...#','#...#'],
  'Y':['#...#','#...#','.#.#.','..#..','..#..','..#..','..#..'],
  'Z':['#####','....#','...#.','..#..','.#...','#....','#####'],
  '0':['.###.','#...#','#...#','#...#','#...#','#...#','.###.'],
  '1':['..#..','.##..','..#..','..#..','..#..','..#..','.###.'],
  '2':['.###.','#...#','....#','...#.','..#..','.#...','#####'],
  '3':['.###.','#...#','....#','..##.','....#','#...#','.###.'],
  '4':['...#.','..##.','.#.#.','#..#.','#####','...#.','...#.'],
  '5':['#####','#....','####.','....#','....#','#...#','.###.'],
  '6':['.###.','#....','####.','#...#','#...#','#...#','.###.'],
  '7':['#####','....#','...#.','..#..','.#...','.#...','.#...'],
  '8':['.###.','#...#','#...#','.###.','#...#','#...#','.###.'],
  '9':['.###.','#...#','#...#','.####','....#','....#','.###.'],
  '.':['.....','.....','.....','.....','.....','.....','..#..'],
  '!':['..#..','..#..','..#..','..#..','..#..','.....','..#..'],
  '-':['.....','.....','.....','#####','.....','.....','.....'],
  ':':['.....','.....','..#..','.....','.....','..#..','.....'],
  '/':['....#','....#','...#.','..#..','.#...','#....','#....'],
  ' ':['.....','.....','.....','.....','.....','.....','.....']
};

/* ============================ AUDIO ================================ */
var audio = null;
var muted = false;

function initAudio(){
  if(!audio){
    var AC = window.AudioContext || window.webkitAudioContext;
    if(!AC) return;
    audio = new AC();
  }
  if(audio && audio.state === 'suspended') audio.resume();
}

function tone(f0, f1, dur, type, vol, delay){
  if(!audio || muted) return;
  delay = delay || 0;
  var t = audio.currentTime + delay;
  var o = audio.createOscillator();
  var g = audio.createGain();
  o.type = type;
  o.frequency.setValueAtTime(Math.max(30, f0), t);
  o.frequency.exponentialRampToValueAtTime(Math.max(30, f1), t + dur);
  g.gain.setValueAtTime(vol, t);
  g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  o.connect(g); g.connect(audio.destination);
  o.start(t); o.stop(t + dur + 0.03);
}

function noise(dur, vol, cutoff, delay){
  if(!audio || muted) return;
  delay = delay || 0;
  var t = audio.currentTime + delay;
  var n = Math.floor(audio.sampleRate * dur);
  var buffer = audio.createBuffer(1, n, audio.sampleRate);
  var d = buffer.getChannelData(0);
  for(var i = 0; i < n; i++) d[i] = Math.random() * 2 - 1;
  var src = audio.createBufferSource();
  src.buffer = buffer;
  var f = audio.createBiquadFilter();
  f.type = 'lowpass'; f.frequency.value = cutoff;
  var g = audio.createGain();
  g.gain.setValueAtTime(vol, t);
  g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  src.connect(f); f.connect(g); g.connect(audio.destination);
  src.start(t);
}

function playFlap(){ noise(0.07, 0.16, 2600); tone(520, 300, 0.06, 'triangle', 0.06); }
function playScore(){ tone(987, 1046, 0.08, 'square', 0.05); tone(1318, 1568, 0.10, 'square', 0.05, 0.08); }
function playHit(){ noise(0.12, 0.35, 900); tone(200, 70, 0.16, 'sawtooth', 0.16); }
function playDie(){ tone(600, 120, 0.45, 'square', 0.10); }

/* ============================= STATE =============================== */
var state = 'ready';            // ready | playing | dying | gameover
var bird = { y: 224, vy: 0, rot: 0 };
var wingT = 0, wingIdle = 0;
var pipes = [];
var score = 0, best = 0, newBest = false;
var offset = 0;                 // ground scroll distance
var tNow = 0;                   // wall-clock seconds for animations
var flash = 0;
var gameoverDelay = 0;

/* ========================== PERSISTENCE ============================ */
function loadBest(){ try { return parseInt(localStorage.getItem('flappy_best') || '0', 10) || 0; } catch(e){ return 0; } }
function saveBest(v){ try { localStorage.setItem('flappy_best', String(v)); } catch(e){} }
function loadMute(){ try { return localStorage.getItem('flappy_mute') === '1'; } catch(e){ return false; } }
function saveMute(){ try { localStorage.setItem('flappy_mute', muted ? '1' : '0'); } catch(e){} }

best = loadBest();
muted = loadMute();

/* ========================== BACKGROUND ASSETS ====================== */
var clouds = [], towers = [], bushes = [];
(function genAssets(){
  for(var i = 0; i < 5; i++){
    clouds.push({ x: 20 + i*58 + Math.random()*20, y: 40 + Math.random()*120, s: 0.7 + Math.random()*0.7 });
  }
  var tx = 0;
  while(tx < W + 120){
    var w = 34 + Math.random()*34, h = 44 + Math.random()*52;
    towers.push({ x: tx, w: w, h: h });
    tx += w * (0.5 + Math.random()*0.7) + 6;
  }
  var bx = 0;
  while(bx < W + 80){
    var bw = 26 + Math.random()*24;
    bushes.push({ x: bx, w: bw });
    bx += bw * (0.6 + Math.random()*0.7) + 4;
  }
})();

/* ============================ HELPERS ============================== */
function wrap(v, lo, hi){
  var span = hi - lo;
  return lo + (((v - lo) % span) + span) % span;
}
function clamp(v, a, b){ return v < a ? a : (v > b ? b : v); }

function rrect(x, y, w, h, r){
  bctx.beginPath();
  bctx.moveTo(x + r, y);
  bctx.arcTo(x + w, y, x + w, y + h, r);
  bctx.arcTo(x + w, y + h, x, y + h, r);
  bctx.arcTo(x, y + h, x, y, r);
  bctx.arcTo(x, y, x + w, y, r);
  bctx.closePath();
}

function starPath(cx, cy, outer, inner, points, rot){
  rot = rot || -Math.PI/2;
  bctx.beginPath();
  for(var i = 0; i < points*2; i++){
    var r = (i % 2 === 0) ? outer : inner;
    var a = rot + i * Math.PI / points;
    var px = cx + Math.cos(a)*r, py = cy + Math.sin(a)*r;
    if(i === 0) bctx.moveTo(px, py); else bctx.lineTo(px, py);
  }
  bctx.closePath();
}

/* Draw all '#' pixels of a string. expand adds outline thickness. */
function drawGlyphs(str, sx, y, cell, color, spacing, expand){
  bctx.fillStyle = color;
  var cx = sx;
  for(var i = 0; i < str.length; i++){
    var g = FONT[str.charAt(i).toUpperCase()] || FONT[' '];
    for(var r = 0; r < 7; r++){
      var row = g[r];
      for(var c = 0; c < 5; c++){
        if(row.charAt(c) === '#'){
          bctx.fillRect(cx + c*cell - expand, y + r*cell - expand, cell + expand*2, cell + expand*2);
        }
      }
    }
    cx += 5*cell + spacing;
  }
}

function drawPixelText(str, x, y, opts){
  opts = opts || {};
  var cell = opts.cell || 6;
  var spacing = (opts.spacing != null) ? opts.spacing : Math.max(1, Math.round(cell*0.5));
  var align = opts.align || 'center';
  var w = str.length * 5*cell + (str.length - 1) * spacing;
  var sx = align === 'center' ? Math.round(x - w/2) : (align === 'right' ? Math.round(x - w) : Math.round(x));
  var o = (opts.outline != null) ? opts.outline : Math.max(1, Math.round(cell*0.3));

  if(opts.shadow){
    drawGlyphs(str, sx + (opts.shadowX || 2), y + (opts.shadowY || 2), cell,
               opts.shadowColor || 'rgba(0,0,0,0.35)', spacing, 0);
  }
  drawGlyphs(str, sx, y, cell, opts.outlineColor || '#2b2b2b', spacing, o);
  drawGlyphs(str, sx, y, cell, opts.fill || '#ffffff', spacing, 0);
}

function circleRect(cx, cy, r, rx, ry, rw, rh){
  var nx = clamp(cx, rx, rx + rw);
  var ny = clamp(cy, ry, ry + rh);
  var dx = cx - nx, dy = cy - ny;
  return dx*dx + dy*dy < r*r;
}

/* ========================== DRAWING ================================ */
function drawSky(){
  var g = bctx.createLinearGradient(0, 0, 0, H);
  g.addColorStop(0, SKY_TOP);
  g.addColorStop(0.55, SKY_MID);
  g.addColorStop(1, SKY_BOT);
  bctx.fillStyle = g;
  bctx.fillRect(0, 0, W, H);
}

function drawClouds(){
  bctx.fillStyle = '#FFFFFF';
  for(var i = 0; i < clouds.length; i++){
    var c = clouds[i];
    var x = wrap(c.x - offset*0.25 + 40, 0, W + 80) - 40;
    var s = c.s;
    bctx.beginPath();
    bctx.arc(x, c.y, 14*s, 0, 7);
    bctx.arc(x + 14*s, c.y + 3*s, 11*s, 0, 7);
    bctx.arc(x - 14*s, c.y + 4*s, 10*s, 0, 7);
    bctx.arc(x + 6*s, c.y - 8*s, 12*s, 0, 7);
    bctx.fill();
  }
}

function drawSkyline(){
  // distant towers (arch shapes)
  bctx.fillStyle = TOWER_C;
  for(var i = 0; i < towers.length; i++){
    var t = towers[i];
    var x = wrap(t.x - offset*0.45 + 70, 0, W + 140) - 70;
    var y = groundY - t.h;
    bctx.beginPath();
    bctx.moveTo(x, groundY);
    bctx.lineTo(x, y + t.w/2);
    bctx.arc(x + t.w/2, y + t.w/2, t.w/2, Math.PI, 0);
    bctx.lineTo(x + t.w, groundY);
    bctx.closePath();
    bctx.fill();
  }
  // nearer bushes
  bctx.fillStyle = BUSH_C;
  for(var j = 0; j < bushes.length; j++){
    var b = bushes[j];
    var bx = wrap(b.x - offset*0.75 + 60, 0, W + 120) - 60;
    bctx.beginPath();
    bctx.moveTo(bx, groundY);
    bctx.arc(bx + b.w/2, groundY, b.w/2, Math.PI, 0);
    bctx.closePath();
    bctx.fill();
  }
}

function drawGround(){
  // dirt
  bctx.fillStyle = DIRT;
  bctx.fillRect(0, groundY, W, GROUND_H);

  // diagonal stripes
  bctx.save();
  bctx.beginPath();
  bctx.rect(0, groundY, W, GROUND_H);
  bctx.clip();
  bctx.fillStyle = DIRT_DARK;
  var period = 24, stripeW = 13, slant = -24;
  var x = -period + (offset % period);
  for(; x < W + 40; x += period){
    bctx.beginPath();
    bctx.moveTo(x, groundY);
    bctx.lineTo(x + stripeW, groundY);
    bctx.lineTo(x + stripeW + slant, H);
    bctx.lineTo(x + slant, H);
    bctx.closePath();
    bctx.fill();
  }
  bctx.restore();

  // grass strip
  bctx.fillStyle = GRASS;
  bctx.fillRect(0, groundY, W, 16);
  bctx.fillStyle = GRASS_LIGHT;
  bctx.fillRect(0, groundY, W, 5);
  bctx.fillStyle = GRASS_DARK;
  bctx.fillRect(0, groundY + 16, W, 3);
}

function drawPipeBody(x, yTop, h){
  bctx.fillStyle = PIPE.base;
  bctx.fillRect(x, yTop, PIPE_W, h);
  bctx.fillStyle = PIPE.light;
  bctx.fillRect(x + 4, yTop, 6, h);
  bctx.fillStyle = PIPE.dark;
  bctx.fillRect(x + PIPE_W - 10, yTop, 10, h);
  bctx.strokeStyle = PIPE.outline;
  bctx.lineWidth = 1;
  bctx.strokeRect(x + 0.5, yTop + 0.5, PIPE_W - 1, h - 1);
}

function drawPipeCap(x, yTop){
  var over = 5;
  var cw = PIPE_W + over*2;
  bctx.fillStyle = PIPE.base;
  bctx.fillRect(x - over, yTop, cw, CAP_H);
  bctx.fillStyle = PIPE.light;
  bctx.fillRect(x + 4, yTop, 6, CAP_H);
  bctx.fillStyle = PIPE.dark;
  bctx.fillRect(x + PIPE_W - 10, yTop, 10, CAP_H);
  bctx.strokeStyle = PIPE.outline;
  bctx.lineWidth = 1;
  bctx.strokeRect(x - over + 0.5, yTop + 0.5, cw - 1, CAP_H - 1);
}

function drawPipes(){
  for(var i = 0; i < pipes.length; i++){
    var p = pipes[i];
    var gapTop = p.gy - GAP/2;
    var gapBot = p.gy + GAP/2;

    // upper pipe + cap at its bottom
    drawPipeBody(p.x, 0, gapTop);
    drawPipeCap(p.x, gapTop - CAP_H);

    // lower pipe + cap at its top (extends slightly behind grass)
    drawPipeBody(p.x, gapBot, groundY + 6 - gapBot);
    drawPipeCap(p.x, gapBot);
  }
}

function drawBird(x, y, rot, wingAng){
  bctx.save();
  bctx.translate(x, y);
  bctx.rotate(rot);

  // body
  bctx.fillStyle = BIRD;
  bctx.strokeStyle = BIRD_OUT;
  bctx.lineWidth = 1.4;
  bctx.beginPath();
  bctx.arc(0, 0, 13, 0, Math.PI*2);
  bctx.fill();
  bctx.stroke();

  // subtle lower shading
  bctx.fillStyle = 'rgba(0,0,0,0.06)';
  bctx.beginPath();
  bctx.arc(0, 2, 11, 0, Math.PI);
  bctx.fill();

  // belly highlight
  bctx.fillStyle = BIRD_LIGHT;
  bctx.beginPath();
  bctx.ellipse(1, 5, 8, 6, 0, 0, Math.PI*2);
  bctx.fill();

  // wing
  bctx.save();
  bctx.translate(2, -1);
  bctx.rotate(wingAng);
  bctx.fillStyle = WING;
  bctx.strokeStyle = BIRD_OUT;
  bctx.lineWidth = 1.2;
  bctx.beginPath();
  bctx.ellipse(-3, 3, 8, 5, -0.25, 0, Math.PI*2);
  bctx.fill();
  bctx.stroke();
  bctx.restore();

  // eye
  bctx.fillStyle = '#FFFFFF';
  bctx.beginPath();
  bctx.arc(5, -5, 4.7, 0, Math.PI*2);
  bctx.fill();
  bctx.fillStyle = '#1B1B1B';
  bctx.beginPath();
  bctx.arc(6.4, -5, 2.4, 0, Math.PI*2);
  bctx.fill();
  bctx.fillStyle = '#FFFFFF';
  bctx.beginPath();
  bctx.arc(7.2, -6, 0.9, 0, Math.PI*2);
  bctx.fill();

  // beak
  bctx.strokeStyle = 'rgba(120,50,0,0.35)';
  bctx.lineWidth = 1;
  bctx.fillStyle = BEAK_TOP;
  rrect(8, -3.5, 10, 4.6, 2.2);
  bctx.fill();
  bctx.stroke();
  bctx.fillStyle = BEAK_BOT;
  rrect(7, 2.2, 8, 4.4, 2);
  bctx.fill();
  bctx.stroke();

  bctx.restore();
}

function drawMedal(cx, cy, r, tier){
  var outer, mid, inner, star;
  if(tier === 1){ outer='#8A5A20'; mid='#D98A3D'; inner='#F0C082'; star='#B96A24'; }
  else if(tier === 2){ outer='#7A7A7A'; mid='#D8D8D8'; inner='#F2F2F2'; star='#A8A8A8'; }
  else if(tier === 3){ outer='#B07D14'; mid='#FFD74A'; inner='#FFF2B0'; star='#DFA61C'; }
  else { outer='#7F8C96'; mid='#E8EEF4'; inner='#FFFFFF'; star='#C0CDD8'; }

  bctx.fillStyle = outer;
  bctx.beginPath(); bctx.arc(cx, cy, r, 0, 7); bctx.fill();
  bctx.fillStyle = mid;
  bctx.beginPath(); bctx.arc(cx, cy, r - 5, 0, 7); bctx.fill();
  bctx.strokeStyle = inner;
  bctx.lineWidth = 2;
  bctx.beginPath(); bctx.arc(cx, cy, r - 9, 0, 7); bctx.stroke();
  bctx.fillStyle = star;
  starPath(cx, cy, r*0.52, r*0.22, 5);
  bctx.fill();
  bctx.fillStyle = 'rgba(255,255,255,0.5)';
  bctx.beginPath(); bctx.arc(cx - r*0.32, cy - r*0.35, r*0.16, 0, 7); bctx.fill();
}

function drawMuteButton(){
  var cx = W - 24, cy = 22, r = 12;
  bctx.fillStyle = 'rgba(0,0,0,0.18)';
  bctx.beginPath(); bctx.arc(cx, cy, r, 0, 7); bctx.fill();
  bctx.fillStyle = 'rgba(255,255,255,0.85)';
  bctx.beginPath(); bctx.arc(cx, cy, r - 2, 0, 7); bctx.fill();

  var ink = '#43524A';
  // speaker body
  bctx.fillStyle = ink;
  bctx.fillRect(cx - 7.5, cy - 5, 4, 10);
  bctx.beginPath();
  bctx.moveTo(cx - 5.5, cy - 2.5);
  bctx.lineTo(cx - 2.5, cy - 2.5);
  bctx.lineTo(cx, cy - 5.5);
  bctx.lineTo(cx, cy + 5.5);
  bctx.lineTo(cx - 2.5, cy + 2.5);
  bctx.lineTo(cx - 5.5, cy + 2.5);
  bctx.closePath();
  bctx.fill();

  if(!muted){
    bctx.strokeStyle = ink;
    bctx.lineWidth = 1.4;
    bctx.beginPath(); bctx.arc(cx + 2.5, cy, 5, -0.8, 0.8); bctx.stroke();
    bctx.beginPath(); bctx.arc(cx + 2.5, cy, 8.6, -0.7, 0.7); bctx.stroke();
  } else {
    bctx.strokeStyle = '#C0392B';
    bctx.lineWidth = 1.8;
    bctx.beginPath(); bctx.moveTo(cx + 1, cy - 5.5); bctx.lineTo(cx + 11, cy + 5.5); bctx.stroke();
    bctx.beginPath(); bctx.moveTo(cx + 11, cy - 5.5); bctx.lineTo(cx + 1, cy + 5.5); bctx.stroke();
  }
}

function drawReplayButton(cx, cy){
  var r = 22 + Math.sin(tNow*3) * 1.5;
  // drop shadow
  bctx.fillStyle = 'rgba(0,0,0,0.15)';
  bctx.beginPath(); bctx.arc(cx, cy + 3, r, 0, 7); bctx.fill();
  // base
  var g = bctx.createLinearGradient(cx, cy - r, cx, cy + r);
  g.addColorStop(0, '#9EDC5C');
  g.addColorStop(0.5, '#6FB83A');
  g.addColorStop(1, '#4F9226');
  bctx.fillStyle = g;
  bctx.beginPath(); bctx.arc(cx, cy, r, 0, 7); bctx.fill();
  bctx.strokeStyle = '#3C6E1C';
  bctx.lineWidth = 2;
  bctx.stroke();
  // play triangle
  bctx.fillStyle = '#FFFFFF';
  bctx.beginPath();
  bctx.moveTo(cx - 4, cy - 9);
  bctx.lineTo(cx + 9, cy);
  bctx.lineTo(cx - 4, cy + 9);
  bctx.closePath();
  bctx.fill();
}

/* ============================== UI ================================= */
function drawScoreHUD(){
  drawPixelText(String(score), W/2, 38, {
    cell: 13, spacing: 3, fill: '#FFFFFF', outlineColor: '#2B2B2B', outline: 4,
    shadow: true, shadowX: 3, shadowY: 3, shadowColor: 'rgba(0,0,0,0.3)', align: 'center'
  });
}

function drawReadyUI(){
  drawPixelText('GET READY!', W/2, 100, {
    cell: 5, spacing: 1, fill: '#FFFFFF', outlineColor: '#332E1A', outline: 2,
    shadow: true, shadowX: 2, shadowY: 3, shadowColor: 'rgba(0,0,0,0.3)', align: 'center'
  });
  var a = 0.45 + 0.55 * Math.abs(Math.sin(tNow*3.5));
  bctx.save();
  bctx.globalAlpha = a;
  drawPixelText('TAP OR SPACE', W/2, 330, {
    cell: 3, spacing: 1, fill: '#FFFFFF', outlineColor: '#2B2B2B', outline: 1, align: 'center'
  });
  bctx.restore();
}

function drawGameOverUI(){
  drawPixelText('GAME OVER', W/2, 100, {
    cell: 6, spacing: 1, fill: '#FFFFFF', outlineColor: '#2B2B2B', outline: 2,
    shadow: true, shadowX: 2, shadowY: 3, shadowColor: 'rgba(0,0,0,0.3)', align: 'center'
  });

  // panel
  var px = 24, py = 156, pw = 240, ph = 190;
  bctx.fillStyle = '#7A5B2B';
  rrect(px, py, pw, ph, 10);
  bctx.fill();
  bctx.fillStyle = '#E7DD91';
  rrect(px + 5, py + 5, pw - 10, ph - 10, 7);
  bctx.fill();
  bctx.fillStyle = 'rgba(255,255,255,0.35)';
  rrect(px + 5, py + 5, pw - 10, 8, 4);
  bctx.fill();

  // medal
  var tier = score >= 40 ? 4 : score >= 30 ? 3 : score >= 20 ? 2 : score >= 10 ? 1 : 0;
  if(tier > 0) drawMedal(92, 250, 30, tier);

  // right column
  var rx = 198;
  drawPixelText('SCORE', rx, 172, { cell: 3, spacing: 1, fill: '#FFFFFF', outlineColor: '#6A4A21', outline: 1, align: 'center' });
  drawPixelText(String(score), rx, 196, { cell: 6, spacing: 1, fill: '#FFFFFF', outlineColor: '#3A3A2A', outline: 2, align: 'center' });
  drawPixelText('BEST', rx, 250, { cell: 3, spacing: 1, fill: '#FFFFFF', outlineColor: '#6A4A21', outline: 1, align: 'center' });
  drawPixelText(String(best), rx, 274, { cell: 6, spacing: 1, fill: '#FFFFFF', outlineColor: '#3A3A2A', outline: 2, align: 'center' });
  if(newBest){
    drawPixelText('NEW', rx, 326, { cell: 2, spacing: 1, fill: '#EE3333', outlineColor: '#7A1010', outline: 1, align: 'center' });
  }

  drawReplayButton(W/2, 366);
}

/* ============================== RENDER ============================= */
function render(){
  bctx.clearRect(0, 0, W, H);

  drawSky();
  drawSkyline();
  drawClouds();
  drawPipes();
  drawGround();

  // bird
  var wingAng = -0.15 + Math.sin(wingIdle)*0.08 - wingT*1.05;
  if(state === 'ready') drawBird(BIRD_X, bird.y, Math.sin(wingIdle)*0.08, wingAng);
  else drawBird(BIRD_X, bird.y, bird.rot, wingAng);

  // impact flash
  if(flash > 0){
    bctx.fillStyle = 'rgba(255,255,255,' + Math.min(0.9, flash).toFixed(3) + ')';
    bctx.fillRect(0, 0, W, H);
  }

  if(state === 'ready') drawReadyUI();
  else if(state === 'gameover') drawGameOverUI();
  else drawScoreHUD(); // playing & dying

  drawMuteButton();

  // blit low-res buffer -> display canvas (nearest neighbour = crisp pixels)
  mainCtx.imageSmoothingEnabled = false;
  mainCtx.clearRect(0, 0, canvas.width, canvas.height);
  mainCtx.drawImage(buf, 0, 0, W, H, 0, 0, canvas.width, canvas.height);
}

/* ============================== LOGIC ============================== */
function randGapY(){
  var minY = GAP/2 + 50;
  var maxY = groundY - 60 - GAP/2;
  return minY + Math.random() * (maxY - minY);
}

function spawnPipe(){
  var lastX = pipes.length ? pipes[pipes.length - 1].x : W + 120;
  pipes.push({ x: lastX + SPAWN_GAP, gy: randGapY(), scored: false });
}

function flap(){
  if(state !== 'playing') return;
  bird.vy = FLAP;
  wingT = 1;
  playFlap();
}

function startGame(){
  score = 0;
  newBest = false;
  pipes = [];
  bird.y = 220;
  bird.vy = 0;
  bird.rot = 0;
  state = 'playing';
  flap();
}

function die(fromGround){
  if(state !== 'playing') return;
  state = 'dying';
  playHit();
  flash = 0.8;
  bird.vy = fromGround ? bird.vy : -1.8;
}

function enterGameOver(){
  state = 'gameover';
  playDie();
  flash = 0.5;
  gameoverDelay = 0.7;
  if(score > best){
    best = score;
    saveBest(best);
    newBest = true;
  }
}

function resetToReady(){
  state = 'ready';
  score = 0;
  pipes = [];
  bird.y = 220;
  bird.vy = 0;
  bird.rot = 0;
  newBest = false;
}

function update(dt){
  tNow += dt;
  var f = dt * 60;

  wingIdle += dt * (state === 'playing' ? 9 : 3.5);
  wingT = Math.max(0, wingT - dt * 5);
  if(flash > 0) flash = Math.max(0, flash - dt * 4);

  if(state === 'ready'){
    bird.y = 224 + Math.sin(wingIdle) * 7;
    bird.rot = 0;
  }
  else if(state === 'playing'){
    bird.vy = Math.min(bird.vy + GRAVITY * f, MAXFALL);
    bird.y += bird.vy * f;
    if(bird.y < R){ bird.y = R; if(bird.vy < 0) bird.vy = 0; }

    offset += SPEED * f;

    var died = false;
    for(var i = 0; i < pipes.length; i++){
      var p = pipes[i];
      p.x -= SPEED * f;

      if(!p.scored && p.x + PIPE_W < BIRD_X - R){
        p.scored = true;
        score++;
        playScore();
      }
      if(circleRect(BIRD_X, bird.y, R, p.x, 0, PIPE_W, p.gy - GAP/2) ||
         circleRect(BIRD_X, bird.y, R, p.x, p.gy + GAP/2, PIPE_W, groundY - (p.gy + GAP/2))){
        die(false);
        died = true;
        break;
      }
    }
    pipes = pipes.filter(function(pp){ return pp.x > -PIPE_W - 20; });

    if(!died){
      while(!pipes.length || pipes[pipes.length - 1].x < W + 300) spawnPipe();
      if(bird.y + R >= groundY) die(true);
    }

    // end of playing
    if(state !== 'playing') return;
  }
  else if(state === 'dying'){
    bird.vy = Math.min(bird.vy + GRAVITY * f, MAXFALL);
    bird.y += bird.vy * f;
    if(bird.y + R >= groundY){
      bird.y = groundY - R;
      enterGameOver();
    }
  }
  else if(state === 'gameover'){
    if(gameoverDelay > 0) gameoverDelay -= dt;
  }

  // bird tilt
  var targetRot;
  if(state === 'playing' || state === 'dying' || state === 'gameover'){
    var extra = (state === 'dying' || state === 'gameover') ? 0.35 : 0;
    targetRot = clamp(bird.vy * 0.11 + extra, -0.35, 1.5);
  } else {
    targetRot = 0;
  }
  bird.rot += (targetRot - bird.rot) * Math.min(1, dt * 12);
}

/* ============================== INPUT ============================== */
function toLogical(x, y){
  var rect = canvas.getBoundingClientRect();
  return { x: (x - rect.left) / rect.width * W, y: (y - rect.top) / rect.height * H };
}
function muteTapped(x, y){
  var p = toLogical(x, y);
  return Math.hypot(p.x - (W - 24), p.y - 22) < 20;
}
function replayTapped(x, y){
  var p = toLogical(x, y);
  return Math.hypot(p.x - W/2, p.y - 366) < 40;
}

function handleTap(x, y){
  initAudio();

  if(typeof x === 'number' && typeof y === 'number'){
    if(muteTapped(x, y)){ muted = !muted; saveMute(); return; }
    if(state === 'gameover' && replayTapped(x, y)){ resetToReady(); return; }
  }

  if(state === 'ready') startGame();
  else if(state === 'playing') flap();
  else if(state === 'gameover' && gameoverDelay <= 0) resetToReady();
}

var pointerSeen = false;
canvas.addEventListener('pointerdown', function(e){
  pointerSeen = true;
  e.preventDefault();
  handleTap(e.clientX, e.clientY);
}, { passive: false });

canvas.addEventListener('touchstart', function(e){
  if(pointerSeen) return;
  e.preventDefault();
  var t = e.changedTouches[0];
  handleTap(t.clientX, t.clientY);
}, { passive: false });

window.addEventListener('keydown', function(e){
  if(e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'Enter'){
    e.preventDefault();
    handleTap();
  }
});

/* ============================= RESIZE ============================== */
function resize(){
  var scale = Math.min(window.innerWidth / W, window.innerHeight / H);
  var cw = Math.max(1, Math.floor(W * scale));
  var ch = Math.max(1, Math.floor(H * scale));
  var dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.style.width = cw + 'px';
  canvas.style.height = ch + 'px';
  canvas.width = Math.max(1, Math.round(cw * dpr));
  canvas.height = Math.max(1, Math.round(ch * dpr));
}
window.addEventListener('resize', resize);
window.addEventListener('orientationchange', function(){ setTimeout(resize, 120); });

/* ============================== LOOP =============================== */
var last = performance.now();
function loop(now){
  var dt = Math.min(0.05, (now - last) / 1000);
  last = now;
  update(dt);
  render();
  requestAnimationFrame(loop);
}

resize();
requestAnimationFrame(loop);

})();
</script>
</body>
</html>
```

### How to play
- **Tap / Click / Spacebar / ↑ / W** — flap.
- **Endless** — pipes keep spawning forever, no level cap, best score saved locally.
- Mute button is in the top-right corner.
- Medal tiers: Bronze (10+), Silver (20+), Gold (30+), Platinum (40+).

The whole game runs off a 288×512 low-res buffer that's upscaled with nearest-neighbor filtering for that classic chunky-pixel look, while the layout scales responsively to fit any screen or phone orientation.
