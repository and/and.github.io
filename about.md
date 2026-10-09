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

Take turns drawing a line between two dots. Close a box and it's yours, and you go again. I play blue.

<div class="dab">
  <style>
    .dab { --dab-ink:#222; --dab-you:#d9480f; --dab-cpu:#1c7ed6; --dab-faint:#e4e0d8;
           display:flex; flex-direction:column; align-items:center; gap:10px;
           margin:8px 0 24px; padding:20px 16px; border:1px solid var(--dab-faint); border-radius:12px; }
    .dab p { margin:0; }
    .dab-bar { display:flex; gap:16px; align-items:center; flex-wrap:wrap; justify-content:center; }
    .dab-name { display:flex; align-items:center; gap:8px; font-size:15px; }
    .dab input, .dab select, .dab button { font:inherit; font-size:15px; padding:6px 12px; border-radius:8px;
           border:1px solid var(--dab-faint); background:transparent; color:var(--dab-ink); }
    .dab input { width:11em; }
    .dab button { cursor:pointer; }
    .dab-score b { font-size:1.3rem; }
    .dab-you { color:var(--dab-you); } .dab-cpu { color:var(--dab-cpu); }
    .dab-status { min-height:1.4em; font-weight:600; }
    .dab svg { width:min(84vw,340px); height:auto; touch-action:manipulation; }
    .dab .edge { stroke:var(--dab-faint); stroke-width:8; stroke-linecap:round; }
    .dab .edge.free { cursor:pointer; }
    .dab .edge.free:hover { stroke:var(--dab-ink); opacity:.45; }
    .dab .edge.you { stroke:var(--dab-you); } .dab .edge.cpu { stroke:var(--dab-cpu); }
    .dab .edge.last { stroke-width:11; }
    .dab .box { opacity:.22; } .dab .box.you { fill:var(--dab-you); } .dab .box.cpu { fill:var(--dab-cpu); }
    .dab .lbl { font:700 34px system-ui,sans-serif; text-anchor:middle; dominant-baseline:central; pointer-events:none; }
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

function render() {
  svg.innerHTML = '';
  BOX_MASKS.forEach((_,i) => {
    const r=Math.floor(i/2), c=i%2, [x,y]=dotXY(r,c);
    if (boxOwner[i]) {
      svg.append(mk('rect',{x:x+4,y:y+4,width:92,height:92,rx:6,class:'box '+boxOwner[i]}));
      const t=mk('text',{x:x+50,y:y+50,class:'lbl '+boxOwner[i]}); t.textContent = boxOwner[i]==='you' ? initial() : 'a'; svg.append(t);
    }
  });
  for (let e=0;e<12;e++) {
    const [[x1,y1],[x2,y2]] = edgeEnds(e), taken = mask>>e & 1;
    const l = mk('line',{x1,y1,x2,y2,class:'edge '+(taken?owner[e]:'free')+(e===lastEdge?' last':'')});
    if (!taken) {
      l.addEventListener('click', () => humanMove(e));
      // wider invisible hit area for fingers
      const hit = mk('line',{x1,y1,x2,y2,stroke:'transparent','stroke-width':30,style:'cursor:pointer'});
      hit.addEventListener('click', () => humanMove(e));
      svg.append(l, hit);
    } else svg.append(l);
  }
  for (let r=0;r<3;r++) for (let c=0;c<3;c++) { const [x,y]=dotXY(r,c); svg.append(mk('circle',{cx:x,cy:y,r:9,class:'dot'})); }
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
  if (finishIfOver()) return;
  if (g) { $('dab-status').textContent = 'Box! Go again.'; return; }
  turn = 'cpu'; setTimeout(cpuTurn, 450);
}

function cpuTurn() {
  busy = true; $('dab-status').textContent = 'ānand is thinking…';
  const step = () => {
    const e = $('dab-level').value==='hard' ? hardMove(mask) : easyMove(mask);
    const g = play(e,'cpu'); render();
    if (finishIfOver()) { busy=false; return; }
    if (g) { setTimeout(step, 550); return; }      // closed a box -> moves again
    turn='you'; busy=false; $('dab-status').textContent = 'Your turn.';
  };
  setTimeout(step, 400);
}

function newGame() {
  mask=0; owner=Array(12).fill(null); boxOwner=Array(4).fill(null);
  turn='you'; over=false; busy=false; lastEdge=-1;
  $('dab-status').textContent='Your turn. Click a gap between two dots.'; render();
}
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
newGame();
showName();
})();
</script>
