<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<title>Cat Runner — Offline HTML Game</title>
<style>
  :root{
    --bg:#dff6ff;      /* day sky */
    --bg-night:#0d1321;/* night sky */
    --ground:#654321;  /* ground color */
    --ui:#111;
    --ui-weak:#777;
    --accent:#ff7a18;
  }
  html,body{height:100%;margin:0;background:linear-gradient(#ffffff00,#ffffff00) var(--bg);font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial,sans-serif;-webkit-user-select:none;user-select:none}
  #wrap{height:100%;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:10px}
  canvas{width:min(calc(100vw - 20px), 840px);height:calc(min(100vw - 20px, 840px)/1.8);max-height:70vh;background:transparent;border-radius:12px;box-shadow:0 8px 24px rgba(0,0,0,.15);touch-action:none}
  #hud{width:min(calc(100vw - 20px), 840px);display:flex;align-items:center;justify-content:space-between;gap:8px}
  .pill{display:inline-flex;align-items:center;gap:8px;border-radius:999px;padding:8px 12px;background:#ffffffcc;backdrop-filter:saturate(1.2) blur(6px);border:1px solid #0001}
  #score{font-weight:700;color:var(--ui)}
  #hi{font-weight:600;color:var(--ui-weak)}
  #controls{display:flex;gap:8px;flex-wrap:wrap}
  button{border:0;border-radius:999px;padding:10px 14px;background:var(--accent);color:#fff;font-weight:700;letter-spacing:.3px}
  button.alt{background:#fff;color:#222;border:1px solid #0002}
  button.ghost{background:#00000010;color:#222}
  #tips{color:#000a;font-size:12px;text-align:center}
  #settingsPanel{
    position:fixed;inset:auto 0 0 0;transform:translateY(100%);
    transition:transform .25s ease; background:#fff; border-top-left-radius:16px;border-top-right-radius:16px;
    box-shadow:0 -10px 30px rgba(0,0,0,.15); padding:14px; max-height:70vh; overflow:auto
  }
  #settingsPanel.open{transform:translateY(0)}
  .row{display:flex;align-items:center;justify-content:space-between;padding:8px 0;border-bottom:1px dashed #0002}
  .row:last-child{border-bottom:0}
  .toggle{appearance:none;width:44px;height:26px;border-radius:999px;background:#ddd;position:relative;outline:none;cursor:pointer;transition:background .2s}
  .toggle:checked{background:var(--accent)}
  .toggle::after{content:"";position:absolute;left:3px;top:3px;width:20px;height:20px;border-radius:50%;background:#fff;transition:left .2s}
  .toggle:checked::after{left:21px}
  .badge{font-size:11px;color:#fff;background:#0006;padding:2px 8px;border-radius:999px;margin-left:6px}
</style>
</head>
<body>
<div id="wrap">
  <canvas id="game" width="840" height="468" aria-label="Cat Runner Game"></canvas>
  <div id="hud">
    <div class="pill" id="score">Score: 0</div>
    <div style="display:flex;align-items:center;gap:8px">
      <div class="pill" id="hi">High: 0</div>
      <div id="controls">
        <button id="btnJump">JUMP</button>
        <button id="btnDuck" class="alt">DUCK</button>
        <button id="btnPause" class="ghost">PAUSE</button>
        <button id="btnSettings" class="ghost" aria-expanded="false">⚙️</button>
      </div>
    </div>
  </div>
  <div id="tips" class="pill" style="max-width:min(calc(100vw - 20px), 840px)">
    Tap JUMP • Double-tap for double jump • Hold DUCK to slide • Avoid obstacles • Collect ⭐ for points • Power-ups: 🛡️ shield, 🐟 fish (magnet), ⏳ slow-mo. Keyboard: Space/W = jump, S/↓ = duck, P = pause, R = restart.
  </div>
</div>

<!-- Settings drawer -->
<div id="settingsPanel" role="dialog" aria-modal="true" aria-label="Settings">
  <div class="row"><strong>Sound</strong><input id="optSound" class="toggle" type="checkbox" checked></div>
  <div class="row"><strong>Vibration (Android)</strong><input id="optVibrate" class="toggle" type="checkbox" checked></div>
  <div class="row"><strong>Day / Night Cycle</strong><input id="optCycle" class="toggle" type="checkbox" checked></div>
  <div class="row"><strong>Show FPS</strong><input id="optFps" class="toggle" type="checkbox"></div>
  <div class="row"><strong>Left-handed UI</strong><input id="optLeft" class="toggle" type="checkbox"></div>
  <div class="row"><strong>Difficulty</strong>
    <select id="optDiff">
      <option value="easy">Easy</option>
      <option value="normal" selected>Normal</option>
      <option value="hard">Hard</option>
      <option value="insane">Insane</option>
    </select>
  </div>
  <div class="row">
    <button id="btnReset" class="alt">Reset High Score</button>
    <span class="badge">Local only</span>
  </div>
</div>

<script>
(() => {
  "use strict";

  /*** Helpers ***/
  const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
  const rnd=(a,b)=>Math.random()*(b-a)+a;
  const choice=a=>a[(Math.random()*a.length)|0];

  // Persisted settings
  const LS_KEY="catrunner_v1";
  const store={
    save(d){try{localStorage.setItem(LS_KEY,JSON.stringify(d));}catch{}},
    load(){try{return JSON.parse(localStorage.getItem(LS_KEY))||{};}catch{ return {};}}
  };

  /*** Audio (no external files; generated with WebAudio) ***/
  const audioCtx = (window.AudioContext||window.webkitAudioContext) ? new (window.AudioContext||window.webkitAudioContext)() : null;
  let audioEnabled = true;
  function beep({f=440, t=0.08, type="square", gain=0.03}={}){
    if(!audioEnabled||!audioCtx) return;
    const o=audioCtx.createOscillator(), g=audioCtx.createGain();
    o.type=type; o.frequency.value=f;
    g.gain.value=gain;
    o.connect(g); g.connect(audioCtx.destination);
    o.start();
    setTimeout(()=>{o.stop();}, t*1000);
  }
  const sfx = {
    jump(){ beep({f:520, t:.09, type:"square"}); },
    coin(){ beep({f:880, t:.07, type:"sine", gain:.05}); },
    hit(){  beep({f:140, t:.12, type:"sawtooth", gain:.06}); },
    power(){beep({f:600, t:.12, type:"triangle", gain:.05});}
  };

  /*** Canvas & UI ***/
  const cvs = document.getElementById("game");
  const ctx = cvs.getContext("2d");
  const scoreEl = document.getElementById("score");
  const hiEl = document.getElementById("hi");
  const btnJump = document.getElementById("btnJump");
  const btnDuck = document.getElementById("btnDuck");
  const btnPause = document.getElementById("btnPause");
  const btnSettings = document.getElementById("btnSettings");
  const panel = document.getElementById("settingsPanel");
  const optSound = document.getElementById("optSound");
  const optVibrate = document.getElementById("optVibrate");
  const optCycle = document.getElementById("optCycle");
  const optFps = document.getElementById("optFps");
  const optLeft = document.getElementById("optLeft");
  const optDiff = document.getElementById("optDiff");
  const btnReset = document.getElementById("btnReset");

  // Option: left-handed swaps control order
  function applyLeftHanded(on){
    const controls = document.getElementById("controls");
    controls.style.flexDirection = on ? "row-reverse" : "row";
  }

  /*** Game State ***/
  const G = {
    started:false, paused:false, over:false,
    t:0, dt:0, last:performance.now(),
    speed:8, // base speed in world px/frame@60
    gravity:0.85, groundY: 380, // ground baseline
    score:0, hiscore:0,
    night:false, cycle:true,
    difficulty: "normal",
    fps:0,
    waves:0
  };

  // Load persisted options
  {
    const saved = store.load();
    G.hiscore = saved.hiscore||0;
    audioEnabled = saved.sound ?? true;
    optSound.checked = audioEnabled;
    optVibrate.checked = saved.vibrate ?? true;
    G.cycle = saved.cycle ?? true; optCycle.checked = G.cycle;
    optFps.checked = saved.fps ?? false;
    optLeft.checked = saved.left ?? false; applyLeftHanded(optLeft.checked);
    G.difficulty = saved.diff || "normal"; optDiff.value = G.difficulty;
    updateHi();
  }

  /*** Entities ***/
  class Cat {
    constructor(){
      this.reset();
    }
    reset(){
      this.x = 120; this.y = G.groundY;
      this.vy = 0;
      this.size = 46;
      this.ducking = false;
      this.onGround = true;
      this.doubleLeft = 1;
      this.invincible = 0; // frames
      this.magnet = 0;
      this.slowmo = 0;
      this.blink = 0;
    }
    aabb(){
      const h = this.ducking ? this.size*0.6 : this.size;
      const y = this.ducking ? this.y+this.size*0.4 : this.y;
      return {x:this.x-22, y:y-h, w:44, h};
    }
    jump(){
      if(this.onGround){
        this.vy = -15; this.onGround=false; this.doubleLeft=1; sfx.jump();
      } else if(this.doubleLeft>0){
        this.vy = -14; this.doubleLeft=0; sfx.jump();
      }
    }
    update(){
      const slow = this.slowmo>0 ? 0.5 : 1;
      this.vy += G.gravity*slow;
      this.y += this.vy*slow;
      if(this.y >= G.groundY){ this.y = G.groundY; this.vy=0; this.onGround=true; }
      this.invincible = Math.max(0, this.invincible-1);
      this.magnet = Math.max(0, this.magnet-1);
      this.slowmo = Math.max(0, this.slowmo-1);
      this.blink = (this.invincible>0) ? (this.blink+1)%20 : 0;
    }
    draw(){
      // Cute cat drawn with shapes (no images)
      const t = performance.now()/1000;
      const a = this.aabb();
      ctx.save();
      ctx.translate(this.x, this.y);
      ctx.scale(1,1);
      // Shadow
      ctx.fillStyle="rgba(0,0,0,.15)";
      ctx.beginPath(); ctx.ellipse(0,8,30,8,0,0,Math.PI*2); ctx.fill();

      if(this.invincible>0 && this.blink<10){ ctx.globalAlpha=.45; }

      // Body
      ctx.globalCompositeOperation="source-over";
      ctx.fillStyle="#ffd6e8"; // pink cat
      ctx.strokeStyle="#d88fb0";
      ctx.lineWidth=2;
      // torso
      roundRect(-26,-38,52,36,12,true,true);
      // head
      roundRect(-22,-70,44,34,10,true,true);
      // ears
      triangle(-16,-70,-4,-90,4,-70,"#ffd6e8","#d88fb0");
      triangle(16,-70,4,-90,-4,-70,"#ffd6e8","#d88fb0");
      // face
      ctx.fillStyle="#222"; // eyes
      const blink = (Math.sin(t*3)+1)/2 < .05;
      const eyeH = blink?2:6;
      ctx.beginPath(); ctx.ellipse(-8,-53,4,eyeH,0,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(8,-53,4,eyeH,0,0,Math.PI*2); ctx.fill();
      // nose
      ctx.fillStyle="#e75480";
      ctx.beginPath(); ctx.moveTo(0,-47); ctx.lineTo(-4,-43); ctx.lineTo(4,-43); ctx.closePath(); ctx.fill();
      // mouth
      ctx.strokeStyle="#333"; ctx.lineWidth=1.8;
      ctx.beginPath(); ctx.moveTo(0,-43); ctx.bezierCurveTo(-4,-40,-6,-38,-8,-38); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(0,-43); ctx.bezierCurveTo(4,-40,6,-38,8,-38); ctx.stroke();

      // legs (simple run cycle)
      const run = Math.max(0.3, Math.min(1.6, (G.speed)/7));
      const phase = t*8*run;
      const leg = (x,y,s)=>{ ctx.save(); ctx.translate(x,y); ctx.rotate(Math.sin(phase+s)*0.6); roundRect(-4,0,8,18,3,true,false); ctx.restore(); };
      leg(-14,-14,0);
      leg( 14,-14,Math.PI);
      // tail
      ctx.save(); ctx.translate(24,-42); ctx.rotate(Math.sin(phase)*0.3 - (this.ducking?0.8:0)); roundRect(0,-4,26,8,4,true,false); ctx.restore();

      // Fish magnet aura
      if(this.magnet>0){
        ctx.strokeStyle="#00a2ff88"; ctx.lineWidth=3;
        ctx.beginPath(); ctx.arc(0,-40,36,0,Math.PI*2); ctx.stroke();
      }

      // Slow-mo aura
      if(this.slowmo>0){
        ctx.strokeStyle="#8b5cf6aa"; ctx.lineWidth=3;
        ctx.beginPath(); ctx.arc(0,-40,46,0,Math.PI*2); ctx.stroke();
      }

      // Shield
      if(this.invincible>0){
        ctx.strokeStyle="#00c853aa"; ctx.lineWidth=6;
        ctx.beginPath(); ctx.arc(0,-38,54,0,Math.PI*2); ctx.stroke();
      }

      ctx.restore();

      // Hitbox debug (toggle manually if needed)
      // ctx.fillStyle="rgba(255,0,0,.2)"; ctx.fillRect(a.x,a.y,a.w,a.h);
    }
  }

  class Obstacle {
    constructor(type,speed){
      this.type=type; // 'box' | 'dog' | 'bird'
      this.x = cvs.width + rnd(0,80);
      this.y = G.groundY;
      this.w = 40; this.h = 40;
      this.vx = -speed;
      if(type==="dog"){ this.w=50; this.h=45;}
      if(type==="bird"){ this.w=42; this.h=32; this.y -= choice([30,70]); }
    }
    aabb(){ return {x:this.x-this.w/2, y:this.y-this.h, w:this.w, h:this.h}; }
    update(){
      const slow = player.slowmo>0 ? 0.6 : 1;
      this.x += this.vx*slow;
    }
    draw(){
      ctx.save(); ctx.translate(this.x, this.y);
      // shadow
      ctx.fillStyle="rgba(0,0,0,.15)";
      ctx.beginPath(); ctx.ellipse(0,6,this.w*.5,6,0,0,Math.PI*2); ctx.fill();

      if(this.type==="box"){
        ctx.fillStyle="#c58f3d"; ctx.strokeStyle="#8a5a12"; ctx.lineWidth=3;
        roundRect(-this.w/2,-this.h,this.w,this.h,6,true,true);
      } else if(this.type==="dog"){
        // simple brown dog
        ctx.fillStyle="#b67f5a"; ctx.strokeStyle="#70442a"; ctx.lineWidth=2;
        roundRect(-24,-38,48,34,10,true,true); // body
        roundRect(6,-58,30,26,8,true,true);    // head
        triangle(28,-58,20,-72,14,-58,"#b67f5a","#70442a"); // ear
        ctx.fillStyle="#222"; ctx.beginPath(); ctx.arc(26,-48,3,0,Math.PI*2); ctx.fill(); // eye
      } else { // bird
        ctx.fillStyle="#444"; ctx.strokeStyle="#222"; ctx.lineWidth=2;
        roundRect(-18,-18,36,20,10,true,true);
        triangle(20,-10,32,-12,20,-6,"#444","#222");
      }
      ctx.restore();

      // Debug hitbox
      // const a=this.aabb(); ctx.fillStyle="rgba(255,0,0,.18)"; ctx.fillRect(a.x,a.y,a.w,a.h);
    }
  }

  class Coin {
    constructor(){
      this.x=cvs.width + rnd(0,60);
      this.y=G.groundY - rnd(30,140);
      this.r=10; this.vx = -G.speed*0.9;
      this.t=0;
    }
    aabb(){ return {x:this.x-this.r, y:this.y-this.r, w:this.r*2, h:this.r*2}; }
    update(){
      const slow = player.slowmo>0 ? 0.6 : 1;
      this.x += -G.speed*0.9*slow;
      this.t+=0.2;
    }
    draw(){
      ctx.save(); ctx.translate(this.x,this.y);
      ctx.rotate(Math.sin(this.t)*0.3);
      ctx.fillStyle="#ffcf33"; ctx.strokeStyle="#b08900"; ctx.lineWidth=2;
      ctx.beginPath(); ctx.ellipse(0,0,12,12,0,0,Math.PI*2); ctx.fill(); ctx.stroke();
      ctx.fillStyle="#b08900"; ctx.font="bold 12px system-ui,Segoe UI"; ctx.textAlign="center"; ctx.textBaseline="middle";
      ctx.fillText("⭐",0,1);
      ctx.restore();
    }
  }

  class PowerUp {
    constructor(kind){
      this.kind = kind; // 'shield' 'magnet' 'slow'
      this.x = cvs.width + rnd(0,40);
      this.y = G.groundY - rnd(50,120);
      this.vx = -G.speed*0.9;
      this.r = 14;
      this.t=0;
    }
    aabb(){ return {x:this.x-16, y:this.y-16, w:32, h:32}; }
    update(){
      const slow = player.slowmo>0 ? 0.6 : 1;
      this.x += -G.speed*0.9*slow;
      this.t+=0.2;
    }
    draw(){
      ctx.save(); ctx.translate(this.x,this.y);
      ctx.globalAlpha=0.95;
      const ring = (c)=>{ ctx.strokeStyle=c; ctx.lineWidth=4; ctx.beginPath(); ctx.arc(0,0,18,0,Math.PI*2); ctx.stroke(); };
      if(this.kind==="shield"){ ring("#00c853aa"); ctx.fillStyle="#00c853"; ctx.beginPath(); ctx.arc(0,0,10,0,Math.PI*2); ctx.fill(); ctx.fillStyle="#fff"; ctx.font="bold 12px system-ui"; ctx.textAlign="center"; ctx.textBaseline="middle"; ctx.fillText("🛡️",0,1); }
      if(this.kind==="magnet"){ ring("#00a2ffaa"); ctx.fillStyle="#00a2ff"; ctx.beginPath(); ctx.arc(0,0,10,0,Math.PI*2); ctx.fill(); ctx.fillStyle="#fff"; ctx.fillText("🐟",0,1); }
      if(this.kind==="slow"){ ring("#8b5cf6aa"); ctx.fillStyle="#8b5cf6"; ctx.beginPath(); ctx.arc(0,0,10,0,Math.PI*2); ctx.fill(); ctx.fillStyle="#fff"; ctx.fillText("⏳",0,1); }
      ctx.restore();
    }
  }

  const player = new Cat();
  const obstacles = [];
  const coins = [];
  const powerups = [];

  /*** Difficulty scaling ***/
  function diffMultiplier(){
    switch(G.difficulty){
      case "easy": return 0.9;
      case "normal": return 1.0;
      case "hard": return 1.15;
      case "insane": return 1.35;
    }
    return 1;
  }

  /*** Spawners ***/
  let nextObs = 0, nextCoin = 0, nextPU = 0;
  function schedule(){
    nextObs = G.t + rnd(40, 85) / diffMultiplier();
    nextCoin = G.t + rnd(20, 45);
    nextPU   = G.t + rnd(240, 420);
  }

  /*** Collision ***/
  function overlaps(a,b){
    return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
  }

  /*** World Draw ***/
  const parallax = [
    {y: 320, h:2, c:"#00000015", speed:0.4, dash: [8,8]},
    {y: 330, h:2, c:"#00000010", speed:0.7, dash: [16,10]},
    {y: 340, h:4, c:"#00000020", speed:1.0, dash: [0,0]}
  ];
  let px=0;
  function drawWorld(){
    // sky
    const cycle = G.cycle ? (Math.sin(G.t/6000)+1)/2 : (G.night?1:0);
    const bg = lerpColor(getCssVar("--bg"), getCssVar("--bg-night"), cycle);
    document.body.style.background = bg;

    // distant sun/moon
    ctx.save();
    ctx.globalAlpha=0.7;
    ctx.fillStyle = cycle<0.5 ? "#ffd26f" : "#c2d7ff";
    ctx.beginPath();
    ctx.arc(700,80,30,0,Math.PI*2); ctx.fill();
    ctx.restore();

    // ground band
    ctx.fillStyle=getCssVar("--ground");
    ctx.fillRect(0,G.groundY+10,cvs.width,6);

    // parallax lines
    px += G.speed;
    parallax.forEach((p,i)=>{
      ctx.save();
      ctx.strokeStyle=p.c; ctx.lineWidth=p.h; ctx.setLineDash(p.dash);
      ctx.beginPath();
      ctx.moveTo((-px*p.speed)%cvs.width, p.y);
      ctx.lineTo(cvs.width, p.y);
      ctx.stroke();
      ctx.restore();
    });
  }

  /*** Main Loop ***/
  function loop(now){
    G.dt = now - G.last; G.last = now; G.t += G.dt;

    // fps
    if(optFps.checked){ G.fps = Math.round(1000/Math.max(1,G.dt)); }

    if(!G.paused && G.started && !G.over){
      // Speed ramps with score
      const accel = 0.0005 * diffMultiplier();
      G.speed = Math.min(20*diffMultiplier(), 8*diffMultiplier() + (G.score*accel));

      // Spawn
      if(G.t > nextObs){
        const t = choice(["box","dog","bird","box","dog","box"]); // weights
        obstacles.push(new Obstacle(t, G.speed));
        schedule();
      }
      if(G.t > nextCoin){ coins.push(new Coin()); nextCoin = G.t + rnd(20,50); }
      if(G.t > nextPU){ powerups.push(new PowerUp(choice(["shield","magnet","slow"]))); nextPU = G.t + rnd(280,480); }

      // Update entities
      player.update();
      obstacles.forEach(o=>o.update());
      coins.forEach(c=>c.update());
      powerups.forEach(p=>p.update());

      // Magnet effect
      if(player.magnet>0){
        coins.forEach(c=>{
          const dx = (player.x - c.x), dy = ((player.y-40) - c.y);
          const dist = Math.hypot(dx,dy);
          if(dist<180){
            c.x += dx * 0.06;
            c.y += dy * 0.06;
          }
        });
      }

      // Collisions
      const pa = player.aabb();
      // coins
      for(let i=coins.length-1;i>=0;i--){
        const c=coins[i], ca=c.aabb();
        if(overlaps(pa, ca)){
          coins.splice(i,1);
          G.score += 10;
          sfx.coin();
        }
      }
      // powerups
      for(let i=powerups.length-1;i>=0;i--){
        const p=powerups[i], aa=p.aabb();
        if(overlaps(pa, aa)){
          powerups.splice(i,1);
          if(p.kind==="shield") player.invincible = 60*8;
          if(p.kind==="magnet") player.magnet = 60*10;
          if(p.kind==="slow")   player.slowmo = 60*6;
          sfx.power();
        }
      }
      // obstacles
      for(let i=obstacles.length-1;i>=0;i--){
        const o=obstacles[i], oa=o.aabb();
        if(overlaps(pa, oa)){
          if(player.invincible>0){
            // consume partial shield
            obstacles.splice(i,1);
            player.invincible = Math.max(0, player.invincible-60*2);
          }else{
            gameOver();
            break;
          }
        }
      }

      // Cleanup off-screen
      purge(obstacles, o=>o.x < -80);
      purge(coins, c=>c.x < -40);
      purge(powerups, p=>p.x < -40);

      // Score tick
      G.score += 0.2 * diffMultiplier();
      updateScore();
    }

    // Render
    ctx.clearRect(0,0,cvs.width,cvs.height);
    drawWorld();
    coins.forEach(c=>c.draw());
    powerups.forEach(p=>p.draw());
    obstacles.forEach(o=>o.draw());
    player.draw();

    // UI overlays
    drawUI();

    requestAnimationFrame(loop);
  }

  function drawUI(){
    // ground line
    ctx.strokeStyle="#00000015"; ctx.lineWidth=2;
    ctx.beginPath(); ctx.moveTo(0,G.groundY); ctx.lineTo(cvs.width,G.groundY); ctx.stroke();

    // paused/over banners
    ctx.save(); ctx.textAlign="center"; ctx.textBaseline="middle";
    if(!G.started){
      banner("CAT RUNNER", "Tap JUMP or press Space to start");
    } else if(G.paused){
      banner("PAUSED", "Tap PAUSE or press P to resume");
    } else if(G.over){
      banner("GAME OVER", "Press R or tap JUMP to retry");
    }
    // FPS
    if(optFps.checked){
      ctx.fillStyle="#000b"; ctx.font="12px monospace"; ctx.textAlign="left";
      ctx.fillText(`FPS ${G.fps}`, 8, 16);
    }
    ctx.restore();
  }

  function banner(title, subtitle){
    ctx.save();
    ctx.globalAlpha=0.9;
    ctx.fillStyle="#fff"; ctx.strokeStyle="#0001";
    roundRect(cvs.width/2-220, cvs.height/2-80, 440, 150, 14, true, true);
    ctx.fillStyle="#111"; ctx.font="700 28px system-ui,Segoe UI,Roboto";
    ctx.textAlign="center"; ctx.fillText(title, cvs.width/2, cvs.height/2-24);
    ctx.fillStyle="#555"; ctx.font="500 16px system-ui,Segoe UI";
    ctx.fillText(subtitle, cvs.width/2, cvs.height/2+12);
    ctx.restore();
  }

  /*** UI state updates ***/
  function updateScore(){
    scoreEl.textContent = "Score: " + Math.floor(G.score);
    if(G.score>G.hiscore){
      G.hiscore = Math.floor(G.score);
      updateHi();
      persist();
    }
  }
  function updateHi(){
    hiEl.textContent = "High: " + Math.floor(G.hiscore);
  }
  function persist(){
    store.save({
      hiscore:G.hiscore,
      sound:audioEnabled,
      vibrate:optVibrate.checked,
      cycle:G.cycle,
      fps:optFps.checked,
      left:optLeft.checked,
      diff:G.difficulty
    });
  }

  /*** Controls ***/
  function startIfNeeded(){
    if(!G.started){ G.started=true; schedule(); }
    if(G.over){ restart(); }
  }
  function jump(){ startIfNeeded(); player.jump(); }
  let duckHeld=false;
  function duck(on){
    duckHeld = on;
    player.ducking = on && player.onGround;
  }
  function pauseToggle(){
    if(!G.started || G.over) return;
    G.paused = !G.paused;
    if(!G.paused && audioCtx && audioCtx.state==="suspended"){ audioCtx.resume?.(); }
  }
  function gameOver(){
    G.over=true; sfx.hit(); if(optVibrate.checked && navigator.vibrate){ navigator.vibrate([60,40,60]); }
    persist();
  }
  function restart(){
    G.over=false; G.score=0; G.speed=8*diffMultiplier(); obstacles.length=0; coins.length=0; powerups.length=0;
    player.reset(); schedule();
  }

  // Buttons
  btnJump.addEventListener("pointerdown", e=>{ e.preventDefault(); jump(); });
  btnDuck.addEventListener("pointerdown", e=>{ e.preventDefault(); duck(true); });
  btnDuck.addEventListener("pointerup",   e=>{ e.preventDefault(); duck(false); });
  btnDuck.addEventListener("pointercancel", ()=>duck(false));
  btnPause.addEventListener("click", ()=>pauseToggle());

  // Settings
  btnSettings.addEventListener("click", ()=>{
    const open = !panel.classList.contains("open");
    panel.classList.toggle("open", open);
    btnSettings.setAttribute("aria-expanded", open?"true":"false");
  });
  optSound.addEventListener("change", ()=>{ audioEnabled = optSound.checked; persist(); if(audioEnabled) sfx.coin(); });
  optVibrate.addEventListener("change", ()=>{ persist(); });
  optCycle.addEventListener("change", ()=>{ G.cycle = optCycle.checked; persist(); });
  optFps.addEventListener("change", ()=>{ persist(); });
  optLeft.addEventListener("change", ()=>{ applyLeftHanded(optLeft.checked); persist(); });
  optDiff.addEventListener("change", ()=>{
    G.difficulty = optDiff.value; persist();
    // Slight feedback
    sfx.power();
  });
  btnReset.addEventListener("click", ()=>{ G.hiscore=0; updateHi(); persist(); });

  // Keyboard
  window.addEventListener("keydown", (e)=>{
    if(["Space","KeyW","ArrowUp"].includes(e.code)){ e.preventDefault(); jump(); }
    if(["KeyS","ArrowDown"].includes(e.code)){ e.preventDefault(); duck(true); }
    if(e.code==="KeyP"){ pauseToggle(); }
    if(e.code==="KeyR"){ restart(); }
  });
  window.addEventListener("keyup", (e)=>{
    if(["KeyS","ArrowDown"].includes(e.code)){ duck(false); }
  });

  // Touch: double-tap for double jump is naturally handled by jump() again while airborne

  // Resize handling for crisp canvas on high-DPI
  function resize(){
    const dpr = Math.min(2, window.devicePixelRatio||1);
    const cssW = cvs.clientWidth;
    const cssH = cvs.clientHeight;
    cvs.width = Math.round(cssW * dpr);
    cvs.height= Math.round(cssH * dpr);
    ctx.setTransform(dpr,0,0,dpr,0,0);
  }
  new ResizeObserver(()=>resize()).observe(cvs);
  resize();

  // Main loop start
  requestAnimationFrame(loop);

  /*** Utilities (drawing, colors, arrays) ***/
  function roundRect(x,y,w,h,r,fill,stroke){
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    if(fill) ctx.fill();
    if(stroke) ctx.stroke();
  }
  function triangle(x1,y1,x2,y2,x3,y3,fill,stroke){
    ctx.beginPath(); ctx.moveTo(x1,y1); ctx.lineTo(x2,y2); ctx.lineTo(x3,y3); ctx.closePath();
    if(fill){ ctx.fillStyle=fill; ctx.fill(); }
    if(stroke){ ctx.strokeStyle=stroke; ctx.stroke(); }
  }
  function purge(arr,pred){ for(let i=arr.length-1;i>=0;i--){ if(pred(arr[i])) arr.splice(i,1); } }
  function getCssVar(name){
    return getComputedStyle(document.documentElement).getPropertyValue(name).trim();
  }
  function lerpColor(a,b,t){
    const ca=parseColor(a), cb=parseColor(b);
    const m=x=>Math.round(ca[x]+(cb[x]-ca[x])*t);
    return `rgb(${m(0)},${m(1)},${m(2)})`;
  }
  function parseColor(c){
    // supports #rrggbb or rgb(r,g,b)
    if(c.startsWith("#")){
      const r=parseInt(c.slice(1,3),16), g=parseInt(c.slice(3,5),16), b=parseInt(c.slice(5,7),16);
      return [r,g,b];
    }
    const m=c.match(/rgb\((\d+),\s*(\d+),\s*(\d+)\)/); if(m) return [parseInt(m[1]),parseInt(m[2]),parseInt(m[3])];
    return [223,246,255];
  }

  // Start on first user interaction to allow audio on mobile
  window.addEventListener("pointerdown", ()=>{
    if(audioCtx && audioCtx.state==="suspended"){ audioCtx.resume?.(); }
  }, {once:true});

})();
</script>
</body>
</html>
