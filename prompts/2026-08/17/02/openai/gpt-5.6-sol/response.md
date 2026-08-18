I can’t redistribute the original proprietary Flappy Bird artwork or claim an exact asset-for-asset replica. Below is a complete single-file alternative with original, embedded pixel-style artwork, endless procedural pipes, responsive controls, sound, scoring, and saved high scores.

Save as `index.html` and open it in a browser:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
  />
  <meta name="theme-color" content="#62c9de" />
  <title>Flappy Flight</title>

  <style>
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    html,
    body {
      width: 100%;
      height: 100%;
      margin: 0;
      overflow: hidden;
      background:
        radial-gradient(circle at 50% 25%, #bff5ff 0 15%, transparent 45%),
        linear-gradient(#68d4e8, #56b9d5);
      font-family: Arial, Helvetica, sans-serif;
      touch-action: none;
      user-select: none;
    }

    body {
      display: grid;
      place-items: center;
    }

    #game-shell {
      position: relative;
      width: min(100vw, calc(100vh * 0.5625));
      height: min(100vh, calc(100vw * 1.7777778));
      overflow: hidden;
      background: #62c9de;
      box-shadow:
        0 0 0 2px rgba(0, 0, 0, 0.35),
        0 18px 60px rgba(0, 0, 0, 0.35);
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
      image-rendering: pixelated;
      image-rendering: crisp-edges;
      cursor: pointer;
    }

    #sound-button {
      position: absolute;
      z-index: 5;
      top: max(10px, env(safe-area-inset-top));
      right: 10px;
      width: 42px;
      height: 42px;
      padding: 0;
      border: 3px solid #54351e;
      border-radius: 8px;
      color: #fff;
      background: rgba(34, 123, 146, 0.82);
      box-shadow:
        inset 0 0 0 2px rgba(255, 255, 255, 0.35),
        0 3px 0 #3b2617;
      font-size: 20px;
      line-height: 36px;
      cursor: pointer;
    }

    #sound-button:active {
      transform: translateY(2px);
      box-shadow:
        inset 0 0 0 2px rgba(255, 255, 255, 0.35),
        0 1px 0 #3b2617;
    }

    #hint {
      position: absolute;
      z-index: 4;
      left: 50%;
      bottom: max(14px, env(safe-area-inset-bottom));
      transform: translateX(-50%);
      padding: 6px 10px;
      border-radius: 20px;
      color: rgba(255, 255, 255, 0.95);
      background: rgba(0, 0, 0, 0.18);
      font-size: clamp(10px, 2.7vw, 13px);
      font-weight: 700;
      letter-spacing: 0.5px;
      white-space: nowrap;
      pointer-events: none;
      opacity: 0.9;
    }

    @media (max-width: 500px) {
      #game-shell {
        width: 100vw;
        height: 100vh;
        box-shadow: none;
      }
    }
  </style>
</head>

