<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol's RNG — Milestone Luck</title>
  <style>
    body { background:#0e0f13; color:#eee; font-family:sans-serif; display:grid; place-items:center; min-height:100vh; }
    .app { background:#151822; padding:20px; border-radius:12px; width:600px; max-width:95vw; }
    button { margin:6px; padding:10px 16px; border:none; border-radius:8px; cursor:pointer; background:#1b2232; color:#eee; font-weight:600; }
    button:disabled { opacity:0.4; cursor:not-allowed; }
    .result { font-size:24px; margin-top:12px; }
    table { width:100%; margin-top:12px; border-collapse:collapse; }
    td,th { padding:6px; border-bottom:1px solid #333; }
  </style>
</head>
<body>
  <div class="app">
    <h2>Sol’s RNG — Milestone Luck</h2>
    <div>
      <button id="btnRoll">Roll</button>
      <button id="btnAuto" disabled>Auto (x100)</button>
      <button id="btnReset">Reset</button>
    </div>
    <div class="result" id="resultText">Ready to roll</div>
    <div id="stats"></div>
    <table id="probTable">
      <thead><tr><th>Rarity</th><th>Weight</th><th>Chance</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>

<script>
const BASE_RARITIES = [
  { key:"common", name:"Common", weight:980000 },
  { key:"uncommon", name:"Uncommon", weight:18000 },
  { key:"rare", name:"Rare", weight:1800 },
  { key:"epic", name:"Epic", weight:180 },
  { key:"legendary", name:"Legendary", weight:18 },
  { key:"divine", name:"Divine", weight:1 },
];

const state = { rolls:0 };

function sumWeights(arr){ return arr.reduce((s,r)=>s+r.weight,0); }
function formatChance(p){ return (p*100).toFixed(4)+"%"; }

function getMultiplier(){
  if(state.rolls>0 && state.rolls % 250 === 0) return 10;
  if(state.rolls>0 && state.rolls % 50 === 0) return 2;
  return 1;
}

function applyMultiplier(base, mult){
  // multiply all weights by 1, then apply multiplier to rarer tiers
  const arr = JSON.parse(JSON.stringify(base));
  // simplest: multiply all rarities equally
  arr.forEach(r => r.weight = r.weight * (r.key==="common" ? 1 : mult));
  return arr;
}

function pickTier(rarities){
  const total = sumWeights(rarities);
  let r = Math.random()*total, acc=0;
  for(const tier of rarities){
    acc += tier.weight;
    if(r <= acc) return tier;
  }
  return rarities[0];
}

function rollOnce(){
  state.rolls++;
  const mult = getMultiplier();
  const rarities = applyMultiplier(BASE_RARITIES, mult);
  const result = pickTier(rarities);
  document.getElementById("resultText").textContent =
    `Roll #${state.rolls}: ${result.name} ${mult>1?`(x${mult} luck!)`:""}`;
  renderStats(mult);
}

function autoRoll(n=100){
  for(let i=0;i<n;i++) rollOnce();
}

function renderStats(mult){
  const stats = document.getElementById("stats");
  stats.textContent = `Total Rolls: ${state.rolls} | Current Multiplier: x${mult}`;
  if(state.rolls>=100) document.getElementById("btnAuto").disabled=false;
  renderTable(mult);
}

function renderTable(mult){
  const rarities = applyMultiplier(BASE_RARITIES, mult);
  const total = sumWeights(rarities);
  document.querySelector("#probTable tbody").innerHTML =
    rarities.map(r=>`<tr><td>${r.name}</td><td>${r.weight}</td><td>${formatChance(r.weight/total)}</td></tr>`).join("");
}

document.getElementById("btnRoll").onclick=rollOnce;
document.getElementById("btnAuto").onclick=()=>autoRoll(100);
document.getElementById("btnReset").onclick=()=>{
  state.rolls=0;
  document.getElementById("resultText").textContent="Ready to roll";
  document.getElementById("btnAuto").disabled=true;
  renderStats(1);
};

renderStats(1);
</script>
</body>
</html>
