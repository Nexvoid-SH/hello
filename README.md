<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Your Name — Dev Card</title>
<style>
  :root{
    --bg: #0a0e14;
    --bg2: #0d1420;
    --node: #64ffda;
    --node-dim: #1c3b3a;
    --line: rgba(100,255,218,0.35);
    --text: #c9d6e3;
    --text-dim: #5c7080;
    --accent: #ff7b72;
  }
  *{ box-sizing:border-box; }
  html,body{
    margin:0; padding:0; height:100%;
    background: radial-gradient(ellipse at top, var(--bg2), var(--bg));
    color: var(--text);
    font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, Menlo, Consolas, monospace;
    overflow: hidden;
  }
  #canvas{ position:fixed; inset:0; z-index:1; }
  .card{
    position:relative; z-index:2;
    max-width: 640px;
    margin: 8vh auto 0;
    padding: 40px 44px;
    background: rgba(10,14,20,0.55);
    border: 1px solid rgba(100,255,218,0.18);
    border-radius: 4px;
    backdrop-filter: blur(6px);
    box-shadow: 0 0 60px rgba(0,0,0,0.4);
  }
  .eyebrow{
    font-size: 12px; letter-spacing: 3px; text-transform: uppercase;
    color: var(--node); opacity:0.8; margin-bottom: 14px;
  }
  h1{
    font-size: 40px; margin: 0 0 6px; color: #eef2f5; font-weight: 600;
    letter-spacing: -0.5px;
  }
  .role{ color: var(--text-dim); font-size: 15px; margin-bottom: 26px; }
  .role span{ color: var(--accent); }
  p.bio{ font-size: 14px; line-height: 1.7; color: var(--text); max-width: 46ch; }
  .links{ margin-top: 28px; display:flex; gap: 14px; flex-wrap: wrap; }
  .links a{
    color: var(--node); text-decoration:none; font-size: 13px;
    border: 1px solid var(--node-dim); padding: 8px 14px; border-radius: 2px;
    transition: all .2s ease;
  }
  .links a:hover{ background: var(--node); color: #0a0e14; box-shadow: 0 0 18px var(--node); }
  .hint{
    position:absolute; bottom: 14px; right: 18px; font-size: 11px; color: var(--text-dim);
  }
  ::selection{ background: var(--node); color: #0a0e14; }
</style>
</head>
<body>

<canvas id="canvas"></canvas>

<div class="card">
  <div class="eyebrow">// node_status: active</div>
  <h1>Your Name</h1>
  <div class="role">Software Engineer — <span>backend & distributed systems</span></div>
  <p class="bio">
    I build things that need to stay up at 3am. Currently deep in Rust,
    queues, and convincing databases to behave. This card is a live
    network — move your cursor, watch it react.
  </p>
  <div class="links">
    <a href="https://github.com/yourhandle">GitHub</a>
    <a href="https://yourportfolio.com">Portfolio</a>
    <a href="mailto:you@example.com">Email</a>
  </div>
</div>

<div class="hint">move your mouse ↝</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let w, h;
function resize(){
  w = canvas.width = window.innerWidth;
  h = canvas.height = window.innerHeight;
}
resize();
window.addEventListener('resize', resize);

const mouse = { x: w/2, y: h/2, active:false };
window.addEventListener('mousemove', e => {
  mouse.x = e.clientX; mouse.y = e.clientY; mouse.active = true;
});
window.addEventListener('mouseleave', () => mouse.active = false);

// Node field
const NODE_COUNT = 70;
const LINK_DIST = 130;
const MOUSE_DIST = 180;
let nodes = [];

function initNodes(){
  nodes = [];
  for(let i=0;i<NODE_COUNT;i++){
    nodes.push({
      x: Math.random()*w,
      y: Math.random()*h,
      vx: (Math.random()-0.5)*0.25,
      vy: (Math.random()-0.5)*0.25,
      r: 1.4 + Math.random()*1.6
    });
  }
}
initNodes();
window.addEventListener('resize', initNodes);

function dist(a,b,c,d){ return Math.hypot(a-c,b-d); }

function tick(){
  ctx.clearRect(0,0,w,h);

  // update
  for(const n of nodes){
    n.x += n.vx; n.y += n.vy;
    if(n.x < 0 || n.x > w) n.vx *= -1;
    if(n.y < 0 || n.y > h) n.vy *= -1;
  }

  // node-node links
  ctx.lineWidth = 1;
  for(let i=0;i<nodes.length;i++){
    for(let j=i+1;j<nodes.length;j++){
      const d = dist(nodes[i].x, nodes[i].y, nodes[j].x, nodes[j].y);
      if(d < LINK_DIST){
        const alpha = (1 - d/LINK_DIST) * 0.35;
        ctx.strokeStyle = `rgba(100,255,218,${alpha})`;
        ctx.beginPath();
        ctx.moveTo(nodes[i].x, nodes[i].y);
        ctx.lineTo(nodes[j].x, nodes[j].y);
        ctx.stroke();
      }
    }
  }

  // mouse links (the interactive signature)
  if(mouse.active){
    for(const n of nodes){
      const d = dist(mouse.x, mouse.y, n.x, n.y);
      if(d < MOUSE_DIST){
        const alpha = (1 - d/MOUSE_DIST) * 0.9;
        ctx.strokeStyle = `rgba(255,123,114,${alpha})`;
        ctx.lineWidth = 1.2;
        ctx.beginPath();
        ctx.moveTo(mouse.x, mouse.y);
        ctx.lineTo(n.x, n.y);
        ctx.stroke();

        // ping node bigger when near cursor
        ctx.beginPath();
        ctx.fillStyle = `rgba(255,123,114,${alpha})`;
        ctx.arc(n.x, n.y, n.r + (1-d/MOUSE_DIST)*3, 0, Math.PI*2);
        ctx.fill();
      }
    }
    // cursor node itself
    ctx.beginPath();
    ctx.fillStyle = '#ff7b72';
    ctx.shadowColor = '#ff7b72';
    ctx.shadowBlur = 12;
    ctx.arc(mouse.x, mouse.y, 3, 0, Math.PI*2);
    ctx.fill();
    ctx.shadowBlur = 0;
  }

  // draw regular nodes
  ctx.fillStyle = '#64ffda';
  for(const n of nodes){
    ctx.globalAlpha = 0.75;
    ctx.beginPath();
    ctx.arc(n.x, n.y, n.r, 0, Math.PI*2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;

  requestAnimationFrame(tick);
}
tick();
</script>

</body>
</html>
