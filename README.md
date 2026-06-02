[room (1).html](https://github.com/user-attachments/files/28489660/room.1.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>陳言星</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #0a0908; overflow: hidden; }
canvas { display: block; width: 100vw; height: 100vh; touch-action: none; }
#hint {
  position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
  font-size: 10px; letter-spacing: 0.16em; color: rgba(245,240,232,0.3);
  text-transform: uppercase; pointer-events: none;
  animation: fade 4s ease forwards 1s;
}
@keyframes fade { 0%{opacity:1} 70%{opacity:1} 100%{opacity:0} }
</style>
</head>
<body>
<canvas id="c"></canvas>
<div id="hint">拖曳旋轉探索空間</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const canvas = document.getElementById('c');
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, powerPreference: 'high-performance' });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0a0908);
scene.fog = new THREE.FogExp2(0x0a0908, 0.018);

const camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 50);
camera.position.set(0, 0, 0.1);

// ── FLOOR: concrete tile texture via canvas ──
const tileCanvas = document.createElement('canvas');
tileCanvas.width = 512; tileCanvas.height = 512;
const tc = tileCanvas.getContext('2d');
// base concrete grey
tc.fillStyle = '#5a5a58';
tc.fillRect(0, 0, 512, 512);
// noise for concrete texture
for (let i = 0; i < 8000; i++) {
  const x = Math.random() * 512, y = Math.random() * 512;
  const v = Math.floor(Math.random() * 30) - 15;
  const c = 90 + v;
  tc.fillStyle = `rgba(${c},${c},${c-2},0.18)`;
  tc.fillRect(x, y, Math.random()*3+1, Math.random()*3+1);
}
// large tile grout lines (2x2 grid = big tiles)
tc.strokeStyle = 'rgba(60,60,58,0.25)';
tc.lineWidth = 2;
tc.beginPath(); tc.moveTo(256,0); tc.lineTo(256,512); tc.stroke();
tc.beginPath(); tc.moveTo(0,256); tc.lineTo(512,256); tc.stroke();
// subtle tile edge highlight
tc.strokeStyle = 'rgba(120,120,118,0.2)';
tc.lineWidth = 1;
tc.strokeRect(4, 4, 248, 248);
tc.strokeRect(260, 4, 248, 248);
tc.strokeRect(4, 260, 248, 248);
tc.strokeRect(260, 260, 248, 248);

const tileTex = new THREE.CanvasTexture(tileCanvas);
tileTex.wrapS = tileTex.wrapT = THREE.RepeatWrapping;
tileTex.repeat.set(4, 5); // big tiles = fewer repeats

const floorMat = new THREE.MeshStandardMaterial({
  map: tileTex, roughness: 0.85, metalness: 0.05
});
const floor = new THREE.Mesh(new THREE.BoxGeometry(10, 0.1, 12), floorMat);
floor.position.set(0, -3, 0);
floor.receiveShadow = true;
scene.add(floor);

// ── WALLS & CEILING ──
function wall(w, h, d, color, x, y, z) {
  const m = new THREE.Mesh(
    new THREE.BoxGeometry(w, h, d),
    new THREE.MeshStandardMaterial({ color, roughness: 0.92, metalness: 0.02 })
  );
  m.position.set(x, y, z);
  m.receiveShadow = true;
  scene.add(m);
}
wall(10, 0.1, 12, 0x1a1815, 0, 3, 0);       // ceiling
wall(10, 6, 0.15, 0x1e1c1a, 0, 0, -6);      // back
wall(0.15, 6, 12, 0x1c1a18, -5, 0, 0);      // left
wall(0.15, 6, 12, 0x1a1816, 5, 0, 0);       // right

// ── WINDOW: white metal frame, 2 panes only ──
const metalMat = new THREE.MeshStandardMaterial({
  color: 0xe8e8e8, roughness: 0.2, metalness: 0.85
});
const winW = 2.4, winH = 2.8, winZ = -6 + 0.12;

// outer frame only (4 bars) — no inner dividers
const mkBar = (w, h, d, x, y) => {
  const b = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), metalMat);
  b.position.set(x, y, winZ);
  scene.add(b);
};
mkBar(winW + 0.14, 0.08, 0.1, 2.0, winH/2 + 0.04);   // top
mkBar(winW + 0.14, 0.08, 0.1, 2.0, -winH/2 - 0.04);  // bottom
mkBar(0.08, winH + 0.08, 0.1, 2.0 - winW/2 - 0.04, 0);  // left
mkBar(0.08, winH + 0.08, 0.1, 2.0 + winW/2 + 0.04, 0);  // right
// single center vertical divider only
mkBar(0.05, winH, 0.08, 2.0, 0);

// glass: 2 panes
const glassMat = new THREE.MeshStandardMaterial({
  color: 0xc8dde8, roughness: 0.0, metalness: 0.1,
  transparent: true, opacity: 0.12, side: THREE.DoubleSide
});
const mkPane = (w, h, x) => {
  const g = new THREE.Mesh(new THREE.PlaneGeometry(w, h), glassMat);
  g.position.set(x, 0, winZ + 0.01);
  scene.add(g);
};
mkPane(winW/2 - 0.04, winH - 0.06, 2.0 - winW/4);  // left pane
mkPane(winW/2 - 0.04, winH - 0.06, 2.0 + winW/4);  // right pane

// window light
const winLight = new THREE.PointLight(0xc8dde8, 0.6, 6);
winLight.position.set(2.0, 0.5, -5.5);
scene.add(winLight);

// ── BED (headboard in corner: right wall + back wall, bed extends left) ──
const whiteMat = new THREE.MeshStandardMaterial({ color: 0xf0eeea, roughness: 0.5, metalness: 0.15 });
const mattressMat = new THREE.MeshStandardMaterial({ color: 0x1a1a1a, roughness: 0.92 });

