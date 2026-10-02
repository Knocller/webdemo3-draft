<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>3D Studio Pro</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;600&display=swap" rel="stylesheet">
<style>
:root{--bg:#0d1020;--panel:rgba(251,252,254,.92);--ink:#1b2433;--muted:#5d6b82;--line:#c9d2df;--accent:#3556f5;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--panel:rgba(24,30,46,.9);--ink:#e8edf6;--muted:#93a1ba;--line:#33405a;--accent:#7b93ff}}
:root[data-theme="dark"]{--panel:rgba(24,30,46,.9);--ink:#e8edf6;--muted:#93a1ba;--line:#33405a;--accent:#7b93ff}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--ink);font-family:"Space Grotesk",system-ui,sans-serif;overflow:hidden;font-size:14px}
#stage{position:fixed;inset:0;touch-action:none;cursor:grab}
canvas{display:block;width:100%;height:100%}
#top{position:fixed;left:12px;top:calc(12px + env(safe-area-inset-top,0px));pointer-events:none;display:flex;flex-direction:column;gap:8px;align-items:flex-start}
#top>*{pointer-events:auto}
h1{font-size:1.4rem;margin:0;font-weight:600;letter-spacing:-.02em;color:#fff;text-shadow:0 2px 12px rgba(0,0,0,.5);pointer-events:none}
#bar{display:flex;flex-wrap:wrap;gap:6px;max-width:min(560px,calc(100vw - 340px))}
#bar button{background:rgba(255,255,255,.14);color:#fff;border-color:rgba(255,255,255,.28);backdrop-filter:blur(8px)}
aside{position:fixed;right:12px;top:calc(12px + env(safe-area-inset-top,0px));bottom:calc(12px + env(safe-area-inset-bottom,0px));width:300px;background:var(--panel);backdrop-filter:blur(14px);border:1px solid var(--line);border-radius:14px;padding:14px;overflow-y:auto;display:flex;flex-direction:column;gap:12px}
.tabs{display:grid;grid-template-columns:1fr 1fr;gap:6px}
button,select{font:inherit;font-size:.85rem;padding:7px 10px;border-radius:8px;border:1px solid var(--line);background:var(--panel);color:var(--ink);cursor:pointer}
button:hover,select:hover{border-color:var(--accent)}
button[aria-pressed="true"]{background:var(--accent);border-color:var(--accent);color:#fff}
button.danger{color:#e0584a}
button:focus-visible,input:focus-visible,select:focus-visible{outline:2px solid var(--accent);outline-offset:2px}
.group{display:grid;gap:9px}
.group h2{font-size:.85rem;margin:4px 0 0;font-weight:600;color:var(--muted)}
.row{display:flex;align-items:center;justify-content:space-between;gap:10px}
.row label{color:var(--muted);flex:0 0 78px}
.row input[type=range]{flex:1;min-width:0;accent-color:var(--accent)}
.row output{flex:0 0 40px;text-align:right;font-variant-numeric:tabular-nums}
.row select{flex:1;min-width:0}
input[type=color]{width:44px;height:28px;border:1px solid var(--line);border-radius:6px;background:none;padding:2px}
input[type=checkbox]{width:18px;height:18px;accent-color:var(--accent)}
#list{display:grid;gap:5px;max-height:130px;overflow-y:auto}
#list button{text-align:left}
.actions{display:flex;gap:6px;flex-wrap:wrap}
.empty{color:var(--muted);margin:0}
.hint{position:fixed;left:12px;bottom:calc(12px + env(safe-area-inset-bottom,0px));color:rgba(255,255,255,.7);margin:0;pointer-events:none;text-shadow:0 1px 6px rgba(0,0,0,.6)}
@media (max-width:720px){aside{top:auto;left:12px;right:12px;width:auto;max-height:46vh}#bar{max-width:100%}.hint{display:none}}
</style>
</head>
<body>
<div id="stage"></div>
<div id="top"><h1>3D Studio Pro</h1><div id="bar" role="toolbar" aria-label="Add shape"></div></div>
<p class="hint">Drag to orbit. Scroll or pinch to zoom. Click a shape to select. Delete removes it.</p>
<aside>
  <div class="tabs"><button id="tObj" aria-pressed="true">Object</button><button id="tScene" aria-pressed="false">Scene</button></div>
  <div id="objTab" class="group"></div>
  <div id="sceneTab" class="group" hidden></div>
</aside>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
(function(){
var $=function(id){return document.getElementById(id)};
var stage=$('stage');
var renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping;renderer.toneMappingExposure=1.1;
renderer.outputEncoding=THREE.sRGBEncoding;
stage.appendChild(renderer.domElement);
var scene=new THREE.Scene();
var camera=new THREE.PerspectiveCamera(48,1,0.1,200);
var cam={dist:11,yaw:0.7,pitch:0.32,auto:true,speed:0.15};

/* ---------- Sky, fog, environment ---------- */
var skyMat=new THREE.ShaderMaterial({side:THREE.BackSide,depthWrite:false,
 uniforms:{t:{value:new THREE.Color()},b:{value:new THREE.Color()},sc:{value:new THREE.Color()},sd:{value:new THREE.Vector3(0,1,0)}},
 vertexShader:'varying vec3 v;void main(){v=normalize(position);gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.);}',
 fragmentShader:'uniform vec3 t,b,sc,sd;varying vec3 v;void main(){float k=smoothstep(-0.05,0.85,v.y);vec3 c=mix(b,t,k);float g=max(dot(v,normalize(sd)),0.);c+=sc*(pow(g,48.)*0.9+pow(g,6.)*0.18);gl_FragColor=vec4(c,1.);}'});
var sky=new THREE.Mesh(new THREE.SphereGeometry(80,32,16),skyMat);scene.add(sky);
var envScene=new THREE.Scene();envScene.add(new THREE.Mesh(new THREE.SphereGeometry(50,32,16),skyMat));
var pmrem=new THREE.PMREMGenerator(renderer),envRT=null;
function rebuildEnv(){if(envRT)envRT.dispose();envRT=pmrem.fromScene(envScene);scene.environment=envRT.texture}
scene.fog=new THREE.FogExp2(0x000000,0.015);

var hemi=new THREE.HemisphereLight(0xffffff,0x222244,0.5);scene.add(hemi);
var key=new THREE.DirectionalLight(0xffffff,1.2);key.castShadow=true;
key.shadow.mapSize.set(2048,2048);key.shadow.bias=-0.0004;key.shadow.radius=4;
['left','bottom'].forEach(function(k){key.shadow.camera[k]=-12});['right','top'].forEach(function(k){key.shadow.camera[k]=12});
key.shadow.camera.far=50;scene.add(key);
var acc=[0xff4d8d,0x3fd4ff].map(function(c){var l=new THREE.PointLight(c,1.2,16,2);scene.add(l);return l});
var lightAngle=40;
function setLight(){var a=lightAngle*Math.PI/180;key.position.set(Math.cos(a)*10,9,Math.sin(a)*10);skyMat.uniforms.sd.value.copy(key.position)}

/* ---------- World ---------- */
var FLOOR=-2;
var floorMat=new THREE.MeshStandardMaterial({color:0x222222,roughness:0.28,metalness:0.35});
var floor=new THREE.Mesh(new THREE.CircleGeometry(70,64),floorMat);
floor.rotation.x=-Math.PI/2;floor.position.y=FLOOR;floor.receiveShadow=true;scene.add(floor);
var grid=new THREE.GridHelper(24,24,0x8896ad,0x8896ad);grid.position.y=FLOOR+0.01;grid.material.transparent=true;grid.material.opacity=0.22;scene.add(grid);
var mtMat=new THREE.MeshStandardMaterial({color:0x334455,roughness:1,flatShading:true});
for(var i=0;i<42;i++){
 var a=i/42*Math.PI*2+Math.random()*0.1,r=38+Math.random()*14,h=5+Math.random()*13;
 var m=new THREE.Mesh(new THREE.ConeGeometry(4+Math.random()*5,h,5+Math.floor(Math.random()*3)),mtMat);
 m.position.set(Math.cos(a)*r,FLOOR+h/2-0.4,Math.sin(a)*r);m.rotation.y=Math.random()*6;scene.add(m);
}
var starPos=new Float32Array(900);
for(var i=0;i<300;i++){var u=Math.random()*Math.PI*2,y=0.08+Math.random()*0.92,s=Math.sqrt(1-y*y);
 starPos[i*3]=Math.cos(u)*s*70;starPos[i*3+1]=y*70;starPos[i*3+2]=Math.sin(u)*s*70}
var sg=new THREE.BufferGeometry();sg.setAttribute('position',new THREE.BufferAttribute(starPos,3));
var stars=new THREE.Points(sg,new THREE.PointsMaterial({color:0xffffff,size:0.45,transparent:true,fog:false,depthWrite:false}));scene.add(stars);
var dustN=260,dustPos=new Float32Array(dustN*3);
for(var i=0;i<dustN;i++){dustPos[i*3]=(Math.random()-0.5)*26;dustPos[i*3+1]=FLOOR+Math.random()*10;dustPos[i*3+2]=(Math.random()-0.5)*26}
var dg=new THREE.BufferGeometry();dg.setAttribute('position',new THREE.BufferAttribute(dustPos,3));
var dust=new THREE.Points(dg,new THREE.PointsMaterial({color:0xffffff,size:0.07,transparent:true,opacity:0.55,depthWrite:false}));scene.add(dust);

var PRE={
 Dusk:{t:'#25225a',b:'#ff9468',sun:'#ffb784',si:1.3,h:0.55,f:0.016,floor:'#1d1a33',stars:0.55,mt:'#3a2a55',a1:'#ff4d8d',a2:'#6aa8ff'},
 Neon:{t:'#04010c',b:'#4b1a8f',sun:'#c26bff',si:0.9,h:0.35,f:0.02,floor:'#0b0720',stars:1,mt:'#1a0d3a',a1:'#ff2fb3',a2:'#27e6ff'},
 Day:{t:'#3d7de0',b:'#e4f1ff',sun:'#fff3d6',si:2,h:0.75,f:0.011,floor:'#bcc7d8',stars:0,mt:'#7f9bbd',a1:'#ffffff',a2:'#ffe9b0'},
 Studio:{t:'#272c38',b:'#a9b2c3',sun:'#ffffff',si:1.6,h:0.6,f:0.007,floor:'#363c4a',stars:0,mt:'#566073',a1:'#8fb4ff',a2:'#ffb08f'}
};
var cur='Dusk';
function preset(n){
 var p=PRE[n];cur=n;
 skyMat.uniforms.t.value.set(p.t);skyMat.uniforms.b.value.set(p.b);skyMat.uniforms.sc.value.set(p.sun);
 scene.fog.color.set(p.b);scene.fog.density=p.f;
 key.color.set(p.sun);key.intensity=p.si;hemi.intensity=p.h;hemi.color.set(p.t);hemi.groundColor.set(p.floor);
 floorMat.color.set(p.floor);mtMat.color.set(p.mt);stars.material.opacity=p.stars;stars.visible=p.stars>0;
 acc[0].color.set(p.a1);acc[1].color.set(p.a2);
 setLight();rebuildEnv();syncs.forEach(function(f){f()});
 document.querySelectorAll('#presets button').forEach(function(b){b.setAttribute('aria-pressed',b.textContent===n)});
}

/* ---------- Shapes, materials, textures ---------- */
var G={
 Cube:function(){return new THREE.BoxGeometry(1.6,1.6,1.6,1,1,1)},
 Sphere:function(){return new THREE.SphereGeometry(1,64,48)},
 Torus:function(){return new THREE.TorusGeometry(0.9,0.35,32,96)},
 Knot:function(){return new THREE.TorusKnotGeometry(0.8,0.27,220,32)},
 Cone:function(){return new THREE.ConeGeometry(1,2,48)},
 Cylinder:function(){return new THREE.CylinderGeometry(0.8,0.8,1.8,48)},
 Gem:function(){return new THREE.IcosahedronGeometry(1.1,0)},
 Octa:function(){return new THREE.OctahedronGeometry(1.2,0)},
 Dodeca:function(){return new THREE.DodecahedronGeometry(1.1,0)},
 Vase:function(){var p=[];for(var i=0;i<=14;i++){var t=i/14;p.push(new THREE.Vector2(0.35+Math.sin(t*Math.PI*1.6)*0.5+t*0.2,t*2-1))}return new THREE.LatheGeometry(p,48)},
 Spiral:function(){var pts=[];for(var i=0;i<=60;i++){var t=i/60*Math.PI*6;pts.push(new THREE.Vector3(Math.cos(t)*0.8,i/60*2.2-1.1,Math.sin(t)*0.8))}return new THREE.TubeGeometry(new THREE.CatmullRomCurve3(pts),160,0.12,12)}
};
var FLAT={Gem:1,Octa:1,Dodeca:1};
var MATS={
 Plastic:{r:.4,m:.05,cc:.3,o:1,e:0},Chrome:{c:'#eef3fb',r:.04,m:1,cc:0,o:1,e:0},Gold:{c:'#ffc857',r:.18,m:1,cc:0,o:1,e:0},
 Glass:{c:'#cfe8ff',r:.03,m:0,cc:1,o:.35,e:0},Neon:{r:.3,m:0,cc:0,o:1,e:1.6},Matte:{r:1,m:0,cc:0,o:1,e:0}
};
var TX={};
function tex(k){
 if(k==='None')return null;if(TX[k])return TX[k];
 var c=document.createElement('canvas');c.width=c.height=256;var g=c.getContext('2d');
 if(k==='Checker'){for(var y=0;y<8;y++)for(var x=0;x<8;x++){g.fillStyle=(x+y)%2?'#fff':'#777';g.fillRect(x*32,y*32,32,32)}}
 else if(k==='Stripes'){for(var x=0;x<8;x++){g.fillStyle=x%2?'#fff':'#666';g.fillRect(x*32,0,32,256)}}
 else if(k==='Grid'){g.fillStyle='#ddd';g.fillRect(0,0,256,256);g.strokeStyle='#333';g.lineWidth=6;for(var i=0;i<=256;i+=64){g.beginPath();g.moveTo(i,0);g.lineTo(i,256);g.moveTo(0,i);g.lineTo(256,i);g.stroke()}}
 else{var d=g.createImageData(256,256);for(var i=0;i<d.data.length;i+=4){var v=140+Math.random()*115;d.data[i]=d.data[i+1]=d.data[i+2]=v;d.data[i+3]=255}g.putImageData(d,0,0)}
 var t=new THREE.CanvasTexture(c);t.wrapS=t.wrapT=THREE.RepeatWrapping;t.repeat.set(2,2);t.encoding=THREE.sRGBEncoding;t.anisotropy=8;TX[k]=t;return t;
}
var PALETTE=[0x3556f5,0xf5576c,0x2fbf8f,0xf5b835,0x9b59d6,0x16a6c9];
var objs=[],sel=null,count=0;
var box=new THREE.BoxHelper(new THREE.Mesh(new THREE.BoxGeometry(1,1,1)),0xffd24a);box.visible=false;scene.add(box);

function add(type,pos,mat){
  var gm=new THREE.MeshPhysicalMaterial({color:PALETTE[count%PALETTE.length],roughness:0.4,metalness:0.05,clearcoat:0.3,clearcoatRoughness:0.2,flatShading:!!FLAT[type]});
  var m=new THREE.Mesh(G[type](),gm);m.castShadow=true;m.receiveShadow=true;
  var n=objs.length;
  m.position.set(pos?pos[0]:((n%5)-2)*2,pos?pos[1]:0,pos?pos[2]:Math.floor(n/5)*2);
  m.userData={name:type+' '+(++count),spin:0,float:0,by:m.position.y,ph:Math.random()*6,tex:'None'};
  if(mat)applyMat(m,mat);
  scene.add(m);objs.push(m);select(m);return m;
}
function applyMat(m,k){
  var p=MATS[k],M=m.material;
  if(p.c)M.color.set(p.c);
  M.roughness=p.r;M.metalness=p.m;M.clearcoat=p.cc;M.opacity=p.o;M.transparent=p.o<1;M.depthWrite=p.o>=1;
  M.emissive.set(p.e>0?M.color:0x000000);M.emissiveIntensity=p.e;M.needsUpdate=true;
}
function select(m){sel=m||null;refresh()}
function kill(m){scene.remove(m);m.geometry.dispose();m.material.dispose()}
function remove(){if(!sel)return;kill(sel);objs.splice(objs.indexOf(sel),1);select(null)}
function duplicate(){
  if(!sel)return;var c=sel.clone();c.geometry=sel.geometry.clone();c.material=sel.material.clone();
  c.userData=Object.assign({},sel.userData,{name:sel.userData.name+' copy'});
  c.position.x+=0.8;c.position.z+=0.8;c.userData.by=c.position.y;scene.add(c);objs.push(c);select(c);
}
function clearAll(){objs.forEach(kill);objs=[];sel=null;count=0}

/* ---------- UI helpers ---------- */
var bar=$('bar');
Object.keys(G).forEach(function(t){var b=document.createElement('button');b.textContent='+ '+t;b.onclick=function(){add(t)};bar.appendChild(b)});
var syncs=[];
function row(p,label,el,id){var r=document.createElement('div');r.className='row';var l=document.createElement('label');l.htmlFor=id;l.textContent=label;r.appendChild(l);r.appendChild(el);p.appendChild(r);return r}
function uid(){return 'u'+Math.random().toString(36).slice(2)}
function slider(p,label,min,max,step,get,set,need){
  var id=uid(),i=document.createElement('input');i.type='range';i.id=id;i.min=min;i.max=max;i.step=step;
  var o=document.createElement('output'),fmt=function(v){return step<1?(+v).toFixed(2):Math.round(v)};
  i.oninput=function(){if(need&&!sel)return;set(parseFloat(i.value));o.textContent=fmt(i.value)};
  var r=row(p,label,i,id);r.appendChild(o);
  syncs.push(function(){if(need&&!sel)return;var v=get();i.value=v;o.textContent=fmt(v)});
}
function color(p,label,get,set,need){
  var id=uid(),i=document.createElement('input');i.type='color';i.id=id;
  i.oninput=function(){if(need&&!sel)return;set(i.value)};row(p,label,i,id);
  syncs.push(function(){if(need&&!sel)return;i.value=get()});
}
function check(p,label,get,set,need){
  var id=uid(),i=document.createElement('input');i.type='checkbox';i.id=id;
  i.onchange=function(){if(need&&!sel)return;set(i.checked)};row(p,label,i,id);
  syncs.push(function(){if(need&&!sel)return;i.checked=get()});
}
function dropdown(p,label,opts,get,set,need){
  var id=uid(),s=document.createElement('select');s.id=id;
  opts.forEach(function(o){var e=document.createElement('option');e.textContent=o;s.appendChild(e)});
  s.onchange=function(){if(need&&!sel)return;set(s.value)};row(p,label,s,id);
  syncs.push(function(){if(need&&!sel)return;s.value=get()});
}
function head(p,t){var h=document.createElement('h2');h.textContent=t;p.appendChild(h)}
function btn(p,t,fn,cls){var b=document.createElement('button');b.textContent=t;b.onclick=fn;if(cls)b.className=cls;p.appendChild(b);return b}
var D=180/Math.PI,R=Math.PI/180;

/* Object tab */
var objTab=$('objTab');
var list=document.createElement('div');list.id='list';objTab.appendChild(list);
var empty=document.createElement('p');empty.className='empty';empty.textContent='Nothing selected. Click a shape, or add one from the top bar.';objTab.appendChild(empty);
var props=document.createElement('div');props.className='group';objTab.appendChild(props);
var acts=document.createElement('div');acts.className='actions';
btn(acts,'Duplicate',duplicate);btn(acts,'Delete',remove,'danger');props.appendChild(acts);
head(props,'Position');
['x','y','z'].forEach(function(a){slider(props,a.toUpperCase(),-8,8,0.05,function(){return a==='y'?sel.userData.by:sel.position[a]},function(v){if(a==='y')sel.userData.by=v;else sel.position[a]=v},true)});
head(props,'Rotation (degrees)');
['x','y','z'].forEach(function(a){slider(props,a.toUpperCase(),0,360,1,function(){return ((sel.rotation[a]*D)%360+360)%360},function(v){sel.rotation[a]=v*R},true)});
head(props,'Size');
slider(props,'Scale',0.2,3,0.05,function(){return sel.scale.x},function(v){sel.scale.set(v,v,v)},true);
head(props,'Material');
dropdown(props,'Preset',['Plastic','Chrome','Gold','Glass','Neon','Matte'],function(){return sel.userData.mat||'Plastic'},function(v){sel.userData.mat=v;applyMat(sel,v);syncs.forEach(function(f){f()})},true);
dropdown(props,'Texture',['None','Checker','Stripes','Grid','Noise'],function(){return sel.userData.tex},function(v){sel.userData.tex=v;sel.material.map=tex(v);sel.material.needsUpdate=true},true);
color(props,'Color',function(){return '#'+sel.material.color.getHexString()},function(v){sel.material.color.set(v);if(sel.material.emissiveIntensity>0)sel.material.emissive.set(v)},true);
slider(props,'Roughness',0,1,0.01,function(){return sel.material.roughness},function(v){sel.material.roughness=v},true);
slider(props,'Metalness',0,1,0.01,function(){return sel.material.metalness},function(v){sel.material.metalness=v},true);
slider(props,'Clearcoat',0,1,0.01,function(){return sel.material.clearcoat},function(v){sel.material.clearcoat=v},true);
slider(props,'Glow',0,3,0.05,function(){return sel.material.emissiveIntensity},function(v){sel.material.emissiveIntensity=v;sel.material.emissive.set(v>0?sel.material.color:0x000000)},true);
slider(props,'Opacity',0.1,1,0.01,function(){return sel.material.opacity},function(v){var M=sel.material;M.opacity=v;M.transparent=v<1;M.depthWrite=v>=1;M.needsUpdate=true},true);
check(props,'Wireframe',function(){return sel.material.wireframe},function(v){sel.material.wireframe=v},true);
head(props,'Animation');
slider(props,'Spin',0,4,0.1,function(){return sel.userData.spin},function(v){sel.userData.spin=v},true);
slider(props,'Float',0,2,0.05,function(){return sel.userData.float},function(v){sel.userData.float=v},true);

function refresh(){
  list.innerHTML='';
  objs.forEach(function(m){var b=document.createElement('button');b.textContent=m.userData.name;b.setAttribute('aria-pressed',m===sel);b.onclick=function(){select(m)};list.appendChild(b)});
  props.hidden=!sel;empty.hidden=!!sel;
  if(sel){box.visible=true;box.setFromObject(sel)}else box.visible=false;
  syncs.forEach(function(f){f()});
}

/* Scene tab */
var sc=$('sceneTab');
head(sc,'Atmosphere');
var pr=document.createElement('div');pr.id='presets';pr.className='actions';
Object.keys(PRE).forEach(function(n){btn(pr,n,function(){preset(n)})});sc.appendChild(pr);
head(sc,'Lighting');
slider(sc,'Sun',0,3,0.05,function(){return key.intensity},function(v){key.intensity=v});
slider(sc,'Sky light',0,1.5,0.05,function(){return hemi.intensity},function(v){hemi.intensity=v});
slider(sc,'Sun angle',0,360,1,function(){return lightAngle},function(v){lightAngle=v;setLight()});
slider(sc,'Exposure',0.3,2.5,0.05,function(){return renderer.toneMappingExposure},function(v){renderer.toneMappingExposure=v});
slider(sc,'Accents',0,3,0.05,function(){return acc[0].intensity},function(v){acc.forEach(function(l){l.intensity=v})});
check(sc,'Shadows',function(){return floor.receiveShadow},function(v){key.castShadow=v;floor.receiveShadow=v;floorMat.needsUpdate=true});
head(sc,'World');
slider(sc,'Fog',0,0.06,0.001,function(){return scene.fog.density},function(v){scene.fog.density=v});
slider(sc,'Reflect',0,1,0.01,function(){return floorMat.metalness},function(v){floorMat.metalness=v;floorMat.roughness=0.55-v*0.4});
check(sc,'Grid',function(){return grid.visible},function(v){grid.visible=v});
check(sc,'Auto-orbit',function(){return cam.auto},function(v){cam.auto=v});
head(sc,'Scene');
var sa=document.createElement('div');sa.className='actions';
btn(sa,'Reset camera',function(){cam.dist=11;cam.yaw=0.7;cam.pitch=0.32});
btn(sa,'Showcase',showcase);
btn(sa,'Random scene',function(){
  clearAll();var t=Object.keys(G),mk=Object.keys(MATS);
  for(var i=0;i<10;i++){var m=add(t[Math.floor(Math.random()*t.length)],[(Math.random()-0.5)*10,(Math.random()-0.5)*3,(Math.random()-0.5)*10],mk[Math.floor(Math.random()*mk.length)]);
    m.userData.mat=null;m.rotation.set(Math.random()*6,Math.random()*6,0);m.scale.setScalar(0.6+Math.random()*0.9);m.userData.spin=Math.random()*1.5;m.userData.float=Math.random()}
  select(null)});
btn(sa,'Clear all',function(){clearAll();refresh()},'danger');
sc.appendChild(sa);

function showcase(){
  clearAll();
  var k=add('Knot',[0,0.2,0],'Chrome');k.userData.mat='Chrome';k.userData.spin=0.6;k.scale.setScalar(1.3);
  var a=add('Sphere',[-3.4,-0.8,0.6],'Gold');a.userData.mat='Gold';a.userData.float=0.6;
  var b=add('Gem',[3.4,0,0.4],'Glass');b.userData.mat='Glass';b.userData.spin=0.5;b.userData.float=0.8;
  var c=add('Torus',[0,-0.4,3.2],'Neon');c.userData.mat='Neon';c.material.color.set('#ff3fa4');c.material.emissive.set('#ff3fa4');c.rotation.x=1.2;c.userData.spin=0.8;
  var d=add('Vase',[-2.6,-1,-3],'Plastic');d.userData.mat='Plastic';d.material.color.set('#2fbf8f');
  var e=add('Spiral',[2.8,-0.9,-3],'Neon');e.userData.mat='Neon';e.material.color.set('#27e6ff');e.material.emissive.set('#27e6ff');e.userData.spin=0.7;
  var f=add('Cube',[-5.2,-1.2,-1.4],'Plastic');f.userData.mat='Plastic';f.userData.tex='Checker';f.material.map=tex('Checker');f.material.color.set('#f5b835');f.scale.setScalar(0.8);f.rotation.y=0.6;
  objs.forEach(function(m){m.userData.by=m.position.y});
  select(null);
}

$('tObj').onclick=$('tScene').onclick=function(e){
  var o=e.target===$('tObj');
  $('tObj').setAttribute('aria-pressed',o);$('tScene').setAttribute('aria-pressed',!o);objTab.hidden=!o;sc.hidden=o;
};
addEventListener('keydown',function(e){
  if((e.key==='Delete'||e.key==='Backspace')&&sel&&!/INPUT|TEXTAREA|SELECT/.test(e.target.tagName)){e.preventDefault();remove()}
});

/* ---------- Camera input + picking ---------- */
var pts={},last=0,px=0,py=0,moved=0,ray=new THREE.Raycaster(),mouse=new THREE.Vector2();
function clamp(v,a,b){return Math.max(a,Math.min(b,v))}
stage.addEventListener('pointerdown',function(e){stage.setPointerCapture(e.pointerId);pts[e.pointerId]=e;px=e.clientX;py=e.clientY;last=0;moved=0});
stage.addEventListener('pointermove',function(e){
  if(!pts[e.pointerId])return;
  var ids=Object.keys(pts);pts[e.pointerId]=e;
  if(ids.length===2){var a=pts[ids[0]],b=pts[ids[1]],d=Math.hypot(a.clientX-b.clientX,a.clientY-b.clientY);if(last)cam.dist=clamp(cam.dist*last/d,3,30);last=d;moved=99;return}
  moved+=Math.abs(e.clientX-px)+Math.abs(e.clientY-py);
  cam.yaw-=(e.clientX-px)*0.008;cam.pitch=clamp(cam.pitch+(e.clientY-py)*0.008,-0.1,1.45);px=e.clientX;py=e.clientY;
});
function up(e){
  var single=Object.keys(pts).length===1;
  if(e.type==='pointerup'&&single&&moved<6){
    var r=stage.getBoundingClientRect();
    mouse.set(((e.clientX-r.left)/r.width)*2-1,-((e.clientY-r.top)/r.height)*2+1);
    ray.setFromCamera(mouse,camera);var hit=ray.intersectObjects(objs)[0];select(hit?hit.object:null);
  }
  delete pts[e.pointerId];last=0;
}
stage.addEventListener('pointerup',up);stage.addEventListener('pointercancel',up);
stage.addEventListener('wheel',function(e){e.preventDefault();cam.dist=clamp(cam.dist+e.deltaY*0.01,3,30)},{passive:false});
function resize(){var w=stage.clientWidth,h=stage.clientHeight;renderer.setSize(w,h,false);camera.aspect=w/h;camera.updateProjectionMatrix()}
addEventListener('resize',resize);resize();

/* ---------- Loop ---------- */
var reduce=matchMedia('(prefers-reduced-motion: reduce)').matches,clock=new THREE.Clock(),T=0;
function tick(){
  var dt=clock.getDelta();T+=dt;
  if(!reduce){
    objs.forEach(function(m){var u=m.userData;if(u.spin)m.rotation.y+=dt*u.spin;m.position.y=u.by+(u.float?Math.sin(T*1.4+u.ph)*0.3*u.float:0)});
    if(cam.auto&&!Object.keys(pts).length)cam.yaw+=dt*cam.speed;
    acc[0].position.set(Math.cos(T*0.5)*6,1.5+Math.sin(T*0.7),Math.sin(T*0.5)*6);
    acc[1].position.set(Math.cos(T*0.4+3)*7,0.5+Math.cos(T*0.6),Math.sin(T*0.4+3)*7);
    var dp=dg.attributes.position.array;
    for(var i=0;i<dustN;i++){dp[i*3+1]+=dt*0.15;dp[i*3]+=Math.sin(T+i)*dt*0.05;if(dp[i*3+1]>FLOOR+10)dp[i*3+1]=FLOOR}
    dg.attributes.position.needsUpdate=true;stars.rotation.y+=dt*0.004;
  }
  camera.position.set(Math.sin(cam.yaw)*Math.cos(cam.pitch)*cam.dist,Math.sin(cam.pitch)*cam.dist,Math.cos(cam.yaw)*Math.cos(cam.pitch)*cam.dist);
  camera.lookAt(0,-0.3,0);
  if(sel)box.setFromObject(sel);
  renderer.render(scene,camera);
  requestAnimationFrame(tick);
}
preset('Dusk');showcase();
tick();
})();
</script>
</body>
</html>
