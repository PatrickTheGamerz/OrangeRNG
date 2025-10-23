<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — With Weathers & Totems</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body{margin:0;font-family:sans-serif;background:#0e0f13;color:#e7e9ee;display:grid;place-items:center;min-height:100vh;}
    .app{width:980px;max-width:96vw;background:#151822;border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;}
    .roll-area{min-height:260px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .result{font-size:28px;font-weight:700;text-align:center;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;}
    .banner{position:absolute;left:50%;transform:translateX(-50%);font-weight:700;text-align:center;opacity:0;pointer-events:none;padding:6px 10px;border-radius:10px;}
    .banner.luck{top:14px;font-size:18px;}
    .banner.new{top:120px;color:#6ea8fe;font-size:16px;}
    .banner.announce{top:14px;font-size:16px;border:1px solid #2a3449;background:#1b2232;}
    .banner.weather{top:14px;color:#4ec3ff;font-size:18px;}
    .fadeout{animation:fadeout 3.6s forwards;}
    @keyframes fadeout{0%{opacity:1;}70%{opacity:1;}100%{opacity:0;filter:blur(4px)}}
    .active-effects{position:absolute;bottom:10px;right:10px;font-size:12px;text-align:right;max-width:46%;display:flex;flex-direction:column;align-items:flex-end;gap:2px;}
    .effect-entry{font-weight:600;display:block;padding:2px 6px;border-radius:8px;border:1px solid #2a3449;background:#1b2232;}
    button{background:#1b2232;color:#e7e9ee;border:1px solid #2a3449;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;}
    button:disabled{opacity:0.4;cursor:not-allowed;}
  </style>
</head>
<body>
  <div class="app">
    <div class="content">
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
          <div class="active-effects" id="activeEffects"></div>
        </div>
        <div class="controls">
          <button id="btnRoll">Roll</button>
        </div>
      </div>
    </div>
  </div>

<script>
/* ---------------- Weather System ---------------- */
const WEATHERS = {
  normal: [
    {name:"Storm", color:"b-rare"},
    {name:"Blizzard", color:"b-epic"},
    {name:"Sunny Radiance", color:"b-common"}
  ],
  rare: [
    {name:"Meteor Storm", color:"b-legendary"},
    {name:"Aurora Veil", color:"b-mythic"}
  ],
  super: [
    {name:"Eternal Eclipse", color:"b-divine"},
    {name:"Cosmic Tempest", color:"b-omniversal"}
  ]
};

let state = {
  effectInstances: []
};

function spawnBanner(text,type,colorClass){
  const rollArea=document.getElementById("rollArea");
  const div=document.createElement("div");
  div.className=`banner ${type} fadeout ${colorClass||""}`;
  div.textContent=text;
  div.addEventListener("animationend",()=>div.remove());
  rollArea.appendChild(div);
}

function addEffect(effect,isWeather=false){
  const now=Date.now();
  const durMs=effect.duration*1000;
  const inst={
    name:effect.name,
    type:isWeather?"weather":effect.type,
    amount:effect.amount||0,
    target:effect.target,
    expiresAt:now+durMs,
    rarityKey:effect.rarity||"common",
    weather:isWeather
  };
  state.effectInstances.push(inst);
  renderActiveEffects();
}

function renderActiveEffects(){
  const el=document.getElementById("activeEffects");
  el.innerHTML="";
  const now=Date.now();
  const active=state.effectInstances.filter(e=>e.expiresAt>now);
  // Weather always last
  const weathers=active.filter(e=>e.weather);
  const others=active.filter(e=>!e.weather);
  const sorted=[...others,...weathers];
  sorted.forEach(e=>{
    const left=Math.max(0,Math.ceil((e.expiresAt-now)/1000));
    const div=document.createElement("div");
    div.className=`effect-entry ${e.colorClass||""}`;
    div.textContent=`${e.name}: ${left}s left`;
    el.appendChild(div);
  });
}

/* ---------------- Weather Random Events ---------------- */
function triggerRandomWeather(){
  const roll=Math.random();
  let pool, rarity;
  if(roll<0.7){ pool=WEATHERS.normal; rarity="common"; }
  else if(roll<0.95){ pool=WEATHERS.rare; rarity="rare"; }
  else { pool=WEATHERS.super; rarity="legendary"; }
  const w=pool[Math.floor(Math.random()*pool.length)];
  const dur=100+Math.floor(Math.random()*200);
  addEffect({name:w.name,duration:dur,rarity:rarity,colorClass:w.color},true);
  spawnBanner(`${w.name} started`,"weather",w.color);
  scheduleNextWeather();
}

function scheduleNextWeather(){
  const delay=(240+Math.random()*480)*1000; // 4–12 minutes
  setTimeout(triggerRandomWeather,delay);
}
scheduleNextWeather();

/* ---------------- Roll Simulation ---------------- */
function rollOnce(){
  document.getElementById("resultText").textContent="Rolled something!";
  spawnBanner("Roll happened","luck","b-common");
}

document.getElementById("btnRoll").addEventListener("click",rollOnce);

</script>
</body>
</html>
