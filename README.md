<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Abschlussprüfungstimer</title>
  <style>
    :root {
      --purple:       #5b3fa6;
      --purple-light: #7c5cce;
      --purple-pale:  #ede8f9;
      --purple-dark:  #3d2875;
      --bg:           #f4f0fc;
      --card:         #ffffff;
      --text:         #2d1b69;
      --muted:        #9588c0;
      --border:       #d4c8f0;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      background: var(--bg);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: var(--text);
    }

    /* ── SETUP ─────────────────────────────────────────────── */
    #setupScreen {
      background: var(--card);
      border-radius: 28px;
      padding: 52px 48px;
      width: 500px;
      box-shadow: 0 12px 48px rgba(91,63,166,.18);
      border: 2px solid var(--border);
    }

    .setup-title {
      font-size: 30px;
      font-weight: 800;
      color: var(--purple-dark);
      letter-spacing: -.5px;
    }
    .setup-sub {
      color: var(--muted);
      font-size: 14px;
      margin: 4px 0 36px;
    }

    .form-group { margin-bottom: 24px; }

    .form-label {
      display: block;
      font-size: 11px;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: .1em;
      color: var(--muted);
      margin-bottom: 8px;
    }

    input[type="text"], input[type="time"] {
      width: 100%;
      padding: 12px 16px;
      border: 2px solid var(--border);
      border-radius: 12px;
      font-size: 16px;
      color: var(--text);
      background: var(--bg);
      outline: none;
      transition: border-color .2s;
    }
    input:focus { border-color: var(--purple-light); }

    .niveau-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 12px;
    }
    .niveau-btn {
      border: 2px solid var(--border);
      border-radius: 14px;
      padding: 18px 8px;
      cursor: pointer;
      text-align: center;
      transition: all .2s;
      background: var(--bg);
      user-select: none;
    }
    .niveau-btn:hover { border-color: var(--purple-light); background: var(--purple-pale); }
    .niveau-btn.selected { border-color: var(--purple); background: var(--purple); color: #fff; }
    .niveau-btn .nb-name { font-size: 22px; font-weight: 800; }
    .niveau-btn .nb-desc { font-size: 11px; opacity: .75; margin-top: 4px; }

    /* Zeiten-Panel */
    #zeitenPanel {
      display: none;
      margin-top: 16px;
      background: var(--purple-pale);
      border-radius: 14px;
      padding: 18px 20px 6px;
    }
    #zeitenPanel .zp-title {
      font-size: 11px;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: .1em;
      color: var(--muted);
      margin-bottom: 14px;
    }
    #zeitenPanel input[type="text"] {
      background: var(--card);
      text-align: center;
      font-size: 18px;
      font-weight: 700;
      color: var(--purple-dark);
      padding: 10px 12px;
    }
    /* Phasen-Flash */
    #phaseFlash {
      display: none;
      position: fixed;
      top: 32px;
      left: 50%;
      transform: translateX(-50%);
      background: var(--purple-dark);
      color: #fff;
      padding: 14px 36px;
      border-radius: 999px;
      font-size: 16px;
      font-weight: 700;
      letter-spacing: .03em;
      z-index: 300;
      pointer-events: none;
      animation: flashIn .25s ease;
    }
    @keyframes flashIn {
      from { opacity: 0; transform: translateX(-50%) translateY(-10px); }
      to   { opacity: 1; transform: translateX(-50%) translateY(0); }
    }

    /* Laufender Timer: Zeit-Anpassung */
    .timer-adj { display: flex; gap: 10px; margin-top: 2px; }
    .btn-timer-adj {
      padding: 8px 20px;
      border: 2px solid var(--border);
      border-radius: 999px;
      background: var(--bg);
      color: var(--purple);
      font-size: 13px;
      font-weight: 800;
      cursor: pointer;
      transition: all .15s;
    }
    .btn-timer-adj:hover { border-color: var(--purple-light); background: var(--purple-pale); }

    .row2 { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }

    .btn-start {
      width: 100%;
      padding: 16px;
      background: var(--purple);
      color: #fff;
      border: none;
      border-radius: 14px;
      font-size: 17px;
      font-weight: 700;
      cursor: pointer;
      margin-top: 8px;
      transition: background .2s, transform .1s;
      letter-spacing: .01em;
    }
    .btn-start:hover { background: var(--purple-dark); transform: translateY(-1px); }
    .btn-start:active { transform: translateY(0); }

    /* ── TIMER ─────────────────────────────────────────────── */
    #timerScreen { display: none; width: 740px; }

    .board {
      background: var(--card);
      border-radius: 28px;
      border: 3px solid var(--purple);
      padding: 52px 60px 44px;
      box-shadow: 0 12px 48px rgba(91,63,166,.18);
      position: relative;
    }
    .board.finished {
      animation: pulse-glow 2s ease-in-out infinite;
    }
    @keyframes pulse-glow {
      0%,100% { box-shadow: 0 12px 48px rgba(91,63,166,.18); }
      50%      { box-shadow: 0 12px 72px rgba(91,63,166,.45); }
    }

    .board-layout {
      display: grid;
      grid-template-columns: 190px 1fr 190px;
      align-items: center;
    }

    /* left column */
    .col-left { display: flex; flex-direction: column; gap: 24px; }
    /* right column */
    .col-right { display: flex; flex-direction: column; gap: 18px; align-items: flex-end; }

    .info-block .ib-label {
      font-size: 11px;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: .1em;
      color: var(--muted);
      margin-bottom: 4px;
    }
    .info-block .ib-value {
      font-size: 24px;
      font-weight: 800;
      color: var(--purple-dark);
    }
    .col-right .info-block { text-align: right; }

    /* center column */
    .col-center {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
      padding: 0 16px;
    }

    /* Analog SVG clock */
    .svg-clock { width: 200px; height: 200px; }

    /* Digital countdown */
    .digital {
      font-size: 48px;
      font-weight: 900;
      letter-spacing: -.03em;
      color: var(--purple);
      font-variant-numeric: tabular-nums;
      line-height: 1;
    }
    .digital.warning { color: #e05; }

    .phase-name {
      font-size: 12px;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: .12em;
      color: var(--muted);
    }

    /* Phase dots */
    .phase-dots { display: flex; gap: 8px; }
    .pd {
      width: 10px; height: 10px;
      border-radius: 50%;
      background: var(--border);
      transition: background .3s;
    }
    .pd.active { background: var(--purple); }
    .pd.done   { background: var(--purple-light); }

    /* Progress bar */
    .progress-wrap {
      grid-column: 1 / -1;
      height: 6px;
      background: var(--purple-pale);
      border-radius: 3px;
      overflow: hidden;
      margin-top: 32px;
    }
    .progress-fill {
      height: 100%;
      background: var(--purple);
      border-radius: 3px;
      transition: width 1s linear;
      width: 0%;
    }

    /* Viel Erfolg */
    .viel-erfolg {
      grid-column: 1 / -1;
      text-align: center;
      font-size: 34px;
      font-weight: 800;
      font-style: italic;
      color: var(--purple);
      margin-top: 20px;
      letter-spacing: -.5px;
    }

    /* NTA badge */
    .nta-badge {
      font-size: 14px;
      font-weight: 700;
      color: var(--purple-light);
    }

    /* reset link */
    .reset-link {
      display: block;
      text-align: center;
      margin-top: 14px;
      font-size: 13px;
      color: var(--muted);
      cursor: pointer;
      background: none;
      border: none;
      font-family: inherit;
      transition: color .2s;
    }
    .reset-link:hover { color: var(--purple); }

    /* ── CONFIRMATION OVERLAY ──────────────────────────────── */
    #confirmOverlay {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(45,27,105,.55);
      backdrop-filter: blur(10px);
      align-items: center;
      justify-content: center;
      z-index: 200;
    }
    .confirm-card {
      background: var(--card);
      border-radius: 28px;
      padding: 52px 48px;
      text-align: center;
      max-width: 420px;
      width: 90%;
      box-shadow: 0 24px 80px rgba(0,0,0,.25);
    }
    .confirm-card .cc-emoji { font-size: 56px; margin-bottom: 16px; }
    .confirm-card h2 {
      font-size: 26px;
      font-weight: 800;
      color: var(--purple-dark);
      margin-bottom: 10px;
    }
    .confirm-card p { color: var(--muted); font-size: 15px; line-height: 1.6; margin-bottom: 36px; }
    .confirm-card strong { color: var(--purple-dark); }
    .btn-confirm {
      padding: 16px 52px;
      background: var(--purple);
      color: #fff;
      border: none;
      border-radius: 14px;
      font-size: 17px;
      font-weight: 700;
      cursor: pointer;
      transition: background .2s;
    }
    .btn-confirm:hover { background: var(--purple-dark); }

    /* ── FINISHED OVERLAY ──────────────────────────────────── */
    #doneOverlay {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(45,27,105,.55);
      backdrop-filter: blur(10px);
      align-items: center;
      justify-content: center;
      z-index: 200;
    }
    .done-card {
      background: var(--card);
      border-radius: 28px;
      padding: 52px 48px;
      text-align: center;
      max-width: 420px;
      width: 90%;
      box-shadow: 0 24px 80px rgba(0,0,0,.25);
    }
    .done-card .dc-emoji { font-size: 64px; margin-bottom: 16px; }
    .done-card h2 {
      font-size: 30px;
      font-weight: 900;
      color: var(--purple-dark);
      margin-bottom: 10px;
    }
    .done-card p { color: var(--muted); font-size: 15px; margin-bottom: 36px; }
    .btn-new {
      padding: 16px 40px;
      background: var(--purple);
      color: #fff;
      border: none;
      border-radius: 14px;
      font-size: 17px;
      font-weight: 700;
      cursor: pointer;
      transition: background .2s;
    }
    .btn-new:hover { background: var(--purple-dark); }
  </style>
