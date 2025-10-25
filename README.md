<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Cosmic Particles — Interactive Canvas</title>
  <style>
    /* Minimal, modern styling */
    :root{--bg:#0b1020;--glass:rgba(255,255,255,0.06)}
    html,body{height:100%;margin:0;background:var(--bg);font-family:Inter,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial}
    .wrap{position:relative;height:100vh;display:flex;align-items:center;justify-content:center;overflow:hidden}
    canvas{display:block}

    .ui {
      position:fixed;left:18px;top:18px;backdrop-filter:blur(6px);background:var(--glass);padding:10px;border-radius:10px;color:#dfe7ff;font-size:13px;box-shadow:0 6px 18px rgba(2,6,23,0.6)
    }
    .ui button{margin-top:8px;background:transparent;border:1px solid rgba(255,255,255,0.08);padding:6px 10px;border-radius:8px;color:inherit;cursor:pointer}
    .title{position:fixed;right:18px;top:18px;color:#94a3ff;font-weight:700;letter-spacing:0.6px}
    .hint{opacity:0.8;font-size:12px;margin-top:4px}

    /* Footer credit */
    .credit{position:fixed;right:18px;bottom:18px;color:#9aa7ff;font-size:12px;opacity:0.9}

    @media (max-width:600px){.ui{left:10px;top:auto;bottom:10px}}
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="c"></canvas>
    <div class="ui">
      <div>Cosmic Particles — interactive demo</div>
      <div class="hint">Move pointer or click to spawn particles. Press 'C' to clear.</div>
      <div style="margin-top:8px">Size: <input id="size" type="range" min="1" max="8" value="3" /></div>
      <button id="download">Download PNG</button>
      <button id="toggleTrail">Toggle Trail</button>
    </div>
    <div class="title">Open & share on GitHub (single file)</div>
    <div class="credit">Made with JavaScript • (c) You</div>
  </div>

  <script>
    // Cosmic Particles — single-file interactive canvas
    // Open this file in your browser (double-click or drag into browser). No external libs.

    const canvas = document.getElementById('c');
    const ctx = canvas.getContext('2d');
    let W = canvas.width = innerWidth;
    let H = canvas.height = innerHeight;

    // options (editable)
    const opts = {
      particleCount: 120,
      baseSize: 2.5,
      speed: 0.98,
      gravity: 0.06,
      attraction: 0.08
n    };

    // state
    let particles = [];
    let mouse = {x: W/2, y: H/2, down:false};
    let trail = true;

    // helpers
    function rand(min, max){ return Math.random()*(max-min)+min }
    function hueFromPos(x,y){ return Math.floor((x/W)*180 + (y/H)*60 + (Date.now()/60)%120) }

    class Particle {
      constructor(x,y){
        this.x = x; this.y = y;
        this.vx = rand(-1.4,1.4);
        this.vy = rand(-1.4,1.4);
        this.size = rand(opts.baseSize*0.6, opts.baseSize*1.6);
        this.life = rand(80, 220);
        this.maxLife = this.life;
        this.h = hueFromPos(x,y);
      }
      update(){
        // gentle attraction to mouse and center
        const cx = W/2, cy = H/2;
        const ax = (mouse.x - this.x) * (mouse.down ? 0.12 : 0.02);
        const ay = (mouse.y - this.y) * (mouse.down ? 0.12 : 0.02);
        const bx = (cx - this.x) * 0.0012;
        const by = (cy - this.y) * 0.0012;

        this.vx += ax + bx;
        this.vy += ay + by + opts.gravity * 0.02;

        this.vx *= opts.speed;
        this.vy *= opts.speed;

        this.x += this.vx;
        this.y += this.vy;

        // life
        this.life -= 1;
        if(this.life < 0){
          // respawn near mouse
          this.x = mouse.x + rand(-40,40);
          this.y = mouse.y + rand(-40,40);
          this.vx = rand(-2,2);
          this.vy = rand(-2,2);
          this.life = this.maxLife = rand(80,220);
          this.size = rand(opts.baseSize*0.6, opts.baseSize*1.6);
          this.h = hueFromPos(this.x,this.y);
        }
      }
      draw(ctx){
        const t = 1 - (this.life/this.maxLife);
        const alpha = Math.max(0.08, 1 - t);
        ctx.beginPath();
        const g = ctx.createRadialGradient(this.x,this.y,0,this.x,this.y,this.size*12);
        g.addColorStop(0, `hsla(${this.h}, 95%, 60%, ${alpha})`);
        g.addColorStop(0.5, `hsla(${this.h+40}, 90%, 45%, ${alpha*0.6})`);
        g.addColorStop(1, `hsla(${this.h+80}, 80%, 20%, 0)`);
        ctx.fillStyle = g;
        ctx.arc(this.x,this.y,this.size*6,0,Math.PI*2);
        ctx.fill();

        // small core
        ctx.beginPath();
        ctx.fillStyle = `hsla(${this.h}, 95%, 60%, ${0.8*alpha})`;
        ctx.arc(this.x,this.y,this.size,0,Math.PI*2);
        ctx.fill();
      }
    }

    function init(){
      particles = [];
      for(let i=0;i<opts.particleCount;i++){
        particles.push(new Particle(rand(0,W), rand(0,H)));
      }
    }

    function resize(){
      W = canvas.width = innerWidth;
      H = canvas.height = innerHeight;
    }

    function frame(){
      if(!trail){
        ctx.clearRect(0,0,W,H);
      } else {
        ctx.fillStyle = 'rgba(6,10,20,0.12)';
        ctx.fillRect(0,0,W,H);
      }

      // subtle center glow
      const lg = ctx.createRadialGradient(W/2,H/2,0,W/2,H/2,Math.max(W,H)/2);
      lg.addColorStop(0,'rgba(10,14,30,0.06)');
      lg.addColorStop(1,'rgba(2,4,8,0)');
      ctx.fillStyle = lg;
      ctx.fillRect(0,0,W,H);

      for(const p of particles){
        p.update();
        p.draw(ctx);
      }

      requestAnimationFrame(frame);
    }

    // events
    addEventListener('resize', resize);
    addEventListener('mousemove', (e)=>{ mouse.x = e.clientX; mouse.y = e.clientY; });
    addEventListener('mousedown', (e)=>{ mouse.down = true; spawnBurst(e.clientX,e.clientY); });
    addEventListener('mouseup', ()=>{ mouse.down = false; });
    addEventListener('touchstart', (e)=>{ const t=e.touches[0]; mouse.x=t.clientX; mouse.y=t.clientY; mouse.down=true; spawnBurst(mouse.x,mouse.y); }, {passive:true});
    addEventListener('touchend', ()=>{ mouse.down=false }, {passive:true});
    addEventListener('touchmove', (e)=>{ const t=e.touches[0]; mouse.x=t.clientX; mouse.y=t.clientY }, {passive:true});

    addEventListener('keydown', (e)=>{
      if(e.key.toLowerCase()==='c'){ init(); }
      if(e.key.toLowerCase()==='t'){ trail = !trail; }
    });

    // UI bindings
    document.getElementById('size').addEventListener('input', (e)=>{ opts.baseSize = Number(e.target.value); });
    document.getElementById('download').addEventListener('click', ()=>{
      const link = document.createElement('a');
      link.download = 'cosmic-particles.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    });
    document.getElementById('toggleTrail').addEventListener('click', ()=>{ trail = !trail; document.getElementById('toggleTrail').innerText = trail ? 'Toggle Trail (on)' : 'Toggle Trail (off)'; });

    function spawnBurst(x,y){
      for(let i=0;i<12;i++){
        const p = new Particle(x + rand(-8,8), y + rand(-8,8));
        p.vx = rand(-6,6);
        p.vy = rand(-6,6);
        p.size = rand(1,6);
        p.life = p.maxLife = rand(60,180);
        particles.push(p);
      }
      // keep array bounded
      if(particles.length > 1200) particles.splice(0, particles.length-1200);
    }

    // initialize and run
    init();
    frame();

    // small instructions on load
    console.log('Cosmic Particles ready. Move your mouse or tap to interact.');
  </script>
</body>
</html>
