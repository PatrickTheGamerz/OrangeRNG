<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Expanded with Items, Consumables, Exclusive</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab;
      --accent:#6ea8fe; --gold:#ffd700; --warn:#ff6666;
    }
    body{margin:0;font-family:sans-serif;background:var(--bg);color:var(--text);display:grid;place-items:center;min-height:100vh;}
    .app{width:980px;max-width:96vw;background:var(--panel);border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;}

    /* Roll area and banners */
    .roll-area{min-height:260px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .result{font-size:28px;font-weight:700;text-align:center;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;}
    .banner{position:absolute;left:50%;transform:translateX(-50%);font-weight:700;text-align:center;opacity:0;pointer-events:none;}
    .banner.luck{top:14px;color:var(--gold);font-size:18px;}
    .banner.new{top:120px;color:var(--accent);font-size:16px;}
    .banner.announce{top:-34px;color:var(--warn);font-size:16px;}
    .fadeout{animation:fadeout 3.6s forwards;}
    @keyframes fadeout{0%{opacity:1;filter:blur(0)}70%{opacity:1;}100%{opacity:0;filter:blur(4px)}}
    .active-effects{position:absolute;bottom:10px;left:10px;font-size:12px;}
    .effect-entry{margin-top:2px;}

    /* Controls */
    .controls{display:flex;gap:12px;padding-top:12px;flex-wrap:wrap;align-items:center;}
    button{
      background:#1b2232;color:var(--text);border:1px solid #2a3449;
      padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;
      transition:background .2s, box-shadow .2s, transform .06s;
    }
    button:hover{background:#232c41;box-shadow:0 6px 18px rgba(110,168,254,0.12);}
    button:active{transform:translateY(1px);}
    button:disabled{opacity:0.4;cursor:not-allowed;}

    /* Index */
    .index-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:12px;}
    .index-section{background:#0f1320;border:1px solid #252b39;border-radius:10px;padding:12px;position:relative;}
    .index-section h4{margin:0 0 8px;font-size:14px;color:var(--muted);display:flex;align-items:center;gap:8px;}
    .completion{position:absolute;top:8px;right:12px;font-size:12px;color:var(--accent);}
    .badge{display:inline-block;padding:2px 8px;border-radius:999px;font-size:12px;font-weight:700;}
    .index-list{list-style:none;padding:0;margin:0;}
    .index-item{display:flex;justify-content:space-between;align-items:center;padding:6px 0;border-bottom:1px solid #242a38;font-size:14px;}
    .locked{filter:blur(3px);opacity:0.6;}
    .unlocked{color:var(--accent);font-weight:600;}

    /* Inventory */
    .inv-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px;gap:10px;}
    .inv-title{display:flex;align-items:center;gap:10px;}
    .inv-stats{font-size:13px;color:var(--muted);}
    .inv-warning{color:var(--warn);font-size:13px;}
    .inv-list{list-style:none;padding:0;margin:0;}
    .inv-list li{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid #242a38;padding:6px 0;}
    .inv-actions{display:flex;gap:8px;}

    /* Mode selector and Auto-Sell carousel side-by-side */
    .mode-wrap{display:flex;align-items:center;gap:12px;}
    .mode-label{font-size:13px;color:var(--muted);}
    .mode-carousel{display:flex;align-items:center;gap:6px;}
    .mode-value{min-width:80px;text-align:center;padding:8px 10px;border:1px solid #2a3449;border-radius:8px;background:#1b2232;}
    .mode-arrow{padding:6px 10px;}

    .autosell-wrap{display:flex;align-items:center;gap:8px; margin-left:auto;}
    .autosell-label{font-size:13px;color:var(--muted);}
    .autosell-carousel{display:flex;align-items:center;gap:6px;}
    .autosell-value{min-width:120px;text-align:center;padding:8px 10px;border:1px solid #2a3449;border-radius:8px;background:#1b2232;}
    .autosell-arrow{padding:6px 10px;}

    /* Rarity colors */
    .b-worthless{background:#1a1a1a;color:#b3b3b3;} .b-trash{background:#211616;color:#d28f8f;}
    .b-common{background:#1f2635;color:#c7d1e5;} .b-uncommon{background:#14261d;color:#7ae08f;}
    .b-rare{background:#221a35;color:#caa6ff;} .b-epic{background:#10233a;color:#7bb7ff;}
    .b-legendary{background:#2a1f12;color:#ffbf66;} .b-mythic{background:#2a142a;color:#ff7ee6;}
    .b-divine{background:#2a2a12;color:#ffd700;} .b-celestial{background:#142a2a;color:#7affff;}
    .b-transcendent{background:#1a1a2f;color:#a0a7ff;} .b-eternal{background:#1a2f2a;color:#9cf2c7;}
    .b-omniversal{background:#2f1a2f;color:#ff9cff;}
    .b-exclusive{background:linear-gradient(270deg,red,orange,yellow,green,blue,indigo,violet);background-size:400% 400%;animation:rainbow 6s linear infinite;color:white;}
    @keyframes rainbow{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}

    /* Glow effect */
    .glow{position:absolute;inset:-40%;border-radius:50%;background:radial-gradient(closest-side,rgba(110,168,254,0.25),transparent 65%);filter:blur(12px);animation:glow 1.1s ease-out forwards;}
    @keyframes glow{0%{opacity:0;transform:scale(0.7)}50%{opacity:1}100%{opacity:0;transform:scale(1.2)}}
  </style>
</head>
<body>
  <div class="app">
    <div class="content">
      <!-- Rolling panel -->
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
          <div class="active-effects" id="activeEffects"></div>
        </div>
        <div class="controls">
          <button id="btnRoll">Roll</button>
          <button id="btnAuto" disabled>Auto Roll (locked)</button>
          <button id="btnIndex">Index</button>
          <button id="btnInventory">Inventory</button>
        </div>
      </div>

      <!-- Index panel -->
      <div class="panel" id="indexPanel" style="display:none;">
        <h3 style="margin:0 0 10px;display:flex;justify-content:space-between;">
          <span>Index</span><span id="indexCompletion"></span>
        </h3>
        <div class="index-grid" id="indexGrid"></div>
      </div>

      <!-- Inventory panel -->
      <div class="panel" id="inventoryPanel" style="display:none;">
        <div class="inv-header">
          <div class="mode-wrap">
            <span class="mode-label">Mode:</span>
            <div class="mode-carousel">
              <button class="mode-arrow" id="modePrev">‹</button>
              <div class="mode-value" id="modeValue">Rolled</div>
              <button class="mode-arrow" id="modeNext">›</button>
            </div>
          </div>
          <div class="autosell-wrap">
            <span class="autosell-label">Auto-Sell:</span>
            <div class="autosell-carousel">
              <button class="autosell-arrow" id="autoSellPrev">‹</button>
              <div class="autosell-value" id="autoSellValue">Off</div>
              <button class="autosell-arrow" id="autoSellNext">›</button>
            </div>
          </div>
        </div>
        <ul id="inventoryList" class="inv-list"></ul>
      </div>
    </div>
  </div>

  <script>
    /* ---------------- Tiers & Items ---------------- */
    const TIERS=[
      {key:"worthless",name:"Worthless",weight:1200000,colorClass:"b-worthless"},
      {key:"trash",name:"Trash",weight:600000,colorClass:"b-trash"},
      {key:"common",name:"Common",weight:300000,colorClass:"b-common"},
      {key:"uncommon",name:"Uncommon",weight:60000,colorClass:"b-uncommon"},
      {key:"rare",name:"Rare",weight:9000,colorClass:"b-rare"},
      {key:"epic",name:"Epic",weight:1200,colorClass:"b-epic"},
      {key:"legendary",name:"Legendary",weight:160,colorClass:"b-legendary"},
      {key:"mythic",name:"Mythic",weight:40,colorClass:"b-mythic"},
      {key:"divine",name:"Divine",weight:16,colorClass:"b-divine"},
      {key:"celestial",name:"Celestial",weight:8,colorClass:"b-celestial"},
      {key:"transcendent",name:"Transcendent",weight:4,colorClass:"b-transcendent"},
      {key:"eternal",name:"Eternal",weight:2,colorClass:"b-eternal"},
      {key:"omniversal",name:"Omniversal",weight:1,colorClass:"b-omniversal"},
      {key:"exclusive",name:"Exclusive",weight:0,colorClass:"b-exclusive"} // index-only, blurred until discovered
    ];

    const INDEX_ITEMS={
      worthless:["Flicker Dust","Cracked Ash","Dim Mote","Frayed Thread","Worn Chip","Hollow Grain","Stale Ember","Silt Speck","Faded Spark","Withered Flake","Dull Scale","Spent Echo"],
      trash:["Bent Sigil","Scuffed Gear","Fractured Bead","Tarnished Ring","Splintered Token","Bruised Charm","Chipped Prism","Dented Halo","Crater Chip","Scratched Fang","Bruised Petal","Muddled Rune"],
      common:["Ember Shard","Faded Leaf","Whisper Pebble","Ash Fragment","Dull Crystal","Hollow Feather","Broken Gear","Dim Lantern","Rusted Token","Clouded Glass","Forgotten Coin","Waning Shell"],
      uncommon:["Azure Bloom","Twilight Fang","Iron Relic","Verdant Gem","Lunar Petal","Singing Shell","Obsidian Fang","Frosted Charm","Ancient Rune","Shimmering Scale","Echo Stone","Glimmer Root"],
      rare:["Star Fragment","Void Pearl","Radiant Fang","Solar Bloom","Abyssal Shard","Eternal Vine","Frostheart","Ember Crown","Phantom Mask","Spirit Lantern","Dreamcatcher","Arcane Relic"],
      epic:["Dragon’s Heart","Celestial Fang","Prism Core","Eternal Flame","Shadow Crown","Aurora Bloom","Titan’s Fang","Rift Crystal","Cosmic Lantern","Astral Rune","Phoenix Ash","Storm Relic"],
      legendary:["Sol’s Tear","Moonveil Crown","Eternal Star","Riftwalker’s Fang","Celestial Bloom","Radiant Halo","Abyss Crown","Prism Heart","Void Lantern","Chrono Relic","Phoenix Crown","Astral Flame"],
      mythic:["Solstice Crown","Eclipse Heart","Infinity Core","Astral Diadem","Rift Monarch","Nova Grimoire","Umbra Scepter","Parallax Halo","Quasar Thorn","Aether Loom","Aurora Sigil","Prime Catalyst"],
      divine:["Sol’s Core","Eternal Sun","Celestial Orb","Divine Halo","Radiant Crown","Cosmic Tear","Riftheart","Astral Bloom","Phoenix Soul","Chrono Star","Abyssal Flame","Prism Crown"],
      celestial:["Stellar Crown","Singularity Shard","Event Horizon","Nebula Heart","Galactic Sigil","Nova Crown","Comet Ring","Quasar Core","Ecliptic Rune","Parhelion Gem","Aurora Diadem","Aether Crown"],
      transcendent:["Transcendent Eye","Omni Sigil","Hyperion Core","Timeweaver Crest","Axis Heart","Prime Star","Beyond Rune","Unbound Halo","Perennial Flame","Limitless Gem","Meta Crown","Supernal Tear"],
      eternal:["Eternal Bloom","Forever Star","Unending Crown","Ceaseless Orb","Timeless Fang","Endless Prism","Sempiternal Rune","Undying Flame","Ageless Halo","Perpetual Core","Immortal Sigil","Infinite Diadem"],
      omniversal:["Omniversal Heart","All-Crown","Totality Core","Panreality Halo","Absolute Sigil","Everything Rune","Boundless Star","Cosmos Crown","Axis of All","Prime Totality","Universal Eye","Omega Diadem"],
      exclusive:[]
    };

    /* ---------------- Consumables (items appear in Rolled with 3x harder chance) ---------------- */
    // Effects: luck (+%), speed (+%), bias (tier weight +%), guarantee (next roll highest tier)
    const ITEM_DROPS={
      worthless:[
        { name:"Vial of Luck", rarity:"worthless", type:"luck", amount:0.01, duration:40 },
        { name:"Cracked Token of Chance", rarity:"worthless", type:"bias", target:"common", amount:0.05, duration:60 }
      ],
      trash:[
        { name:"Spoiled Luck Potion", rarity:"trash", type:"luck", amount:0.05, duration:60 },
        { name:"Tarnished Coin", rarity:"trash", type:"bias", target:"uncommon", amount:0.05, duration:60 }
      ],
      common:[
        { name:"Luck Potion", rarity:"common", type:"luck", amount:0.25, duration:90 },
        { name:"Charm of Fortune", rarity:"common", type:"bias", target:"rare", amount:0.07, duration:60 }
      ],
      uncommon:[
        { name:"Strong Luck Potion", rarity:"uncommon", type:"luck", amount:0.50, duration:150 },
        { name:"Sigil of Odds", rarity:"uncommon", type:"bias", target:"epic", amount:0.08, duration:60 }
      ],
      rare:[
        { name:"Greater Luck Potion", rarity:"rare", type:"luck", amount:1.00, duration:60 },
        { name:"Greater Luck Charm", rarity:"rare", type:"bias", target:"legendary", amount:0.10, duration:60 }
      ],
      epic:[
        { name:"Epic Luck Potion", rarity:"epic", type:"luck", amount:1.75, duration:40 },
        { name:"Crystal of Providence", rarity:"epic", type:"bias", target:"mythic", amount:0.12, duration:60 }
      ],
      legendary:[
        { name:"Elixir of Destiny", rarity:"legendary", type:"luck", amount:2.50, duration:30 },
        { name:"Relic of Destiny", rarity:"legendary", type:"bias", target:"divine", amount:0.15, duration:60 },
        { name:"Speed Potion", rarity:"legendary", type:"speed", amount:0.25, duration:60 }
      ],
      mythic:[
        { name:"Fatebinder’s Draught", rarity:"mythic", type:"luck", amount:3.75, duration:30 },
        { name:"Fate‑Twister’s Seal", rarity:"mythic", type:"bias", target:"celestial", amount:0.18, duration:60 }
      ],
      divine:[
        { name:"Elixir of Fortune", rarity:"divine", type:"luck", amount:5.00, duration:20 },
        { name:"Blessed Talisman", rarity:"divine", type:"bias", target:"transcendent", amount:0.22, duration:60 },
        { name:"Elixir of Quickening", rarity:"divine", type:"speed", amount:0.50, duration:45 }
      ],
      celestial:[
        { name:"Starlight Elixir", rarity:"celestial", type:"luck", amount:7.50, duration:20 },
        { name:"Starlight Sigil", rarity:"celestial", type:"bias", target:"eternal", amount:0.26, duration:60 }
      ],
      transcendent:[
        { name:"Paradox Brew", rarity:"transcendent", type:"luck", amount:10.00, duration:15 },
        { name:"Paradox Shard", rarity:"transcendent", type:"bias", target:"omniversal", amount:0.30, duration:60 },
        { name:"Godly Potion of Haste", rarity:"transcendent", type:"speed", amount:1.00, duration:30 }
      ],
      eternal:[
        { name:"Godly Potion", rarity:"eternal", type:"luck", amount:15.00, duration:15 },
        { name:"Godly Relic", rarity:"eternal", type:"bias", target:"omniversal", amount:0.50, duration:60 }
      ],
      omniversal:[
        { name:"Origin Draught", rarity:"omniversal", type:"luck", amount:25.00, duration:20 },
        { name:"Origin Draught of Speed", rarity:"omniversal", type:"speed", amount:2.50, duration:25 },
        { name:"Origin Crystal", rarity:"omniversal", type:"guarantee", amount:1, duration:1 }
      ]
    };

    const LUCK_TARGET_KEYS=["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];

    /* ---------------- Persistence ---------------- */
    const STORAGE_KEYS={
      rolls:"sol_rng_rolls",
      unlocks:"sol_rng_unlocks",
      auto:"sol_rng_auto",
      inv:"sol_rng_inventory",
      autoSell:"sol_rng_autosell",
      effects:"sol_rng_effects",
      guarantee:"sol_rng_guarantee",
      mode:"sol_rng_mode",
      exclusiveDiscovered:"sol_rng_exclusive_discovered"
    };

    function buildInitialUnlocks(def){
      const o={}; for(const k in def){ o[k]={}; def[k].forEach(n=>o[k][n]=false); } return o;
    }
    function reviveUnlocks(u){
      const f=buildInitialUnlocks(INDEX_ITEMS);
      for(const k in f){ for(const n of Object.keys(f[k])){ f[k][n]=u[k]&&typeof u[k][n]==="boolean"?u[k][n]:false; } }
      return f;
    }

    /* ---------------- State ---------------- */
    const ROLLED_MAX=10;      // rolled capacity /10
    const ITEMS_MAX=50;       // items capacity /50
    const BASE_AUTO_INTERVAL=120; // ms

    const state={
      rolls:0,
      unlocks:buildInitialUnlocks(INDEX_ITEMS),
      auto:false,
      autoInterval:null,
      inventoryRolled:[],   // only index items; capacity 10
      inventoryItems:[],    // consumables; capacity 50
      autoSell:"off",       // threshold for rolled only (items unaffected)
      fullAnnouncedRolled:false,
      fullAnnouncedItems:false,
      activeEffects:{ luck:0, speed:0, bias:{} }, // additive stacking
      effectTimers:[], // timers for effects
      nextRollGuaranteeHighest:false,
      mode:"Rolled",         // Inventory view mode: Rolled | Items
      exclusiveDiscovered:false
    };

    function loadState(){
      const rolls=parseInt(localStorage.getItem(STORAGE_KEYS.rolls)||"0",10);
      const unlocksRaw=localStorage.getItem(STORAGE_KEYS.unlocks);
      const autoRaw=localStorage.getItem(STORAGE_KEYS.auto);
      const invRaw=localStorage.getItem(STORAGE_KEYS.inv);
      const autoSellRaw=localStorage.getItem(STORAGE_KEYS.autoSell);
      const effectsRaw=localStorage.getItem(STORAGE_KEYS.effects);
      const guaranteeRaw=localStorage.getItem(STORAGE_KEYS.guarantee);
      const modeRaw=localStorage.getItem(STORAGE_KEYS.mode);
      const exclRaw=localStorage.getItem(STORAGE_KEYS.exclusiveDiscovered);

      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      const inv = invRaw?JSON.parse(invRaw):{rolled:[],items:[]};
      state.inventoryRolled = inv.rolled || [];
      state.inventoryItems = inv.items || [];
      state.autoSell=autoSellRaw||"off";
      state.activeEffects=effectsRaw?JSON.parse(effectsRaw):{luck:0,speed:0,bias:{}};
      state.nextRollGuaranteeHighest=guaranteeRaw==="true";
      state.mode = modeRaw || "Rolled";
      state.exclusiveDiscovered = exclRaw==="true";
    }
    function saveState(){
      localStorage.setItem(STORAGE_KEYS.rolls,String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks,JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto,state.auto?"true":"false");
      localStorage.setItem(STORAGE_KEYS.inv,JSON.stringify({rolled:state.inventoryRolled,items:state.inventoryItems}));
      localStorage.setItem(STORAGE_KEYS.autoSell,state.autoSell);
      localStorage.setItem(STORAGE_KEYS.effects,JSON.stringify(state.activeEffects));
      localStorage.setItem(STORAGE_KEYS.guarantee,state.nextRollGuaranteeHighest?"true":"false");
      localStorage.setItem(STORAGE_KEYS.mode,state.mode);
      localStorage.setItem(STORAGE_KEYS.exclusiveDiscovered,state.exclusiveDiscovered?"true":"false");
    }

    /* ---------------- Utils ---------------- */
    function clone(o){ return JSON.parse(JSON.stringify(o)); }
    function sumWeights(a){ return a.reduce((s,r)=>s+r.weight,0); }
    function luckMilestoneForRoll(n){ if(n===250) return 10; if(n===50) return 2; return 1; }

    function spawnBanner(text,type){
      const rollArea=document.getElementById("rollArea");
      const div=document.createElement("div");
      div.className=`banner ${type} fadeout`;
      div.textContent=text;
      div.addEventListener("animationend",()=>div.remove());
      rollArea.appendChild(div);
    }

    function shouldAutoSell(tierKey){
      const order=TIERS.map(t=>t.key); // includes exclusive but we will ignore it via threshold rules
      const thr=state.autoSell;
      if(thr==="off") return false;
      if(tierKey==="exclusive") return false;
      return order.indexOf(tierKey) <= order.indexOf(thr);
    }

    function applyWeightModifiers(baseTiers, milestoneMult){
      const tiers=clone(baseTiers);
      // milestone luck
      if(milestoneMult>1){ for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= milestoneMult; } }
      // consumable luck
      if(state.activeEffects.luck>0){
        const mult = 1 + state.activeEffects.luck;
        for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= mult; }
      }
      // bias
      for(const key in state.activeEffects.bias){
        const amt=state.activeEffects.bias[key];
        const t=tiers.find(x=>x.key===key);
        if(t){ t.weight *= (1+amt); }
      }
      // exclusive stays blurred and not rollable unless discovered by item? (kept unrollabe here)
      return tiers;
    }

    function toChances(tiers){
      const total=sumWeights(tiers);
      return tiers.map(t=>({ ...t, chance: total>0 ? t.weight/total : 0 }));
    }
    function pickTier(chances){
      const r=Math.random(); let acc=0;
      for(let i=chances.length-1;i>=0;i--){
        acc += chances[i].chance;
        if(r<=acc) return chances[i];
      }
      return chances[0];
    }
    function pickIndexItem(tierKey){
      const list=INDEX_ITEMS[tierKey]||[];
      if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* Items chance: appears within rolled flow with 3x harder rate than base tiers.
       We model this by rolling an "item event" coin: small chance per roll, scaled by rarity weights /3. */
    function buildItemTierWeightsFromIndex(baseTiers){
      // use TIERS weights divided by 3 (harder)
      return baseTiers.map(t=>({ ...t, weight: t.weight/3 }));
    }
    function pickConsumableFromTier(tierKey){
      const list=ITEM_DROPS[tierKey]||[];
      if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* ---------------- Effects ---------------- */
    function addEffect(effect){
      // stacking: add amounts; time stacks by scheduling independent timers
      if(effect.type==="luck"){
        state.activeEffects.luck += effect.amount;
        scheduleEffectTimer(()=>{ state.activeEffects.luck -= effect.amount; saveState(); updateAutoInterval(); }, effect.duration);
        updateAutoInterval();
      } else if(effect.type==="speed"){
        state.activeEffects.speed += effect.amount;
        scheduleEffectTimer(()=>{ state.activeEffects.speed -= effect.amount; saveState(); updateAutoInterval(); }, effect.duration);
        updateAutoInterval();
      } else if(effect.type==="bias"){
        const prev=state.activeEffects.bias[effect.target]||0;
        state.activeEffects.bias[effect.target]=prev+effect.amount;
        scheduleEffectTimer(()=>{
          state.activeEffects.bias[effect.target] -= effect.amount;
          if(state.activeEffects.bias[effect.target] <= 0) delete state.activeEffects.bias[effect.target];
          saveState();
        }, effect.duration);
      } else if(effect.type==="guarantee"){
        state.nextRollGuaranteeHighest = true;
        // Consumed on next roll; no timer needed, but add safety 30s expiry
        scheduleEffectTimer(()=>{ state.nextRollGuaranteeHighest=false; saveState(); }, 30);
      }
      saveState();
      renderActiveEffects();
    }

    function scheduleEffectTimer(cb, seconds){
      const id=setTimeout(()=>{ cb(); renderActiveEffects(); }, seconds*1000);
      state.effectTimers.push(id);
    }

    function updateAutoInterval(){
      if(state.autoInterval){ clearInterval(state.autoInterval); state.autoInterval=null; }
      if(state.auto){
        const mult = 1 + (state.activeEffects.speed||0);
        const interval = Math.max(40, Math.round(BASE_AUTO_INTERVAL / mult));
        state.autoInterval = setInterval(()=>{ if(elAutoBtn.disabled){ toggleAuto(); return; } rollOnce(); }, interval);
      }
    }

    function renderActiveEffects(){
      const el=document.getElementById("activeEffects");
      el.innerHTML="";
      // luck
      if(state.activeEffects.luck>0){
        const e=document.createElement("div");
        e.className="effect-entry";
        e.textContent=`Luck +${Math.round(state.activeEffects.luck*100)}%`;
        el.appendChild(e);
      }
      // speed
      if(state.activeEffects.speed>0){
        const e=document.createElement("div");
        e.className="effect-entry";
        e.textContent=`Auto speed +${Math.round(state.activeEffects.speed*100)}%`;
        el.appendChild(e);
      }
      // bias summary
      const keys=Object.keys(state.activeEffects.bias||{});
      if(keys.length){
        const e=document.createElement("div");
        e.className="effect-entry";
        const parts=keys.map(k=>`${k.toUpperCase()} +${Math.round(state.activeEffects.bias[k]*100)}%`);
        e.textContent=`Bias: ${parts.join(", ")}`;
        el.appendChild(e);
      }
      // guarantee flag
      if(state.nextRollGuaranteeHighest){
        const e=document.createElement("div");
        e.className="effect-entry";
        e.textContent=`Next roll guaranteed highest tier`;
        el.appendChild(e);
      }
    }

    /* ---------------- Rolling ---------------- */
    function rollOnce(){
      const upcoming=state.rolls+1;
      const milestone=luckMilestoneForRoll(upcoming);
      if(milestone>1) spawnBanner(`${milestone}x luck activated`,"luck");

      // Build tiers; exclusive not rollable unless discovered (stays zero weight)
      let tiers=TIERS.filter(t=>t.key!=="exclusive");
      tiers = applyWeightModifiers(tiers, milestone);

      // If guarantee flag: force highest tier available
      let pickedTier;
      if(state.nextRollGuaranteeHighest){
        const highest = TIERS.filter(t=>t.key!=="exclusive" && (INDEX_ITEMS[t.key]||[]).length>0).slice(-1)[0];
        pickedTier = highest;
        state.nextRollGuaranteeHighest=false; // consumed
      } else {
        const chances=toChances(tiers);
        pickedTier = pickTier(chances);
      }

      const tierKey = pickedTier.key;
      const tierName = pickedTier.name;

      // Decide if this roll becomes a consumable item instead (3x harder than normal)
      const itemTiers = buildItemTierWeightsFromIndex(TIERS.filter(t=>t.key!=="exclusive"));
      const itemWeighted = applyWeightModifiers(itemTiers, milestone);
      const itemChances = toChances(itemWeighted);
      // Small probability gate for item occurrence: scale by overall rarity distribution
      // We'll use a base 10% chance, modified by luck (items shouldn't be autosold anyway)
      const baseItemChance = 0.10;
      const luckBoost = Math.min(0.50, state.activeEffects.luck * 0.05); // gentle influence
      const rollItem = Math.random() < (baseItemChance + luckBoost);

      state.rolls++;

      let isNew=false;
      let displayName=null;
      let displayRarityClass=null;

      if(rollItem){
        // Pick an item consumable from a tier
        const itemTier = pickTier(itemChances);
        const drop = pickConsumableFromTier(itemTier.key);
        if(drop){
          // Add to items inventory (no auto-sell; capacity 50)
          if(state.inventoryItems.length < ITEMS_MAX){
            state.inventoryItems.push({ type:"consumable", tier:itemTier.key, tierName:itemTier.name, name:drop.name, roll:state.rolls, effect:drop });
            addEffect(drop);
            displayName = drop.name;
            displayRarityClass = TIERS.find(t=>t.key===drop.rarity)?.colorClass || "";
          } else {
            if(!state.fullAnnouncedItems){ spawnBanner(`Items inventory is full ${ITEMS_MAX}/${ITEMS_MAX}`,"announce"); state.fullAnnouncedItems=true; }
            displayName = drop.name;
            displayRarityClass = TIERS.find(t=>t.key===drop.rarity)?.colorClass || "";
          }
        } else {
          // fallback to normal index item
          const itemName = pickIndexItem(tierKey);
          displayName = itemName || tierName;
          displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey, tierName, itemName, milestone);
          if(itemName) isNew = markNew(tierKey, itemName);
        }
      } else {
        // Normal index roll
        const itemName = pickIndexItem(tierKey);
        displayName = itemName || tierName;
        displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
        processIndexItem(tierKey, tierName, itemName, milestone);
        if(itemName) isNew = markNew(tierKey, itemName);
      }

      saveState();

      showGlow();
      renderResult(displayName, tierKey, displayRarityClass);
      if(isNew) spawnBanner(`NEW collected: [${tierName}] ${displayName}`,"new");
      renderButtonsState();
      renderIndex();
      renderInventory();
    }

    function processIndexItem(tierKey, tierName, itemName, milestone){
      // Auto-sell applies only to Rolled (index items), not consumables
      const toKeep = !shouldAutoSell(tierKey);
      if(toKeep){
        if(state.inventoryRolled.length < ROLLED_MAX){
          state.inventoryRolled.push({ type:"index", tier:tierKey, tierName, name:itemName, roll:state.rolls, milestone });
        } else {
          if(!state.fullAnnouncedRolled){ spawnBanner(`Rolled inventory is full ${ROLLED_MAX}/${ROLLED_MAX}`,"announce"); state.fullAnnouncedRolled=true; }
        }
      }
    }
    function markNew(tierKey,itemName){
      if(!state.unlocks[tierKey][itemName]){
        state.unlocks[tierKey][itemName]=true;
        // If exclusive item ever added, discovery toggles blur off (future use)
        if(tierKey==="exclusive") state.exclusiveDiscovered=true;
        return true;
      }
      return false;
    }

    /* ---------------- UI ---------------- */
    const elResult=document.getElementById("resultText");
    const elRarity=document.getElementById("rarityText");
    const elAutoBtn=document.getElementById("btnAuto");
    const elIndexBtn=document.getElementById("btnIndex");
    const elInventoryBtn=document.getElementById("btnInventory");
    const elIndexPanel=document.getElementById("indexPanel");
    const elIndexGrid=document.getElementById("indexGrid");
    const elIndexCompletion=document.getElementById("indexCompletion");
    const elInventoryPanel=document.getElementById("inventoryPanel");
    const elInventoryList=document.getElementById("inventoryList");

    const elAutoSellPrev=document.getElementById("autoSellPrev");
    const elAutoSellNext=document.getElementById("autoSellNext");
    const elAutoSellValue=document.getElementById("autoSellValue");

    const elModePrev=document.getElementById("modePrev");
    const elModeNext=document.getElementById("modeNext");
    const elModeValue=document.getElementById("modeValue");

    function renderResult(name, tierKey, rarityClass){
      // Show only item name on main line
      elResult.textContent = name || "Unknown";
      // Show rarity pill below "Ready to roll" like before
      elRarity.textContent = tierKey ? tierKey.toUpperCase() : "";
      elRarity.className = "rarity badge " + (rarityClass || "");
    }

    function renderButtonsState(){
      if(state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)"; }
      elAutoSellValue.textContent = state.autoSell==="off" ? "Off" : labelForAutoSell(state.autoSell);
      elModeValue.textContent = state.mode;
    }
    function labelForAutoSell(val){ const t=TIERS.find(x=>x.key===val); return t? `${t.name}+` : "Off"; }

    function tierCompletion(key){
      const items=INDEX_ITEMS[key]||[]; const unlocked=items.filter(n=>state.unlocks[key][n]).length;
      const pct=items.length? Math.round(unlocked/items.length*100) : 0;
      return { unlocked, total:items.length, percent:pct };
    }
    function totalCompletion(){
      let u=0,t=0; for(const k in INDEX_ITEMS){ const c=tierCompletion(k); u+=c.unlocked; t+=c.total; }
      return { unlocked:u,total:t,percent: t? Math.round(u/t*100) : 0 };
    }

    function renderIndex(){
      const tot=totalCompletion(); elIndexCompletion.textContent=`Total ${tot.percent}%`;
      elIndexGrid.innerHTML="";
      TIERS.forEach(tier=>{
        const section=document.createElement("div");
        section.className="index-section";
        const comp=tierCompletion(tier.key);
        const badge = `<span class="badge ${tier.colorClass}">${tier.name}</span>`;
        section.innerHTML=`
          <h4>${badge}</h4>
          <div class="completion">${comp.percent}%</div>
        `;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(tier.key==="exclusive"){
          const li=document.createElement("li"); li.className="index-item";
          li.innerHTML=`<span class="${state.exclusiveDiscovered ? "" : "locked"}">${state.exclusiveDiscovered ? "Exclusive discovered items will appear here" : "Secret tier (blurred)"}</span>
                        <span class="${state.exclusiveDiscovered ? "unlocked" : "locked"}">${state.exclusiveDiscovered ? "Unlocked" : "Locked"}</span>`;
          ul.appendChild(li);
        } else if(!items.length){
          const li=document.createElement("li"); li.className="index-item";
          li.innerHTML=`<span class="locked">No items defined</span>`; ul.appendChild(li);
        } else {
          items.forEach(name=>{
            const li=document.createElement("li"); li.className="index-item";
            const unlocked=!!state.unlocks[tier.key][name];
            li.innerHTML=`
              <span class="${unlocked? "": "locked"}">${name}</span>
              <span class="${unlocked? "unlocked": "locked"}">${unlocked? "Unlocked": "Locked"}</span>
            `;
            ul.appendChild(li);
          });
        }
        section.appendChild(ul);
        elIndexGrid.appendChild(section);
      });
    }

    function renderInventory(){
      elInventoryList.innerHTML="";
      if(state.mode==="Rolled"){
        const items=[...state.inventoryRolled].reverse();
        items.forEach(entry=>{
          const tier=TIERS.find(t=>t.key===entry.tier); const badgeClass=tier?tier.colorClass:"";
          const li=document.createElement("li");
          const left=document.createElement("div");
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"} • #${entry.roll}${entry.milestone>1?` • ${entry.milestone}x`:``}`;
          const right=document.createElement("div"); right.className="inv-actions";
          const del=document.createElement("button"); del.textContent="Delete";
          del.addEventListener("click",()=>deleteRolledEntry(entry));
          right.appendChild(del);
          li.appendChild(left); li.appendChild(right);
          elInventoryList.appendChild(li);
        });
        // capacity line appended
        const cap=document.createElement("div");
        cap.className="inv-stats";
        cap.style.marginTop="8px";
        cap.textContent=`Rolled capacity ${state.inventoryRolled.length}/${ROLLED_MAX}`;
        elInventoryList.appendChild(cap);
      } else {
        const items=[...state.inventoryItems].reverse();
        items.forEach(entry=>{
          const tier=TIERS.find(t=>t.key===entry.tier); const badgeClass=tier?tier.colorClass:"";
          const li=document.createElement("li");
          const left=document.createElement("div");
          const e=entry.effect;
          const effColorClass = badgeClass;
          const effDesc = e.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                        : e.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                        : e.type==="bias" ? `Bias → ${e.target.toUpperCase()} +${Math.round(e.amount*100)}%`
                        : e.type==="guarantee" ? `Guarantee highest next roll` : "";
          left.innerHTML=`<span class="badge ${effColorClass}">${entry.tierName}</span> — ${entry.name} (${effDesc}) • #${entry.roll}`;
          const right=document.createElement("div"); right.className="inv-actions";
          const use=document.createElement("button"); use.textContent="Use";
          use.addEventListener("click",()=>useItemEntry(entry));
          const del=document.createElement("button"); del.textContent="Delete";
          del.addEventListener("click",()=>deleteItemEntry(entry));
          right.appendChild(use); right.appendChild(del);
          li.appendChild(left); li.appendChild(right);
          elInventoryList.appendChild(li);
        });
        const cap=document.createElement("div");
        cap.className="inv-stats";
        cap.style.marginTop="8px";
        cap.textContent=`Items capacity ${state.inventoryItems.length}/${ITEMS_MAX}`;
        elInventoryList.appendChild(cap);
      }
    }

    function deleteRolledEntry(entry){
      const order=TIERS.map(t=>t.key);
      const high=order.indexOf(entry.tier) >= order.indexOf("divine");
      if(high){
        const ok=confirm(`Delete "${entry.name}" [${entry.tierName}]?`);
        if(!ok) return;
      }
      const idx=state.inventoryRolled.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){
        state.inventoryRolled.splice(idx,1);
        saveState(); renderInventory();
        state.fullAnnouncedRolled=false;
      }
    }
    function deleteItemEntry(entry){
      const idx=state.inventoryItems.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){
        state.inventoryItems.splice(idx,1);
        saveState(); renderInventory();
        state.fullAnnouncedItems=false;
      }
    }
    function useItemEntry(entry){
      // Using applies effect again (stacking multiplier and time)
      if(entry.effect){ addEffect(entry.effect); spawnBanner(`Used: ${entry.name}`,"announce"); }
      // After use, remove one instance
      deleteItemEntry(entry);
      renderActiveEffects();
    }

    function showGlow(){ const rollArea=document.getElementById("rollArea"); const g=document.createElement("div"); g.className="glow"; rollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Mode & Auto-Sell carousels ---------------- */
    const autoSellOptions=["off","worthless","trash","common","uncommon","rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];
    function setAutoSell(value){ state.autoSell=value; saveState(); renderButtonsState(); }

    const modes=["Rolled","Items"];
    function setMode(value){ state.mode=value; saveState(); renderButtonsState(); renderInventory(); }

    function cycle(list, current, dir){
      const idx=list.indexOf(current);
      let next=idx;
      if(dir<0) next = idx<=0 ? list.length-1 : idx-1;
      else next = idx>=list.length-1 ? 0 : idx+1;
      return list[next];
    }

    /* ---------------- Auto clicker ---------------- */
    function toggleAuto(){
      if(elAutoBtn.disabled) return;
      if(state.auto){
        state.auto=false; clearInterval(state.autoInterval); state.autoInterval=null; elAutoBtn.textContent="Auto Roll: Off";
      } else {
        state.auto=true; elAutoBtn.textContent="Auto Roll: On";
        updateAutoInterval();
      }
      saveState();
    }

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
    document.getElementById("btnAuto").addEventListener("click",toggleAuto);

    elIndexBtn.addEventListener("click",()=>{
      const vis=elIndexPanel.style.display!=="none";
      if(vis){ elIndexPanel.style.display="none"; }
      else { elIndexPanel.style.display="block"; elInventoryPanel.style.display="none"; renderIndex(); }
    });
    elInventoryBtn.addEventListener("click",()=>{
      const vis=elInventoryPanel.style.display!=="none";
      if(vis){ elInventoryPanel.style.display="none"; }
      else { elInventoryPanel.style.display="block"; elIndexPanel.style.display="none"; renderInventory(); }
    });

    elAutoSellPrev.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,state.autoSell,-1)));
    elAutoSellNext.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,state.autoSell,1)));

    elModePrev.addEventListener("click",()=>setMode(cycle(modes,state.mode,-1)));
    elModeNext.addEventListener("click",()=>setMode(cycle(modes,state.mode,1)));

    /* ---------------- Init ---------------- */
    loadState();
    renderButtonsState();
    renderActiveEffects();
    if(state.auto && state.rolls>=50){
      elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On";
      updateAutoInterval();
    } else {
      elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)";
    }
  </script>
</body>
</html>