function makeDotTex() {
  const dc = document.createElement('canvas');
  dc.width = 256; dc.height = 256;
  const ctx = dc.getContext('2d');
  ctx.fillStyle = '#e8e2d8';
  ctx.fillRect(0, 0, 256, 256);
  ctx.fillStyle = '#a8a09a';
  for (let row = 0; row < 16; row++) {
    for (let col = 0; col < 16; col++) {
      ctx.beginPath();
      ctx.arc(col * 16 + 8, row * 16 + 8, 1.8, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  const t = new THREE.CanvasTexture(dc);
  t.wrapS = t.wrapT = THREE.RepeatWrapping;
  t.repeat.set(3, 4);
  return t;
}
const dottedMat = new THREE.MeshStandardMaterial({ map: makeDotTex(), roughness: 0.95 });

// bed: headboard in right+back corner, extends left along back wall
// platform (runs along X axis)
const bedPlatform = new THREE.Mesh(new THREE.BoxGeometry(3.8, 0.16, 2.2), whiteMat);
bedPlatform.position.set(2.9, -2.92, -4.9);
scene.add(bedPlatform);

// 4 legs
const legG2 = new THREE.BoxGeometry(0.1, 0.28, 0.1);
[[1.1, -5.85], [4.7, -5.85], [1.1, -3.95], [4.7, -3.95]].forEach(([x, z]) => {
  const l = new THREE.Mesh(legG2, whiteMat);
  l.position.set(x, -3.14, z);
  scene.add(l);
});

// mattress
const mattress = new THREE.Mesh(new THREE.BoxGeometry(3.6, 0.24, 2.1), mattressMat);
mattress.position.set(2.9, -2.68, -4.9);
scene.add(mattress);

// headboard against right wall
const headboard = new THREE.Mesh(new THREE.BoxGeometry(0.1, 1.0, 2.2), whiteMat);
headboard.position.set(4.88, -2.42, -4.9);
scene.add(headboard);

// duvet
const duvet = new THREE.Mesh(new THREE.BoxGeometry(2.6, 0.15, 2.05), dottedMat);
duvet.position.set(2.3, -2.5, -4.9);
scene.add(duvet);

// one pillow near headboard (right side)
const pillow = new THREE.Mesh(new THREE.BoxGeometry(0.55, 0.14, 1.1), dottedMat);
pillow.position.set(4.45, -2.49, -4.9);
scene.add(pillow);


// ── BOOKSHELVES (open shelf units, 2x2 grid, front-right corner) ──
const swMat = new THREE.MeshStandardMaterial({ color: 0x3d2010, roughness: 0.85, metalness: 0.02 });
const slMat = new THREE.MeshStandardMaterial({ color: 0x5c3418, roughness: 0.82, metalness: 0.02 });

function makeOpenShelf(cx, cz) {
  const w = 2.0, h = 2.2, d = 0.48, t = 0.055;
  const by = -3;

  // left panel
  const lp = new THREE.Mesh(new THREE.BoxGeometry(t, h, d), swMat);
  lp.position.set(cx - w/2 + t/2, by + h/2, cz);
  scene.add(lp);

  // right panel
  const rp = new THREE.Mesh(new THREE.BoxGeometry(t, h, d), swMat);
  rp.position.set(cx + w/2 - t/2, by + h/2, cz);
  scene.add(rp);

  // back panel
  const bp = new THREE.Mesh(new THREE.BoxGeometry(w, h, t), swMat);
  bp.position.set(cx, by + h/2, cz + d/2 - t/2);
  scene.add(bp);

  // bottom board
  const btm = new THREE.Mesh(new THREE.BoxGeometry(w - t*2, t, d), slMat);
  btm.position.set(cx, by + t/2, cz);
  scene.add(btm);

  // top board
  const top = new THREE.Mesh(new THREE.BoxGeometry(w + 0.02, t, d + 0.02), slMat);
  top.position.set(cx, by + h - t/2, cz);
  scene.add(top);

  // 2 horizontal shelves → 3 rows
  [h/3, h*2/3].forEach(yOff => {
    const shelf = new THREE.Mesh(new THREE.BoxGeometry(w - t*2, t, d - 0.02), slMat);
    shelf.position.set(cx, by + yOff, cz);
    scene.add(shelf);
  });
}

makeOpenShelf(3.95, 5.76);
makeOpenShelf(1.9, 5.76);



// ── DOLLHOUSE (purple, 2 floors, pitched roof, white trim, in front of bed) ──
const dhWall = new THREE.MeshStandardMaterial({ color: 0xb48fd4, roughness: 0.85 });
const dhWhite = new THREE.MeshStandardMaterial({ color: 0xf0eeea, roughness: 0.6, metalness: 0.1 });
const dhDark = new THREE.MeshStandardMaterial({ color: 0x1a1020, roughness: 0.9 });
const dhRoof = new THREE.MeshStandardMaterial({ color: 0xe8e4de, roughness: 0.75 });

const dhX = 0, dhY = -3, dhZ = -2.0;
const dhW = 1.1*0.6, dhH = 1.4*0.6, dhD = 0.6*0.6;

// === BODY ===
// floor 1
const f1 = new THREE.Mesh(new THREE.BoxGeometry(dhW, dhH/2, dhD), dhWall);
f1.position.set(dhX, dhY + dhH/4, dhZ);
scene.add(f1);

// floor 2
const f2 = new THREE.Mesh(new THREE.BoxGeometry(dhW, dhH/2, dhD), dhWall);
f2.position.set(dhX, dhY + dhH*3/4, dhZ);
scene.add(f2);

// white corner trims (4 vertical strips)
[-dhW/2, dhW/2].forEach(x => {
  const trim = new THREE.Mesh(new THREE.BoxGeometry(0.045, dhH, 0.065), dhWhite);
  trim.position.set(dhX + x, dhY + dhH/2, dhZ - dhD/2 + 0.03);
  scene.add(trim);
});

// floor divider strip
const divStrip = new THREE.Mesh(new THREE.BoxGeometry(dhW + 0.01, 0.05, dhD + 0.01), dhWhite);
divStrip.position.set(dhX, dhY + dhH/2, dhZ);
scene.add(divStrip);

// base strip
const baseStrip = new THREE.Mesh(new THREE.BoxGeometry(dhW + 0.02, 0.05, dhD + 0.02), dhWhite);
baseStrip.position.set(dhX, dhY + 0.025, dhZ);
scene.add(baseStrip);

// === ROOF (pitched) ===
// two sloped panels using rotated boxes
const roofH = 0.42*0.6, roofW = dhW/2 + 0.18;
[-1, 1].forEach(side => {
  const panel = new THREE.Mesh(new THREE.BoxGeometry(roofW, 0.05, dhD + 0.12), dhRoof);
  panel.position.set(dhX + side * roofW/2 * 0.5, dhY + dhH + roofH/2 - 0.05, dhZ);
  panel.rotation.z = side * -0.42;
  scene.add(panel);
});
// roof ridge cap
const ridge = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.07, dhD + 0.14), dhWhite);
ridge.position.set(dhX, dhY + dhH + roofH - 0.04, dhZ);
scene.add(ridge);
// gable end pieces removed

// === WINDOWS (white frames, dark glass) ===
function dhWindow(wx, wy, wz, sw, sh) {
  // dark glass
  const glass = new THREE.Mesh(new THREE.BoxGeometry(sw, sh, 0.03), dhDark);
  glass.position.set(wx, wy, wz);
  scene.add(glass);
  // white outer frame
  const frame = new THREE.Mesh(new THREE.BoxGeometry(sw + 0.06, sh + 0.06, 0.025), dhWhite);
  frame.position.set(wx, wy, wz + 0.008);
  scene.add(frame);
  // re-add glass on top
  const glass2 = new THREE.Mesh(new THREE.BoxGeometry(sw, sh, 0.03), dhDark);
  glass2.position.set(wx, wy, wz + 0.018);
  scene.add(glass2);
  // cross dividers
  const hbar = new THREE.Mesh(new THREE.BoxGeometry(sw, 0.02, 0.035), dhWhite);
  hbar.position.set(wx, wy, wz + 0.022);
  scene.add(hbar);
  const vbar = new THREE.Mesh(new THREE.BoxGeometry(0.02, sh, 0.035), dhWhite);
  vbar.position.set(wx, wy, wz + 0.022);
  scene.add(vbar);
}

// house is 0.6 scale: dhW=0.66, dhH=0.84, dhD=0.36
// front face z = dhZ - 0.18 - 0.01 = -2.19
const fz = dhZ - 0.19; // front face of dollhouse

// floor 2 (y from -3+0.63 to -3+0.84): centre y = -2.58
// 3 windows across
dhWindow(dhX - 0.18, -2.58, fz, 0.11, 0.11);
dhWindow(dhX,        -2.58, fz, 0.11, 0.11);
dhWindow(dhX + 0.18, -2.58, fz, 0.11, 0.11);

// floor 1 (y from -3 to -3+0.42): centre y = -2.79
// 2 side windows
dhWindow(dhX - 0.22, -2.79, fz, 0.11, 0.13);
dhWindow(dhX + 0.22, -2.79, fz, 0.11, 0.13);

// centre door floor 1
const dW = 0.1, dH = 0.2;
const doorGlass = new THREE.Mesh(new THREE.BoxGeometry(dW, dH, 0.035), dhDark);
doorGlass.position.set(dhX, -3 + dH/2 + 0.02, fz);
scene.add(doorGlass);
const doorFr = new THREE.Mesh(new THREE.BoxGeometry(dW+0.05, dH+0.04, 0.025), dhWhite);
doorFr.position.set(dhX, -3 + dH/2 + 0.02, fz + 0.005);
scene.add(doorFr);
const doorFill = new THREE.Mesh(new THREE.BoxGeometry(dW, dH, 0.035), dhDark);
doorFill.position.set(dhX, -3 + dH/2 + 0.02, fz + 0.018);
scene.add(doorFill);
// arch over door
const arch = new THREE.Mesh(
  new THREE.CylinderGeometry(0.05, 0.05, 0.025, 10, 1, false, 0, Math.PI),
  dhWhite
);
arch.rotation.z = Math.PI/2;
arch.rotation.y = Math.PI/2;
arch.position.set(dhX, -3 + dH + 0.045, fz + 0.005);
scene.add(arch);
// knob
const knob = new THREE.Mesh(new THREE.SphereGeometry(0.012, 6, 6), dhWhite);
knob.position.set(dhX + 0.036, -3 + dH/2 + 0.02, fz + 0.032);
scene.add(knob);

// ── CEILING LAMP ──
const lampMat = new THREE.MeshStandardMaterial({ color: 0x2a2418, roughness: 0.8 });
const cord = new THREE.Mesh(new THREE.CylinderGeometry(0.015, 0.015, 0.7, 6), lampMat);
cord.position.set(0, 2.65, 0);
scene.add(cord);

const shade = new THREE.Mesh(
  new THREE.CylinderGeometry(0.32, 0.16, 0.22, 12, 1, true),
  new THREE.MeshStandardMaterial({ color: 0x282018, side: THREE.DoubleSide, roughness: 0.8 })
);
shade.position.set(0, 2.25, 0);
scene.add(shade);

const bulbMat = new THREE.MeshStandardMaterial({
  color: 0xfff5d0, emissive: 0xffd070, emissiveIntensity: 2.0
});
const bulb = new THREE.Mesh(new THREE.SphereGeometry(0.07, 8, 8), bulbMat);
bulb.position.set(0, 2.28, 0);
scene.add(bulb);

// ── LIGHTS ──
scene.add(new THREE.AmbientLight(0x2a2418, 0.45));

const mainLight = new THREE.PointLight(0xfff5d0, 2.8, 14);
mainLight.position.set(0, 2.25, 0);
mainLight.castShadow = true;
mainLight.shadow.mapSize.set(512, 512);
scene.add(mainLight);

// ── CONTROLS ──
let yaw = 0, pitch = 0, targetYaw = 0, targetPitch = 0;
const keys = {};
window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
window.addEventListener('keyup',   e => keys[e.key.toLowerCase()] = false);

// mouse look
let mouseDown = false, mPrevX = 0, mPrevY = 0;
canvas.addEventListener('mousedown', e => { mouseDown = true; mPrevX = e.clientX; mPrevY = e.clientY; });
window.addEventListener('mouseup',   () => mouseDown = false);
window.addEventListener('mousemove', e => {
  if (!mouseDown) return;
  targetYaw  -= (e.clientX - mPrevX) * 0.004;
  targetPitch -= (e.clientY - mPrevY) * 0.003;
  targetPitch = Math.max(-0.5, Math.min(0.5, targetPitch));
  mPrevX = e.clientX; mPrevY = e.clientY;
});

window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

// ── JOYSTICK ──
let joyDx = 0, joyDy = 0, joyTouchId = null;
let lookTouchId = null, lookPrevX = 0, lookPrevY = 0;

const S = 120, K = 50, jR = (S - K) / 2;
const joy = document.createElement('div');
joy.style.cssText = `position:fixed;left:24px;bottom:36px;width:${S}px;height:${S}px;border-radius:50%;background:rgba(255,255,255,0.1);border:1.5px solid rgba(255,255,255,0.25);z-index:999;`;
const knob = document.createElement('div');
knob.style.cssText = `position:absolute;width:${K}px;height:${K}px;border-radius:50%;background:rgba(255,255,255,0.4);left:${(S-K)/2}px;top:${(S-K)/2}px;pointer-events:none;`;
joy.appendChild(knob); document.body.appendChild(joy);

function joyReset() {
  joyDx = 0; joyDy = 0; joyTouchId = null;
  knob.style.left = (S-K)/2 + 'px';
  knob.style.top  = (S-K)/2 + 'px';
}

joy.addEventListener('touchstart', e => {
  e.preventDefault();
  if (joyTouchId !== null) return;
  const t = e.changedTouches[0];
  joyTouchId = t.identifier;
}, { passive: false });

document.addEventListener('touchmove', e => {
  for (const t of e.changedTouches) {
    if (t.identifier === joyTouchId) {
      e.preventDefault();
      const r = joy.getBoundingClientRect();
      let dx = t.clientX - (r.left + S/2);
      let dy = t.clientY - (r.top  + S/2);
      const d = Math.sqrt(dx*dx+dy*dy);
      if (d > jR) { dx = dx/d*jR; dy = dy/d*jR; }
      joyDx = dx/jR; joyDy = dy/jR;
      knob.style.left = ((S-K)/2 + dx) + 'px';
      knob.style.top  = ((S-K)/2 + dy) + 'px';
    } else if (t.identifier === lookTouchId) {
      targetYaw  -= (t.clientX - lookPrevX) * 0.004;
      targetPitch -= (t.clientY - lookPrevY) * 0.003;
      targetPitch = Math.max(-0.5, Math.min(0.5, targetPitch));
      lookPrevX = t.clientX; lookPrevY = t.clientY;
    }
  }
}, { passive: false });

document.addEventListener('touchend', e => {
  for (const t of e.changedTouches) {
    if (t.identifier === joyTouchId) joyReset();
    if (t.identifier === lookTouchId) lookTouchId = null;
  }
});

document.addEventListener('touchstart', e => {
  for (const t of e.changedTouches) {
    const r = joy.getBoundingClientRect();
    const inJoy = t.clientX >= r.left && t.clientX <= r.right && t.clientY >= r.top && t.clientY <= r.bottom;
    if (!inJoy && lookTouchId === null) {
      lookTouchId = t.identifier;
      lookPrevX = t.clientX; lookPrevY = t.clientY;
    }
  }
}, { passive: true });

// ── LOOP ──
const clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  clock.getDelta();
  const t = clock.getElapsedTime();
  const speed = 0.055;

  yaw   += (targetYaw - yaw) * 0.1;
  pitch += (targetPitch - pitch) * 0.1;

  const fwd   = new THREE.Vector3(Math.sin(yaw), 0, -Math.cos(yaw));
  const right = new THREE.Vector3(Math.cos(yaw), 0,  Math.sin(yaw));

  if (keys['w']||keys['arrowup'])    camera.position.addScaledVector(fwd,   speed);
  if (keys['s']||keys['arrowdown'])  camera.position.addScaledVector(fwd,  -speed);
  if (keys['a']||keys['arrowleft'])  camera.position.addScaledVector(right,-speed);
  if (keys['d']||keys['arrowright']) camera.position.addScaledVector(right, speed);

  camera.position.addScaledVector(fwd,  -joyDy * speed);
  camera.position.addScaledVector(right, joyDx * speed);

  camera.position.x = Math.max(-4.4, Math.min(4.4, camera.position.x));
  camera.position.z = Math.max(-5.4, Math.min(5.4, camera.position.z));
  camera.position.y = 0;

  camera.lookAt(camera.position.clone().add(
    new THREE.Vector3(Math.sin(yaw)*Math.cos(pitch), Math.sin(pitch), -Math.cos(yaw)*Math.cos(pitch))
  ));

  mainLight.intensity = 2.8 + Math.sin(t*0.8)*0.08;
  bulbMat.emissiveIntensity = 2.0 + Math.sin(t*0.8)*0.15;
  renderer.render(scene, camera);
}
animate();
</script>
<script>
const canvas = document.getElementById('c');
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, powerPreference: 'high-performance' });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0a0908);
scene.fog = new THREE.FogExp2(0x0a0908, 0.018);

const camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 50);
camera.position.set(0, 0, 0.1);

// ── FLOOR: concrete tile texture via canvas ──
const tileCanvas = document.createElement('canvas');
tileCanvas.width = 512; tileCanvas.height = 512;
const tc = tileCanvas.getContext('2d');
// base concrete grey
tc.fillStyle = '#5a5a58';
tc.fillRect(0, 0, 512, 512);
// noise for concrete texture
for (let i = 0; i < 8000; i++) {
  const x = Math.random() * 512, y = Math.random() * 512;
  const v = Math.floor(Math.random() * 30) - 15;
  const c = 90 + v;
  tc.fillStyle = `rgba(${c},${c},${c-2},0.18)`;
  tc.fillRect(x, y, Math.random()*3+1, Math.random()*3+1);
}
// large tile grout lines (2x2 grid = big tiles)
tc.strokeStyle = 'rgba(60,60,58,0.25)';
tc.lineWidth = 2;
tc.beginPath(); tc.moveTo(256,0); tc.lineTo(256,512); tc.stroke();
tc.beginPath(); tc.moveTo(0,256); tc.lineTo(512,256); tc.stroke();
// subtle tile edge highlight
tc.strokeStyle = 'rgba(120,120,118,0.2)';
tc.lineWidth = 1;
tc.strokeRect(4, 4, 248, 248);
tc.strokeRect(260, 4, 248, 248);
tc.strokeRect(4, 260, 248, 248);
tc.strokeRect(260, 260, 248, 248);

