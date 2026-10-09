---
layout: page
title: About
permalink: /about/
---

<div class="about-me">
  <a href="https://github.com/and" aria-label="ānand on GitHub"><img src="https://avatars.githubusercontent.com/u/24557?v=4&amp;s=176" alt="ānand" width="88" height="88"></a>
  <div>
    <p class="about-name">ānand</p>
    <p class="about-link"><a href="https://github.com/and">github.com/and</a></p>
  </div>
</div>

This is my log book, where I write up what I'm learning.

## Play me at Dots and Boxes

Take turns joining two dots. Close a box to win it and go again. I'm blue.

<div class="dab">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Caveat:wght@400;700&display=swap">
  <style>
    /* pencil on graph paper: graphite ink, two coloured pencils, hand-drawn edges */
    .dab { --dab-ink:#3a3a3a; --dab-soft:#9a958c; --dab-you:#c8551b; --dab-cpu:#1f68b5; --dab-paper:#fffdf7; --dab-grid:rgba(60,110,180,.09);
           display:flex; flex-direction:column; align-items:center; gap:10px;
           margin:8px 0 24px; padding:22px 16px 18px; color:var(--dab-ink);
           font-family:"Caveat","Segoe Print","Comic Sans MS",cursive; font-size:23px; line-height:1.2;
           background-color:var(--dab-paper);
           background-image:linear-gradient(var(--dab-grid) 1px,transparent 1px),linear-gradient(90deg,var(--dab-grid) 1px,transparent 1px);
           background-size:20px 20px;
           border:2px solid var(--dab-ink); border-radius:255px 18px 225px 18px/18px 225px 18px 255px; }
    .dab p { margin:0; }
    .dab-bar { display:flex; gap:18px; align-items:center; flex-wrap:wrap; justify-content:center; }
    .dab-name { display:flex; align-items:baseline; gap:8px; }
    .dab input, .dab select, .dab button { font:inherit; color:var(--dab-ink); background:transparent; }
    .dab input { width:8em; padding:0 4px; border:0; border-bottom:2px solid var(--dab-ink); border-radius:0 0 40px 6px/0 0 4px 3px; }
    .dab input:focus { outline:none; border-bottom-color:var(--dab-you); }
    .dab select, .dab button { padding:2px 14px; border:2px solid var(--dab-ink); border-radius:14px 4px 12px 5px/5px 12px 4px 14px; }
    .dab button { cursor:pointer; transition:transform .15s; }
    .dab button:hover { transform:rotate(-2deg); }
    .dab select:focus-visible, .dab button:focus-visible { outline:2px dashed var(--dab-you); outline-offset:3px; }
    .dab-score b { font-size:1.5em; font-weight:700; }
    .dab-you { color:var(--dab-you); } .dab-cpu { color:var(--dab-cpu); }
    .dab-status { min-height:1.3em; font-weight:700; }
    .dab svg { width:min(84vw,340px); height:auto; touch-action:manipulation; overflow:visible; }
    .dab .edge, .dab .edge2 { fill:none; stroke-linecap:round; }
    .dab .edge.free { stroke:var(--dab-soft); stroke-width:2.2; stroke-dasharray:1 8; }
    .dab .edge.free.hover { stroke:var(--dab-ink); stroke-width:3; stroke-dasharray:none; opacity:.45; }
    .dab .edge.you { stroke:var(--dab-you); stroke-width:4.5; } .dab .edge.cpu { stroke:var(--dab-cpu); stroke-width:4.5; }
    .dab .edge.last { stroke-width:6.5; }
    .dab .edge2 { stroke-width:1.6; opacity:.55; } .dab .edge2.you { stroke:var(--dab-you); } .dab .edge2.cpu { stroke:var(--dab-cpu); }
    .dab .box { opacity:.8; }
    .dab .lbl { font:700 56px "Caveat","Segoe Print",cursive; text-anchor:middle; dominant-baseline:central; pointer-events:none; }
    .dab .lbl.you { fill:var(--dab-you); } .dab .lbl.cpu { fill:var(--dab-cpu); }
    .dab .dot { fill:var(--dab-ink); }
  </style>
  <label class="dab-name">Your name <input id="dab-name" maxlength="20" placeholder="You" autocomplete="off"></label>
  <div class="dab-bar">
    <span class="dab-score dab-you"><span id="dab-you-name">You</span> <b id="dab-sy">0</b></span>
    <span class="dab-score dab-cpu">ānand <b id="dab-sc">0</b></span>
    <select id="dab-level" aria-label="Difficulty">
      <option value="hard">Hard (perfect play)</option>
      <option value="easy">Easy (greedy)</option>
    </select>
    <button id="dab-reset" type="button">New game</button>
  </div>
  <div class="dab-status" id="dab-status"></div>
  <svg id="dab-board" viewBox="0 0 280 280" role="img" aria-label="Dots and boxes board"></svg>
