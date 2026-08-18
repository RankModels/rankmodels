Here's a complete, playable CSS-only Snake. Save it as `snake.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>CSS Snake — zero JavaScript</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;min-height:100vh;display:flex;align-items:center;justify-content:center;
  background:radial-gradient(circle at 50% 0,#12321f,#07110d 70%);
  color:#dff5e6;font-family:ui-monospace,Menlo,Consolas,monospace;
  counter-reset:sc;
}
/* hidden game state — opacity, NOT display:none, so CSS counters still see them */
.st,.fd{position:fixed;top:0;left:0;width:1px;height:1px;opacity:0;pointer-events:none;
  -webkit-appearance:none;appearance:none;margin:0;border:0}

.wrap{--cell:clamp(22px,6.2vw,42px);--fade:.55s;padding:18px}
header{width:calc(var(--cell)*12);margin:0 auto 12px}
.row{display:flex;justify-content:space-between;align-items:baseline;gap:12px}
h1{font-size:15px;letter-spacing:.3em;margin:0;text-transform:uppercase;color:#7ee39b}
.score{font-size:13px;color:#9fe8b7}
.score::after{content:" " counter(sc) " / 12";color:#fff;font-weight:700}
.bar{height:8px;border-radius:99px;background:#0d2418;overflow:hidden;margin-top:10px;border:1px solid #1d4530}
.bar>span{display:block;height:100%;width:100%;background:#4ade80;
  animation:drain 60s linear forwards paused}
#start:checked ~ .wrap .bar>span{animation-play-state:running}
@keyframes drain{0%{width:100%;background:#4ade80}60%{background:#facc15}100%{width:0;background:#ef4444}}

/* ---------- board ---------- */
.board{position:relative;margin:0 auto;width:calc(var(--cell)*12);
  display:grid;grid-template-columns:repeat(12,var(--cell));
  background:#0a1a12;border:2px solid #1d4530;border-radius:12px;overflow:hidden;
  cursor:none;box-shadow:0 22px 60px rgba(0,0,0,.6)}
.board>i{position:relative;width:var(--cell);height:var(--cell);
  box-shadow:inset 0 0 0 1px rgba(126,227,155,.05)}

/* snake body: lights up instantly on hover, fades slowly = a trailing tail */
.board>i::after{content:"";position:absolute;inset:8%;border-radius:28%;z-index:1;
  background:linear-gradient(145deg,#6ef29a,#25b062);opacity:0;transform:scale(.35);
  transition:opacity var(--fade) linear,transform var(--fade) linear}
.board>i:hover::after{opacity:1;transform:scale(1);transition:none;
  background:linear-gradient(145deg,#e6fff0,#59e58a);box-shadow:0 0 16px rgba(110,242,154,.55)}
/* eyes on the head only */
.board>i::before{content:"";position:absolute;inset:8%;border-radius:28%;z-index:2;opacity:0;
  background:radial-gradient(circle at 34% 40%,#07150e 13%,transparent 14%),
             radial-gradient(circle at 66% 40%,#07150e 13%,transparent 14%);
  transition:opacity .09s linear}
.board>i:hover::before{opacity:1}

/* ---------- apples ---------- */
.food{position:absolute;display:none;z-index:3;border-radius:28%;cursor:none;
  width:var(--cell);height:var(--cell);
  left:calc(var(--x)*var(--cell));top:calc(var(--y)*var(--cell))}
.food::before{content:"";position:absolute;inset:20%;border-radius:50%;
  background:radial-gradient(circle at 32% 30%,#ffa3a3,#df2138 70%);
  box-shadow:0 0 14px rgba(223,33,56,.6);animation:pulse 1.1s ease-in-out infinite}
.food::after{content:"";position:absolute;left:54%;top:10%;width:24%;height:15%;
  border-radius:50% 50% 50% 0;background:#48c46a;transform:rotate(-20deg)}
@keyframes pulse{50%{transform:scale(.8)}}
.food:hover{background:linear-gradient(145deg,#e6fff0,#59e58a);box-shadow:0 0 16px rgba(110,242,154,.6)}
.food:hover::before{inset:32%}

/* apples appear one at a time */
.wrap label[for="f1"]{display:block}
#f1:checked ~ .wrap label[for="f2"]{display:block}
#f2:checked ~ .wrap label[for="f3"]{display:block}
#f3:checked ~ .wrap label[for="f4"]{display:block}
#f4:checked ~ .wrap label[for="f5"]{display:block}
#f5:checked ~ .wrap label[for="f6"]{display:block}
#f6:checked ~ .wrap label[for="f7"]{display:block}
#f7:checked ~ .wrap label[for="f8"]{display:block}
#f8:checked ~ .wrap label[for="f9"]{display:block}
#f9:checked ~ .wrap label[for="f10"]{display:block}
#f10:checked ~ .wrap label[for="f11"]{display:block}
#f11:checked ~ .wrap label[for="f12"]{display:block}
/* eaten apples vanish (must come after the rules above) */
#f1:checked ~ .wrap label[for="f1"]{display:none}
#f2:checked ~ .wrap label[for="f2"]{display:none}
#f3:checked ~ .wrap label[for="f3"]{display:none}
#f4:checked ~ .wrap label[for="f4"]{display:none}
#f5:checked ~ .wrap label[for="f5"]{display:none}
#f6:checked ~ .wrap label[for="f6"]{display:none}
#f7:checked ~ .wrap label[for="f7"]{display:none}
#f8:checked ~ .wrap label[for="f8"]{display:none}
#f9:checked ~ .wrap label[for="f9"]{display:none}
#f10:checked ~ .wrap label[for="f10"]{display:none}
#f11:checked ~ .wrap label[for="f11"]{display:none}
#f12:checked ~ .wrap label[for="f12"]{display:none}

/* the snake grows as you eat */
#f2:checked  ~ .wrap{--fade:.7s}
#f4:checked  ~ .wrap{--fade:.85s}
#f6:checked  ~ .wrap{--fade:1s}
#f8:checked  ~ .wrap{--fade:1.2s}
#f10:checked ~ .wrap{--fade:1.4s}

/* score counter, pure CSS */
.fd:checked{counter-increment:sc}

/* ---------- overlays ---------- */
.ov{position:absolute;inset:0;z-index:5;display:flex;flex-direction:column;
  align-items:center;justify-content:center;gap:14px;text-align:center;padding:22px;
  background:rgba(4,12,8,.93);cursor:pointer;opacity:0;visibility:hidden}
.ov b{font-size:20px;letter-spacing:.2em;text-transform:uppercase;color:#7ee39b}
.ov em{font-style:normal;font-size:12.5px;line-height:1.7;color:#a9c9b6;max-width:28ch}
.btn{background:#4ade80;color:#08170f;border-radius:99px;padding:9px 22px;
  font-size:12.5px;font-weight:700;text-decoration:none;letter-spacing:.08em}
.start{opacity:1;visibility:visible}
#start:checked ~ .wrap .start{display:none}
.over{animation:reveal .5s linear 60s both paused}
#start:checked ~ .wrap .over{animation-play-state:running}
#f12:checked ~ .wrap .over{display:none}
#f12:checked ~ .wrap .win{opacity:1;visibility:visible}
@keyframes reveal{from{opacity:0;visibility:hidden}to{opacity:1;visibility:visible}}

footer{width:calc(var(--cell)*12);margin:12px auto 0;font-size:11px;line-height:1.7;color:#5f8471}
</style>
</head>
<body>

<!-- game state lives entirely in these checkboxes -->
<input type="checkbox" id="start" class="st">
<input type="checkbox" id="f1"  class="fd"><input type="checkbox" id="f2"  class="fd">
<input type="checkbox" id="f3"  class="fd"><input type="checkbox" id="f4"  class="fd">
<input type="checkbox" id="f5"  class="fd"><input type="checkbox" id="f6"  class="fd">
<input type="checkbox" id="f7"  class="fd"><input type="checkbox" id="f8"  class="fd">
<input type="checkbox" id="f9"  class="fd"><input type="checkbox" id="f10" class="fd">
<input type="checkbox" id="f11" class="fd"><input type="checkbox" id="f12" class="fd">

<div class="wrap">
  <header>
    <div class="row"><h1>CSS Snake</h1><span class="score">apples</span></div>
    <div class="bar"><span></span></div>
  </header>

  <div class="board">
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>
    <i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i>

    <label class="food" for="f1"  style="--x:8;--y:3"></label>
    <label class="food" for="f2"  style="--x:2;--y:7"></label>
    <label class="food" for="f3"  style="--x:10;--y:9"></label>
    <label class="food" for="f4"  style="--x:5;--y:1"></label>
    <label class="food" for="f5"  style="--x:1;--y:4"></label>
    <label class="food" for="f6"  style="--x:7;--y:10"></label>
    <label class="food" for="f7"  style="--x:11;--y:5"></label>
    <label class="food" for="f8"  style="--x:3;--y:11"></label>
    <label class="food" for="f9"  style="--x:6;--y:6"></label>
    <label class="food" for="f10" style="--x:0;--y:9"></label>
    <label class="food" for="f11" style="--x:9;--y:0"></label>
    <label class="food" for="f12" style="--x:4;--y:8"></label>

    <label class="ov start" for="start">
      <b>CSS Snake</b>
      <em>Steer the snake with your cursor — it follows you and trails behind.
      Click an apple to eat it. Twelve apples, sixty seconds.</em>
      <span class="btn">Start</span>
    </label>

    <div class="ov over">
      <b>Time's up</b>
      <em>You ate <span class="score"></span> apples.</em>
      <a class="btn" href="">Play again</a>
    </div>

    <div class="ov win">
      <b>Full belly!</b>
      <em>All twelve apples eaten. The snake thanks you.</em>
      <a class="btn" href="">Play again</a>
    </div>
  </div>

  <footer>No JavaScript: the tail is a hover transition, the score is a CSS counter,
  the timer is a paused animation, and eaten apples are checked checkboxes.</footer>
</div>

</body>
</html>
```

How the tricks work: every one of the 144 grid squares turns bright green instantly on `:hover` and then fades back out over `--fade` seconds, so a trail of squares lingers behind your cursor — that's the snake. Eating is the checkbox hack: each apple is a `<label>` for a hidden checkbox, and `#f3:checked ~ .wrap label[for="f4"]` reveals the next one, so apples appear in sequence. The score is a real CSS counter (`counter-increment` on `:checked` inputs), which is why the checkboxes are hidden with opacity rather than `display:none` — hidden boxes don't increment counters. The clock is a 60-second animation that sits `paused` until the start checkbox flips it to `running`, and a second animation with a 60s delay fades in the game-over screen.

Two honest limitations of the no-JS constraint: it needs a real pointer (hover doesn't exist on touchscreens), and there's no self-collision or wall death, because CSS can't remember that something happened unless you click. Restarting is a plain link back to the same page.
