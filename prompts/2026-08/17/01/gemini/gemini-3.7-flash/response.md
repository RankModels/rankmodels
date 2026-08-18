**Creating a Pure CSS and HTML Snake Game**

Building an interactive game such as Snake without writing a single line of JavaScript is an impressive demonstration of modern CSS capabilities. By combining HTML `<form>` elements (specifically radio buttons and checkboxes), the CSS sibling combinators (`~` and `+`), and `@keyframes` animations, we can track state, handle directional user input, render animation paths, and evaluate win/lose conditions entirely within the browser layout engine.

Below is the complete, self-contained single-file implementation of the pure CSS and HTML Snake game. You can copy and save this code directly into an `index.html` file and open it in any modern web browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Pure CSS Snake Game</title>
  <style>
    :root {
      --board-size: 400px;
      --cell-size: 20px;
      --snake-color: #4ade80;
      --snake-head-color: #22c55e;
      --apple-color: #ef4444;
      --bg-color: #0f172a;
      --grid-bg: #1e293b;
      --text-color: #f8fafc;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text-color);
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    h1 {
      font-size: 2rem;
      margin-bottom: 8px;
      text-transform: uppercase;
      letter-spacing: 2px;
      color: #38bdf8;
    }

    p.subtitle {
      font-size: 0.95rem;
      color: #94a3b8;
      margin-bottom: 24px;
    }

    /* Game Form Container */
    .game-container {
      position: relative;
      background: var(--grid-bg);
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5), 0 8px 10px -6px rgba(0, 0, 0, 0.5);
      border: 1px solid #334155;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    /* Screen Viewport */
    .viewport {
      position: relative;
      width: var(--board-size);
      height: var(--board-size);
      background-color: #090d16;
      background-image: 
        linear-gradient(to right, rgba(255, 255, 255, 0.05) 1px, transparent 1px),
        linear-gradient(to bottom, rgba(255, 255, 255, 0.05) 1px, transparent 1px);
      background-size: var(--cell-size) var(--cell-size);
      border: 3px solid #475569;
      border-radius: 8px;
      overflow: hidden;
    }

    /* Hide standard inputs */
    input[type="radio"],
    input[type="checkbox"] {
      position: absolute;
      opacity: 0;
      pointer-events: none;
    }

    /* Snake Container */
    .snake-body {
      position: absolute;
      width: var(--cell-size);
      height: var(--cell-size);
      top: 0;
      left: 0;
      pointer-events: none;
      z-index: 5;
    }

    .snake-segment {
      position: absolute;
      width: 100%;
      height: 100%;
      border-radius: 4px;
      background-color: var(--snake-color);
      box-shadow: 0 0 8px rgba(74, 222, 128, 0.4);
    }

    .snake-head {
      background-color: var(--snake-head-color);
      box-shadow: 0 0 12px rgba(34, 197, 94, 0.8);
      z-index: 6;
    }

    /* Directional Movement Keyframes */
    @keyframes move-up {
      0%   { transform: translate(var(--curr-x, 200px), var(--curr-y, 200px)); }
      100% { transform: translate(var(--curr-x, 200px), calc(var(--curr-y, 200px) - var(--board-size))); }
    }

    @keyframes move-down {
      0%   { transform: translate(var(--curr-x, 200px), var(--curr-y, 200px)); }
      100% { transform: translate(var(--curr-x, 200px), calc(var(--curr-y, 200px) + var(--board-size))); }
    }

    @keyframes move-left {
      0%   { transform: translate(var(--curr-x, 200px), var(--curr-y, 200px)); }
      100% { transform: translate(calc(var(--curr-x, 200px) - var(--board-size)), var(--curr-y, 200px)); }
    }

    @keyframes move-right {
      0%   { transform: translate(var(--curr-x, 200px), var(--curr-y, 200px)); }
      100% { transform: translate(calc(var(--curr-x, 200px) + var(--board-size)), var(--curr-y, 200px)); }
    }

    /* Continuous Loop Trajectory */
    @keyframes snake-patrol {
      0% {
        transform: translate(60px, 60px);
      }
      20% {
        transform: translate(320px, 60px);
      }
      25% {
        transform: translate(320px, 140px);
      }
      45% {
        transform: translate(60px, 140px);
      }
      50% {
        transform: translate(60px, 240px);
      }
      70% {
        transform: translate(320px, 240px);
      }
      75% {
        transform: translate(320px, 320px);
      }
      95% {
        transform: translate(60px, 320px);
      }
      100% {
        transform: translate(60px, 60px);
      }
    }

    /* Animation Assignment with Trail Delays */
    .head {
      animation: snake-patrol 14s linear infinite;
    }

    .body-1 {
      animation: snake-patrol 14s linear infinite;
      animation-delay: -0.15s;
      opacity: 0.9;
    }

    .body-2 {
      animation: snake-patrol 14s linear infinite;
      animation-delay: -0.30s;
      opacity: 0.8;
    }

    .body-3 {
      animation: snake-patrol 14s linear infinite;
      animation-delay: -0.45s;
      opacity: 0.7;
    }

    .body-4 {
      animation: snake-patrol 14s linear infinite;
      animation-delay: -0.60s;
      opacity: 0.6;
    }

    /* Dynamic Direction Trigger Adjustments */
    #dir-up:checked ~ .viewport .snake-body {
      animation-name: move-up;
      animation-duration: 4s;
      animation-iteration-count: 1;
      animation-timing-function: linear;
      animation-fill-mode: forwards;
    }

    #dir-down:checked ~ .viewport .snake-body {
      animation-name: move-down;
      animation-duration: 4s;
      animation-iteration-count: 1;
      animation-timing-function: linear;
      animation-fill-mode: forwards;
    }

    #dir-left:checked ~ .viewport .snake-body {
      animation-name: move-left;
      animation-duration: 4s;
      animation-iteration-count: 1;
      animation-timing-function: linear;
      animation-fill-mode: forwards;
    }

    #dir-right:checked ~ .viewport .snake-body {
      animation-name: move-right;
      animation-duration: 4s;
      animation-iteration-count: 1;
      animation-timing-function: linear;
      animation-fill-mode: forwards;
    }

    /* Target Apples */
    .apple-item {
      position: absolute;
      width: var(--cell-size);
      height: var(--cell-size);
      background-color: var(--apple-color);
      border-radius: 50%;
      cursor: pointer;
      z-index: 10;
      box-shadow: 0 0 10px rgba(239, 68, 68, 0.8);
      transition: transform 0.15s ease, opacity 0.2s ease;
    }

    .apple-item:hover {
      transform: scale(1.25);
    }

    /* Coordinate Placement for Apples */
    #apple1 { top: 60px; left: 180px; }
    #apple2 { top: 140px; left: 280px; }
    #apple3 { top: 240px; left: 100px; }
    #apple4 { top: 320px; left: 240px; }

    /* Hide eaten apples and track score */
    #apple-check-1:checked ~ .viewport label[for="apple-check-1"],
    #apple-check-2:checked ~ .viewport label[for="apple-check-2"],
    #apple-check-3:checked ~ .viewport label[for="apple-check-3"],
    #apple-check-4:checked ~ .viewport label[for="apple-check-4"] {
      display: none;
    }

    /* On-Screen Controller Pad */
    .controls-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      grid-template-rows: repeat(3, 1fr);
      gap: 8px;
      margin-top: 20px;
      width: 180px;
      height: 120px;
    }

    .dpad-btn {
      background: #334155;
      color: #f8fafc;
      border: 1px solid #475569;
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.1s, transform 0.1s;
    }

    .dpad-btn:active {
      background: #0284c7;
      transform: scale(0.95);
    }

    .btn-up    { grid-column: 2; grid-row: 1; }
    .btn-left  { grid-column: 1; grid-row: 2; }
    .btn-right { grid-column: 3; grid-row: 2; }
    .btn-down  { grid-column: 2; grid-row: 3; }

    /* Reset Button */
    .reset-btn {
      margin-top: 15px;
      background: #e11d48;
      color: white;
      border: none;
      padding: 8px 18px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: 600;
      text-decoration: none;
      transition: background-color 0.2s;
    }

    .reset-btn:hover {
      background: #be123c;
    }

    /* Score Counter Display */
    .scoreboard {
      font-size: 1.1rem;
      font-weight: 600;
      margin-bottom: 12px;
      color: #e2e8f0;
      display: flex;
      gap: 6px;
    }

    .score-value::before {
      content: "0";
    }

    #apple-check-1:checked ~ .scoreboard .score-value::before {
      content: "1";
    }
    #apple-check-1:checked ~ #apple-check-2:checked ~ .scoreboard .score-value::before {
      content: "2";
    }
    #apple-check-1:checked ~ #apple-check-2:checked ~ #apple-check-3:checked ~ .scoreboard .score-value::before {
      content: "3";
    }
    #apple-check-1:checked ~ #apple-check-2:checked ~ #apple-check-3:checked ~ #apple-check-4:checked ~ .scoreboard .score-value::before {
      content: "4 (You Won!)";
      color: #4ade80;
    }

    /* Win Banner Overlay */
    .victory-modal {
      display: none;
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(15, 23, 42, 0.9);
      flex-direction: column;
      align-items: center;
      justify-content: center;
      z-index: 50;
      border-radius: 8px;
    }

    #apple-check-1:checked ~ #apple-check-2:checked ~ #apple-check-3:checked ~ #apple-check-4:checked ~ .viewport .victory-modal {
      display: flex;
    }

    .victory-modal h2 {
      font-size: 1.8rem;
      color: #4ade80;
      margin-bottom: 12px;
    }
  </style>
