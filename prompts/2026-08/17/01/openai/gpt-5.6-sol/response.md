Below is a complete single-file, JavaScript-free CSS Snake game. Because CSS cannot implement a continuously controlled game loop, this version is turn-based: click or tap each apple to move and grow the snake.

Save it as `snake.html` and open it in a modern browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CSS-Only Snake</title>

<style>
  * {
    box-sizing: border-box;
  }

  :root {
    color-scheme: dark;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }

  body {
    min-height: 100vh;
    margin: 0;
    display: grid;
    place-items: center;
    padding: 20px;
    overflow-x: hidden;
    background:
      radial-gradient(circle at top, #263b35 0, #111815 45%, #070a09 100%);
    color: #f4fff8;
  }

  form {
    width: min(92vw, 620px);
  }

  /* Keep the game-state checkboxes functional but invisible. */
  .state {
    position: fixed;
    left: -9999px;
    opacity: 0;
  }

  .game {
    padding: clamp(16px, 4vw, 30px);
    border: 1px solid #ffffff1f;
    border-radius: 28px;
    background: #101713e8;
    box-shadow:
      0 30px 80px #0009,
      inset 0 1px #ffffff14;
    backdrop-filter: blur(12px);
  }

  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    margin-bottom: 18px;
  }

  h1 {
    margin: 0;
    color: #9cff8a;
    font-size: clamp(1.35rem, 5vw, 2.2rem);
    line-height: 1;
    letter-spacing: -0.04em;
  }

  h1 span {
    display: block;
    margin-top: 7px;
    color: #a8b9ae;
    font-size: 0.72rem;
    font-weight: 500;
    letter-spacing: 0.04em;
  }

  .score {
    min-width: 92px;
    padding: 9px 13px;
    border: 1px solid #9cff8a35;
    border-radius: 14px;
    background: #9cff8a10;
    color: #baffae;
    font-weight: 800;
    text-align: center;
  }

  .score::after {
    content: "0 / 8";
  }

  /* Score states */
  #apple1:checked ~ .game .score::after { content: "1 / 8"; }
  #apple2:checked ~ .game .score::after { content: "2 / 8"; }
  #apple3:checked ~ .game .score::after { content: "3 / 8"; }
  #apple4:checked ~ .game .score::after { content: "4 / 8"; }
  #apple5:checked ~ .game .score::after { content: "5 / 8"; }
  #apple6:checked ~ .game .score::after { content: "6 / 8"; }
  #apple7:checked ~ .game .score::after { content: "7 / 8"; }
  #apple8:checked ~ .game .score::after { content: "8 / 8"; }

  .board {
    position: relative;
    width: 100%;
    aspect-ratio: 1;
    overflow: hidden;
    border: clamp(7px, 2vw, 12px) solid #26382f;
    border-radius: 20px;
    background-color: #132019;
    background-image:
      linear-gradient(#b5ffb20a 1px, transparent 1px),
      linear-gradient(90deg, #b5ffb20a 1px, transparent 1px),
      radial-gradient(circle at center, #1b2b22, #101912);
    background-size:
      10% 10%,
      10% 10%,
      100% 100%;
    box-shadow:
      inset 0 0 45px #0008,
      0 12px 30px #0006;
  }

  /*
    Each board object occupies one cell in a 10 × 10 grid.
    --x and --y use one-based grid coordinates.
  */
  .cell {
    --x: 1;
    --y: 1;

    position: absolute;
    left: calc((var(--x) - 1) * 10%);
    top: calc((var(--y) - 1) * 10%);
    width: 10%;
    aspect-ratio: 1;
    padding: 0.7%;
  }

  .snake-part::before,
  .head::before {
    content: "";
    display: block;
    width: 100%;
    height: 100%;
    border-radius: 30%;
    background:
      linear-gradient(145deg, #a7ff86, #35c85f 60%, #169744);
    box-shadow:
      inset 3px 3px 5px #ffffff45,
      inset -4px -4px 7px #075e2c80,
      0 3px 7px #0006;
  }

  .snake-part {
    display: none;
    transform: scale(0.86);
  }

  .tail {
    display: block;
    --x: 3;
    --y: 5;
  }

  .tail::before {
    border-radius: 55% 30% 30% 55%;
  }

  .head {
    --x: 4;
    --y: 5;

    z-index: 5;
    transition:
      left 220ms ease,
      top 220ms ease,
      transform 160ms ease;
  }

  .head::before {
    border-radius: 35%;
    background:
      radial-gradient(circle at 68% 28%, #092416 0 5%, white 6% 10%, transparent 11%),
      radial-gradient(circle at 68% 72%, #092416 0 5%, white 6% 10%, transparent 11%),
      linear-gradient(145deg, #c6ff91, #45dd64 60%, #18a54c);
  }

  /* Move the head as apples are collected. */
  #apple1:checked ~ .game .head { --x: 5; --y: 5; }
  #apple2:checked ~ .game .head { --x: 6; --y: 5; }
  #apple3:checked ~ .game .head { --x: 7; --y: 5; }
  #apple4:checked ~ .game .head { --x: 7; --y: 4; }
  #apple5:checked ~ .game .head { --x: 7; --y: 3; }
  #apple6:checked ~ .game .head { --x: 6; --y: 3; }
  #apple7:checked ~ .game .head { --x: 5; --y: 3; }
  #apple8:checked ~ .game .head { --x: 5; --y: 2; }

  /* Grow the body one segment at a time. */
  #apple1:checked ~ .game .segment-1,
  #apple2:checked ~ .game .segment-2,
  #apple3:checked ~ .game .segment-3,
  #apple4:checked ~ .game .segment-4,
  #apple5:checked ~ .game .segment-5,
  #apple6:checked ~ .game .segment-6,
  #apple7:checked ~ .game .segment-7,
  #apple8:checked ~ .game .segment-8 {
    display: block;
    animation: grow 220ms ease-out;
  }

  @keyframes grow {
    from {
      opacity: 0;
      transform: scale(0);
    }
    to {
      opacity: 1;
      transform: scale(0.86);
    }
  }

  /* Apples are labels connected to the hidden checkboxes. */
  .apple {
    z-index: 6;
    display: none;
    place-items: center;
    padding: 0;
    border: 0;
    cursor: pointer;
    user-select: none;
    filter: drop-shadow(0 4px 5px #0008);
    animation: apple-pulse 850ms ease-in-out infinite alternate;
  }

  .apple::before {
    content: "🍎";
    font-size: clamp(20px, 6vw, 42px);
    line-height: 1;
    transition: transform 130ms ease;
  }

  .apple:hover::before {
    transform: scale(1.25) rotate(-8deg);
  }

  .apple:active::before {
    transform: scale(0.85);
  }

  @keyframes apple-pulse {
    to {
      transform: scale(1.12);
      filter: drop-shadow(0 0 9px #ff594a99);
    }
  }

  /* Only the next apple in the sequence is visible. */
  #apple1:not(:checked) ~ .game .target-1 {
    display: grid;
  }

  #apple1:checked ~ #apple2:not(:checked) ~ .game .target-2 {
    display: grid;
  }

  #apple2:checked ~ #apple3:not(:checked) ~ .game .target-3 {
    display: grid;
  }

  #apple3:checked ~ #apple4:not(:checked) ~ .game .target-4 {
    display: grid;
  }

  #apple4:checked ~ #apple5:not(:checked) ~ .game .target-5 {
    display: grid;
  }

  #apple5:checked ~ #apple6:not(:checked) ~ .game .target-6 {
    display: grid;
  }

  #apple6:checked ~ #apple7:not(:checked) ~ .game .target-7 {
    display: grid;
  }

  #apple7:checked ~ #apple8:not(:checked) ~ .game .target-8 {
    display: grid;
  }

  .instructions {
    margin: 16px 0 0;
    color: #a9b9af;
    font-size: 0.9rem;
    text-align: center;
  }

  .win {
    position: absolute;
    inset: 0;
    z-index: 20;
    display: grid;
    place-items: center;
    padding: 25px;
    background: #07110bdd;
    opacity: 0;
    visibility: hidden;
    transform: scale(1.08);
    transition: 300ms ease;
    backdrop-filter: blur(8px);
  }

  #apple8:checked ~ .game .win {
    opacity: 1;
    visibility: visible;
    transform: scale(1);
  }

  .win-card {
    width: min(100%, 330px);
    padding: 28px;
    border: 1px solid #9cff8a44;
    border-radius: 24px;
    background: #15241a;
    box-shadow: 0 20px 50px #0009;
    text-align: center;
  }

  .trophy {
    margin-bottom: 8px;
    font-size: 3.4rem;
  }

  .win h2 {
    margin: 0;
    color: #aaff91;
    font-size: 2rem;
  }

  .win p {
    margin: 8px 0 20px;
    color: #b4c3b9;
  }

  button {
    width: 100%;
    padding: 12px 18px;
    border: 0;
    border-radius: 13px;
    background: linear-gradient(135deg, #a6ff84, #42d663);
    color: #092412;
    font: inherit;
    font-weight: 900;
    cursor: pointer;
    box-shadow: 0 7px 20px #45df6540;
  }

  button:hover {
    filter: brightness(1.08);
  }

  button:active {
    transform: translateY(1px);
  }

  @media (prefers-reduced-motion: reduce) {
    *,
    *::before {
      animation: none !important;
      transition: none !important;
    }
  }
</style>
</head>

<body>
<form>
  <!-- Each checkbox represents one turn of the game. -->
  <input class="state" type="checkbox" id="apple1">
  <input class="state" type="checkbox" id="apple2">
  <input class="state" type="checkbox" id="apple3">
  <input class="state" type="checkbox" id="apple4">
  <input class="state" type="checkbox" id="apple5">
  <input class="state" type="checkbox" id="apple6">
  <input class="state" type="checkbox" id="apple7">
  <input class="state" type="checkbox" id="apple8">

  <main class="game">
    <header>
      <h1>
        CSS Snake
        <span>No JavaScript required</span>
      </h1>
      <div class="score" aria-label="Score: "></div>
    </header>

    <div class="board">
      <!-- Permanent starting tail -->
      <div class="cell snake-part tail"></div>

      <!-- Body segments appear as the snake grows. -->
      <div class="cell snake-part segment-1" style="--x:4; --y:5"></div>
      <div class="cell snake-part segment-2" style="--x:5; --y:5"></div>
      <div class="cell snake-part segment-3" style="--x:6; --y:5"></div>
      <div class="cell snake-part segment-4" style="--x:7; --y:5"></div>
      <div class="cell snake-part segment-5" style="--x:7; --y:4"></div>
      <div class="cell snake-part segment-6" style="--x:7; --y:3"></div>
      <div class="cell snake-part segment-7" style="--x:6; --y:3"></div>
      <div class="cell snake-part segment-8" style="--x:5; --y:3"></div>

      <!-- Snake head -->
      <div class="cell head" aria-hidden="true"></div>

      <!-- Clickable apples -->
      <label class="cell apple target-1"
             style="--x:5; --y:5"
             for="apple1"
             title="Eat apple 1"></label>

      <label class="cell apple target-2"
             style="--x:6; --y:5"
             for="apple2"
             title="Eat apple 2"></label>

      <label class="cell apple target-3"
             style="--x:7; --y:5"
             for="apple3"
             title="Eat apple 3"></label>

      <label class="cell apple target-4"
             style="--x:7; --y:4"
             for="apple4"
             title="Eat apple 4"></label>

      <label class="cell apple target-5"
             style="--x:7; --y:3"
             for="apple5"
             title="Eat apple 5"></label>

      <label class="cell apple target-6"
             style="--x:6; --y:3"
             for="apple6"
             title="Eat apple 6"></label>

      <label class="cell apple target-7"
             style="--x:5; --y:3"
             for="apple7"
             title="Eat apple 7"></label>

      <label class="cell apple target-8"
             style="--x:5; --y:2"
             for="apple8"
             title="Eat apple 8"></label>

      <section class="win">
        <div class="win-card">
          <div class="trophy">🏆</div>
          <h2>You win!</h2>
          <p>The snake ate all eight apples.</p>
          <button type="reset">Play again</button>
        </div>
      </section>
    </div>

    <p class="instructions">
      Click or tap each apple to move and grow the snake.
    </p>
  </main>
</form>
</body>
</html>
```
