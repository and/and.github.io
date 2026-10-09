---
layout: post
title: "Zipper Lists, Step by Step"
date: 2026-10-09 17:30:00 +0530
categories: [dsa]
permalink: /dsa/zipper-lists/
excerpt: "Weave two linked lists together so the result alternates nodes from each, by relinking the nodes that already exist. Step through the pointers one move at a time."
---

`zipper_lists(head_1, head_2)`, from Structy's Linked List I.

Given the heads of two linked lists, weave them together so the result alternates nodes from each list: list 1's head, then list 2's head, then list 1's second node, and so on. If one list runs out first, the rest of the other list goes on the end.

```
list_1: 1 → 3 → 5
list_2: 2 → 4 → 6
result: 1 → 2 → 3 → 4 → 5 → 6
```

No new nodes are made. Three pointers rewrite each node's `.next` in place: `tail` is the end of the result so far, and `current_1` and `current_2` are the next unused node in each list.

<div class="zipper-viz">
  <style>
    /* Scoped to .zipper-viz so nothing here leaks into the rest of the page —
       in particular the "button" and "line" rules stay off-limits outside it. */
    .zipper-viz{
      --zv-bg:#faf7f2; --zv-surface:#ffffff; --zv-ink:#1f1b16; --zv-muted:#6b6358;
      --zv-line:#d8d0c4; --zv-list1:#3454d1; --zv-list2:#c9742a; --zv-tail:#1f9d7c;
      max-width:480px; margin:28px auto; padding:20px;
      background:var(--zv-surface); color:var(--zv-ink);
      border:1px solid var(--zv-line); border-radius:16px;
      font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
      box-sizing:border-box;
    }
    .zipper-viz *{ box-sizing:border-box; }
    .zipper-viz .zv-eyebrow{
      font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
      font-size:11px; letter-spacing:.08em; text-transform:uppercase;
      color:var(--zv-muted); margin:0 0 4px;
    }
    .zipper-viz .zv-h1{ font-size:18px; margin:0 0 4px; font-weight:700; }
    .zipper-viz .zv-sub{ color:var(--zv-muted); font-size:13px; margin:0 0 18px; line-height:1.5; }
    .zipper-viz .zv-legend{ display:flex; gap:14px; flex-wrap:wrap; font-size:12px; color:var(--zv-muted); margin-bottom:16px; }
    .zipper-viz .zv-legend span{ display:inline-flex; align-items:center; gap:6px; }
    .zipper-viz .zv-dot{ width:9px; height:9px; border-radius:50%; display:inline-block; }

    .zipper-viz .zv-stage{
      position:relative; background:var(--zv-bg); border:1px solid var(--zv-line);
      border-radius:14px; padding:56px 8px 48px; margin-bottom:16px;
    }
    .zipper-viz .zv-row{ display:grid; grid-template-columns:repeat(3,1fr); position:relative; z-index:2; }
    .zipper-viz .zv-row1{ margin-bottom:76px; }
    .zipper-viz .zv-cell{ display:flex; justify-content:center; }
    .zipper-viz .zv-node{
      width:52px; height:52px; border-radius:50%;
      display:flex; align-items:center; justify-content:center;
      font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
      font-weight:600; font-size:17px;
      background:var(--zv-surface); border:2px solid var(--zv-line); color:var(--zv-ink);
      transition:border-color .25s ease, opacity .25s ease;
    }
    .zipper-viz .zv-row1 .zv-node{ border-color:var(--zv-list1); }
    .zipper-viz .zv-row2 .zv-node{ border-color:var(--zv-list2); }
    .zipper-viz .zv-node.zv-spent{ opacity:.4; }

    .zipper-viz svg.zv-edges{ position:absolute; inset:0; width:100%; height:100%; z-index:1; overflow:visible; }
    .zipper-viz svg.zv-edges line{ stroke:var(--zv-muted); stroke-width:2; }
    .zipper-viz svg.zv-edges line.zv-active{ stroke:var(--zv-tail); stroke-width:2.5; }
    .zipper-viz svg.zv-edges polygon{ fill:var(--zv-muted); }
    .zipper-viz svg.zv-edges polygon.zv-active{ fill:var(--zv-tail); }

    .zipper-viz .zv-ptr{
      position:absolute; transform:translate(-50%,-50%);
      transition:left .45s cubic-bezier(.4,0,.2,1), top .45s cubic-bezier(.4,0,.2,1), opacity .3s ease;
      font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
      font-size:11px; font-weight:600;
      padding:3px 9px; border-radius:999px; white-space:nowrap; z-index:3;
      box-shadow:0 1px 2px rgba(0,0,0,.15);
    }
    .zipper-viz #zv-ptr-tail{ background:var(--zv-tail); color:#06251d; }
    .zipper-viz #zv-ptr-cur1{ background:var(--zv-list1); color:#0a1440; }
    .zipper-viz #zv-ptr-cur2{ background:var(--zv-list2); color:#2c1300; }

    .zipper-viz .zv-desc{ font-size:14.5px; line-height:1.55; min-height:64px; margin-bottom:14px; }
    .zipper-viz .zv-desc b{
      font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
      font-weight:600; font-size:13.5px;
    }

    .zipper-viz .zv-controls{ display:flex; align-items:center; gap:10px; }
    .zipper-viz button{
      font-family:inherit; font-size:13px; font-weight:500;
      padding:9px 16px; border-radius:9px; border:1px solid var(--zv-line);
      background:var(--zv-surface); color:var(--zv-ink); cursor:pointer;
    }
    .zipper-viz button:disabled{ opacity:.35; cursor:default; }
    .zipper-viz button.zv-primary{ background:var(--zv-ink); color:var(--zv-bg); border-color:var(--zv-ink); }
    .zipper-viz .zv-stepcount{
      font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
      font-size:12px; color:var(--zv-muted); margin-left:auto;
    }
  </style>

  <p class="zv-eyebrow">structy · linked list i</p>
  <p class="zv-h1">zipper_lists(head_1, head_2)</p>
  <p class="zv-sub">list_1 = 1 → 3 → 5, list_2 = 2 → 4 → 6. One node attached per pass, alternating lists. Step through at your own pace — try predicting the next move before you click.</p>

  <div class="zv-legend">
    <span><span class="zv-dot" style="background:var(--zv-list1)"></span>list_1 / current_1</span>
    <span><span class="zv-dot" style="background:var(--zv-list2)"></span>list_2 / current_2</span>
    <span><span class="zv-dot" style="background:var(--zv-tail)"></span>tail</span>
  </div>

  <div class="zv-stage" id="zv-viz">
    <div class="zv-row zv-row1">
      <div class="zv-cell"><div class="zv-node" data-id="n1">1</div></div>
      <div class="zv-cell"><div class="zv-node" data-id="n3">3</div></div>
      <div class="zv-cell"><div class="zv-node" data-id="n5">5</div></div>
    </div>
    <div class="zv-row zv-row2">
      <div class="zv-cell"><div class="zv-node" data-id="n2">2</div></div>
      <div class="zv-cell"><div class="zv-node" data-id="n4">4</div></div>
      <div class="zv-cell"><div class="zv-node" data-id="n6">6</div></div>
    </div>
    <svg class="zv-edges" id="zv-edges"></svg>
    <div class="zv-ptr" id="zv-ptr-tail">tail</div>
    <div class="zv-ptr" id="zv-ptr-cur1">cur₁</div>
    <div class="zv-ptr" id="zv-ptr-cur2">cur₂</div>
  </div>

  <div class="zv-desc" id="zv-desc"></div>

  <div class="zv-controls">
    <button id="zv-restartBtn">Restart</button>
    <button id="zv-prevBtn">&larr; Prev</button>
    <button id="zv-nextBtn" class="zv-primary">Next &rarr;</button>
    <span class="zv-stepcount" id="zv-stepCount"></span>
  </div>