const tileTex = new THREE.CanvasTexture(tileCanvas);
tileTex.wrapS = tileTex.wrapT = THREE.RepeatWrapping;
tileTex.repeat.set(4, 5); // big tiles = fewer repeats

const floorMat = new THREE.MeshStandardMaterial({
  map: tileTex, roughness: 0.85, metalness: 0.05
});
const floor = new THREE.Mesh(new THREE.BoxGeometry(10, 0.1, 12), floorMat);
floor.position.set(0, -3, 0);
floor.receiveShadow = true;
scene.add(floor);

// ── WALLS & CEILING ──
function wall(w, h, d, color, x, y, z) {
  const m = new THREE.Mesh(
    new THREE.BoxGeometry(w, h, d),
    new THREE.MeshStandardMaterial({ color, roughness: 0.92, metalness: 0.02 })
  );
  m.position.set(x, y, z);
  m.receiveShadow = true;
  scene.add(m);
}
wall(10, 0.1, 12, 0x1a1815, 0, 3, 0);       // ceiling
wall(10, 6, 0.15, 0x1e1c1a, 0, 0, -6);      // back
wall(0.15, 6, 12, 0x1c1a18, -5, 0, 0);      // left
wall(0.15, 6, 12, 0x1a1816, 5, 0, 0);       // right

