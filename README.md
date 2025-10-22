<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Expanded with Consumables, Index, Auto-Sell</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab;
      --accent:#6ea8fe; --gold:#ffd700; --warn:#ff6666;
    }
    body{margin:0;font-family:sans-serif;background:#0e0f13;color:var(--text);display:grid;place-items:center;min-height:100vh;}
    .app{width:980px;max-width:96vw;background:#151822;border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;}

    /* Roll area and banners */
    .roll-area{min-height:240px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .banner {
      position:absolute; left:50%; transform:translateX(-50%);
      font-weight:700; text-align:center; opacity:0; pointer-events:none;
      animation: fadeout 3.6s forwards;
    }
    .banner.luck { top:14px; color:var(--gold); font-size:18px; }
    .banner.new  { top:120px; color:var(--accent); font-size:16px; } /* directly under "Ready to roll" / result */
    .banner.announce { top:-34px; color:var(--warn); font-size:16px; }
    @keyframes fadeout { 0%{opacity:1; filter:blur(0)} 70%{opacity:1;} 100%{opacity:0; filter:blur(4px)} }

    .result{font-size:28px;font-weight:700;text-align:center;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;}

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

    /* Auto-sell chooser anchored to right corner of inventory; compact carousel */
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
        </div>
        <div class="controls">
          <button id="btnRoll">Rolls</button>
          <button id="btnItemRoll">Items</button>
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
          <div class="inv-title">
            <h3 style="margin:0;">Inventory</h3>
            <span class="inv-stats" id="inventoryStats"></span>
          </div>
          <span class="inv-warning" id="invWarning" style="display:none;"></span>

          <!-- Compact Auto-Sell carousel on the right -->
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
      {key:"exclusive",name:"Exclusive",weight:0,colorClass:"b-exclusive"} // index-only, empty and blurred
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
      exclusive:[] // intentionally empty
    };

    /* ---------------- Consumable Items (items mode, 4x rarer base than normal tiers) ---------------- */
    const ITEM_DROPS = {
      worthless: [
        { name:"Vial of Luck", type:"luck", amount:0.01, duration:40 },
        { name:"Cracked Token of Chance", type:"bias", target:"common", amount:0.05, duration:60 }
      ],
      trash: [
        { name:"Spoiled Luck Potion", type:"luck", amount:0.05, duration:60 },
        { name:"Tarnished Coin", type:"bias", target:"uncommon", amount:0.05, duration:60 }
      ],
      common: [
        { name:"Luck Potion", type:"luck", amount:0.25, duration:90 },
        { name:"Charm of Fortune", type:"bias", target:"rare", amount:0.07, duration:60 }
      ],
      uncommon: [
        { name:"Strong Luck Potion", type:"luck", amount:0.50, duration:150 },
        { name:"Sigil of Odds", type:"bias", target:"epic", amount:0.08, duration:60 }
      ],
      rare: [
        { name:"Greater Luck Potion", type:"luck", amount:1.00, duration:60 },
        { name:"Greater Luck Charm", type:"bias", target:"legendary", amount:0.10, duration:60 }
      ],
      epic: [
        { name:"Epic Luck Potion", type:"luck", amount:1.75, duration:40 },
        { name:"Crystal of Providence", type:"bias", target:"mythic", amount:0.12, duration:60 }
      ],
      legendary: [
        { name:"Elixir of Destiny", type:"luck", amount:2.50, duration:30 },
        { name:"Relic of Destiny", type:"bias", target:"divine", amount:0.15, duration:60 },
        { name:"Speed Potion", type:"speed", amount:0.25, duration:60 }
      ],
      mythic: [
        { name:"Fatebinder’s Draught", type:"luck", amount:3.75, duration:30 },
        { name:"Fate‑Twister’s Seal", type:"bias", target:"celestial", amount:0.18, duration:60 }
      ],
      divine: [
        { name:"Elixir of Fortune", type:"luck", amount:5.00, duration:20 },
        { name:"Blessed Talisman", type:"bias", target:"transcendent", amount:0.22, duration:60 },
        { name:"Elixir of Quickening", type:"speed", amount:0.50, duration:45 }
      ],
      celestial: [
        { name:"Starlight Elixir", type:"luck", amount:7.50, duration:20 },
        { name:"Starlight Sigil", type:"bias", target:"eternal", amount:0.26, duration:60 }
      ],
      transcendent: [
        { name:"Paradox Brew", type:"luck", amount:10.00, duration:15 },
        { name:"Paradox Shard", type:"bias", target:"omniversal", amount:0.30, duration:60 },
        { name:"Godly Potion of Haste", type:"speed", amount:1.00, duration:30 }
      ],
      eternal: [
        { name:"Godly Potion", type:"luck", amount:15.00, duration:15 },
        { name:"Godly Relic", type:"bias", target:"omniversal", amount:0.50, duration:60 }
      ],
      omniversal: [
        { name:"Origin Draught", type:"luck", amount:25.00, duration:20 },
        { name:"Origin Crystal", type:"guarantee", amount:1, duration:1 }, // next roll guarantee highest tier
        { name:"Origin Draught of Speed", type:"speed", amount:2.50, duration:25 }
      ]
    };

    /* ---------------- Luck milestones (on exact roll only) ---------------- */
    function luckMilestoneForRoll(n){ if(n===250) return 10; if(n===50) return 2; return 1; }

    /* ---------------- Persistence ---------------- */
    const STORAGE_KEYS={ rolls:"sol_rng_rolls", unlocks:"sol_rng_unlocks", auto:"sol_rng_auto", inv:"sol_rng_inventory", autoSell:"sol_rng_autosell", effects:"sol_rng_effects", guarantee:"sol_rng_guarantee" };
    function buildInitialUnlocks(def){ const o={}; for(const k in def){ o[k]={}; def[k].forEach(n=>o[k][n]=false); } return o; }
    function reviveUnlocks(u){
      const f=buildInitialUnlocks(INDEX_ITEMS);
      for(const k in f){ for(const n of Object.keys(f[k])){ f[k][n]=u[k]&&typeof u[k][n]==="boolean"?u[k][n]:false; } }
      return f;
    }
    function loadState(){
      const rolls=parseInt(localStorage.getItem(STORAGE_KEYS.rolls)||"0",10);
      const unlocksRaw=localStorage.getItem(STORAGE_KEYS.unlocks);
      const autoRaw=localStorage.getItem(STORAGE_KEYS.auto);
      const invRaw=localStorage.getItem(STORAGE_KEYS.inv);
      const autoSellRaw=localStorage.getItem(STORAGE_KEYS.autoSell);
      const effectsRaw=localStorage.getItem(STORAGE_KEYS.effects);
      const guaranteeRaw=localStorage.getItem(STORAGE_KEYS.guarantee);

      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      state.inventory=invRaw?JSON.parse(invRaw):[];
      state.autoSell=autoSellRaw||"off";
      state.activeEffects=effectsRaw?JSON.parse(effectsRaw):{luck:0, speed:0, bias:{}};
      state.nextRollGuaranteeHighest=guaranteeRaw==="true";
    }
    function saveState(){
      localStorage.setItem(STORAGE_KEYS.rolls,String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks,JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto,state.auto?"true":"false");
      localStorage.setItem(STORAGE_KEYS.inv,JSON.stringify(state.inventory));
      localStorage.setItem(STORAGE_KEYS.autoSell,state.autoSell);
      localStorage.setItem(STORAGE_KEYS.effects,JSON.stringify(state.activeEffects));
      localStorage.setItem(STORAGE_KEYS.guarantee,state.nextRollGuaranteeHighest?"true":"false");
    }

    /* ---------------- State ---------------- */
    const INVENTORY_MAX=50; // items capacity /50
    const BASE_AUTO_INTERVAL=120; // ms
    const state={
      rolls:0,
      unlocks:buildInitialUnlocks(INDEX_ITEMS),
      auto:false,
      autoInterval:null,
      inventory:[],
      autoSell:"off",
      fullAnnounced:false,
      activeEffects:{ luck:0, speed:0, bias:{} }, // luck: +% (additive), speed: +% (additive), bias: {tierKey: +weight%}
      nextRollGuaranteeHighest:false
    };

    /* ---------------- Utils ---------------- */
    function clone(o){ return JSON.parse(JSON.stringify(o)); }
    function sumWeights(a){ return a.reduce((s,r)=>s+r.weight,0); }
    function pickTier(chances){
      const r=Math.random(); let acc=0;
      for(let i=chances.length-1;i>=0;i--){ acc+=chances[i].chance; if(r<=acc) return { index:i, item:chances[i] }; }
      return { index:0, item:chances[0] };
    }
    function pickItemNameForTier(key){
      const list=INDEX_ITEMS[key]||[]; if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* ---------------- Weight adjustments (luck milestone, consumable luck, bias, guarantee) ---------------- */
    const LUCK_TARGET_KEYS=["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];
    function computeWeightedTiersForItemRoll(){
      // Items are 4x rarer: reduce weights by 4 compared to TIERS
      return TIERS.map(t=>{
        const base = t.weight / 4;
        return { ...t, weight: base };
      });
    }
    function applyAllWeightModifiers(baseTiers, milestoneMult){
      const tiers=clone(baseTiers);
      // 1) Milestone luck (on exact 50/250)
      if(milestoneMult>1){
        for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= milestoneMult; }
      }
      // 2) Consumable luck (additive percent: +25% => *1.25)
      if(state.activeEffects.luck>0){
        const mult = 1 + state.activeEffects.luck;
        for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= mult; }
      }
      // 3) Bias weights toward target tiers (additive percent on that tier only)
      if(state.activeEffects.bias){
        for(const key in state.activeEffects.bias){
          const amt = state.activeEffects.bias[key];
          const t = tiers.find(x=>x.key===key);
          if(t){ t.weight *= (1 + amt); }
        }
      }
      // 4) Guarantee highest tier (next roll only)
      if(state.nextRollGuaranteeHighest){
        // Find highest tier that has weight > 0 and at least one item in index
        const eligible = TIERS.filter(t=>t.key!=="exclusive" && (INDEX_ITEMS[t.key]||[]).length>0);
        const highest = eligible[eligible.length-1];
        for(const t of tiers){ t.weight = (t.key===highest.key ? 1 : 0); }
      }
      return tiers;
    }
    function toChances(tiers){ const total=sumWeights(tiers); return tiers.map(t=>({ ...t, chance: total>0? (t.weight/total) : 0 })); }

    /* ---------------- Banners (ephemeral, persist across quick re-rolls) ---------------- */
    const rollArea=document.getElementById("rollArea");
    function spawnBanner(text,type){
      const div=document.createElement("div");
      div.className=`banner ${type}`;
      div.textContent=text;
      div.addEventListener("animationend",()=>div.remove());
      rollArea.appendChild(div);
      void div.offsetWidth;
      div.style.animation="fadeout 3.6s forwards";
    }
    function showLuckBanner(mult){ if(mult>1) spawnBanner(`${mult}x luck activated`,"luck"); }
    function showNewBanner(tierName,itemName){ spawnBanner(`NEW collected: [${tierName}] ${itemName}`,"new"); }
    function showAnnouncement(text){ spawnBanner(text,"announce"); }

    /* ---------------- Auto-sell ---------------- */
    function shouldAutoSell(tierKey){
      const order=TIERS.map(t=>t.key);
      const thr=state.autoSell;
      if(thr==="off") return false;
      return order.indexOf(tierKey) <= order.indexOf(thr);
    }

    /* ---------------- Effects application & timers ---------------- */
    const effectTimers = []; // store timer ids
    function addEffect(effect){
      // Stacking: additively accumulate
      if(effect.type==="luck"){
        state.activeEffects.luck += effect.amount; // e.g., +0.25
        const id=setTimeout(()=>{ state.activeEffects.luck -= effect.amount; saveState(); updateAutoInterval(); }, effect.duration*1000);
        effectTimers.push(id);
      } else if(effect.type==="speed"){
        state.activeEffects.speed += effect.amount; // e.g., +1.00 => 100% faster
        const id=setTimeout(()=>{ state.activeEffects.speed -= effect.amount; saveState(); updateAutoInterval(); }, effect.duration*1000);
        effectTimers.push(id);
        updateAutoInterval();
      } else if(effect.type==="bias"){
        const prev = state.activeEffects.bias[effect.target]||0;
        state.activeEffects.bias[effect.target] = prev + effect.amount;
        const id=setTimeout(()=>{
          state.activeEffects.bias[effect.target] -= effect.amount;
          if(state.activeEffects.bias[effect.target] <= 0) delete state.activeEffects.bias[effect.target];
          saveState();
        }, effect.duration*1000);
        effectTimers.push(id);
      } else if(effect.type==="guarantee"){
        state.nextRollGuaranteeHighest = true;
        const id=setTimeout(()=>{ state.nextRollGuaranteeHighest=false; saveState(); }, 1*1000); // one roll; safety timeout
        effectTimers.push(id);
      }
      saveState();
    }
    function updateAutoInterval(){
      // Recompute interval based on speed effects
      if(state.autoInterval){
        clearInterval(state.autoInterval);
        state.autoInterval = null;
      }
      if(state.auto){
        const mult = 1 + (state.activeEffects.speed || 0);
        const interval = Math.max(40, Math.round(BASE_AUTO_INTERVAL / mult));
        state.autoInterval = setInterval(()=>{
          if(elAutoBtn.disabled){ toggleAuto(); return; }
          rollOnce();
        }, interval);
      }
    }

    /* ---------------- Roll logic (normal index items) ---------------- */
    function rollOnce(){
      const upcoming=state.rolls+1;
      const milestone=luckMilestoneForRoll(upcoming);
      showLuckBanner(milestone);

      const weighted = applyAllWeightModifiers(TIERS, milestone);
      const chances = toChances(weighted.filter(t=>t.key!=="exclusive")); // exclude exclusive from rolling
      const tierPick = pickTier(chances);
      const key = tierPick.item.key, tierName = tierPick.item.name;
      const itemName = pickItemNameForTier(key);

      state.rolls++;

      // Index unlock
      let isNew=false;
      if(itemName && !state.unlocks[key][itemName]){
        state.unlocks[key][itemName]=true;
        isNew=true;
      }

      // Inventory add (subject to auto-sell and capacity)
      if(!shouldAutoSell(key)){
        if(state.inventory.length<INVENTORY_MAX){
          state.inventory.push({ type:"index", tier:key, tierName, name:itemName, roll:state.rolls, milestone, luckBuff:state.activeEffects.luck });
        } else {
          if(!state.fullAnnounced){ showAnnouncement(`Inventory is full ${INVENTORY_MAX}/${INVENTORY_MAX}`); state.fullAnnounced=true; }
        }
      }
      if(state.inventory.length<INVENTORY_MAX) state.fullAnnounced=false;

      // If guarantee flag was set, consume it after this roll
      if(state.nextRollGuaranteeHighest){ state.nextRollGuaranteeHighest=false; }

      saveState();

      showGlow();
      // Show only the item name
      renderResultName(itemName || tierName);
      // New banner
      if(isNew) showNewBanner(tierName,itemName);
      renderButtonsState();
      renderIndex();
      renderInventory();
    }

    /* ---------------- Item roll logic (consumables) ---------------- */
    function itemRollOnce(){
      const upcoming=state.rolls+1;
      const milestone=luckMilestoneForRoll(upcoming);
      showLuckBanner(milestone);

      const baseItemTiers = computeWeightedTiersForItemRoll();
      const weighted = applyAllWeightModifiers(baseItemTiers, milestone);
      const chances = toChances(weighted.filter(t=>ITEM_DROPS[t.key])); // only tiers with consumables

      const tierPick = pickTier(chances);
      const key = tierPick.item.key;
      const tierName = TIERS.find(t=>t.key===key).name;

      const list = ITEM_DROPS[key] || [];
      const drop = list[Math.floor(Math.random()*list.length)];

      state.rolls++;

      // Add consumable to inventory (stackable effects)
      if(state.inventory.length<INVENTORY_MAX){
        state.inventory.push({ type:"consumable", tier:key, tierName, name:drop.name, roll:state.rolls, effect:drop });
        // Apply effect immediately
        applyConsumable(drop);
      } else {
        if(!state.fullAnnounced){ showAnnouncement(`Inventory is full ${INVENTORY_MAX}/${INVENTORY_MAX}`); state.fullAnnounced=true; }
      }
      if(state.inventory.length<INVENTORY_MAX) state.fullAnnounced=false;

      // If guarantee flag was set, consume it after this roll
      if(state.nextRollGuaranteeHighest){ state.nextRollGuaranteeHighest=false; }

      saveState();

      showGlow();
      renderResultName(drop.name);
      renderButtonsState();
      renderIndex();
      renderInventory();
    }

    function applyConsumable(drop){
      if(drop.type==="luck"){
        addEffect({ type:"luck", amount:drop.amount, duration:drop.duration });
        spawnBanner(`Luck +${Math.round(drop.amount*100)}% for ${drop.duration}s`,"luck");
      } else if(drop.type==="speed"){
        addEffect({ type:"speed", amount:drop.amount, duration:drop.duration });
        spawnBanner(`Auto speed +${Math.round(drop.amount*100)}% for ${drop.duration}s`,"announce");
      } else if(drop.type==="bias"){
        addEffect({ type:"bias", target:drop.target, amount:drop.amount, duration:drop.duration });
        spawnBanner(`Bias toward ${drop.target.toUpperCase()} for ${drop.duration}s`,"announce");
      } else if(drop.type==="guarantee"){
        addEffect({ type:"guarantee", amount:1, duration:1 });
        spawnBanner(`Next roll guaranteed highest tier`,"announce");
      }
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

    /* ---------------- UI elements ---------------- */
    const elResult=document.getElementById("resultText");
    const elRarity=document.getElementById("rarityText");
    const elAutoBtn=document.getElementById("btnAuto");
    const elIndexBtn=document.getElementById("btnIndex");
    const elInventoryBtn=document.getElementById("btnInventory");
    const elIndexPanel=document.getElementById("indexPanel");
    const elIndexGrid=document.getElementById("indexGrid");
    const elIndexCompletion=document.getElementById("indexCompletion");
    const elInventoryPanel=document.getElementById("inventoryPanel");
    const elInventoryStats=document.getElementById("inventoryStats");
    const elInventoryList=document.getElementById("inventoryList");
    const elInvWarning=document.getElementById("invWarning");
    const elAutoSellPrev=document.getElementById("autoSellPrev");
    const elAutoSellNext=document.getElementById("autoSellNext");
    const elAutoSellValue=document.getElementById("autoSellValue");
    const elRollBtn=document.getElementById("btnRoll");
    const elItemRollBtn=document.getElementById("btnItemRoll");

    function renderResultName(name){
      elResult.textContent = name;
      // Keep rarity pill reflecting last tier if available (optional: keep previous)
      // We'll clear rarity text to avoid clutter since result shows only name.
      elRarity.textContent = "";
      elRarity.className = "rarity";
    }

    function renderButtonsState(){
      if(state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)"; }
      elAutoSellValue.textContent = state.autoSell==="off" ? "Off" : labelForAutoSell(state.autoSell);
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
        // Exclusive blurred header
        const nameBadge = `<span class="badge ${tier.colorClass}">${tier.name}</span>`;
        section.innerHTML=`
          <h4>${nameBadge}</h4>
          <div class="completion">${comp.percent}%</div>
        `;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(tier.key==="exclusive"){
          const li=document.createElement("li"); li.className="index-item";
          li.innerHTML=`<span class="locked">Coming soon</span><span class="locked">Locked</span>`;
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
      elInventoryStats.textContent=`Capacity ${state.inventory.length}/${INVENTORY_MAX}`;
      elInvWarning.style.display = state.inventory.length>=INVENTORY_MAX ? "inline" : "none";
      elInvWarning.textContent = state.inventory.length>=INVENTORY_MAX ? "Full" : "";

      elInventoryList.innerHTML="";
      const items=[...state.inventory].reverse();
      items.forEach(entry=>{
        const tier=TIERS.find(t=>t.key===entry.tier); const badgeClass=tier?tier.colorClass:"";
        const li=document.createElement("li");
        const left=document.createElement("div");

        if(entry.type==="index"){
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"} • #${entry.roll}${entry.milestone>1?` • ${entry.milestone}x`:``}`;
        } else if(entry.type==="consumable"){
          const e=entry.effect;
          const effDesc = e.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                        : e.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                        : e.type==="bias" ? `Bias → ${e.target.toUpperCase()} +${Math.round(e.amount*100)}%`
                        : e.type==="guarantee" ? `Guarantee highest next roll`
                        : "";
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name} (${effDesc}) • #${entry.roll}`;
        }

        const right=document.createElement("div"); right.className="inv-actions";
        const del=document.createElement("button"); del.textContent="Delete";
        del.addEventListener("click",()=>deleteInventoryEntry(entry));
        right.appendChild(del);
        li.appendChild(left); li.appendChild(right);
        elInventoryList.appendChild(li);
      });
    }

    function deleteInventoryEntry(entry){
      const order=TIERS.map(t=>t.key);
      const high=order.indexOf(entry.tier) >= order.indexOf("divine");
      if(high){
        const ok=confirm(`Delete "${entry.name}" [${entry.tierName}]?`);
        if(!ok) return;
      }
      const idx=state.inventory.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){
        state.inventory.splice(idx,1);
        saveState(); renderInventory();
        if(state.inventory.length<INVENTORY_MAX) state.fullAnnounced=false;
      }
    }

    function showGlow(){ const g=document.createElement("div"); g.className="glow"; rollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Auto-Sell carousel ---------------- */
    const autoSellOptions=["off","worthless","trash","common","uncommon","rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];
    function setAutoSell(value){
      state.autoSell=value; saveState(); renderButtonsState();
    }
    function cycleAutoSell(dir){
      const idx=autoSellOptions.indexOf(state.autoSell);
      let next = idx;
      if(dir<0){ next = (idx<=0? autoSellOptions.length-1 : idx-1); }
      else { next = (idx>=autoSellOptions.length-1? 0 : idx+1); }
      setAutoSell(autoSellOptions[next]);
    }

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
    document.getElementById("btnItemRoll").addEventListener("click",itemRollOnce);
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

    // Auto-sell carousel arrows
    elAutoSellPrev.addEventListener("click",()=>cycleAutoSell(-1));
    elAutoSellNext.addEventListener("click",()=>cycleAutoSell(1));

    /* ---------------- Init ---------------- */
    loadState();
    renderButtonsState();
    if(state.auto && state.rolls>=50){
      elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On";
      updateAutoInterval();
    } else {
      elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)";
    }
  </script>
</body>
</html>
