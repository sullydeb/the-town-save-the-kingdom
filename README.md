# the-town-save-the-kingdom

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The Town: Save the Kingdom 3D</title>
<style>
body{margin:0;overflow:hidden;background:#83d9ff;font-family:Arial;color:white}
#ui{position:absolute;top:14px;left:14px;background:#17234add;padding:14px 18px;border-radius:14px;line-height:1.6;z-index:2}
#help{position:absolute;bottom:14px;left:14px;background:#17234add;padding:12px;border-radius:12px;z-index:2}
#select{position:absolute;inset:0;background:#17234a;display:flex;align-items:center;justify-content:center;z-index:5}
#panel{background:#283d69;padding:30px;border-radius:20px;text-align:center;max-width:600px}
button{border:0;border-radius:10px;padding:12px;margin:6px;background:#ffd34e;color:#382b13;font-weight:bold;cursor:pointer}
button:hover{background:#fff0a6}
#status{color:#ffe27a;font-weight:bold}
</style>
</head>
<body>
<div id="ui">
 <b>👑 THE TOWN: SAVE THE KINGDOM</b><br>
 Hero: <span id="hero">Dabully (Deb)</span><br>
 Coins: <span id="coins">0</span><br>
 Location: Royal Capital<br>
 <span id="status">Explore the kingdom!</span>
</div>
<div id="help">
 WASD: Move · Mouse: Camera · Space: Jump · Shift: Sprint<br>
 1–4: Change hero · E: Collect nearby coins
</div>
<div id="select">
 <div id="panel">
  <h1>👑 THE TOWN 👑</h1>
  <h2>Save the Kingdom 3D</h2>
  <p>Choose your hero and begin your adventure!</p>
  <button onclick="start(0)">🍀 Dabully (Deb)</button>
  <button onclick="start(1)">🌌 CG (Colbster)</button>
  <button onclick="start(2)">🤠 Texas (Tex)</button>
  <button onclick="start(3)">👞 The Shoe Shiner</button>
  <p>A colorful fantasy adventure begins...</p>
 </div>
</div>
<script type="importmap">
{"imports":{"three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js"}}
</script>
<script type="module">
import * as THREE from "three";

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x83d9ff);
scene.fog=new THREE.Fog(0x83d9ff,65,180);

const camera=new THREE.PerspectiveCamera(65,innerWidth/innerHeight,.1,500);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.outputColorSpace=THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x54704b,2));
const sun=new THREE.DirectionalLight(0xfff0c2,3);
sun.position.set(-35,65,25);
sun.castShadow=true;
scene.add(sun);

function mat(color,roughness=.8){return new THREE.MeshStandardMaterial({color,roughness})}
function block(x,y,z,sx,sy,sz,color){
 const m=new THREE.Mesh(new THREE.BoxGeometry(sx,sy,sz),mat(color));
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;scene.add(m);return m;
}
function ball(x,y,z,r,color){
 const m=new THREE.Mesh(new THREE.SphereGeometry(r,16,12),mat(color));
 m.position.set(x,y,z);m.castShadow=true;scene.add(m);return m;
}

// Kingdom ground
block(0,-.6,0,220,1,220,0x55b94f);
block(0,-.05,0,12,.12,210,0xb9a078);
block(0,.02,0,210,.12,12,0xb9a078);

// Palace
block(0,5,-37,27,10,19,0xf0d38c);
block(0,12,-37,9,5,9,0xf8e3a8);
for(let x of [-10,-5,5,10]){
 block(x,8,-27,2,16,2,0xe6c06b);
 const roof=new THREE.Mesh(new THREE.ConeGeometry(2,4,4),mat(0xc84b39));
 roof.position.set(x,18,-27);roof.rotation.y=Math.PI/4;scene.add(roof);
}
const palaceRoof=new THREE.Mesh(new THREE.ConeGeometry(8,7,4),mat(0xc84b39));
palaceRoof.position.set(0,17,-37);palaceRoof.rotation.y=Math.PI/4;scene.add(palaceRoof);

// Buildings
for(let i=0;i<18;i++){
 const x=(i%6-2.5)*23;
 const z=10+Math.floor(i/6)*28;
 const colors=[0xffc66e,0xe88e85,0x8eb6e9,0xd5a9df,0xe8db9b];
 block(x,3,z,12,7,11,colors[i%colors.length]);
 block(x,7.5,z,13,2,12,0x8d4939);
 block(x,1.7,z-5.55,2.5,3.5, .3,0x69452c);
 for(let wx of [-3,3])block(x+wx,4,z-5.58,1.8,1.8,.25,0x91dfff);
}

// Trees
for(let i=0;i<90;i++){
 const x=(Math.random()-.5)*190,z=(Math.random()-.5)*190;
 if(Math.abs(x)<17&&Math.abs(z)<45)continue;
 block(x,.8,z,.7,2,.7,0x79512e);
 ball(x,2.7,z,2.2,[0x3a9c4c,0x57b94b,0x328a61][i%3]);
}

// Distant hills
for(let i=0;i<20;i++){
 const h=12+Math.random()*22;
 const hill=new THREE.Mesh(new THREE.ConeGeometry(12+Math.random()*12,h,7),mat(i%2?0x62a96b:0x83b66c));
 hill.position.set((Math.random()-.5)*190,h/2-1,-95-Math.random()*30);
 scene.add(hill);
}

