# tomateveneco.github.io
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Sorpresa de cumpleaños</title>
<style>
:root{--bg:#fdf6e3;--fg:#2b2b2b;--card:#fff;--accent:#e8590c;color-scheme:light dark;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#1c1b22;--fg:#f1efe9;--card:#2a2933;--accent:#ff922b}}
:root[data-theme="dark"]{--bg:#1c1b22;--fg:#f1efe9;--card:#2a2933;--accent:#ff922b}
html,body{margin:0;height:100%}
body{background:var(--bg);color:var(--fg);font-family:system-ui,sans-serif;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:8px;padding:8px;box-sizing:border-box;user-select:none;-webkit-user-select:none}
h1{font-size:1.3rem;margin:0;text-align:center}
.dot{display:inline-block;width:.9em;height:.9em;border-radius:50%;background:#ffd43b;border:2px solid #f59f00;vertical-align:-.1em;box-sizing:border-box}
canvas{max-width:100%;height:auto;border-radius:12px;box-shadow:0 4px 16px #0004}
.ctl{display:flex;justify-content:space-between;width:100%;max-width:800px;gap:8px}
.ctl div{display:flex;gap:8px}
button{background:var(--accent);color:#fff;border:0;border-radius:12px;padding:12px 20px;font-size:1.1rem;cursor:pointer;touch-action:none}
#m{position:fixed;inset:0;background:#000a;display:none;align-items:center;justify-content:center;padding:16px}
#m div{position:relative;background:var(--card);color:var(--fg);max-width:420px;width:100%;border-radius:16px;padding:20px;text-align:center;max-height:90%;overflow:auto}
#m img{max-width:100%;max-height:50vh;border-radius:10px}
#mx{position:absolute;top:8px;right:8px;padding:4px 10px;font-size:1rem;background:transparent;color:var(--fg);border:2px solid currentColor;border-radius:50%;touch-action:auto}
</style>
</head>
<body>
<h1 id="t"></h1>
<canvas id="c" width="800" height="480"></canvas>
<div class="ctl"><div><button id="bl">◀</button><button id="br">▶</button></div><div><button id="bo">Abrir (E)</button><button id="bj">⬆</button></div></div>
<div id="m"><div><button id="mx" aria-label="Cerrar">✕</button><h2 id="mt"></h2><img id="mi" alt=""><p id="mp"></p><button id="mb" style="touch-action:auto">¡Genial!</button></div></div>
<script>
// ====== PERSONALIZA AQUÍ ======
const CONFIG = {
  titulo: "¡Feliz cumpleaños, mi amor! 🎂",
  finalTitulo: "¡Feliz cumpleaños, mi amor! 🎉",
  finalTexto: "You made it babeeesss, ojalá te haya gustado este pequeño regalo, hecho con mucho cariño 💖",
  cofres: [
    { monedas: 10, titulo: "Cofre 1", texto: "Aquí va tu primer mensaje de cumpleaños.", foto: "" },
    { monedas: 20, titulo: "Cofre 2", texto: "Aquí va el segundo mensaje o recuerdo.",   foto: "" },
    { monedas: 30, titulo: "Cofre 3", texto: "¡La gran sorpresa final!",                 foto: "" }
  ]
};
// foto: debe ir incrustada (data:image/jpeg;base64,...) porque la página no carga imágenes externas
// ==============================
document.getElementById('t').textContent = CONFIG.titulo;
const cv = document.getElementById('c'), ctx = cv.getContext('2d'), W = 800, H = 480, TOP = 170;
const plats = [
  { x: 0, y: 440, w: 800, h: 40, c: 6 },
  { x: 200, y: 350, w: 120, h: 16, c: 3 }, { x: 480, y: 350, w: 120, h: 16, c: 3 },
  { x: 40, y: 260, w: 140, h: 16, c: 4 }, { x: 620, y: 260, w: 140, h: 16, c: 4 },
  { x: 220, y: TOP, w: 360, h: 16, c: 0 }
];
const COIN_DEF = [
  [80, 410], [150, 410], [230, 410], [470, 410], [530, 410], [740, 410], [775, 410],
  [230, 320], [260, 320], [290, 320], [493, 320], [588, 320], [400, 240],
  [70, 230], [110, 230], [150, 230], [650, 230], [690, 230], [730, 230],
  [250, 70], [350, 70], [450, 70], [550, 70], [240, 140], [560, 140],
  [175, 330], [620, 330], [200, 215], [600, 215], [110, 335]
];
const coins = COIN_DEF.map(([x, y]) => ({ x, y }));
const chests = CONFIG.cofres.map((c, i) => ({ ...c, x: 300 + i * 100, open: false }));
const P = { x: 40, y: 380, w: 26, h: 32, vx: 0, vy: 0, on: false };
let got = 0, modal = false, msg = '', lives = 3, inv = 0, onClose = null;
const spikes = [{ x: 330, y: 440, w: 60 }, { x: 640, y: 440, w: 48 }, { x: 520, y: 350, w: 36 }];
const enemies = [{ x: 430, y: 440, a: 430, b: 580, d: 1, alive: true }, { x: 130, y: 440, a: 130, b: 260, d: 1, alive: true }, { x: 630, y: 260, a: 625, b: 730, d: 1, alive: true }, { x: 45, y: 260, a: 45, b: 148, d: 1, alive: true }];
const k = {}, T = {};
addEventListener('keydown', e => {
  const n = e.key.toLowerCase();
  k[n] = 1;
  if (['arrowup', 'arrowdown', 'arrowleft', 'arrowright', ' '].includes(n)) e.preventDefault();
  if (n === 'e' || n === 'enter') act();
});
addEventListener('keyup', e => { k[e.key.toLowerCase()] = 0; });
[['bl', 'l'], ['br', 'r'], ['bj', 'j']].forEach(([id, f]) => {
  const b = document.getElementById(id);
  b.onpointerdown = e => { e.preventDefault(); T[f] = 1; };
  ['pointerup', 'pointerleave', 'pointercancel'].forEach(ev => b.addEventListener(ev, () => { T[f] = 0; }));
});
document.getElementById('bo').onclick = act;
let AC, songBus = null;
function stopSong() { try { if (songBus) { songBus.gain.setValueAtTime(0, AC.currentTime); songBus.disconnect(); songBus = null; } } catch (e) {} }
function audio() {
  if (!AC) AC = new (window.AudioContext || window.webkitAudioContext)();
  if (AC.state === 'suspended') AC.resume();
  return AC;
}
['pointerdown', 'keydown'].forEach(ev => addEventListener(ev, () => { try { audio(); } catch (e) {} }));
function tone(f, t0, d, type, v, f2, out) {
  try {
    const a = audio(), o = a.createOscillator(), g = a.createGain(), t = a.currentTime + t0;
    o.type = type || 'square'; o.frequency.setValueAtTime(f, t);
    if (f2) o.frequency.exponentialRampToValueAtTime(f2, t + d);
    g.gain.setValueAtTime(v || .12, t); g.gain.exponentialRampToValueAtTime(.001, t + d);
    o.connect(g); g.connect(out || a.destination); o.start(t); o.stop(t + d + .02);
  } catch (e) {}
}
const sfx = {
  coin() { tone(988, 0, .08); tone(1319, .08, .35); },
  chest() { [523, 659, 784, 1047, 1319].forEach((f, i) => tone(f, i * .11, i === 4 ? .5 : .12, 'square', .1)); },
  hurt() { tone(400, 0, .35, 'sawtooth', .15, 90); },
  stomp() { tone(520, 0, .05); tone(260, .05, .14, 'square', .12, 110); },
  song() {
    stopSong();
    try {
      const a = audio(); songBus = a.createGain(); songBus.connect(a.destination);
      let t = .2;
      [[392, .75], [392, .25], [440, 1], [392, 1], [523, 1], [494, 2],
       [392, .75], [392, .25], [440, 1], [392, 1], [587, 1], [523, 2],
       [392, .75], [392, .25], [784, 1], [659, 1], [523, 1], [494, 1], [440, 2],
       [698, .75], [698, .25], [659, 1], [523, 1], [587, 1], [523, 2]].forEach(([f, b]) => {
        tone(f, t, b * .34, 'square', .09, 0, songBus);
        tone(f / 2, t, b * .34, 'triangle', .1, 0, songBus);
        t += b * .36;
      });
    } catch (e) {}
  }
};
function restart() {
  stopSong();
  got = 0; lives = 3; inv = 0; msg = '';
  coins.length = 0; COIN_DEF.forEach(([x, y]) => coins.push({ x, y }));
  chests.forEach(c => { c.open = false; });
  enemies.forEach(e => { e.x = e.a; e.d = 1; e.alive = true; });
  P.x = 40; P.y = 380; P.vx = P.vy = 0;
}
function closeModal() { document.getElementById('m').style.display = 'none'; modal = false; if (onClose) { const f = onClose; onClose = null; f(); } }
document.getElementById('mb').onclick = closeModal;
document.getElementById('mx').onclick = closeModal;
function showModal(title, text, foto, btn, cb, noX) {
  modal = true; onClose = cb || null;
  document.getElementById('mx').style.display = noX ? 'none' : '';
  document.getElementById('mt').textContent = title;
  document.getElementById('mp').textContent = text;
  document.getElementById('mb').textContent = btn;
  const im = document.getElementById('mi');
  im.style.display = foto ? 'inline' : 'none';
  if (foto) im.src = foto;
  document.getElementById('m').style.display = 'flex';
}
function nearChest() {
  return chests.find(c => Math.abs(P.x + P.w / 2 - c.x) < 40 && Math.abs(P.y + P.h - TOP) < 6);
}
function act() {
  if (modal) return;
  const c = nearChest();
  if (!c) return;
  if (c.open || got >= c.monedas) {
    const nuevo = !c.open;
    c.open = true; msg = ''; sfx.chest();
    showModal(c.titulo, c.texto, c.foto, '¡Genial!', () => {
      if (nuevo && chests.every(x => x.open))
        { showModal(CONFIG.finalTitulo, CONFIG.finalTexto, '', 'Volver a empezar', restart, true); sfx.song(); }
    });
  } else msg = 'Te faltan ' + (c.monedas - got) + ' velas';
}
const hit = (a, b) => a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
function hurt() {
  if (inv > 0) return;
  lives = Math.max(0, lives - 1); inv = 110; P.vy = -8; P.on = false; sfx.hurt();
  if (lives === 0) showModal('¡Ay! 💔', 'Orrr noorrrrr, te has quedado sin vidas, pero no pasa nada: tus velas se quedan contigo. ¡Inténtalo otra vez!', '', 'Reintentar', () => { lives = 3; inv = 120; P.x = 40; P.y = 380; P.vx = P.vy = 0; });
}
function update() {
  if (modal) return;
  const l = k.arrowleft || k.a || T.l, r = k.arrowright || k.d || T.r;
  P.vx = ((r ? 1 : 0) - (l ? 1 : 0)) * 4;
  if ((k.arrowup || k.w || k[' '] || T.j) && P.on) { P.vy = -12.5; P.on = false; }
  P.vy += 0.6;
  P.x = Math.max(0, Math.min(W - P.w, P.x + P.vx));
  plats.forEach(q => { if (hit(P, q)) P.x = P.vx > 0 ? q.x - P.w : q.x + q.w; });
  P.y += P.vy; P.on = false;
  plats.forEach(q => {
    if (hit(P, q)) {
      if (P.vy > 0) { P.y = q.y - P.h; P.on = true; } else P.y = q.y + q.h;
      P.vy = 0;
    }
  });
  for (let i = coins.length - 1; i >= 0; i--)
    if (Math.hypot(coins[i].x - (P.x + 13), coins[i].y - (P.y + 16)) < 24) { coins.splice(i, 1); got++; sfx.coin(); }
  if (inv > 0) inv--;
  spikes.forEach(s => { if (hit(P, { x: s.x + 4, y: s.y - 12, w: s.w - 8, h: 12 })) hurt(); });
  enemies.forEach(e => {
    if (!e.alive) return;
    e.x += e.d * 1.2; if (e.x < e.a || e.x > e.b) e.d *= -1;
    const r = { x: e.x, y: e.y - 24, w: 28, h: 24 };
    if (hit(P, r)) {
      if (P.vy > 0 && P.y + P.h - P.vy <= r.y + 8) { e.alive = false; P.vy = -9; sfx.stomp(); } else hurt();
    }
  });
  if (nearChest() === undefined && msg.startsWith('Te')) msg = '';
}
function drawChest(c) {
  const x = c.x - 22, y = TOP - 26;
  ctx.fillStyle = '#a0522d'; ctx.fillRect(x, y + 12, 44, 14);
  ctx.fillStyle = '#8b4513'; ctx.fillRect(x, c.open ? y - 14 : y, 44, 12);
  ctx.fillStyle = '#ffd43b'; ctx.fillRect(x, y + 10, 44, 4); ctx.fillRect(c.x - 4, y + 8, 8, 10);
}
function drawCake(cx, by) {
  let y = by;
  [[210, 52, '#f06595'], [150, 46, '#ffe3a3'], [96, 40, '#f783ac']].forEach(([w, h, col]) => {
    y -= h;
    ctx.fillStyle = col; ctx.fillRect(cx - w / 2, y, w, h);
    ctx.fillStyle = '#fff'; ctx.fillRect(cx - w / 2, y, w, 8);
    for (let x = cx - w / 2 + 7; x < cx + w / 2; x += 14) { ctx.beginPath(); ctx.arc(x, y + 8, 7, 0, 7); ctx.fill(); }
    ctx.fillStyle = '#ffd43b';
    for (let x = cx - w / 2 + 20; x < cx + w / 2 - 10; x += 30) ctx.fillRect(x, y + h / 2 + 4, 6, 6);
  });
  const t = Date.now() / 120;
  for (let i = 0; i < 5; i++) {
    const x = cx - 36 + i * 18, f = 7 + Math.sin(t + i * 2) * 2;
    ctx.fillStyle = i % 2 ? '#74c0fc' : '#ffffff'; ctx.fillRect(x - 3, y - 24, 6, 24);
    ctx.fillStyle = 'rgba(255,220,120,.35)'; ctx.beginPath(); ctx.arc(x, y - 32, 12, 0, 7); ctx.fill();
    ctx.fillStyle = '#ffa94d'; ctx.beginPath(); ctx.ellipse(x, y - 31, 4, f, 0, 0, 7); ctx.fill();
    ctx.fillStyle = '#ffec99'; ctx.beginPath(); ctx.ellipse(x, y - 29, 2, f * .55, 0, 0, 7); ctx.fill();
  }
}
function pill(x, y, w, h) {
  const r = h / 2;
  ctx.beginPath(); ctx.moveTo(x + r, y);
  ctx.arcTo(x + w, y, x + w, y + h, r); ctx.arcTo(x + w, y + h, x, y + h, r);
  ctx.arcTo(x, y + h, x, y, r); ctx.arcTo(x, y, x + w, y, r); ctx.closePath();
}
function drawHud() {
  const abiertos = chests.filter(c => c.open).length;
  ctx.save();
  ctx.textBaseline = 'middle'; ctx.textAlign = 'left';
  ctx.fillStyle = 'rgba(40,20,60,.6)'; pill(10, 10, 272, 38); ctx.fill();
  ctx.font = '20px serif';
  for (let i = 0; i < 3; i++) ctx.fillText(i < lives ? '❤️' : '🖤', 22 + i * 26, 30);
  ctx.font = 'bold 18px system-ui'; ctx.fillStyle = '#fff';
  ctx.fillText('🕯️ ' + got + '/30', 112, 30);
  ctx.fillText('🎁 ' + abiertos + '/3', 202, 30);
  const m = abiertos === 3 ? '🎉 ¡Todas las sorpresas!' : msg;
  if (m) {
    ctx.font = 'bold 16px system-ui';
    const w = ctx.measureText(m).width + 32;
    ctx.fillStyle = 'rgba(40,20,60,.7)'; pill(10, 56, w, 30); ctx.fill();
    ctx.fillStyle = '#fff'; ctx.fillText(m, 26, 71);
  }
  ctx.restore();
}
function cloud(x, y, s) {
  const pf = [[0, 0, 28], [30, -12, 34], [66, -4, 30], [98, 6, 24], [-26, 8, 22]];
  const dot = (a, b, r, col) => { ctx.fillStyle = col; ctx.beginPath(); ctx.arc(x + a * s, y + b * s, r * s, 0, 7); ctx.fill(); };
  pf.forEach(([a, b, r]) => dot(a, b + 6, r, 'rgba(122,62,120,.55)'));
  pf.forEach(([a, b, r]) => dot(a, b, r * .9, 'rgba(255,170,140,.92)'));
  pf.forEach(([a, b, r]) => dot(a - 4, b - 6, r * .55, 'rgba(255,232,200,.9)'));
}
function draw() {
  const g = ctx.createLinearGradient(0, 0, 0, H);
  [[0, '#2f2a6b'], [.3, '#8e3f87'], [.55, '#e0567a'], [.78, '#ff8f5a'], [1, '#ffd08a']].forEach(([o, c]) => g.addColorStop(o, c));
  ctx.fillStyle = g; ctx.fillRect(0, 0, W, H);
  ctx.fillStyle = 'rgba(255,255,255,.7)';
  for (let i = 0; i < 14; i++) ctx.fillRect((i * 131) % W, (i * 37) % 110, 2, 2);
  const sg = ctx.createRadialGradient(400, 430, 10, 400, 430, 220);
  sg.addColorStop(0, 'rgba(255,230,150,.9)'); sg.addColorStop(1, 'rgba(255,230,150,0)');
  ctx.fillStyle = sg; ctx.fillRect(0, 210, W, 270);
  ctx.globalAlpha = .42; drawCake(400, 440); ctx.globalAlpha = 1;
  const hz = ctx.createLinearGradient(0, 250, 0, 440);
  hz.addColorStop(0, 'rgba(255,143,90,0)'); hz.addColorStop(1, 'rgba(255,208,138,.6)');
  ctx.fillStyle = hz; ctx.fillRect(0, 250, W, 190);
  const now = Date.now() / 1000;
  [[60, 70, 1, 6], [520, 120, 1.3, 4], [300, 45, .8, 8], [700, 200, 1, 5]].forEach(([x, y, s, v]) =>
    cloud((x + now * v) % (W + 260) - 130, y, s));
  plats.forEach(q => {
    ctx.fillStyle = '#e6b36a'; ctx.fillRect(q.x, q.y, q.w, q.h);
    ctx.fillStyle = '#c98f45'; ctx.fillRect(q.x, q.y + q.h - 3, q.w, 3);
    for (let x = q.x + 14; q.h > 20 && x < q.x + q.w - 6; x += 23) ctx.fillRect(x, q.y + 16 + (x * 7) % 14, 5, 3);
    ctx.fillStyle = '#ffc9dd'; ctx.fillRect(q.x, q.y, q.w, 7);
    for (let x = q.x + 6; x < q.x + q.w - 5; x += 12) { ctx.beginPath(); ctx.arc(x, q.y + 7, 6, 0, 7); ctx.fill(); }
    const sp = ['#ff6b6b', '#4dabf7', '#ffd43b', '#69db7c', '#b197fc'];
    for (let i = 0; i < q.w / 13; i++) { ctx.fillStyle = sp[i % 5]; ctx.fillRect(q.x + 4 + i * 13 + (i * 5) % 4, q.y + 1 + (i * 3) % 5, 4, 2); }
  });
  const t = Date.now() / 300;
  const cols = ['#ff8fab', '#74c0fc', '#ffd43b', '#b197fc', '#8ce99a'];
  coins.forEach((c, i) => {
    const y = c.y - 4 + Math.sin(t + i) * 2, f = 5 + Math.sin(t * 3 + i * 2);
    ctx.fillStyle = cols[i % 5]; ctx.fillRect(c.x - 4, y - 2, 8, 16);
    ctx.fillStyle = 'rgba(255,255,255,.75)'; ctx.fillRect(c.x - 4, y + 2, 8, 3); ctx.fillRect(c.x - 4, y + 8, 8, 3);
    ctx.fillStyle = 'rgba(255,220,120,.35)'; ctx.beginPath(); ctx.arc(c.x, y - 9, 10, 0, 7); ctx.fill();
    ctx.fillStyle = '#ffa94d'; ctx.beginPath(); ctx.ellipse(c.x, y - 8, 3.5, f, 0, 0, 7); ctx.fill();
    ctx.fillStyle = '#ffec99'; ctx.beginPath(); ctx.ellipse(c.x, y - 7, 1.8, f * .55, 0, 0, 7); ctx.fill();
  });
  ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  ctx.fillStyle = '#d5dbe1'; ctx.strokeStyle = '#4a5560'; ctx.lineWidth = 1.5;
  spikes.forEach(s => { for (let i = 0; i < s.w / 12; i++) { const x = s.x + i * 12; ctx.beginPath(); ctx.moveTo(x, s.y); ctx.lineTo(x + 6, s.y - 16); ctx.lineTo(x + 12, s.y); ctx.closePath(); ctx.fill(); ctx.stroke(); } });
  ctx.font = '28px serif';
  enemies.forEach(e => { if (e.alive) ctx.fillText('🦐', e.x + 14, e.y - 12); });
  const n = nearChest();
  chests.forEach(c => {
    drawChest(c);
    ctx.font = 'bold 15px system-ui'; ctx.fillStyle = '#fff'; ctx.strokeStyle = '#2b1b3dcc'; ctx.lineWidth = 4; ctx.lineJoin = 'round';
    const lab = c.open ? '✨ abierto ✨' : got >= c.monedas ? '🔓 listo' : '🔒 ' + c.monedas + ' velas';
    ctx.strokeText(lab, c.x, TOP - 44); ctx.fillText(lab, c.x, TOP - 44);
    if (n === c && !c.open && got >= c.monedas) { ctx.strokeText('Pulsa E', c.x, TOP - 62); ctx.fillText('Pulsa E', c.x, TOP - 62); }
  });
  if (inv % 12 < 7) { ctx.font = '30px serif'; ctx.fillText('🥳', P.x + 13, P.y + 18); }
  drawHud();
}
(function loop() { update(); draw(); requestAnimationFrame(loop); })();
showModal('¡Hola, mi amor! 💖', 'Reúne todas las velas y descubre la sorpresa dentro de los cofres <3', '', '¡A jugar!');
</script>
</body>
</html>
