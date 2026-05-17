
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>the party</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #212922;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    font-family: Georgia, serif;
    cursor: none;
    user-select: none;
  }

  #room {
    position: relative;
    width: 100%;
    height: 100%;
  }

  #cursor {
    position: fixed;
    width: 6px;
    height: 6px;
    background: #6b6058;
    border-radius: 50%;
    pointer-events: none;
    transform: translate(-50%, -50%);
    z-index: 100;
    transition: transform 0.1s ease;
  }

  #cursor-glow {
    position: fixed;
    width: 220px;
    height: 220px;
    border-radius: 50%;
    pointer-events: none;
    transform: translate(-50%, -50%);
    z-index: 1;
    background: radial-gradient(circle, rgba(160,148,132,0.055) 0%, transparent 68%);
  }

  .noise {
    position: absolute;
    color: #e1ea9f;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 13px;
    font-weight: 400;
    line-height: 1.55;
    opacity: 0;
    transition: opacity 1.5s ease;
    pointer-events: none;
    max-width: 210px;
  }

  .noise.on { opacity: 1; }

  #stage {
    position: absolute;
    bottom: 35%;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    width: 400px;
    pointer-events: none;
    z-index: 10;
  }

  #your-line {
    font-family: Georgia, 'Times New Roman', serif;
    font-style: italic;
    font-size: 17px;
    color: #d3adee;
    letter-spacing: 0.025em;
    line-height: 1.85;
    opacity: 0;
    transition: opacity 0.55s ease;
    min-height: 32px;
  }

  #your-line.show { opacity: 1; }
  #your-line.hide { opacity: 0; transition: opacity 0.18s ease; }

  #hint {
    margin-top: 16px;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 10px;
    letter-spacing: 0.22em;
    color: #d3adee;
    text-transform: lowercase;
    opacity: 0;
    transition: opacity 1.4s ease;
  }

  #hint.show { opacity: 1; }

  #intro {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    z-index: 50;
    transition: opacity 2s ease;
  }

  #intro h1 {
    font-family: Georgia, serif;
    font-weight: 400;
    font-size: 22px;
    color: #bdb4a8;
    letter-spacing: 0.12em;
    margin-bottom: 16px;
  }

  #intro p {
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 10px;
    letter-spacing: 0.2em;
    color: #44403c;
    text-transform: lowercase;
  }

  #intro.gone {
    opacity: 0;
    pointer-events: none;
  }

  .pop {
    position: absolute;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 13px;
    color: #d0c8b8;
    pointer-events: none;
    z-index: 20;
    opacity: 0;
    animation: popin 2.1s ease forwards;
  }

  @keyframes popin {
    0%   { opacity: 0; transform: scale(0.88); }
    18%  { opacity: 1; transform: scale(1); }
    72%  { opacity: 1; }
    100% { opacity: 0; }
  }

  #footer {
    position: absolute;
    bottom: 22px;
    left: 50%;
    transform: translateX(-50%);
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 10px;
    letter-spacing: 0.18em;
    color: #2e2b28;
    text-transform: lowercase;
    pointer-events: none;
    z-index: 5;
  }
</style>
</head>
<body>

<div id="cursor"></div>
<div id="cursor-glow"></div>

<div id="room">
  <div id="intro">
    <h1>the party</h1>
    <p>move your mouse to enter the room</p>
  </div>

  <div id="stage">
    <div id="your-line"></div>
    <div id="hint">any minute now</div>
  </div>

  <div id="footer">move to fill the room &nbsp;&middot;&nbsp; click to make it louder</div>
</div>

