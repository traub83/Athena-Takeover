<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The Era of Athena</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@300;400;600&family=Cinzel+Decorative:wght@300&display=swap');
  * { margin:0; padding:0; box-sizing:border-box; }
  body { background:#000; overflow:hidden; font-family:'Cinzel',serif; }
  canvas { display:block; }

  /* ── STORY SCREEN ── */
  #story-screen {
    position:fixed; inset:0; background:rgba(0,0,0,0.88);
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    z-index:100; pointer-events:all;
  }
  #story-text {
    color:#e8d5a3; font-size:clamp(16px,2.5vw,26px); font-weight:300;
    letter-spacing:0.12em; text-align:center; line-height:1.9;
    max-width:680px; padding:0 30px;
    text-shadow: 0 0 40px rgba(232,213,163,0.6);
    opacity:0; transition: opacity 1.2s ease;
  }
  #story-screen .skip-btn {
    position:absolute; bottom:32px; right:32px;
    background:rgba(255,255,255,0.06); border:1px solid rgba(255,255,255,0.18);
    color:#888; font-family:'Cinzel',serif; font-size:10px; letter-spacing:0.2em;
    padding:8px 18px; cursor:pointer; text-transform:uppercase;
    transition:all 0.2s;
  }
  #story-screen .skip-btn:hover { color:#fff; border-color:rgba(255,255,255,0.4); }

  /* ── MAIN UI ── */
  #ui {
    position:fixed; inset:0; pointer-events:none; z-index:10;
    opacity:0; transition:opacity 1.5s;
  }
  #ui.visible { opacity:1; }

  #title {
    position:absolute; top:24px; left:50%; transform:translateX(-50%);
    text-align:center; color:#e8d5a3; letter-spacing:0.3em;
    text-transform:uppercase; font-size:10px; opacity:0.65;
  }
  #title h1 { font-family:'Cinzel Decorative',serif; font-size:20px; font-weight:300; letter-spacing:0.45em; color:#fff; margin-bottom:4px; }

  #phase-label {
    position:absolute; bottom:34px; left:50%; transform:translateX(-50%);
    color:#e8d5a3; font-size:12px; letter-spacing:0.22em; text-align:center;
    text-shadow:0 0 18px rgba(232,213,163,0.6); white-space:nowrap;
  }

  /* ── OBSERVATORY PANEL ── */
  #athena-info {
    position:absolute; top:50%; left:50%;
    transform:translate(-50%,-50%);
    color:#00ff88; font-size:16px; letter-spacing:0.16em;
    opacity:0; pointer-events:none;
    text-align:left; line-height:2;
    background:rgba(0,10,5,0.92);
    border:2px solid rgba(0,255,120,0.6);
    border-left:5px solid #00ff88;
    padding:28px 36px;
    font-family:'Courier New',monospace;
    box-shadow:0 0 60px rgba(0,255,100,0.3), 0 0 120px rgba(0,255,100,0.1), inset 0 0 30px rgba(0,255,100,0.06);
    min-width:380px; z-index:25;
  }
  #athena-info .obs-title {
    color:#00ffaa; font-size:16px; letter-spacing:0.25em;
    text-transform:uppercase; font-weight:bold;
    border-bottom:1px solid rgba(0,255,120,0.3);
    padding-bottom:7px; margin-bottom:10px;
    text-shadow:0 0 10px rgba(0,255,150,0.8);
  }
  #athena-info .threat-bar {
    height:8px; background:rgba(0,255,80,0.15);
    border:1px solid rgba(0,255,80,0.3); border-radius:2px;
    margin:4px 0 10px; overflow:hidden; position:relative;
  }
  #athena-info .threat-fill {
    height:100%; width:0%;
    background:linear-gradient(90deg,#ff4400,#ff8800,#ffcc00,#ff0000);
    transition:width 2s ease; box-shadow:0 0 10px #ff4400;
    animation:threatPulse 0.8s ease-in-out infinite alternate;
  }
  @keyframes threatPulse {
    from{box-shadow:0 0 8px #ff4400;} to{box-shadow:0 0 20px #ff0000,0 0 40px #ff4400;}
  }
  #athena-info .recommended {
    color:#ffcc00; font-size:0.85em; letter-spacing:0.12em;
    line-height:1.7; margin-top:6px;
    text-shadow:0 0 8px rgba(255,200,0,0.6);
  }
  #athena-info .blink { animation:blink 0.9s step-end infinite; }
  @keyframes blink { 0%,100%{opacity:1;} 50%{opacity:0;} }

  /* ── COUNTDOWN OVERLAY ── */
  #countdown-overlay {
    position:absolute; inset:0;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    pointer-events:none; z-index:20; opacity:0; transition:opacity 0.5s;
    background:none;
  }
  #countdown-overlay.visible { opacity:1; }
  #countdown-text {
    font-family:'Cinzel Decorative',serif;
    color:#fff; font-size:clamp(48px,10vw,110px); font-weight:300;
    letter-spacing:0.15em; text-align:center;
    text-shadow:0 0 40px rgba(255,80,0,0.9), 0 0 80px rgba(255,40,0,0.5);
    transition:opacity 0.15s, transform 0.15s;
  }
  #countdown-sub {
    color:#ffaa44; font-size:clamp(10px,1.4vw,14px); letter-spacing:0.4em;
    margin-top:10px; text-transform:uppercase;
    text-shadow:0 0 14px rgba(255,150,50,0.8);
  }

  /* Approach story lines — layered above 3D, no background block */
  #approach-story {
    position:fixed; top:35%; left:50%; transform:translate(-50%,-50%);
    text-align:center; color:#f0d8ff; font-size:clamp(24px,4.5vw,58px);
    font-family:'Cinzel',serif; font-weight:600;
    letter-spacing:0.22em; line-height:1.7; pointer-events:none;
    text-shadow:0 0 20px rgba(255,255,255,0.9),
                0 0 50px rgba(220,140,255,1),
                0 0 100px rgba(160,60,255,0.7);
    z-index:30; opacity:0; transition:opacity 1.2s;
    background:none;
  }
  #approach-story.visible { opacity:1; }

  #impact-overlay {
    position:absolute; inset:0;
    background:radial-gradient(ellipse at center,rgba(255,120,20,0) 0%,rgba(255,40,0,0) 100%);
    opacity:0; pointer-events:none; transition:opacity 0.1s; mix-blend-mode:screen;
  }

  /* ── TAKEOVER SCREEN ── */
  #takeover-screen {
    position:absolute; inset:0; background:rgba(0,0,0,0.95);
    opacity:0; display:flex; flex-direction:column; align-items:center;
    justify-content:center; pointer-events:none; transition:opacity 2.5s, background 2s ease;
  }
  #takeover-screen.visible { opacity:1; }
  #takeover-screen h2 {
    font-family:'Cinzel Decorative',serif;
    color:#b8a0e8; font-size:clamp(20px,3vw,36px); font-weight:300;
    letter-spacing:0.35em; text-transform:uppercase;
    text-shadow:0 0 60px rgba(184,160,232,1),0 0 120px rgba(184,160,232,0.5);
    margin-bottom:18px; text-align:center;
    animation:pulse 2.5s ease-in-out infinite;
  }
  #takeover-screen .sub {
    color:#c8a0ff; font-size:clamp(12px,1.6vw,18px); letter-spacing:0.2em;
    text-align:center; line-height:2.2; margin-bottom:22px;
    text-shadow:0 0 20px rgba(200,160,255,0.5);
  }
  #takeover-screen .credit {
    color:rgba(255,255,255,0.3); font-size:10px; letter-spacing:0.3em; text-transform:uppercase;
    margin-top:10px;
  }
  @keyframes pulse {
    0%,100%{text-shadow:0 0 60px rgba(184,160,232,1),0 0 120px rgba(184,160,232,0.5);}
    50%{text-shadow:0 0 90px rgba(184,160,232,1),0 0 180px rgba(184,160,232,0.8),0 0 240px rgba(184,160,232,0.3);}
  }

  /* ── CONTROLS ── */
  #controls {
    position:absolute; bottom:64px; right:26px;
    pointer-events:all; display:flex; gap:8px;
  }
  .ctrl-btn {
    background:rgba(255,255,255,0.05); border:1px solid rgba(255,255,255,0.14);
    color:#999; padding:7px 14px; font-family:'Cinzel',serif;
    font-size:9px; letter-spacing:0.18em; cursor:pointer;
    text-transform:uppercase; transition:all 0.2s;
  }
  .ctrl-btn:hover { background:rgba(255,255,255,0.12); color:#fff; border-color:rgba(255,255,255,0.4); }

  /* ── PLANET LABELS ── */
  .planet-label {
    position:absolute; pointer-events:none;
    transform:translate(-50%,-100%);
    display:flex; flex-direction:column; align-items:center; gap:3px;
    transition:opacity 0.4s;
  }
  .planet-label span {
    font-family:'Cinzel',serif; font-size:9px; letter-spacing:0.2em;
    text-transform:uppercase; background:rgba(0,0,0,0.55);
    padding:2px 8px; border-radius:2px; white-space:nowrap;
    text-shadow:0 0 10px currentColor; color:inherit;
  }
  .planet-label .dot { width:3px; height:3px; border-radius:50%; background:currentColor; }

  .lbl-QuadCities { color:#ffe060; }
  .lbl-Beer       { color:#f0c030; }
  .lbl-Vodka      { color:#a0d8ff; }
  .lbl-State18    { color:#50d0ff; }
  .lbl-Gin        { color:#ff7050; }
  .lbl-Tequila    { color:#ffb040; }
  .lbl-Margaritas { color:#b0ff80; }
  .lbl-Whiskey    { color:#60e8ff; }
  .lbl-Rum        { color:#6080ff; }
  .lbl-Athena     { color:#dd88ff; }
  .lbl-Uber       { color:#00ffcc; }

  /* ── CREDIT ── */
  #credit {
    position:absolute; bottom:12px; left:50%; transform:translateX(-50%);
    color:rgba(255,255,255,0.22); font-size:9px; letter-spacing:0.28em; text-transform:uppercase;
  }
</style>
</head>
<body>

<!-- STORY SCREEN -->
<div id="story-screen">
  <div id="story-text"></div>
  <button class="skip-btn" onclick="skipStory()">Skip →</button>
</div>

<!-- TAP TO BEGIN — guarantees audio unlock -->
<div id="tap-screen" style="
  position:fixed; inset:0; background:#000;
  display:flex; flex-direction:column; align-items:center; justify-content:center;
  z-index:999; cursor:pointer;
" onclick="tapToBegin()">
  <div style="
    font-family:'Cinzel Decorative',serif; font-size:clamp(28px,5vw,60px);
    color:#fff; letter-spacing:0.3em; text-transform:uppercase; text-align:center;
    text-shadow:0 0 40px rgba(255,255,255,0.8), 0 0 80px rgba(200,150,255,0.5);
    animation:tapPulse 1.8s ease-in-out infinite;
  ">The Era of Athena</div>
  <div style="
    font-family:'Cinzel',serif; font-size:clamp(12px,2vw,18px);
    color:#888; letter-spacing:0.4em; margin-top:30px; text-transform:uppercase;
    animation:tapPulse 1.8s ease-in-out infinite 0.4s;
  ">Tap anywhere to begin</div>
  <style>
    @keyframes tapPulse {
      0%,100%{opacity:0.5;} 50%{opacity:1;}
    }
  </style>
</div>

<!-- MAIN UI -->
<div id="ui">
  <div id="title">
    <h1>The Era of Athena</h1>
    <span>Quad Cities Orbit of Fun</span>
  </div>
  <div id="athena-info" style="display:none">
    <div class="obs-title">⚠ Quad Cities Observatory</div>
    <div>OBJECT: <span style="color:#ff8844">ATHENA</span></div>
    <div>CLASS: <span style="color:#ff8844">AMAZING BARTENDER</span></div>
    <div>TRAJECTORY: <span style="color:#ff8844">STATE 18</span></div>
    <div style="margin-top:8px">Threat Level:</div>
    <div class="threat-bar"><div class="threat-fill" id="threat-fill"></div></div>
    <div style="color:#ff4444;font-size:11px;letter-spacing:0.2em;" class="blink">██████████ 100%</div>
    <div class="recommended" style="margin-top:10px">
      Recommended Action:<br>
      <span style="color:#fff">Proceed immediately to</span><br>
      <span style="color:#50d0ff;font-size:11px;letter-spacing:0.18em;">STATE 18</span><span style="color:#fff"> for Safety</span>
    </div>
  </div>
  <div id="countdown-overlay">
    <div id="countdown-text"></div>
    <div id="countdown-sub"></div>
  </div>
  <div id="approach-story"></div>
  <div id="impact-overlay"></div>
  <div id="takeover-screen">
    <h2>Athena Has Taken Over State 18!</h2>
    <div class="sub">
      The collision was not an ending...<br>
      It was a beginning.<br><br>
      Welcome to the new era.<br>
      Welcome to Athena's world!<br><br>
      Come see her — 5 till close!
    </div>
  </div>
  <div id="uber-flash" style="
    position:fixed; bottom:80px; left:0; right:0; display:flex; align-items:center; justify-content:center;
    pointer-events:none; z-index:200; opacity:0; transition:opacity 1s ease;
  ">
    <div style="
      font-family:'Cinzel Decorative',serif; font-size:clamp(20px,3.5vw,42px);
      color:#00ffcc; text-align:center; letter-spacing:0.2em; font-weight:400;
      text-shadow:0 0 40px rgba(0,255,200,0.9), 0 0 80px rgba(0,255,200,0.4);
      background:rgba(0,0,0,0.6); padding:20px 50px; border:1px solid rgba(0,255,200,0.3);
      border-radius:4px;
    ">
      Stay safe — Use an Uber
    </div>
  </div>
  <div id="labels-container"></div>
  <div id="phase-label"></div>
  <div id="controls">
    <button class="ctrl-btn" onclick="resetScene()">↺ Reset</button>
    <button class="ctrl-btn" onclick="triggerAthena()">⚡ Trigger Athena</button>
  </div>
  <div id="credit">Made by Rayme Traub</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ── PHASE DEFINITIONS — must be first ────────────────
const PHASE={STORY:'story',FORMING:'forming',ORBIT:'orbit',COUNTDOWN:'countdown',APPROACH:'approach',IMPACT:'impact',TAKEOVER:'takeover'};
let PHASE_CUR=PHASE.STORY;

// ═══════════════════════════════════════════════════════
//  STORY SEQUENCE
// ═══════════════════════════════════════════════════════
const storyLines = [
  { text:"In the not too distant future", dur:4000, credit:true },
  { text:"On the eve of Thursday,\nthe people of the Quad Cities\nprepared for another ordinary night\nalong the river", dur:5000 },
  { text:"But whispers spread\nacross every bar", dur:5000 },
  { text:"A force of nature\nwas coming", dur:5000 },
  { text:"Her name was Athena", dur:5000, big:true },
  { text:"She brought tequila", dur:5000 },
  { text:"She brought laughter", dur:5000 },
  { text:"She brought chaos", dur:5000 },
  { text:"And after her arrival\nnothing would ever\nbe the same", dur:5000 },
];

let storyIndex = 0;
let storyActive = true;
const storyEl = document.getElementById('story-text');
const storyScreen = document.getElementById('story-screen');

function showNextLine() {
  if (!storyActive) return;
  if (storyIndex >= storyLines.length) {
    storyEl.style.transition = 'opacity 1s ease';
    storyEl.style.opacity = '0';
    setTimeout(() => {
      storyScreen.style.transition = 'opacity 1.8s';
      storyScreen.style.opacity = '0';
      setTimeout(() => { storyScreen.style.display = 'none'; }, 1900);
    }, 1200);
    return;
  }
  const line = storyLines[storyIndex];

  // Fade OUT current text (1s)
  storyEl.style.transition = 'opacity 1s ease';
  storyEl.style.opacity = '0';

  // After fade out + 1s dark pause = 2s gap total, then style and fade IN new line
  setTimeout(() => {
    storyEl.style.transition = 'none';
    storyEl.style.fontSize = line.credit ? 'clamp(24px,4vw,48px)'
                           : line.big    ? 'clamp(22px,3.5vw,38px)'
                           :               'clamp(16px,2.5vw,26px)';
    storyEl.style.fontWeight = line.credit ? '700' : '300';
    storyEl.style.letterSpacing = line.credit ? '0.18em'
                                : line.big    ? '0.25em' : '0.12em';
    storyEl.style.color = line.credit ? '#ffffff'
                        : line.big    ? '#cc88ff' : '#e8d5a3';
    storyEl.style.textShadow = line.credit
      ? '0 0 30px rgba(255,255,255,0.8), 0 0 60px rgba(255,255,255,0.3)'
      : line.big
      ? '0 0 60px rgba(200,120,255,0.9), 0 0 120px rgba(180,80,255,0.5)'
      : '0 0 40px rgba(232,213,163,0.6)';
    storyEl.innerHTML = line.text.replace(/\n/g, '<br>');
    storyEl.style.opacity = '0';

    // Small tick to allow browser to apply opacity:0 before transitioning in
    requestAnimationFrame(() => {
      requestAnimationFrame(() => {
        storyEl.style.transition = 'opacity 1s ease';
        storyEl.style.opacity = '1';
        storyIndex++;
        // Stay visible for 5s, then move on
        setTimeout(showNextLine, line.dur);
      });
    });
  }, 2000); // 1s fade out + 1s dark = 2s gap
}

function skipStory() {
  storyActive = false;
  storyScreen.style.transition = 'opacity 0.6s';
  storyScreen.style.opacity = '0';
  setTimeout(() => { storyScreen.style.display='none'; }, 650);
}
window.skipStory = skipStory;

// ═══════════════════════════════════════════════════════
//  THREE.JS SCENE
// ═══════════════════════════════════════════════════════
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(52, innerWidth/innerHeight, 0.1, 12000);
camera.position.set(0, 90, 200);
camera.lookAt(0,0,0);

const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setSize(innerWidth, innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.insertBefore(renderer.domElement, document.body.firstChild);

// ── LAYERED STARFIELD ────────────────────────────────
function makeStarLayer(count, size, spread, color) {
  const geo = new THREE.BufferGeometry();
  const pos = new Float32Array(count*3);
  for (let i=0; i<count*3; i++) pos[i] = (Math.random()-0.5)*spread;
  geo.setAttribute('position', new THREE.BufferAttribute(pos,3));
  return new THREE.Points(geo, new THREE.PointsMaterial({ color, size, sizeAttenuation:true, transparent:true, opacity:0 }));
}
const starLayers = [
  makeStarLayer(6000, 0.5, 4000, 0xffffff),
  makeStarLayer(3000, 1.0, 3500, 0xffeedd),
  makeStarLayer(1200, 1.6, 3000, 0xaaccff),
  makeStarLayer(500,  2.2, 2500, 0xffd080),
  makeStarLayer(200,  3.0, 2000, 0xff8888),
];
starLayers.forEach(s => scene.add(s));

// Nebula clouds
function makeNebula(x,y,z,col,size) {
  const geo = new THREE.BufferGeometry();
  const pos = new Float32Array(800*3);
  for(let i=0;i<800;i++){
    const r=Math.random()*size, t=Math.random()*Math.PI*2, p=Math.random()*Math.PI;
    pos[i*3]  =x+r*Math.sin(p)*Math.cos(t);
    pos[i*3+1]=y+r*Math.sin(p)*Math.sin(t)*0.3;
    pos[i*3+2]=z+r*Math.cos(p);
  }
  geo.setAttribute('position',new THREE.BufferAttribute(pos,3));
  return new THREE.Points(geo,new THREE.PointsMaterial({color:col,size:2.5,transparent:true,opacity:0,sizeAttenuation:true}));
}
const nebulae=[
  makeNebula(-800,200,-600,0x330055,400),
  makeNebula(900,-100,-800,0x003355,350),
  makeNebula(200,300,-1000,0x440022,300),
];
nebulae.forEach(n=>scene.add(n));

// ── LIGHTING ────────────────────────────────────────
const sunLight = new THREE.PointLight(0xfff0cc, 0, 1500);
sunLight.castShadow = true;
scene.add(sunLight);
const ambLight = new THREE.AmbientLight(0x111133, 0.5);
scene.add(ambLight);

// ── TEXTURE HELPERS ─────────────────────────────────
function makeGradTex(stops, w=512, h=512) {
  const c=document.createElement('canvas'); c.width=w; c.height=h;
  const ctx=c.getContext('2d');
  const g=ctx.createRadialGradient(w/2,h/2,0,w/2,h/2,w/2);
  stops.forEach(([t,col])=>g.addColorStop(t,col));
  ctx.fillStyle=g; ctx.fillRect(0,0,w,h);
  return new THREE.CanvasTexture(c);
}

function makePlanetTex(baseColor, stripeColors, hasAtmo=false) {
  const c=document.createElement('canvas'); c.width=512; c.height=512;
  const ctx=c.getContext('2d');
  ctx.fillStyle=baseColor; ctx.fillRect(0,0,512,512);
  if(stripeColors && stripeColors.length){
    stripeColors.forEach(([y,h,col])=>{
      const g2=ctx.createLinearGradient(0,y,0,y+h);
      g2.addColorStop(0,'rgba(0,0,0,0)');
      g2.addColorStop(0.5,col);
      g2.addColorStop(1,'rgba(0,0,0,0)');
      ctx.fillStyle=g2; ctx.fillRect(0,y,512,h);
    });
  }
  const id=ctx.getImageData(0,0,512,512);
  for(let i=0;i<id.data.length;i+=4){
    const n=(Math.random()-0.5)*22;
    id.data[i]  =Math.min(255,Math.max(0,id.data[i]+n));
    id.data[i+1]=Math.min(255,Math.max(0,id.data[i+1]+n));
    id.data[i+2]=Math.min(255,Math.max(0,id.data[i+2]+n));
  }
  ctx.putImageData(id,0,0);
  if(hasAtmo){
    const rim=ctx.createRadialGradient(256,256,180,256,256,256);
    rim.addColorStop(0,'rgba(0,0,0,0)');
    rim.addColorStop(1,'rgba(100,180,255,0.25)');
    ctx.fillStyle=rim; ctx.fillRect(0,0,512,512);
  }
  return new THREE.CanvasTexture(c);
}

// ── DRINK-THEMED PLANET TEXTURES ─────────────────────
function makeBeerTex() {
  const c=document.createElement('canvas'); c.width=512; c.height=512;
  const ctx=c.getContext('2d');
  // Golden amber body
  const bg=ctx.createLinearGradient(0,0,512,512);
  bg.addColorStop(0,'#e8a010'); bg.addColorStop(0.5,'#d48000'); bg.addColorStop(1,'#b06000');
  ctx.fillStyle=bg; ctx.fillRect(0,0,512,512);
  // Swirling hop bands
  for(let b=0;b<6;b++){
    const by=60+b*72, bh=35;
    const bg2=ctx.createLinearGradient(0,by,0,by+bh);
    bg2.addColorStop(0,'rgba(0,0,0,0)');
    bg2.addColorStop(0.5,`rgba(${180+b*10},${120+b*5},10,0.5)`);
    bg2.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=bg2; ctx.fillRect(0,by,512,bh);
  }
  // Foam polar cap — top
  const foamGrad=ctx.createRadialGradient(256,40,0,256,40,200);
  foamGrad.addColorStop(0,'rgba(255,255,240,0.98)');
  foamGrad.addColorStop(0.4,'rgba(240,235,210,0.85)');
  foamGrad.addColorStop(0.7,'rgba(220,200,150,0.4)');
  foamGrad.addColorStop(1,'rgba(0,0,0,0)');
  ctx.fillStyle=foamGrad; ctx.fillRect(0,0,512,260);
  // Foam bubbles
  ctx.fillStyle='rgba(255,255,250,0.7)';
  for(let i=0;i<60;i++){
    const bx=Math.random()*512, by=Math.random()*130, br=Math.random()*8+2;
    ctx.beginPath(); ctx.arc(bx,by,br,0,Math.PI*2); ctx.fill();
  }
  // Noise
  const id=ctx.getImageData(0,0,512,512);
  for(let i=0;i<id.data.length;i+=4){const n=(Math.random()-0.5)*18; id.data[i]=Math.min(255,Math.max(0,id.data[i]+n)); id.data[i+1]=Math.min(255,Math.max(0,id.data[i+1]+n)); id.data[i+2]=Math.min(255,Math.max(0,id.data[i+2]+n));}
  ctx.putImageData(id,0,0);
  return new THREE.CanvasTexture(c);
}

function makeTequilaTex() {
  const c=document.createElement('canvas'); c.width=512; c.height=512;
  const ctx=c.getContext('2d');
  // Clear golden body
  const bg=ctx.createRadialGradient(256,256,0,256,256,300);
  bg.addColorStop(0,'#f0e060'); bg.addColorStop(0.5,'#c8b020'); bg.addColorStop(1,'#806000');
  ctx.fillStyle=bg; ctx.fillRect(0,0,512,512);
  // Lime-green swirl bands
  for(let i=0;i<5;i++){
    const sy=40+i*90, sh=55;
    const sg=ctx.createLinearGradient(0,sy,0,sy+sh);
    sg.addColorStop(0,'rgba(0,0,0,0)');
    sg.addColorStop(0.5,'rgba(80,200,40,0.55)');
    sg.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=sg; ctx.fillRect(0,sy,512,sh);
  }
  // Lime-green glow overlay
  const lg=ctx.createRadialGradient(256,256,80,256,256,280);
  lg.addColorStop(0,'rgba(100,255,60,0)');
  lg.addColorStop(0.6,'rgba(80,200,30,0.12)');
  lg.addColorStop(1,'rgba(40,160,0,0.35)');
  ctx.fillStyle=lg; ctx.fillRect(0,0,512,512);
  const id=ctx.getImageData(0,0,512,512);
  for(let i=0;i<id.data.length;i+=4){const n=(Math.random()-0.5)*20; id.data[i]=Math.min(255,Math.max(0,id.data[i]+n)); id.data[i+1]=Math.min(255,Math.max(0,id.data[i+1]+n)); id.data[i+2]=Math.min(255,Math.max(0,id.data[i+2]+n));}
  ctx.putImageData(id,0,0);
  return new THREE.CanvasTexture(c);
}

function makeMargaritasTex() {
  const c=document.createElement('canvas'); c.width=512; c.height=512;
  const ctx=c.getContext('2d');
  // Icy teal-green base
  const bg=ctx.createRadialGradient(256,256,0,256,256,300);
  bg.addColorStop(0,'#c8f0e0'); bg.addColorStop(0.4,'#80c8a0'); bg.addColorStop(1,'#206040');
  ctx.fillStyle=bg; ctx.fillRect(0,0,512,512);
  // Frosted salt-rim shimmer bands
  for(let i=0;i<8;i++){
    const sy=i*65, sh=30;
    const sg=ctx.createLinearGradient(0,sy,0,sy+sh);
    sg.addColorStop(0,'rgba(0,0,0,0)');
    sg.addColorStop(0.5,'rgba(220,255,240,0.45)');
    sg.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=sg; ctx.fillRect(0,sy,512,sh);
  }
  // Frost crystal sparkle overlay
  ctx.fillStyle='rgba(200,255,230,0.5)';
  for(let i=0;i<120;i++){
    const fx=Math.random()*512, fy=Math.random()*512, fr=Math.random()*3+0.5;
    ctx.beginPath(); ctx.arc(fx,fy,fr,0,Math.PI*2); ctx.fill();
  }
  // Icy polar cap
  const fg=ctx.createRadialGradient(256,20,0,256,20,180);
  fg.addColorStop(0,'rgba(255,255,255,0.9)');
  fg.addColorStop(0.5,'rgba(200,240,220,0.5)');
  fg.addColorStop(1,'rgba(0,0,0,0)');
  ctx.fillStyle=fg; ctx.fillRect(0,0,512,200);
  const id=ctx.getImageData(0,0,512,512);
  for(let i=0;i<id.data.length;i+=4){const n=(Math.random()-0.5)*15; id.data[i]=Math.min(255,Math.max(0,id.data[i]+n)); id.data[i+1]=Math.min(255,Math.max(0,id.data[i+1]+n)); id.data[i+2]=Math.min(255,Math.max(0,id.data[i+2]+n));}
  ctx.putImageData(id,0,0);
  return new THREE.CanvasTexture(c);
}

function makeWhiskeyTex() {
  const c=document.createElement('canvas'); c.width=512; c.height=512;
  const ctx=c.getContext('2d');
  // Deep amber base
  const bg=ctx.createRadialGradient(256,256,0,256,256,300);
  bg.addColorStop(0,'#c06010'); bg.addColorStop(0.5,'#883000'); bg.addColorStop(1,'#441000');
  ctx.fillStyle=bg; ctx.fillRect(0,0,512,512);
  // Amber swirl stripes — angled for drama
  ctx.save();
  ctx.translate(256,256); ctx.rotate(0.4);
  for(let i=0;i<7;i++){
    const sy=-300+i*90, sh=50;
    const sg=ctx.createLinearGradient(0,sy,0,sy+sh);
    sg.addColorStop(0,'rgba(0,0,0,0)');
    sg.addColorStop(0.5,`rgba(${200+i*8},${100+i*6},${10+i*3},0.6)`);
    sg.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=sg; ctx.fillRect(-300,sy,600,sh);
  }
  ctx.restore();
  // Oak barrel-grain vertical streaks
  for(let i=0;i<12;i++){
    const vx=Math.random()*512;
    const vg=ctx.createLinearGradient(vx,0,vx+8,512);
    vg.addColorStop(0,'rgba(60,20,0,0)');
    vg.addColorStop(0.4,'rgba(60,20,0,0.2)');
    vg.addColorStop(1,'rgba(60,20,0,0)');
    ctx.fillStyle=vg; ctx.fillRect(vx,0,8,512);
  }
  const id=ctx.getImageData(0,0,512,512);
  for(let i=0;i<id.data.length;i+=4){const n=(Math.random()-0.5)*25; id.data[i]=Math.min(255,Math.max(0,id.data[i]+n)); id.data[i+1]=Math.min(255,Math.max(0,id.data[i+1]+n)); id.data[i+2]=Math.min(255,Math.max(0,id.data[i+2]+n));}
  ctx.putImageData(id,0,0);
  return new THREE.CanvasTexture(c);
}

// ── SUN ─────────────────────────────────────────────
const sunTex = makeGradTex([[0,'#fffacc'],[0.2,'#ffdd00'],[0.55,'#ff8800'],[0.85,'#dd2200'],[1,'#880000']]);
const sun = new THREE.Mesh(
  new THREE.SphereGeometry(12,64,64),
  new THREE.MeshBasicMaterial({map:sunTex})
);
sun.visible = false;
scene.add(sun);

// Sun corona layers
for(let i=0;i<3;i++){
  const cMesh=new THREE.Mesh(
    new THREE.SphereGeometry(13+i*1.8,32,32),
    new THREE.MeshBasicMaterial({color:[0xffcc00,0xff8800,0xff4400][i],transparent:true,opacity:[0.07,0.04,0.02][i],side:THREE.BackSide})
  );
  sun.add(cMesh);
}

// ── ORBIT RINGS ─────────────────────────────────────
function makeOrbitRing(r) {
  const pts=[];
  for(let i=0;i<=128;i++){const a=(i/128)*Math.PI*2; pts.push(new THREE.Vector3(Math.cos(a)*r,0,Math.sin(a)*r));}
  const g=new THREE.BufferGeometry().setFromPoints(pts);
  const m=new THREE.LineBasicMaterial({color:0x223344,transparent:true,opacity:0.4});
  const l=new THREE.Line(g,m); l.visible=false; return l;
}

// ── PLANET DEFINITIONS ──────────────────────────────
const planetDefs = [
  { name:'Beer',      key:'Beer',      r:1.8, orbit:22,  speed:4.74, tilt:0,
    tex:()=>makeBeerTex(),
    emissive:0x442200, emi:0.2 },
  { name:'Vodka',     key:'Vodka',     r:3.2, orbit:36,  speed:1.21, tilt:177,
    tex:()=>makePlanetTex('#c8e8ff',[[100,80,'#ffffff',1],[240,60,'#90c0ff',1],[360,70,'#ddeeff',1]]),
    emissive:0x001122, emi:0.1 },
  { name:'State 18',  key:'State18',   r:3.8, orbit:52,  speed:1.0,  tilt:23.4,
    tex:()=>makePlanetTex('#1a6a9a',[[60,40,'#2db050',1],[160,30,'#1a7a3a',1],[280,50,'#3399cc',1],[400,60,'#1a5cbf',1]], true),
    emissive:0x001a44, emi:0.12 },
  { name:'Gin',       key:'Gin',       r:2.5, orbit:70,  speed:0.532,tilt:25,
    tex:()=>makePlanetTex('#cc3010',[[80,50,'#ff6040',1],[200,40,'#aa2008',1],[320,60,'#ff5030',1]]),
    emissive:0x440000, emi:0.2 },
  { name:'Tequila',   key:'Tequila',   r:8.5, orbit:104, speed:0.084,tilt:3,
    tex:()=>makeTequilaTex(),
    emissive:0x1a3300, emi:0.18, glowColor:0x44ff00, glowOpacity:0.1 },
  { name:'Margaritas',key:'Margaritas',r:7.2, orbit:134, speed:0.034,tilt:27,
    tex:()=>makeMargaritasTex(),
    emissive:0x003322, emi:0.18, rings:true },
  { name:'Whiskey',   key:'Whiskey',   r:5.2, orbit:162, speed:0.012,tilt:98,
    tex:()=>makeWhiskeyTex(),
    emissive:0x331100, emi:0.2 },
  { name:'Rum',       key:'Rum',       r:5.0, orbit:188, speed:0.006,tilt:28,
    tex:()=>makePlanetTex('#3050cc',[[90,50,'#6080ff',1],[210,60,'#2040aa',1],[350,55,'#5070ee',1]]),
    emissive:0x000033, emi:0.2 },
];

const planets = {};
const orbitRings = [];

planetDefs.forEach(def => {
  const ring = makeOrbitRing(def.orbit);
  scene.add(ring);
  orbitRings.push(ring);

  const pivot = new THREE.Object3D();
  pivot.rotation.y = Math.random()*Math.PI*2;
  scene.add(pivot);

  const mesh = new THREE.Mesh(
    new THREE.SphereGeometry(def.r,64,64),
    new THREE.MeshPhongMaterial({
      map: def.tex(),
      emissive: new THREE.Color(def.emissive),
      emissiveIntensity: def.emi,
      shininess: def.name==='State 18'?80:30,
      specular: new THREE.Color(0x334455)
    })
  );
  mesh.rotation.z = THREE.MathUtils.degToRad(def.tilt);
  mesh.position.x = def.orbit;
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  mesh.visible = false;
  pivot.add(mesh);

  // Atmosphere glow for State 18
  if(def.name==='State 18'){
    const atmoMesh=new THREE.Mesh(
      new THREE.SphereGeometry(def.r*1.08,32,32),
      new THREE.MeshBasicMaterial({color:0x3399ff,transparent:true,opacity:0.08,side:THREE.BackSide})
    );
    mesh.add(atmoMesh);
  }

  // Rings for Margaritas
  if(def.rings){
    const rGeo=new THREE.RingGeometry(def.r*1.35,def.r*2.3,80);
    const rMat=new THREE.MeshBasicMaterial({color:0x88cc44,transparent:true,opacity:0.5,side:THREE.DoubleSide});
    const rMesh=new THREE.Mesh(rGeo,rMat);
    rMesh.rotation.x=Math.PI/2.2;
    mesh.add(rMesh);
  }

  // Moon for State 18
  if(def.name==='State 18'){
    const moonPiv=new THREE.Object3D();
    const moonMesh=new THREE.Mesh(
      new THREE.SphereGeometry(0.9,24,24),
      new THREE.MeshPhongMaterial({color:0xbbbbbb,shininess:10})
    );
    moonMesh.position.x=6.5;
    moonPiv.add(moonMesh);
    mesh.add(moonPiv);
    planets['Moon']={mesh:moonMesh,pivot:moonPiv};
  }

  planets[def.key] = {mesh, pivot, speed:def.speed, orbitR:def.orbit, ring};
});

// ── ATHENA GODDESS FACE TEXTURE ─────────────────────
function makeAthenaFaceTex(){
  const c=document.createElement('canvas'); c.width=1024; c.height=1024;
  const ctx=c.getContext('2d');
  const bg=ctx.createRadialGradient(512,512,0,512,512,512);
  bg.addColorStop(0,'#2a0a4a'); bg.addColorStop(0.5,'#180630'); bg.addColorStop(1,'#0a0018');
  ctx.fillStyle=bg; ctx.fillRect(0,0,1024,1024);
  for(let i=0;i<18;i++){
    const nx=Math.random()*1024,ny=Math.random()*1024,nr=Math.random()*120+40;
    const ng=ctx.createRadialGradient(nx,ny,0,nx,ny,nr);
    ng.addColorStop(0,`rgba(${80+Math.random()*60},${20+Math.random()*30},${140+Math.random()*80},0.18)`);
    ng.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=ng; ctx.beginPath(); ctx.arc(nx,ny,nr,0,Math.PI*2); ctx.fill();
  }
  ctx.save(); ctx.translate(512,490);
  const faceGrad=ctx.createRadialGradient(0,-20,10,0,10,210);
  faceGrad.addColorStop(0,'#e8b87a'); faceGrad.addColorStop(0.35,'#d4966a');
  faceGrad.addColorStop(0.7,'#b87040'); faceGrad.addColorStop(1,'#7a3a10');
  ctx.fillStyle=faceGrad; ctx.scale(1,1.28);
  ctx.beginPath(); ctx.arc(0,0,190,0,Math.PI*2); ctx.fill();
  ctx.restore();
  function cheek(cx2,cy2){
    const g=ctx.createRadialGradient(cx2,cy2,0,cx2,cy2,60);
    g.addColorStop(0,'rgba(200,90,60,0.22)'); g.addColorStop(1,'rgba(200,90,60,0)');
    ctx.fillStyle=g; ctx.beginPath(); ctx.arc(cx2,cy2,60,0,Math.PI*2); ctx.fill();
  }
  cheek(390,540); cheek(635,540);
  ctx.save();
  const hairGrad=ctx.createLinearGradient(512,200,512,700);
  hairGrad.addColorStop(0,'#1a0800'); hairGrad.addColorStop(0.4,'#3d1a00'); hairGrad.addColorStop(1,'#2a0f00');
  ctx.fillStyle=hairGrad;
  ctx.beginPath(); ctx.moveTo(280,480); ctx.bezierCurveTo(260,300,340,180,512,170);
  ctx.bezierCurveTo(684,180,760,300,740,480); ctx.bezierCurveTo(720,350,650,240,512,230);
  ctx.bezierCurveTo(374,240,300,350,280,480); ctx.fill();
  ctx.beginPath(); ctx.moveTo(310,350); ctx.bezierCurveTo(240,420,200,520,210,680);
  ctx.bezierCurveTo(220,760,270,820,290,870); ctx.bezierCurveTo(270,800,240,720,255,640);
  ctx.bezierCurveTo(265,560,295,470,330,400); ctx.fill();
  ctx.beginPath(); ctx.moveTo(714,350); ctx.bezierCurveTo(780,420,820,520,810,680);
  ctx.bezierCurveTo(800,760,750,820,730,870); ctx.bezierCurveTo(750,800,780,720,765,640);
  ctx.bezierCurveTo(755,560,725,470,690,400); ctx.fill();
  const hsh=ctx.createLinearGradient(430,200,560,280);
  hsh.addColorStop(0,'rgba(120,60,10,0)'); hsh.addColorStop(0.5,'rgba(160,90,20,0.35)'); hsh.addColorStop(1,'rgba(120,60,10,0)');
  ctx.fillStyle=hsh; ctx.beginPath(); ctx.ellipse(500,235,55,80,0.2,0,Math.PI*2); ctx.fill();
  ctx.restore();
  function drawEye(ex,ey,dir){
    ctx.save(); ctx.translate(ex,ey);
    const esock=ctx.createRadialGradient(0,0,0,0,0,38);
    esock.addColorStop(0,'rgba(60,20,0,0.5)'); esock.addColorStop(1,'rgba(60,20,0,0)');
    ctx.fillStyle=esock; ctx.beginPath(); ctx.ellipse(0,0,38,22,0,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='#f0e8d8'; ctx.beginPath(); ctx.ellipse(0,0,32,16,0,0,Math.PI*2); ctx.fill();
    const iris=ctx.createRadialGradient(2,-2,1,0,0,14);
    iris.addColorStop(0,'#7a4010'); iris.addColorStop(0.4,'#4a2008'); iris.addColorStop(0.85,'#1a0800'); iris.addColorStop(1,'#000');
    ctx.fillStyle=iris; ctx.beginPath(); ctx.arc(0,0,14,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='#050100'; ctx.beginPath(); ctx.arc(1,-1,7,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='rgba(255,240,210,0.9)'; ctx.beginPath(); ctx.arc(-4,-5,3.5,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='rgba(255,255,255,0.5)'; ctx.beginPath(); ctx.arc(4,-3,1.5,0,Math.PI*2); ctx.fill();
    ctx.strokeStyle='#0a0400'; ctx.lineWidth=4;
    ctx.beginPath(); ctx.moveTo(-30,-2); ctx.bezierCurveTo(-15,-20,15,-20,30,-2); ctx.stroke();
    ctx.strokeStyle='#050200'; ctx.lineWidth=2;
    for(let i=0;i<8;i++){const lx=-24+i*7,ly=-8+Math.cos(i*0.7)*4; ctx.beginPath(); ctx.moveTo(lx,ly); ctx.lineTo(lx+dir*(i%2===0?3:2),ly-7); ctx.stroke();}
    ctx.strokeStyle='rgba(180,100,50,0.4)'; ctx.lineWidth=1.5;
    ctx.beginPath(); ctx.moveTo(-28,6); ctx.bezierCurveTo(-10,16,10,16,28,6); ctx.stroke();
    ctx.strokeStyle='#1a0800'; ctx.lineWidth=5; ctx.lineCap='round';
    ctx.beginPath(); ctx.moveTo(-34,-26); ctx.bezierCurveTo(-10,-42,14,-40,34,-28); ctx.stroke();
    ctx.restore();
  }
  drawEye(390,460,-1); drawEye(635,460,1);
  ctx.save(); ctx.translate(512,530);
  ctx.strokeStyle='rgba(120,55,10,0.55)'; ctx.lineWidth=2.5; ctx.lineCap='round';
  ctx.beginPath(); ctx.moveTo(-8,-60); ctx.bezierCurveTo(-10,-20,-12,20,-14,60); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(8,-60); ctx.bezierCurveTo(10,-20,12,20,14,60); ctx.stroke();
  ctx.beginPath(); ctx.ellipse(-18,58,14,8,-0.3,0,Math.PI*2); ctx.stroke();
  ctx.beginPath(); ctx.ellipse(18,58,14,8,0.3,0,Math.PI*2); ctx.stroke();
  ctx.restore();
  ctx.save(); ctx.translate(512,650);
  const lipLow=ctx.createLinearGradient(0,-5,0,30);
  lipLow.addColorStop(0,'#c04040'); lipLow.addColorStop(1,'#8a2020');
  ctx.fillStyle=lipLow; ctx.beginPath(); ctx.moveTo(-55,0);
  ctx.bezierCurveTo(-30,40,30,40,55,0); ctx.bezierCurveTo(30,10,-30,10,-55,0); ctx.fill();
  const lipUp=ctx.createLinearGradient(0,-30,0,5);
  lipUp.addColorStop(0,'#b83535'); lipUp.addColorStop(1,'#902828');
  ctx.fillStyle=lipUp; ctx.beginPath(); ctx.moveTo(-55,0);
  ctx.bezierCurveTo(-35,-25,-12,-32,0,-28); ctx.bezierCurveTo(12,-32,35,-25,55,0);
  ctx.bezierCurveTo(30,-8,-30,-8,-55,0); ctx.fill();
  ctx.fillStyle='rgba(255,180,160,0.35)'; ctx.beginPath(); ctx.ellipse(0,-12,25,7,0,0,Math.PI*2); ctx.fill();
  ctx.restore();
  ctx.save();
  const goldGrad=ctx.createLinearGradient(250,290,760,310);
  goldGrad.addColorStop(0,'#7a5500'); goldGrad.addColorStop(0.2,'#ffd060');
  goldGrad.addColorStop(0.5,'#ffe890'); goldGrad.addColorStop(0.8,'#ffd060'); goldGrad.addColorStop(1,'#7a5500');
  ctx.fillStyle=goldGrad;
  ctx.beginPath(); ctx.moveTo(290,310); ctx.bezierCurveTo(380,285,630,285,724,310);
  ctx.bezierCurveTo(630,325,380,325,290,310); ctx.fill();
  const gem=ctx.createRadialGradient(512,302,0,512,302,18);
  gem.addColorStop(0,'#fff'); gem.addColorStop(0.3,'#cc88ff'); gem.addColorStop(1,'#440088');
  ctx.fillStyle=gem; ctx.beginPath(); ctx.arc(512,302,18,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle='#ffd060'; ctx.lineWidth=2; ctx.stroke();
  ctx.restore();
  const aura=ctx.createRadialGradient(512,490,160,512,490,340);
  aura.addColorStop(0,'rgba(150,50,255,0)'); aura.addColorStop(0.7,'rgba(120,30,220,0.08)'); aura.addColorStop(1,'rgba(80,0,180,0.45)');
  ctx.fillStyle=aura; ctx.fillRect(0,0,1024,1024);
  ctx.fillStyle='rgba(255,230,255,0.7)';
  [[300,310],[724,310],[350,600],[670,600],[320,450],[710,450]].forEach(([sx,sy])=>{ctx.beginPath();ctx.arc(sx,sy,2,0,Math.PI*2);ctx.fill();});
  return new THREE.CanvasTexture(c);
}

// ── ATHENA MESH ──────────────────────────────────────
const athenaMesh = new THREE.Mesh(
  new THREE.SphereGeometry(3.5,64,64),
  new THREE.MeshPhongMaterial({map:makeAthenaFaceTex(),emissive:0x220044,emissiveIntensity:0.3,shininess:70})
);
const athenaGlow = new THREE.Mesh(
  new THREE.SphereGeometry(5,32,32),
  new THREE.MeshBasicMaterial({color:0xaa44ff,transparent:true,opacity:0.1,side:THREE.BackSide})
);
athenaMesh.add(athenaGlow);
athenaMesh.visible=false;
scene.add(athenaMesh);

// Trail
const TRAIL_COUNT=400;
const trailGeo=new THREE.BufferGeometry();
const trailPos=new Float32Array(TRAIL_COUNT*3);
trailGeo.setAttribute('position',new THREE.BufferAttribute(trailPos,3));
const trailMat=new THREE.PointsMaterial({color:0xcc88ff,size:1.4,transparent:true,opacity:0,sizeAttenuation:true});
const trail=new THREE.Points(trailGeo,trailMat);
scene.add(trail);
const trailHistory=[];

// Explosion
const EX=700;
const exGeo=new THREE.BufferGeometry();
const exPos=new Float32Array(EX*3);
const exVel=[];
for(let i=0;i<EX;i++){
  const th=Math.random()*Math.PI*2,ph=Math.random()*Math.PI,sp=Math.random()*2.5+0.5;
  exVel.push(new THREE.Vector3(Math.sin(ph)*Math.cos(th)*sp,Math.cos(ph)*sp,Math.sin(ph)*Math.sin(th)*sp));
}
exGeo.setAttribute('position',new THREE.BufferAttribute(exPos,3));
const exMat=new THREE.PointsMaterial({color:0xff6020,size:2.8,transparent:true,opacity:0,sizeAttenuation:true});
const exParts=new THREE.Points(exGeo,exMat);
exParts.visible=false;
scene.add(exParts);

// Second explosion ring — smaller, cooler debris
const EX2=400;
const exGeo2=new THREE.BufferGeometry();
const exPos2=new Float32Array(EX2*3);
const exVel2=[];
for(let i=0;i<EX2;i++){
  const th=Math.random()*Math.PI*2, ph=(Math.random()-0.5)*0.6, sp=Math.random()*1.5+0.3;
  exVel2.push(new THREE.Vector3(Math.cos(th)*Math.cos(ph)*sp, Math.sin(ph)*sp, Math.sin(th)*Math.cos(ph)*sp));
}
exGeo2.setAttribute('position',new THREE.BufferAttribute(exPos2,3));
const exMat2=new THREE.PointsMaterial({color:0xaa44ff,size:1.8,transparent:true,opacity:0,sizeAttenuation:true});
const exParts2=new THREE.Points(exGeo2,exMat2);
exParts2.visible=false;
scene.add(exParts2);

// ── SHOCKWAVE RING ───────────────────────────────────
const shockGeo=new THREE.RingGeometry(0.1,0.8,64);
const shockMat=new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:0,side:THREE.DoubleSide});
const shockRing=new THREE.Mesh(shockGeo,shockMat);
shockRing.visible=false;
scene.add(shockRing);

// Second shockwave (delayed, purple)
const shock2Mat=new THREE.MeshBasicMaterial({color:0xcc88ff,transparent:true,opacity:0,side:THREE.DoubleSide});
const shockRing2=new THREE.Mesh(new THREE.RingGeometry(0.1,0.8,64),shock2Mat);
shockRing2.visible=false;
scene.add(shockRing2);

// ── WHITE FLASH OVERLAY ──────────────────────────────
const whiteFlashDiv=document.createElement('div');
whiteFlashDiv.style.cssText='position:fixed;inset:0;background:#fff;opacity:0;pointer-events:none;z-index:50;transition:opacity 0.05s;';
document.body.appendChild(whiteFlashDiv);

// ── CAMERA SHAKE ─────────────────────────────────────
let shakeT=0, shakeDur=0, shakeMag=0;
function triggerShake(dur, mag){ shakeT=0; shakeDur=dur; shakeMag=mag; }

// ── TWINKLING STARS ──────────────────────────────────
// Store per-star twinkle phase and speed
const twinkleData=[];
starLayers.forEach(layer=>{
  const count=layer.geometry.attributes.position.count;
  const phases=new Float32Array(count);
  const speeds=new Float32Array(count);
  const baseSizes=new Float32Array(count);
  for(let i=0;i<count;i++){
    phases[i]=Math.random()*Math.PI*2;
    speeds[i]=0.5+Math.random()*2.5;
    baseSizes[i]=layer.material.size;
  }
  twinkleData.push({layer,phases,speeds,baseSizes,count});
});

function updateTwinkle(t){
  const baseOpacities=[0.9,0.8,0.7,0.8,0.9];
  twinkleData.forEach(function({layer,baseSizes},i){
    const pulse=0.65+0.35*Math.abs(Math.sin(t*(0.38+i*0.13)+i*1.7));
    layer.material.opacity=baseOpacities[i]*pulse;
    layer.material.size=baseSizes[0]*(0.75+0.25*Math.abs(Math.sin(t*(0.55+i*0.1)+i*2.1)));
  });
}

// ── LABELS ──────────────────────────────────────────
const labelsContainer=document.getElementById('labels-container');
const labelObjects=[];

function makeLabel(text, cssClass, offsetY) {
  const div=document.createElement('div');
  div.className=`planet-label ${cssClass}`;
  div.innerHTML=`<span>${text}</span><div class="dot"></div>`;
  labelsContainer.appendChild(div);
  div.style.opacity='0';
  return div;
}

const sunLbl=makeLabel('Quad Cities','lbl-QuadCities',14);
labelObjects.push({el:sunLbl, getMesh:()=>sun, offsetY:14});

planetDefs.forEach(def=>{
  const lbl=makeLabel(def.name,`lbl-${def.key}`,def.r+2);
  labelObjects.push({el:lbl, getMesh:()=>planets[def.key].mesh, offsetY:def.r+2});
});

const athenaLbl=makeLabel('Athena','lbl-Athena',5);
athenaLbl.style.display='none';
labelObjects.push({el:athenaLbl, getMesh:()=>athenaMesh, offsetY:5, athena:true});

const _sv=new THREE.Vector3();

// ── SIMULATION PHASES ────────────────────────────────

function updateLabels(){
  labelObjects.forEach(item=>{
    if(item.athena){
      item.el.style.display=(PHASE_CUR===PHASE.APPROACH&&athenaMesh.visible)?'flex':'none';
    }
    if(item.ufo){ item.el.style.display='flex'; }
    if(item.el.style.display==='none'&&(item.athena||item.ufo)) return;
    item.getMesh().getWorldPosition(_sv);
    _sv.y+=(item.offsetY||0);
    _sv.project(camera);
    if(_sv.z>1){item.el.style.opacity='0';return;}
    const x=(_sv.x*0.5+0.5)*innerWidth;
    const y=(-_sv.y*0.5+0.5)*innerHeight;
    item.el.style.left=x+'px';
    item.el.style.top=y+'px';
    const off=x<-60||x>innerWidth+60||y<-40||y>innerHeight+40;
    item.el.style.opacity=(off||PHASE_CUR===PHASE.STORY||PHASE_CUR===PHASE.FORMING||PHASE_CUR===PHASE.TAKEOVER)?'0':'1';
  });
}

// ── SIMULATION PHASES ────────────────────────────────
let phaseT=0;
let formingT=0;
let earthWorldPos=new THREE.Vector3();
let impactPos=new THREE.Vector3();
let explodeActive=false;
let explodeT=0;
let impactFlashT=0;
let cameraTarget=new THREE.Vector3(0,90,200);
let cameraLookTarget=new THREE.Vector3(0,0,0);

const phaseLabel=document.getElementById('phase-label');
const athenaInfo=document.getElementById('athena-info');
const impactOverlay=document.getElementById('impact-overlay');
const takeover=document.getElementById('takeover-screen');
const approachStory=document.getElementById('approach-story');
const mainUI=document.getElementById('ui');

const athenaStart=new THREE.Vector3(700,120,350);

const approachLines=[
  "A rogue world has entered the Solar System...",
  "Destination: State 18.",
  "Impact... inevitable."
];
let approachLineIdx=0;

function setPhaseLabel(t){phaseLabel.textContent=t;}

// ── UFO / UBER ────────────────────────────────────────
function makeUFOTex(){
  const c=document.createElement('canvas'); c.width=256; c.height=128;
  const ctx=c.getContext('2d');
  // Body
  const bg=ctx.createRadialGradient(128,64,0,128,64,100);
  bg.addColorStop(0,'#aaeeff'); bg.addColorStop(0.5,'#4488cc'); bg.addColorStop(1,'#112244');
  ctx.fillStyle=bg; ctx.fillRect(0,0,256,128);
  // Window strip
  const ws=ctx.createLinearGradient(0,48,0,80);
  ws.addColorStop(0,'rgba(180,240,255,0.8)'); ws.addColorStop(0.5,'rgba(80,200,255,0.6)'); ws.addColorStop(1,'rgba(180,240,255,0.8)');
  ctx.fillStyle=ws; ctx.fillRect(0,48,256,32);
  // Portholes
  for(let i=0;i<5;i++){
    ctx.beginPath(); ctx.arc(30+i*50,64,9,0,Math.PI*2);
    ctx.fillStyle='rgba(255,255,200,0.9)'; ctx.fill();
    ctx.strokeStyle='rgba(100,200,255,0.6)'; ctx.lineWidth=2; ctx.stroke();
  }
  return new THREE.CanvasTexture(c);
}

// UFO: flat disc shape
const ufoGroup = new THREE.Group();
// Main saucer disc
const ufoDisk = new THREE.Mesh(
  new THREE.CylinderGeometry(5,6,1.2,32),
  new THREE.MeshPhongMaterial({color:0x88ccee,emissive:0x003355,emissiveIntensity:0.4,shininess:120,specular:0x88ffff})
);
ufoGroup.add(ufoDisk);
// Dome on top
const ufoDome = new THREE.Mesh(
  new THREE.SphereGeometry(3,24,12,0,Math.PI*2,0,Math.PI/2),
  new THREE.MeshPhongMaterial({color:0xaaddff,emissive:0x002244,emissiveIntensity:0.5,transparent:true,opacity:0.85,shininess:200})
);
ufoDome.position.y=0.8;
ufoGroup.add(ufoDome);
// Rotating light ring underneath
const ufoRingGeo=new THREE.TorusGeometry(4.5,0.3,8,32);
const ufoRingMat=new THREE.MeshBasicMaterial({color:0x00ffff,transparent:true,opacity:0.8});
const ufoRing=new THREE.Mesh(ufoRingGeo,ufoRingMat);
ufoRing.rotation.x=Math.PI/2;
ufoRing.position.y=-0.5;
ufoGroup.add(ufoRing);
// Blinking landing lights
const lightColors=[0xff4400,0x00ff88,0xffcc00,0xff00aa];
const ufoLights=[];
for(let i=0;i<4;i++){
  const ang=(i/4)*Math.PI*2;
  const lm=new THREE.Mesh(
    new THREE.SphereGeometry(0.4,8,8),
    new THREE.MeshBasicMaterial({color:lightColors[i],transparent:true,opacity:1})
  );
  lm.position.set(Math.cos(ang)*5.2,- 0.6,Math.sin(ang)*5.2);
  ufoGroup.add(lm);
  ufoLights.push({mesh:lm,phase:i*Math.PI/2});
}

// "UBER" label glow light
const ufoPointLight=new THREE.PointLight(0x00ffff,0.8,30);
ufoPointLight.position.y=-1;
ufoGroup.add(ufoPointLight);

ufoGroup.position.set(120,30,60);
ufoGroup.scale.set(0.7,0.7,0.7);
scene.add(ufoGroup);

// UFO state — erratic flight
let ufoTheta=0;

// UFO label
const ufoLbl=makeLabel('Uber','lbl-Uber',8);
labelObjects.push({el:ufoLbl, getMesh:()=>ufoGroup, offsetY:8, ufo:true});
let ufoTargetR=130, ufoCurrentR=130;
let ufoTargetH=28,  ufoCurrentH=28;
let ufoTargetSpeed=0.22, ufoCurrentSpeed=0.22;
let ufoTargetTiltX=0, ufoCurrentTiltX=0;
let ufoTargetTiltZ=0, ufoCurrentTiltZ=0;
let ufoChangeCooldown=0;
let ufoBoostT=0; // sudden burst timer
let ufoWobble=0;

function updateUFO(dt, t){
  // Randomly decide to change orbit/speed/height
  ufoChangeCooldown-=dt;
  if(ufoChangeCooldown<=0){
    // Pick a random new behavior
    const roll=Math.random();
    if(roll<0.25){
      // Dart to inner orbit
      ufoTargetR=20+Math.random()*60;
      ufoTargetSpeed=0.4+Math.random()*0.6;
      ufoTargetH=(Math.random()-0.5)*50;
    } else if(roll<0.5){
      // Cruise outer orbit
      ufoTargetR=100+Math.random()*100;
      ufoTargetSpeed=0.1+Math.random()*0.2;
      ufoTargetH=(Math.random()-0.5)*40;
    } else if(roll<0.7){
      // Sudden burst — speed way up briefly
      ufoTargetSpeed=0.8+Math.random()*1.2;
      ufoBoostT=1.5+Math.random();
    } else if(roll<0.85){
      // Cross to middle system and tilt dramatically
      ufoTargetR=60+Math.random()*80;
      ufoTargetH=30+Math.random()*40;
      ufoTargetTiltX=(Math.random()-0.5)*0.6;
      ufoTargetTiltZ=(Math.random()-0.5)*0.6;
    } else {
      // Quick reverse direction tease
      ufoTargetSpeed*=-0.5;
      ufoTargetH=(Math.random()-0.5)*30;
      ufoTargetTiltX=(Math.random()-0.5)*0.4;
    }
    // Reset tilt sometimes
    if(Math.random()<0.3){ ufoTargetTiltX=0; ufoTargetTiltZ=0; }
    ufoChangeCooldown=2+Math.random()*5;
  }

  // Boost decay
  if(ufoBoostT>0){
    ufoBoostT-=dt;
    if(ufoBoostT<=0) ufoTargetSpeed=0.15+Math.random()*0.3;
  }

  // Smoothly lerp toward targets at different rates (erratic feel)
  const lerpR=dt*0.8;
  const lerpH=dt*1.2;
  const lerpS=dt*1.5;
  const lerpT=dt*0.6;
  ufoCurrentR+=(ufoTargetR-ufoCurrentR)*lerpR;
  ufoCurrentH+=(ufoTargetH-ufoCurrentH)*lerpH;
  ufoCurrentSpeed+=(ufoTargetSpeed-ufoCurrentSpeed)*lerpS;
  ufoCurrentTiltX+=(ufoTargetTiltX-ufoCurrentTiltX)*lerpT;
  ufoCurrentTiltZ+=(ufoTargetTiltZ-ufoCurrentTiltZ)*lerpT;

  // Wobble
  ufoWobble=Math.sin(t*2.3)*0.08+Math.sin(t*3.7)*0.05;

  // Move
  ufoTheta+=dt*ufoCurrentSpeed;
  ufoGroup.position.x=Math.cos(ufoTheta)*ufoCurrentR;
  ufoGroup.position.z=Math.sin(ufoTheta)*ufoCurrentR;
  ufoGroup.position.y=ufoCurrentH+Math.sin(t*1.3)*4+Math.sin(t*2.1)*2;

  // Banking tilt — leans into turns
  ufoGroup.rotation.y+=dt*0.8;
  ufoGroup.rotation.x=ufoCurrentTiltX+ufoWobble;
  ufoGroup.rotation.z=ufoCurrentTiltZ-ufoCurrentSpeed*0.15;

  // Spin ring faster during boost
  ufoRing.rotation.z+=dt*(3+Math.abs(ufoCurrentSpeed)*2);

  // Light colors shift with speed
  const speedFrac=Math.min(Math.abs(ufoCurrentSpeed)/1.2,1);
  ufoRingMat.color.setRGB(0,1-speedFrac*0.5,1-speedFrac*0.3);

  // Blink lights
  ufoLights.forEach(({mesh,phase})=>{
    mesh.material.opacity=0.3+0.7*Math.abs(Math.sin(t*(3+speedFrac*4)+phase));
  });
  ufoPointLight.intensity=(0.5+0.5*Math.sin(t*4))*(1+speedFrac);
  ufoPointLight.color.setRGB(speedFrac, 1-speedFrac*0.5, 1);
}

// ── COUNTDOWN + TRIGGER ──────────────────────────────
const countdownOverlay=document.getElementById('countdown-overlay');
const countdownText=document.getElementById('countdown-text');
const countdownSub=document.getElementById('countdown-sub');

function triggerAthena(){
  if(PHASE_CUR!==PHASE.ORBIT) return;
  PHASE_CUR='countdown';
  setPhaseLabel('');
  startHeartbeat(); // begins immediately, accelerates through approach

  // Flash the warning message immediately on button press
  countdownOverlay.classList.add('visible');
  countdownText.style.transition='none';
  countdownText.style.opacity='0';
  countdownSub.textContent='';

  // Line 1: "AWESOME BARTENDER APPROACHING"
  setTimeout(()=>{
    countdownText.innerHTML='AWESOME BARTENDER<br>APPROACHING';
    countdownText.style.color='#ffcc00';
    countdownText.style.fontSize='clamp(28px,5vw,58px)';
    countdownText.style.textShadow='0 0 40px #ffcc00cc, 0 0 80px #ffaa0066';
    countdownText.style.transition='opacity 0.4s ease';
    countdownText.style.opacity='1';
  }, 100);

  // Line 2: "IMPACT IMMINENT" — swaps in after 2s
  setTimeout(()=>{
    countdownText.style.opacity='0';
    setTimeout(()=>{
      countdownText.innerHTML='⚠ IMPACT IMMINENT ⚠';
      countdownText.style.color='#ff2200';
      countdownText.style.textShadow='0 0 40px #ff2200cc, 0 0 80px #ff000066';
      countdownText.style.opacity='1';
    }, 400);
  }, 2200);

  // After warning, fade out and run countdown
  setTimeout(()=>{
    countdownText.style.opacity='0';
    setTimeout(()=>{
      countdownText.style.fontSize=''; // reset to CSS default
      runCountdown();
    }, 400);
  }, 4200);
}
window.triggerAthena=triggerAthena;

function runCountdown(){
  setPhaseLabel('');

  const steps=[
    {text:'NORMAL THURSDAY...', sub:'', color:'#ffffff', dur:1800, scale:'1'},
    {text:'5', sub:'PREPARE YOURSELVES', color:'#ffdd44', dur:900, scale:'1.1'},
    {text:'4', sub:'PREPARE YOURSELVES', color:'#ffaa22', dur:900, scale:'1.1'},
    {text:'3', sub:'SHE IS COMING', color:'#ff8800', dur:900, scale:'1.15'},
    {text:'2', sub:'SHE IS COMING', color:'#ff5500', dur:900, scale:'1.2'},
    {text:'1', sub:'BRACE FOR IMPACT', color:'#ff2200', dur:900, scale:'1.3'},
    {text:'ATHENA\nDETECTED', sub:'', color:'#cc44ff', dur:1400, scale:'1'},
  ];

  let si=0;
  function nextStep(){
    if(si>=steps.length){
      countdownOverlay.classList.remove('visible');
      setTimeout(launchAthena, 400);
      return;
    }
    const s=steps[si];
    countdownText.style.opacity='0';
    countdownText.style.transform='scale(0.85)';
    setTimeout(()=>{
      countdownText.innerHTML=s.text.replace(/\n/g,'<br>');
      countdownText.style.color=s.color;
      countdownText.style.textShadow=`0 0 40px ${s.color}cc, 0 0 80px ${s.color}66`;
      countdownText.style.opacity='1';
      countdownText.style.transform=`scale(${s.scale})`;
      countdownSub.textContent=s.sub;
      si++;
      setTimeout(nextStep,s.dur);
    },150);
  }
  nextStep();
}

function launchAthena(){
  // Show warning panel via direct JS — no CSS class
  const panel = document.getElementById('athena-info');
  panel.style.opacity = '0';
  panel.style.display = 'block';

  // Fade in over 0.6s
  let fadeInOp = 0;
  const fadeIn = setInterval(()=>{
    fadeInOp = Math.min(1, fadeInOp + 0.06);
    panel.style.opacity = fadeInOp;
    if(fadeInOp >= 1){
      clearInterval(fadeIn);
      setTimeout(()=>{ document.getElementById('threat-fill').style.width='100%'; }, 200);
      setPhaseLabel('QUAD CITIES OBSERVATORY — THREAT DETECTED');

      // Hold for 3 seconds then fade out
      setTimeout(()=>{
        let fadeOutOp = 1;
        const fadeOut = setInterval(()=>{
          fadeOutOp = Math.max(0, fadeOutOp - 0.04);
          panel.style.opacity = fadeOutOp;
          if(fadeOutOp <= 0){
            clearInterval(fadeOut);
            panel.style.display = 'none';
            // Reset threat bar for next time
            const tf = document.getElementById('threat-fill');
            tf.style.transition = 'none';
            tf.style.width = '0%';
            setTimeout(()=>{ tf.style.transition = 'width 2s ease'; }, 50);

            // NOW launch Athena
            PHASE_CUR = PHASE.APPROACH;
            phaseT = 0;
            approachLineIdx = 0;
            athenaMesh.position.copy(athenaStart);
            athenaMesh.visible = true;
            setPhaseLabel('ATHENA INCOMING — BRACE FOR IMPACT');
            showApproachLine();
          }
        }, 30);
      }, 3000);
    }
  }, 30);
}
function beginSimulation(){
  PHASE_CUR=PHASE.FORMING;
  formingT=0;
  mainUI.classList.add('visible');

  // Animate stars appearing
  let starDelay=0;
  starLayers.forEach((layer,i)=>{
    setTimeout(()=>{
      let op=0;
      const iv=setInterval(()=>{
        op=Math.min(1,op+0.04);
        layer.material.opacity=[0.9,0.8,0.7,0.8,0.9][i]*op;
        if(op>=1) clearInterval(iv);
      },40);
    },starDelay);
    starDelay+=400;
  });
  // Nebulae fade in
  nebulae.forEach((n,i)=>{
    setTimeout(()=>{
      let op=0;
      const iv=setInterval(()=>{
        op=Math.min(1,op+0.02);
        n.material.opacity=0.04*op;
        if(op>=1)clearInterval(iv);
      },60);
    },i*700);
  });

  // Sun ignites
  setTimeout(()=>{
    sun.visible=true;
    // Symphony 7 continuous
    let sunOp=0;
    const sv=setInterval(()=>{
      sunOp=Math.min(1,sunOp+0.025);
      sunLight.intensity=sunOp*4;
      if(sunOp>=1){clearInterval(sv); formPlanets();}
    },40);
  },2000);
}

function formPlanets(){
  // Planets materialize one by one
  let delay=300;
  planetDefs.forEach((def,i)=>{
    setTimeout(()=>{
      const p=planets[def.key];
      p.mesh.visible=true;
      p.ring.visible=true;
      let op=0;
      const iv=setInterval(()=>{
        op=Math.min(1,op+0.04);
        p.mesh.material.opacity=op;
        if(op>=1){
          clearInterval(iv);
          if(i===planetDefs.length-1){
            setTimeout(()=>{PHASE_CUR=PHASE.ORBIT; setPhaseLabel('Quad Cities Orbit of Fun');},600);
          }
        }
      },40);
    },delay);
    delay+=350;
  });
}

function triggerAthena(){
  if(PHASE_CUR!==PHASE.ORBIT) return;
  PHASE_CUR=PHASE.APPROACH;
  phaseT=0;
  approachLineIdx=0;
  athenaMesh.position.copy(athenaStart);
  athenaMesh.visible=true;
  athenaInfo.classList.add('visible');
  setPhaseLabel('WARNING — UNIDENTIFIED OBJECT ON COLLISION COURSE');
  showApproachLine();
}
window.triggerAthena=triggerAthena;

function showApproachLine(){
  if(approachLineIdx>=approachLines.length) return;
  approachStory.innerHTML=approachLines[approachLineIdx];
  approachStory.classList.add('visible');
  approachLineIdx++;
  setTimeout(()=>{
    approachStory.classList.remove('visible');
    setTimeout(showApproachLine,1200);
  },2800);
}

function resetScene(){
  // ── Reset all 3D state ──
  PHASE_CUR=PHASE.STORY;
  phaseT=0; shakeT=0;
  athenaMesh.visible=false;
  trail.visible=false;
  exParts.visible=false; exParts2.visible=false;
  explodeActive=false;
  exMat.opacity=0; exMat2.opacity=0;
  shockRing.visible=false; shockRing2.visible=false;
  shockMat.opacity=0; shock2Mat.opacity=0;
  whiteFlashDiv.style.opacity='0';
  document.getElementById('uber-flash').style.opacity='0';
  stopAllNodes(0.3);
  sym7Running = false;
  stopHeartbeat();
  setTimeout(()=>{ if(actx) actx.resume().then(()=>playSym7()); }, 600);
  impactOverlay.style.opacity=0;
  countdownOverlay.classList.remove('visible');
  const tf=document.getElementById('threat-fill');
  tf.style.transition='none'; tf.style.width='0%';
  takeover.classList.remove('visible');
  takeover.style.background='rgba(0,0,0,0.95)';
  athenaInfo.style.opacity='0';
  athenaInfo.style.display='none';
  approachStory.classList.remove('visible');
  setPhaseLabel('');
  cameraTarget.set(0,90,200);
  cameraLookTarget.set(0,0,0);
  camTheta=0; camPhi=0.38; camRadius=210;
  const p=planets['State18'];
  p.mesh.material.emissive=new THREE.Color(0x001a44);
  p.mesh.material.emissiveIntensity=0.12;
  p.mesh.scale.set(1,1,1);

  // ── Hide all planets and sun ──
  sun.visible=false;
  sunLight.intensity=0;
  planetDefs.forEach(def=>{
    planets[def.key].mesh.visible=false;
    planets[def.key].ring.visible=false;
  });
  starLayers.forEach(l=>{ l.material.opacity=0; });
  nebulae.forEach(n=>{ n.material.opacity=0; });

  // ── Restart story screen ──
  storyIndex=0;
  storyActive=true;
  storyEl.style.transition='none';
  storyEl.style.opacity='0';
  storyEl.innerHTML='';
  storyScreen.style.transition='none';
  storyScreen.style.opacity='1';
  storyScreen.style.display='flex';

  // ── Restart simulation & story ──
  setTimeout(()=>{
    beginSimulation();
    showNextLine();
  }, 300);
}
window.resetScene=resetScene;

// ── CAMERA ORBIT ─────────────────────────────────────
let isDragging=false,lastMouse={x:0,y:0};
let camTheta=0,camPhi=0.38,camRadius=210;
renderer.domElement.addEventListener('mousedown',e=>{isDragging=true;lastMouse={x:e.clientX,y:e.clientY};});
renderer.domElement.addEventListener('mousemove',e=>{
  if(!isDragging)return;
  camTheta-=(e.clientX-lastMouse.x)*0.005;
  camPhi-=(e.clientY-lastMouse.y)*0.005;
  camPhi=Math.max(0.01,Math.min(Math.PI/2,camPhi));
  lastMouse={x:e.clientX,y:e.clientY};
});
renderer.domElement.addEventListener('mouseup',()=>isDragging=false);
renderer.domElement.addEventListener('wheel',e=>{
  camRadius+=e.deltaY*0.3;
  camRadius=Math.max(40,Math.min(2000,camRadius));
});
renderer.domElement.addEventListener('touchstart',e=>{isDragging=true;lastMouse={x:e.touches[0].clientX,y:e.touches[0].clientY};});
renderer.domElement.addEventListener('touchmove',e=>{
  if(!isDragging)return;
  camTheta-=(e.touches[0].clientX-lastMouse.x)*0.005;
  camPhi-=(e.touches[0].clientY-lastMouse.y)*0.005;
  camPhi=Math.max(0.01,Math.min(Math.PI/2,camPhi));
  lastMouse={x:e.touches[0].clientX,y:e.touches[0].clientY};
});
renderer.domElement.addEventListener('touchend',()=>isDragging=false);

// ── ANIMATE ──────────────────────────────────────────
const clock=new THREE.Clock();

function animate(){
  requestAnimationFrame(animate);
  const dt=Math.min(clock.getDelta(),0.05);

  if(sun.visible) sun.rotation.y+=dt*0.18;

  if(PHASE_CUR!==PHASE.STORY&&PHASE_CUR!==PHASE.FORMING||PHASE_CUR===PHASE.ORBIT||PHASE_CUR===PHASE.APPROACH||PHASE_CUR===PHASE.IMPACT||PHASE_CUR===PHASE.TAKEOVER){
    planetDefs.forEach(def=>{
      const p=planets[def.key];
      if(!p.mesh.visible) return;
      // Realistic orbital speed scaled for simulation
      p.pivot.rotation.y+=dt*def.speed*0.3;
      // Realistic self-rotation: gas giants spin fast, rocky planets slow
      const spinRate = {
        Beer:0.017, Vodka:0.004, State18:0.45, Gin:0.41,
        Tequila:1.8, Margaritas:1.6, Whiskey:0.9, Rum:0.7
      }[def.key] || 0.4;
      p.mesh.rotation.y+=dt*spinRate;
    });
    if(planets['Moon']) planets['Moon'].pivot.rotation.y+=dt*0.13;
  }

  players_State18_worldPos();

  // APPROACH — slowed for tension
  if(PHASE_CUR===PHASE.APPROACH){
    phaseT+=dt*0.072; // slower: ~14s approach instead of ~6s
    const ease=1-Math.pow(1-Math.min(phaseT,1),3);
    athenaMesh.position.lerpVectors(athenaStart,earthWorldPos,ease);
    athenaMesh.rotation.y+=dt*1.8;
    trail.visible=true;
    trailHistory.unshift(athenaMesh.position.clone());
    if(trailHistory.length>TRAIL_COUNT) trailHistory.pop();
    const tp=trailGeo.attributes.position.array;
    trailHistory.forEach((p,i)=>{tp[i*3]=p.x;tp[i*3+1]=p.y;tp[i*3+2]=p.z;});
    trailGeo.attributes.position.needsUpdate=true;
    // Trail brightens as Athena closes in
    trailMat.opacity=0.3+0.5*Math.min(phaseT,1);
    trailMat.size=0.9+1.2*Math.min(phaseT,1);
    if(phaseT>0.45){
      const ce=(phaseT-0.45)/0.55;
      cameraTarget.lerpVectors(new THREE.Vector3(0,90,200),earthWorldPos.clone().add(new THREE.Vector3(22,16,22)),Math.min(ce,1));
      cameraLookTarget.lerp(earthWorldPos,Math.min(ce*1.5,1));
    }
    if(phaseT>=1){
      PHASE_CUR=PHASE.IMPACT; phaseT=0;
      stopHeartbeat();
      playImpactMusic();
      impactPos.copy(earthWorldPos);
      explodeActive=true; explodeT=0;

      // Prime both particle systems at impact point
      const ep=exGeo.attributes.position.array;
      const ep2=exGeo2.attributes.position.array;
      for(let i=0;i<EX;i++){ep[i*3]=impactPos.x;ep[i*3+1]=impactPos.y;ep[i*3+2]=impactPos.z;}
      for(let i=0;i<EX2;i++){ep2[i*3]=impactPos.x;ep2[i*3+1]=impactPos.y;ep2[i*3+2]=impactPos.z;}
      exGeo.attributes.position.needsUpdate=true;
      exGeo2.attributes.position.needsUpdate=true;
      exMat.opacity=1; exMat2.opacity=0.8;
      exParts.visible=true; exParts2.visible=true;

      // Shockwave rings at impact position
      shockRing.position.copy(impactPos);
      shockRing.lookAt(camera.position);
      shockRing.scale.set(0.1,0.1,0.1);
      shockMat.opacity=1; shockRing.visible=true;

      shockRing2.position.copy(impactPos);
      shockRing2.lookAt(camera.position);
      shockRing2.scale.set(0.1,0.1,0.1);
      shock2Mat.opacity=0; shockRing2.visible=true;

      athenaMesh.visible=false; trail.visible=false;
      impactFlashT=1;
      approachStory.classList.remove('visible');
      setPhaseLabel('IMPACT — ATHENA HAS STRUCK STATE 18');

      // White flash
      whiteFlashDiv.style.opacity='1';
      setTimeout(()=>{ whiteFlashDiv.style.transition='opacity 0.8s'; whiteFlashDiv.style.opacity='0'; }, 80);

      // Camera shake
      triggerShake(0.6, 3.5);

      // Audio — deep rumble via Web Audio API
      try {
        const actx=new (window.AudioContext||window.webkitAudioContext)();
        // Low boom
        const buf=actx.createBuffer(1,actx.sampleRate*1.5,actx.sampleRate);
        const dat=buf.getChannelData(0);
        for(let i=0;i<dat.length;i++) dat[i]=(Math.random()*2-1)*Math.pow(1-i/dat.length,1.5);
        const src=actx.createBufferSource();
        src.buffer=buf;
        const filt=actx.createBiquadFilter();
        filt.type='lowpass'; filt.frequency.value=80;
        const gain=actx.createGain();
        gain.gain.setValueAtTime(0.6,actx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001,actx.currentTime+1.5);
        src.connect(filt); filt.connect(gain); gain.connect(actx.destination);
        src.start();
        // High crack
        const buf2=actx.createBuffer(1,actx.sampleRate*0.3,actx.sampleRate);
        const dat2=buf2.getChannelData(0);
        for(let i=0;i<dat2.length;i++) dat2[i]=(Math.random()*2-1)*Math.pow(1-i/dat2.length,3);
        const src2=actx.createBufferSource(); src2.buffer=buf2;
        const filt2=actx.createBiquadFilter(); filt2.type='highpass'; filt2.frequency.value=800;
        const gain2=actx.createGain();
        gain2.gain.setValueAtTime(0.4,actx.currentTime);
        gain2.gain.exponentialRampToValueAtTime(0.001,actx.currentTime+0.3);
        src2.connect(filt2); filt2.connect(gain2); gain2.connect(actx.destination);
        src2.start();
      } catch(e){}
    }
  }

  // IMPACT
  if(PHASE_CUR===PHASE.IMPACT){
    phaseT+=dt; explodeT+=dt;

    // Camera shake
    if(shakeT<shakeDur){
      shakeT+=dt;
      const decay=1-shakeT/shakeDur;
      camera.position.x+=( Math.random()-0.5)*shakeMag*decay;
      camera.position.y+=( Math.random()-0.5)*shakeMag*decay;
      camera.position.z+=( Math.random()-0.5)*shakeMag*decay;
    }

    // Primary explosion particles
    if(explodeActive){
      const ep=exGeo.attributes.position.array;
      for(let i=0;i<EX;i++){ep[i*3]+=exVel[i].x*dt*16;ep[i*3+1]+=exVel[i].y*dt*16;ep[i*3+2]+=exVel[i].z*dt*16;}
      exGeo.attributes.position.needsUpdate=true;
      exMat.opacity=Math.max(0,1-explodeT/4);
      exMat.color.lerpColors(new THREE.Color(0xffffff),new THREE.Color(0xff4400),Math.min(explodeT*2,1));
    }
    // Secondary debris ring
    {
      const ep2=exGeo2.attributes.position.array;
      for(let i=0;i<EX2;i++){ep2[i*3]+=exVel2[i].x*dt*10;ep2[i*3+1]+=exVel2[i].y*dt*10;ep2[i*3+2]+=exVel2[i].z*dt*10;}
      exGeo2.attributes.position.needsUpdate=true;
      exMat2.opacity=Math.max(0,0.8-explodeT/3.5);
      exMat2.color.lerpColors(new THREE.Color(0xffaa44),new THREE.Color(0xaa44ff),Math.min(explodeT/3,1));
    }

    // Shockwave ring 1 — fast expanding white ring
    if(shockRing.visible){
      const sw=0.5+explodeT*28;
      shockRing.scale.set(sw,sw,sw);
      shockRing.lookAt(camera.position);
      shockMat.opacity=Math.max(0,0.9-explodeT*2.2);
      if(shockMat.opacity<=0) shockRing.visible=false;
    }
    // Shockwave ring 2 — slower purple ring, starts after short delay
    if(shockRing2.visible){
      const delay2=0.2;
      const t2=Math.max(0,explodeT-delay2);
      const sw2=0.5+t2*18;
      shockRing2.scale.set(sw2,sw2,sw2);
      shockRing2.lookAt(camera.position);
      shock2Mat.opacity=Math.max(0,0.7-t2*1.4);
      if(shock2Mat.opacity<=0) shockRing2.visible=false;
    }

    // Orange-red flash overlay (replaces old impactFlashT)
    if(impactFlashT>0){
      impactFlashT-=dt*1.2;
      impactOverlay.style.opacity=Math.max(0,impactFlashT*0.85);
      impactOverlay.style.background=`radial-gradient(ellipse at center,rgba(255,220,120,${Math.max(0,impactFlashT*0.9)}) 0%,rgba(255,60,0,${Math.max(0,impactFlashT*0.7)}) 50%,rgba(80,0,120,${Math.max(0,impactFlashT*0.4)}) 100%)`;
    } else { impactOverlay.style.opacity=0; }

    // State 18 absorbs Athena's purple
    const et2=Math.min(phaseT/4,1);
    const em=planets['State18'].mesh.material;
    em.emissive=new THREE.Color(0x330066);
    em.emissiveIntensity=et2*1.1;
    planets['State18'].mesh.scale.setScalar(1+Math.sin(phaseT*10)*0.04*Math.max(0,1-phaseT/2));

    if(phaseT>6){
      PHASE_CUR=PHASE.TAKEOVER; phaseT=0;
      takeover.classList.add('visible');
      setPhaseLabel('STATE 18 ASSIMILATED — ATHENA REIGNS');
      // Symphony 7 swells back after impact duck
      // Flash "Stay safe, use an Uber" after a short delay
      setTimeout(()=>{
        const uf=document.getElementById('uber-flash');

        // Step 1: fade takeover bg to transparent so 3D + UFO are visible
        takeover.style.transition='opacity 2.5s, background 1.5s ease';
        takeover.style.background='rgba(0,0,0,0)';

        // Step 2: show the sign
        uf.style.opacity='1';

        // Step 3: UFO zooms in close above the sign — give it 1s to arrive before sign shows
        ufoTargetR=18;
        ufoTargetH=20;
        ufoTargetSpeed=2.0;

        // Step 4: after UFO has hovered for 3s, zip it away
        setTimeout(()=>{
          ufoTargetR=400;
          ufoTargetH=120;
          ufoTargetSpeed=6.0;

          // Whoosh sound as it zips
          try{
            const ctx2=getACtx();
            const whoosh=ctx2.createOscillator();
            const wG=ctx2.createGain();
            whoosh.type='sawtooth';
            whoosh.frequency.setValueAtTime(900,ctx2.currentTime);
            whoosh.frequency.exponentialRampToValueAtTime(60,ctx2.currentTime+0.7);
            wG.gain.setValueAtTime(0.35,ctx2.currentTime);
            wG.gain.exponentialRampToValueAtTime(0.001,ctx2.currentTime+0.7);
            whoosh.connect(wG); wG.connect(ctx2.destination);
            whoosh.start(); whoosh.stop(ctx2.currentTime+0.7);
          }catch(e){}

          // Step 5: after UFO is gone (~1.5s), fade sign out AND fade background back to dark
          setTimeout(()=>{
            uf.style.opacity='0';
            takeover.style.transition='opacity 2.5s, background 2s ease';
            takeover.style.background='rgba(0,0,0,0.95)';
            // Settle UFO back to normal orbit
            setTimeout(()=>{ ufoTargetSpeed=0.25; ufoTargetR=150; ufoTargetH=28; }, 1000);
          }, 1500);

        }, 3000); // UFO hovers for 3s

        // Happy ascending sound when UFO arrives
        try{
          const ctx2=getACtx();
          const happyNotes=[523.3,659.3,783.9,1046.5,1318.5,1567.9,2093];
          const revH=makeReverb(2,1);
          const masterH=ctx2.createGain();
          masterH.gain.value=0.5;
          revH.connect(masterH); masterH.connect(ctx2.destination);
          happyNotes.forEach((freq,i)=>{
            const o=ctx2.createOscillator();
            const g=ctx2.createGain();
            o.type='sine'; o.frequency.value=freq;
            const t=ctx2.currentTime+i*0.13;
            g.gain.setValueAtTime(0,t);
            g.gain.linearRampToValueAtTime(0.22,t+0.05);
            g.gain.exponentialRampToValueAtTime(0.001,t+0.7);
            o.connect(g); g.connect(revH); o.start(t); o.stop(t+0.75);
            const o2=ctx2.createOscillator();
            const g2=ctx2.createGain();
            o2.type='triangle'; o2.frequency.value=freq*2.01;
            g2.gain.setValueAtTime(0,t);
            g2.gain.linearRampToValueAtTime(0.07,t+0.03);
            g2.gain.exponentialRampToValueAtTime(0.001,t+0.4);
            o2.connect(g2); g2.connect(revH); o2.start(t); o2.stop(t+0.45);
          });
        }catch(e){}

      }, 6000);
    }
  }

  // TAKEOVER
  if(PHASE_CUR===PHASE.TAKEOVER){
    phaseT+=dt;
    planets['State18'].mesh.scale.setScalar(1+Math.sin(phaseT*2)*0.06);
    cameraLookTarget.lerp(earthWorldPos,0.02);
  }

  // CAMERA
  if(!isDragging&&(PHASE_CUR===PHASE.ORBIT||PHASE_CUR===PHASE.FORMING)) camTheta+=dt*0.04;
  if((PHASE_CUR===PHASE.ORBIT||PHASE_CUR===PHASE.FORMING)||isDragging){
    const cx=Math.sin(camTheta)*Math.cos(camPhi)*camRadius;
    const cy=Math.sin(camPhi)*camRadius;
    const cz=Math.cos(camTheta)*Math.cos(camPhi)*camRadius;
    camera.position.lerp(new THREE.Vector3(cx,cy,cz),0.05);
    camera.lookAt(new THREE.Vector3(0,0,0));
  } else {
    camera.position.lerp(cameraTarget,0.035);
    camera.lookAt(cameraLookTarget);
  }

  updateLabels();
  updateTwinkle(clock.elapsedTime);
  updateUFO(dt, clock.elapsedTime);
  renderer.render(scene,camera);
}

function players_State18_worldPos(){
  if(planets['State18']&&planets['State18'].mesh.visible)
    planets['State18'].mesh.getWorldPosition(earthWorldPos);
}

// ═══════════════════════════════════════════════════════
//  MUSIC ENGINE — Web Audio API synthesized soundtrack
// ═══════════════════════════════════════════════════════
let actx = null;
let musicNodes = {}; // track active nodes so we can stop them

function getACtx(){
  if(!actx) actx = new (window.AudioContext||window.webkitAudioContext)();
  if(actx.state==='suspended') actx.resume();
  return actx;
}

function stopAllMusic(){
  Object.values(musicNodes).forEach(n=>{
    try{ n.stop(0); }catch(e){}
  });
  musicNodes={};
}

// ── helpers ──────────────────────────────────────────
function makeOsc(freq, type, gainVal, fadeIn=0.5, dest){
  const ctx=getACtx();
  const osc=ctx.createOscillator();
  const g=ctx.createGain();
  osc.type=type; osc.frequency.value=freq;
  g.gain.setValueAtTime(0,ctx.currentTime);
  g.gain.linearRampToValueAtTime(gainVal,ctx.currentTime+fadeIn);
  osc.connect(g); g.connect(dest||ctx.destination);
  osc.start();
  return {osc,gain:g};
}

function makePad(freq, gainVal, fadeIn=2){
  const ctx=getACtx();
  const osc1=ctx.createOscillator();
  const osc2=ctx.createOscillator();
  const osc3=ctx.createOscillator();
  const g=ctx.createGain();
  const filt=ctx.createBiquadFilter();
  filt.type='lowpass'; filt.frequency.value=800; filt.Q.value=1;
  osc1.type='sine';     osc1.frequency.value=freq;
  osc2.type='triangle'; osc2.frequency.value=freq*1.002; // slight detune
  osc3.type='sine';     osc3.frequency.value=freq*2.001;
  g.gain.setValueAtTime(0,ctx.currentTime);
  g.gain.linearRampToValueAtTime(gainVal,ctx.currentTime+fadeIn);
  [osc1,osc2,osc3].forEach(o=>{ o.connect(filt); o.start(); });
  filt.connect(g); g.connect(ctx.destination);
  return {oscs:[osc1,osc2,osc3],gain:g,filt};
}

function stopNode(key){
  const n=musicNodes[key];
  if(!n) return;
  const ctx=getACtx();
  const fade=0.8;
  if(n.gain){
    n.gain.gain.cancelScheduledValues(ctx.currentTime);
    n.gain.gain.linearRampToValueAtTime(0,ctx.currentTime+fade);
  }
  const stop=()=>{
    try{ if(n.osc) n.osc.stop(0); }catch(e){}
    try{ if(n.oscs) n.oscs.forEach(o=>o.stop(0)); }catch(e){}
  };
  setTimeout(stop, fade*1000+50);
  delete musicNodes[key];
}

function stopAllNodes(fadeTime=1.5){
  const ctx=getACtx();
  Object.keys(musicNodes).forEach(k=>{
    const n=musicNodes[k];
    if(n && n.gain){
      n.gain.gain.cancelScheduledValues(ctx.currentTime);
      n.gain.gain.linearRampToValueAtTime(0,ctx.currentTime+fadeTime);
    }
    setTimeout(()=>{
      try{ if(n.osc) n.osc.stop(0); }catch(e){}
      try{ if(n.oscs) n.oscs.forEach(o=>o.stop(0)); }catch(e){}
    }, fadeTime*1000+50);
  });
  musicNodes={};
}

// ── REVERB ────────────────────────────────────────────
function makeReverb(duration=3, decay=2){
  const ctx=getACtx();
  const convolver=ctx.createConvolver();
  const len=ctx.sampleRate*duration;
  const buf=ctx.createBuffer(2,len,ctx.sampleRate);
  for(let c=0;c<2;c++){
    const d=buf.getChannelData(c);
    for(let i=0;i<len;i++) d[i]=(Math.random()*2-1)*Math.pow(1-i/len,decay);
  }
  convolver.buffer=buf;
  return convolver;
}

// ── HEARTBEAT ENGINE ─────────────────────────────────
// Starts slow at 60bpm, accelerates to ~160bpm, stops at impact
let heartbeatActive = false;
let heartbeatBPM = 60;
let heartbeatTimer = null;

function startHeartbeat(){
  heartbeatActive = true;
  heartbeatBPM = 60;
  scheduleHeartbeat();
}

function stopHeartbeat(){
  heartbeatActive = false;
  if(heartbeatTimer){ clearTimeout(heartbeatTimer); heartbeatTimer=null; }
}

function scheduleHeartbeat(){
  if(!heartbeatActive) return;
  const ctx = getACtx();

  // Lub — deep thud
  const lub = ctx.createOscillator();
  const lubG = ctx.createGain();
  lub.type = 'sine';
  lub.frequency.setValueAtTime(58, ctx.currentTime);
  lub.frequency.exponentialRampToValueAtTime(32, ctx.currentTime+0.18);
  lubG.gain.setValueAtTime(0.55, ctx.currentTime);
  lubG.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime+0.22);
  lub.connect(lubG); lubG.connect(ctx.destination);
  lub.start(); lub.stop(ctx.currentTime+0.23);

  // Dub — slightly higher, slightly after
  const dubDelay = 0.14;
  const dub = ctx.createOscillator();
  const dubG = ctx.createGain();
  dub.type = 'sine';
  dub.frequency.setValueAtTime(48, ctx.currentTime+dubDelay);
  dub.frequency.exponentialRampToValueAtTime(28, ctx.currentTime+dubDelay+0.15);
  dubG.gain.setValueAtTime(0, ctx.currentTime);
  dubG.gain.setValueAtTime(0.38, ctx.currentTime+dubDelay);
  dubG.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime+dubDelay+0.18);
  dub.connect(dubG); dubG.connect(ctx.destination);
  dub.start(ctx.currentTime+dubDelay);
  dub.stop(ctx.currentTime+dubDelay+0.19);

  // Accelerate gradually — faster as Athena closes in
  heartbeatBPM = Math.min(heartbeatBPM + 1.8, 168);
  const interval = (60 / heartbeatBPM) * 1000;
  heartbeatTimer = setTimeout(scheduleHeartbeat, interval);
}

// ═══════════════════════════════════════════════════════
//  BEETHOVEN SYMPHONY NO. 7 — ALLEGRETTO (2nd movement)
//  One continuous loop from first interaction to the end
// ═══════════════════════════════════════════════════════

// The famous Allegretto theme: A minor, ♩ = 76bpm
// Rhythm pattern: long-short-short-long-long (♩ ♪♪ ♩ ♩)
// [frequency, duration_in_beats]
const A3=220,  C4=261.6, D4=293.7, E4=329.6,
      F4=349.2, G4=392.0, A4=440.0, B4=493.9,
      C5=523.3, D5=587.3, E5=659.3, F5=698.5,
      G5=784.0, A5=880.0;

const SYM7_BEAT = 0.79; // seconds per beat at ♩=76

// Full Allegretto theme — main melody
const sym7Melody = [
  // Theme A — bar 1-4
  [A4,2],[A4,1],[A4,1],[A4,2],[A4,2],
  [G4,2],[F4,1],[E4,1],[F4,2],[F4,2],
  [E4,2],[D4,1],[C4,1],[D4,2],[D4,2],
  [A3,2],[A3,1],[A3,1],[A3,4],
  // Theme A repeat variation
  [A4,2],[A4,1],[A4,1],[A4,2],[A4,2],
  [G4,2],[F4,1],[E4,1],[F4,2],[F4,2],
  [E4,2],[D4,1],[C4,1],[D4,2],[D4,2],
  [A3,4],[A3,4],
  // Theme B — brighter/rising
  [C5,2],[C5,1],[C5,1],[C5,2],[C5,2],
  [B4,2],[A4,1],[G4,1],[A4,2],[A4,2],
  [G4,2],[F4,1],[E4,1],[F4,2],[F4,2],
  [E4,4],[E4,4],
  // Theme C — climax phrase
  [A4,1],[B4,1],[C5,1],[D5,1],[E5,2],[E5,2],
  [D5,1],[C5,1],[B4,1],[A4,1],[B4,2],[B4,2],
  [A4,1],[G4,1],[F4,1],[E4,1],[F4,2],[F4,2],
  [E4,4],[E4,4],
  // Return to Theme A
  [A4,2],[A4,1],[A4,1],[A4,2],[A4,2],
  [G4,2],[F4,1],[E4,1],[F4,2],[F4,2],
  [E4,2],[D4,1],[C4,1],[D4,2],[D4,2],
  [A3,4],[A3,4],
];

// Bass/cello counter-melody
const sym7Bass = [
  [A3,4],[A3,4],
  [F3=174.6,4],[F3,4],
  [E3=164.8,4],[E3,4],
  [A3,4],[A3,4],
  [A3,4],[A3,4],
  [F3,4],[F3,4],
  [E3,4],[E3,4],
  [A3,4],[A3,4],
  [C4,4],[C4,4],
  [A3,4],[A3,4],
  [F3,4],[F3,4],
  [E3,4],[E3,4],
  [A3,2],[E4,2],[A3,2],[E4,2],
  [G3=196,2],[E4,2],[G3,2],[E4,2],
  [F3,2],[D4,2],[F3,2],[D4,2],
  [E3,4],[E3,4],
  [A3,4],[A3,4],
  [F3,4],[F3,4],
  [E3,4],[E3,4],
  [A3,4],[A3,4],
];

// Inner voice (viola) — rhythmic pulse
const sym7Viola = [
  [E4,2],[C4,2],[E4,2],[C4,2],
  [F4,2],[C4,2],[F4,2],[C4,2],
  [E4,2],[C4,2],[E4,2],[C4,2],
  [A3,4],[A3,4],
  [E4,2],[C4,2],[E4,2],[C4,2],
  [F4,2],[C4,2],[F4,2],[C4,2],
  [E4,2],[C4,2],[E4,2],[C4,2],
  [A3,4],[A3,4],
  [G4,2],[E4,2],[G4,2],[E4,2],
  [A4,2],[E4,2],[A4,2],[E4,2],
  [F4,2],[D4,2],[F4,2],[D4,2],
  [E4,4],[E4,4],
  [C5,2],[A4,2],[C5,2],[A4,2],
  [B4,2],[G4,2],[B4,2],[G4,2],
  [A4,2],[F4,2],[A4,2],[F4,2],
  [E4,4],[E4,4],
  [E4,2],[C4,2],[E4,2],[C4,2],
  [F4,2],[C4,2],[F4,2],[C4,2],
  [E4,2],[C4,2],[E4,2],[C4,2],
  [A3,4],[A3,4],
];

let sym7MasterGain = null;
let sym7Running = false;

function playSym7(){
  if(sym7Running) return;
  sym7Running = true;
  const ctx = getACtx();
  const rev = makeReverb(5, 1.8);
  const master = ctx.createGain();
  master.gain.setValueAtTime(0, ctx.currentTime);
  master.gain.linearRampToValueAtTime(0.5, ctx.currentTime+4);
  rev.connect(master);
  master.connect(ctx.destination);
  sym7MasterGain = master;
  musicNodes.sym7Master = {gain: master};

  function scheduleVoice(notes, waveType, gainAmt, startT, vibratoHz=0, vibratoAmt=0){
    let t = startT;
    notes.forEach(([freq, beats])=>{
      const dur = beats * SYM7_BEAT;
      const o = ctx.createOscillator();
      const g = ctx.createGain();
      o.type = waveType;
      o.frequency.value = freq;
      if(vibratoHz > 0){
        const lfo = ctx.createOscillator();
        const lfoG = ctx.createGain();
        lfo.frequency.value = vibratoHz;
        lfoG.gain.value = vibratoAmt;
        lfo.connect(lfoG); lfoG.connect(o.frequency);
        lfo.start(t); lfo.stop(t+dur+0.05);
      }
      g.gain.setValueAtTime(0, t);
      g.gain.linearRampToValueAtTime(gainAmt, t+0.04);
      g.gain.linearRampToValueAtTime(gainAmt*0.85, t+dur-0.08);
      g.gain.linearRampToValueAtTime(0, t+dur);
      o.connect(g); g.connect(rev);
      o.start(t); o.stop(t+dur+0.05);
      t += dur;
    });
    return t; // return end time
  }

  function scheduleLoop(startT){
    const totalBeats = sym7Melody.reduce((s,[,b])=>s+b,0);
    const totalDur = totalBeats * SYM7_BEAT;
    scheduleVoice(sym7Melody, 'sine',     0.28, startT, 5.5, 6);   // melody — violin
    scheduleVoice(sym7Melody, 'triangle', 0.10, startT, 5.0, 4);   // melody 2nd — viola
    scheduleVoice(sym7Bass,   'triangle', 0.18, startT);            // bass — cello
    scheduleVoice(sym7Viola,  'sine',     0.09, startT);            // inner pulse
    // Re-schedule 2s before end so there is no gap
    setTimeout(()=>{
      if(sym7Running) scheduleLoop(startT + totalDur);
    }, (totalDur - 2) * 1000);
  }

  scheduleLoop(ctx.currentTime + 0.3);
}

function sym7SetVolume(vol, fadeTime=2){
  if(!sym7MasterGain) return;
  const ctx = getACtx();
  sym7MasterGain.gain.cancelScheduledValues(ctx.currentTime);
  sym7MasterGain.gain.linearRampToValueAtTime(vol, ctx.currentTime + fadeTime);
}

// Stub out old per-phase music so existing hook calls are safe
function playStoryMusic(){}
function playFormingMusic(){}
function playOrbitMusic(){}
function playApproachMusic(){}
function playTakeoverMusic(){}

// Impact — brief volume duck then swell back
function playImpactMusic(){
  const ctx = getACtx();
  // Duck music during boom
  sym7SetVolume(0.05, 0.1);
  setTimeout(()=> sym7SetVolume(0.55, 3), 2500);

  // Boom and noise
  const sub=ctx.createOscillator(); const subG=ctx.createGain();
  sub.type='sine';
  sub.frequency.setValueAtTime(80,ctx.currentTime);
  sub.frequency.exponentialRampToValueAtTime(20,ctx.currentTime+2);
  subG.gain.setValueAtTime(1.2,ctx.currentTime);
  subG.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+3);
  sub.connect(subG); subG.connect(ctx.destination); sub.start(); sub.stop(ctx.currentTime+3);

  const crunch=ctx.createOscillator(); const crunchG=ctx.createGain();
  crunch.type='sawtooth'; crunch.frequency.value=120;
  crunchG.gain.setValueAtTime(0.5,ctx.currentTime);
  crunchG.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+1.5);
  crunch.connect(crunchG); crunchG.connect(ctx.destination); crunch.start(); crunch.stop(ctx.currentTime+1.5);

  const bufSize=ctx.sampleRate*2;
  const noiseBuf=ctx.createBuffer(1,bufSize,ctx.sampleRate);
  const nd=noiseBuf.getChannelData(0);
  for(let i=0;i<bufSize;i++) nd[i]=Math.random()*2-1;
  const noise=ctx.createBufferSource(); noise.buffer=noiseBuf;
  const noiseG=ctx.createGain(); const noiseFilt=ctx.createBiquadFilter();
  noiseFilt.type='bandpass'; noiseFilt.frequency.value=400; noiseFilt.Q.value=0.5;
  noiseG.gain.setValueAtTime(0.7,ctx.currentTime);
  noiseG.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+2);
  noise.connect(noiseFilt); noiseFilt.connect(noiseG); noiseG.connect(ctx.destination);
  noise.start(); noise.stop(ctx.currentTime+2);
}

// ── Wire music to phase transitions ──────────────────
let musicStarted = false;

function tapToBegin(){
  const tapScreen = document.getElementById('tap-screen');
  if(!tapScreen) return;

  // Unlock AudioContext — must happen inside a user gesture
  try {
    if(!actx) actx = new (window.AudioContext||window.webkitAudioContext)();
    actx.resume().then(()=>{
      if(!musicStarted){
        musicStarted = true;
        playSym7();
      }
    });
  } catch(e){ console.warn('Audio init failed:', e); }

  // Fade out tap screen, show story
  tapScreen.style.transition = 'opacity 0.8s ease';
  tapScreen.style.opacity = '0';
  setTimeout(()=>{
    tapScreen.style.display = 'none';
    // Start story + simulation
    beginSimulationWithMusic();
    setTimeout(showNextLine, 400);
  }, 850);
}
window.tapToBegin = tapToBegin;

function startMusicOnInteraction(){
  if(musicStarted || !actx) return;
  musicStarted = true;
  actx.resume().then(()=> playSym7());
}

const _origSkipStory = skipStory;
window.skipStory = function(){
  startMusicOnInteraction();
  _origSkipStory();
};

const _origBeginSim=beginSimulation;
function beginSimulationWithMusic(){
  _origBeginSim();
}

animate();
window.addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight);
});
// Startup is triggered by tapToBegin() — no auto-start needed
</script>
</body>
</html>