</head>
<body>

<!-- ── SETUP SCREEN ─────────────────────────────────────── -->
<div id="setupScreen">
  <div class="setup-title">Abschlussprüfung</div>
  <div class="setup-sub">Timer konfigurieren</div>

  <div class="form-group">
    <span class="form-label">Abschlussniveau</span>
    <div class="niveau-grid">
      <div class="niveau-btn" data-niveau="Fö9" onclick="selectNiveau(this)">
        <div class="nb-name">Fö9</div>
        <div class="nb-desc">15 + 60 min</div>
      </div>
      <div class="niveau-btn" data-niveau="HS9" onclick="selectNiveau(this)">
        <div class="nb-name">HS9</div>
        <div class="nb-desc">15 + 120 min</div>
      </div>
      <div class="niveau-btn" data-niveau="IGS" onclick="selectNiveau(this)">
        <div class="nb-name">IGS</div>
        <div class="nb-desc">15 + 180 min</div>
      </div>
    </div>
  </div>

  <!-- Zeiten-Panel – erscheint nach Niveau-Auswahl -->
  <div id="zeitenPanel">
    <div class="zp-title">Zeiten anpassen</div>
    <div id="zeitenFields"></div>
  </div>

  <div class="form-group">
    <label class="form-label" for="fachInput">Fach</label>
    <input type="text" id="fachInput" placeholder="z.B. Mathematik" />
  </div>

  <div class="row2">
    <div class="form-group">
      <label class="form-label" for="startzeit">Startzeit</label>
      <input type="time" id="startzeit" value="08:00" />
    </div>
    <div class="form-group">
      <label class="form-label" for="ntaInput">NTA Zusatzzeit (min)</label>
      <input type="text" id="ntaInput" placeholder="0" style="text-align:center;" />
    </div>
  </div>

  <button class="btn-start" onclick="startTimer()">Timer starten →</button>
