# Minecraft-like
Un jeux comme Minecraft 
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Mini Minecraft Web</title>
<style>
body{margin:0;overflow:hidden;}
canvas{display:block;}
#ui{position:absolute;bottom:10px;left:50%;transform:translateX(-50%);display:flex;gap:10px;}
.button{width:60px;height:60px;background:rgba(0,0,0,0.5);color:white;text-align:center;line-height:60px;font-weight:bold;border-radius:10px;user-select:none;}
#hearts{position:absolute;top:10px;left:10px;font-size:24px;color:red;font-weight:bold;user-select:none;}
</style>
</head>
<body>
<div id="hearts">❤❤❤❤❤</div>
<div id="ui">
  <div class="button" id="up">↑</div>
  <div class="button" id="left">←</div>
  <div class="button" id="down">↓</div>
  <div class="button" id="right">→</div>
  <div class="button" id="jump">⬆︎</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/examples/js/controls/PointerLockControls.js"></script>

<script>
// --- Scène et caméra ---
const scene=new THREE.Scene(); scene.background=new THREE.Color(0x87ceeb);
const camera=new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000); camera.position.set(0,2,10);
const renderer=new THREE.WebGLRenderer(); renderer.setSize(window.innerWidth,window.innerHeight); document.body.appendChild(renderer.domElement);

const light=new THREE.DirectionalLight(0xffffff,1); light.position.set(10,20,10); scene.add(light); scene.add(new THREE.AmbientLight(0x404040));
const controls=new THREE.PointerLockControls(camera,renderer.domElement); 
document.body.addEventListener('click',()=>controls.lock());

// --- Blocs et terrain ---
const BLOCK_SIZE=1;
const materials={
  grass:new THREE.MeshStandardMaterial({color:0x228B22}),
  stone:new THREE.MeshStandardMaterial({color:0x888888}),
  creeper:new THREE.MeshStandardMaterial({color:0x00FF00}),
  zombie:new THREE.MeshStandardMaterial({color:0x00AA00}),
  skeleton:new THREE.MeshStandardMaterial({color:0xAAAAAA}),
  spider:new THREE.MeshStandardMaterial({color:0x333333}),
  witch:new THREE.MeshStandardMaterial({color:0x800080}),
  golem:new THREE.MeshStandardMaterial({color:0xCCCCCC}),
  dragon:new THREE.MeshStandardMaterial({color:0x8B0000})
};
const blocks=[]; function addBlock(x,y,z,type='grass'){const c=new THREE.Mesh(new THREE.BoxGeometry(BLOCK_SIZE,BLOCK_SIZE,BLOCK_SIZE),materials[type]); c.position.set(x,y,z); scene.add(c); blocks.push(c); return c;}
// Terrain
for(let x=-10;x<10;x++){for(let z=-10;z<10;z++){addBlock(x,0,z,'grass'); addBlock(x,-1,z,'stone');}}

// --- Joueur ---
let hearts=5; const heartsDiv=document.getElementById('hearts'); function updateHearts(){heartsDiv.textContent='❤'.repeat(Math.floor(hearts));}

// --- Déplacement mobile/PC ---
const move={forward:false,backward:false,left:false,right:false,jump:false};
document.getElementById('up').addEventListener('touchstart',()=>move.forward=true); document.getElementById('up').addEventListener('touchend',()=>move.forward=false);
document.getElementById('down').addEventListener('touchstart',()=>move.backward=true); document.getElementById('down').addEventListener('touchend',()=>move.backward=false);
document.getElementById('left').addEventListener('touchstart',()=>move.left=true); document.getElementById('left').addEventListener('touchend',()=>move.left=false);
document.getElementById('right').addEventListener('touchstart',()=>move.right=true); document.getElementById('right').addEventListener('touchend',()=>move.right=false);
document.getElementById('jump').addEventListener('touchstart',()=>{if(onGround){velocityY=0.2; onGround=false;}});
// Clavier PC
document.addEventListener('keydown',e=>{if(e.code==='KeyW') move.forward=true;if(e.code==='KeyS') move.backward=true;if(e.code==='KeyA') move.left=true;if(e.code==='KeyD') move.right=true;if(e.code==='Space' && onGround){velocityY=0.2;onGround=false;}});
document.addEventListener('keyup',e=>{if(e.code==='KeyW') move.forward=false;if(e.code==='KeyS') move.backward=false;if(e.code==='KeyA') move.left=false;if(e.code==='KeyD') move.right=false;});

// --- Gravité ---
let velocityY=0; let onGround=false;

