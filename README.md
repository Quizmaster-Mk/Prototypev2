<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Avengers vs Villains V2</title>
<style>
body { font-family: Arial, sans-serif; background:#111; color:#eee; text-align:center; }
h1 { color:#f39c12; }
.choice { margin:20px; }
button { padding:10px 20px; margin:5px; cursor:pointer; font-size:16px; }
.bar { height:20px; background:#333; border-radius:10px; margin:10px auto; width:300px; position:relative; overflow:hidden; }
.fill { height:100%; border-radius:10px; transition: width 0.5s ease; }
#log { border:1px solid #555; padding:10px; height:200px; overflow-y:auto; background:#222; text-align:left; margin-top:20px; }
.avatar { font-size:50px; display:inline-block; transition: transform 0.2s; position:relative; }
.floating-dmg { position:absolute; animation: floatUp 1s ease-out forwards; font-weight:bold; }
@keyframes floatUp { from { opacity:1; transform:translateY(0); } to { opacity:0; transform:translateY(-50px); } }
.crit { font-size: 24px; color: yellow; font-weight: bold; }
.shield { box-shadow: 0 0 20px 5px #3498db; border-radius:50%; }
.hit { animation: hitAnim 0.3s; }
@keyframes hitAnim { 0% { transform: scale(1); } 50% { transform: scale(1.2); } 100% { transform: scale(1); } }
</style>
</head>
<body>
<h1>🦸 Avengers vs Villains V2</h1>

<div id="setup">
  <div class="choice">
    <h2>Choisis ton héros</h2>
    <button onclick="selectHero('Iron Man','🤖')">🤖 Iron Man</button>
    <button onclick="selectHero('Thor','⚡')">⚡ Thor</button>
  </div>
  <div class="choice">
    <h2>Choisis ton arme</h2>
    <button onclick="selectWeapon('Répulseurs',4,7)">🔫 Répulseurs (4-7)</button>
    <button onclick="selectWeapon('Mjolnir',5,9)">⚒️ Mjolnir (5-9)</button>
  </div>
  <div class="choice">
    <h2>Choisis ton pouvoir offensif</h2>
    <button onclick="selectPower('Explosion',8,12)">💥 Explosion (8-12)</button>
    <button onclick="selectPower('Frappe divine',8,12)">⚡ Frappe divine (8-12)</button>
  </div>
  <div class="choice">
    <h2>Choisis la mission</h2>
    <button onclick="startMission('Thanos')">🟣 Empêcher Thanos</button>
    <button onclick="startMission('Chitauri')">👽 Sauver New York</button>
  </div>
</div>

<div id="battle" style="display:none;">
  <div>
    <div class="avatar" id="heroAvatar"></div>
    <div id="heroName"></div>
    <div class="bar"><div id="player-bar" class="fill" style="background:#2ecc71; width:100%"></div></div>
    <div>PV: <span id="player-pv"></span></div>
  </div>
  <hr>
  <div>
    <div class="avatar" id="enemyAvatar">🟣</div>
    <div id="enemyName"></div>
    <div class="bar"><div id="enemy-bar" class="fill" style="background:#e74c3c; width:100%"></div></div>
    <div>PV: <span id="enemy-pv"></span></div>
  </div>
  <hr>
  <div id="actions"></div>
  <button id="continueBtn" style="display:none;" onclick="continueBattle()">Continuer</button>
  <div id="log"></div>
</div>

<script>
let state = {
  hero:null, avatar:null, weapon:null, power:null, mission:null,
  player:{pv:30, max:30, usedPower:false, shieldAvailable:true, shieldActive:false},
  enemy:{pv:50, max:50, dmg:[4,8]},
  turn:"player"
};

function selectHero(name, emoji){ state.hero=name; state.avatar=emoji; }
function selectWeapon(name,min,max){ state.weapon={name,min,max}; }
function selectPower(name,min,max){ state.power={name,min,max}; }

function startMission(mission){
  state.mission=mission;
  document.getElementById("setup").style.display="none";
  document.getElementById("battle").style.display="block";
  if(mission==="Thanos"){ state.enemy.pv=50; state.enemy.max=50; state.enemy.dmg=[4,8]; document.getElementById("enemyAvatar").textContent="🟣"; document.getElementById("enemyName").textContent="Thanos"; }
  if(mission==="Chitauri"){ state.enemy.pv=40; state.enemy.max=40; state.enemy.dmg=[3,6]; document.getElementById("enemyAvatar").textContent="👽"; document.getElementById("enemyName").textContent="Chitauri"; }
  state.player.pv=30; state.player.max=30; state.player.usedPower=false;
  state.player.shieldAvailable=true; state.player.shieldActive=false;
  document.getElementById("heroAvatar").textContent=state.avatar;
  document.getElementById("heroName").textContent=state.hero;
  render();
  playerAttackPrompt();
}

function rnd(min,max){ return Math.floor(Math.random()*(max-min+1))+min; }

function render(){
  document.getElementById("player-pv").textContent=state.player.pv+"/"+state.player.max;
  document.getElementById("enemy-pv").textContent=state.enemy.pv+"/"+state.enemy.max;
  document.getElementById("player-bar").style.width=(state.player.pv/state.player.max*100)+"%";
  document.getElementById("enemy-bar").style.width=(state.enemy.pv/state.enemy.max*100)+"%";
}

function pushLog(msg){
  const log=document.getElementById("log");
  log.innerHTML+="<div>"+msg+"</div>";
  log.scrollTop=log.scrollHeight;
}

function showFloatingDmg(target,dmg,color,crit=false){
  const bar=document.getElementById(target);
  const span=document.createElement("span");
  span.className="floating-dmg";
  span.textContent=(dmg>0? "-"+dmg: dmg);
  span.style.color=color;
  if(crit) span.className="floating-dmg crit";
  span.style.left=(bar.offsetLeft+100)+"px";
  span.style.top=(bar.offsetTop-20)+"px";
  document.body.appendChild(span);
  setTimeout(()=>span.remove(),1000);
  document.getElementById(target==="player-bar"?"heroAvatar":"enemyAvatar").classList.add("hit");
  setTimeout(()=>{document.getElementById(target==="player-bar"?"heroAvatar":"enemyAvatar").classList.remove("hit");},300);
}

function playerAttackPrompt(){
  document.getElementById("actions").innerHTML=`
    <button onclick="applyPlayerDamage('Rapide')">⚡ Rapide (3-5)</button>
    <button onclick="applyPlayerDamage('Lourde')">💥 Lourde (5-9)</button>
    <button onclick="applyPlayerDamage('Arme')">${state.weapon?state.weapon.name:"Arme"} (${state.weapon.min}-${state.weapon.max})</button>
    <button ${state.player.usedPower?"disabled":""} onclick="applyPlayerDamage('Pouvoir')">${state.power?state.power.name:"Pouvoir"} (8-12, unique)</button>
    <button ${state.player.shieldAvailable?"":"disabled"} onclick="applyPlayerDamage('Bouclier')">🛡️ Bouclier (bloque 1 attaque)</button>
  `;
}

function applyPlayerDamage(type){
  let dmg=0, miss=false, crit=false;
  if(type==="Rapide") dmg=rnd(3,5);
  if(type==="Lourde") dmg=rnd(5,9);
  if(type==="Arme") dmg=rnd(state.weapon.min,state.weapon.max);
  if(type==="Pouvoir"){ dmg=rnd(8,12); state.player.usedPower=true; }
  if(type==="Bouclier"){ state.player.shieldActive=true; state.player.shieldAvailable=false; pushLog("🛡️ Bouclier activé ! La prochaine attaque sera bloquée."); dmg=0; }
  if(type!=="Bouclier" && Math.random()<0.2){ crit=true; dmg*=2; }
  if(type!=="Bouclier"){ state.enemy.pv=Math.max(0,state.enemy.pv-dmg); render(); showFloatingDmg("enemy-bar",dmg,crit?"yellow":"red",crit); pushLog((crit?"💥 Coup critique ! ":"✅ Tu infliges ")+dmg+" PV"); }
  document.getElementById("actions").innerHTML="";
  document.getElementById("continueBtn").style.display="block";
  state.turn="enemy";
  if(state.enemy.pv<=0){ pushLog("🎉 Ennemi vaincu !"); endGame(); }
}

function continueBattle(){
  document.getElementById("continueBtn").style.display="none";
  if(state.turn==="enemy"){ enemyTurn(); } else { playerAttackPrompt(); }
}

function enemyTurn(){
  if(state.enemy.pv<=0){ endGame(); return; }
  if(state.player.shieldActive){ pushLog("🛡️ Bouclier bloque l'attaque !"); state.player.shieldActive=false; }
  else{
    let dmg=rnd(state.enemy.dmg[0],state.enemy.dmg[1]);
    state.player.pv=Math.max(0,state.player.pv-dmg);
    pushLog("⚔️ L'ennemi attaque et inflige "+dmg+" PV !");
    showFloatingDmg("player-bar",dmg,"red");
  }
  render(); state.turn="player";
  if(state.player.pv<=0){ pushLog("💀 "+state.hero+" est vaincu..."); endGame(); }
  else { document.getElementById("continueBtn").style.display="block"; }
}

function endGame(){ document.getElementById("actions").innerHTML=""; document.getElementById("continueBtn").style.display="none"; }
</script>
</body>
</html>