</div>

<!-- ── TIMER SCREEN ─────────────────────────────────────── -->
<div id="timerScreen">
  <div class="board" id="board">
    <div class="board-layout">

      <!-- Left: Fach + Niveau -->
      <div class="col-left">
        <div class="info-block">
          <div class="ib-label">Fach</div>
          <div class="ib-value" id="dFach">—</div>
        </div>
        <div class="info-block">
          <div class="ib-label">Niveau</div>
          <div class="ib-value" id="dNiveau">—</div>
        </div>
      </div>

      <!-- Center: Clock + Countdown -->
      <div class="col-center">
        <svg class="svg-clock" id="svgClock" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
          <circle cx="100" cy="100" r="95" fill="white" stroke="#5b3fa6" stroke-width="3.5"/>
          <g id="clockMarkers"></g>
          <!-- Progress ring (shows elapsed time of current phase) -->
          <circle id="ringArc" cx="100" cy="100" r="84"
            fill="none" stroke="#ede8f9" stroke-width="9"
            stroke-dasharray="527.8" stroke-dashoffset="0"
            stroke-linecap="round"
            transform="rotate(-90 100 100)"/>
          <!-- Hands (rendered above ring) -->
          <line id="hHour"   x1="100" y1="100" x2="100" y2="58"  stroke="#3d2875" stroke-width="5"   stroke-linecap="round"/>
          <line id="hMinute" x1="100" y1="100" x2="100" y2="32"  stroke="#5b3fa6" stroke-width="3"   stroke-linecap="round"/>
          <line id="hSecond" x1="100" y1="100" x2="100" y2="26"  stroke="#e05070" stroke-width="1.5" stroke-linecap="round"/>
          <circle cx="100" cy="100" r="5" fill="#5b3fa6"/>
        </svg>

        <div class="digital" id="digital">00:00</div>
        <div class="phase-name" id="phaseName">—</div>
        <div class="phase-dots" id="phaseDots"></div>
        <div class="timer-adj">
          <button class="btn-timer-adj" onclick="adjustRunning(-30)">−30s</button>
          <button class="btn-timer-adj" onclick="adjustRunning(+30)">+30s</button>
        </div>
      </div>

      <!-- Right: Beginn + Ende + NTA -->
      <div class="col-right">
        <div class="info-block">
          <div class="ib-label">Beginn</div>
          <div class="ib-value" id="dBeginn">—</div>
        </div>
        <div class="info-block">
          <div class="ib-label">Ende</div>
          <div class="ib-value" id="dEnde">—</div>
        </div>
        <div class="info-block" id="ntaBlock" style="display:none">
          <div class="ib-label">+ NTA</div>
          <div class="nta-badge" id="dNTA">—</div>
        </div>
      </div>

      <!-- Progress bar -->
      <div class="progress-wrap">
        <div class="progress-fill" id="progFill"></div>
      </div>

      <!-- Viel Erfolg -->
      <div class="viel-erfolg">Viel Erfolg!</div>
    </div>
  </div>
  <button class="reset-link" onclick="resetTimer()">↩ Neu konfigurieren</button>