<body>
  <main id="game-shell" aria-label="Flappy Flight game">
    <canvas id="game" width="432" height="768"></canvas>
    <button id="sound-button" aria-label="Toggle sound" title="Toggle sound">🔊</button>
    <div id="hint">CLICK • TAP • SPACE</div>
  </main>

  <script>
    (() => {
      "use strict";

      const canvas = document.getElementById("game");
      const ctx = canvas.getContext("2d", { alpha: false });
      const soundButton = document.getElementById("sound-button");
      const hint = document.getElementById("hint");

      const WIDTH = 432;
      const HEIGHT = 768;
      const GROUND_Y = 650;
      const GROUND_HEIGHT = HEIGHT - GROUND_Y;

      const STATE = Object.freeze({
        READY: 0,
        PLAYING: 1,
        GAME_OVER: 2
      });

      let state = STATE.READY;
      let score = 0;
      let best = loadBest();
      let elapsed = 0;
      let groundOffset = 0;
      let shake = 0;
      let flash = 0;
      let lastTime = performance.now();
      let soundEnabled = true;
      let audioContext = null;

      const bird = {
        x: 118,
        y: 320,
        radius: 17,
        velocity: 0,
        rotation: 0,
        wing: 0
      };

      const physics = {
        gravity: 1500,
        flap: -470,
        maxFall: 720
      };

      const pipeConfig = {
        width: 74,
        gap: 174,
        speed: 145,
        spacing: 224,
        marginTop: 78,
        marginBottom: 86
      };

      let pipes = [];
      let particles = [];

      function loadBest() {
        try {
          return Math.max(
            0,
            Number.parseInt(localStorage.getItem("flappy-flight-best"), 10) || 0
          );
        } catch {
          return 0;
        }
      }

      function saveBest() {
        try {
          localStorage.setItem("flappy-flight-best", String(best));
        } catch {
          // The game still works when storage is unavailable.
        }
      }

      function resetGame() {
        state = STATE.READY;
        score = 0;
        elapsed = 0;
        groundOffset = 0;
        shake = 0;
        flash = 0;
        pipes = [];
        particles = [];

        bird.x = 118;
        bird.y = 320;
        bird.velocity = 0;
        bird.rotation = 0;
        bird.wing = 0;

        hint.style.opacity = "0.9";
      }

      function startGame() {
        state = STATE.PLAYING;
        score = 0;
        pipes = [];
        particles = [];
        bird.y = 320;
        bird.velocity = 0;
        bird.rotation = 0;
        hint.style.opacity = "0";

        spawnPipe(WIDTH + 150);
        spawnPipe(WIDTH + 150 + pipeConfig.spacing);
        flap();
      }

      function flap() {
        if (state !== STATE.PLAYING) return;

        bird.velocity = physics.flap;
        bird.rotation = -0.48;
        bird.wing = 1;

        emitParticles(bird.x - 16, bird.y + 4, 4, "#ffffff");
        playTone(520, 0.055, "square", 0.035, 180);
      }

      function endGame() {
        if (state !== STATE.PLAYING) return;

        state = STATE.GAME_OVER;
        shake = 9;
        flash = 0.75;

        if (score > best) {
          best = score;
          saveBest();
        }

        playHitSound();
        emitParticles(bird.x, bird.y, 10, "#f7df43");
      }

      function primaryAction(event) {
        if (event) event.preventDefault();

        ensureAudio();

        if (state === STATE.READY) {
          startGame();
        } else if (state === STATE.PLAYING) {
          flap();
        } else if (state === STATE.GAME_OVER) {
          resetGame();
        }
      }

      canvas.addEventListener("pointerdown", primaryAction);

      window.addEventListener(
        "keydown",
        event => {
          if (
            event.code === "Space" ||
            event.code === "ArrowUp" ||
            event.code === "KeyW"
          ) {
            primaryAction(event);
          }

          if (event.code === "KeyM") {
            toggleSound();
          }
        },
        { passive: false }
      );

      soundButton.addEventListener("pointerdown", event => {
        event.preventDefault();
        event.stopPropagation();
        toggleSound();
      });

      function toggleSound() {
        soundEnabled = !soundEnabled;
        soundButton.textContent = soundEnabled ? "🔊" : "🔇";
        soundButton.setAttribute(
          "aria-label",
          soundEnabled ? "Mute sound" : "Enable sound"
        );

        if (soundEnabled) {
          ensureAudio();
          playTone(660, 0.05, "square", 0.025);
        }
      }

      function ensureAudio() {
        if (!soundEnabled) return;

        if (!audioContext) {
          const AudioCtx = window.AudioContext || window.webkitAudioContext;
          if (AudioCtx) audioContext = new AudioCtx();
        }

        if (audioContext && audioContext.state === "suspended") {
          audioContext.resume();
        }
      }

      function playTone(
        frequency,
        duration,
        type = "square",
        volume = 0.03,
        slide = 0
      ) {
        if (!soundEnabled) return;

        ensureAudio();
        if (!audioContext) return;

        const now = audioContext.currentTime;
        const oscillator = audioContext.createOscillator();
        const gain = audioContext.createGain();

        oscillator.type = type;
        oscillator.frequency.setValueAtTime(frequency, now);

        if (slide) {
          oscillator.frequency.exponentialRampToValueAtTime(
            Math.max(40, frequency + slide),
            now + duration
          );
        }

        gain.gain.setValueAtTime(volume, now);
        gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);

        oscillator.connect(gain);
        gain.connect(audioContext.destination);

        oscillator.start(now);
        oscillator.stop(now + duration);
      }

      function playScoreSound() {
        playTone(760, 0.06, "square", 0.025, 160);
        setTimeout(() => {
          playTone(980, 0.07, "square", 0.02, 100);
        }, 55);
      }

      function playHitSound() {
        playTone(180, 0.14, "square", 0.05, -100);
        setTimeout(() => {
          playTone(90, 0.22, "sawtooth", 0.035, -40);
        }, 70);
      }

      function randomPipeCenter() {
        const halfGap = pipeConfig.gap / 2;
        const min =
          pipeConfig.marginTop +
          halfGap +
          20;

        const max =
          GROUND_Y -
          pipeConfig.marginBottom -
          halfGap;

        return min + Math.random() * (max - min);
      }

      function spawnPipe(x) {
        pipes.push({
          x,
          center: randomPipeCenter(),
          counted: false
        });
      }

      function emitParticles(x, y, amount, color) {
        for (let i = 0; i < amount; i++) {
          particles.push({
            x,
            y,
            vx: -40 - Math.random() * 100,
            vy: -70 + Math.random() * 140,
            size: 2 + Math.random() * 4,
            life: 0.3 + Math.random() * 0.35,
            maxLife: 0.65,
            color
          });
        }
      }

      function update(dt) {
        elapsed += dt;
        groundOffset =
          (groundOffset + pipeConfig.speed * dt) % 48;

        shake = Math.max(0, shake - 30 * dt);
        flash = Math.max(0, flash - 2.5 * dt);

        updateParticles(dt);

        if (state === STATE.READY) {
          bird.y = 320 + Math.sin(elapsed * 4.5) * 11;
          bird.rotation = Math.sin(elapsed * 4.5) * 0.08;
          bird.wing = (Math.sin(elapsed * 12) + 1) / 2;
          return;
        }

        if (state === STATE.PLAYING) {
          bird.velocity = Math.min(
            physics.maxFall,
            bird.velocity + physics.gravity * dt
          );

          bird.y += bird.velocity * dt;
          bird.rotation = Math.min(
            1.48,
            bird.rotation + 2.35 * dt
          );
          bird.wing = (Math.sin(elapsed * 22) + 1) / 2;

          for (const pipe of pipes) {
            pipe.x -= pipeConfig.speed * dt;

            if (!pipe.counted && pipe.x + pipeConfig.width < bird.x) {
              pipe.counted = true;
              score++;
              flash = 0.18;
              playScoreSound();
            }
          }

          pipes = pipes.filter(
            pipe => pipe.x + pipeConfig.width > -20
          );

          const lastPipe = pipes[pipes.length - 1];

          if (
            !lastPipe ||
            lastPipe.x < WIDTH - pipeConfig.spacing
          ) {
            spawnPipe(
              lastPipe
                ? lastPipe.x + pipeConfig.spacing
                : WIDTH + 100
            );
          }

          if (checkCollisions()) {
            endGame();
          }
        } else if (state === STATE.GAME_OVER) {
          if (bird.y + bird.radius < GROUND_Y) {
            bird.velocity = Math.min(
              physics.maxFall,
              bird.velocity + physics.gravity * dt
            );
            bird.y += bird.velocity * dt;
            bird.rotation = Math.min(
              1.55,
              bird.rotation + 4.5 * dt
            );

            if (bird.y + bird.radius >= GROUND_Y) {
              bird.y = GROUND_Y - bird.radius;
              bird.velocity = 0;
              shake = Math.max(shake, 5);
            }
          }
        }
      }

      function updateParticles(dt) {
        for (const particle of particles) {
          particle.life -= dt;
          particle.x += particle.vx * dt;
          particle.y += particle.vy * dt;
          particle.vy += 240 * dt;
          particle.vx *= Math.pow(0.2, dt);
        }

        particles = particles.filter(
          particle => particle.life > 0
        );
      }

      function checkCollisions() {
        const bx = bird.x;
        const by = bird.y;
        const radius = bird.radius * 0.72;

        if (by - radius < 0) return true;
        if (by + radius >= GROUND_Y) return true;

        for (const pipe of pipes) {
          const left = pipe.x;
          const right = pipe.x + pipeConfig.width;
          const gapTop = pipe.center - pipeConfig.gap / 2;
          const gapBottom = pipe.center + pipeConfig.gap / 2;

          const insidePipeX =
            bx + radius > left &&
            bx - radius < right;

          if (
            insidePipeX &&
            (by - radius < gapTop ||
              by + radius > gapBottom)
          ) {
            return true;
          }
        }

        return false;
      }

      function draw() {
        ctx.save();

        if (shake > 0) {
          ctx.translate(
            (Math.random() - 0.5) * shake,
            (Math.random() - 0.5) * shake
          );
        }

        drawSky();
        drawDistantClouds();
        drawCity();
        drawPipes();
        drawParticles();
        drawGround();
        drawBird();

        if (state === STATE.READY) {
          drawReadyScreen();
        } else if (state === STATE.PLAYING) {
          drawScore();
        } else {
          drawGameOverScreen();
        }

        if (flash > 0) {
          ctx.fillStyle = `rgba(255,255,255,${Math.min(
            0.55,
            flash
          )})`;
          ctx.fillRect(0, 0, WIDTH, HEIGHT);
        }

        ctx.restore();
      }

      function drawSky() {
        const gradient = ctx.createLinearGradient(0, 0, 0, GROUND_Y);
        gradient.addColorStop(0, "#53c5df");
        gradient.addColorStop(0.65, "#78d7df");
        gradient.addColorStop(1, "#b8ecdc");

        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, WIDTH, HEIGHT);

        ctx.fillStyle = "rgba(255,255,255,0.08)";
        for (let y = 28; y < GROUND_Y; y += 34) {
          ctx.fillRect(0, y, WIDTH, 2);
        }
      }

      function drawDistantClouds() {
        const shift = (elapsed * 8) % 520;

        ctx.save();
        ctx.fillStyle = "rgba(255,255,255,0.82)";
        ctx.strokeStyle = "rgba(71,164,177,0.35)";
        ctx.lineWidth = 3;

        for (let i = -1; i < 2; i++) {
          const x = i * 260 - shift + 85;
          const y = 170 + (i % 2) * 80;

          ctx.beginPath();
          ctx.arc(x, y, 24, Math.PI, 0);
          ctx.arc(x + 27, y - 15, 31, Math.PI, 0);
          ctx.arc(x + 62, y, 25, Math.PI, 0);
          ctx.lineTo(x + 87, y + 21);
          ctx.lineTo(x - 24, y + 21);
          ctx.closePath();
          ctx.fill();
          ctx.stroke();
        }

        ctx.restore();
      }

      function drawCity() {
        const base = GROUND_Y - 22;

        ctx.fillStyle = "#d6e7c7";
        ctx.fillRect(0, base - 105, WIDTH, 105);

        ctx.fillStyle = "#accdb6";
        for (let x = -10; x < WIDTH + 25; x += 38) {
          const h = 25 + ((x * 7) % 38 + 38) % 38;
          ctx.fillRect(x, base - h, 30, h);

          ctx.fillStyle = "#eaf2cf";
          ctx.fillRect(x + 6, base - h + 8, 6, 8);
          ctx.fillRect(x + 18, base - h + 8, 6, 8);
          ctx.fillStyle = "#accdb6";
        }

        ctx.fillStyle = "#76c5a1";
        for (let x = -20; x < WIDTH + 30; x += 30) {
          ctx.beginPath();
          ctx.arc(x, base - 5, 25, Math.PI, 0);
          ctx.fill();
        }

        ctx.fillStyle = "#ecf5d4";
        ctx.fillRect(0, base - 3, WIDTH, 7);
      }

      function drawPipes() {
        for (const pipe of pipes) {
          const gapTop = pipe.center - pipeConfig.gap / 2;
          const gapBottom = pipe.center + pipeConfig.gap / 2;

          drawPipe(
            Math.round(pipe.x),
            0,
            pipeConfig.width,
            gapTop,
            true
          );

          drawPipe(
            Math.round(pipe.x),
            gapBottom,
            pipeConfig.width,
            GROUND_Y - gapBottom,
            false
          );
        }
      }

      function drawPipe(x, y, width, height, upsideDown) {
        if (height <= 0) return;

        const capHeight = 34;
        const capOverhang = 7;

        const bodyY = upsideDown ? y : y + capHeight;
        const bodyHeight = height - capHeight;
        const capY = upsideDown
          ? y + height - capHeight
          : y;

        ctx.save();

        ctx.fillStyle = "#29291a";
        ctx.fillRect(
          x - 3,
          bodyY,
          width + 6,
          Math.max(0, bodyHeight)
        );

        const gradient = ctx.createLinearGradient(
          x,
          0,
          x + width,
          0
        );
        gradient.addColorStop(0, "#72a91f");
        gradient.addColorStop(0.13, "#d4ed52");
        gradient.addColorStop(0.28, "#9bd32d");
        gradient.addColorStop(0.63, "#5ca819");
        gradient.addColorStop(0.82, "#438514");
        gradient.addColorStop(1, "#2c6311");

        ctx.fillStyle = gradient;
        ctx.fillRect(
          x,
          bodyY,
          width,
          Math.max(0, bodyHeight)
        );

        ctx.fillStyle = "rgba(255,255,255,0.35)";
        ctx.fillRect(
          x + 10,
          bodyY,
          7,
          Math.max(0, bodyHeight)
        );

        ctx.fillStyle = "#28291a";
        ctx.fillRect(
          x - capOverhang - 3,
          capY - 3,
          width + capOverhang * 2 + 6,
          capHeight + 6
        );

        ctx.fillStyle = gradient;
        ctx.fillRect(
          x - capOverhang,
          capY,
          width + capOverhang * 2,
          capHeight
        );

        ctx.fillStyle = "rgba(255,255,255,0.42)";
        ctx.fillRect(
          x + 5,
          capY + 3,
          8,
          capHeight - 6
        );

        ctx.fillStyle = "#315d11";
        ctx.fillRect(
          x + width - 8,
          capY + 2,
          6,
          capHeight - 4
        );

        ctx.restore();
      }

      function drawGround() {
        ctx.fillStyle = "#5e932a";
        ctx.fillRect(0, GROUND_Y, WIDTH, 8);

        ctx.fillStyle = "#d9ed68";
        ctx.fillRect(0, GROUND_Y + 8, WIDTH, 22);

        ctx.fillStyle = "#8bc63f";
        for (
          let x = -48 - groundOffset;
          x < WIDTH + 48;
          x += 48
        ) {
          ctx.beginPath();
          ctx.moveTo(x, GROUND_Y + 29);
          ctx.lineTo(x + 18, GROUND_Y + 8);
          ctx.lineTo(x + 31, GROUND_Y + 8);
          ctx.lineTo(x + 13, GROUND_Y + 29);
          ctx.closePath();
          ctx.fill();
        }

        ctx.fillStyle = "#5c8a34";
        ctx.fillRect(0, GROUND_Y + 30, WIDTH, 6);

        const dirt = ctx.createLinearGradient(
          0,
          GROUND_Y + 36,
          0,
          HEIGHT
        );
        dirt.addColorStop(0, "#ded690");
        dirt.addColorStop(1, "#c8b870");

        ctx.fillStyle = dirt;
        ctx.fillRect(
          0,
          GROUND_Y + 36,
          WIDTH,
          GROUND_HEIGHT - 36
        );

        ctx.fillStyle = "rgba(135,112,53,0.18)";
        for (let y = GROUND_Y + 50; y < HEIGHT; y += 18) {
          for (let x = (y % 36); x < WIDTH; x += 36) {
            ctx.fillRect(x, y, 15, 3);
          }
        }
      }

      function drawBird() {
        ctx.save();
        ctx.translate(Math.round(bird.x), Math.round(bird.y));
        ctx.rotate(bird.rotation);

        const wingY = 5 + bird.wing * 8;

        // Tail
        ctx.fillStyle = "#231812";
        ctx.beginPath();
        ctx.moveTo(-20, -4);
        ctx.lineTo(-32, -12);
        ctx.lineTo(-30, 7);
        ctx.lineTo(-18, 10);
        ctx.closePath();
        ctx.fill();

        ctx.fillStyle = "#e94f2c";
        ctx.beginPath();
        ctx.moveTo(-19, -2);
        ctx.lineTo(-29, -8);
        ctx.lineTo(-28, 4);
        ctx.lineTo(-17, 7);
        ctx.closePath();
        ctx.fill();

        // Body outline
        ctx.fillStyle = "#211710";
        ctx.beginPath();
        ctx.ellipse(0, 0, 25, 19, 0, 0, Math.PI * 2);
        ctx.fill();

        // Body
        ctx.fillStyle = "#f2d332";
        ctx.beginPath();
        ctx.ellipse(-1, -1, 22, 16, 0, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = "#ffed56";
        ctx.beginPath();
        ctx.ellipse(-6, -7, 14, 8, -0.2, 0, Math.PI * 2);
        ctx.fill();

        // Wing
        ctx.fillStyle = "#261912";
        ctx.beginPath();
        ctx.ellipse(
          -11,
          wingY,
          15,
          9,
          -0.12,
          0,
          Math.PI * 2
        );
        ctx.fill();

        ctx.fillStyle = "#f7f2d6";
        ctx.beginPath();
        ctx.ellipse(
          -11,
          wingY - 1,
          12,
          6,
          -0.12,
          0,
          Math.PI * 2
        );
        ctx.fill();

        // Eye
        ctx.fillStyle = "#241a14";
        ctx.beginPath();
        ctx.arc(11, -10, 10, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = "#ffffff";
        ctx.beginPath();
        ctx.arc(11, -11, 7, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = "#17120f";
        ctx.fillRect(13, -15, 4, 8);

        // Beak
        ctx.fillStyle = "#271812";
        ctx.fillRect(15, -3, 24, 12);

        ctx.fillStyle = "#f26b31";
        ctx.fillRect(17, -1, 19, 5);

        ctx.fillStyle = "#de3e25";
        ctx.fillRect(17, 4, 16, 4);

        ctx.restore();
      }

      function drawParticles() {
        for (const particle of particles) {
          const alpha = Math.max(
            0,
            particle.life / particle.maxLife
          );

          ctx.fillStyle = colorWithAlpha(
            particle.color,
            alpha
          );

          ctx.fillRect(
            Math.round(particle.x),
            Math.round(particle.y),
            Math.ceil(particle.size),
            Math.ceil(particle.size)
          );
        }
      }

      function colorWithAlpha(color, alpha) {
        if (color.startsWith("#")) {
          const hex = color.slice(1);
          const value = Number.parseInt(hex, 16);
          const r = (value >> 16) & 255;
          const g = (value >> 8) & 255;
          const b = value & 255;
          return `rgba(${r},${g},${b},${alpha})`;
        }

        return color;
      }

      function drawReadyScreen() {
        drawTitle("FLAPPY", "FLIGHT", 112);

        drawPixelText(
          "GET READY!",
          WIDTH / 2,
          258,
          34,
          "#ffffff",
          "#4d3827",
          5
        );

        ctx.save();
        ctx.translate(WIDTH / 2, 425);

        ctx.fillStyle = "rgba(255,255,255,0.95)";
        ctx.strokeStyle = "#4d3827";
        ctx.lineWidth = 5;

        roundedRect(-59, -45, 118, 90, 12);
        ctx.fill();
        ctx.stroke();

        ctx.fillStyle = "#efc94c";
        ctx.beginPath();
        ctx.arc(0, -8, 16, 0, Math.PI * 2);
        ctx.fill();
        ctx.stroke();

        ctx.fillStyle = "#4d3827";
        ctx.beginPath();
        ctx.moveTo(-17, 14);
        ctx.lineTo(17, 14);
        ctx.lineTo(0, 34);
        ctx.closePath();
        ctx.fill();

        ctx.restore();

        drawPixelText(
          "TAP TO FLY",
          WIDTH / 2,
          498,
          22,
          "#ffffff",
          "#4d3827",
          4
        );
      }

      function drawTitle(top, bottom, y) {
        ctx.save();
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.font = "900 57px Arial Black, Arial, sans-serif";
        ctx.lineJoin = "round";
        ctx.lineWidth = 12;
        ctx.strokeStyle = "#51351f";
        ctx.fillStyle = "#ffffff";

        ctx.strokeText(top, WIDTH / 2, y);
        ctx.fillText(top, WIDTH / 2, y);

        ctx.font = "900 49px Arial Black, Arial, sans-serif";
        ctx.strokeText(bottom, WIDTH / 2, y + 52);

        const titleGradient = ctx.createLinearGradient(
          0,
          y + 28,
          0,
          y + 76
        );
        titleGradient.addColorStop(0, "#ffe559");
        titleGradient.addColorStop(1, "#f0a72e");

        ctx.fillStyle = titleGradient;
        ctx.fillText(bottom, WIDTH / 2, y + 52);
        ctx.restore();
      }

      function drawScore() {
        drawPixelText(
          String(score),
          WIDTH / 2,
          82,
          54,
          "#ffffff",
          "#493523",
          6
        );
      }

      function drawGameOverScreen() {
        drawPixelText(
          "GAME OVER",
          WIDTH / 2,
          160,
          43,
          "#f6f1dd",
          "#4f3420",
          6
        );

        drawScoreCard();

        drawPixelText(
          "TAP TO RETRY",
          WIDTH / 2,
          514,
          24,
          "#ffffff",
          "#4f3420",
          4
        );
      }

      function drawScoreCard() {
        const x = 62;
        const y = 230;
        const width = 308;
        const height = 205;

        ctx.save();

        ctx.fillStyle = "rgba(70,45,26,0.28)";
        roundedRect(x + 5, y + 8, width, height, 12);
        ctx.fill();

        ctx.fillStyle = "#e7d49b";
        ctx.strokeStyle = "#563821";
        ctx.lineWidth = 6;
        roundedRect(x, y, width, height, 12);
        ctx.fill();
        ctx.stroke();

        ctx.fillStyle = "#f5e8bd";
        roundedRect(x + 11, y + 11, width - 22, height - 22, 7);
        ctx.fill();

        ctx.textAlign = "right";
        ctx.textBaseline = "middle";
        ctx.font = "900 18px Arial, sans-serif";
        ctx.fillStyle = "#8d795c";
        ctx.fillText("SCORE", x + width - 22, y + 45);
        ctx.fillText("BEST", x + width - 22, y + 112);

        drawSmallNumber(
          String(score),
          x + width - 24,
          y + 76
        );

        drawSmallNumber(
          String(best),
          x + width - 24,
          y + 143
        );

        drawMedal(x + 72, y + 103, score);

        if (score > 0 && score >= best) {
          ctx.textAlign = "center";
          ctx.font = "900 14px Arial, sans-serif";
          ctx.fillStyle = "#ef633d";
          ctx.fillText("NEW!", x + width - 53, y + 170);
        }

        ctx.restore();
      }

      function drawMedal(x, y, value) {
        let color = "#c78545";
        let highlight = "#edb36d";

        if (value >= 10) {
          color = "#aeb9bd";
          highlight = "#eef3f3";
        }

        if (value >= 20) {
          color = "#e3af31";
          highlight = "#ffe56a";
        }

        if (value >= 40) {
          color = "#d75c45";
          highlight = "#ff9e72";
        }

        ctx.save();
        ctx.translate(x, y);

        ctx.fillStyle = "#55402d";
        ctx.beginPath();
        ctx.arc(0, 0, 36, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = color;
        ctx.beginPath();
        ctx.arc(0, 0, 31, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = highlight;
        ctx.beginPath();
        ctx.moveTo(0, -23);
        ctx.lineTo(7, -7);
        ctx.lineTo(24, -7);
        ctx.lineTo(11, 4);
        ctx.lineTo(16, 21);
        ctx.lineTo(0, 11);
        ctx.lineTo(-16, 21);
        ctx.lineTo(-11, 4);
        ctx.lineTo(-24, -7);
        ctx.lineTo(-7, -7);
        ctx.closePath();
        ctx.fill();

        ctx.restore();
      }

      function drawSmallNumber(text, x, y) {
        ctx.save();
        ctx.textAlign = "right";
        ctx.textBaseline = "middle";
        ctx.font = "900 34px Arial Black, Arial, sans-serif";
        ctx.lineWidth = 4;
        ctx.strokeStyle = "#4b3929";
        ctx.fillStyle = "#ffffff";
        ctx.strokeText(text, x, y);
        ctx.fillText(text, x, y);
        ctx.restore();
      }

      function drawPixelText(
        text,
        x,
        y,
        size,
        fill,
        outline,
        outlineWidth
      ) {
        ctx.save();
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.font = `900 ${size}px Arial Black, Arial, sans-serif`;
        ctx.lineJoin = "round";
        ctx.lineWidth = outlineWidth;
        ctx.strokeStyle = outline;
        ctx.fillStyle = fill;
        ctx.strokeText(text, x, y);
        ctx.fillText(text, x, y);
        ctx.restore();
      }

      function roundedRect(x, y, width, height, radius) {
        const r = Math.min(
          radius,
          width / 2,
          height / 2
        );

        ctx.beginPath();
        ctx.moveTo(x + r, y);
        ctx.arcTo(x + width, y, x + width, y + height, r);
        ctx.arcTo(
          x + width,
          y + height,
          x,
          y + height,
          r
        );
        ctx.arcTo(x, y + height, x, y, r);
        ctx.arcTo(x, y, x + width, y, r);
        ctx.closePath();
      }

      function gameLoop(now) {
        const dt = Math.min(0.033, (now - lastTime) / 1000);
        lastTime = now;

        update(dt);
        draw();

        requestAnimationFrame(gameLoop);
      }

      document.addEventListener("visibilitychange", () => {
        lastTime = performance.now();
      });

      resetGame();
      requestAnimationFrame(gameLoop);
    })();
  </script>
</body>
</html>
```
