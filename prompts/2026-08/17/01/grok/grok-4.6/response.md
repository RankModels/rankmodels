A real-time loop needs JS, so this is a turn-based CSS-only snake: click a glowing neighbor (or the D-pad), chomp apples to grow, all in one file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CSS Snake — no JavaScript</title>
<style>
  @import url("https://fonts.googleapis.com/css2?family=Press+Start+2P&family=VT323&display=swap");

  :root {
    --bg: #070b08;
    --panel: #101710;
    --line: #1c2a1c;
    --lime: #39ff14;
    --lime-dim: #1d8a2e;
    --apple: #ff3b3b;
    --head: #5cff6a;
    --text: #c8ffc8;
    --muted: #6e9a6e;
    --n: 8;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { min-height: 100%; }
  body {
    font-family: VT323, ui-monospace, monospace;
    background:
      radial-gradient(1000px 500px at 50% -10%, #163016 0%, transparent 55%),
      var(--bg);
    color: var(--text);
    display: grid;
    place-items: center;
    padding: 20px 14px 32px;
  }

  .controls {
    position: absolute;
    width: 1px; height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
  }

  .cabinet {
    width: min(100%, 520px);
    background: linear-gradient(180deg, #161e16, #0c120c);
    border: 3px solid #243324;
    border-radius: 28px;
    box-shadow:
      0 0 0 1px #000,
      0 20px 50px rgba(0,0,0,.45),
      inset 0 1px 0 rgba(255,255,255,.04);
    padding: 18px 18px 16px;
    position: relative;
  }

  header.top {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-bottom: 12px;
    gap: 10px;
  }
  .brand {
    font-family: "Press Start 2P", monospace;
    font-size: 13px;
    letter-spacing: 1px;
    color: var(--lime);
    text-shadow: 0 0 12px rgba(57,255,20,.45);
  }
  .brand small {
    display: block;
    margin-top: 6px;
    font-size: 8px;
    color: var(--muted);
    letter-spacing: 0;
    text-shadow: none;
    font-family: VT323, monospace;
    font-size: 16px;
  }
  .hud {
    text-align: right;
    font-size: 22px;
    line-height: 1;
    color: var(--lime);
  }
  .hud span { display: block; color: var(--muted); font-size: 15px; letter-spacing: 1px; }

  .score::after { content: "0"; }
  body:has(#f1:checked) .score::after { content: "1"; }
  body:has(#f1:checked):has(#f2:checked) .score::after { content: "2"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked) .score::after { content: "3"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked) .score::after { content: "4"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked) .score::after { content: "5"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked):has(#f6:checked) .score::after { content: "6"; }

  .len::after { content: "3"; }
  body:has(#f1:checked) .len::after { content: "4"; }
  body:has(#f1:checked):has(#f2:checked) .len::after { content: "5"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked) .len::after { content: "6"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked) .len::after { content: "7"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked) .len::after { content: "8"; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked):has(#f6:checked) .len::after { content: "9"; }

  .screen {
    position: relative;
    background: #071109;
    border-radius: 16px;
    overflow: hidden;
    box-shadow: inset 0 0 0 2px #142014, inset 0 0 40px rgba(0,0,0,.55);
  }
  .screen::after {
    content: "";
    pointer-events: none;
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
      to bottom,
      transparent 0 2px,
      rgba(0,0,0,.12) 2px 3px
    );
    z-index: 8;
    mix-blend-mode: multiply;
  }

  .board-wrap {
    position: relative;
    width: 100%;
    aspect-ratio: 1;
  }
  .board {
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    grid-template-rows: repeat(8, 1fr);
    width: 100%;
    height: 100%;
    background:
      linear-gradient(var(--line) 1px, transparent 1px) 0 0 / 12.5% 12.5%,
      linear-gradient(90deg, var(--line) 1px, transparent 1px) 0 0 / 12.5% 12.5%,
      #0a140c;
  }
  .cell { position: relative; }
  .cell > label.move {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 2;
  }

  /* valid-move glow */
  .glow {
    background: rgba(57,255,20,.10);
    box-shadow: inset 0 0 0 2px rgba(57,255,20,.45);
    cursor: pointer;
    pointer-events: auto;
  }

  .snake {
    position: absolute;
    width: 12.5%;
    height: 12.5%;
    left: 0; top: 0;
    pointer-events: none;
    z-index: 4;
    transition: transform .16s cubic-bezier(.2,.85,.2,1);
    display: grid;
    place-items: center;
    filter: drop-shadow(0 0 8px rgba(57,255,20,.35));
  }
  .head {
    width: 72%;
    height: 72%;
    border-radius: 38% 42% 36% 40%;
    background: radial-gradient(circle at 35% 30%, #b6ff9a, var(--head) 42%, #14a33a);
    position: relative;
  }
  .head .eye {
    position: absolute;
    top: 26%;
    width: 18%;
    height: 18%;
    background: #062006;
    border-radius: 50%;
    box-shadow: 4px 0 0 -1px #eaffea inset;
  }
  .head .eye.l { left: 22%; }
  .head .eye.r { right: 22%; }
  .head .tongue {
    position: absolute;
    left: 50%;
    bottom: -18%;
    width: 10%;
    height: 22%;
    background: #ff4d6a;
    transform: translateX(-50%);
    clip-path: polygon(0 0, 100% 0, 100% 60%, 50% 100%, 0 60%);
    animation: flick 1.1s infinite;
  }
  @keyframes flick {
    0%, 70%, 100% { transform: translateX(-50%) scaleY(.2); opacity: .2; }
    78%, 88% { transform: translateX(-50%) scaleY(1); opacity: 1; }
  }

  /* coiled tail grows with each apple */
  .head {
    box-shadow:
      -10px 7px 0 -2px var(--lime-dim),
      -4px 14px 0 -3px #178a32;
  }
  body:has(#f1:checked) .head {
    box-shadow:
      -11px 6px 0 -1px var(--lime-dim),
      -16px 14px 0 -2px #1a9a38,
      -8px 20px 0 -3px #147a2c;
  }
  body:has(#f1:checked):has(#f2:checked) .head {
    box-shadow:
      -11px 6px 0 0 var(--lime-dim),
      -18px 13px 0 -1px #1a9a38,
      -16px 22px 0 -2px #147a2c,
      -6px 26px 0 -3px #0f6624;
  }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked) .head {
    box-shadow:
      -12px 5px 0 0 var(--lime-dim),
      -20px 12px 0 0 #1a9a38,
      -20px 22px 0 -1px #147a2c,
      -10px 28px 0 -2px #0f6624,
      2px 28px 0 -3px #0c541d;
  }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked) .head {
    box-shadow:
      -12px 5px 0 1px var(--lime-dim),
      -22px 12px 0 0 #1a9a38,
      -24px 22px 0 -1px #147a2c,
      -14px 30px 0 -1px #0f6624,
      0px 32px 0 -2px #0c541d,
      12px 26px 0 -3px #0a4618;
  }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked) .head {
    box-shadow:
      -12px 4px 0 1px var(--lime-dim),
      -23px 11px 0 0 #1a9a38,
      -26px 22px 0 0 #147a2c,
      -18px 32px 0 -1px #0f6624,
      -2px 34px 0 -1px #0c541d,
      12px 30px 0 -2px #0a4618,
      16px 18px 0 -3px #083812;
  }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked):has(#f6:checked) .head {
    box-shadow:
      -12px 4px 0 2px var(--lime-dim),
      -24px 10px 0 1px #1a9a38,
      -28px 22px 0 0 #147a2c,
      -20px 34px 0 0 #0f6624,
      -4px 36px 0 -1px #0c541d,
      12px 32px 0 -1px #0a4618,
      20px 20px 0 -2px #083812,
      14px 8px 0 -3px #06280d;
  }

  /* head position: pRC → translate(C00%, R00%) */
  body:has(#p00:checked) .snake{transform:translate(0%,0%)}
  body:has(#p01:checked) .snake{transform:translate(100%,0%)}
  body:has(#p02:checked) .snake{transform:translate(200%,0%)}
  body:has(#p03:checked) .snake{transform:translate(300%,0%)}
  body:has(#p04:checked) .snake{transform:translate(400%,0%)}
  body:has(#p05:checked) .snake{transform:translate(500%,0%)}
  body:has(#p06:checked) .snake{transform:translate(600%,0%)}
  body:has(#p07:checked) .snake{transform:translate(700%,0%)}
  body:has(#p10:checked) .snake{transform:translate(0%,100%)}
  body:has(#p11:checked) .snake{transform:translate(100%,100%)}
  body:has(#p12:checked) .snake{transform:translate(200%,100%)}
  body:has(#p13:checked) .snake{transform:translate(300%,100%)}
  body:has(#p14:checked) .snake{transform:translate(400%,100%)}
  body:has(#p15:checked) .snake{transform:translate(500%,100%)}
  body:has(#p16:checked) .snake{transform:translate(600%,100%)}
  body:has(#p17:checked) .snake{transform:translate(700%,100%)}
  body:has(#p20:checked) .snake{transform:translate(0%,200%)}
  body:has(#p21:checked) .snake{transform:translate(100%,200%)}
  body:has(#p22:checked) .snake{transform:translate(200%,200%)}
  body:has(#p23:checked) .snake{transform:translate(300%,200%)}
  body:has(#p24:checked) .snake{transform:translate(400%,200%)}
  body:has(#p25:checked) .snake{transform:translate(500%,200%)}
  body:has(#p26:checked) .snake{transform:translate(600%,200%)}
  body:has(#p27:checked) .snake{transform:translate(700%,200%)}
  body:has(#p30:checked) .snake{transform:translate(0%,300%)}
  body:has(#p31:checked) .snake{transform:translate(100%,300%)}
  body:has(#p32:checked) .snake{transform:translate(200%,300%)}
  body:has(#p33:checked) .snake{transform:translate(300%,300%)}
  body:has(#p34:checked) .snake{transform:translate(400%,300%)}
  body:has(#p35:checked) .snake{transform:translate(500%,300%)}
  body:has(#p36:checked) .snake{transform:translate(600%,300%)}
  body:has(#p37:checked) .snake{transform:translate(700%,300%)}
  body:has(#p40:checked) .snake{transform:translate(0%,400%)}
  body:has(#p41:checked) .snake{transform:translate(100%,400%)}
  body:has(#p42:checked) .snake{transform:translate(200%,400%)}
  body:has(#p43:checked) .snake{transform:translate(300%,400%)}
  body:has(#p44:checked) .snake{transform:translate(400%,400%)}
  body:has(#p45:checked) .snake{transform:translate(500%,400%)}
  body:has(#p46:checked) .snake{transform:translate(600%,400%)}
  body:has(#p47:checked) .snake{transform:translate(700%,400%)}
  body:has(#p50:checked) .snake{transform:translate(0%,500%)}
  body:has(#p51:checked) .snake{transform:translate(100%,500%)}
  body:has(#p52:checked) .snake{transform:translate(200%,500%)}
  body:has(#p53:checked) .snake{transform:translate(300%,500%)}
  body:has(#p54:checked) .snake{transform:translate(400%,500%)}
  body:has(#p55:checked) .snake{transform:translate(500%,500%)}
  body:has(#p56:checked) .snake{transform:translate(600%,500%)}
  body:has(#p57:checked) .snake{transform:translate(700%,500%)}
  body:has(#p60:checked) .snake{transform:translate(0%,600%)}
  body:has(#p61:checked) .snake{transform:translate(100%,600%)}
  body:has(#p62:checked) .snake{transform:translate(200%,600%)}
  body:has(#p63:checked) .snake{transform:translate(300%,600%)}
  body:has(#p64:checked) .snake{transform:translate(400%,600%)}
  body:has(#p65:checked) .snake{transform:translate(500%,600%)}
  body:has(#p66:checked) .snake{transform:translate(600%,600%)}
  body:has(#p67:checked) .snake{transform:translate(700%,600%)}
  body:has(#p70:checked) .snake{transform:translate(0%,700%)}
  body:has(#p71:checked) .snake{transform:translate(100%,700%)}
  body:has(#p72:checked) .snake{transform:translate(200%,700%)}
  body:has(#p73:checked) .snake{transform:translate(300%,700%)}
  body:has(#p74:checked) .snake{transform:translate(400%,700%)}
  body:has(#p75:checked) .snake{transform:translate(500%,700%)}
  body:has(#p76:checked) .snake{transform:translate(600%,700%)}
  body:has(#p77:checked) .snake{transform:translate(700%,700%)}

  /* apples */
  .apple {
    display: none;
    position: absolute;
    inset: 22%;
    z-index: 3;
    pointer-events: none;
  }
  .apple i {
    display: block;
    width: 100%;
    height: 100%;
    background: radial-gradient(circle at 30% 30%, #ff8080, var(--apple) 46%, #b10f0f);
    border-radius: 50% 50% 48% 52%;
    box-shadow: 0 0 10px rgba(255,50,50,.45);
    position: relative;
  }
  .apple i::before {
    content: "";
    position: absolute;
    top: -18%;
    left: 42%;
    width: 18%;
    height: 28%;
    background: #3d9a2a;
    border-radius: 40% 80% 20% 60%;
    transform: rotate(25deg);
  }
  .a1 { display: block; }
  body:has(#f1:checked) .a1 { display: none; }
  body:has(#f1:checked) .a2 { display: block; }
  body:has(#f2:checked) .a2 { display: none; }
  body:has(#f2:checked) .a3 { display: block; }
  body:has(#f3:checked) .a3 { display: none; }
  body:has(#f3:checked) .a4 { display: block; }
  body:has(#f4:checked) .a4 { display: none; }
  body:has(#f4:checked) .a5 { display: block; }
  body:has(#f5:checked) .a5 { display: none; }
  body:has(#f5:checked) .a6 { display: block; }
  body:has(#f6:checked) .a6 { display: none; }

  body:has(#p26:checked):not(:has(#f1:checked)) .a1,
  body:has(#p07:checked):has(#f1:checked):not(:has(#f2:checked)) .a2,
  body:has(#p57:checked):has(#f2:checked):not(:has(#f3:checked)) .a3,
  body:has(#p71:checked):has(#f3:checked):not(:has(#f4:checked)) .a4,
  body:has(#p40:checked):has(#f4:checked):not(:has(#f5:checked)) .a5,
  body:has(#p33:checked):has(#f5:checked):not(:has(#f6:checked)) .a6 {
    animation: pulse .5s ease-in-out infinite;
  }
  @keyframes pulse {
    50% { transform: scale(1.18); filter: brightness(1.25); }
  }

  .below {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    margin-top: 12px;
    min-height: 52px;
  }
  .hint {
    color: var(--muted);
    font-size: 20px;
    flex: 1;
  }
  .hint::after { content: "Click a glowing neighbor — or use the D-pad."; }
  body:has(#p26:checked):not(:has(#f1:checked)) .hint::after,
  body:has(#p07:checked):has(#f1:checked):not(:has(#f2:checked)) .hint::after,
  body:has(#p57:checked):has(#f2:checked):not(:has(#f3:checked)) .hint::after,
  body:has(#p71:checked):has(#f3:checked):not(:has(#f4:checked)) .hint::after,
  body:has(#p40:checked):has(#f4:checked):not(:has(#f5:checked)) .hint::after,
  body:has(#p33:checked):has(#f5:checked):not(:has(#f6:checked)) .hint::after {
    content: "CHOMP! Click EAT to grow.";
    color: #ffd166;
  }

  .eat {
    display: none;
    align-items: center;
    justify-content: center;
    gap: 8px;
    background: #ff3b3b;
    color: #fff;
    font-family: "Press Start 2P", monospace;
    font-size: 10px;
    border-radius: 999px;
    padding: 12px 16px;
    cursor: pointer;
    box-shadow: 0 0 0 3px #5a1010, 0 8px 20px rgba(255,40,40,.3);
    animation: pulse .5s ease-in-out infinite;
    user-select: none;
  }
  body:has(#p26:checked):not(:has(#f1:checked)) .e1,
  body:has(#p07:checked):has(#f1:checked):not(:has(#f2:checked)) .e2,
  body:has(#p57:checked):has(#f2:checked):not(:has(#f3:checked)) .e3,
  body:has(#p71:checked):has(#f3:checked):not(:has(#f4:checked)) .e4,
  body:has(#p40:checked):has(#f4:checked):not(:has(#f5:checked)) .e5,
  body:has(#p33:checked):has(#f5:checked):not(:has(#f6:checked)) .e6 { display: flex; }

  .dpad {
    display: grid;
    grid-template-areas:
      ". u ."
      "l c r"
      ". d .";
    grid-template-columns: 58px 58px 58px;
    grid-template-rows: 50px 50px 50px;
    gap: 6px;
    justify-content: center;
    margin-top: 8px;
  }
  .key {
    position: relative;
    background: #1a241a;
    border: 2px solid #2c3c2c;
    border-radius: 12px;
    display: grid;
    place-items: center;
    color: var(--lime);
    font-size: 20px;
    opacity: .32;
    box-shadow: inset 0 -3px 0 #0a0e0a;
    user-select: none;
  }
  .key.u { grid-area: u; }
  .key.l { grid-area: l; }
  .key.r { grid-area: r; }
  .key.d { grid-area: d; }
  .key.c {
    grid-area: c;
    opacity: 1;
    background: #101810;
    color: #3d5a3d;
    font-size: 11px;
    letter-spacing: 1px;
  }
  .key label {
    position: absolute;
    inset: 0;
    pointer-events: none;
    cursor: pointer;
  }

  /* D-pad lights up when that direction is legal */
  body:has(#p10:checked) .u, body:has(#p11:checked) .u, body:has(#p12:checked) .u, body:has(#p13:checked) .u,
  body:has(#p14:checked) .u, body:has(#p15:checked) .u, body:has(#p16:checked) .u, body:has(#p17:checked) .u,
  body:has(#p20:checked) .u, body:has(#p21:checked) .u, body:has(#p22:checked) .u, body:has(#p23:checked) .u,
  body:has(#p24:checked) .u, body:has(#p25:checked) .u, body:has(#p26:checked) .u, body:has(#p27:checked) .u,
  body:has(#p30:checked) .u, body:has(#p31:checked) .u, body:has(#p32:checked) .u, body:has(#p33:checked) .u,
  body:has(#p34:checked) .u, body:has(#p35:checked) .u, body:has(#p36:checked) .u, body:has(#p37:checked) .u,
  body:has(#p40:checked) .u, body:has(#p41:checked) .u, body:has(#p42:checked) .u, body:has(#p43:checked) .u,
  body:has(#p44:checked) .u, body:has(#p45:checked) .u, body:has(#p46:checked) .u, body:has(#p47:checked) .u,
  body:has(#p50:checked) .u, body:has(#p51:checked) .u, body:has(#p52:checked) .u, body:has(#p53:checked) .u,
  body:has(#p54:checked) .u, body:has(#p55:checked) .u, body:has(#p56:checked) .u, body:has(#p57:checked) .u,
  body:has(#p60:checked) .u, body:has(#p61:checked) .u, body:has(#p62:checked) .u, body:has(#p63:checked) .u,
  body:has(#p64:checked) .u, body:has(#p65:checked) .u, body:has(#p66:checked) .u, body:has(#p67:checked) .u,
  body:has(#p70:checked) .u, body:has(#p71:checked) .u, body:has(#p72:checked) .u, body:has(#p73:checked) .u,
  body:has(#p74:checked) .u, body:has(#p75:checked) .u, body:has(#p76:checked) .u, body:has(#p77:checked) .u { opacity: 1; }

  body:has(#p00:checked) .d, body:has(#p01:checked) .d, body:has(#p02:checked) .d, body:has(#p03:checked) .d,
  body:has(#p04:checked) .d, body:has(#p05:checked) .d, body:has(#p06:checked) .d, body:has(#p07:checked) .d,
  body:has(#p10:checked) .d, body:has(#p11:checked) .d, body:has(#p12:checked) .d, body:has(#p13:checked) .d,
  body:has(#p14:checked) .d, body:has(#p15:checked) .d, body:has(#p16:checked) .d, body:has(#p17:checked) .d,
  body:has(#p20:checked) .d, body:has(#p21:checked) .d, body:has(#p22:checked) .d, body:has(#p23:checked) .d,
  body:has(#p24:checked) .d, body:has(#p25:checked) .d, body:has(#p26:checked) .d, body:has(#p27:checked) .d,
  body:has(#p30:checked) .d, body:has(#p31:checked) .d, body:has(#p32:checked) .d, body:has(#p33:checked) .d,
  body:has(#p34:checked) .d, body:has(#p35:checked) .d, body:has(#p36:checked) .d, body:has(#p37:checked) .d,
  body:has(#p40:checked) .d, body:has(#p41:checked) .d, body:has(#p42:checked) .d, body:has(#p43:checked) .d,
  body:has(#p44:checked) .d, body:has(#p45:checked) .d, body:has(#p46:checked) .d, body:has(#p47:checked) .d,
  body:has(#p50:checked) .d, body:has(#p51:checked) .d, body:has(#p52:checked) .d, body:has(#p53:checked) .d,
  body:has(#p54:checked) .d, body:has(#p55:checked) .d, body:has(#p56:checked) .d, body:has(#p57:checked) .d,
  body:has(#p60:checked) .d, body:has(#p61:checked) .d, body:has(#p62:checked) .d, body:has(#p63:checked) .d,
  body:has(#p64:checked) .d, body:has(#p65:checked) .d, body:has(#p66:checked) .d, body:has(#p67:checked) .d { opacity: 1; }

  body:has(#p01:checked) .l, body:has(#p02:checked) .l, body:has(#p03:checked) .l, body:has(#p04:checked) .l,
  body:has(#p05:checked) .l, body:has(#p06:checked) .l, body:has(#p07:checked) .l,
  body:has(#p11:checked) .l, body:has(#p12:checked) .l, body:has(#p13:checked) .l, body:has(#p14:checked) .l,
  body:has(#p15:checked) .l, body:has(#p16:checked) .l, body:has(#p17:checked) .l,
  body:has(#p21:checked) .l, body:has(#p22:checked) .l, body:has(#p23:checked) .l, body:has(#p24:checked) .l,
  body:has(#p25:checked) .l, body:has(#p26:checked) .l, body:has(#p27:checked) .l,
  body:has(#p31:checked) .l, body:has(#p32:checked) .l, body:has(#p33:checked) .l, body:has(#p34:checked) .l,
  body:has(#p35:checked) .l, body:has(#p36:checked) .l, body:has(#p37:checked) .l,
  body:has(#p41:checked) .l, body:has(#p42:checked) .l, body:has(#p43:checked) .l, body:has(#p44:checked) .l,
  body:has(#p45:checked) .l, body:has(#p46:checked) .l, body:has(#p47:checked) .l,
  body:has(#p51:checked) .l, body:has(#p52:checked) .l, body:has(#p53:checked) .l, body:has(#p54:checked) .l,
  body:has(#p55:checked) .l, body:has(#p56:checked) .l, body:has(#p57:checked) .l,
  body:has(#p61:checked) .l, body:has(#p62:checked) .l, body:has(#p63:checked) .l, body:has(#p64:checked) .l,
  body:has(#p65:checked) .l, body:has(#p66:checked) .l, body:has(#p67:checked) .l,
  body:has(#p71:checked) .l, body:has(#p72:checked) .l, body:has(#p73:checked) .l, body:has(#p74:checked) .l,
  body:has(#p75:checked) .l, body:has(#p76:checked) .l, body:has(#p77:checked) .l { opacity: 1; }

  body:has(#p00:checked) .r, body:has(#p01:checked) .r, body:has(#p02:checked) .r, body:has(#p03:checked) .r,
  body:has(#p04:checked) .r, body:has(#p05:checked) .r, body:has(#p06:checked) .r,
  body:has(#p10:checked) .r, body:has(#p11:checked) .r, body:has(#p12:checked) .r, body:has(#p13:checked) .r,
  body:has(#p14:checked) .r, body:has(#p15:checked) .r, body:has(#p16:checked) .r,
  body:has(#p20:checked) .r, body:has(#p21:checked) .r, body:has(#p22:checked) .r, body:has(#p23:checked) .r,
  body:has(#p24:checked) .r, body:has(#p25:checked) .r, body:has(#p26:checked) .r,
  body:has(#p30:checked) .r, body:has(#p31:checked) .r, body:has(#p32:checked) .r, body:has(#p33:checked) .r,
  body:has(#p34:checked) .r, body:has(#p35:checked) .r, body:has(#p36:checked) .r,
  body:has(#p40:checked) .r, body:has(#p41:checked) .r, body:has(#p42:checked) .r, body:has(#p43:checked) .r,
  body:has(#p44:checked) .r, body:has(#p45:checked) .r, body:has(#p46:checked) .r,
  body:has(#p50:checked) .r, body:has(#p51:checked) .r, body:has(#p52:checked) .r, body:has(#p53:checked) .r,
  body:has(#p54:checked) .r, body:has(#p55:checked) .r, body:has(#p56:checked) .r,
  body:has(#p60:checked) .r, body:has(#p61:checked) .r, body:has(#p62:checked) .r, body:has(#p63:checked) .r,
  body:has(#p64:checked) .r, body:has(#p65:checked) .r, body:has(#p66:checked) .r,
  body:has(#p70:checked) .r, body:has(#p71:checked) .r, body:has(#p72:checked) .r, body:has(#p73:checked) .r,
  body:has(#p74:checked) .r, body:has(#p75:checked) .r, body:has(#p76:checked) .r { opacity: 1; }

  /* D-pad click targets */
  body:has(#p10:checked) .u [for=p00], body:has(#p11:checked) .u [for=p01], body:has(#p12:checked) .u [for=p02], body:has(#p13:checked) .u [for=p03],
  body:has(#p14:checked) .u [for=p04], body:has(#p15:checked) .u [for=p05], body:has(#p16:checked) .u [for=p06], body:has(#p17:checked) .u [for=p07],
  body:has(#p20:checked) .u [for=p10], body:has(#p21:checked) .u [for=p11], body:has(#p22:checked) .u [for=p12], body:has(#p23:checked) .u [for=p13],
  body:has(#p24:checked) .u [for=p14], body:has(#p25:checked) .u [for=p15], body:has(#p26:checked) .u [for=p16], body:has(#p27:checked) .u [for=p17],
  body:has(#p30:checked) .u [for=p20], body:has(#p31:checked) .u [for=p21], body:has(#p32:checked) .u [for=p22], body:has(#p33:checked) .u [for=p23],
  body:has(#p34:checked) .u [for=p24], body:has(#p35:checked) .u [for=p25], body:has(#p36:checked) .u [for=p26], body:has(#p37:checked) .u [for=p27],
  body:has(#p40:checked) .u [for=p30], body:has(#p41:checked) .u [for=p31], body:has(#p42:checked) .u [for=p32], body:has(#p43:checked) .u [for=p33],
  body:has(#p44:checked) .u [for=p34], body:has(#p45:checked) .u [for=p35], body:has(#p46:checked) .u [for=p36], body:has(#p47:checked) .u [for=p37],
  body:has(#p50:checked) .u [for=p40], body:has(#p51:checked) .u [for=p41], body:has(#p52:checked) .u [for=p42], body:has(#p53:checked) .u [for=p43],
  body:has(#p54:checked) .u [for=p44], body:has(#p55:checked) .u [for=p45], body:has(#p56:checked) .u [for=p46], body:has(#p57:checked) .u [for=p47],
  body:has(#p60:checked) .u [for=p50], body:has(#p61:checked) .u [for=p51], body:has(#p62:checked) .u [for=p52], body:has(#p63:checked) .u [for=p53],
  body:has(#p64:checked) .u [for=p54], body:has(#p65:checked) .u [for=p55], body:has(#p66:checked) .u [for=p56], body:has(#p67:checked) .u [for=p57],
  body:has(#p70:checked) .u [for=p60], body:has(#p71:checked) .u [for=p61], body:has(#p72:checked) .u [for=p62], body:has(#p73:checked) .u [for=p63],
  body:has(#p74:checked) .u [for=p64], body:has(#p75:checked) .u [for=p65], body:has(#p76:checked) .u [for=p66], body:has(#p77:checked) .u [for=p67],
  body:has(#p00:checked) .d [for=p10], body:has(#p01:checked) .d [for=p11], body:has(#p02:checked) .d [for=p12], body:has(#p03:checked) .d [for=p13],
  body:has(#p04:checked) .d [for=p14], body:has(#p05:checked) .d [for=p15], body:has(#p06:checked) .d [for=p16], body:has(#p07:checked) .d [for=p17],
  body:has(#p10:checked) .d [for=p20], body:has(#p11:checked) .d [for=p21], body:has(#p12:checked) .d [for=p22], body:has(#p13:checked) .d [for=p23],
  body:has(#p14:checked) .d [for=p24], body:has(#p15:checked) .d [for=p25], body:has(#p16:checked) .d [for=p26], body:has(#p17:checked) .d [for=p27],
  body:has(#p20:checked) .d [for=p30], body:has(#p21:checked) .d [for=p31], body:has(#p22:checked) .d [for=p32], body:has(#p23:checked) .d [for=p33],
  body:has(#p24:checked) .d [for=p34], body:has(#p25:checked) .d [for=p35], body:has(#p26:checked) .d [for=p36], body:has(#p27:checked) .d [for=p37],
  body:has(#p30:checked) .d [for=p40], body:has(#p31:checked) .d [for=p41], body:has(#p32:checked) .d [for=p42], body:has(#p33:checked) .d [for=p43],
  body:has(#p34:checked) .d [for=p44], body:has(#p35:checked) .d [for=p45], body:has(#p36:checked) .d [for=p46], body:has(#p37:checked) .d [for=p47],
  body:has(#p40:checked) .d [for=p50], body:has(#p41:checked) .d [for=p51], body:has(#p42:checked) .d [for=p52], body:has(#p43:checked) .d [for=p53],
  body:has(#p44:checked) .d [for=p54], body:has(#p45:checked) .d [for=p55], body:has(#p46:checked) .d [for=p56], body:has(#p47:checked) .d [for=p57],
  body:has(#p50:checked) .d [for=p60], body:has(#p51:checked) .d [for=p61], body:has(#p52:checked) .d [for=p62], body:has(#p53:checked) .d [for=p63],
  body:has(#p54:checked) .d [for=p64], body:has(#p55:checked) .d [for=p65], body:has(#p56:checked) .d [for=p66], body:has(#p57:checked) .d [for=p67],
  body:has(#p60:checked) .d [for=p70], body:has(#p61:checked) .d [for=p71], body:has(#p62:checked) .d [for=p72], body:has(#p63:checked) .d [for=p73],
  body:has(#p64:checked) .d [for=p74], body:has(#p65:checked) .d [for=p75], body:has(#p66:checked) .d [for=p76], body:has(#p67:checked) .d [for=p77],
  body:has(#p01:checked) .l [for=p00], body:has(#p02:checked) .l [for=p01], body:has(#p03:checked) .l [for=p02], body:has(#p04:checked) .l [for=p03],
  body:has(#p05:checked) .l [for=p04], body:has(#p06:checked) .l [for=p05], body:has(#p07:checked) .l [for=p06],
  body:has(#p11:checked) .l [for=p10], body:has(#p12:checked) .l [for=p11], body:has(#p13:checked) .l [for=p12], body:has(#p14:checked) .l [for=p13],
  body:has(#p15:checked) .l [for=p14], body:has(#p16:checked) .l [for=p15], body:has(#p17:checked) .l [for=p16],
  body:has(#p21:checked) .l [for=p20], body:has(#p22:checked) .l [for=p21], body:has(#p23:checked) .l [for=p22], body:has(#p24:checked) .l [for=p23],
  body:has(#p25:checked) .l [for=p24], body:has(#p26:checked) .l [for=p25], body:has(#p27:checked) .l [for=p26],
  body:has(#p31:checked) .l [for=p30], body:has(#p32:checked) .l [for=p31], body:has(#p33:checked) .l [for=p32], body:has(#p34:checked) .l [for=p33],
  body:has(#p35:checked) .l [for=p34], body:has(#p36:checked) .l [for=p35], body:has(#p37:checked) .l [for=p36],
  body:has(#p41:checked) .l [for=p40], body:has(#p42:checked) .l [for=p41], body:has(#p43:checked) .l [for=p42], body:has(#p44:checked) .l [for=p43],
  body:has(#p45:checked) .l [for=p44], body:has(#p46:checked) .l [for=p45], body:has(#p47:checked) .l [for=p46],
  body:has(#p51:checked) .l [for=p50], body:has(#p52:checked) .l [for=p51], body:has(#p53:checked) .l [for=p52], body:has(#p54:checked) .l [for=p53],
  body:has(#p55:checked) .l [for=p54], body:has(#p56:checked) .l [for=p55], body:has(#p57:checked) .l [for=p56],
  body:has(#p61:checked) .l [for=p60], body:has(#p62:checked) .l [for=p61], body:has(#p63:checked) .l [for=p62], body:has(#p64:checked) .l [for=p63],
  body:has(#p65:checked) .l [for=p64], body:has(#p66:checked) .l [for=p65], body:has(#p67:checked) .l [for=p66],
  body:has(#p71:checked) .l [for=p70], body:has(#p72:checked) .l [for=p71], body:has(#p73:checked) .l [for=p72], body:has(#p74:checked) .l [for=p73],
  body:has(#p75:checked) .l [for=p74], body:has(#p76:checked) .l [for=p75], body:has(#p77:checked) .l [for=p76],
  body:has(#p00:checked) .r [for=p01], body:has(#p01:checked) .r [for=p02], body:has(#p02:checked) .r [for=p03], body:has(#p03:checked) .r [for=p04],
  body:has(#p04:checked) .r [for=p05], body:has(#p05:checked) .r [for=p06], body:has(#p06:checked) .r [for=p07],
  body:has(#p10:checked) .r [for=p11], body:has(#p11:checked) .r [for=p12], body:has(#p12:checked) .r [for=p13], body:has(#p13:checked) .r [for=p14],
  body:has(#p14:checked) .r [for=p15], body:has(#p15:checked) .r [for=p16], body:has(#p16:checked) .r [for=p17],
  body:has(#p20:checked) .r [for=p21], body:has(#p21:checked) .r [for=p22], body:has(#p22:checked) .r [for=p23], body:has(#p23:checked) .r [for=p24],
  body:has(#p24:checked) .r [for=p25], body:has(#p25:checked) .r [for=p26], body:has(#p26:checked) .r [for=p27],
  body:has(#p30:checked) .r [for=p31], body:has(#p31:checked) .r [for=p32], body:has(#p32:checked) .r [for=p33], body:has(#p33:checked) .r [for=p34],
  body:has(#p34:checked) .r [for=p35], body:has(#p35:checked) .r [for=p36], body:has(#p36:checked) .r [for=p37],
  body:has(#p40:checked) .r [for=p41], body:has(#p41:checked) .r [for=p42], body:has(#p42:checked) .r [for=p43], body:has(#p43:checked) .r [for=p44],
  body:has(#p44:checked) .r [for=p45], body:has(#p45:checked) .r [for=p46], body:has(#p46:checked) .r [for=p47],
  body:has(#p50:checked) .r [for=p51], body:has(#p51:checked) .r [for=p52], body:has(#p52:checked) .r [for=p53], body:has(#p53:checked) .r [for=p54],
  body:has(#p54:checked) .r [for=p55], body:has(#p55:checked) .r [for=p56], body:has(#p56:checked) .r [for=p57],
  body:has(#p60:checked) .r [for=p61], body:has(#p61:checked) .r [for=p62], body:has(#p62:checked) .r [for=p63], body:has(#p63:checked) .r [for=p64],
  body:has(#p64:checked) .r [for=p65], body:has(#p65:checked) .r [for=p66], body:has(#p66:checked) .r [for=p67],
  body:has(#p70:checked) .r [for=p71], body:has(#p71:checked) .r [for=p72], body:has(#p72:checked) .r [for=p73], body:has(#p73:checked) .r [for=p74],
  body:has(#p74:checked) .r [for=p75], body:has(#p75:checked) .r [for=p76], body:has(#p76:checked) .r [for=p77] {
    pointer-events: auto;
  }

  /* grid neighbor highlights + clicks */
  body:has(#p00:checked) .c01 .move, body:has(#p00:checked) .c10 .move,
  body:has(#p01:checked) .c00 .move, body:has(#p01:checked) .c02 .move, body:has(#p01:checked) .c11 .move,
  body:has(#p02:checked) .c01 .move, body:has(#p02:checked) .c03 .move, body:has(#p02:checked) .c12 .move,
  body:has(#p03:checked) .c02 .move, body:has(#p03:checked) .c04 .move, body:has(#p03:checked) .c13 .move,
  body:has(#p04:checked) .c03 .move, body:has(#p04:checked) .c05 .move, body:has(#p04:checked) .c14 .move,
  body:has(#p05:checked) .c04 .move, body:has(#p05:checked) .c06 .move, body:has(#p05:checked) .c15 .move,
  body:has(#p06:checked) .c05 .move, body:has(#p06:checked) .c07 .move, body:has(#p06:checked) .c16 .move,
  body:has(#p07:checked) .c06 .move, body:has(#p07:checked) .c17 .move,
  body:has(#p10:checked) .c00 .move, body:has(#p10:checked) .c11 .move, body:has(#p10:checked) .c20 .move,
  body:has(#p11:checked) .c01 .move, body:has(#p11:checked) .c10 .move, body:has(#p11:checked) .c12 .move, body:has(#p11:checked) .c21 .move,
  body:has(#p12:checked) .c02 .move, body:has(#p12:checked) .c11 .move, body:has(#p12:checked) .c13 .move, body:has(#p12:checked) .c22 .move,
  body:has(#p13:checked) .c03 .move, body:has(#p13:checked) .c12 .move, body:has(#p13:checked) .c14 .move, body:has(#p13:checked) .c23 .move,
  body:has(#p14:checked) .c04 .move, body:has(#p14:checked) .c13 .move, body:has(#p14:checked) .c15 .move, body:has(#p14:checked) .c24 .move,
  body:has(#p15:checked) .c05 .move, body:has(#p15:checked) .c14 .move, body:has(#p15:checked) .c16 .move, body:has(#p15:checked) .c25 .move,
  body:has(#p16:checked) .c06 .move, body:has(#p16:checked) .c15 .move, body:has(#p16:checked) .c17 .move, body:has(#p16:checked) .c26 .move,
  body:has(#p17:checked) .c07 .move, body:has(#p17:checked) .c16 .move, body:has(#p17:checked) .c27 .move,
  body:has(#p20:checked) .c10 .move, body:has(#p20:checked) .c21 .move, body:has(#p20:checked) .c30 .move,
  body:has(#p21:checked) .c11 .move, body:has(#p21:checked) .c20 .move, body:has(#p21:checked) .c22 .move, body:has(#p21:checked) .c31 .move,
  body:has(#p22:checked) .c12 .move, body:has(#p22:checked) .c21 .move, body:has(#p22:checked) .c23 .move, body:has(#p22:checked) .c32 .move,
  body:has(#p23:checked) .c13 .move, body:has(#p23:checked) .c22 .move, body:has(#p23:checked) .c24 .move, body:has(#p23:checked) .c33 .move,
  body:has(#p24:checked) .c14 .move, body:has(#p24:checked) .c23 .move, body:has(#p24:checked) .c25 .move, body:has(#p24:checked) .c34 .move,
  body:has(#p25:checked) .c15 .move, body:has(#p25:checked) .c24 .move, body:has(#p25:checked) .c26 .move, body:has(#p25:checked) .c35 .move,
  body:has(#p26:checked) .c16 .move, body:has(#p26:checked) .c25 .move, body:has(#p26:checked) .c27 .move, body:has(#p26:checked) .c36 .move,
  body:has(#p27:checked) .c17 .move, body:has(#p27:checked) .c26 .move, body:has(#p27:checked) .c37 .move,
  body:has(#p30:checked) .c20 .move, body:has(#p30:checked) .c31 .move, body:has(#p30:checked) .c40 .move,
  body:has(#p31:checked) .c21 .move, body:has(#p31:checked) .c30 .move, body:has(#p31:checked) .c32 .move, body:has(#p31:checked) .c41 .move,
  body:has(#p32:checked) .c22 .move, body:has(#p32:checked) .c31 .move, body:has(#p32:checked) .c33 .move, body:has(#p32:checked) .c42 .move,
  body:has(#p33:checked) .c23 .move, body:has(#p33:checked) .c32 .move, body:has(#p33:checked) .c34 .move, body:has(#p33:checked) .c43 .move,
  body:has(#p34:checked) .c24 .move, body:has(#p34:checked) .c33 .move, body:has(#p34:checked) .c35 .move, body:has(#p34:checked) .c44 .move,
  body:has(#p35:checked) .c25 .move, body:has(#p35:checked) .c34 .move, body:has(#p35:checked) .c36 .move, body:has(#p35:checked) .c45 .move,
  body:has(#p36:checked) .c26 .move, body:has(#p36:checked) .c35 .move, body:has(#p36:checked) .c37 .move, body:has(#p36:checked) .c46 .move,
  body:has(#p37:checked) .c27 .move, body:has(#p37:checked) .c36 .move, body:has(#p37:checked) .c47 .move,
  body:has(#p40:checked) .c30 .move, body:has(#p40:checked) .c41 .move, body:has(#p40:checked) .c50 .move,
  body:has(#p41:checked) .c31 .move, body:has(#p41:checked) .c40 .move, body:has(#p41:checked) .c42 .move, body:has(#p41:checked) .c51 .move,
  body:has(#p42:checked) .c32 .move, body:has(#p42:checked) .c41 .move, body:has(#p42:checked) .c43 .move, body:has(#p42:checked) .c52 .move,
  body:has(#p43:checked) .c33 .move, body:has(#p43:checked) .c42 .move, body:has(#p43:checked) .c44 .move, body:has(#p43:checked) .c53 .move,
  body:has(#p44:checked) .c34 .move, body:has(#p44:checked) .c43 .move, body:has(#p44:checked) .c45 .move, body:has(#p44:checked) .c54 .move,
  body:has(#p45:checked) .c35 .move, body:has(#p45:checked) .c44 .move, body:has(#p45:checked) .c46 .move, body:has(#p45:checked) .c55 .move,
  body:has(#p46:checked) .c36 .move, body:has(#p46:checked) .c45 .move, body:has(#p46:checked) .c47 .move, body:has(#p46:checked) .c56 .move,
  body:has(#p47:checked) .c37 .move, body:has(#p47:checked) .c46 .move, body:has(#p47:checked) .c57 .move,
  body:has(#p50:checked) .c40 .move, body:has(#p50:checked) .c51 .move, body:has(#p50:checked) .c60 .move,
  body:has(#p51:checked) .c41 .move, body:has(#p51:checked) .c50 .move, body:has(#p51:checked) .c52 .move, body:has(#p51:checked) .c61 .move,
  body:has(#p52:checked) .c42 .move, body:has(#p52:checked) .c51 .move, body:has(#p52:checked) .c53 .move, body:has(#p52:checked) .c62 .move,
  body:has(#p53:checked) .c43 .move, body:has(#p53:checked) .c52 .move, body:has(#p53:checked) .c54 .move, body:has(#p53:checked) .c63 .move,
  body:has(#p54:checked) .c44 .move, body:has(#p54:checked) .c53 .move, body:has(#p54:checked) .c55 .move, body:has(#p54:checked) .c64 .move,
  body:has(#p55:checked) .c45 .move, body:has(#p55:checked) .c54 .move, body:has(#p55:checked) .c56 .move, body:has(#p55:checked) .c65 .move,
  body:has(#p56:checked) .c46 .move, body:has(#p56:checked) .c55 .move, body:has(#p56:checked) .c57 .move, body:has(#p56:checked) .c66 .move,
  body:has(#p57:checked) .c47 .move, body:has(#p57:checked) .c56 .move, body:has(#p57:checked) .c67 .move,
  body:has(#p60:checked) .c50 .move, body:has(#p60:checked) .c61 .move, body:has(#p60:checked) .c70 .move,
  body:has(#p61:checked) .c51 .move, body:has(#p61:checked) .c60 .move, body:has(#p61:checked) .c62 .move, body:has(#p61:checked) .c71 .move,
  body:has(#p62:checked) .c52 .move, body:has(#p62:checked) .c61 .move, body:has(#p62:checked) .c63 .move, body:has(#p62:checked) .c72 .move,
  body:has(#p63:checked) .c53 .move, body:has(#p63:checked) .c62 .move, body:has(#p63:checked) .c64 .move, body:has(#p63:checked) .c73 .move,
  body:has(#p64:checked) .c54 .move, body:has(#p64:checked) .c63 .move, body:has(#p64:checked) .c65 .move, body:has(#p64:checked) .c74 .move,
  body:has(#p65:checked) .c55 .move, body:has(#p65:checked) .c64 .move, body:has(#p65:checked) .c66 .move, body:has(#p65:checked) .c75 .move,
  body:has(#p66:checked) .c56 .move, body:has(#p66:checked) .c65 .move, body:has(#p66:checked) .c67 .move, body:has(#p66:checked) .c76 .move,
  body:has(#p67:checked) .c57 .move, body:has(#p67:checked) .c66 .move, body:has(#p67:checked) .c77 .move,
  body:has(#p70:checked) .c60 .move, body:has(#p70:checked) .c71 .move,
  body:has(#p71:checked) .c61 .move, body:has(#p71:checked) .c70 .move, body:has(#p71:checked) .c72 .move,
  body:has(#p72:checked) .c62 .move, body:has(#p72:checked) .c71 .move, body:has(#p72:checked) .c73 .move,
  body:has(#p73:checked) .c63 .move, body:has(#p73:checked) .c72 .move, body:has(#p73:checked) .c74 .move,
  body:has(#p74:checked) .c64 .move, body:has(#p74:checked) .c73 .move, body:has(#p74:checked) .c75 .move,
  body:has(#p75:checked) .c65 .move, body:has(#p75:checked) .c74 .move, body:has(#p75:checked) .c76 .move,
  body:has(#p76:checked) .c66 .move, body:has(#p76:checked) .c75 .move, body:has(#p76:checked) .c77 .move,
  body:has(#p77:checked) .c67 .move, body:has(#p77:checked) .c76 .move { pointer-events: auto; cursor: pointer; background: rgba(57,255,20,.10); box-shadow: inset 0 0 0 2px rgba(57,255,20,.5); }

  .foot {
    margin-top: 14px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: var(--muted);
    font-size: 17px;
  }
  .foot a {
    color: var(--lime);
    text-decoration: none;
    border-bottom: 1px dotted var(--lime-dim);
  }

  /* start + win overlays */
  .overlay {
    position: absolute;
    inset: 0;
    z-index: 12;
    background: rgba(4,10,5,.88);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 24px;
    gap: 12px;
  }
  .overlay h2 {
    font-family: "Press Start 2P", monospace;
    font-size: 16px;
    color: var(--lime);
    line-height: 1.5;
  }
  .overlay p { font-size: 20px; color: #9ccc9c; max-width: 28ch; }
  .play, .again {
    display: inline-flex;
    margin-top: 8px;
    background: var(--lime);
    color: #062006;
    font-family: "Press Start 2P", monospace;
    font-size: 11px;
    padding: 14px 18px;
    border-radius: 10px;
    cursor: pointer;
    text-decoration: none;
    box-shadow: 0 4px 0 #148a22;
  }
  .play:hover, .again:hover { filter: brightness(1.08); }

  body:has(#go:checked) .intro { display: none; }
  .win { display: none; }
  body:has(#f1:checked):has(#f2:checked):has(#f3:checked):has(#f4:checked):has(#f5:checked):has(#f6:checked) .win {
    display: flex;
  }

  @media (max-width: 520px) {
    .brand { font-size: 11px; }
    .dpad { grid-template-columns: 50px 50px 50px; grid-template-rows: 44px 44px 44px; }
  }
</style>
</head>
<body>
<div class="controls">
  <input type="checkbox" id="go">
  <input type="radio" name="pos" id="p00">
  <input type="radio" name="pos" id="p01">
  <input type="radio" name="pos" id="p02">
  <input type="radio" name="pos" id="p03">
  <input type="radio" name="pos" id="p04">
  <input type="radio" name="pos" id="p05">
  <input type="radio" name="pos" id="p06">
  <input type="radio" name="pos" id="p07">
  <input type="radio" name="pos" id="p10">
  <input type="radio" name="pos" id="p11">
  <input type="radio" name="pos" id="p12">
  <input type="radio" name="pos" id="p13">
  <input type="radio" name="pos" id="p14">
  <input type="radio" name="pos" id="p15">
  <input type="radio" name="pos" id="p16">
  <input type="radio" name="pos" id="p17">
  <input type="radio" name="pos" id="p20">
  <input type="radio" name="pos" id="p21">
  <input type="radio" name="pos" id="p22">
  <input type="radio" name="pos" id="p23">
  <input type="radio" name="pos" id="p24">
  <input type="radio" name="pos" id="p25">
  <input type="radio" name="pos" id="p26">
  <input type="radio" name="pos" id="p27">
  <input type="radio" name="pos" id="p30">
  <input type="radio" name="pos" id="p31">
  <input type="radio" name="pos" id="p32">
  <input type="radio" name="pos" id="p33">
  <input type="radio" name="pos" id="p34">
  <input type="radio" name="pos" id="p35">
  <input type="radio" name="pos" id="p36">
  <input type="radio" name="pos" id="p37">
  <input type="radio" name="pos" id="p40">
  <input type="radio" name="pos" id="p41">
  <input type="radio" name="pos" id="p42">
  <input type="radio" name="pos" id="p43" checked>
  <input type="radio" name="pos" id="p44">
  <input type="radio" name="pos" id="p45">
  <input type="radio" name="pos" id="p46">
  <input type="radio" name="pos" id="p47">
  <input type="radio" name="pos" id="p50">
  <input type="radio" name="pos" id="p51">
  <input type="radio" name="pos" id="p52">
  <input type="radio" name="pos" id="p53">
  <input type="radio" name="pos" id="p54">
  <input type="radio" name="pos" id="p55">
  <input type="radio" name="pos" id="p56">
  <input type="radio" name="pos" id="p57">
  <input type="radio" name="pos" id="p60">
  <input type="radio" name="pos" id="p61">
  <input type="radio" name="pos" id="p62">
  <input type="radio" name="pos" id="p63">
  <input type="radio" name="pos" id="p64">
  <input type="radio" name="pos" id="p65">
  <input type="radio" name="pos" id="p66">
  <input type="radio" name="pos" id="p67">
  <input type="radio" name="pos" id="p70">
  <input type="radio" name="pos" id="p71">
  <input type="radio" name="pos" id="p72">
  <input type="radio" name="pos" id="p73">
  <input type="radio" name="pos" id="p74">
  <input type="radio" name="pos" id="p75">
  <input type="radio" name="pos" id="p76">
  <input type="radio" name="pos" id="p77">
  <input type="checkbox" id="f1">
  <input type="checkbox" id="f2">
  <input type="checkbox" id="f3">
  <input type="checkbox" id="f4">
  <input type="checkbox" id="f5">
  <input type="checkbox" id="f6">
</div>

<div class="cabinet">
  <header class="top">
    <div class="brand">CSS SNAKE<small>0% JavaScript · 100% :has()</small></div>
    <div class="hud">
      <span>SCORE</span><b class="score"></b>
    </div>
    <div class="hud">
      <span>LEN</span><b class="len"></b>
    </div>
  </header>

  <div class="screen">
    <div class="board-wrap">
      <div class="board">
        <div class="cell c00"><label class="move" for="p00"></label></div>
        <div class="cell c01"><label class="move" for="p01"></label></div>
        <div class="cell c02"><label class="move" for="p02"></label></div>
        <div class="cell c03"><label class="move" for="p03"></label></div>
        <div class="cell c04"><label class="move" for="p04"></label></div>
        <div class="cell c05"><label class="move" for="p05"></label></div>
        <div class="cell c06"><label class="move" for="p06"></label></div>
        <div class="cell c07"><label class="move" for="p07"></label><span class="apple a2"><i></i></span></div>
        <div class="cell c10"><label class="move" for="p10"></label></div>
        <div class="cell c11"><label class="move" for="p11"></label></div>
        <div class="cell c12"><label class="move" for="p12"></label></div>
        <div class="cell c13"><label class="move" for="p13"></label></div>
        <div class="cell c14"><label class="move" for="p14"></label></div>
        <div class="cell c15"><label class="move" for="p15"></label></div>
        <div class="cell c16"><label class="move" for="p16"></label></div>
        <div class="cell c17"><label class="move" for="p17"></label></div>
        <div class="cell c20"><label class="move" for="p20"></label></div>
        <div class="cell c21"><label class="move" for="p21"></label></div>
        <div class="cell c22"><label class="move" for="p22"></label></div>
        <div class="cell c23"><label class="move" for="p23"></label></div>
        <div class="cell c24"><label class="move" for="p24"></label></div>
        <div class="cell c25"><label class="move" for="p25"></label></div>
        <div class="cell c26"><label class="move" for="p26"></label><span class="apple a1"><i></i></span></div>
        <div class="cell c27"><label class="move" for="p27"></label></div>
        <div class="cell c30"><label class="move" for="p30"></label></div>
        <div class="cell c31"><label class="move" for="p31"></label></div>
        <div class="cell c32"><label class="move" for="p32"></label></div>
        <div class="cell c33"><label class="move" for="p33"></label><span class="apple a6"><i></i></span></div>
        <div class="cell c34"><label class="move" for="p34"></label></div>
        <div class="cell c35"><label class="move" for="p35"></label></div>
        <div class="cell c36"><label class="move" for="p36"></label></div>
        <div class="cell c37"><label class="move" for="p37"></label></div>
        <div class="cell c40"><label class="move" for="p40"></label><span class="apple a5"><i></i></span></div>
        <div class="cell c41"><label class="move" for="p41"></label></div>
        <div class="cell c42"><label class="move" for="p42"></label></div>
        <div class="cell c43"><label class="move" for="p43"></label></div>
        <div class="cell c44"><label class="move" for="p44"></label></div>
        <div class="cell c45"><label class="move" for="p45"></label></div>
        <div class="cell c46"><label class="move" for="p46"></label></div>
        <div class="cell c47"><label class="move" for="p47"></label></div>
        <div class="cell c50"><label class="move" for="p50"></label></div>
        <div class="cell c51"><label class="move" for="p51"></label></div>
        <div class="cell c52"><label class="move" for="p52"></label></div>
        <div class="cell c53"><label class="move" for="p53"></label></div>
        <div class="cell c54"><label class="move" for="p54"></label></div>
        <div class="cell c55"><label class="move" for="p55"></label></div>
        <div class="cell c56"><label class="move" for="p56"></label></div>
        <div class="cell c57"><label class="move" for="p57"></label><span class="apple a3"><i></i></span></div>
        <div class="cell c60"><label class="move" for="p60"></label></div>
        <div class="cell c61"><label class="move" for="p61"></label></div>
        <div class="cell c62"><label class="move" for="p62"></label></div>
        <div class="cell c63"><label class="move" for="p63"></label></div>
        <div class="cell c64"><label class="move" for="p64"></label></div>
        <div class="cell c65"><label class="move" for="p65"></label></div>
        <div class="cell c66"><label class="move" for="p66"></label></div>
        <div class="cell c67"><label class="move" for="p67"></label></div>
        <div class="cell c70"><label class="move" for="p70"></label></div>
        <div class="cell c71"><label class="move" for="p71"></label><span class="apple a4"><i></i></span></div>
        <div class="cell c72"><label class="move" for="p72"></label></div>
        <div class="cell c73"><label class="move" for="p73"></label></div>
        <div class="cell c74"><label class="move" for="p74"></label></div>
        <div class="cell c75"><label class="move" for="p75"></label></div>
        <div class="cell c76"><label class="move" for="p76"></label></div>
        <div class="cell c77"><label class="move" for="p77"></label></div>
      </div>

      <div class="snake">
        <div class="head">
          <span class="eye l"></span>
          <span class="eye r"></span>
          <span class="tongue"></span>
        </div>
      </div>
    </div>

    <div class="overlay intro">
      <h2>CSS SNAKE</h2>
      <p>No JavaScript, so this slither is turn-based. Move one cell at a time, eat all 6 apples, grow a ridiculous tail.</p>
      <label class="play" for="go">PRESS START</label>
    </div>
    <div class="overlay win">
      <h2>YOU WIN</h2>
      <p>Six apples. Zero scripts. You are a CSS creature now.</p>
      <a class="again" href="">PLAY AGAIN</a>
    </div>
  </div>

  <div class="below">
    <p class="hint"></p>
    <label class="eat e1" for="f1">EAT</label>
    <label class="eat e2" for="f2">EAT</label>
    <label class="eat e3" for="f3">EAT</label>
    <label class="eat e4" for="f4">EAT</label>
    <label class="eat e5" for="f5">EAT</label>
    <label class="eat e6" for="f6">EAT</label>
  </div>

  <div class="dpad">
    <div class="key u">▲
      <label for="p00"></label><label for="p01"></label><label for="p02"></label><label for="p03"></label><label for="p04"></label><label for="p05"></label><label for="p06"></label><label for="p07"></label>
      <label for="p10"></label><label for="p11"></label><label for="p12"></label><label for="p13"></label><label for="p14"></label><label for="p15"></label><label for="p16"></label><label for="p17"></label>
      <label for="p20"></label><label for="p21"></label><label for="p22"></label><label for="p23"></label><label for="p24"></label><label for="p25"></label><label for="p26"></label><label for="p27"></label>
      <label for="p30"></label><label for="p31"></label><label for="p32"></label><label for="p33"></label><label for="p34"></label><label for="p35"></label><label for="p36"></label><label for="p37"></label>
      <label for="p40"></label><label for="p41"></label><label for="p42"></label><label for="p43"></label><label for="p44"></label><label for="p45"></label><label for="p46"></label><label for="p47"></label>
      <label for="p50"></label><label for="p51"></label><label for="p52"></label><label for="p53"></label><label for="p54"></label><label for="p55"></label><label for="p56"></label><label for="p57"></label>
      <label for="p60"></label><label for="p61"></label><label for="p62"></label><label for="p63"></label><label for="p64"></label><label for="p65"></label><label for="p66"></label><label for="p67"></label>
    </div>
    <div class="key l">◀
      <label for="p00"></label><label for="p01"></label><label for="p02"></label><label for="p03"></label><label for="p04"></label><label for="p05"></label><label for="p06"></label>
      <label for="p10"></label><label for="p11"></label><label for="p12"></label><label for="p13"></label><label for="p14"></label><label for="p15"></label><label for="p16"></label>
      <label for="p20"></label><label for="p21"></label><label for="p22"></label><label for="p23"></label><label for="p24"></label><label for="p25"></label><label for="p26"></label>
      <label for="p30"></label><label for="p31"></label><label for="p32"></label><label for="p33"></label><label for="p34"></label><label for="p35"></label><label for="p36"></label>
      <label for="p40"></label><label for="p41"></label><label for="p42"></label><label for="p43"></label><label for="p44"></label><label for="p45"></label><label for="p46"></label>
      <label for="p50"></label><label for="p51"></label><label for="p52"></label><label for="p53"></label><label for="p54"></label><label for="p55"></label><label for="p56"></label>
      <label for="p60"></label><label for="p61"></label><label for="p62"></label><label for="p63"></label><label for="p64"></label><label for="p65"></label><label for="p66"></label>
      <label for="p70"></label><label for="p71"></label><label for="p72"></label><label for="p73"></label><label for="p74"></label><label for="p75"></label><label for="p76"></label>
    </div>
    <div class="key c">CSS</div>
    <div class="key r">▶
      <label for="p01"></label><label for="p02"></label><label for="p03"></label><label for="p04"></label><label for="p05"></label><label for="p06"></label><label for="p07"></label>
      <label for="p11"></label><label for="p12"></label><label for="p13"></label><label for="p14"></label><label for="p15"></label><label for="p16"></label><label for="p17"></label>
      <label for="p21"></label><label for="p22"></label><label for="p23"></label><label for="p24"></label><label for="p25"></label><label for="p26"></label><label for="p27"></label>
      <label for="p31"></label><label for="p32"></label><label for="p33"></label><label for="p34"></label><label for="p35"></label><label for="p36"></label><label for="p37"></label>
      <label for="p41"></label><label for="p42"></label><label for="p43"></label><label for="p44"></label><label for="p45"></label><label for="p46"></label><label for="p47"></label>
      <label for="p51"></label><label for="p52"></label><label for="p53"></label><label for="p54"></label><label for="p55"></label><label for="p56"></label><label for="p57"></label>
      <label for="p61"></label><label for="p62"></label><label for="p63"></label><label for="p64"></label><label for="p65"></label><label for="p66"></label><label for="p67"></label>
      <label for="p71"></label><label for="p72"></label><label for="p73"></label><label for="p74"></label><label for="p75"></label><label for="p76"></label><label for="p77"></label>
    </div>
    <div class="key d">▼
      <label for="p10"></label><label for="p11"></label><label for="p12"></label><label for="p13"></label><label for="p14"></label><label for="p15"></label><label for="p16"></label><label for="p17"></label>
      <label for="p20"></label><label for="p21"></label><label for="p22"></label><label for="p23"></label><label for="p24"></label><label for="p25"></label><label for="p26"></label><label for="p27"></label>
      <label for="p30"></label><label for="p31"></label><label for="p32"></label><label for="p33"></label><label for="p34"></label><label for="p35"></label><label for="p36"></label><label for="p37"></label>
      <label for="p40"></label><label for="p41"></label><label for="p42"></label><label for="p43"></label><label for="p44"></label><label for="p45"></label><label for="p46"></label><label for="p47"></label>
      <label for="p50"></label><label for="p51"></label><label for="p52"></label><label for="p53"></label><label for="p54"></label><label for="p55"></label><label for="p56"></label><label for="p57"></label>
      <label for="p60"></label><label for="p61"></label><label for="p62"></label><label for="p63"></label><label for="p64"></label><label for="p65"></label><label for="p66"></label><label for="p67"></label>
      <label for="p70"></label><label for="p71"></label><label for="p72"></label><label for="p73"></label><label for="p74"></label><label for="p75"></label><label for="p76"></label><label for="p77"></label>
    </div>
  </div>

  <div class="foot">
    <span>Walls stop you. CSS can't run a game loop.</span>
    <a href="">Reset</a>
  </div>
</div>
</body>
</html>
```

Save that as `snake.html` and open it in a browser.

**How it works (and why it’s turn-based):** CSS has no game loop, no memory of a sliding tail, and no way to uncheck the last body cell. So this version uses radios for the head, `:has()` for legal neighbor moves, and checkboxes for eaten apples.

**Play:**
1. Press **START**
2. Click a glowing neighbor, or the D-pad
3. Land on an apple → hit **EAT** to grow
4. Collect all 6 to win

Needs a modern browser with `:has()` (Chrome, Firefox, Safari, Edge).