</div>

<!-- ── PHASE FLASH ───────────────────────────────────────── -->
<div id="phaseFlash"></div>

<!-- ── DONE OVERLAY ──────────────────────────────────────── -->
<div id="doneOverlay">
  <div class="done-card">
    <div class="dc-emoji">🎉</div>
    <h2>Zeit abgelaufen!</h2>
    <p id="doneBody"></p>
    <button class="btn-new" onclick="resetTimer()">Neuer Timer</button>
  </div>
</div>

<script>
/* ── Config ─────────────────────────────────────────────── */
const NIVEAUS = {
  'Fö9': { phases: ['Hörverstehen', 'Schreibzeit'], mins: [15, 60]  },
  'HS9': { phases: ['Hörverstehen', 'Schreibzeit'], mins: [15, 120] },
  'IGS': { phases: ['Auswahlzeit', 'Arbeitszeit'],   mins: [15, 180] },
};

let cfg         = {};
let phaseIdx    = 0;
let secsLeft    = 0;
let totalSecs   = 0;
let ticker      = null;
let clockTicker = null;

/* ── Setup helpers ──────────────────────────────────────── */
function selectNiveau(el) {
  document.querySelectorAll('.niveau-btn').forEach(b => b.classList.remove('selected'));
  el.classList.add('selected');

  const niv    = el.dataset.niveau;
  const nData  = NIVEAUS[niv];
  const fields = document.getElementById('zeitenFields');
  const panel  = document.getElementById('zeitenPanel');

  const mkField = (label, id, mins) => `
    <div class="form-group">
      <label class="form-label">${label} (min)</label>
      <input type="text" id="${id}" value="${mins}" />
    </div>`;

  if (nData.phases.length === 2) {
    fields.innerHTML = `<div class="row2">
      ${mkField(nData.phases[0], 'p0min', nData.mins[0])}
      ${mkField(nData.phases[1], 'p1min', nData.mins[1])}
    </div>`;
  } else {
    fields.innerHTML = mkField(nData.phases[0], 'p0min', nData.mins[0]);
  }
  panel.style.display = 'block';
}