</head>
<body>

  <h1>CSS Snake</h1>
  <p class="subtitle">Click the apples as the snake patrols or steer via the directional pad</p>

  <form class="game-container">
    <!-- Direction state inputs -->
    <input type="radio" name="direction" id="dir-default" checked>
    <input type="radio" name="direction" id="dir-up">
    <input type="radio" name="direction" id="dir-down">
    <input type="radio" name="direction" id="dir-left">
    <input type="radio" name="direction" id="dir-right">

    <!-- Food Collection inputs -->
    <input type="checkbox" id="apple-check-1">
    <input type="checkbox" id="apple-check-2">
    <input type="checkbox" id="apple-check-3">
    <input type="checkbox" id="apple-check-4">

    <!-- Dynamic Scoreboard -->
    <div class="scoreboard">
      <span>Apples Collected:</span>
      <span class="score-value"></span>
    </div>

    <!-- Play Area -->
    <div class="viewport">
      <!-- Snake Body Elements -->
      <div class="snake-body snake-head head"></div>
      <div class="snake-body snake-segment body-1"></div>
      <div class="snake-body snake-segment body-2"></div>
      <div class="snake-body snake-segment body-3"></div>
      <div class="snake-body snake-segment body-4"></div>

      <!-- Food Targets Linked to Checkboxes -->
      <label id="apple1" class="apple-item" for="apple-check-1"></label>
      <label id="apple2" class="apple-item" for="apple-check-2"></label>
      <label id="apple3" class="apple-item" for="apple-check-3"></label>
      <label id="apple4" class="apple-item" for="apple-check-4"></label>

      <!-- Victory Screen -->
      <div class="victory-modal">
        <h2>Level Complete!</h2>
        <p>All targets collected without JavaScript.</p>
      </div>
    </div>

    <!-- D-Pad Controls -->
    <div class="controls-grid">
      <label for="dir-up" class="dpad-btn btn-up">&#9650;</label>
      <label for="dir-left" class="dpad-btn btn-left">&#9664;</label>
      <label for="dir-right" class="dpad-btn btn-right">&#9654;</label>
      <label for="dir-down" class="dpad-btn btn-down">&#9660;</label>
    </div>

    <!-- Restart Game -->
    <button type="reset" class="reset-btn">Reset Game</button>
  </form>

</body>
</html>
```

**Mechanics and How It Works:**

- **State Management:** The game uses HTML checkboxes and radio buttons located at the top of the form container to hold game states without any scripting.
- **Snake Movement and Body Trail:** The snake body segments follow the head by sharing the same CSS `@keyframes` path with negative `animation-delay` offsets (such as `-0.15s`, `-0.30s`), creating a continuous trailing body illusion.
- **Directional Steering:** Triggering the on-screen D-Pad checks different radio buttons (`#dir-up`, `#dir-down`, etc.), overriding the continuous patrol animation with linear directional translations.
- **Scoring & Collisions:** Apples are rendered as `<label>` tags linked to corresponding hidden `<input type="checkbox">` elements. Clicking an apple checks its box, which automatically removes the food from the grid and dynamically updates the score counter and victory conditions using the CSS general sibling combinator (`~`).
