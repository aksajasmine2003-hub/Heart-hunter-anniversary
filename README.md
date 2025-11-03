<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
  <title>Heart Hunter - Happy Anniversary, Aaru!</title>
  <style>
    * { margin:0; padding:0; box-sizing:border-box; }
    body { background:#000; overflow:hidden; touch-action:none; font-family:Arial; }
    canvas { display:block; margin:0 auto; image-rendering:pixelated; }
    #ui { position:fixed; bottom:0; width:100%; height:80px; background:rgba(0,0,0,0.7); display:flex; }
    .btn { flex:1; color:white; font-size:24px; text-align:center; line-height:80px; user-select:none; }
    #left { background:rgba(0,100,255,0.7); }
    #right { background:rgba(255,0,0,0.7); }
    #jump { background:rgba(0,200,0,0.7); font-weight:bold; }
    #msg { position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:rgba(20,20,40,0.95); color:#ffb6c1; padding:20px; border-radius:15px; text-align:center; max-width:90%; font-size:18px; display:none; z-index:100; line-height:1.6; }
    #final { position:fixed; inset:0; background:rgba(0,0,0,0.9); color:#ffb6c1; display:none; flex-direction:column; justify-content:center; align-items:center; padding:20px; text-align:center; z-index:200; }
    h1 { font-size:28px; margin:10px 0; }
    p { margin:8px 0; line-height:1.5; }
  </style>
</head>
<body>
  <canvas id="game"></canvas>
  <div id="ui">
    <div id="left" class="btn">Left</div>
    <div id="jump" class="btn">JUMP</div>
    <div id="right" class="btn">Right</div>
  </div>
  <div id="msg"></div>
  <div id="final">
    <h1>Happy Anniversary, Aaru!</h1>
    <p>Every heart is a memory we made...</p>
    <p>From our first unofficial date to that kiss in Martin...</p>
    <p>I love you more every day.</p>
    <p>Forever yours,<br><b>Ashu</b></p>
    <button onclick="location.reload()" style="margin-top:20px;padding:10px 20px;font-size:18px;background:#d4a5a5;color:white;border:none;border-radius:8px;">Play Again</button>
  </div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    canvas.width = 800;
    canvas.height = 400;

    const scale = Math.min(window.innerWidth / 800, (window.innerHeight - 80) / 400);
    canvas.style.width = (800 * scale) + 'px';
    canvas.style.height = (400 * scale) + 'px';

    let player = { x: 50, y: 350, w: 24, h: 36, vy: 0, onGround: true };
    let platforms = [
      {x:0, y:380, w:800, h:20},
      {x:200, y:300, w:120, h:20},
      {x:420, y:220, w:140, h:20},
      {x:620, y:150, w:100, h:20}
    ];
    let hearts = [
      {x:260, y:270}, {x:480, y:190}, {x:670, y:120}, {x:150, y:150}, {x:720, y:280}
    ].map(h => ({...h, collected: false}));
    let obstacles = [{x:350, y:360, w:20, h:20}, {x:580, y:360, w:20, h:20}];
    let score = 0;
    let msg = document.getElementById('msg');
    let final = document.getElementById('final');
    let touching = { left:false, right:false };

    const notes = [
      "Remember our first unofficial date? You were so nervous and I was totally unaware that it's a date",
      "Do you remember our first kiss on the movie theatre where we watched the movie Martin? I still remember that your lips were so soft",
      "The day you said 'I love you Ashu' for the first time..My heart still races thinking about it",
      "Our silly inside jokes- only we get them.. they make everyday better",
      "Every moment with you feels like a adventure...I can't wait for a life time more"
    ];

    document.getElementById('left').ontouchstart = () => touching.left = true;
    document.getElementById('left').ontouchend = () => touching.left = false;
    document.getElementById('right').ontouchstart = () => touching.right = true;
    document.getElementById('right').ontouchend = () => touching.right = false;
    document.getElementById('jump').ontouchstart = () => { if(player.onGround) { player.vy = -15; player.onGround = false; } };

    window.onkeydown = e => {
      if(e.key === 'ArrowLeft') touching.left = true;
      if(e.key === 'ArrowRight') touching.right = true;
      if(e.key === ' ' && player.onGround) { player.vy = -15; player.onGround = false; }
    };
    window.onkeyup = e => {
      if(e.key === 'ArrowLeft') touching.left = false;
      if(e.key === 'ArrowRight') touching.right = false;
    };

    function update() {
      if(touching.left) player.x -= 5;
      if(touching.right) player.x += 5;

      player.vy += 0.8;
      player.y += player.vy;

      player.onGround = false;
      for(let p of platforms) {
        if(player.x + player.w > p.x && player.x < p.x + p.w &&
           player.y + player.h > p.y && player.y + player.h < p.y + p.h + 5 && player.vy > 0) {
          player.y = p.y - player.h;
          player.vy = 0;
          player.onGround = true;
        }
      }

      for(let h of hearts) {
        if(!h.collected && Math.hypot(player.x + 12 - h.x, player.y + 18 - h.y) < 25) {
          h.collected = true;
          score++;
          showMessage(notes[score-1]);
        }
      }

      for(let o of obstacles) {
        if(player.x + player.w > o.x && player.x < o.x + o.w &&
           player.y + player.h > o.y && player.y < o.y + o.w) {
          player.x = 50; player.y = 350; player.vy = 0;
        }
      }

      if(score === 5) final.style.display = 'flex';

      player.x = Math.max(0, Math.min(800 - player.w, player.x));
      if(player.y > 400) { player.y = 350; player.vy = 0; }

      draw();
      requestAnimationFrame(update);
    }

    function draw() {
      ctx.fillStyle = '#000'; ctx.fillRect(0,0,800,400);
      ctx.fillStyle = '#0c0';
      platforms.forEach(p => ctx.fillRect(p.x, p.y, p.w, p.h));
      ctx.fillStyle = '#f00';
      hearts.forEach(h => { if(!h.collected) { ctx.beginPath(); ctx.arc(h.x, h.y, 8, 0, Math.PI*2); ctx.fill(); } });
      ctx.fillStyle = '#aaa';
      obstacles.forEach(o => ctx.fillRect(o.x, o.y, o.w, o.h));
      ctx.fillStyle = '#00f'; ctx.fillRect(player.x, player.y, player.w, player.h);

      ctx.fillStyle = '#fff'; ctx.font = '24px Arial';
      ctx.fillText(`Hearts: ${score}/5`, 20, 40);
      ctx.fillText(`Our November 4, 2024`, 500, 40);
    }

    function showMessage(text) {
      msg.innerHTML = text.replace(/\.\./g, '.<br>');
      msg.style.display = 'block';
      setTimeout(() => msg.style.display = 'none', 5000);
    }

    update();
  </script>
</body>
</html>
