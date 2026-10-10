<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Zombie Uprising: Changing Worlds</title>
<style>
  * { box-sizing: border-box; }
  body { margin: 0; overflow: hidden; background: #111; color: white; font-family: Arial, sans-serif; }
  canvas { display: block; }
  #hud { position: fixed; inset: 0; pointer-events: none; text-shadow: 1px 2px 3px #000; }
  .panel { background: rgba(0,0,0,.62); border: 1px solid rgba(255,255,255,.22); border-radius: 8px; padding: 10px 13px; }
  #topLeft { position:absolute; top:14px; left:14px; min-width:190px; }
  #topRight { position:absolute; top:14px; right:14px; text-align:right; }
  #bottomLeft { position:absolute; bottom:16px; left:16px; }
  #bottomRight { position:absolute; bottom:16px; right:16px; text-align:right; }
  #crosshair { position:absolute; left:50%; top:50%; transform:translate(-50%,-50%); font-size:26px; font-weight:bold; color:#fff; }
  #hitmarker { position:absolute; left:50%; top:50%; transform:translate(-50%,-50%); color:#ff5757; font-size:25px; font-weight:bold; opacity:0; }
  #message { position:absolute; top:24%; left:50%; transform:translateX(-50%); font-size:24px; font-weight:bold; text-align:center; white-space:pre-line; }
  #healthOuter { width:180px; height:13px; background:#451d1d; border:1px solid #ddd; margin-top:5px; }
  #healthBar { height:100%; width:100%; background:#35d45a; }
  #bossOuter { width:280px; height:14px; background:#333; border:1px solid #fff; margin-top:5px; display:none; }
  #bossBar { height:100%; width:100%; background:#c12d39; }
  #overlay { position:fixed; inset:0; background:radial-gradient(ellipse at center,rgba(26,47,39,.74),rgba(2,5,8,.96)), repeating-linear-gradient(135deg,transparent 0 18px,rgba(255,255,255,.025) 18px 19px); display:flex; align-items:center; justify-content:center; z-index:10; backdrop-filter:blur(5px); }
  #box { width:min(600px,92vw); max-height:90vh; overflow:auto; padding:clamp(22px,5vw,42px); background:linear-gradient(145deg,rgba(21,34,38,.97),rgba(7,13,17,.98)); border:1px solid rgba(151,255,181,.6); border-radius:18px; text-align:center; box-shadow:0 0 0 1px rgba(255,255,255,.05),0 20px 80px #000,0 0 45px rgba(64,255,133,.12); }
  #box h1 { margin:0 0 12px; color:#b7ffb7; font-size:clamp(30px,6vw,48px); letter-spacing:2px; line-height:1.05; text-shadow:0 0 24px rgba(105,255,140,.3); }
  #overlayText { color:#e5f1e8; font-size:17px; }
  #overlayExtra { margin:20px auto; max-width:440px; }
  #overlayButtons { display:flex; flex-direction:column; gap:8px; width:min(280px,100%); margin:22px auto 0; }
  button { cursor:pointer; background:linear-gradient(180deg,#49a95c,#26713b); color:white; border:1px solid rgba(200,255,210,.25); border-radius:9px; padding:13px 18px; margin:0; font-size:15px; font-weight:800; letter-spacing:1px; transition:transform .15s,filter .15s,background .15s; }
  button:hover { background:linear-gradient(180deg,#66ce79,#348d4b); filter:brightness(1.08); transform:translateY(-1px); }
  #helpBtn { background:rgba(255,255,255,.07); border-color:rgba(255,255,255,.22); }
  #helpBtn:hover { background:rgba(255,255,255,.15); }
  .shopRow { display:flex; justify-content:space-between; align-items:center; gap:10px; border-bottom:1px solid #52605b; padding:8px 0; text-align:left; }
  .small { font-size:13px; color:#d0d0d0; line-height:1.5; }
  #terrainName { color:#ffe58a; font-weight:bold; }
  body.damageFlash::after { content:''; position:fixed; inset:0; pointer-events:none; background:rgba(255,25,25,.24); box-shadow:inset 0 0 90px rgba(255,0,0,.75); z-index:8; }
</style>
</head>
<body>
<div id="hud">
  <div id="topLeft" class="panel">
    <div><b>WAVE:</b> <span id="wave">0</span></div>
    <div><b>WORLD:</b> <span id="terrainName">CITY STREET</span></div>
    <div><b>ZOMBIES:</b> <span id="zombies">0</span></div>
    <div style="margin-top:7px"><b>HEALTH</b> <span id="healthText">100</span>/100</div>
    <div id="healthOuter"><div id="healthBar"></div></div>
  </div>
  <div id="topRight" class="panel">
    <div><b>CASH:</b> $<span id="cash">100</span></div>
    <div><b>KILLS:</b> <span id="kills">0</span></div>
    <div><b>WEAPON:</b> <span id="weapon">Pistol</span></div>
    <div><b>AMMO:</b> <span id="ammo">12 / 60</span></div>
    <div id="bossOuter"><div id="bossBar"></div></div>
  </div>
  <div id="bottomLeft" class="panel small">WASD move · Mouse aim · Click shoot<br>Shift sprint · R reload · E open chest<br>B weapon shop · Esc pause</div>
  <div id="bottomRight" class="panel"><b id="status">Ready</b></div>
  <div id="crosshair">+</div><div id="hitmarker">×</div>
  <div id="message"></div>
</div>
<div id="overlay">
  <div id="box">
    <div id="menuEyebrow" style="color:#ff7777;font-size:12px;font-weight:800;letter-spacing:5px;margin-bottom:12px">SURVIVE THE OUTBREAK</div>
    <h1 id="overlayTitle">ZOMBIE UPRISING</h1>
    <p id="overlayText">Survive the waves. Every wave changes the world!</p>
    <div id="overlayExtra" class="small">
      Move with WASD, aim with your mouse, and click or press Space to shoot.<br>
      Clear each wave to unlock a supply chest. Bosses appear every 5 waves.
    </div>
    <div id="overlayButtons"><button id="startBtn">▶ &nbsp; PLAY</button><button id="helpBtn">HOW TO PLAY</button></div>
  </div>
</div>
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
