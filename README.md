<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Exclusive, Weather, and Effects Upgrade</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root{
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab;
      --accent:#6ea8fe; --gold:#ffd700; --warn:#ff6666; --weather:#4ec3ff;
    }
    body{margin:0;font-family:sans-serif;background:var(--bg);color:var(--text);display:grid;place-items:center;min-height:100vh;}
    .app{width:980px;max-width:96vw;background:var(--panel);border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;}

    /* Roll area and banners */
    .roll-area{min-height:360px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .result{font-size:28px;font-weight:700;text-align:center;position:relative;z-index:3;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;position:relative;z-index:3;}
    .banner{
      position:absolute;left:50%;transform:translateX(-50%);padding:10px 14px;border-radius:12px;z-index:6;
      background:rgba(20,25,40,0.9);border:1px solid rgba(255,255,255,0.14);box-shadow:0 6px 18px rgba(0,0,0,0.6);backdrop-filter:blur(5px);
      font-weight:700;text-align:center;color:var(--text);opacity:0;
    }
    .banner.show{animation:slideIn .35s ease-out, fadeout 3.6s forwards;}
    .banner.luck{top:14px;}
    .banner.new{top:120px;}
    .banner.weather{top:14px;}
    .banner.announce{top:60px;}
    @keyframes slideIn{from{transform:translate(-50%,-16px);opacity:0}to{transform:translate(-50%,0);opacity:1}}
    @keyframes fadeout{0%{opacity:1}70%{opacity:1}100%{opacity:0;filter:blur(4px)}}

    /* Active effects tray */
    .active-effects{
      position:absolute;bottom:10px;right:10px;z-index:5;font-size:12px;max-width:70%;
      display:flex;flex-direction:column;align-items:flex-end;gap:6px;
    }
    .effect-entry{
      display:flex;gap:8px;align-items:center;padding:6px 10px;border-radius:10px;
      background:rgba(27,34,50,0.9);border:1px solid #2a3449;box-shadow:0 6px 18px rgba(0,0,0,0.25);
      font-weight:600;
    }

    /* Weather backdrops + premium animations */
    .weather-bg{position:absolute;inset:0;z-index:1;pointer-events:none;opacity:0;animation:weatherFadeIn .7s ease-out forwards;}
    @keyframes weatherFadeIn{from{opacity:0}to{opacity:1}}
    .wb-sunny{background:radial-gradient(120% 120% at 50% 10%, rgba(255,215,120,0.22), transparent 60%), linear-gradient(180deg, rgba(255,232,170,0.08), rgba(0,0,0,0));}
    .wb-storm{background:linear-gradient(180deg, rgba(25,30,55,0.7), rgba(0,0,0,0.7));}
    .wb-blizzard{background:linear-gradient(180deg, rgba(200,230,255,0.25), rgba(0,0,0,0.5));}
    .wb-meteor{background:radial-gradient(160% 160% at 20% -20%, rgba(255,120,80,0.20), transparent 60%), linear-gradient(180deg, rgba(120,50,30,0.25), rgba(0,0,0,0.55));}
    .wb-aurora{background:linear-gradient(120deg,rgba(120,200,255,0.25),rgba(180,120,255,0.25),rgba(120,255,200,0.25));background-size:600% 600%;animation:auroraShift 18s ease infinite;}
    @keyframes auroraShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
    .wb-eclipse{background:radial-gradient(circle at 50% 50%,rgba(0,0,0,0.92),rgba(0,0,0,0.75)),radial-gradient(circle at 50% 50%,rgba(255,200,120,0.08),transparent 70%);}
    .wb-tempest{background:radial-gradient(circle at 50% 50%,rgba(120,80,255,0.25),transparent 70%),radial-gradient(circle at 70% 30%,rgba(80,200,255,0.25),transparent 70%);}
    .wb-fog{background:radial-gradient(100% 100% at 50% 50%, rgba(185,195,210,0.14), rgba(0,0,0,0.55));}

    /* Weather particle containers */
    .particles{position:absolute;inset:0;overflow:hidden;filter:blur(0.3px);}
    /* Rain with lightning */
    .rain-drop{position:absolute;width:2px;height:18px;background:linear-gradient(to bottom,rgba(180,200,255,0.95),rgba(180,200,255,0.25));border-radius:1px;}
    @keyframes rainFall{0%{transform:translateY(-60px)}100%{transform:translateY(420px)}}
    .lightning{position:absolute;left:10%;top:-10%;width:2px;height:200px;background:linear-gradient(to bottom,rgba(255,255,255,0.95),transparent);filter:blur(1px);opacity:0;animation:flash 5s ease-in-out infinite;}
    @keyframes flash{0%,92%{opacity:0}93%{opacity:1}95%{opacity:0}100%{opacity:0}}
    /* Snow swirl */
    .snow-flake{position:absolute;width:6px;height:6px;background:white;border-radius:50%;opacity:0.9;box-shadow:0 0 6px rgba(255,255,255,0.5);}
    @keyframes snowFall{0%{transform:translateY(-40px) translateX(0)}50%{transform:translateY(200px) translateX(16px)}100%{transform:translateY(420px) translateX(0)}}
    /* Fog drift layers */
    .fog-mist{position:absolute;left:-40%;top:0;width:180%;height:120%;background:radial-gradient(circle,rgba(255,255,255,0.08),transparent 70%);filter:blur(8px);opacity:0.75;animation:fogDrift 24s linear infinite;}
    @keyframes fogDrift{0%{transform:translateX(0)}50%{transform:translateX(10%)}100%{transform:translateX(0)}}
    /* Aurora ribbons */
    .ribbon{position:absolute;width:60%;height:16px;left:20%;top:18%;border-radius:999px;filter:blur(2px);opacity:0.6;animation:ribbonWave 12s ease-in-out infinite;}
    @keyframes ribbonWave{0%{transform:translateY(0) skewX(6deg)}50%{transform:translateY(40px) skewX(-6deg)}100%{transform:translateY(0) skewX(6deg)}}
    /* Tempest cosmic particles */
    .cosmic{position:absolute;width:4px;height:4px;border-radius:50%;background:radial-gradient(circle,rgba(255,255,255,0.9),rgba(255,255,255,0));opacity:0.8;}
    @keyframes drift{0%{transform:translate(0,0)}50%{transform:translate(12px,-16px)}100%{transform:translate(0,0)}}
    /* Eclipse corona pulse */
    .corona{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:240px;height:240px;border-radius:50%;box-shadow:0 0 80px rgba(255,200,120,0.2);animation:pulse 5s ease-in-out infinite;}
    @keyframes pulse{0%{box-shadow:0 0 80px rgba(255,200,120,0.15)}50%{box-shadow:0 0 120px rgba(255,200,120,0.35)}100%{box-shadow:0 0 80px rgba(255,200,120,0.15)}}

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
    .inv-stats{font-size:13px;color:var(--muted);}
    .inv-list{list-style:none;padding:0;margin:0;}
    .inv-list li{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid #242a38;padding:6px 0;}
    .inv-actions{display:flex;gap:8px;}

    /* Mode selector and Auto-Sell carousel */
    .mode-wrap{display:flex;align-items:center;gap:12px;}
    .mode-label{font-size:13px;color:var(--muted);}
    .mode-carousel{display:flex;align-items:center;gap:6px;}
    .mode-value{min-width:80px;text-align:center;padding:8px 10px;border:1px solid #2a3449;border-radius:8px;background:#1b2232;}
    .mode-arrow{padding:6px 10px;}
    .autosell-wrap{display:flex;align-items:center;gap:8px;margin-left:auto;}
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
    /* Exclusive continuous rainbow */
    .b-exclusive{
      background:conic-gradient(from 0deg, red, orange, yellow, green, blue, indigo, violet, red);
      animation:exclusiveSpin 8s linear infinite; color:white;
    }
    @keyframes exclusiveSpin{0%{filter:hue-rotate(0deg)}100%{filter:hue-rotate(360deg)}}

    /* Glow effect */
    .glow{position:absolute;inset:-40%;border-radius:50%;background:radial-gradient(closest-side,rgba(110,168,254,0.25),transparent 65%);filter:blur(12px);animation:glow 1.1s ease-out forwards;z-index:2;}
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
      {key:"exclusive",name:"Exclusive",weight:0,colorClass:"b-exclusive"} // event-only drops
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
        // Basic weather totems (Legendary)
        { name:"Storm Totem", rarity:"legendary", type:"totem", weather:"Storm" },
        { name:"Blizzard Totem", rarity:"legendary", type:"totem", weather:"Blizzard" },
        { name:"Sun Totem", rarity:"legendary", type:"totem", weather:"Sunny Radiance" },
        { name:"Fog Totem", rarity:"legendary", type:"totem", weather:"Fog" },
        // Random Event Totem (Divine)
        { name:"Random Event Totem", rarity:"divine", type:"totem_random" }
      ],
      uncommon:[
        { name:"Strong Luck Potion", rarity:"uncommon", type:"luck", amount:0.50, duration:150 },
        { name:"Sigil of Odds", rarity:"uncommon", type:"bias", target:"epic", amount:0.08, duration:60 }
      ],
      rare:[
        { name:"Greater Luck Potion", rarity:"rare", type:"luck", amount:1.00, duration:60 },
        { name:"Greater Luck Charm", rarity:"rare", type:"bias", target:"legendary", amount:0.10, duration:60 },
        // Rare weather totems (Divine)
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
        { name:"Speed Potion", rarity:"legendary", type:"speed", amount:0.25, duration:60 },
        // Super-rare weather totems (Transcendent)
        { name:"Eternal Eclipse Totem", rarity:"transcendent", type:"totem", weather:"Eternal Eclipse" },
        { name:"Cosmic Tempest Totem", rarity:"transcendent", type:"totem", weather:"Cosmic Tempest" }
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
      ],
      exclusive:[]
    };

    const LUCK_TARGET_KEYS=["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal","exclusive"];

    /* ---------------- Weather definitions & effects ---------------- */
    const WEATHERS = {
      normal: [
        {name:"Storm", colorClass:"b-rare", effects:{luck:+0.12}},
        {name:"Blizzard", colorClass:"b-epic", effects:{luck:+0.15}},
        {name:"Sunny Radiance", colorClass:"b-common", effects:{luck:+0.06}},
        {name:"Fog", colorClass:"b-uncommon", effects:{luck:+0.08}}
      ],
      rare: [
        {name:"Meteor Storm", colorClass:"b-legendary", effects:{luck:+0.22}},
        {name:"Aurora Veil", colorClass:"b-mythic", effects:{luck:+0.28}}
      ],
      super: [
        {name:"Eternal Eclipse", colorClass:"b-divine", effects:{luck:+0.80, biasItem:{name:"Timeweaver Crest", boost:+0.50}}},
        {name:"Cosmic Tempest", colorClass:"b-omniversal", effects:{luck:+0.65, biasItem:{name:"Axis of All", boost:+0.35}}}
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
    const BASE_AUTO_INTERVAL=140; // ms
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
      activeEffects:{ luck:0, speed:0, bias:{}, weatherBiasItem:null },
      effectInstances:[], // {name,type,amount,target?,expiresAt,rarityKey,weather?,meta?}
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
      const rollArea=document.getElementById("rollArea");
      const div=document.createElement("div");
      const useType = type==="activate" ? "announce" : type;
      div.className=`banner ${useType} show ${colorClass?colorClass:''}`;
      div.textContent=text;
      div.addEventListener("animationend",()=>div.remove());
      rollArea.appendChild(div);
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
      // Improved rarity selection: iterate forward accumulating chance, fallback to lowest rarity when precision issues
      let r=Math.random(), acc=0;
      for(let i=0;i<chances.length;i++){
        acc += chances[i].chance;
        if(r<=acc) return chances[i];
      }
      return chances[chances.length-1];
    }
    function pickIndexItem(tierKey){
      const list=INDEX_ITEMS[tierKey]||[];
      if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* Items much rarer; group rarity penalty (+2 tiers) and large divisor */
    function buildItemTierWeightsFromIndex(baseTiers){
      const order = baseTiers.map(t=>t.key);
      return baseTiers.map(t=>{
        const idx = order.indexOf(t.key);
        const penalizedKey = order[Math.min(idx+2, order.length-1)];
        const penalizedTier = baseTiers.find(x=>x.key===penalizedKey);
        const weight = (penalizedTier?.weight || t.weight) / 8000; // harsher divisor → rarer items
        return { ...t, weight };
      });
    }

    function pickConsumableFromTier(tierKey){
      const list=ITEM_DROPS[tierKey]||[];
      if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }

    /* ---------------- Weather backdrop & animations ---------------- */
    function renderWeatherBackdrop(){
      const area = document.getElementById("rollArea");
      const old = area.querySelector(".weather-bg");
      if(old) old.remove();

      const now = Date.now();
      const weather = state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      if(!weather) return;

      const map = {
        "Sunny Radiance":"wb-sunny",
        "Storm":"wb-storm",
        "Blizzard":"wb-blizzard",
        "Meteor Storm":"wb-meteor",
        "Aurora Veil":"wb-aurora",
        "Eternal Eclipse":"wb-eclipse",
        "Cosmic Tempest":"wb-tempest",
        "Fog":"wb-fog"
      };
      const bg = document.createElement("div");
      const cls = map[weather.name] || "wb-sunny";
      bg.className = "weather-bg " + cls;

      const particles = document.createElement("div");
      particles.className = "particles";

      if(cls==="wb-storm"){
        // heavy rain
        for(let i=0;i<100;i++){
          const p=document.createElement("span");
          p.className="rain-drop";
          p.style.left = Math.floor(Math.random()*100)+"%";
          p.style.top = (-60 - Math.random()*160) + "px";
          p.style.animation = `rainFall ${0.7 + Math.random()*0.6}s linear infinite`;
          p.style.opacity = 0.5 + Math.random()*0.5;
          particles.appendChild(p);
        }
        const bolt=document.createElement("div");
        bolt.className="lightning";
        particles.appendChild(bolt);
      } else if(cls==="wb-blizzard"){
        for(let i=0;i<60;i++){
          const s=document.createElement("span");
          s.className="snow-flake";
          s.style.left = Math.floor(Math.random()*100)+"%";
          s.style.top = (-60 - Math.random()*180) + "px";
          s.style.animation = `snowFall ${3.2 + Math.random()*2.6}s linear infinite`;
          s.style.opacity = 0.6 + Math.random()*0.4;
          particles.appendChild(s);
        }
      } else if(cls==="wb-fog"){
        const fog1=document.createElement("div"); fog1.className="fog-mist"; fog1.style.opacity="0.7";
        const fog2=document.createElement("div"); fog2.className="fog-mist"; fog2.style.opacity="0.45"; fog2.style.animationDuration="32s";
        particles.appendChild(fog1); particles.appendChild(fog2);
      } else if(cls==="wb-aurora"){
        const ribbon1=document.createElement("div"); ribbon1.className="ribbon"; ribbon1.style.background="linear-gradient(90deg, rgba(160,255,220,0.6), rgba(150,120,255,0.6))";
        const ribbon2=document.createElement("div"); ribbon2.className="ribbon"; ribbon2.style.top="30%"; ribbon2.style.animationDuration="16s"; ribbon2.style.background="linear-gradient(90deg, rgba(120,200,255,0.6), rgba(120,255,200,0.6))";
        particles.appendChild(ribbon1); particles.appendChild(ribbon2);
      } else if(cls==="wb-eclipse"){
        const corona=document.createElement("div"); corona.className="corona";
        particles.appendChild(corona);
      } else if(cls==="wb-tempest"){
        for(let i=0;i<30;i++){
          const c=document.createElement("span");
          c.className="cosmic";
          c.style.left = Math.floor(Math.random()*100)+"%";
          c.style.top = Math.floor(Math.random()*100)+"%";
          c.style.animation = `drift ${8 + Math.random()*6}s ease-in-out infinite`;
          particles.appendChild(c);
        }
      } else if(cls==="wb-meteor"){
        for(let i=0;i<12;i++){
          const m=document.createElement("span");
          m.className="cosmic";
          m.style.left = Math.floor(Math.random()*100)+"%";
          m.style.top = Math.floor(Math.random()*40)+"%";
          m.style.width = m.style.height = (Math.random()*3+2)+"px";
          m.style.animation = `drift ${10 + Math.random()*6}s ease-in-out infinite`;
          particles.appendChild(m);
        }
      }

      bg.appendChild(particles);
      area.appendChild(bg);
    }

    /* ---------------- Effects (timed instances + totals) ---------------- */
    function deriveEffectsTotals(){
      const totals={ luck:0, speed:0, bias:{}, weatherBiasItem:null };
      const now=Date.now();
      for(const e of state.effectInstances){
        if(e.expiresAt <= now) continue;
        if(e.type==="luck") totals.luck += e.amount;
        else if(e.type==="speed") totals.speed += e.amount;
        else if(e.type==="bias"){
          totals.bias[e.target] = (totals.bias[e.target]||0) + e.amount;
        } else if(e.type==="weather" && e.meta){
          totals.weatherBiasItem = e.meta.biasItem || null;
          totals.luck += (e.meta.luck || 0);
        }
      }
      state.activeEffects = totals;
    }

    // Stack with max cap; weather stacks duration only
    function addEffect(effect){
      const now = Date.now();
      const durMs = (effect.duration||0) * 1000;
      const capMs = MAX_EFFECT_SECONDS * 1000;

      const keyMatch = e =>
        e.name === effect.name &&
        (effect.type === "weather" ? e.type==="weather" : e.type === effect.type) &&
        (e.target || null) === (effect.target || null);

      const existingIdx = state.effectInstances.findIndex(keyMatch);

      if(existingIdx >= 0){
        const existing = state.effectInstances[existingIdx];
        const remaining = Math.max(0, existing.expiresAt - now);
        const extended = Math.min(capMs, remaining + durMs);
        existing.expiresAt = now + extended;
        if(effect.type!=="weather"){
          existing.amount += (effect.amount||0);
        } else {
          existing.meta = effect.meta || existing.meta;
        }
        existing.rarityKey = effect.rarity || existing.rarityKey;
      } else {
        const inst={
          name: effect.name,
          type: effect.type,
          amount: effect.amount||0,
          target: effect.target,
          expiresAt: durMs ? now + Math.min(capMs, durMs) : now,
          rarityKey: effect.rarity || "common",
          weather: effect.type==="weather",
          meta: effect.meta || null
        };
        state.effectInstances.push(inst);
      }

      deriveEffectsTotals();
      saveState();
      renderActiveEffects();
      renderWeatherBackdrop();
      updateAutoInterval();
    }

    /* prune expired effects and update display every second */
    setInterval(()=>{
      const now=Date.now();
      const before=state.effectInstances.length;
      state.effectInstances = state.effectInstances.filter(e=>e.expiresAt>now);
      if(state.effectInstances.length!==before){
        deriveEffectsTotals();
        saveState();
        renderWeatherBackdrop();
        updateAutoInterval();
      }
      renderActiveEffects();
    },1000);

    function formatSecondsLeft(ms){
      const s = Math.max(0, Math.ceil(ms/1000));
      return `${s}s`;
    }

    function renderActiveEffects(){
      const el=document.getElementById("activeEffects");
      el.innerHTML="";
      const now=Date.now();
      const others = state.effectInstances.filter(e=>e.expiresAt>now && !e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      const weathers = state.effectInstances.filter(e=>e.expiresAt>now && e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      const active=[...others,...weathers];
      for(const e of active){
        const left = formatSecondsLeft(e.expiresAt - now);
        const colorClass = TIERS.find(t=>t.key===e.rarityKey)?.colorClass || "";
        const div=document.createElement("div");
        div.className=`effect-entry ${colorClass}`;
        div.textContent = `${e.name}: ${left}`;
        el.appendChild(div);
      }
    }

    function colorClassForWeather(name){
      const w = [...WEATHERS.normal, ...WEATHERS.rare, ...WEATHERS.super].find(x=>x.name===name);
      return w?.colorClass || "";
    }

    function showGlow(){ const rollArea=document.getElementById("rollArea"); const g=document.createElement("div"); g.className="glow"; rollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Mode & Auto-Sell carousels ---------------- */
    const autoSellOptions=["off","worthless","trash","common","uncommon","rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal","exclusive"];

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
    function updateAutoInterval(){
      if(state.autoInterval){ clearInterval(state.autoInterval); state.autoInterval=null; }
      if(state.auto){
        const mult = 1 + (state.activeEffects.speed||0);
        const interval = Math.max(60, Math.round(BASE_AUTO_INTERVAL / mult));
        state.autoInterval = setInterval(()=>{ if(elAutoBtn.disabled){ toggleAuto(); return; } rollOnce(); }, interval);
      }
    }
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

    /* ---------------- Weather random scheduling ---------------- */
    function triggerRandomWeather(){
      // Weighted categories: normal 70%, rare 25%, super 5%
      const r=Math.random();
      let pool = WEATHERS.normal;
      if(r>=0.70 && r<0.95) pool = WEATHERS.rare;
      else if(r>=0.95) pool = WEATHERS.super;
      const w = pool[Math.floor(Math.random()*pool.length)];
      const dur = 100 + Math.floor(Math.random()*201); // 100–300s
      const meta = { luck: (w.effects?.luck)||0, biasItem: (w.effects?.biasItem)||null };
      addEffect({ name:w.name, type:"weather", duration:dur, rarity: classToTierKey(w.colorClass), meta });
      spawnBanner(`${w.name} started`, "weather", w.colorClass);
    }
    function scheduleNextWeather(){
      const delayMs = (240 + Math.random()*480) * 1000; // 4–12 minutes
      setTimeout(()=>{ triggerRandomWeather(); scheduleNextWeather(); }, delayMs);
    }
    function classToTierKey(colorClass){
      const t = TIERS.find(x=>x.colorClass===colorClass);
      return t ? t.key : "common";
    }

    /* ---------------- Rolling ---------------- */
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

    function rollOnce(){
      const upcoming=state.rolls+1;
      const milestone=luckMilestoneForRoll(upcoming);
      if(milestone>1) spawnBanner(`${milestone}x luck activated`,"luck","b-divine");

      // Build tiers excluding Exclusive (event-only)
      let tiers=TIERS.filter(t=>t.key!=="exclusive");
      tiers = applyWeightModifiers(tiers, milestone);

      const now = Date.now();
      const wEff = state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      const exclusiveActive = wEff && (wEff.name==="Eternal Eclipse" || wEff.name==="Cosmic Tempest");

      // Weighted pick
      const chances=toChances(tiers);
      let pickedTier = pickTier(chances);
      let tierKey = pickedTier.key;
      let tierName = pickedTier.name;

      // Exclusive override gate during events
      if(exclusiveActive){
        const secretGateBase = 0.00005; // 0.005%
        const luckBoostSecret = Math.min(0.00002, (state.activeEffects.luck||0) * 0.000005);
        const passExclusive = Math.random() < (secretGateBase + luckBoostSecret);
        if(passExclusive){
          tierKey = "exclusive";
          tierName = "Exclusive";
        }
      }

      // Items rarity coin
      const itemTiers = buildItemTierWeightsFromIndex(TIERS.filter(t=>t.key!=="exclusive"));
      const itemWeighted = applyWeightModifiers(itemTiers, milestone);
      const itemChances = toChances(itemWeighted);

      const baseItemChance = 0.02; // 2% base
      const luckBoost = Math.min(0.03, (state.activeEffects.luck||0) * 0.015);
      const rollItem = Math.random() < (baseItemChance + luckBoost);

      state.rolls++;

      let isNew=false;
      let displayName=null;
      let displayRarityClass=null;

      function tryWeatherItemBias(defaultTierKey){
        const bias = state.activeEffects.weatherBiasItem; // {name, boost}
        if(!bias) return null;
        const items = INDEX_ITEMS[defaultTierKey] || [];
        if(!items.includes(bias.name)) return null;
        const pass = Math.random() < (bias.boost || 0);
        return pass ? bias.name : null;
      }

      if(rollItem){
        const itemTier = pickTier(itemChances);
        const drop = pickConsumableFromTier(itemTier.key);
        if(drop){
          const isTotem = drop.type==="totem" || drop.type==="totem_random";
          let allow = true;
          if(isTotem){
            if(drop.type==="totem_random"){ allow = Math.random() < (1/350); }
            else {
              const wName = drop.weather || "";
              const cat = weatherCategoryForName(wName);
              const mult = cat==="normal" ? 220 : cat==="rare" ? 350 : 500;
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
                if(!state.fullAnnouncedItems){ spawnBanner(`Items inventory is full ${ITEMS_MAX}/${ITEMS_MAX}`,"announce",""); state.fullAnnouncedItems=true; }
              }
            }
          } else {
            const biasPick = tryWeatherItemBias(tierKey);
            const itemName = biasPick || pickIndexItem(tierKey);
            displayName = itemName || tierName;
            displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
            processIndexItem(tierKey, tierName, itemName, milestone);
            if(itemName) isNew = markNew(tierKey, itemName);
          }
        } else {
          const biasPick = tryWeatherItemBias(tierKey);
          const itemName = biasPick || pickIndexItem(tierKey);
          displayName = itemName || tierName;
          displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey, tierName, itemName, milestone);
          if(itemName) isNew = markNew(tierKey, itemName);
        }
      } else {
        if(tierKey==="exclusive"){
          const wName = wEff?.name || "";
          let candidates = [];
          if(wName==="Eternal Eclipse") candidates = ["Eclipse Gem"];
          else if(wName==="Cosmic Tempest") candidates = ["Cosmic Gem"];
          const itemName = candidates.length ? candidates[Math.floor(Math.random()*candidates.length)] : null;
          displayName = itemName || "Exclusive";
          displayRarityClass = "b-exclusive";
          processIndexItem("exclusive", "Exclusive", itemName, milestone);
          if(itemName) isNew = markNew("exclusive", itemName);
        } else {
          const biasPick = tryWeatherItemBias(tierKey);
          const itemName = biasPick || pickIndexItem(tierKey);
          displayName = itemName || tierName;
          displayRarityClass = TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey, tierName, itemName, milestone);
          if(itemName) isNew = markNew(tierKey, itemName);
        }
      }

      saveState();

      showGlow();
      renderResult(displayName, tierKey, displayRarityClass);
      if(isNew) spawnBanner(`NEW collected: [${tierName}] ${displayName}`,"new",displayRarityClass);
      renderButtonsState();
      renderIndex();
      renderInventory();
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
      elResult.textContent = name || "Unknown";
      elRarity.textContent = tierKey ? tierKey.toUpperCase() : "";
      elRarity.className = "rarity badge " + (rarityClass || "");
      renderWeatherBackdrop();
    }

    function renderButtonsState(){
      if(state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)"; }

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
        const badge = `<span class="badge ${tier.colorClass} ${isExclusive?'locked':''}">${isExclusive?'Exclusive':tier.name}</span>`;
        const percentHtml = `<div class="completion ${isExclusive?'locked':''}">${isExclusive? '—' : comp.percent + '%'}</div>`;
        section.innerHTML=`
          <h4>${badge}</h4>
          ${percentHtml}
        `;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(isExclusive){
          ["Eclipse Gem","Cosmic Gem"].forEach(n=>{
            const li=document.createElement("li"); li.className="index-item";
            const unlocked=!!state.unlocks[tier.key][n];
            li.innerHTML=`<span class="${unlocked? "": "locked"}">${n}</span>
                          <span class="${unlocked? "unlocked": "locked"}">${unlocked? "Unlocked": "Locked"}</span>`;
            ul.appendChild(li);
          });
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
          const effDesc = e?.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                        : e?.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                        : e?.type==="bias" ? `Bias → ${e.target.toUpperCase()} +${Math.round(e.amount*100)}%`
                        : e?.type==="guarantee" ? `Guarantee highest next roll`
                        : e?.type==="totem" ? `Summons ${e.weather}`
                        : e?.type==="totem_random" ? `Summons random weather` : "";
          left.innerHTML=`<span class="badge ${effColorClass}">${entry.tierName}</span> — ${entry.name}${effDesc?` (${effDesc})`:''} • #${entry.roll}`;
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
      if(entry.effect){
        const eff = entry.effect;
        if(eff.type==="totem"){
          const metaSrc = [...WEATHERS.normal, ...WEATHERS.rare, ...WEATHERS.super].find(w=>w.name===eff.weather);
          const dur = 100 + Math.floor(Math.random()*201);
          const meta = { luck: (metaSrc?.effects?.luck)||0, biasItem: (metaSrc?.effects?.biasItem)||null };
          addEffect({ name: eff.weather, type:"weather", duration: dur, rarity: classToTierKey(colorClassForWeather(eff.weather)), meta });
          const colorClass = colorClassForWeather(eff.weather);
          spawnBanner(`Activated ${entry.name}`,"activate",colorClass);
          spawnBanner(`${eff.weather} started`, "weather", colorClass);
        } else if(eff.type==="totem_random"){
          const r=Math.random();
          let pool = WEATHERS.normal;
          if(r>=0.70 && r<0.95) pool = WEATHERS.rare;
          else if(r>=0.95) pool = WEATHERS.super;
          const w = pool[Math.floor(Math.random()*pool.length)];
          const dur = 100 + Math.floor(Math.random()*201);
          const meta = { luck: (w.effects?.luck)||0, biasItem: (w.effects?.biasItem)||null };
          addEffect({ name: w.name, type:"weather", duration: dur, rarity: classToTierKey(w.colorClass), meta });
          spawnBanner(`Activated ${entry.name}`,"activate",w.colorClass);
          spawnBanner(`${w.name} started`,"weather",w.colorClass);
        } else {
          addEffect({ name: eff.name, type: eff.type, amount: eff.amount, target: eff.target, duration: eff.duration, rarity: eff.rarity });
          const colorClass = TIERS.find(t=>t.key===eff.rarity)?.colorClass || "";
          spawnBanner(`Activated ${entry.name}`,"activate",colorClass);
        }
      }
      deleteItemEntry(entry);
      renderActiveEffects();
    }

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
    document.getElementById("btnAuto").addEventListener("click",toggleAuto);

    const elAutoSellPrev=document.getElementById("autoSellPrev");
    const elAutoSellNext=document.getElementById("autoSellNext");
    const elModePrev=document.getElementById("modePrev");
    const elModeNext=document.getElementById("modeNext");

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

    elAutoSellPrev.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),-1)));
    elAutoSellNext.addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),1)));

    elModePrev.addEventListener("click",()=>setMode(cycle(modes,state.mode,-1)));
    elModeNext.addEventListener("click",()=>setMode(cycle(modes,state.mode,1)));

    /* ---------------- Init ---------------- */
    loadState();
    renderButtonsState();
    renderActiveEffects();
    renderWeatherBackdrop();
    if(state.auto && state.rolls>=50){
      elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On";
      updateAutoInterval();
    } else {
      elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)";
    }
    scheduleNextWeather();
  </script>
</body>
</html>