<script>
const noisePool = [
  "oh my god, no way —",
  "wait, say that again?",
  "I literally can't.",
  "okay but SAME.",
  "hahaha stop it.",
  "no but listen —",
  "she did NOT.",
  "I'm obsessed with this.",
  "right? right??",
  "okay so basically —",
  "that's so funny because —",
  "wait who told you?",
  "I love this for you.",
  "genuinely iconic.",
  "okay I'm crying.",
  "no but actually though.",
  "stop stop stop —",
  "you had to be there.",
  "I mean, obviously.",
  "yes! exactly!",
  "I can't even.",
  "no yeah, totally.",
  "same, honestly.",
  "wait but also —",
  "that is SO valid.",
  "right? it's just —",
  "I know, I KNOW.",
  "wait hold on —",
  "this is everything.",
  "okay but real talk —",
  "no that's so funny.",
  "I'm so serious right now.",
  "you're insane, I love it.",
  "okay but can we talk about —",
  "no no no, listen,",
  "hahaha oh my god.",
  "omg send that to me", "ew I look so bad", "did you hear??"
];

const yourLines = [
    "I practiced this on the  way here.",
    "Where should I stand?",
    "Any minute now I'll say it.",
    "No no I can't do it.",
    "I hope they're still my friends after this."
    ""
];

let started = false;
let lineIdx = 0;
let noiseEls = [];
let gapTimer = null;

const room    = document.getElementById('room');
const intro   = document.getElementById('intro');
const yourLine = document.getElementById('your-line');
const hint    = document.getElementById('hint');
const cursor  = document.getElementById('cursor');
const glow    = document.getElementById('cursor-glow');

function r(min, max) { return Math.random() * (max - min) + min; }

const zones = [
  [2, 22], [74, 94],
  [2, 22], [74, 94],
  [4, 38], [56, 90],
  [4, 38], [56, 90],
  [2, 18], [78, 96],
];

function spawnNoise() {
  const el = document.createElement('div');
  el.className = 'noise';
  const zone = zones[Math.floor(Math.random() * zones.length)];
  el.style.left = r(zone[0], zone[1]) + '%';
  el.style.top  = r(6, 88) + '%';
  el.textContent = noisePool[Math.floor(Math.random() * noisePool.length)];
  room.appendChild(el);
  noiseEls.push(el);
  requestAnimationFrame(() => el.classList.add('on'));
  const life = r(3000, 7500);
  setTimeout(() => {
    el.classList.remove('on');
    setTimeout(() => {
      el.remove();
      noiseEls = noiseEls.filter(e => e !== el);
    }, 1600);
  }, life);
}

function popBurst(x, y) {
  const el = document.createElement('div');
  el.className = 'pop';
  el.textContent = noisePool[Math.floor(Math.random() * noisePool.length)];
  el.style.left = (x + r(-110, 110)) + 'px';
  el.style.top  = (y + r(-55, 55)) + 'px';
  room.appendChild(el);
  setTimeout(() => el.remove(), 2300);
}

function showYourLine() {
  const line = yourLines[lineIdx % yourLines.length];
  lineIdx++;
  yourLine.textContent = line;
  hint.classList.remove('show');
  yourLine.classList.remove('hide');
  yourLine.classList.add('show');
  gapTimer = setTimeout(buryLine, r(1500, 2800));
}

function buryLine() {
  yourLine.classList.add('hide');
  yourLine.classList.remove('show');
  setTimeout(() => {
    hint.classList.add('show');
    gapTimer = setTimeout(showYourLine, r(5000, 10000));
  }, 800);
}

function startRoom() {
  started = true;
  intro.classList.add('gone');
  for (let i = 0; i < 8; i++) setTimeout(spawnNoise, i * 260);
  setInterval(() => { if (noiseEls.length < 12) spawnNoise(); }, 950);
  setTimeout(() => {
    hint.classList.add('show');
    gapTimer = setTimeout(showYourLine, 3200);
  }, 2000);
}

document.addEventListener('mousemove', (e) => {
  cursor.style.left = e.clientX + 'px';
  cursor.style.top  = e.clientY + 'px';
  glow.style.left   = e.clientX + 'px';
  glow.style.top    = e.clientY + 'px';
  if (!started) startRoom();
  if (Math.random() < 0.032) popBurst(e.clientX, e.clientY);
});

document.addEventListener('click', (e) => {
  for (let i = 0; i < 5; i++) {
    setTimeout(() => popBurst(e.clientX, e.clientY), i * 90);
  }
});
</script>
</body>
</html>