function fmtSecs(secs) {
  const m = Math.floor(secs / 60), s = secs % 60;
  return s > 0 ? `${m} min ${s} s` : `${m} min`;
}

function addMins(timeStr, mins) {
  const [h, m] = timeStr.split(':').map(Number);
  const t = h * 60 + m + mins;
  return `${String(Math.floor(t / 60) % 24).padStart(2,'0')}:${String(t % 60).padStart(2,'0')}`;
}

/* ── Init SVG clock markers ─────────────────────────────── */
(function initMarkers() {
  const g = document.getElementById('clockMarkers');
  for (let i = 0; i < 60; i++) {
    const a   = (i / 60) * 2 * Math.PI - Math.PI / 2;
    const big = i % 5 === 0;
    const r1  = big ? 78 : 82;
    const r2  = 90;
    const x1  = (100 + r1 * Math.cos(a)).toFixed(2);
    const y1  = (100 + r1 * Math.sin(a)).toFixed(2);
    const x2  = (100 + r2 * Math.cos(a)).toFixed(2);
    const y2  = (100 + r2 * Math.sin(a)).toFixed(2);
    const ln  = document.createElementNS('http://www.w3.org/2000/svg','line');
    ln.setAttribute('x1', x1); ln.setAttribute('y1', y1);
    ln.setAttribute('x2', x2); ln.setAttribute('y2', y2);
    ln.setAttribute('stroke', '#5b3fa6');
    ln.setAttribute('stroke-width', big ? '2.5' : '1');
    g.appendChild(ln);
  }
})();

/* ── Real-time clock hands ──────────────────────────────── */
function tickClock() {
  const now  = new Date();
  const h12  = now.getHours() % 12;
  const min  = now.getMinutes();
  const sec  = now.getSeconds();
  const ms   = now.getMilliseconds();

  const hDeg = (h12 / 12 + min / 720 + sec / 43200) * 360 - 90;
  const mDeg = (min / 60 + sec / 3600) * 360 - 90;
  const sDeg = (sec / 60 + ms / 60000) * 360 - 90;

  setHand('hHour',   hDeg, 42);
  setHand('hMinute', mDeg, 68);
  setHand('hSecond', sDeg, 74);
}

function setHand(id, deg, len) {
  const r = deg * Math.PI / 180;
  const x = (100 + len * Math.cos(r)).toFixed(2);
  const y = (100 + len * Math.sin(r)).toFixed(2);
  const el = document.getElementById(id);
  el.setAttribute('x2', x);
  el.setAttribute('y2', y);
}

/* ── Start timer ────────────────────────────────────────── */
function startTimer() {
  const sel = document.querySelector('.niveau-btn.selected');
  if (!sel) { alert('Bitte Abschlussniveau wählen.'); return; }

  const niveau = sel.dataset.niveau;
  const fach   = document.getElementById('fachInput').value.trim() || 'Prüfung';
  const start  = document.getElementById('startzeit').value || '08:00';
  const nta    = parseInt(document.getElementById('ntaInput').value) || 0;

  const nData  = NIVEAUS[niveau];
  const phaseSecs = nData.phases.map((_, i) => {
    const el = document.getElementById(`p${i}min`);
    const val = parseInt(el?.value);
    return (isNaN(val) || val <= 0) ? nData.mins[i] * 60 : val * 60;
  });
  phaseSecs[phaseSecs.length - 1] += nta * 60;     // NTA on last phase
  const totalMins = phaseSecs.reduce((a, b) => a + b, 0) / 60;
  const end = addMins(start, totalMins);

  cfg = { fach, niveau, start, end, nta, phases: nData.phases, phaseSecs };

  // Display
  document.getElementById('dFach').textContent   = fach;
  document.getElementById('dNiveau').textContent = niveau;
  document.getElementById('dBeginn').textContent = start;
  document.getElementById('dEnde').textContent   = end;

  if (nta > 0) {
    document.getElementById('ntaBlock').style.display = '';
    document.getElementById('dNTA').textContent = `${nta} min`;
  } else {
    document.getElementById('ntaBlock').style.display = 'none';
  }

  // Phase dots
  const dotsEl = document.getElementById('phaseDots');
  dotsEl.innerHTML = '';
  cfg.phases.forEach((_, i) => {
    const d = document.createElement('div');
    d.className = 'pd';
    d.id = `pd${i}`;
    dotsEl.appendChild(d);
  });

  document.getElementById('setupScreen').style.display  = 'none';
  document.getElementById('timerScreen').style.display  = 'block';

  clockTicker = setInterval(tickClock, 100);
  tickClock();

  phaseIdx = 0;
  runPhase(0);
}