// --- Mobs ---
const mobs=[];
function spawnMob(type,x,z){const m=new THREE.Mesh(new THREE.BoxGeometry(0.8,1.8,0.8),materials[type]); m.position.set(x,1,z); m.type=type; scene.add(m); mobs.push(m); return m;}
spawnMob('creeper',0,-5); spawnMob('zombie',3,-7); spawnMob('skeleton',-3,-7); spawnMob('spider',-5,-5);
const witches=[]; for(let i=0;i<2;i++){ const w=spawnMob('witch',Math.random()*10-5,Math.random()*10-5); witches.push({mesh:w,cooldown:0}); }
const golem=spawnMob('golem',0,5);
const dragon=new THREE.Mesh(new THREE.BoxGeometry(3,2,6), materials.dragon); dragon.position.set(0,5,-10); scene.add(dragon);
const dragonFireballs=[];

// --- Projectiles sorcières ---
const projectiles=[];
function shootProjectile(from,target){
  const p=new THREE.Mesh(new THREE.SphereGeometry(0.2,8,8), new THREE.MeshStandardMaterial({color:0xFF00FF}));
  p.position.copy(from.position);
  p.velocity=new THREE.Vector3((target.x-from.position.x)*0.05,(target.y-from.position.y)*0.05,(target.z-from.position.z)*0.05);
  scene.add(p); projectiles.push(p);
}

// --- Animation ---
function animate(){
  requestAnimationFrame(animate);

  const speed=0.1;
  if(move.forward) controls.moveForward(speed);
  if(move.backward) controls.moveForward(-speed);
  if(move.left) controls.moveRight(-speed);
  if(move.right) controls.moveRight(speed);

  velocityY-=0.01; camera.position.y+=velocityY;
  if(camera.position.y<=2){camera.position.y=2; velocityY=0; onGround=true;}

  // IA mobs
  mobs.forEach(m=>{
    if(m.type==='golem') return;
    const dx=camera.position.x-m.position.x; const dz=camera.position.z-m.position.z;
    const dist=Math.sqrt(dx*dx+dz*dz);
    if(dist<10){ m.position.x+=dx*0.01; m.position.z+=dz*0.01; if(dist<1.5 && m.type!=='witch'){hearts-=0.02;if(hearts<0)hearts=0;updateHearts();}}
  });

  // Sorcières
  witches.forEach(w=>{
    w.cooldown--; const dx=camera.position.x-w.mesh.position.x; const dz=camera.position.z-w.mesh.position.z;
    const dist=Math.sqrt(dx*dx+dz*dz);
    if(dist<8 && w.cooldown<=0){ shootProjectile(w.mesh,camera.position); w.cooldown=100; }
  });

  projectiles.forEach((p,i)=>{ p.position.add(p.velocity); const d=camera.position.distanceTo(p.position); if(d<1){hearts-=0.05;if(hearts<0)hearts=0;updateHearts(); scene.remove(p); projectiles.splice(i,1);}});

  // Golem repousse mobs
  mobs.forEach(m=>{if(['creeper','zombie','skeleton','spider','witch'].includes(m.type)){const dx=golem.position.x-m.position.x;const dz=golem.position.z-m.position.z;const dist=Math.sqrt(dx*dx+dz*dz);if(dist<3){m.position.x-=dx*0.1;m.position.z-=dz*0.1;}}});

  // Dragon suit joueur et crache feu
  const dx=camera.position.x-dragon.position.x; const dy=camera.position.y-dragon.position.y; const dz=camera.position.z-dragon.position.z;
  const dist=Math.sqrt(dx*dx+dy*dy+dz*dz);
  if(dist<20){dragon.position.x+=dx*0.01; dragon.position.y+=dy*0.005; dragon.position.z+=dz*0.01; if(Math.random()<0.01){const fire=new THREE.Mesh(new THREE.SphereGeometry(0.3,8,8),new THREE.MeshStandardMaterial({color:0xFF4500})); fire.position.copy(dragon.position); fire.velocity=new THREE.Vector3(dx*0.05,dy*0.05,dz*0.05); scene.add(fire); dragonFireballs.push(fire);}}
  dragonFireballs.forEach((f,i)=>{f.position.add(f.velocity); const d=camera.position.distanceTo(f.position); if(d<1){hearts-=0.1;if(hearts<0)hearts=0;updateHearts(); scene.remove(f); dragonFireballs.splice(i,1);}});

  // Mort joueur
  if(hearts<=0){alert("Vous êtes mort !"); location.reload();}

  renderer.render(scene,camera);
}
animate();

// Resize
window.addEventListener('resize',()=>{camera.aspect=window.innerWidth/window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth,window.innerHeight);});
</script>
</body>
</html>