</div>

<script>
(function(){
  var frames = [
    { tail:'n1', cur1:'n3', cur2:'n2',
      next:{n1:'n3',n3:'n5',n5:null,n2:'n4',n4:'n6',n6:null},
      desc:"<b>Setup, before the loop runs.</b> tail = head_1 (node 1). current_1 = head_1.next (node 3) — captured right now, before anything is overwritten. current_2 = head_2 (node 2)." },
    { tail:'n1', cur1:'n3', cur2:'n2',
      next:{n1:'n2',n3:'n5',n5:null,n2:'n4',n4:'n6',n6:null}, edge:['n1','n2'],
      desc:"<b>Pass 1 (count=0, even): tail.next = current_2</b> — node 1 now points to node 2 instead of node 3. (current_1 is unaffected — it already moved off node 1 in setup.)" },
    { tail:'n2', cur1:'n3', cur2:'n4',
      next:{n1:'n2',n3:'n5',n5:null,n2:'n4',n4:'n6',n6:null},
      desc:"tail moves to node 2 — the node just attached. current_2 moves to node 4, one step past the node it just handed over." },
    { tail:'n2', cur1:'n3', cur2:'n4',
      next:{n1:'n2',n3:'n5',n5:null,n2:'n3',n4:'n6',n6:null}, edge:['n2','n3'],
      desc:"<b>Pass 2 (count=1, odd): tail.next = current_1</b> — node 2 now points to node 3." },
    { tail:'n3', cur1:'n5', cur2:'n4',
      next:{n1:'n2',n3:'n5',n5:null,n2:'n3',n4:'n6',n6:null},
      desc:"tail moves to node 3. current_1 moves to node 5." },
    { tail:'n3', cur1:'n5', cur2:'n4',
      next:{n1:'n2',n3:'n4',n5:null,n2:'n3',n4:'n6',n6:null}, edge:['n3','n4'],
      desc:"<b>Pass 3 (count=2, even): tail.next = current_2</b> — node 3 now points to node 4." },
    { tail:'n4', cur1:'n5', cur2:'n6',
      next:{n1:'n2',n3:'n4',n5:null,n2:'n3',n4:'n6',n6:null},
      desc:"tail moves to node 4. current_2 moves to node 6." },
    { tail:'n4', cur1:'n5', cur2:'n6',
      next:{n1:'n2',n3:'n4',n5:null,n2:'n3',n4:'n5',n6:null}, edge:['n4','n5'],
      desc:"<b>Pass 4 (count=3, odd): tail.next = current_1</b> — node 4 now points to node 5." },
    { tail:'n5', cur1:null, cur2:'n6',
      next:{n1:'n2',n3:'n4',n5:null,n2:'n3',n4:'n5',n6:null},
      desc:"tail moves to node 5. current_1 tries to advance past node 5 — but node 5 has no next, so current_1 becomes None. The loop condition (current_1 and current_2) fails, and the loop stops." },
    { tail:'n5', cur1:null, cur2:'n6',
      next:{n1:'n2',n3:'n4',n5:'n6',n2:'n3',n4:'n5',n6:null}, edge:['n5','n6'],
      desc:"Outside the loop: current_1 is None, but current_2 still has node 6 left over. <b>tail.next = current_2</b> attaches it." },
    { tail:'n5', cur1:null, cur2:'n6',
      next:{n1:'n2',n3:'n4',n5:'n6',n2:'n3',n4:'n5',n6:null},
      desc:"<b>Done.</b> Follow the arrows from node 1: 1 → 2 → 3 → 4 → 5 → 6. Return head_1 (node 1) as the head of the zipped list." }
  ];

  var idx = 0;
  var viz = document.getElementById('zv-viz');
  var svg = document.getElementById('zv-edges');
  var descEl = document.getElementById('zv-desc');
  var stepCountEl = document.getElementById('zv-stepCount');
  var prevBtn = document.getElementById('zv-prevBtn');
  var nextBtn = document.getElementById('zv-nextBtn');
  var restartBtn = document.getElementById('zv-restartBtn');

  function nodeEl(id){ return viz.querySelector('.zv-node[data-id="'+id+'"]'); }
  function rowOf(id){ return (id==='n1'||id==='n3'||id==='n5') ? 1 : 2; }

  function pt(id){
    var vr = viz.getBoundingClientRect();
    var nr = nodeEl(id).getBoundingClientRect();
    return { x: nr.left + nr.width/2 - vr.left, y: nr.top + nr.height/2 - vr.top, r: nr.width/2 };
  }

  function drawEdges(frame){
    var vr = viz.getBoundingClientRect();
    svg.setAttribute('viewBox', '0 0 ' + vr.width + ' ' + vr.height);
    var html = '';
    var active = frame.edge ? frame.edge[0]+'>'+frame.edge[1] : null;
    Object.keys(frame.next).forEach(function(src){
      var dst = frame.next[src];
      if(!dst) return;
      var a = pt(src), b = pt(dst);
      var dx = b.x - a.x, dy = b.y - a.y;
      var len = Math.sqrt(dx*dx + dy*dy);
      var ux = dx/len, uy = dy/len;
      var x1 = a.x + ux*a.r, y1 = a.y + uy*a.r;
      var x2 = b.x - ux*(b.r+9), y2 = b.y - uy*(b.r+9);
      var isActive = active === (src+'>'+dst);
      html += '<line x1="'+x1+'" y1="'+y1+'" x2="'+x2+'" y2="'+y2+'" class="'+(isActive?'zv-active':'')+'"/>';
      var ang = Math.atan2(y2-y1, x2-x1);
      var ah = 7;
      var p1x = x2 - ah*Math.cos(ang-0.4), p1y = y2 - ah*Math.sin(ang-0.4);
      var p2x = x2 - ah*Math.cos(ang+0.4), p2y = y2 - ah*Math.sin(ang+0.4);
      html += '<polygon points="'+x2+','+y2+' '+p1x+','+p1y+' '+p2x+','+p2y+'" class="'+(isActive?'zv-active':'')+'"/>';
    });
    svg.innerHTML = html;
  }

  function placePtr(el, id, side){
    if(!id){ el.style.opacity = '0'; return; }
    el.style.opacity = '1';
    var p = pt(id);
    var offset = side === 'above' ? -40 : 40;
    el.style.left = p.x + 'px';
    el.style.top = (p.y + offset) + 'px';
  }

  function render(){
    var frame = frames[idx];
    drawEdges(frame);

    var tailEl = document.getElementById('zv-ptr-tail');
    var cur1El = document.getElementById('zv-ptr-cur1');
    placePtr(tailEl, frame.tail, rowOf(frame.tail)===1 ? 'above':'below');
    placePtr(cur1El, frame.cur1, 'above');
    cur1El.textContent = frame.cur1 ? 'cur₁' : 'cur₁: None';
    placePtr(document.getElementById('zv-ptr-cur2'), frame.cur2, 'below');

    if(frame.tail === frame.cur1){
      tailEl.style.left = (parseFloat(tailEl.style.left) - 24) + 'px';
      cur1El.style.left = (parseFloat(cur1El.style.left) + 24) + 'px';
    }

    viz.querySelectorAll('.zv-node').forEach(function(n){ n.classList.remove('zv-spent'); });
    if(frame.cur1===null) nodeEl('n5').classList.add('zv-spent');

    descEl.innerHTML = frame.desc;
    stepCountEl.textContent = 'Step ' + (idx+1) + ' of ' + frames.length;
    prevBtn.disabled = idx === 0;
    nextBtn.disabled = idx === frames.length - 1;
  }

  prevBtn.addEventListener('click', function(){ if(idx>0){ idx--; render(); } });
  nextBtn.addEventListener('click', function(){ if(idx<frames.length-1){ idx++; render(); } });
  restartBtn.addEventListener('click', function(){ idx = 0; render(); });
  window.addEventListener('resize', render);

  render();
})();
</script>
