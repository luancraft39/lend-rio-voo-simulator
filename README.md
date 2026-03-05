<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Simulador 3D Mouse + Touch</title>
<style>
body { margin:0; overflow:hidden; background:black; font-family:Arial; touch-action:none; }

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
</style>
</head>
<body>

<div id="menu">
<h1>SIMULADOR 3D</h1>
<button onclick="startGame()">INICIAR</button>
</div>

<div id="hud">
SPD: <span id="spd">0</span>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>

<script>
let scene, camera, renderer;
let plane;
let speed=0, yaw=0, pitch=0, roll=0;
let camYaw=0, camPitch=0;
let keys={}, terrainSize=600, chunks={};
let music;

let isMobile = 'ontouchstart' in window;
let lastX=0, lastY=0;
let isDragging=false;

function startGame(){
 document.getElementById("menu").style.display="none";
 document.getElementById("hud").style.display="block";

 music = new Audio("https://files.catbox.moe/7kdya6.mp3");
 music.loop = true;
 music.volume = 0.6;
 music.play().catch(()=>{});

 init();
}

function init(){
 scene=new THREE.Scene();
 scene.background=new THREE.Color(0x87CEEB);

 camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,50000);

 renderer=new THREE.WebGLRenderer({antialias:true});
 renderer.setSize(innerWidth,innerHeight);
 document.body.appendChild(renderer.domElement);

 scene.add(new THREE.AmbientLight(0xffffff,0.6));
 let sun=new THREE.DirectionalLight(0xffffff,1);
 sun.position.set(500,800,200);
 scene.add(sun);

 createPlane();

 document.addEventListener("keydown",e=>keys[e.key]=true);
 document.addEventListener("keyup",e=>keys[e.key]=false);

 // 🖱 PC - só gira segurando
 document.addEventListener("mousedown", e=>{
   if(!isMobile){
     isDragging=true;
   }
 });

 document.addEventListener("mouseup", e=>{
   isDragging=false;
 });

 document.addEventListener("mousemove", e=>{
   if(!isMobile && isDragging){
     camYaw -= e.movementX * 0.002;
     camPitch -= e.movementY * 0.002;
   }
 });

 // 📱 Mobile Touch (arrastar normal)
 document.addEventListener("touchstart", e=>{
   lastX = e.touches[0].clientX;
   lastY = e.touches[0].clientY;
 });

 document.addEventListener("touchmove", e=>{
   let dx = e.touches[0].clientX - lastX;
   let dy = e.touches[0].clientY - lastY;
   camYaw -= dx * 0.005;
   camPitch -= dy * 0.005;
   lastX = e.touches[0].clientX;
   lastY = e.touches[0].clientY;
 });

 animate();
}

function createPlane(){
 plane = new THREE.Group();

 let body = new THREE.Mesh(
  new THREE.CylinderGeometry(5,5,60,12),
  new THREE.MeshLambertMaterial({color:0xffffff})
 );
 body.rotation.z=Math.PI/2;
 plane.add(body);

 let wing = new THREE.Mesh(
  new THREE.BoxGeometry(80,2,20),
  new THREE.MeshLambertMaterial({color:0xff0000})
 );
 plane.add(wing);

 plane.position.set(0,300,0);
 scene.add(plane);
}

function getHeight(x,z){
 return Math.sin(x*0.003)*150 + Math.cos(z*0.003)*150;
}

function createChunk(cx,cz){
 let key=cx+","+cz;
 if(chunks[key]) return;

 let geo=new THREE.PlaneGeometry(terrainSize,terrainSize,80,80);
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

 chunks[key]=true;
}

function animate(){
 requestAnimationFrame(animate);

 if(keys["w"]) speed+=0.1;
 if(keys["s"]) speed-=0.1;
 if(keys["ArrowLeft"]) yaw+=0.02;
 if(keys["ArrowRight"]) yaw-=0.02;

 plane.rotation.set(pitch,yaw,roll);
 plane.translateX(speed);

 let camOffset = new THREE.Vector3(0,40,150);
 camOffset.applyAxisAngle(new THREE.Vector3(0,1,0), camYaw);
 camOffset.applyAxisAngle(new THREE.Vector3(1,0,0), camPitch);

 let desiredPos = plane.position.clone().add(camOffset);

 camera.position.lerp(desiredPos,0.1);
 camera.lookAt(plane.position);

 let cx=Math.floor(plane.position.x/terrainSize);
 let cz=Math.floor(plane.position.z/terrainSize);

 for(let x=-2;x<=2;x++){
  for(let z=-2;z<=2;z++){
   createChunk(cx+x,cz+z);
  }
 }

 document.getElementById("spd").innerText=speed.toFixed(1);

 renderer.render(scene,camera);
}
</script>

</body>
</html>
