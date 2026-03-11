# Rocker_boogie_suspension_sim_bymark
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>NASA Rocker-Bogie Simulator</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700&display=swap');
* { margin:0; padding:0; box-sizing:border-box; }
body { background:#060c16; overflow:hidden; font-family:'Share Tech Mono',monospace; color:#8ab8d8; }
#app { display:flex; height:100vh; width:100vw; }

/* SIDEBAR */
#sidebar {
  width:240px; min-width:240px; background:#080f1c;
  border-right:1px solid #0d2035;
  display:flex; flex-direction:column; overflow-y:auto; z-index:10;
}
.sb-sec { border-bottom:1px solid #0d2035; padding:12px; }
.sb-title {
  font-family:'Orbitron',monospace; font-size:8px; font-weight:700;
  letter-spacing:3px; color:#00f5c8; margin-bottom:10px;
}
.sc-btn {
  display:block; width:100%; background:transparent;
  border:1px solid #0d2035; color:#6a9ab8;
  font-family:'Share Tech Mono',monospace; font-size:10px;
  letter-spacing:1px; padding:7px 10px; cursor:pointer;
  margin-bottom:4px; border-radius:3px; text-align:left;
  transition:all 0.15s;
}
.sc-btn:hover { color:#aad4f0; border-color:#1a3050; }
.sc-btn.active { color:#00f5c8; border-color:#00f5c8; background:rgba(0,245,200,0.06); }
.tele-row { display:flex; justify-content:space-between; align-items:center; margin-bottom:5px; }
.tele-lbl { font-size:9px; color:#2a4a6a; letter-spacing:1px; }
.tele-val { font-family:'Orbitron',monospace; font-size:10px; font-weight:700; color:#00f5c8; }
.tele-val.warn { color:#ff8c00; }
.tele-val.crit { color:#ff3a3a; }
.vb-row { display:flex; flex-wrap:wrap; gap:4px; }
.vb {
  flex:1; min-width:50px; background:transparent;
  border:1px solid #0d2035; color:#2a4a6a;
  font-family:'Share Tech Mono',monospace; font-size:9px;
  padding:5px 3px; cursor:pointer; border-radius:2px;
  text-align:center; transition:all 0.15s;
}
.vb:hover,.vb.active { border-color:#00f5c8; color:#00f5c8; background:rgba(0,245,200,0.07); }
.spd-wrap { display:flex; align-items:center; gap:6px; margin-top:5px; }
.spd-wrap label { font-size:8px; color:#2a4a6a; }
input[type=range] { flex:1; accent-color:#00f5c8; cursor:pointer; }
#info-txt { font-size:9px; line-height:1.9; color:#2a4a6a; }

/* MAIN */
#main { flex:1; position:relative; display:flex; flex-direction:column; }
#cw { flex:1; position:relative; }
canvas { display:block; width:100%; height:100%; }

/* BOTTOM BAR */
#bot {
  height:46px; min-height:46px; background:#080f1c;
  border-top:1px solid #0d2035;
  display:flex; align-items:center; padding:0 16px; gap:14px;
}
#play-btn {
  background:#00f5c8; color:#060c16; border:none;
  font-family:'Orbitron',monospace; font-size:10px; font-weight:700;
  letter-spacing:2px; padding:8px 20px; cursor:pointer; border-radius:3px;
}
.bb-lbl { font-size:9px; color:#1a3050; }
.bb-val { font-family:'Orbitron',monospace; font-size:10px; color:#00f5c8; margin-right:12px; }

/* LEGEND */
#leg {
  position:absolute; bottom:10px; right:10px;
  background:rgba(6,12,22,0.90); border:1px solid #0d2035;
  border-radius:4px; padding:9px 12px; pointer-events:none;
}
.lr { display:flex; align-items:center; gap:7px; font-size:9px; color:#2a4a6a; margin-bottom:4px; }
.ld { width:9px; height:9px; border-radius:1px; flex-shrink:0; }

/* NASA LABEL */
#nasa-lbl {
  position:absolute; top:12px; left:12px; pointer-events:none;
}
#nasa-lbl h1 { font-family:'Orbitron',monospace; font-size:9px; letter-spacing:3px; color:#00f5c8; }
#nasa-lbl h2 { font-family:'Orbitron',monospace; font-size:16px; font-weight:700; color:#fff; margin-top:2px; }
#nasa-lbl p  { font-size:9px; color:#1a3050; letter-spacing:2px; margin-top:2px; }
</style>
</head>
<body>
<div id="app">
<div id="sidebar">

  <div class="sb-sec">
    <div class="sb-title">// TERRAIN SCENARIO</div>
    <button class="sc-btn active" id="sc-flat"  onclick="setSc('flat')">▬&nbsp; FLAT GROUND</button>
    <button class="sc-btn"        id="sc-rock"  onclick="setSc('rock')">◆&nbsp; ROCK OBSTACLE</button>
    <button class="sc-btn"        id="sc-rocks" onclick="setSc('rocks')">◈&nbsp; ROCKY FIELD</button>
    <button class="sc-btn"        id="sc-slope" onclick="setSc('slope')">◥&nbsp; SLOPE 25°</button>
    <button class="sc-btn"        id="sc-ditch" onclick="setSc('ditch')">▽&nbsp; DITCH</button>
    <button class="sc-btn"        id="sc-step"  onclick="setSc('step')">▤&nbsp; STEP CLIMB</button>
    <button class="sc-btn"        id="sc-sand"  onclick="setSc('sand')">〜&nbsp; SAND DUNES</button>
  </div>

  <div class="sb-sec">
    <div class="sb-title">// TELEMETRY</div>
    <div class="tele-row"><span class="tele-lbl">CHASSIS PITCH</span><span class="tele-val" id="tv-pitch">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">CHASSIS ROLL</span><span class="tele-val" id="tv-roll">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">ROCKER L</span><span class="tele-val" id="tv-rl">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">ROCKER R</span><span class="tele-val" id="tv-rr">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">BOGIE L</span><span class="tele-val" id="tv-bl">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">BOGIE R</span><span class="tele-val" id="tv-br">0.0°</span></div>
    <div class="tele-row"><span class="tele-lbl">WHEELS ON GND</span><span class="tele-val" id="tv-wg">6/6</span></div>
  </div>

  <div class="sb-sec">
    <div class="sb-title">// CAMERA</div>
    <div class="vb-row">
      <button class="vb active" id="vb-iso"    onclick="setView('iso')">ISO</button>
      <button class="vb"        id="vb-side"   onclick="setView('side')">SIDE</button>
      <button class="vb"        id="vb-front"  onclick="setView('front')">FRONT</button>
      <button class="vb"        id="vb-top"    onclick="setView('top')">TOP</button>
      <button class="vb"        id="vb-follow" onclick="setView('follow')">FOLLOW</button>
    </div>
  </div>

  <div class="sb-sec">
    <div class="sb-title">// SPEED</div>
    <div class="spd-wrap">
      <label>SLOW</label>
      <input type="range" id="spd" min="0.2" max="3" step="0.1" value="1" oninput="simSpd=+this.value">
      <label>FAST</label>
    </div>
  </div>

  <div class="sb-sec">
    <div class="sb-title">// SCENARIO INFO</div>
    <div id="info-txt">Rover on flat ground.<br>All 6 wheels contact.<br>Differential bar level.</div>
  </div>

</div>

<div id="main">
  <div id="cw">
    <canvas id="c"></canvas>
    <div id="nasa-lbl">
      <h1>IRC ROVER — NASA CONFIGURATION</h1>
      <h2>Rocker-Bogie Suspension</h2>
      <p>SIDE-MOUNTED · FORWARD-FACING WHEELS · PASSIVE DIFF</p>
    </div>
    <div id="leg">
      <div class="lr"><div class="ld" style="background:#18c898"></div>ROCKER ARM (AL 6061)</div>
      <div class="lr"><div class="ld" style="background:#d08820"></div>BOGIE ARM (AL 6061)</div>
      <div class="lr"><div class="ld" style="background:#cc2222"></div>PIVOT JOINTS (STEEL)</div>
      <div class="lr"><div class="ld" style="background:#4466dd"></div>DIFFERENTIAL BAR</div>
      <div class="lr"><div class="ld" style="background:#445566"></div>WHEELS + MOTORS</div>
      <div class="lr"><div class="ld" style="background:#1a2b3c"></div>CHASSIS</div>
    </div>
  </div>
  <div id="bot">
    <button id="play-btn" onclick="togglePlay()">▶ PLAY</button>
    <span class="bb-lbl">SCENARIO&nbsp;</span><span class="bb-val" id="bb-sc">FLAT GROUND</span>
    <span class="bb-lbl">PROGRESS&nbsp;</span><span class="bb-val" id="bb-prog">0%</span>
    <span class="bb-lbl">TIME&nbsp;</span><span class="bb-val" id="bb-time">0.0s</span>
    <span style="flex:1"></span>
    <span style="font-size:8px;color:#0d2035">DRAG:ROTATE · SCROLL:ZOOM · RIGHT-DRAG:PAN</span>
  </div>
</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ═══════════════════════════════════════
// RENDERER
// ═══════════════════════════════════════
var canvas = document.getElementById('c');
var cw     = document.getElementById('cw');

var renderer = new THREE.WebGLRenderer({ canvas:canvas, antialias:true });
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type    = THREE.PCFSoftShadowMap;
renderer.toneMapping       = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.1;

var scene  = new THREE.Scene();
scene.background = new THREE.Color(0x060c16);
scene.fog        = new THREE.Fog(0x060c16, 7, 18);

var camera = new THREE.PerspectiveCamera(44, 1, 0.01, 40);

function onResize(){
  var w=cw.clientWidth, h=cw.clientHeight;
  renderer.setSize(w,h);
  camera.aspect = w/h;
  camera.updateProjectionMatrix();
}
window.addEventListener('resize', onResize);
setTimeout(onResize, 80);

// ═══════════════════════════════════════
// LIGHTS
// ═══════════════════════════════════════
scene.add(new THREE.AmbientLight(0x253550, 1.2));

var sun = new THREE.DirectionalLight(0xfff8e8, 1.9);
sun.position.set(5, 9, 4);
sun.castShadow = true;
sun.shadow.mapSize.width = sun.shadow.mapSize.height = 2048;
sun.shadow.camera.left=-4; sun.shadow.camera.right=4;
sun.shadow.camera.top=4;   sun.shadow.camera.bottom=-4;
sun.shadow.camera.near=1;  sun.shadow.camera.far=20;
scene.add(sun);

var fill = new THREE.DirectionalLight(0x00f5c8, 0.35);
fill.position.set(-3,2,-2); scene.add(fill);
var rim  = new THREE.DirectionalLight(0x4466ff, 0.2);
rim.position.set(0,0,-5); scene.add(rim);

// ═══════════════════════════════════════
// MATERIALS
// ═══════════════════════════════════════
function M(c,m,r){ return new THREE.MeshStandardMaterial({color:c,metalness:m||0.7,roughness:r||0.3}); }

var mRocker  = M(0x18c898, 0.75, 0.28);
var mBogie   = M(0xd08820, 0.75, 0.28);
var mPivot   = M(0xcc2222, 0.92, 0.18);
var mDiff    = M(0x4466dd, 0.65, 0.40);
var mWheel   = M(0x445566, 0.40, 0.75);
var mTire    = M(0x181820, 0.05, 0.95);
var mChassis = M(0x1a2b3c, 0.85, 0.25);
var mMotor   = M(0x222233, 0.90, 0.20);
var mDetail  = M(0x0d1a28, 0.50, 0.65);

// ═══════════════════════════════════════
// PRIMITIVES
// ═══════════════════════════════════════
function box(w,h,d,m){
  var me=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),m);
  me.castShadow=me.receiveShadow=true; return me;
}
function cyl(rt,rb,ht,m,s){
  var me=new THREE.Mesh(new THREE.CylinderGeometry(rt,rb,ht,s||16),m);
  me.castShadow=true; return me;
}
function sph(r,m){
  var me=new THREE.Mesh(new THREE.SphereGeometry(r,14,10),m);
  me.castShadow=true; return me;
}

// Orient a mesh so its LOCAL +Z points from p1 → p2
function orientBeam(mesh, x1,y1,z1, x2,y2,z2){
  var dx=x2-x1, dy=y2-y1, dz=z2-z1;
  var len=Math.sqrt(dx*dx+dy*dy+dz*dz);
  mesh.position.set((x1+x2)/2,(y1+y2)/2,(z1+z2)/2);
  var dir=new THREE.Vector3(dx,dy,dz).normalize();
  var up =new THREE.Vector3(0,1,0);
  if(Math.abs(dir.dot(up))>0.99) up.set(1,0,0);
  var quat=new THREE.Quaternion();
  quat.setFromUnitVectors(new THREE.Vector3(0,0,1), dir);
  mesh.setRotationFromQuaternion(quat);
  return len;
}

// ═══════════════════════════════════════
// ROVER DIMENSIONS (meters)
// ═══════════════════════════════════════
// COORDINATE SYSTEM (NASA style):
//   +X  = rover FORWARD
//   +Y  = UP
//   +Z  = rover LEFT side  (right side = -Z)
//
// NASA Curiosity key proportions (scaled to ~20kg rover):
//   Wheel radius        = 110mm
//   Chassis width       = 500mm
//   Chassis length      = 600mm
//   Chassis height      = 180mm
//   Ground clearance    = 170mm
//   Rocker pivot X      = ~40mm forward of chassis center
//   Rocker front arm    = 240mm  (pivot → front wheel)
//   Rocker rear arm     = 160mm  (pivot → bogie pivot)
//   Bogie half length   = 140mm  (bogie pivot → each wheel)

var WR   = 0.110;   // wheel radius
var WW   = 0.090;   // wheel width (tread depth)
var GC   = 0.170;   // ground clearance
var CHL  = 0.600;   // chassis length  X
var CHW  = 0.500;   // chassis width   Z
var CHH  = 0.180;   // chassis height  Y

var wheelCY  = WR;                // wheel center Y (on flat ground)
var chassBot = wheelCY + GC;      // chassis bottom Y
var chassMid = chassBot + CHH/2;  // chassis center Y
var chassTop = chassBot + CHH;    // chassis top Y

// Rocker pivot is bolted to the OUTER SIDE FACE of chassis
// In local rover coords (rover at origin):
var RP_X     =  0.04;       // slightly forward of chassis center
var RP_Y_LOC =  0;          // at chassis center height (local to chassis)
var RP_Z_ABS =  CHW / 2;    // distance from rover centerline to pivot

// Arm lengths
var RF = 0.240;  // rocker front  (pivot → front wheel)
var RR = 0.160;  // rocker rear   (pivot → bogie pivot)
var BH = 0.140;  // bogie half    (bogie pivot → each wheel)

// Neutral angles on flat ground
var R_ANG = 12 * Math.PI/180;   // rocker slopes down toward front
var B_ANG =  3 * Math.PI/180;   // bogie nearly horizontal

// ═══════════════════════════════════════
// WHEEL ASSEMBLY
// Axis along Z so wheel faces forward (rolls in X direction)
// ═══════════════════════════════════════
function makeWheel(){
  var g = new THREE.Group();

  // Main cylindrical body — axis Z
  var disc = cyl(WR, WR, WW, mWheel, 32);
  disc.rotation.x = Math.PI/2;
  g.add(disc);

  // Rubber tread — torus in X-Y plane → wheel faces forward ✓
  var tread = new THREE.Mesh(
    new THREE.TorusGeometry(WR*0.93, 0.020, 12, 40), mTire);
  g.add(tread);

  // Grousers (paddle lugs) — like NASA's 48 grouser wheels
  for(var i=0;i<12;i++){
    var a=(i/12)*Math.PI*2;
    var lug=box(WW*0.80, 0.014, 0.038, mTire);
    lug.position.set(0, Math.sin(a)*WR*0.91, Math.cos(a)*WR*0.91);
    lug.rotation.x = -a;
    g.add(lug);
  }

  // Spokes
  for(var j=0;j<5;j++){
    var sp=box(WW*0.65, 0.011, 0.011, mMotor);
    sp.rotation.z = (j/5)*Math.PI;
    g.add(sp);
  }

  // Hub
  var hub=cyl(0.024,0.024,WW+0.014,mPivot,10);
  hub.rotation.x=Math.PI/2; g.add(hub);

  // Motor housing (inside)
  var mot=cyl(0.031,0.031,0.072,mMotor,12);
  mot.rotation.x=Math.PI/2; g.add(mot);

  return g;
}

// ═══════════════════════════════════════
// ARM SEGMENT — hollow box profile
// built along LOCAL X axis, length=1 (scale to fit)
// ═══════════════════════════════════════
function makeArmProfile(m){
  var g=new THREE.Group();
  // outer shell — 1 unit long
  var outer=box(1.0, 0.028, 0.024, m); g.add(outer);
  // inner recess
  var inner=box(0.82, 0.013, 0.012, mDetail); g.add(inner);
  // end caps
  var c1=box(0.014,0.028,0.024,mPivot); c1.position.x=-0.5; g.add(c1);
  var c2=box(0.014,0.028,0.024,mPivot); c2.position.x= 0.5; g.add(c2);
  return g;
}

// Place an arm between two points (purely in X-Y plane, at fixed Z)
// Returns the group
function placeArm2D(x1,y1, x2,y2, z, m){
  var dx=x2-x1, dy=y2-y1;
  var len=Math.sqrt(dx*dx+dy*dy);
  var ang=Math.atan2(dy,dx);
  var arm=makeArmProfile(m);
  arm.scale.x=len;
  arm.position.set((x1+x2)/2, (y1+y2)/2, z);
  arm.rotation.z=ang;
  scene.add(arm);
  return arm;
}

// ═══════════════════════════════════════
// PIVOT JOINT
// ═══════════════════════════════════════
function makePivot(r){
  r=r||0.030;
  var g=new THREE.Group();
  g.add(sph(r, mPivot));
  // collar ring — runs along Z (left-right) for side-mounted arm
  var ring=cyl(r*1.55,r*1.55,0.013,mPivot,14);
  ring.rotation.x=Math.PI/2; g.add(ring);
  return g;
}

// ═══════════════════════════════════════
// SCENE OBJECTS (static chassis + dynamic arm meshes)
// ═══════════════════════════════════════

// Chassis group (moves as whole unit)
var roverGrp = new THREE.Group();
scene.add(roverGrp);

// --- CHASSIS BOX ---
var chassGrp = new THREE.Group();
roverGrp.add(chassGrp);

var chassBox = box(CHL, CHH, CHW, mChassis);
chassGrp.add(chassBox);
// Top panel
var tpan=box(CHL-0.04,0.005,CHW-0.04,mDetail);
tpan.position.y=CHH/2; chassGrp.add(tpan);
// Electronics bay
var elec=box(CHL*0.5,0.10,CHW*0.56,M(0x101c2c,0.6,0.5));
elec.position.set(-0.02,CHH/2+0.05,0); chassGrp.add(elec);
// Vent slots
for(var vi=0;vi<5;vi++){
  var vnt=box(CHL*0.44,0.010,0.011,mDetail);
  vnt.position.set(-0.02,CHH/2+0.09,(vi-2)*0.065); chassGrp.add(vnt);
}
// Camera mast
var mast=box(0.036,0.16,0.036,mChassis);
mast.position.set(CHL/2-0.04,CHH/2+0.08,0); chassGrp.add(mast);
var camH=box(0.068,0.044,0.068,M(0x101c2c,0.6,0.5));
camH.position.set(CHL/2-0.018,CHH/2+0.18,0); chassGrp.add(camH);
// Stereo lenses
[-0.024,0.024].forEach(function(lz){
  var lens=cyl(0.013,0.013,0.025,M(0x111111,0.4,0.9),10);
  lens.rotation.z=Math.PI/2;
  lens.position.set(CHL/2+0.01,CHH/2+0.178,lz); chassGrp.add(lens);
});
// Antenna
var antB=cyl(0.013,0.013,0.065,M(0x333333,0.85,0.2));
antB.position.set(-0.08,CHH/2+0.033,0.10); chassGrp.add(antB);
var antP=cyl(0.005,0.005,0.16,M(0xaaaaaa,0.9,0.1));
antP.position.set(-0.08,CHH/2+0.14,0.10); chassGrp.add(antP);
var antTip=sph(0.014,M(0xffaa00,0.8,0.3));
antTip.position.set(-0.08,CHH/2+0.225,0.10); chassGrp.add(antTip);

// --- DIFFERENTIAL BAR ---
// Runs along the TOP of chassis, left-right (Z axis)
// Center pivot on chassis top centerline
// Each end links down to respective rocker pivot
var diffGrp = new THREE.Group();
roverGrp.add(diffGrp);

var DIFF_Y_LOCAL = CHH/2 + 0.045; // above chassis top
var DIFF_SPAN    = RP_Z_ABS * 0.78;

var diffBar = cyl(0.017,0.017,DIFF_SPAN*2,mDiff,12);
diffBar.rotation.x=Math.PI/2; diffGrp.add(diffBar);
var diffCtrPiv = sph(0.022,mPivot); diffGrp.add(diffCtrPiv);

// End caps + links (updated each frame)
var diffEndL = cyl(0.019,0.019,0.022,mDiff,10);
diffEndL.rotation.x=Math.PI/2; diffGrp.add(diffEndL);
var diffEndR = cyl(0.019,0.019,0.022,mDiff,10);
diffEndR.rotation.x=Math.PI/2; diffGrp.add(diffEndR);

// Diff links (box beams updated each frame)
var diffLinkL = box(0.012,0.012,0.2,mDiff);
var diffLinkR = box(0.012,0.012,0.2,mDiff);
roverGrp.add(diffLinkL); roverGrp.add(diffLinkR);

// ═══════════════════════════════════════
// DYNAMIC SUSPENSION MESHES (per side)
// These get repositioned every frame
// ═══════════════════════════════════════
function makeSide(sign){
  // sign: +1=left(+Z)  -1=right(-Z)
  var armZ  = sign * RP_Z_ABS;          // Z of arm plane
  var whlZ  = sign * (RP_Z_ABS+0.040); // Z of wheel plane (slightly more outboard)

  // Rocker front arm (pivot→front wheel) — scaled box
  var rockerFront = makeArmProfile(mRocker);
  roverGrp.add(rockerFront);

  // Rocker rear arm (pivot→bogie pivot)
  var rockerRear = makeArmProfile(mRocker);
  roverGrp.add(rockerRear);

  // Bogie arm (mid wheel ↔ rear wheel)
  var bogieArm = makeArmProfile(mBogie);
  roverGrp.add(bogieArm);

  // Vertical drop links (arm endpoint → wheel hub)
  var dropF = box(0.012,0.012,0.01,mRocker); roverGrp.add(dropF);
  var dropM = box(0.012,0.012,0.01,mBogie);  roverGrp.add(dropM);
  var dropR = box(0.012,0.012,0.01,mBogie);  roverGrp.add(dropR);

  // Z-bracket (arm plane → wheel plane)
  var zbrkF = box(0.014,0.014,0.01,mChassis); roverGrp.add(zbrkF);
  var zbrkM = box(0.014,0.014,0.01,mChassis); roverGrp.add(zbrkM);
  var zbrkR = box(0.014,0.014,0.01,mChassis); roverGrp.add(zbrkR);

  // Wheels
  var wF=makeWheel(); roverGrp.add(wF);
  var wM=makeWheel(); roverGrp.add(wM);
  var wR=makeWheel(); roverGrp.add(wR);

  // Pivot spheres
  var pivRP = makePivot(0.032); roverGrp.add(pivRP);  // rocker pivot
  var pivBP = makePivot(0.026); roverGrp.add(pivBP);  // bogie pivot
  // Wheel hub pivots
  var pivWF=sph(0.017,mPivot); roverGrp.add(pivWF);
  var pivWM=sph(0.017,mPivot); roverGrp.add(pivWM);
  var pivWR=sph(0.017,mPivot); roverGrp.add(pivWR);

  // Chassis bracket (plate on side wall)
  var brkt=box(0.055,CHH*0.62,0.014,M(0x334455,0.85,0.2));
  roverGrp.add(brkt);

  // Fender
  var fend=box(0.68,0.008,0.048,M(0x1a2a3a,0.6,0.5));
  roverGrp.add(fend);

  return {
    sign:sign, armZ:armZ, whlZ:whlZ,
    rockerFront:rockerFront, rockerRear:rockerRear, bogieArm:bogieArm,
    dropF:dropF, dropM:dropM, dropR:dropR,
    zbrkF:zbrkF, zbrkM:zbrkM, zbrkR:zbrkR,
    wF:wF, wM:wM, wR:wR,
    pivRP:pivRP, pivBP:pivBP,
    pivWF:pivWF, pivWM:pivWM, pivWR:pivWR,
    brkt:brkt, fend:fend,
    // state (world Y of each wheel center)
    yF:WR, yM:WR, yR:WR,
    // computed angles (for telemetry)
    rockerAng:0, bogieAng:0
  };
}

var sideL = makeSide(+1);
var sideR = makeSide(-1);

// ═══════════════════════════════════════
// UPDATE ONE SIDE
// chassMidWorld = world Y of chassis center
// The arm plane is at Z = armZ (world, since rover moves only in X)
// ═══════════════════════════════════════
function updateSide(sd, chassMidWorld){
  var aZ  = sd.armZ;   // arm plane Z
  var wZ  = sd.whlZ;   // wheel plane Z

  // In rover-local coords (rover at X=0, Y=0 each frame):
  // chassis mid at Y=chassMidWorld
  // Rocker pivot:
  var rpX = RP_X;
  var rpY = chassMidWorld; // RP is at chassis mid height
  var rpZ = aZ;

  // Wheel positions come from terrain sampling (world Y)
  var yF  = sd.yF;  // front wheel center Y (world)
  var yM  = sd.yM;
  var yR  = sd.yR;

  // Front wheel X offset from rover center
  var fwX = RP_X + RF * Math.cos(R_ANG);
  var mwX = RP_X - RR - BH * Math.cos(B_ANG);
  var rwX = RP_X - RR + BH * Math.cos(B_ANG) - BH*2*Math.cos(B_ANG); // = RP_X-RR-BH
  // Simpler explicit:
  var bogPivX = RP_X - RR;
  var midWX   = bogPivX + BH;
  var rearWX  = bogPivX - BH;

  // Bogie pivot Y: average of mid and rear wheel Y, adjusted
  var bogPivY = (yM + yR) / 2 + 0.015;

  // Rocker pivot is fixed to chassis → rpY = chassMidWorld
  // Front arm: from rocker pivot to front wheel hub
  // The arm in XY plane (at Z=aZ)
  var rfDx = fwX  - rpX,  rfDy = yF  - rpY;
  var rfLen = Math.sqrt(rfDx*rfDx+rfDy*rfDy);
  var rfAng = Math.atan2(rfDy,rfDx);

  // Rear arm: from rocker pivot to bogie pivot
  var rrDx = bogPivX - rpX, rrDy = bogPivY - rpY;
  var rrLen = Math.sqrt(rrDx*rrDx+rrDy*rrDy);
  var rrAng = Math.atan2(rrDy,rrDx);

  // Bogie arm: from mid wheel to rear wheel (through bogie pivot center)
  var bmDx = midWX  - rearWX, bmDy = yM - yR;
  var bmLen = Math.sqrt(bmDx*bmDx+bmDy*bmDy);
  var bmAng = Math.atan2(bmDy,bmDx);

  // Store angles for telemetry
  sd.rockerAng = rfAng;
  sd.bogieAng  = bmAng;

  // ── PLACE ARM MESHES (in rover-local space via roverGrp) ──

  // Rocker front arm
  sd.rockerFront.scale.x = rfLen;
  sd.rockerFront.position.set((rpX+fwX)/2, (rpY+yF)/2, aZ);
  sd.rockerFront.rotation.set(0,0,rfAng);

  // Rocker rear arm
  sd.rockerRear.scale.x = rrLen;
  sd.rockerRear.position.set((rpX+bogPivX)/2, (rpY+bogPivY)/2, aZ);
  sd.rockerRear.rotation.set(0,0,rrAng);

  // Bogie arm
  sd.bogieArm.scale.x = bmLen;
  sd.bogieArm.position.set((midWX+rearWX)/2, (yM+yR)/2, aZ);
  sd.bogieArm.rotation.set(0,0,bmAng);

  // Pivots
  sd.pivRP.position.set(rpX,    rpY,    aZ);
  sd.pivBP.position.set(bogPivX,bogPivY,aZ);
  sd.pivWF.position.set(fwX,  yF, aZ);
  sd.pivWM.position.set(midWX,yM, aZ);
  sd.pivWR.position.set(rearWX,yR, aZ);

  // Drop links (vertical: arm endpoint → wheel center Y, at arm Z)
  function setDropLink(mesh, wx, wy_arm, wy_whl){
    var dy=Math.abs(wy_arm-wy_whl);
    if(dy<0.005){ mesh.visible=false; return; }
    mesh.visible=true;
    mesh.scale.y=dy;
    mesh.position.set(wx,(wy_arm+wy_whl)/2, aZ);
    mesh.rotation.set(0,0,0);
  }
  setDropLink(sd.dropF, fwX,   yF,   yF);   // front arm end is at wheel Y already
  setDropLink(sd.dropM, midWX, yM,   yM);
  setDropLink(sd.dropR, rearWX,yR,   yR);

  // Z-brackets (from arm Z to wheel Z)
  function setZbrk(mesh,wx,wy){
    var dz=Math.abs(wZ-aZ);
    mesh.scale.z=dz;
    mesh.position.set(wx,wy,(aZ+wZ)/2);
    mesh.rotation.set(0,0,0);
    mesh.visible=true;
  }
  setZbrk(sd.zbrkF, fwX,   yF);
  setZbrk(sd.zbrkM, midWX, yM);
  setZbrk(sd.zbrkR, rearWX,yR);

  // Wheels (at wheel plane wZ, wheel center Y)
  sd.wF.position.set(fwX,   yF, wZ);
  sd.wM.position.set(midWX, yM, wZ);
  sd.wR.position.set(rearWX,yR, wZ);

  // Spin wheels
  sd.wF.rotation.z += simSpd*0.025;
  sd.wM.rotation.z += simSpd*0.025;
  sd.wR.rotation.z += simSpd*0.025;

  // Chassis side bracket
  sd.brkt.position.set(rpX, chassMidWorld, sd.sign*(CHW/2-0.007));

  // Fender
  var fendCX = (fwX+rearWX)/2;
  var fendY  = Math.max(yF,yM,yR) + WR + 0.018;
  sd.fend.position.set(fendCX, fendY, wZ + sd.sign*0.005);
}

// ═══════════════════════════════════════
// GROUND + GRID
// ═══════════════════════════════════════
var gndMesh = new THREE.Mesh(
  new THREE.PlaneGeometry(14,4),
  new THREE.MeshStandardMaterial({color:0x080f1c,roughness:1,metalness:0})
);
gndMesh.rotation.x=-Math.PI/2; gndMesh.position.set(2,-0.006,0);
gndMesh.receiveShadow=true; scene.add(gndMesh);

var grid=new THREE.GridHelper(14,60,0x0d2030,0x080f1c);
grid.position.set(2,0,0); scene.add(grid);

// Terrain mesh (rebuilt per scenario)
var terrMesh=null;

// Trail
var trail=[];
for(var ti=0;ti<25;ti++){
  var td=sph(0.015,new THREE.MeshStandardMaterial({color:0x00f5c8,transparent:true,opacity:0}));
  scene.add(td); trail.push({mesh:td,x:0,y:0,age:0});
}
var trailHead=0;

// ═══════════════════════════════════════
// TERRAIN SCENARIOS
// ═══════════════════════════════════════
var scenarios = {
  flat:  { name:'FLAT GROUND',   info:'All 6 wheels on ground.\nRocker/bogie nearly neutral.\nDiff bar stays horizontal.',
    fn:function(x){ return 0; }, col:0x0f2018 },
  rock:  { name:'ROCK OBSTACLE', info:'Front wheel contacts rock.\nRocker tips up at front.\nChassis tilts half the angle.',
    fn:function(x){ var d=x-1.2; return d>-0.12&&d<0.12? 0.10*(1-Math.pow(d/0.12,2)):0; }, col:0x201a10 },
  rocks: { name:'ROCKY FIELD',   info:'Multiple obstacles.\nPassive articulation adapts.\nBody motion halved by design.',
    fn:function(x){
      var h=0;
      [[0.7,0.10,0.09],[1.4,0.12,0.10],[2.1,0.09,0.07],[2.8,0.13,0.11],[3.5,0.10,0.08]].forEach(function(r){
        var d=x-r[0]; if(Math.abs(d)<r[1]) h=Math.max(h,r[2]*(1-Math.pow(d/r[1],2)));
      }); return h; }, col:0x201a10 },
  slope: { name:'SLOPE ASCENT 25°', info:'Climbing 25° incline.\nLoad shifts rearward.\nChassis tilt ≈ slope angle.',
    fn:function(x){ return x>0.8?Math.min((x-0.8)*Math.tan(25*Math.PI/180),1.5):0; }, col:0x101828 },
  ditch: { name:'DITCH CROSSING',   info:'200mm ditch width.\nBogie dips, rocker lifts.\nAll wheels stay grounded.',
    fn:function(x){ var d=x-1.5; return Math.abs(d)<0.22?-0.18*(1-Math.pow(d/0.22,2)):0; }, col:0x101820 },
  step:  { name:'STEP CLIMB',       info:'100mm step height.\nFront wheel drives into step.\nBogie follows smoothly.',
    fn:function(x){ return x>1.0?0.10:0; }, col:0x181820 },
  sand:  { name:'SAND DUNES',       info:'Undulating soft terrain.\nContinuous passive adaptation.\nWide wheels resist sinkage.',
    fn:function(x){ return 0.06*Math.sin(x*2.2)+0.04*Math.sin(x*3.8+1.0)+0.025*Math.cos(x*1.3+0.4); }, col:0x282010 }
};

var currentSc = 'flat';

function buildTerrain(key){
  if(terrMesh) scene.remove(terrMesh);
  var sc=scenarios[key];
  var N=200, xMin=-1.0, xMax=5.5, tw=1.5;
  var verts=[], idx=[];
  for(var i=0;i<N;i++){
    var x=xMin+(xMax-xMin)*(i/(N-1));
    var y=sc.fn(x);
    verts.push(x,y-0.004,-tw/2);
    verts.push(x,y-0.004, tw/2);
  }
  for(var i=0;i<N-1;i++){
    var a=i*2,b=i*2+1,c=(i+1)*2,d=(i+1)*2+1;
    idx.push(a,b,c); idx.push(b,d,c);
  }
  var geo=new THREE.BufferGeometry();
  geo.setAttribute('position',new THREE.Float32BufferAttribute(verts,3));
  geo.setIndex(idx); geo.computeVertexNormals();
  terrMesh=new THREE.Mesh(geo,new THREE.MeshStandardMaterial({color:sc.col,roughness:0.95,metalness:0,side:THREE.DoubleSide}));
  terrMesh.receiveShadow=true; scene.add(terrMesh);
}
buildTerrain('flat');

window.setSc=function(key){
  currentSc=key; simT=0;
  document.querySelectorAll('.sc-btn').forEach(function(b){b.classList.remove('active');});
  document.getElementById('sc-'+key).classList.add('active');
  document.getElementById('bb-sc').textContent=scenarios[key].name;
  document.getElementById('info-txt').textContent=scenarios[key].info;
  buildTerrain(key);
};

// ═══════════════════════════════════════
// SIMULATION STATE
// ═══════════════════════════════════════
var playing=false, simT=0, simSpd=1.0;
var xStart=-0.8, xEnd=4.8;

window.togglePlay=function(){
  playing=!playing;
  document.getElementById('play-btn').textContent=playing?'⏸ PAUSE':'▶ PLAY';
};
window.setView=function(v){
  currentView=v;
  document.querySelectorAll('.vb').forEach(function(b){b.classList.remove('active');});
  document.getElementById('vb-'+v).classList.add('active');
  var p=viewPresets[v];
  if(p&&v!=='follow'){ orb.theta=p.theta;orb.phi=p.phi;orb.r=p.r; orb.tgt.set(0.5,0.35,0); }
};

function getTH(x){ return scenarios[currentSc].fn(x); }

// Wheel X offsets relative to rover center X
var FWX_OFF = RP_X + RF*Math.cos(R_ANG);  //  ~0.275
var MWX_OFF = RP_X - RR + BH;             //  ~0.020
var RWX_OFF = RP_X - RR - BH;             // ~-0.260

var lastT=null;

function simStep(dt){
  if(!playing) return;
  simSpd=+document.getElementById('spd').value;
  simT+=dt/10*simSpd;
  if(simT>1) simT=0;

  var rx = xStart+(xEnd-xStart)*simT;  // rover world X

  // Terrain Y at each wheel (world coords)
  var yFL = Math.max(WR, getTH(rx+FWX_OFF)+WR);
  var yML = Math.max(WR, getTH(rx+MWX_OFF)+WR);
  var yRL = Math.max(WR, getTH(rx+RWX_OFF)+WR);
  var yFR = Math.max(WR, getTH(rx+FWX_OFF)+WR);
  var yMR = Math.max(WR, getTH(rx+MWX_OFF)+WR);
  var yRR = Math.max(WR, getTH(rx+RWX_OFF)+WR);

  // Chassis height = average of all 6 wheel centers + ground clearance + half chassis
  var avgY=(yFL+yML+yRL+yFR+yMR+yRR)/6;
  var chassMidW = avgY + GC + CHH/2;

  // Chassis pitch from front vs rear
  var pitchAng = Math.atan2((yFL+yFR)/2 - (yRL+yRR)/2, FWX_OFF-RWX_OFF)*0.45;
  // Chassis roll from L vs R
  var rollAng  = Math.atan2((yFL+yML+yRL)/3-(yFR+yMR+yRR)/3, CHW)*0.5;

  // Move entire rover group
  roverGrp.position.set(rx, 0, 0);

  // Chassis sub-group offset
  chassGrp.position.set(0, chassMidW, 0);
  chassGrp.rotation.set(rollAng, 0, pitchAng);

  // Differential bar: sits on top of chassis, rocks opposite to roll
  diffGrp.position.set(0.04, chassMidW + DIFF_Y_LOCAL, 0);
  diffGrp.rotation.z = -rollAng * 1.1;
  diffEndL.position.z =  DIFF_SPAN;
  diffEndR.position.z = -DIFF_SPAN;

  // Diff links from bar ends to rocker pivots
  // Left side: bar end at (diffGrp world X, diffGrp world Y, diffGrp world Z+SPAN)
  var dlY=chassMidW+DIFF_Y_LOCAL;
  orientBeam(diffLinkL, rx+0.04,dlY, RP_Z_ABS*0.78, rx+0.04,chassMidW, RP_Z_ABS);
  orientBeam(diffLinkR, rx+0.04,dlY,-RP_Z_ABS*0.78, rx+0.04,chassMidW,-RP_Z_ABS);

  // Feed wheel heights into sides (relative to roverGrp which is at rx,0,0)
  sideL.yF = yFL;
  sideL.yM = yML;
  sideL.yR = yRL;
  sideR.yF = yFR;
  sideR.yM = yMR;
  sideR.yR = yRR;

  updateSide(sideL, chassMidW);
  updateSide(sideR, chassMidW);

  // Trail
  var trailY = avgY+0.02;
  trail[trailHead].mesh.position.set(rx, trailY, 0);
  trail[trailHead].age=0; trailHead=(trailHead+1)%trail.length;
  trail.forEach(function(t){
    t.age+=dt;
    t.mesh.material.opacity=Math.max(0,0.65-t.age*0.4);
  });

  // Follow cam
  if(currentView==='follow') orb.tgt.set(rx,chassMidW,0);

  // Telemetry
  var pDeg=(pitchAng*180/Math.PI).toFixed(1);
  var rDeg=(rollAng*180/Math.PI).toFixed(1);
  var rlDeg=(sideL.rockerAng*180/Math.PI).toFixed(1);
  var rrDeg=(sideR.rockerAng*180/Math.PI).toFixed(1);
  var blDeg=(sideL.bogieAng*180/Math.PI).toFixed(1);
  var brDeg=(sideR.bogieAng*180/Math.PI).toFixed(1);

  function tv(id,v,w,c){
    var el=document.getElementById(id); el.textContent=v;
    el.className='tele-val'+(c?' crit':w?' warn':'');
  }
  tv('tv-pitch',pDeg+'°', Math.abs(pitchAng)>0.18, Math.abs(pitchAng)>0.3);
  tv('tv-roll', rDeg+'°', Math.abs(rollAng)>0.15,  Math.abs(rollAng)>0.28);
  tv('tv-rl',  rlDeg+'°',false,false);
  tv('tv-rr',  rrDeg+'°',false,false);
  tv('tv-bl',  blDeg+'°',false,false);
  tv('tv-br',  brDeg+'°',false,false);

  document.getElementById('bb-prog').textContent=Math.round(simT*100)+'%';
  document.getElementById('bb-time').textContent=(simT*10/simSpd).toFixed(1)+'s';
}

// ═══════════════════════════════════════
// CAMERA ORBIT
// ═══════════════════════════════════════
var currentView='iso';
var orb={ theta:0.72, phi:1.02, r:3.8, tgt:new THREE.Vector3(0.5,0.35,0) };
var viewPresets={
  iso:    {theta:0.72,  phi:1.02, r:3.8},
  side:   {theta:Math.PI/2, phi:1.1, r:3.5},
  front:  {theta:0,    phi:1.1,  r:3.4},
  top:    {theta:0.72, phi:0.12, r:5.0},
  follow: {theta:-0.3, phi:0.95, r:3.2}
};

var isDrg=false, isRDrg=false, pmx=0,pmy=0;
canvas.addEventListener('mousedown',function(e){isDrg=true;isRDrg=e.button===2;pmx=e.clientX;pmy=e.clientY;});
canvas.addEventListener('contextmenu',function(e){e.preventDefault();});
window.addEventListener('mouseup',function(){isDrg=false;});
window.addEventListener('mousemove',function(e){
  if(!isDrg)return;
  var dx=e.clientX-pmx, dy=e.clientY-pmy;
  if(isRDrg){orb.tgt.x-=dx*0.003;orb.tgt.y+=dy*0.003;}
  else{orb.theta-=dx*0.009; orb.phi=Math.max(0.06,Math.min(1.52,orb.phi+dy*0.009));}
  pmx=e.clientX;pmy=e.clientY;
});
canvas.addEventListener('wheel',function(e){
  orb.r=Math.max(1.0,Math.min(10,orb.r+e.deltaY*0.004)); e.preventDefault();
},{passive:false});
var ltch=null;
canvas.addEventListener('touchstart',function(e){ltch=e.touches[0];},{passive:true});
canvas.addEventListener('touchmove',function(e){
  if(!ltch)return; e.preventDefault();
  var t=e.touches[0];
  orb.theta-=(t.clientX-ltch.clientX)*0.012;
  orb.phi=Math.max(0.06,Math.min(1.52,orb.phi+(t.clientY-ltch.clientY)*0.012));
  ltch=t;
},{passive:false});

// ═══════════════════════════════════════
// RENDER LOOP
// ═══════════════════════════════════════
function loop(ts){
  requestAnimationFrame(loop);
  var dt=lastT?(Math.min(ts-lastT,80))/1000:0.016; lastT=ts;
  simStep(dt);
  camera.position.set(
    orb.tgt.x + orb.r*Math.sin(orb.phi)*Math.sin(orb.theta),
    orb.tgt.y + orb.r*Math.cos(orb.phi),
    orb.tgt.z + orb.r*Math.sin(orb.phi)*Math.cos(orb.theta)
  );
  camera.lookAt(orb.tgt);
  renderer.render(scene,camera);
}
loop(0);

// Auto-start on rock scenario
setTimeout(function(){
  setSc('rock');
  playing=true;
  document.getElementById('play-btn').textContent='⏸ PAUSE';
},500);
</script>
</body>
</html>
