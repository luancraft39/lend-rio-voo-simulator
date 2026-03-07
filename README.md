<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Legendary Flight Simulator</title>

<style>
body{
margin:0;
overflow:hidden;
background:black;
font-family:Arial;
touch-action:none;
}

#menu{
position:absolute;
width:100%;
height:100%;
background:black;
color:white;
display:flex;
flex-direction:column;
justify-content:center;
align-items:center;
text-align:center;
}

#menu h1{
font-size:50px;
margin:10px;
}

button{
padding:15px 30px;
font-size:20px;
cursor:pointer;
margin-top:20px;
}

#hud{
position:absolute;
top:10px;
left:10px;
color:#00ff00;
font-family:monospace;
display:none;
}

#mobileControls{
position:absolute;
bottom:20px;
left:20px;
display:none;
}

.mobileBtn{
width:60px;
height:60px;
font-size:25px;
margin:5px;
}
</style>
</head>

<body>

<div id="menu">
<h1>Legendary Flight Simulator</h1>
<p>Criador: Lendário Studio</p>
<p>Versão 2.0</p>
<button onclick="startGame()">INICIAR</button>
</div>

<div id="hud">
SPD: <span id="spd">0</span>
</div>

<div id="mobileControls">
<div>
<button class="mobileBtn" ontouchstart="keys['w']=true" ontouchend="keys['w']=false">?</button>
</div>

<div>
<button class="mobileBtn" ontouchstart="keys['a']=true" ontouchend="keys['a']=false">?</button>
<button class="mobileBtn" ontouchstart="keys['s']=true" ontouchend="keys['s']=false">?</button>
<button class="mobileBtn" ontouchstart="keys['d']=true" ontouchend="keys['d']=false">?</button>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>

<script>

let scene,camera,renderer;
let plane;
let yaw=0;
let speed=0;
let normalSpeed=50;
let turboSpeed=120;

let camYaw=0;
let camPitch=0;
let camDistance=150;

let keys={};
let terrainSize=600;

let chunks={};
let villages={};
let npcs=[];
let cars=[];

let isMobile='ontouchstart' in window;
let dragging=false;

let music;

function startGame(){

document.getElementById("menu").style.display="none";
document.getElementById("hud").style.display="block";

/* MUSICA */
music = new Audio("https://files.catbox.moe/7kdya6.mp3");
music.loop = true;
music.volume = 0.6;
music.play().catch(()=>{});

if(isMobile)
document.getElementById("mobileControls").style.display="block";

init();
}

function init(){

scene=new THREE.Scene();
scene.background=new THREE.Color(0x87CEEB);

camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,50000);

renderer=new THREE.WebGLRenderer({
antialias:false,
powerPreference:"high-performance"
});

renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(1);

document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff,0.6));

let sun=new THREE.DirectionalLight(0xffffff,1);
sun.position.set(500,800,200);
scene.add(sun);

createPlane();

document.addEventListener("keydown",e=>keys[e.key.toLowerCase()]=true);
document.addEventListener("keyup",e=>keys[e.key.toLowerCase()]=false);

document.addEventListener("mousedown",()=>dragging=true);
document.addEventListener("mouseup",()=>dragging=false);

document.addEventListener("mousemove",e=>{
if(dragging){
camYaw-=e.movementX*0.003;
camPitch-=e.movementY*0.003;
}
});

document.addEventListener("wheel",e=>{
camDistance+=e.deltaY*0.2;
camDistance=Math.max(30,Math.min(400,camDistance));
});

animate();
}

function createPlane(){

plane=new THREE.Group();

let body=new THREE.Mesh(
new THREE.CylinderGeometry(5,5,60,12),
new THREE.MeshLambertMaterial({color:0xffffff})
);

body.rotation.z=Math.PI/2;
plane.add(body);

let wing=new THREE.Mesh(
new THREE.BoxGeometry(80,2,20),
new THREE.MeshLambertMaterial({color:0xff0000})
);

plane.add(wing);

plane.position.set(0,300,0);

scene.add(plane);
}

function getHeight(x,z){

let base=Math.sin(x*0.003)*150+Math.cos(z*0.003)*150;
let river=Math.sin(z*0.002)*50;

if(Math.abs(x-river)<30)
return -20;

return base;
}

function createWater(cx,cz){

let geo=new THREE.PlaneGeometry(terrainSize,terrainSize);
geo.rotateX(-Math.PI/2);

let mat=new THREE.MeshLambertMaterial({
color:0x3399ff,
transparent:true,
opacity:0.8
});

let water=new THREE.Mesh(geo,mat);

water.position.set(cx*terrainSize,-19,cz*terrainSize);

scene.add(water);
}

function createChunk(cx,cz){

let key=cx+","+cz;
if(chunks[key])return;

let geo=new THREE.PlaneGeometry(terrainSize,terrainSize,32,32);
geo.rotateX(-Math.PI/2);

for(let i=0;i<geo.attributes.position.count;i++){

let x=geo.attributes.position.getX(i)+cx*terrainSize;
let z=geo.attributes.position.getZ(i)+cz*terrainSize;

geo.attributes.position.setY(i,getHeight(x,z));
}

geo.computeVertexNormals();

let mat=new THREE.MeshLambertMaterial({color:0x22aa22});

let mesh=new THREE.Mesh(geo,mat);

mesh.position.set(cx*terrainSize,0,cz*terrainSize);

scene.add(mesh);

createWater(cx,cz);

chunks[key]=true;
}

function animate(){

requestAnimationFrame(animate);

if(keys["w"]) speed=normalSpeed;
else speed=0;

if(keys["arrowup"]) speed=turboSpeed;

if(keys["a"]) yaw+=0.03;
if(keys["d"]) yaw-=0.03;

if(keys["e"]) plane.position.y-=3;
if(keys["q"]) plane.position.y+=3;

plane.rotation.y=yaw;
plane.translateX(speed*0.1);

let offset=new THREE.Vector3(0,40,camDistance);

offset.applyAxisAngle(new THREE.Vector3(0,1,0),camYaw);
offset.applyAxisAngle(new THREE.Vector3(1,0,0),camPitch);

let pos=plane.position.clone().add(offset);

camera.position.lerp(pos,0.1);
camera.lookAt(plane.position);

let cx=Math.floor(plane.position.x/terrainSize);
let cz=Math.floor(plane.position.z/terrainSize);

for(let x=-1;x<=1;x++)
for(let z=-1;z<=1;z++)
createChunk(cx+x,cz+z);

document.getElementById("spd").innerText=speed;

renderer.render(scene,camera);
}

</script>
</body>
</html>
