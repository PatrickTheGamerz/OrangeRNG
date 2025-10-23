<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Weather, Totems, Eclipse & Cosmic Gems</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab;
      --accent:#6ea8fe; --gold:#ffd700; --warn:#ff6666; --weather:#4ec3ff;
    }
    body{margin:0;font-family:sans-serif;background:var(--bg);color:var(--text);display:grid;place-items:center;min-height:100vh;overflow:hidden;}
    .app{width:980px;max-width:96vw;background:var(--panel);border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;position:relative;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;position:relative;z-index:2;}

    /* Roll area and banners */
    .roll-area{min-height:300px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .result{font-size:28px;font-weight:700;text-align:center;z-index:2;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;z-index:2;}
    .banner{position:absolute;left:50%;transform:translateX(-50%);font-weight:700;text-align:center;opacity:0;pointer-events:none;padding:6px 10px;border-radius:10px;z-index:3;}
    .banner.luck{top:14px;font-size:18px;color:var(--gold);}
    .banner.new{top:120px;color:var(--accent);font-size:16px;}
    .banner.announce{top:14px;font-size:16px;border:1px solid #2a3449;background:#1b2232;}
    .banner.weather{top:14px;font-size:18px;color:var(--weather);}
    .fadeout{animation:fadeout 3.6s forwards;}
    @keyframes fadeout{0%{opacity:1;}70%{opacity:1;}100%{opacity:0;filter:blur(4px)}}

    /* Active effects bottom-right (stacked vertically, slightly transparent) */
    .active-effects{
      position:absolute;bottom:10px;right:10px;font-size:12px;text-align:right;max-width:46%;
      display:flex;flex-direction:column;align-items:flex-end;gap:4px;z-index:3;
    }
    .effect-entry{
      font-weight:600;display:block;padding:2px 8px;border-radius:8px;
      border:1px solid #2a3449;background:rgba(27,34,50,0.55);
      backdrop-filter: blur(6px);
    }

    /* Controls */
    .controls{display:flex;gap:12px;padding-top:12px;flex-wrap:wrap;align-items:center;}
    button{background:#1b2232;color:var(--text);border:1px solid #2a3449;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600;transition:background .2s, box-shadow .2s, transform .06s;}
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
    .inv-stats{font-size:13px;color:var(--muted);}
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
    .autosell-value{min-width:160px;text-align:center;padding:8px 10px;border:1px solid #2a3449;border-radius:8px;background:#1b2232;}

    /* Rarity colors */
    .b-worthless{background:#1a1a1a;color:#b3b3b3;} .b-trash{background:#211616;color:#d28f8f;}
    .b-common{background:#1f2635;color:#c7d1e5;} .b-uncommon{background:#14261d;color:#7ae08f;}
    .b-rare{background:#221a35;color:#caa6ff;} .b-epic{background:#10233a;color:#7bb7ff;}
    .b-legendary{background:#2a1f12;color:#ffbf66;} .b-mythic{background:#2a142a;color:#ff7ee6;}
    .b-divine{background:#2a2a12;color:#ffd700;} .b-celestial{background:#142a2a;color:#7affff;}
    .b-transcendent{background:#1a1a2f;color:#a0a7ff;} .b-eternal{background:#1a2f2a;color:#9cf2c7;}
    .b-omniversal{background:#2f1a2f;color:#ff9cff;}
    .b-exclusive{background:linear-gradient(270deg,red,orange,yellow,green,blue,indigo,violet);background-size:400% 400%;animation:rainbow 6s linear infinite;color:white;}
    .b-eclipse{background:#120f22;color:#a38bff;}
    .b-cosmic{background:#0f1a22;color:#7affff;}
    @keyframes rainbow{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}

    /* Glow effect */
    .glow{position:absolute;inset:-40%;border-radius:50%;background:radial-gradient(closest-side,rgba(110,168,254,0.25),transparent 65%);filter:blur(12px);animation:glow 1.1s ease-out forwards;z-index:2;}
    @keyframes glow{0%{opacity:0;transform:scale(0.7)}50%{opacity:1}100%{opacity:0;transform:scale(1.2)}}

    /* Weather background effects: layered, animated */
    .weather-bg{position:absolute;inset:0;z-index:1;pointer-events:none;}
    /* Storm: moving clouds + lightning flashes */
    .w-storm{background:
      radial-gradient(circle at 30% 20%,rgba(255,255,255,0.04),transparent 40%),
      radial-gradient(circle at 70% 30%,rgba(255,255,255,0.03),transparent 50%),
      linear-gradient(180deg,rgba(10,12,18,0.8),rgba(10,12,18,0.9));
      animation:cloudDrift 60s linear infinite;
    }
    @keyframes cloudDrift{from{background-position:0 0,0 0,0 0}to{background-position:200% 100%, -200% 100%,0 0}}
    .w-storm::after{
      content:""; position:absolute; inset:0;
      background:radial-gradient(circle at 50% 30%,rgba(255,255,255,0.0),rgba(255,255,255,0.0) 30%,rgba(255,255,255,0.6) 31%,rgba(255,255,255,0.0) 32%);
      animation:lightning 7s infinite;
    }
    @keyframes lightning{0%,92%{opacity:0}93%{opacity:0.8}94%{opacity:0}98%{opacity:0.6}100%{opacity:0}}

    /* Blizzard: drifting snow layers */
    .w-blizzard{
      background:linear-gradient(180deg,rgba(220,230,255,0.04),rgba(10,12,18,0.9));
    }
    .w-blizzard::before,.w-blizzard::after{
      content:""; position:absolute; inset:0;
      background-image:
        radial-gradient(2px 2px at 20% 20%,rgba(255,255,255,0.8),transparent),
        radial-gradient(2px 2px at 40% 60%,rgba(255,255,255,0.8),transparent),
        radial-gradient(2px 2px at 80% 30%,rgba(255,255,255,0.8),transparent),
        radial-gradient(2px 2px at 60% 80%,rgba(255,255,255,0.8),transparent);
      background-repeat:repeat;
      animation:snowDrift 18s linear infinite;
    }
    .w-blizzard::after{animation-duration:26s;opacity:0.7;}
    @keyframes snowDrift{from{background-position:0 0}to{background-position:-200% 200%}}

    /* Eclipse: vignette + subtle corona */
    .w-eclipse{
      background:
        radial-gradient(circle at 50% 50%,rgba(0,0,0,0.75),rgba(0,0,0,0.9) 38%,rgba(0,0,0,1) 60%),
        radial-gradient(circle at 50% 50%,rgba(140,120,255,0.15),transparent 50%);
      animation:eclipsePulse 12s ease-in-out infinite;
    }
    @keyframes eclipsePulse{0%,100%{filter:brightness(0.9)}50%{filter:brightness(1.05)}}

    /* Cosmic Tempest: swirling nebula */
    .w-cosmic{
      background:
        radial-gradient(closest-side,rgba(128,0,255,0.25),transparent 65%),
        conic-gradient(from 0deg,rgba(128,0,255,0.25),rgba(0,255,255,0.25),rgba(255,0,160,0.25),rgba(128,0,255,0.25));
      animation:cosmicSpin 40s linear infinite;
    }
    @keyframes cosmicSpin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
  </style>
</head>
<body>
  <div class="app">
    <div id="weatherBg" class="weather-bg"></div>
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
    /* ---------------- DOM Handles (first) ---------------- */
    const elRollArea = document.getElementById("rollArea");
    const elResult = document.getElementById("resultText");
    const elRarity = document.getElementById("rarityText");
    const elActiveEffects = document.getElementById("activeEffects");
    const elWeatherBg = document.getElementById("weatherBg");

    const elBtnRoll = document.getElementById("btnRoll");
    const elBtnAuto = document.getElementById("btnAuto");
    const elBtnIndex = document.getElementById("btnIndex");
    const elBtnInventory = document.getElementById("btnInventory");

    const elIndexPanel = document.getElementById("indexPanel");
    const elIndexGrid = document.getElementById("indexGrid");
    const elIndexCompletion = document.getElementById("indexCompletion");

    const elInventoryPanel = document.getElementById("inventoryPanel");
    const elInventoryList = document.getElementById("inventoryList");

    const elAutoSellPrev = document.getElementById("autoSellPrev");
    const elAutoSellNext = document.getElementById("autoSellNext");
    const elAutoSellValue = document.getElementById("autoSellValue");

    const elModePrev = document.getElementById("modePrev");
    const elModeNext = document.getElementById("modeNext");
    const elModeValue = document.getElementById("modeValue");

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
      {key:"exclusive",name:"Exclusive",weight:0,colorClass:"b-exclusive"} // event drops only
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
      exclusive:["Eclipse Gem","Cosmic Gem"] // event-only
    };

    /* ---------------- Consumables & Totems ---------------- */
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
        { name:"Charm of Fortune", rarity:"common", type:"bias", target:"rare", amount:0.07, duration:60 },
        { name:"Storm Totem", rarity:"legendary", type:"totem", weather:"Storm" },
        { name:"Blizzard Totem", rarity:"legendary", type:"totem", weather:"Blizzard" },
        { name:"Sun Totem", rarity:"legendary", type:"totem", weather:"Sunny Radiance" },
        { name:"Random Event Totem", rarity:"divine", type:"totem_random" }
      ],
      uncommon:[
        { name:"Strong Luck Potion", rarity:"uncommon", type:"luck", amount:0.50, duration:150 },
        { name:"Sigil of Odds", rarity:"uncommon", type:"bias", target:"epic", amount:0.08, duration:60 }
      ],
      rare:[
        { name:"Greater Luck Potion", rarity:"rare", type:"luck", amount:1.00, duration:60 },
        { name:"Greater Luck Charm", rarity:"rare", type:"bias", target:"legendary", amount:0.10, duration:60 },
        { name:"Meteor Storm Totem", rarity:"divine", type:"totem", weather:"Meteor Storm" },
        { name:"Aurora Veil Totem", rarity:"divine", type:"totem", weather:"Aurora Veil" }
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
        { name:"Godly Potion of Haste", rarity:"transcendent", type:"speed", amount:1.00, duration:30 },
        { name:"Eternal Eclipse Totem", rarity:"transcendent", type:"totem", weather:"Eternal Eclipse" },
        { name:"Cosmic Tempest Totem", rarity:"transcendent", type:"totem", weather:"Cosmic Tempest" }
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

    /* ---------------- Weather definitions (with stat effects) ---------------- */
    const WEATHERS = {
      normal: [
        {name:"Storm", colorClass:"b-legendary", effect:{type:"luck",amount:0.20}, bg:"w-storm"},
        {name:"Blizzard", colorClass:"b-legendary", effect:{type:"luck",amount:0.30}, bg:"w-blizzard"},
        {name:"Sunny Radiance", colorClass:"b-legendary", effect:{type:"bias",target:"rare",amount:0.10}, bg:""}
      ],
      rare: [
        {name:"Meteor Storm", colorClass:"b-divine", effect:{type:"luck",amount:0.50}, bg:"w-storm"},
        {name:"Aurora Veil", colorClass:"b-divine", effect:{type:"bias",target:"epic",amount:0.20}, bg:""}
      ],
      super: [
        {name:"Eternal Eclipse", colorClass:"b-transcendent", effect:{type:"bias",target:"transcendent",amount:0.30}, bg:"w-eclipse", specialDropKey:"Eclipse Gem"},
        {name:"Cosmic Tempest", colorClass:"b-transcendent", effect:{type:"bias",target:"omniversal",amount:0.30}, bg:"w-cosmic", specialDropKey:"Cosmic Gem"}
      ]
    };

    /* ---------------- Persistence ---------------- */
    const STORAGE_KEYS={
      rolls:"sol_rng_rolls",
      unlocks:"sol_rng_unlocks",
      auto:"sol_rng_auto",
      inv:"sol_rng_inventory",
      autoSellRolled:"sol_rng_autosell_rolled",
      autoSellItems:"sol_rng_autosell_items",
      effects:"sol_rng_effects_instances",
      mode:"sol_rng_mode"
    };

    function buildInitialUnlocks(def){
      const o={}; for(const k in def){ o[k]={}; def[k].forEach(n=>o[k][n]=false); } return o;
    }
    function reviveUnlocks(u){
      const f=buildInitialUnlocks(INDEX_ITEMS);
      for(const k in f){ for(const n of Object.keys(f[k])){ f[k][n]=u&&u[k]&&typeof u[k][n]==="boolean"?u[k][n]:false; } }
      return f;
    }

    /* ---------------- State ---------------- */
    const ROLLED_MAX=10;
    const ITEMS_MAX=50;
    const BASE_AUTO_INTERVAL=120; // ms
    const MAX_EFFECT_SECONDS=250;

    const state={
      rolls:0,
      unlocks:buildInitialUnlocks(INDEX_ITEMS),
      auto:false,
      autoInterval:null,
      inventoryRolled:[],
      inventoryItems:[],
      autoSellRolled:"off",
      autoSellItems:"off",
      fullAnnouncedRolled:false,
      fullAnnouncedItems:false,
      activeEffects:{ luck:0, speed:0, bias:{} },
      effectInstances:[], // {name,type,amount,target?,expiresAt,rarityKey,weather?,bg?}
      mode:"Rolled"
    };

    function loadState(){
      const rolls=parseInt(localStorage.getItem(STORAGE_KEYS.rolls)||"0",10);
      const unlocksRaw=localStorage.getItem(STORAGE_KEYS.unlocks);
      const autoRaw=localStorage.getItem(STORAGE_KEYS.auto);
      const invRaw=localStorage.getItem(STORAGE_KEYS.inv);
      const autoSellRolledRaw=localStorage.getItem(STORAGE_KEYS.autoSellRolled);
      const autoSellItemsRaw=localStorage.getItem(STORAGE_KEYS.autoSellItems);
      const effInstRaw=localStorage.getItem(STORAGE_KEYS.effects);
      const modeRaw=localStorage.getItem(STORAGE_KEYS.mode);

      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      const inv = invRaw?JSON.parse(invRaw):{rolled:[],items:[]};
      state.inventoryRolled = inv.rolled || [];
      state.inventoryItems = inv.items || [];
      state.autoSellRolled=autoSellRolledRaw||"off";
      state.autoSellItems=autoSellItemsRaw||"off";
      state.effectInstances = effInstRaw ? JSON.parse(effInstRaw) : [];
      const now=Date.now();
      state.effectInstances = state.effectInstances.filter(e=>e.expiresAt>now);
      deriveEffectsTotals();
      state.mode = modeRaw || "Rolled";
    }
    function saveState(){
      localStorage.setItem(STORAGE_KEYS.rolls,String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks,JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto,state.auto?"true":"false");
      localStorage.setItem(STORAGE_KEYS.inv,JSON.stringify({rolled:state.inventoryRolled,items:state.inventoryItems}));
      localStorage.setItem(STORAGE_KEYS.autoSellRolled,state.autoSellRolled);
      localStorage.setItem(STORAGE_KEYS.autoSellItems,state.autoSellItems);
      localStorage.setItem(STORAGE_KEYS.effects,JSON.stringify(state.effectInstances));
      localStorage.setItem(STORAGE_KEYS.mode,state.mode);
    }

    /* ---------------- Utils ---------------- */
    function clone(o){ return JSON.parse(JSON.stringify(o)); }
    function sumWeights(a){ return a.reduce((s,r)=>s+r.weight,0); }
    function luckMilestoneForRoll(n){ if(n===200) return 10; if(n===50) return 2; return 1; }

    function spawnBanner(text,type,colorClass){
      const div=document.createElement("div");
      const useType = type==="activate" ? "luck" : type;
      div.className=`banner ${useType} fadeout ${colorClass?colorClass:''}`;
      div.textContent=text;
      div.addEventListener("animationend",()=>div.remove());
      elRollArea.appendChild(div);
    }

    function shouldAutoSellRolled(tierKey){
      const order=TIERS.map(t=>t.key);
      const thr=state.autoSellRolled;
      if(thr==="off") return false;
      if(tierKey==="exclusive") return false;
      return order.indexOf(tierKey) <= order.indexOf(thr);
    }
    function shouldAutoSellItems(tierKey){
      const order=TIERS.map(t=>t.key);
      const thr=state.autoSellItems;
      if(thr==="off") return false;
      if(tierKey==="exclusive") return false;
      return order.indexOf(tierKey) <= order.indexOf(thr);
    }

    function applyWeightModifiers(baseTiers, milestoneMult){
      const tiers=clone(baseTiers);
      if(milestoneMult>1){ for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= milestoneMult; } }
      if(state.activeEffects.luck>0){
        const mult = 1 + state.activeEffects.luck;
        for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= mult; }
      }
      for(const key in state.activeEffects.bias){
        const amt=state.activeEffects.bias[key];
        const t=tiers.find(x=>x.key===key);
        if(t){ t.weight *= (1+amt); }
      }
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

    /* Items: extremely rare. Shift +2 rarity weights and hard-divide by 6000 */
    function buildItemTierWeightsShifted(baseTiers){
      return baseTiers.map((t,i)=>{
        const shiftedIndex = Math.min(baseTiers.length-1, i+2); // +2 rarity
        const shiftedWeight = baseTiers[shiftedIndex].weight;
        return { ...t, weight: shiftedWeight / 6000 };
      });
    }

    function pickConsumableFromTier(tierKey){
      const list=ITEM_DROPS[tierKey]||[];
      if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* ---------------- Effects (timed instances + derived totals) ---------------- */
    function deriveEffectsTotals(){
      const totals={ luck:0, speed:0, bias:{} };
      const now=Date.now();
      for(const e of state.effectInstances){
        if(e.expiresAt <= now) continue;
        if(e.type==="luck") totals.luck += e.amount;
        else if(e.type==="speed") totals.speed += e.amount;
        else if(e.type==="bias"){
          totals.bias[e.target] = (totals.bias[e.target]||0) + e.amount;
        }
      }
      state.activeEffects = totals;
    }

    function addEffect(effect){
      const now = Date.now();
      const durMs = (effect.duration||0) * 1000;
      const capMs = MAX_EFFECT_SECONDS * 1000;

      const existingIdx = state.effectInstances.findIndex(e =>
        e.name === effect.name &&
        e.type === effect.type &&
        (e.target || null) === (effect.target || null)
      );

      if(existingIdx >= 0){
        const existing = state.effectInstances[existingIdx];
        const remaining = Math.max(0, existing.expiresAt - now);
        const extended = Math.min(capMs, remaining + durMs);
        existing.expiresAt = now + extended;
        if(effect.type!=="weather"){
          existing.amount += (effect.amount||0);
        }
        existing.rarityKey = effect.rarity || existing.rarityKey;
        existing.bg = effect.bg || existing.bg;
      } else {
        const inst={
          name: effect.name,
          type: effect.type,
          amount: effect.amount||0,
          target: effect.target,
          expiresAt: durMs ? now + Math.min(capMs, durMs) : now,
          rarityKey: effect.rarity || "common",
          weather: effect.type==="weather",
          bg: effect.bg || ""
        };
        state.effectInstances.push(inst);
      }

      deriveEffectsTotals();
      saveState();
      renderActiveEffects();
      updateAutoInterval();
      if(effect.type==="weather") applyWeatherBg(effect.bg || "");
    }

    /* prune expired effects and update display every second */
    setInterval(()=>{
      const now=Date.now();
      const before=state.effectInstances.length;
      state.effectInstances = state.effectInstances.filter(e=>e.expiresAt>now);
      if(state.effectInstances.length!==before){
        deriveEffectsTotals();
        saveState();
        updateAutoInterval();
        const activeWeather = getActiveWeather();
        applyWeatherBg(activeWeather?.bg ? activeWeather.bg : "");
      }
      renderActiveEffects();
    },1000);

    function formatSecondsLeft(ms){
      const s = Math.max(0, Math.ceil(ms/1000));
      return `${s}s left`;
    }

    function renderActiveEffects(){
      elActiveEffects.innerHTML="";
      const now=Date.now();
      const others = state.effectInstances.filter(e=>e.expiresAt>now && !e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      const weathers = state.effectInstances.filter(e=>e.expiresAt>now && e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      const active=[...others,...weathers];
      for(const e of active){
        const left = formatSecondsLeft(e.expiresAt - now);
        const colorClass = TIERS.find(t=>t.key===e.rarityKey)?.colorClass || "";
        const div=document.createElement("div");
        div.className=`effect-entry ${colorClass}`;
        const desc = e.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                  : e.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                  : e.type==="bias" ? `Bias → ${e.target?.toUpperCase()} +${Math.round(e.amount*100)}%`
                  : e.type==="guarantee" ? `Guarantee highest next roll` : "";
        div.textContent = e.type==="weather"
          ? `${e.name}: ${left}`
          : `${e.name}: ${left}${desc? ' • '+desc : ''}`;
        elActiveEffects.appendChild(div);
      }
    }

    function updateAutoInterval(){
      if(state.autoInterval){ clearInterval(state.autoInterval); state.autoInterval=null; }
      if(state.auto){
        const mult = 1 + (state.activeEffects.speed||0);
        const interval = Math.max(40, Math.round(BASE_AUTO_INTERVAL / mult));
        state.autoInterval = setInterval(()=>{ if(elBtnAuto.disabled){ toggleAuto(); return; } rollOnce(); }, interval);
      }
    }

    /* ---------------- Weather random scheduling ---------------- */
    function scheduleNextWeather(){
      const delayMs = (240 + Math.random()*480) * 1000; // 4–12 minutes
      setTimeout(()=>{ triggerRandomWeather(); scheduleNextWeather(); }, delayMs);
    }
    function triggerRandomWeather(){
      const r=Math.random();
      let pool = WEATHERS.normal;
      if(r>=0.70 && r<0.95) pool = WEATHERS.rare;
      else if(r>=0.95) pool = WEATHERS.super;
      const w = pool[Math.floor(Math.random()*pool.length)];
      const dur = 100 + Math.floor(Math.random()*201); // 100–300s
      addEffect({ name:w.name, type:"weather", duration: dur, rarity: classToTierKey(w.colorClass), bg: w.bg });
      spawnBanner(`${w.name} started`, "weather", w.colorClass);
      const bonus = w.effect;
      if(bonus){
        addEffect({ name:`${w.name} Bonus`, type:bonus.type, target:bonus.target, amount:bonus.amount, duration: dur, rarity: classToTierKey(w.colorClass) });
      }
    }
    function classToTierKey(colorClass){
      const t = TIERS.find(x=>x.colorClass===colorClass);
      return t ? t.key : "common";
    }
    function getActiveWeather(){
      const now = Date.now();
      return state.effectInstances.find(e=>e.weather && e.expiresAt>now) || null;
    }
    function applyWeatherBg(bgClass){
      elWeatherBg.className = "weather-bg " + (bgClass||"");
    }

    /* ---------------- Special weather-only drops ---------------- */
    function maybeWeatherExclusiveOverride(){
      const w = getActiveWeather();
      if(!w) return null;
      if(w.name==="Eternal Eclipse"){
        if(Math.random() < 1/200000) return { name:"Eclipse Gem", colorClass:"b-eclipse", tierKey:"exclusive" };
      } else if(w.name==="Cosmic Tempest"){
        if(Math.random() < 1/200000) return { name:"Cosmic Gem", colorClass:"b-cosmic", tierKey:"exclusive" };
      }
      return null;
    }

    /* ---------------- Rolling ---------------- */
    function rollOnce(){
      const upcoming=state.rolls+1;
      const milestone=luckMilestoneForRoll(upcoming);
      if(milestone>1) spawnBanner(`${milestone}x luck activated`,"luck");

      let tiers=TIERS.filter(t=>t.key!=="exclusive");
      tiers = applyWeightModifiers(tiers, milestone);

      const chances=toChances(tiers);
      const pickedTier = pickTier(chances);
      const tierKey = pickedTier.key;
      const tierName = pickedTier.name;

      const special = maybeWeatherExclusiveOverride();

      const itemTiers = buildItemTierWeightsShifted(TIERS.filter(t=>t.key!=="exclusive"));
      const itemWeighted = applyWeightModifiers(itemTiers, milestone);
      const itemChances = toChances(itemWeighted);

      const baseItemChance = 0.03;
      const luckBoost = Math.min(0.30, state.activeEffects.luck * 0.03);
      const rollItem = Math.random() < (baseItemChance + luckBoost);

      state.rolls++;

      let isNew=false;
      let displayName=null;
      let displayRarityClass=null;

      if(special){
        displayName = special.name;
        displayRarityClass = special.colorClass;
        processIndexItem("exclusive", "Exclusive", special.name, milestone);
        isNew = markNew("exclusive", special.name);
      } else if(rollItem){
        const itemTier = pickTier(itemChances);
        const drop = pickConsumableFromTier(itemTier.key);
        if(drop){
          const isTotem = drop.type==="totem" || drop.type==="totem_random";
          let allow = true;
          if(isTotem){
            if(drop.type==="totem_random"){ allow = Math.random() < (1/150); }
            else {
              const cat = weatherCategoryForName(drop.weather||"");
              const mult = cat==="normal" ? 100 : cat==="rare" ? 200 : 300;
              allow = Math.random() < (1/mult);
            }
          }
          if(allow){
            displayName = drop.name;
            displayRarityClass = TIERS.find(t=>t.key===drop.rarity)?.colorClass || "";
            const autosell = shouldAutoSellItems(itemTier.key);
            if(!autosell){
              if(state.inventoryItems.length < ITEMS_MAX){
                state.inventoryItems.push({ type: isTotem? drop.type : "consumable", tier:itemTier.key, tierName:itemTier.name, name:drop.name, roll:state.rolls, effect:drop });
              } else {
                if(!state.fullAnnouncedItems){ spawnBanner(`Items inventory is full ${ITEMS_MAX}/${ITEMS_MAX}`,"announce"); state.fullAnnouncedItems=true; }
              }
            }
          } else {
            const itemName = pickIndexItem(tierKey);
            displayName = itemName || tierName;
            displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
            processIndexItem(tierKey, tierName, itemName, milestone);
            if(itemName) isNew = markNew(tierKey, itemName);
          }
        } else {
          const itemName = pickIndexItem(tierKey);
          displayName = itemName || tierName;
          displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey, tierName, itemName, milestone);
          if(itemName) isNew = markNew(tierKey, itemName);
        }
      } else {
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

    function weatherCategoryForName(name){
      if(WEATHERS.normal.find(w=>w.name===name)) return "normal";
      if(WEATHERS.rare.find(w=>w.name===name)) return "rare";
      if(WEATHERS.super.find(w=>w.name===name)) return "super";
      return "normal";
    }

    function processIndexItem(tierKey, tierName, itemName, milestone){
      const toKeep = !shouldAutoSellRolled(tierKey);
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
        return true;
      }
      return false;
    }

    /* ---------------- UI ---------------- */
    function renderResult(name, tierKey, rarityClass){
      elResult.textContent = name || "Unknown";
      elRarity.textContent = tierKey ? tierKey.toUpperCase() : "";
      elRarity.className = "rarity badge " + (rarityClass || "");
    }

    function renderButtonsState(){
      if(state.rolls>=50){ elBtnAuto.disabled=false; elBtnAuto.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elBtnAuto.disabled=true; elBtnAuto.textContent="Auto Roll (locked)"; }

      const currentThreshold = state.mode==="Items" ? state.autoSellItems : state.autoSellRolled;
      elAutoSellValue.textContent = currentThreshold==="off" ? "Off" : labelForAutoSell(currentThreshold);
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
        const isExclusive = tier.key==="exclusive";
        const badgeClass = isExclusive ? "b-exclusive" : tier.colorClass;
        const badge = `<span class="badge ${badgeClass}">${isExclusive? "Exclusive" : tier.name}</span>`;
        const percentHtml = `<div class="completion ${isExclusive?'locked':''}">${isExclusive? '—' : comp.percent + '%'}</div>`;
        section.innerHTML=`<h4>${badge}</h4>${percentHtml}`;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(isExclusive){
          items.forEach(name=>{
            const unlocked=!!state.unlocks[tier.key][name];
            const li=document.createElement("li"); li.className="index-item";
            const colorClass = name==="Eclipse Gem" ? "b-eclipse" : name==="Cosmic Gem" ? "b-cosmic" : "b-exclusive";
            li.innerHTML=`
              <span class="badge ${colorClass} ${unlocked? '' : 'locked'}">${name}</span>
              <span class="${unlocked? "unlocked": "locked"}">${unlocked? "Unlocked (event only)": "Locked (event only)"}</span>
            `;
            ul.appendChild(li);
          });
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
          const nameBadgeClass = entry.name==="Eclipse Gem" ? "b-eclipse" : entry.name==="Cosmic Gem" ? "b-cosmic" : badgeClass;
          left.innerHTML=`<span class="badge ${nameBadgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"} • #${entry.roll}${entry.milestone>1?` • ${entry.milestone}x`:``}`;
          const right=document.createElement("div"); right.className="inv-actions";
          const del=document.createElement("button"); del.textContent="Delete";
          del.addEventListener("click",()=>deleteRolledEntry(entry));
          right.appendChild(del);
          li.appendChild(left); li.appendChild(right);
          elInventoryList.appendChild(li);
        });
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
          const effDesc = e?.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                        : e?.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                        : e?.type==="bias" ? `Bias → ${e.target.toUpperCase()} +${Math.round(e.amount*100)}%`
                        : e?.type==="guarantee" ? `Guarantee highest next roll`
                        : e?.type==="totem" ? `Summons ${e.weather}`
                        : e?.type==="totem_random" ? `Summons random weather` : "";
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name}${effDesc?` (${effDesc})`:''} • #${entry.roll}`;
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
      if(entry.effect){
        const eff = entry.effect;
        if(eff.type==="totem"){
          const dur = 100 + Math.floor(Math.random()*201);
          const weatherDef = findWeatherDef(eff.weather);
          const rarity = classToTierKey(weatherDef?.colorClass || "b-legendary");
          addEffect({ name: eff.weather, type:"weather", duration: dur, rarity, bg: weatherDef?.bg || "" });
          if(weatherDef?.effect){
            addEffect({ name:`${eff.weather} Bonus`, type:weatherDef.effect.type, target:weatherDef.effect.target, amount:weatherDef.effect.amount, duration: dur, rarity });
          }
          spawnBanner(`${entry.name} summoned ${eff.weather}`,"weather",weatherDef?.colorClass || "b-legendary");
        } else if(eff.type==="totem_random"){
          const r=Math.random();
          let pool = WEATHERS.normal;
          if(r>=0.70 && r<0.95) pool = WEATHERS.rare;
          else if(r>=0.95) pool = WEATHERS.super;
          const w = pool[Math.floor(Math.random()*pool.length)];
          const dur = 100 + Math.floor(Math.random()*201);
          const rarity = classToTierKey(w.colorClass);
          addEffect({ name: w.name, type:"weather", duration: dur, rarity, bg: w.bg });
          if(w.effect){
            addEffect({ name:`${w.name} Bonus`, type:w.effect.type, target:w.effect.target, amount:w.effect.amount, duration: dur, rarity });
          }
          spawnBanner(`${entry.name} summoned ${w.name}`,"weather",w.colorClass);
        } else {
          addEffect({ name: eff.name, type: eff.type, amount: eff.amount, target: eff.target, duration: eff.duration, rarity: eff.rarity });
          const colorClass = TIERS.find(t=>t.key===eff.rarity)?.colorClass || "";
          spawnBanner(`Activated ${entry.name}`,"activate",colorClass);
        }
      }
      deleteItemEntry(entry);
      renderActiveEffects();
    }

    function findWeatherDef(name){
      return [...WEATHERS.normal, ...WEATHERS.rare, ...WEATHERS.super].find(w=>w.name===name) || null;
    }

    function showGlow(){ const g=document.createElement("div"); g.className="glow"; elRollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Mode & Auto-Sell carousels ---------------- */
    const autoSellOptions=["off","worthless","trash","common","uncommon","rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];

    function setAutoSell(value){
      if(state.mode==="Items"){
        state.autoSellItems=value;
      } else {
        state.autoSellRolled=value;
      }
      saveState();
      renderButtonsState();
    }

    const modes=["Rolled","Items"];
    function setMode(value){
      state.mode=value;
      saveState();
      renderButtonsState();
      renderInventory();
    }

    function cycle(list, current, dir){
      const idx=list.indexOf(current);
      let next=idx;
      if(dir<0) next = idx<=0 ? list.length-1 : idx-1;
      else next = idx>=list.length-1 ? 0 : idx+1;
      return list[next];
    }

    /* ---------------- Auto clicker ---------------- */
    function toggleAuto(){
      if(elBtnAuto.disabled) return;
      if(state.auto){
        state.auto=false; clearInterval(state.autoInterval); state.autoInterval=null; elBtnAuto.textContent="Auto Roll: Off";
      } else {
        state.auto=true; elBtnAuto.textContent="Auto Roll: On";
        updateAutoInterval();
      }
      saveState();
    }

    /* ---------------- Hooks ---------------- */
    elBtnRoll.addEventListener("click",rollOnce);
    elBtnAuto.addEventListener("click",toggleAuto);

    elBtnIndex.addEventListener("click",()=>{
      const vis=elIndexPanel.style.display!=="none";
      if(vis){ elIndexPanel.style.display="none"; }
      else { elIndexPanel.style.display="block"; elInventoryPanel.style.display="none"; renderIndex(); }
    });
    elBtnInventory.addEventListener("click",()=>{
      const vis=elInventoryPanel.style.display!=="none";
      if(vis){ elInventoryPanel.style.display="none"; }
      else { elInventoryPanel.style.display="block"; elIndexPanel.style.display="none"; renderInventory(); }
    });

    elAutoSellPrev.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),-1)));
    elAutoSellNext.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),1)));

    elModePrev.addEventListener("click",()=>setMode(cycle(modes,state.mode,-1)));
    elModeNext.addEventListener("click",()=>setMode(cycle(modes,state.mode,1)));

    /* ---------------- Init ---------------- */
    loadState();
    renderButtonsState();
    renderActiveEffects();
    if(state.auto && state.rolls>=50){
      elBtnAuto.disabled=false; elBtnAuto.textContent="Auto Roll: On";
      updateAutoInterval();
    } else {
      elBtnAuto.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)";
    }
    scheduleNextWeather();
  </script>
</body>
</html>
