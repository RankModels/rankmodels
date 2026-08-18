Here's a complete, self-contained Flappy Bird game. One important note before the code: the original Flappy Bird sprite files are copyrighted assets owned by Dong Nguyen, so they can't legally be redistributed inside your file. Instead, I recreated the bird pixel-by-pixel in code (yellow body, orange beak, white eye, flapping orange wing with 3 animation frames), along with the signature green gradient pipes, scrolling tan ground with diagonal stripes, the teal sky with a pale city skyline and bushes, plus synthesized flap/point/hit/die sounds — so it looks and feels like the real game without shipping the protected asset files.

**How to play:** Save as `index.html` and open in any browser. Click, tap, or press Space to flap. It's endless, responsive (scales to any screen), and saves your best score locally.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Flappy Bird</title>
<style>
  html,body{margin:0;padding:0;height:100%;background:#0c0c0c;overflow:hidden}
  body{display:flex;align-items:center;justify-content:center}
  canvas{image-rendering:pixelated;image-rendering:crisp-edges;touch-action:none;display:block}
</style>
</head>
<body>
<canvas id="game" width="288" height="512"></canvas>
<script>
"use strict";
const cv=document.getElementById('game'), ctx=cv.getContext('2d');
ctx.lineJoin='round';

/* ---------- constants (native 288x512, like the original) ---------- */
const W=288,H=512,GROUND_H=96,PLAY_H=H-GROUND_H;
const BX=88, GRAV=1400, FLAP=-395, TERM=580, SPEED=128;
const PIPE_W=52, GAP=126, SPACING=180, BASE_Y=250;

function resize(){
  const s=Math.min(innerWidth/W,innerHeight/H);
  cv.style.width=(W*s)+'px'; cv.style.height=(H*s)+'px';
}
addEventListener('resize',resize); resize();

/* ---------- pixel-art bird (hand-recreated, 17x15) ---------- */
const PAL={K:'#543847',E:'#543847',H:'#FCEE9B',Y:'#FBD000',B:'#F9EFC3',
           W:'#FFFFFF',R:'#F66055',O:'#F96A00',D:'#D94E00',G:'#F9A825'};
const BODY=[
".....KKKKKK......",
"...KKHHHHYYKK....",
"..KHHHHHYYYYKWWK.",
".KHHYYYYYYYKWWWK.",
"KYYYYYYYYYYKWWEWK",
"KYYYYYYYYYRKWWWWK",
"KYYYYYYYYYYKOOOOK",
".KYYYYYYYYYKOOOOK",
".KYYYYYYYYKKKKKKK",
".KYYYYYYYYKDDDDK.",
"..KYYYYYYYKDDDK..",
"..KBBBBBYYYKK....",
".KBBBBBBYYK......",
"..KKKKKKKK.......",
"................."];
const D='.................';
function wing(def){const a=new Array(15).fill(D);for(const k in def)a[k]=def[k];return a;}
const W_UP  =wing({4:'..KKK...........',5:'.KGGGK...........',6:'.KGGGGK..........',7:'..KKKK...........'});
const W_MID =wing({9:'..KKK...........',10:'.KGGGK...........',11:'KGGGGGK..........',12:'.KKKKKK...........'});
const W_DOWN=wing({11:'..KKK...........',12:'.KGGGK...........',13:'KGGGGGK..........',14:'.KKKKKK...........'});
function merge(base,over){return base.map((r,j)=>{const o=over[j];let s='';
  for(let i=0;i<17;i++){const c=o[i];s+=(c&&c!=='.')?c:r[i];}return s;});}
function makeSprite(rows){const c=document.createElement('canvas');c.width=17;c.height=15;
  const g=c.getContext('2d');
  rows.forEach((r,j)=>{for(let i=0;i<17;i++){const col=PAL[r[i]];
    if(col){g.fillStyle=col;g.fillRect(i,j,1,1);}}});
  return c;}
const FRAMES=[makeSprite(merge(BODY,W_UP)),makeSprite(merge(BODY,W_MID)),makeSprite(merge(BODY,W_DOWN))];

/* ---------- sound (WebAudio, no external files) ---------- */
let ac=null;
function audio(){try{
  if(!ac)ac=new (window.AudioContext||window.webkitAudioContext)();
  if(ac.state==='suspended')ac.resume();}catch(e){}}
function tone(f0,f1,dur,type,vol){if(!ac)return;
  const o=ac.createOscillator(),g=ac.createGain(),n=ac.currentTime;
  o.type=type;o.frequency.setValueAtTime(f0,n);
  o.frequency.exponentialRampToValueAtTime(Math.max(1,f1),n+dur);
  g.gain.setValueAtTime(vol,n);g.gain.exponentialRampToValueAtTime(0.0001,n+dur);
  o.connect(g).connect(ac.destination);o.start(n);o.stop(n+dur);}
function sfxFlap(){tone(520,260,0.09,'triangle',0.25);}
function sfxPoint(){tone(950,950,0.07,'sine',0.22);setTimeout(()=>tone(1400,1400,0.12,'sine',0.22),75);}
function sfxHit(){if(!ac)return;
  const b=ac.createBuffer(1,ac.sampleRate*0.2,ac.sampleRate),d=b.getChannelData(0);
  for(let i=0;i<d.length;i++)d[i]=(Math.random()*2-1)*(1-i/d.length);
  const s=ac.createBufferSource();s.buffer=b;const g=ac.createGain();g.gain.value=0.4;
  s.connect(g).connect(ac.destination);s.start();}
function sfxDie(){tone(500,80,0.6,'sawtooth',0.18);}

/* ---------- state ---------- */
let t=0,state='ready',deadT=0,landed=false,flash=0,newBest=false;
let score=0,best=parseInt(localStorage.getItem('fbBest')||'0',10)||0;
let bird={y:BASE_Y,v:0,rot:0};
let pipes=[],groundOff=0,cityOff=0,bushOff=0;

/* ---------- scenery ---------- */
const BL=[];let _bx=0;
while(_bx<W+60){const w=16+Math.random()*26;BL.push({x:_bx,w,h:18+Math.random()*60});_bx+=w+3+Math.random()*6;}
const CITYW=_bx;
function drawCity(){
  const off=cityOff%CITYW;
  for(let pass=0;pass<2;pass++){
    const base=pass*CITYW-off;
    for(const b of BL){
      const x=b.x+base; if(x>W||x+b.w<0)continue;
      ctx.fillStyle='#A9DDD7';ctx.fillRect(x,PLAY_H-b.h,b.w,b.h);
      ctx.fillStyle='#CBF0EA';
      for(let wy=PLAY_H-b.h+5;wy<PLAY_H-6;wy+=9)
        for(let wx=x+3;wx<x+b.w-4;wx+=7)ctx.fillRect(wx,wy,3,4);
    }
  }
}
function drawBushes(){
  const s=26,off=bushOff%s;
  ctx.fillStyle='#B7E196';
  for(let x=-s-off;x<W+s;x+=s){ctx.beginPath();ctx.arc(x,PLAY_H+12,16,0,7);ctx.fill();}
  ctx.fillStyle='#CDEDA9';
  for(let x=-s-off+13;x<W+s;x+=s){ctx.beginPath();ctx.arc(x,PLAY_H+8,15,0,7);ctx.fill();}
}
function drawGround(){
  const gy=PLAY_H;
  ctx.fillStyle='#A2DE62';ctx.fillRect(0,gy,W,3);
  ctx.fillStyle='#79BE45';ctx.fillRect(0,gy+3,W,3);
  ctx.fillStyle='#5E9B33';
  const o2=groundOff%24;
  for(let x=-24-o2;x<W;x+=24)ctx.fillRect(x,gy+2,10,2);
  ctx.fillStyle='#E3D999';ctx.fillRect(0,gy+6,W,H-gy-6);
  ctx.fillStyle='#D6C87E';
  const off=groundOff%48;
  for(let x=-48-off;x<W+48;x+=48){
    ctx.beginPath();ctx.moveTo(x,gy+6);ctx.lineTo(x+24,gy+6);
    ctx.lineTo(x+40,H);ctx.lineTo(x+16,H);ctx.closePath();ctx.fill();
  }
}

/* ---------- pipes ---------- */
function pipeGrad(x,w){
  const g=ctx.createLinearGradient(x,0,x+w,0);
  g.addColorStop(0,'#4E8016');g.addColorStop(.12,'#6BB329');g.addColorStop(.25,'#9DE35A');
  g.addColorStop(.33,'#D6F5A0');g.addColorStop(.42,'#8FD34C');g.addColorStop(.78,'#6BB329');
  g.addColorStop(1,'#4E8016');return g;}
function pipeSeg(x,y,h){if(h<=0)return;
  ctx.fillStyle='#543847';ctx.fillRect(x-2,y-2,PIPE_W+4,h+4);
  ctx.fillStyle=pipeGrad(x,PIPE_W);ctx.fillRect(x,y,PIPE_W,h);}
function pipeCap(x,y){
  ctx.fillStyle='#543847';ctx.fillRect(x-6,y-2,PIPE_W+12,30);
  ctx.fillStyle=pipeGrad(x-4,PIPE_W+8);ctx.fillRect(x-4,y,PIPE_W+8,26);
  ctx.fillStyle='rgba(0,0,0,0.12)';ctx.fillRect(x-4,y+22,PIPE_W+8,4);}
function drawPipe(p){
  const topH=p.gapY-GAP/2,botY=p.gapY+GAP/2;
  pipeSeg(p.x,-6,topH-26+6); pipeCap(p.x,topH-26);
  pipeSeg(p.x,botY+26,PLAY_H-botY-26+6); pipeCap(p.x,botY);}

/* ---------- text / UI ---------- */
function text(str,x,y,size,fill,align){
  ctx.font='900 '+size+'px "Arial Rounded MT Bold","Arial Black",Arial,sans-serif';
  ctx.textAlign=align||'center';ctx.textBaseline='middle';
  ctx.lineWidth=Math.max(3,size/5.5);ctx.strokeStyle='#543847';
  ctx.strokeText(str,x,y);ctx.fillStyle=fill;ctx.fillText(str,x,y);}
function rr(x,y,w,h,r){ctx.beginPath();ctx.moveTo(x+r,y);
  ctx.arcTo(x+w,y,x+w,y+h,r);ctx.arcTo(x+w,y+h,x,y+h,r);
  ctx.arcTo(x,y+h,x,y,r);ctx.arcTo(x,y,x+w,y,r);ctx.closePath();}
function star(cx,cy,r){ctx.beginPath();
  for(let i=0;i<10;i++){const a=-Math.PI/2+i*Math.PI/5,rr2=i%2?r*0.45:r;
    ctx[i?'lineTo':'moveTo'](cx+Math.cos(a)*rr2,cy+Math.sin(a)*rr2);}
  ctx.closePath();ctx.fill();}
function medalColor(){if(score>=40)return'#8BE3EE';if(score>=30)return'#F7CE4D';
  if(score>=20)return'#D9DDE2';if(score>=10)return'#E09A5C';return null;}

function drawReady(){
  text('GET READY!',W/2,140,26,'#F9D648');
  ctx.save();ctx.globalAlpha=0.55+0.45*Math.sin(t*6);
  text('TAP  /  SPACE  TO  FLAP',W/2,310,13,'#fff');ctx.restore();}
function drawPanel(){
  const p=Math.min(1,Math.max(0,(deadT-0.55)/0.45));
  const e=1-Math.pow(1-p,3), y=H-(H-168)*e;
  ctx.save();ctx.globalAlpha=e;
  text('GAME OVER',W/2,120,26,'#FBEFC9');
  rr(24,y,240,126,10);ctx.fillStyle='#DED895';ctx.fill();
  ctx.lineWidth=3;ctx.strokeStyle='#543847';ctx.stroke();
  rr(30,y+6,228,114,7);ctx.lineWidth=2;
  ctx.strokeStyle='rgba(255,255,255,.45)';ctx.stroke();
  const mc=medalColor();
  ctx.beginPath();ctx.arc(76,y+63,21,0,7);ctx.fillStyle='#543847';ctx.fill();
  ctx.beginPath();ctx.arc(76,y+63,18,0,7);ctx.fillStyle=mc||'#C9B96F';ctx.fill();
  if(mc){ctx.fillStyle='rgba(255,255,255,.85)';star(76,y+63,9);}
  text('SCORE',246,y+34,11,'#fff','right');
  text(String(score),246,y+53,18,'#fff','right');
  text('BEST',246,y+84,11,'#fff','right');
  text(String(best),246,y+103,18,'#fff','right');
  if(newBest){rr(148,y+76,42,17,5);ctx.fillStyle='#E8442E';ctx.fill();
    text('NEW',169,y+85,10,'#fff');}
  ctx.restore();
  if(p>=1){ctx.save();ctx.globalAlpha=0.5+0.5*Math.sin(t*6);
    text('TAP TO RESTART',W/2,360,13,'#fff');ctx.restore();}}

/* ---------- game logic ---------- */
function spawnPipe(){pipes.push({x:W+10,gapY:100+Math.random()*(PLAY_H-200),scored:false});}
function circRect(cx,cy,r,rx,ry,rw,rh){
  const nx=Math.max(rx,Math.min(cx,rx+rw)),ny=Math.max(ry,Math.min(cy,ry+rh));
  return (cx-nx)*(cx-nx)+(cy-ny)*(cy-ny)<r*r;}
function hitPipe(p){
  const topH=p.gapY-GAP/2,botY=p.gapY+GAP/2;
  return circRect(BX,bird.y,11,p.x,-60,PIPE_W,topH+60)||
         circRect(BX,bird.y,11,p.x,botY,PIPE_W,PLAY_H-botY);}
function die(byPipe){
  if(state!=='play')return;
  state='dead';deadT=0;flash=1;sfxHit();sfxDie();
  if(byPipe){bird.v=-300;landed=false;}else{bird.v=0;landed=true;}
  try{localStorage.setItem('fbBest',best);}catch(e){}}
function doFlap(){bird.v=FLAP;sfxFlap();}
function restart(){
  score=0;pipes=[];bird={y:BASE_Y,v:0,rot:0};
  newBest=false;state='ready';deadT=0;landed=false;flash=0;}
function press(){
  audio();
  if(state==='ready'){state='play';doFlap();}
  else if(state==='play')doFlap();
  else if(state==='dead'&&deadT>1.0&&landed)restart();}

function update(dt){
  t+=dt;flash=Math.max(0,flash-dt*2.2);
  if(state!=='dead'){groundOff+=SPEED*dt;cityOff+=SPEED*0.25*dt;bushOff+=SPEED*0.55*dt;}
  if(state==='ready'){
    bird.y=BASE_Y+Math.sin(t*5)*6;bird.rot=0;bird.v=0;
  }else if(state==='play'){
    bird.v=Math.min(bird.v+GRAV*dt,TERM);bird.y+=bird.v*dt;
    const tgt=bird.v<0?-22:Math.min(85,bird.v*0.16);
    bird.rot+=(tgt-bird.rot)*Math.min(1,dt*12);
    if(bird.y<12){bird.y=12;if(bird.v<0)bird.v=0;}
    for(const p of pipes)p.x-=SPEED*dt;
    if(pipes.length===0||pipes[pipes.length-1].x<W-SPACING)spawnPipe();
    if(pipes[0]&&pipes[0].x<-80)pipes.shift();
    for(const p of pipes){
      if(!p.scored&&p.x+PIPE_W<BX){p.scored=true;score++;sfxPoint();
        if(score>best){best=score;newBest=true;}}
      if(hitPipe(p)){die(true);break;}}
    if(state==='play'&&bird.y+12>=PLAY_H){bird.y=PLAY_H-14;die(false);}
  }else{ /* dead */
    deadT+=dt;
    if(!landed){
      bird.v=Math.min(bird.v+GRAV*dt,TERM);bird.y+=bird.v*dt;
      bird.rot+=(90-bird.rot)*Math.min(1,dt*10);
      if(bird.y>=PLAY_H-14){bird.y=PLAY_H-14;landed=true;}}
  }
}

function drawBird(){
  const idx=state==='dead'?1:[0,1,2,1][Math.floor(t*9)%4];
  ctx.save();ctx.translate(BX,bird.y);ctx.rotate(bird.rot*Math.PI/180);
  ctx.drawImage(FRAMES[idx],-26,-23,51,45);ctx.restore();}

function render(){
  ctx.fillStyle='#70C5CE';ctx.fillRect(0,0,W,H);
  drawCity();drawBushes();
  for(const p of pipes)drawPipe(p);
  drawGround();drawBird();
  if(state==='play')text(String(score),W/2,72,36,'#fff');
  if(state==='ready')drawReady();
  if(state==='dead'&&deadT>0.55)drawPanel();
  if(flash>0){ctx.fillStyle='rgba(255,255,255,'+flash+')';ctx.fillRect(0,0,W,H);}
}

/* ---------- input & main loop ---------- */
addEventListener('keydown',e=>{
  if(e.code==='Space'||e.code==='ArrowUp'||e.code==='KeyW'){e.preventDefault();press();}});
cv.addEventListener('pointerdown',e=>{e.preventDefault();press();});
document.addEventListener('touchmove',e=>e.preventDefault(),{passive:false});

let last=performance.now();
function loop(now){
  const dt=Math.min(0.033,(now-last)/1000);last=now;
  update(dt);render();requestAnimationFrame(loop);}
requestAnimationFrame(loop);
</script>
</body>
</html>
```

**What's included:**

- **Faithful visuals** — the bird is a hand-drawn 17×15 pixel sprite with three wing-flap frames, rotation physics (tilts up on flap, nosedives when falling, spins face-down on death), rendered at native 288×512 like the original. Pipes use the classic green highlight gradient with dark outlines and wider caps, and the ground scrolls with diagonal stripes plus the moving green edge dashes.
- **Authentic game feel** — gravity, flap impulse, terminal velocity, pipe gap, and speed are tuned to match the original's difficulty, with a "Get Ready" screen, death flash, fall-to-ground sequence, and a sliding score panel with medals (bronze/silver/gold/platinum), best score, and a "NEW" badge.
- **Endless and responsive** — pipes spawn forever, the canvas letterboxes perfectly on any screen size or phone, and controls work with tap, click, Space, W, or Arrow Up.
- **Zero external dependencies** — sprites are generated on an offscreen canvas and sounds are synthesized with WebAudio, so the single file works offline.

If you'd like, I can also add a night-mode palette, difficulty ramping (pipes speeding up over time), or adjustable gap size.