// Player
const heroColors=[0x22a85c,0x6652d8,0x9b6032,0x6d6b61];
let heroIndex=0,player,coins=0;
const heroNames=["Dabully (Deb)","CG (Colbster)","Texas (Tex)","The Shoe Shiner"];
const keys={};
let yaw=0,pitch=.15,velocityY=0,onGround=true;
const clock=new THREE.Clock();

function makePlayer(){
 if(player)scene.remove(player);
 player=new THREE.Group();
 const body=new THREE.Mesh(new THREE.CapsuleGeometry(.65,.9,5,10),mat(heroColors[heroIndex]));
 body.position.y=1.25;body.castShadow=true;player.add(body);
 const head=ball(0,2.45,0,.52,0xf0bc91);player.add(head);
 if(heroIndex===0){
   // Curly red hair and magical flames
   for(let i=0;i<13;i++){
    const a=i*Math.PI*2/13;
    const curl=ball(Math.cos(a)*.36,2.75+Math.random()*.25,Math.sin(a)*.3,.22,0xb94727);
    player.add(curl);
   }
   const flame=ball(0,3.15,0,.35,0xff7427);player.add(flame);
   const inner=ball(0,3.35,0,.17,0xffe45e);player.add(inner);
 }else if(heroIndex===1){
   const hat=block(0,3,0,1.5,.12,1.5,0x17162d);hat.position.y=3;
   const star=ball(0,3.2,0,.13,0xffdc58);player.add(star);
 }else if(heroIndex===2){
   const hat=block(0,2.95,0,1.7,.16,1.1,0x56321f);hat.position.y=2.95;
 }else{
   for(let i=0;i<5;i++)player.add(ball((Math.random()-.5)*1.1,2.4+Math.random()*.3,(Math.random()-.5)*.5,.15,0x55452d));
 }
 player.position.set(0,0,0);scene.add(player);
 document.getElementById("hero").textContent=heroNames[heroIndex];
}
function start(n){heroIndex=n;makePlayer();document.getElementById("select").style.display="none";}

const pickups=[];
for(let i=0;i<75;i++){
 let x=(Math.random()-.5)*170,z=(Math.random()-.5)*170;
 if(Math.abs(x)<8&&Math.abs(z)<10){x+=18;}
 const coin=new THREE.Mesh(new THREE.TorusGeometry(.48,.14,8,20),mat(0xffd52a,.25));
 coin.position.set(x,.9,z);coin.rotation.x=Math.PI/2;
 coin.castShadow=true;scene.add(coin);pickups.push(coin);
}

// Input
addEventListener("keydown",e=>{
 keys[e.code]=true;
 if(e.code.startsWith("Digit")&&document.getElementById("select").style.display==="none"){
  const n=Number(e.code.slice(5))-1;if(n>=0&&n<4){heroIndex=n;makePlayer();}
 }
 if(e.code==="Space"&&onGround&&player){velocityY=9;onGround=false;}
 if(e.code==="KeyE"&&player)collect();
});
addEventListener("keyup",e=>keys[e.code]=false);
let dragging=false,lastX=0,lastY=0;
renderer.domElement.addEventListener("pointerdown",e=>{dragging=true;lastX=e.clientX;lastY=e.clientY;});
addEventListener("pointerup",()=>dragging=false);
addEventListener("pointermove",e=>{
 if(!dragging)return;
 yaw-=(e.clientX-lastX)*.005;
 pitch=Math.max(-.1,Math.min(.75,pitch+(e.clientY-lastY)*.003));
 lastX=e.clientX;lastY=e.clientY;
});

function collect(){
 for(const c of pickups){
  if(c.visible&&c.position.distanceTo(player.position)<3){
   c.visible=false;coins+=5;
   document.getElementById("coins").textContent=coins;
   document.getElementById("status").textContent="+5 coins!";
  }
 }
}
function animate(){
 requestAnimationFrame(animate);
 const dt=Math.min(clock.getDelta(),.04);
 if(player){
  let forward=(keys.KeyW?1:0)-(keys.KeyS?1:0);
  let sideways=(keys.KeyD?1:0)-(keys.KeyA?1:0);
  const speed=keys.ShiftLeft?12:7;
  const move=new THREE.Vector3(sideways,0,-forward);
  if(move.lengthSq()>0){
   move.normalize().applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
   player.position.addScaledVector(move,speed*dt);
   player.rotation.y=Math.atan2(move.x,move.z);
  }
  velocityY-=22*dt;player.position.y+=velocityY*dt;
  if(player.position.y<0){player.position.y=0;velocityY=0;onGround=true;}
  player.position.x=THREE.MathUtils.clamp(player.position.x,-100,100);
  player.position.z=THREE.MathUtils.clamp(player.position.z,-100,100);
  const offset=new THREE.Vector3(Math.sin(yaw)*13,6+pitch*8,Math.cos(yaw)*13);
  camera.position.copy(player.position).add(offset);
  camera.lookAt(player.position.x,player.position.y+1.4,player.position.z);
 }
 for(const c of pickups)if(c.visible){c.rotation.z+=dt*2;c.position.y=.9+Math.sin(clock.elapsedTime*3+c.position.x)*.15;}
 renderer.render(scene,camera);
}
addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
animate();
</script>
</body>
</html>