// ── WINDOW: white metal frame, 2 panes only ──
const metalMat = new THREE.MeshStandardMaterial({
  color: 0xe8e8e8, roughness: 0.2, metalness: 0.85
});
const winW = 2.4, winH = 2.8, winZ = -6 + 0.12;

// outer frame only (4 bars) — no inner dividers
const mkBar = (w, h, d, x, y) => {
  const b = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), metalMat);
  b.position.set(x, y, winZ);
  scene.add(b);
};
mkBar(winW + 0.14, 0.08, 0.1, 2.0, winH/2 + 0.04);   // top
mkBar(winW + 0.14, 0.08, 0.1, 2.0, -winH/2 - 0.04);  // bottom
mkBar(0.08, winH + 0.08, 0.1, 2.0 - winW/2 - 0.04, 0);  // left
mkBar(0.08, winH + 0.08, 0.1, 2.0 + winW/2 + 0.04, 0);  // right
// single center vertical divider only
mkBar(0.05, winH, 0.08, 2.0, 0);

// glass: 2 panes
const glassMat = new THREE.MeshStandardMaterial({
  color: 0xc8dde8, roughness: 0.0, metalness: 0.1,
  transparent: true, opacity: 0.12, side: THREE.DoubleSide
});
const mkPane = (w, h, x) => {
  const g = new THREE.Mesh(new THREE.PlaneGeometry(w, h), glassMat);
  g.position.set(x, 0, winZ + 0.01);
  scene.add(g);
};
mkPane(winW/2 - 0.04, winH - 0.06, 2.0 - winW/4);  // left pane
mkPane(winW/2 - 0.04, winH - 0.06, 2.0 + winW/4);  // right pane

// window light
const winLight = new THREE.PointLight(0xc8dde8, 0.6, 6);
winLight.position.set(2.0, 0.5, -5.5);
scene.add(winLight);

// ── BED (headboard in corner: right wall + back wall, bed extends left) ──
const whiteMat = new THREE.MeshStandardMaterial({ color: 0xf0eeea, roughness: 0.5, metalness: 0.15 });
const mattressMat = new THREE.MeshStandardMaterial({ color: 0x1a1a1a, roughness: 0.92 });