/* ── Run phase ──────────────────────────────────────────── */
function runPhase(idx) {
  phaseIdx  = idx;
  totalSecs = cfg.phaseSecs[idx];
  secsLeft  = totalSecs;

  document.getElementById('phaseName').textContent = cfg.phases[idx];
  updateDots();
  clearInterval(ticker);
  ticker = setInterval(tick, 1000);
  render();
}

function tick() {
  if (secsLeft > 0) {
    secsLeft--;
    render();
  } else {
    clearInterval(ticker);
    onPhaseEnd();
  }
}

/* ── Render countdown + ring ────────────────────────────── */
function render() {
  const m  = Math.floor(secsLeft / 60);
  const s  = secsLeft % 60;
  const el = document.getElementById('digital');
  el.textContent = `${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
  el.classList.toggle('warning', secsLeft <= 300 && secsLeft > 0);

  const pct  = 1 - secsLeft / totalSecs;
  const circ = 2 * Math.PI * 84;                   // r=84 → 527.8
  const arc  = document.getElementById('ringArc');
  arc.setAttribute('stroke-dasharray',  circ.toFixed(1));
  arc.setAttribute('stroke-dashoffset', (circ * (1 - pct)).toFixed(1));
  arc.setAttribute('stroke', secsLeft <= 300 ? '#e05070' : '#5b3fa6');

  document.getElementById('progFill').style.width = `${pct * 100}%`;
}

/* ── Phase end ──────────────────────────────────────────── */
function onPhaseEnd() {
  const next = phaseIdx + 1;
  if (next < cfg.phases.length) {
    const flash = document.getElementById('phaseFlash');
    flash.textContent = `${cfg.phases[phaseIdx]} beendet — ${cfg.phases[next]} beginnt`;
    flash.style.display = 'block';
    setTimeout(() => { flash.style.display = 'none'; }, 3000);
    runPhase(next);
  } else {
    document.getElementById('doneBody').innerHTML =
      `<strong>${cfg.fach}</strong> – Prüfungszeit beendet.<br>Ende: ${cfg.end} Uhr`;
    document.getElementById('doneOverlay').style.display = 'flex';
    document.getElementById('board').classList.add('finished');
    document.getElementById('digital').textContent = '00:00';
    document.getElementById('progFill').style.width = '100%';
    updateDots();
  }
}

function adjustRunning(delta) {
  secsLeft = Math.max(1, secsLeft + delta);
  const [h, m] = cfg.end.split(':').map(Number);
  let endSecs = h * 3600 + m * 60 + delta;
  const newH = Math.floor(endSecs / 3600) % 24;
  const newM = Math.floor((endSecs % 3600) / 60);
  cfg.end = `${String(newH).padStart(2,'0')}:${String(newM).padStart(2,'0')}`;
  document.getElementById('dEnde').textContent = cfg.end;
  render();
}

/* ── Dots ───────────────────────────────────────────────── */
function updateDots() {
  cfg.phases.forEach((_, i) => {
    const d = document.getElementById(`pd${i}`);
    if (!d) return;
    const done = i < phaseIdx || (i === phaseIdx && secsLeft === 0);
    d.className = 'pd' + (done ? ' done' : i === phaseIdx ? ' active' : '');
  });
}

/* ── Reset ──────────────────────────────────────────────── */
function resetTimer() {
  clearInterval(ticker);
  clearInterval(clockTicker);
  document.getElementById('doneOverlay').style.display    = 'none';
  document.getElementById('board').classList.remove('finished');
  document.getElementById('timerScreen').style.display    = 'none';
  document.getElementById('setupScreen').style.display    = '';
  document.getElementById('zeitenPanel').style.display    = 'none';
  document.querySelectorAll('.niveau-btn').forEach(b => b.classList.remove('selected'));
}
</script>
</body>
</html>