</div>

<script>
(function(){
/* ---------- Board model ----------
   3x3 dots -> 2x2 boxes -> 12 edges, stored as bits of a 12-bit mask.
   Horizontal edge H(r,c): bit r*2+c      (r 0..2, c 0..1)  -> bits 0..5
   Vertical   edge V(r,c): bit 6+r*3+c    (r 0..1, c 0..2)  -> bits 6..11   */
const H = (r,c) => r*2+c, V = (r,c) => 6+r*3+c;
const BOX_MASKS = [];                       // bitmask of the 4 sides of each box
for (let r=0;r<2;r++) for (let c=0;c<2;c++)
  BOX_MASKS.push([H(r,c),H(r+1,c),V(r,c),V(r,c+1)].reduce((m,e)=>m|1<<e,0));
const FULL = (1<<12)-1;

// How many boxes does drawing edge e close, given the current mask?
function closed(mask, e) {
  const after = mask | 1<<e; let n = 0;
  for (const bm of BOX_MASKS) if ((bm>>e & 1) && (after & bm) === bm) n++;
  return n;
}

/* ---------- Computer: perfect play via memoised negamax ----------
   best(mask) = best (my boxes - opponent boxes) the player to move can still
   achieve from this position. Only 2^12 = 4096 positions, so we solve it fully.
   Closing a box keeps the turn (add gain, don't flip sign); otherwise the turn
   passes (flip sign).                                                       */
const memo = new Map();
function best(mask) {
  if (mask === FULL) return 0;
  if (memo.has(mask)) return memo.get(mask);
  let v = -99;
  for (let e=0;e<12;e++) if (!(mask>>e & 1)) {
    const g = closed(mask,e), nm = mask | 1<<e;
    v = Math.max(v, g ? g + best(nm) : -best(nm));
  }
  memo.set(mask, v); return v;
}
function moveValue(mask, e) {
  const g = closed(mask,e), nm = mask | 1<<e;
  return g ? g + best(nm) : -best(nm);
}
const free = mask => [...Array(12).keys()].filter(e => !(mask>>e & 1));
const pick = a => a[Math.floor(Math.random()*a.length)];

function hardMove(mask) {
  const fm = free(mask), vals = fm.map(e => moveValue(mask,e)), top = Math.max(...vals);
  return pick(fm.filter((e,i) => vals[i] === top));        // random among equally good moves
}
// Easy: take a box if possible; else avoid giving the opponent a 3-sided box; else random.
function easyMove(mask) {
  const fm = free(mask), take = fm.filter(e => closed(mask,e));
  if (take.length) return pick(take);
  const safe = fm.filter(e => !BOX_MASKS.some(bm => (bm>>e & 1) && popcount((mask|1<<e) & bm) === 3));
  return pick(safe.length ? safe : fm);
}
const popcount = n => { let c=0; while(n){c+=n&1;n>>=1;} return c; };

/* ---------- UI / game flow ---------- */
const svg = document.getElementById('dab-board'), NS = 'http://www.w3.org/2000/svg';
const $ = id => document.getElementById(id);
let mask, owner, boxOwner, turn, over, lastEdge, busy;

const dotXY = (r,c) => [40+100*c, 40+100*r];
function edgeEnds(e) {
  if (e < 6) { const r=Math.floor(e/2), c=e%2; return [dotXY(r,c), dotXY(r,c+1)]; }
  const k=e-6, r=Math.floor(k/3), c=k%3; return [dotXY(r,c), dotXY(r+1,c)];
}
function mk(tag, attrs) { const el=document.createElementNS(NS,tag); for (const k in attrs) el.setAttribute(k,attrs[k]); return el; }

// Pencil look: a wobble filter with paper grain, hatched boxes, and each line drawn twice.
const DEFS = '<defs>' +
  '<filter id="dab-pencil" x="-10%" y="-10%" width="120%" height="120%">' +
    '<feTurbulence type="fractalNoise" baseFrequency="0.04" numOctaves="2" seed="3" result="warp"/>' +
    '<feDisplacementMap in="SourceGraphic" in2="warp" scale="3" xChannelSelector="R" yChannelSelector="G" result="wobbly"/>' +
    '<feTurbulence type="fractalNoise" baseFrequency="1.1" numOctaves="1" seed="7" result="grain"/>' +
    '<feColorMatrix in="grain" type="matrix" values="0 0 0 0 0  0 0 0 0 0  0 0 0 0 0  -1.4 0 0 0 1.35" result="grainAlpha"/>' +
    '<feComposite in="wobbly" in2="grainAlpha" operator="in"/>' +
  '</filter>' +
  '<pattern id="dab-hatch-you" width="7" height="7" patternUnits="userSpaceOnUse" patternTransform="rotate(35)"><line x1="0" y1="0" x2="0" y2="7" stroke="#c8551b" stroke-width="1.7"/></pattern>' +
  '<pattern id="dab-hatch-cpu" width="7" height="7" patternUnits="userSpaceOnUse" patternTransform="rotate(-35)"><line x1="0" y1="0" x2="0" y2="7" stroke="#1f68b5" stroke-width="1.7"/></pattern>' +
  '</defs>';

// A slightly bent stroke. The bend is fixed per edge so lines don't jump between renders.
function sketch(seed, x1, y1, x2, y2) {
  const s = Math.sin(seed * 12.9898) * 43758.5453, j = s - Math.floor(s) - 0.5;
  const mx = (x1+x2)/2, my = (y1+y2)/2, horiz = y1 === y2;
  const cx = mx + j*(horiz ? 4 : 7), cy = my + j*(horiz ? 7 : 4);
  return 'M'+(x1+j*3)+' '+(y1-j*2)+' Q'+cx+' '+cy+' '+(x2-j*2)+' '+(y2+j*3);
}

function render() {
  svg.innerHTML = DEFS;
  const ink = mk('g', {filter:'url(#dab-pencil)'});
  svg.append(ink);
  BOX_MASKS.forEach((_,i) => {
    const r=Math.floor(i/2), c=i%2, [x,y]=dotXY(r,c);
    if (boxOwner[i]) {
      ink.append(mk('rect',{x:x+10,y:y+10,width:80,height:80,fill:'url(#dab-hatch-'+boxOwner[i]+')',class:'box'}));
      const t=mk('text',{x:x+50,y:y+48,class:'lbl '+boxOwner[i]}); t.textContent = boxOwner[i]==='you' ? initial() : 'a'; ink.append(t);
    }
  });
  for (let e=0;e<12;e++) {
    const [[x1,y1],[x2,y2]] = edgeEnds(e), taken = mask>>e & 1;
    const l = mk('path',{d:sketch(e+1,x1,y1,x2,y2),class:'edge '+(taken?owner[e]:'free')+(e===lastEdge?' last':'')});
    ink.append(l);
    if (taken) {
      ink.append(mk('path',{d:sketch(e+1.37,x1+1,y1+1,x2+1,y2+1),class:'edge2 '+owner[e]}));
    } else {
      // wider invisible hit area for fingers
      const hit = mk('line',{x1,y1,x2,y2,stroke:'transparent','stroke-width':30,style:'cursor:pointer'});
      hit.addEventListener('click', () => humanMove(e));
      hit.addEventListener('pointerenter', () => l.classList.add('hover'));
      hit.addEventListener('pointerleave', () => l.classList.remove('hover'));
      svg.append(hit);
    }
  }
  for (let r=0;r<3;r++) for (let c=0;c<3;c++) { const [x,y]=dotXY(r,c); ink.append(mk('circle',{cx:x+(r-c)*0.6,cy:y+(c-r)*0.5,r:5.5,class:'dot'})); }
  $('dab-sy').textContent = boxOwner.filter(b=>b==='you').length;
  $('dab-sc').textContent = boxOwner.filter(b=>b==='cpu').length;
}

function play(e, who) {            // returns number of boxes closed
  const g = closed(mask,e);
  mask |= 1<<e; owner[e] = who; lastEdge = e;
  if (g) BOX_MASKS.forEach((bm,i) => { if (!boxOwner[i] && (mask&bm)===bm) boxOwner[i]=who; });
  return g;
}

function finishIfOver() {
  if (mask !== FULL) return false;
  over = true;
  const y=boxOwner.filter(b=>b==='you').length, c=4-y;
  $('dab-status').textContent = y>c ? (nameInput.value.trim() ? playerName() + ' wins! 🎉' : 'You win! 🎉') : y<c ? 'ānand wins.' : "It's a draw (2–2).";
  return true;
}

function humanMove(e) {
  if (over || busy || turn!=='you' || (mask>>e & 1)) return;
  const g = play(e,'you'); render();
  if (finishIfOver()) { saveGame(); return; }
  if (g) { $('dab-status').textContent = 'Box! Go again.'; saveGame(); return; }
  turn = 'cpu'; saveGame(); setTimeout(cpuTurn, 450);
}

function cpuTurn() {
  busy = true; $('dab-status').textContent = 'ānand is thinking…';
  const step = () => {
    const e = $('dab-level').value==='hard' ? hardMove(mask) : easyMove(mask);
    const g = play(e,'cpu'); render();
    if (finishIfOver()) { busy=false; saveGame(); return; }
    if (g) { saveGame(); setTimeout(step, 550); return; }      // closed a box -> moves again
    turn='you'; busy=false; saveGame(); $('dab-status').textContent = 'Your turn.';
  };
  setTimeout(step, 400);
}

function newGame() {
  mask=0; owner=Array(12).fill(null); boxOwner=Array(4).fill(null);
  turn='you'; over=false; busy=false; lastEdge=-1;
  $('dab-status').textContent='Your turn. Click a gap between two dots.'; render(); saveGame();
}

/* ---------- the game in progress, kept in this browser ---------- */
const GAME_KEY = 'and-log-dab-game';
function saveGame() {
  try { localStorage.setItem(GAME_KEY, JSON.stringify({ mask, owner, boxOwner, turn, over, lastEdge, level: $('dab-level').value })); } catch (err) {}
}
function loadGame() {
  try {
    const g = JSON.parse(localStorage.getItem(GAME_KEY));
    if (g && Number.isInteger(g.mask) && Array.isArray(g.owner) && g.owner.length === 12 && Array.isArray(g.boxOwner) && g.boxOwner.length === 4) return g;
  } catch (err) {}
  return null;
}
$('dab-level').addEventListener('change', saveGame);
$('dab-reset').addEventListener('click', newGame);

/* ---------- player name, kept in this browser ---------- */
const NAME_KEY = 'and-log-dab-name', nameInput = $('dab-name');
const playerName = () => nameInput.value.trim() || 'You';
const initial = () => [...playerName()][0].toUpperCase();
try { nameInput.value = localStorage.getItem(NAME_KEY) || ''; } catch (err) {}
function showName() { $('dab-you-name').textContent = playerName(); render(); }
nameInput.addEventListener('input', () => {
  try { localStorage.setItem(NAME_KEY, nameInput.value.trim()); } catch (err) {}
  showName();
});
const saved = loadGame();
if (saved) {
  ({ mask, owner, boxOwner, turn, over, lastEdge } = saved);
  busy = false;
  if (saved.level) $('dab-level').value = saved.level;
  render();
  if (!finishIfOver()) {
    if (turn === 'cpu') cpuTurn();
    else $('dab-status').textContent = 'Your turn.';
  }
} else newGame();
showName();
})();
</script>