function makeDotTex() {
  const dc = document.createElement('canvas');
  dc.width = 256; dc.height = 256;
  const ctx = dc.getContext('2d');
  ctx.fillStyle = '#e8e2d8';
  ctx.fillRect(0, 0, 256, 256);
  ctx.fillStyle = '#a8a09a';
  for (let row = 0; row < 16; row++) {
    for (let col = 0; col < 16; col++) {
      ctx.beginPath();
      ctx.arc(col * 16 + 8, row * 16 + 8, 1.8, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  const t = new THREE.CanvasTexture(dc);
  t.wrapS = t.wrapT = THREE.RepeatWrapping;
  t.repeat.set(3, 4);
  return t;
}
const dottedMat = new THREE.MeshStandardMaterial({ map: makeDotTex(), roughness: 0.95 });

// bed: headboard in right+back corner, extends left along back wall
// platform (runs along X axis)
const bedPlatform = new THREE.Mesh(new THREE.BoxGeometry(3.8, 0.16, 2.2), whiteMat);
bedPlatform.position.set(2.9, -2.92, -4.9);
scene.add(bedPlatform);

// 4 legs
const legG2 = new THREE.BoxGeometry(0.1, 0.28, 0.1);
[[1.1, -5.85], [4.7, -5.85], [1.1, -3.95], [4.7, -3.95]].forEach(([x, z]) => {
  const l = new THREE.Mesh(legG2, whiteMat);
  l.position.set(x, -3.14, z);
  scene.add(l);
});

// mattress
const mattress = new THREE.Mesh(new THREE.BoxGeometry(3.6, 0.24, 2.1), mattressMat);
mattress.position.set(2.9, -2.68, -4.9);
scene.add(mattress);

// headboard against right wall
const headboard = new THREE.Mesh(new THREE.BoxGeometry(0.1, 1.0, 2.2), whiteMat);
headboard.position.set(4.88, -2.42, -4.9);
scene.add(headboard);

// duvet
const duvet = new THREE.Mesh(new THREE.BoxGeometry(2.6, 0.15, 2.05), dottedMat);
duvet.position.set(2.3, -2.5, -4.9);
scene.add(duvet);

// one pillow near headboard (right side)
const pillow = new THREE.Mesh(new THREE.BoxGeometry(0.55, 0.14, 1.1), dottedMat);
pillow.position.set(4.45, -2.49, -4.9);
scene.add(pillow);


// ── BOOKSHELVES (open shelf units, 2x2 grid, front-right corner) ──
const swMat = new THREE.MeshStandardMaterial({ color: 0x3d2010, roughness: 0.85, metalness: 0.02 });
const slMat = new THREE.MeshStandardMaterial({ color: 0x5c3418, roughness: 0.82, metalness: 0.02 });

function makeOpenShelf(cx, cz) {
  const w = 2.0, h = 2.2, d = 0.48, t = 0.055;
  const by = -3;

  // left panel
  const lp = new THREE.Mesh(new THREE.BoxGeometry(t, h, d), swMat);
  lp.position.set(cx - w/2 + t/2, by + h/2, cz);
  scene.add(lp);

  // right panel
  const rp = new THREE.Mesh(new THREE.BoxGeometry(t, h, d), swMat);
  rp.position.set(cx + w/2 - t/2, by + h/2, cz);
  scene.add(rp);

  // back panel
  const bp = new THREE.Mesh(new THREE.BoxGeometry(w, h, t), swMat);
  bp.position.set(cx, by + h/2, cz + d/2 - t/2);
  scene.add(bp);

  // bottom board
  const btm = new THREE.Mesh(new THREE.BoxGeometry(w - t*2, t, d), slMat);
  btm.position.set(cx, by + t/2, cz);
  scene.add(btm);

  // top board
  const top = new THREE.Mesh(new THREE.BoxGeometry(w + 0.02, t, d + 0.02), slMat);
  top.position.set(cx, by + h - t/2, cz);
  scene.add(top);

  // 2 horizontal shelves → 3 rows
  [h/3, h*2/3].forEach(yOff => {
    const shelf = new THREE.Mesh(new THREE.BoxGeometry(w - t*2, t, d - 0.02), slMat);
    shelf.position.set(cx, by + yOff, cz);
    scene.add(shelf);
  });
}

makeOpenShelf(3.95, 5.76);
makeOpenShelf(1.9, 5.76);



// ── DOLLHOUSE (purple, 2 floors, pitched roof, white trim, in front of bed) ──
const dhWall = new THREE.MeshStandardMaterial({ color: 0xb48fd4, roughness: 0.85 });
const dhWhite = new THREE.MeshStandardMaterial({ color: 0xf0eeea, roughness: 0.6, metalness: 0.1 });
const dhDark = new THREE.MeshStandardMaterial({ color: 0x1a1020, roughness: 0.9 });
const dhRoof = new THREE.MeshStandardMaterial({ color: 0xe8e4de, roughness: 0.75 });

const dhX = 0, dhY = -3, dhZ = -2.0;
const dhW = 1.1*0.6, dhH = 1.4*0.6, dhD = 0.6*0.6;

// === BODY ===
// floor 1
const f1 = new THREE.Mesh(new THREE.BoxGeometry(dhW, dhH/2, dhD), dhWall);
f1.position.set(dhX, dhY + dhH/4, dhZ);
scene.add(f1);

// floor 2
const f2 = new THREE.Mesh(new THREE.BoxGeometry(dhW, dhH/2, dhD), dhWall);
f2.position.set(dhX, dhY + dhH*3/4, dhZ);
scene.add(f2);

// white corner trims (4 vertical strips)
[-dhW/2, dhW/2].forEach(x => {
  const trim = new THREE.Mesh(new THREE.BoxGeometry(0.045, dhH, 0.065), dhWhite);
  trim.position.set(dhX + x, dhY + dhH/2, dhZ - dhD/2 + 0.03);
  scene.add(trim);
});

// floor divider strip
const divStrip = new THREE.Mesh(new THREE.BoxGeometry(dhW + 0.01, 0.05, dhD + 0.01), dhWhite);
divStrip.position.set(dhX, dhY + dhH/2, dhZ);
scene.add(divStrip);

// base strip
const baseStrip = new THREE.Mesh(new THREE.BoxGeometry(dhW + 0.02, 0.05, dhD + 0.02), dhWhite);
baseStrip.position.set(dhX, dhY + 0.025, dhZ);
scene.add(baseStrip);

// === ROOF (pitched) ===
// two sloped panels using rotated boxes
const roofH = 0.42*0.6, roofW = dhW/2 + 0.18;
[-1, 1].forEach(side => {
  const panel = new THREE.Mesh(new THREE.BoxGeometry(roofW, 0.05, dhD + 0.12), dhRoof);
  panel.position.set(dhX + side * roofW/2 * 0.5, dhY + dhH + roofH/2 - 0.05, dhZ);
  panel.rotation.z = side * -0.42;
  scene.add(panel);
});
// roof ridge cap
const ridge = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.07, dhD + 0.14), dhWhite);
ridge.position.set(dhX, dhY + dhH + roofH - 0.04, dhZ);
scene.add(ridge);
// gable end pieces removed

// === WINDOWS (white frames, dark glass) ===
function dhWindow(wx, wy, wz, sw, sh) {
  // dark glass
  const glass = new THREE.Mesh(new THREE.BoxGeometry(sw, sh, 0.03), dhDark);
  glass.position.set(wx, wy, wz);
  scene.add(glass);
  // white outer frame
  const frame = new THREE.Mesh(new THREE.BoxGeometry(sw + 0.06, sh + 0.06, 0.025), dhWhite);
  frame.position.set(wx, wy, wz + 0.008);
  scene.add(frame);
  // re-add glass on top
  const glass2 = new THREE.Mesh(new THREE.BoxGeometry(sw, sh, 0.03), dhDark);
  glass2.position.set(wx, wy, wz + 0.018);
  scene.add(glass2);
  // cross dividers
  const hbar = new THREE.Mesh(new THREE.BoxGeometry(sw, 0.02, 0.035), dhWhite);
  hbar.position.set(wx, wy, wz + 0.022);
  scene.add(hbar);
  const vbar = new THREE.Mesh(new THREE.BoxGeometry(0.02, sh, 0.035), dhWhite);
  vbar.position.set(wx, wy, wz + 0.022);
  scene.add(vbar);
}

// house is 0.6 scale: dhW=0.66, dhH=0.84, dhD=0.36
// front face z = dhZ - 0.18 - 0.01 = -2.19
const fz = dhZ - 0.19; // front face of dollhouse

// floor 2 (y from -3+0.63 to -3+0.84): centre y = -2.58
// 3 windows across
dhWindow(dhX - 0.18, -2.58, fz, 0.11, 0.11);
dhWindow(dhX,        -2.58, fz, 0.11, 0.11);
dhWindow(dhX + 0.18, -2.58, fz, 0.11, 0.11);

// floor 1 (y from -3 to -3+0.42): centre y = -2.79
// 2 side windows
dhWindow(dhX - 0.22, -2.79, fz, 0.11, 0.13);
dhWindow(dhX + 0.22, -2.79, fz, 0.11, 0.13);

// centre door floor 1
const dW = 0.1, dH = 0.2;
const doorGlass = new THREE.Mesh(new THREE.BoxGeometry(dW, dH, 0.035), dhDark);
doorGlass.position.set(dhX, -3 + dH/2 + 0.02, fz);
scene.add(doorGlass);
const doorFr = new THREE.Mesh(new THREE.BoxGeometry(dW+0.05, dH+0.04, 0.025), dhWhite);
doorFr.position.set(dhX, -3 + dH/2 + 0.02, fz + 0.005);
scene.add(doorFr);
const doorFill = new THREE.Mesh(new THREE.BoxGeometry(dW, dH, 0.035), dhDark);
doorFill.position.set(dhX, -3 + dH/2 + 0.02, fz + 0.018);
scene.add(doorFill);
// arch over door
const arch = new THREE.Mesh(
  new THREE.CylinderGeometry(0.05, 0.05, 0.025, 10, 1, false, 0, Math.PI),
  dhWhite
);
arch.rotation.z = Math.PI/2;
arch.rotation.y = Math.PI/2;
arch.position.set(dhX, -3 + dH + 0.045, fz + 0.005);
scene.add(arch);
// knob
const knob = new THREE.Mesh(new THREE.SphereGeometry(0.012, 6, 6), dhWhite);
knob.position.set(dhX + 0.036, -3 + dH/2 + 0.02, fz + 0.032);
scene.add(knob);

// ── CEILING LAMP ──
const lampMat = new THREE.MeshStandardMaterial({ color: 0x2a2418, roughness: 0.8 });
const cord = new THREE.Mesh(new THREE.CylinderGeometry(0.015, 0.015, 0.7, 6), lampMat);
cord.position.set(0, 2.65, 0);
scene.add(cord);

const shade = new THREE.Mesh(
  new THREE.CylinderGeometry(0.32, 0.16, 0.22, 12, 1, true),
  new THREE.MeshStandardMaterial({ color: 0x282018, side: THREE.DoubleSide, roughness: 0.8 })
);
shade.position.set(0, 2.25, 0);
scene.add(shade);

const bulbMat = new THREE.MeshStandardMaterial({
  color: 0xfff5d0, emissive: 0xffd070, emissiveIntensity: 2.0
});
const bulb = new THREE.Mesh(new THREE.SphereGeometry(0.07, 8, 8), bulbMat);
bulb.position.set(0, 2.28, 0);
scene.add(bulb);

// ── LIGHTS ──
scene.add(new THREE.AmbientLight(0x2a2418, 0.45));

const mainLight = new THREE.PointLight(0xfff5d0, 2.8, 14);
mainLight.position.set(0, 2.25, 0);
mainLight.castShadow = true;
mainLight.shadow.mapSize.set(512, 512);
scene.add(mainLight);

// ── CONTROLS ──
let isDragging = false, prevX = 0, prevY = 0;
let yaw = 0, pitch = 0, targetYaw = 0, targetPitch = 0;
const keys = {};

// look drag (right half of screen on mobile)
const onDown = (x, y) => { isDragging = true; prevX = x; prevY = y; };
const onMove = (x, y) => {
  if (!isDragging) return;
  targetYaw  -= (x - prevX) * 0.004;
  targetPitch -= (y - prevY) * 0.003;
  targetPitch = Math.max(-0.5, Math.min(0.5, targetPitch));
  prevX = x; prevY = y;
};
const onUp = () => { isDragging = false; };

canvas.addEventListener('mousedown', e => onDown(e.clientX, e.clientY));
window.addEventListener('mousemove', e => onMove(e.clientX, e.clientY));
window.addEventListener('mouseup', onUp);

// WASD
window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
window.addEventListener('keyup',   e => keys[e.key.toLowerCase()] = false);

// ── VIRTUAL JOYSTICK ──
const joySize = 120, knobSize = 48, joyR = (joySize - knobSize) / 2;
const joyZone = document.createElement('div');
joyZone.style.cssText = `position:fixed;left:20px;bottom:30px;width:${joySize}px;height:${joySize}px;border-radius:50%;background:rgba(255,255,255,0.1);border:1.5px solid rgba(255,255,255,0.2);z-index:100;touch-action:none;`;
const joyKnob = document.createElement('div');
joyKnob.style.cssText = `position:absolute;width:${knobSize}px;height:${knobSize}px;border-radius:50%;background:rgba(255,255,255,0.35);left:${(joySize-knobSize)/2}px;top:${(joySize-knobSize)/2}px;`;
joyZone.appendChild(joyKnob);
document.body.appendChild(joyZone);

let joyActive = false, joyTouchId = -1, joyDx = 0, joyDy = 0;

joyZone.addEventListener('touchstart', e => {
  e.preventDefault();
  e.stopPropagation();
  const t = e.changedTouches[0];
  joyTouchId = t.identifier;
  joyActive = true;
}, { passive: false });

joyZone.addEventListener('touchmove', e => {
  e.preventDefault();
  e.stopPropagation();
  for (const t of e.changedTouches) {
    if (t.identifier !== joyTouchId) continue;
    const rect = joyZone.getBoundingClientRect();
    let dx = t.clientX - (rect.left + joySize/2);
    let dy = t.clientY - (rect.top  + joySize/2);
    const dist = Math.sqrt(dx*dx + dy*dy);
    if (dist > joyR) { dx = dx/dist*joyR; dy = dy/dist*joyR; }
    joyDx = dx / joyR;
    joyDy = dy / joyR;
    joyKnob.style.left = ((joySize-knobSize)/2 + dx) + 'px';
    joyKnob.style.top  = ((joySize-knobSize)/2 + dy) + 'px';
  }
}, { passive: false });

joyZone.addEventListener('touchend', e => {
  e.preventDefault();
  joyActive = false; joyTouchId = -1; joyDx = 0; joyDy = 0;
  joyKnob.style.left = ((joySize-knobSize)/2) + 'px';
  joyKnob.style.top  = ((joySize-knobSize)/2) + 'px';
}, { passive: false });

// look drag — entire canvas, ignore joystick area
let lookId = -1, lookPrevX = 0, lookPrevY = 0;
canvas.addEventListener('touchstart', e => {
  for (const t of e.changedTouches) {
    if (lookId === -1 && t.identifier !== joyTouchId) {
      lookId = t.identifier; lookPrevX = t.clientX; lookPrevY = t.clientY;
    }
  }
}, { passive: true });
canvas.addEventListener('touchmove', e => {
  for (const t of e.changedTouches) {
    if (t.identifier === lookId) {
      targetYaw  -= (t.clientX - lookPrevX) * 0.004;
      targetPitch -= (t.clientY - lookPrevY) * 0.003;
      targetPitch = Math.max(-0.5, Math.min(0.5, targetPitch));
      lookPrevX = t.clientX; lookPrevY = t.clientY;
    }
  }
}, { passive: true });
canvas.addEventListener('touchend', e => {
  for (const t of e.changedTouches) {
    if (t.identifier === lookId) lookId = -1;
  }
}, { passive: true });

window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

// ── LOOP ──
const clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  const t = clock.getElapsedTime();
  yaw   += (targetYaw - yaw) * 0.08;
  pitch += (targetPitch - pitch) * 0.08;
  const dir = new THREE.Vector3(
    Math.sin(yaw) * Math.cos(pitch),
    Math.sin(pitch),
    -Math.cos(yaw) * Math.cos(pitch)
  );
  camera.lookAt(camera.position.clone().add(dir));
  mainLight.intensity = 2.8 + Math.sin(t * 0.8) * 0.08;
  bulbMat.emissiveIntensity = 2.0 + Math.sin(t * 0.8) * 0.15;
  renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
