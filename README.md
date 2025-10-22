<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Expanded</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0e0f13;
      --panel: #151822;
      --text: #e7e9ee;
      --muted: #9aa0ab;
      --accent: #6ea8fe;
      --gold: #ffd700;
    }
    body {
      margin: 0;
      font-family: system-ui, Segoe UI, Roboto, Arial, sans-serif;
      background: radial-gradient(1200px 600px at 50% -200px, #1a2030 0%, var(--bg) 50%, var(--bg) 100%);
      color: var(--text);
      min-height: 100vh; display: grid; place-items: center;
    }
    .app {
      width: 960px; max-width: 96vw;
      background: linear-gradient(180deg, #141821 0%, var(--panel) 100%);
      border: 1px solid #252b39; border-radius: 14px;
      box-shadow: 0 20px 60px rgba(0,0,0,0.5);
      overflow: hidden;
    }
    .content { padding: 20px; display: grid; gap: 18px; }
    .panel {
      background: #121521; border: 1px solid #242a38; border-radius: 12px; padding: 16px;
    }
    .roll-area { min-height: 220px; display: grid; place-items: center; position: relative; overflow: hidden; }
    .luck-banner { font-size: 16px; font-weight: 700; color: var(--gold); margin-bottom: 8px; min-height: 22px; text-align: center; }
    .result { font-size: 28px; font-weight: 700; text-align: center; text-shadow: 0 0 12px rgba(255,255,255,0.15); }
    .rarity { margin-top: 6px; font-size: 14px; font-weight: 600; letter-spacing: 0.3px; text-transform: uppercase; }
    .controls { display: flex; gap: 12px; padding-top: 12px; }
    button {
      background: #1b2232; color: var(--text); border: 1px solid #2a3449;
      padding: 12px 16px; border-radius: 10px; cursor: pointer; font-weight: 600;
      transition: transform 0.06s ease, background 0.2s ease, box-shadow 0.2s ease;
    }
    button:hover { background: #232c41; box-shadow: 0 6px 18px rgba(110,168,254,0.12); }
    button:active { transform: translateY(1px); }
    button:disabled { opacity: 0.45; cursor: not-allowed; }
    .new-banner { margin-top: 8px; font-size: 14px; color: var(--accent); font-weight: 600; min-height: 20px; text-align: center; }

    /* Index and Inventory */
    .index-grid { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 12px; }
    .index-section { background: #0f1320; border: 1px solid #252b39; border-radius: 10px; padding: 12px; }
    .index-section h4 { margin: 0 0 8px; font-size: 14px; color: var(--muted); display: flex; align-items: center; gap: 8px; }
    .badge { display: inline-block; padding: 2px 8px; border-radius: 999px; font-size: 12px; font-weight: 700; }
    .index-list { list-style: none; padding: 0; margin: 0; }
    .index-item { display: flex; justify-content: space-between; align-items: center; padding: 6px 0; border-bottom: 1px solid #242a38; font-size: 14px; }
    .locked { filter: blur(3px); opacity: 0.6; }
    .unlocked { color: var(--accent); font-weight: 600; }

    .inv-header { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 8px; }
    .inv-stats { font-size: 13px; color: var(--muted); }

    /* Distinct rarity badge colors */
    .b-worthless    { background: #1a1a1a; color: #b3b3b3; }
    .b-trash        { background: #211616; color: #d28f8f; }
    .b-common       { background: #1f2635; color: #c7d1e5; }
    .b-uncommon     { background: #14261d; color: #7ae08f; }
    .b-rare         { background: #221a35; color: #caa6ff; }
    .b-epic         { background: #10233a; color: #7bb7ff; }
    .b-legendary    { background: #2a1f12; color: #ffbf66; }
    .b-mythic       { background: #2a142a; color: #ff7ee6; }
    .b-divine       { background: #2a2a12; color: #ffd700; }
    .b-celestial    { background: #142a2a; color: #7affff; }
    .b-transcendent { background: #1a1a2f; color: #a0a7ff; }
    .b-eternal      { background: #1a2f2a; color: #9cf2c7; }
    .b-omniversal   { background: #2f1a2f; color: #ff9cff; }

    /* Glow effect */
    .glow {
      position: absolute; inset: -40%; border-radius: 50%;
      background: radial-gradient(closest-side, rgba(110,168,254,0.25), transparent 65%);
      filter: blur(12px); transform: scale(0.7); opacity: 0; pointer-events: none;
      animation: glow 1.1s ease-out forwards;
    }
    @keyframes glow { 0%{opacity:0; transform:scale(0.7)} 50%{opacity:1} 100%{opacity:0; transform:scale(1.2)} }
  </style>
</head>
<body>
  <div class="app">
    <div class="content">
      <!-- Rolling panel -->
      <div class="panel">
        <div class="luck-banner" id="luckBanner"></div>
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
        </div>
        <div class="controls">
          <button id="btnRoll">Roll</button>
          <button id="btnAuto" disabled>Auto Roll (locked)</button>
          <button id="btnIndex">Index</button>
          <button id="btnInventory">Inventory</button>
        </div>
        <div class="new-banner" id="newBanner"></div>
      </div>

      <!-- Index panel (hidden until button clicked) -->
      <div class="panel" id="indexPanel" style="display:none;">
        <h3 style="margin:0 0 10px;">Index</h3>
        <div class="index-grid" id="indexGrid"></div>
      </div>

      <!-- Inventory panel (hidden until button clicked) -->
      <div class="panel" id="inventoryPanel" style="display:none;">
        <div class="inv-header">
          <h3 style="margin:0;">Inventory</h3>
          <div class="inv-stats" id="inventoryStats"></div>
        </div>
        <ul id="inventoryList" style="list-style:none; padding:0; margin:0;"></ul>
      </div>
    </div>
  </div>

  <script>
    // -----------------------------
    // CONFIG: tiers and weights (higher index = rarer)
    const TIERS = [
      { key: "worthless",    name: "Worthless",    weight: 1200000, colorClass: "b-worthless" },
      { key: "trash",        name: "Trash",        weight: 600000,  colorClass: "b-trash" },
      { key: "common",       name: "Common",       weight: 300000,  colorClass: "b-common" },
      { key: "uncommon",     name: "Uncommon",     weight: 60000,   colorClass: "b-uncommon" },
      { key: "rare",         name: "Rare",         weight: 9000,    colorClass: "b-rare" },
      { key: "epic",         name: "Epic",         weight: 1200,    colorClass: "b-epic" },
      { key: "legendary",    name: "Legendary",    weight: 160,     colorClass: "b-legendary" },
      { key: "mythic",       name: "Mythic",       weight: 40,      colorClass: "b-mythic" },
      { key: "divine",       name: "Divine",       weight: 16,      colorClass: "b-divine" },
      { key: "celestial",    name: "Celestial",    weight: 8,       colorClass: "b-celestial" },
      { key: "transcendent", name: "Transcendent", weight: 4,       colorClass: "b-transcendent" },
      { key: "eternal",      name: "Eternal",      weight: 2,       colorClass: "b-eternal" },
      { key: "omniversal",   name: "Omniversal",   weight: 1,       colorClass: "b-omniversal" },
    ];

    // Items per rarity (12+ each, names only)
    const INDEX_ITEMS = {
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
    };

    // Luck affects higher tiers only
    const LUCK_TARGET_KEYS = ["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];

    // -----------------------------
    // PERSISTENCE
    const STORAGE_KEYS = {
      rolls: "sol_rng_rolls",
      unlocks: "sol_rng_unlocks",
      auto: "sol_rng_auto",
      inv: "sol_rng_inventory"
    };

    function buildInitialUnlocks(indexDef) {
      const out = {};
      for (const key in indexDef) {
        out[key] = {};
        indexDef[key].forEach(name => { out[key][name] = false; });
      }
      return out;
    }
    function reviveUnlocks(unlocks) {
      const fresh = buildInitialUnlocks(INDEX_ITEMS);
      for (const key in fresh) {
        for (const name of Object.keys(fresh[key])) {
          fresh[key][name] = unlocks[key] && typeof unlocks[key][name] === "boolean" ? unlocks[key][name] : false;
        }
      }
      return fresh;
    }
    function loadState() {
      const rolls = parseInt(localStorage.getItem(STORAGE_KEYS.rolls) || "0", 10);
      const unlocksRaw = localStorage.getItem(STORAGE_KEYS.unlocks);
      const autoRaw = localStorage.getItem(STORAGE_KEYS.auto);
      const invRaw = localStorage.getItem(STORAGE_KEYS.inv);
      state.rolls = Number.isFinite(rolls) ? rolls : 0;
      state.unlocks = unlocksRaw ? reviveUnlocks(JSON.parse(unlocksRaw)) : buildInitialUnlocks(INDEX_ITEMS);
      state.auto = autoRaw === "true";
      state.inventory = invRaw ? JSON.parse(invRaw) : []; // [{tier,name,roll}]
    }
    function saveState() {
      localStorage.setItem(STORAGE_KEYS.rolls, String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks, JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto, state.auto ? "true" : "false");
      localStorage.setItem(STORAGE_KEYS.inv, JSON.stringify(state.inventory));
    }

    // -----------------------------
    // STATE
    const state = {
      rolls: 0,
      unlocks: buildInitialUnlocks(INDEX_ITEMS),
      auto: false,
      autoInterval: null,
      inventory: [] // collected items by roll order
    };

    // -----------------------------
    // UTILS
    function clone(obj) { return JSON.parse(JSON.stringify(obj)); }
    function sumWeights(arr) { return arr.reduce((s, r) => s + r.weight, 0); }

    // Luck milestones: activate only ON the exact roll numbers
    function luckForUpcomingRoll(upcomingRollNumber) {
      if (upcomingRollNumber === 250) return 10;
      if (upcomingRollNumber === 50) return 2;
      return 1;
    }

    function applyMilestoneLuckToWeights(baseTiers, tempLuck) {
      const tiers = clone(baseTiers);
      if (tempLuck > 1) {
        for (const t of tiers) {
          if (LUCK_TARGET_KEYS.includes(t.key)) t.weight *= tempLuck;
        }
      }
      return tiers;
    }

    function toChances(tiers) {
      const total = sumWeights(tiers);
      return tiers.map(t => ({ ...t, chance: t.weight / total }));
    }

    function pickTier(chances) {
      const r = Math.random();
      let acc = 0;
      for (let i = chances.length - 1; i >= 0; i--) {
        acc += chances[i].chance;
        if (r <= acc) return { index: i, item: chances[i], roll: r, acc };
      }
      return { index: 0, item: chances[0], roll: r, acc };
    }

    function pickItemNameForTier(tierKey) {
      const list = INDEX_ITEMS[tierKey] || [];
      if (!list.length) return null;
      const i = Math.floor(Math.random() * list.length);
      return list[i];
    }

    // -----------------------------
    // ROLL LOGIC
    function rollOnce() {
      const upcoming = state.rolls + 1;
      const tempLuck = luckForUpcomingRoll(upcoming);

      // Luck banner visible only when activated
      renderLuckBanner(tempLuck);

      const weighted = applyMilestoneLuckToWeights(TIERS, tempLuck);
      const chances = toChances(weighted);
      const tierPick = pickTier(chances);
      const pickedTierKey = tierPick.item.key;
      const pickedTierName = tierPick.item.name;
      const pickedName = pickItemNameForTier(pickedTierKey);

      state.rolls++;

      // Unlock item via roll and show "NEW collected" banner when first obtained
      let isNew = false;
      if (pickedName && !state.unlocks[pickedTierKey][pickedName]) {
        state.unlocks[pickedTierKey][pickedName] = true;
        isNew = true;
      }

      // Add to inventory log
      state.inventory.push({ tier: pickedTierKey, tierName: pickedTierName, name: pickedName, roll: state.rolls, luck: tempLuck });

      saveState();

      showGlow();
      renderResult(tierPick.item, pickedName);
      renderNewBanner(isNew, pickedTierName, pickedName);
      renderStats();
      renderIndex();      // live update
      renderInventory();  // live update

      // Auto unlock at 50 rolls
      if (state.rolls >= 50 && elAutoBtn.disabled) {
        elAutoBtn.disabled = false;
        elAutoBtn.textContent = state.auto ? "Auto Roll: On" : "Auto Roll: Off";
      }
    }

    // -----------------------------
    // AUTO CLICKER
    function toggleAuto() {
      if (elAutoBtn.disabled) return;
      if (state.auto) {
        state.auto = false;
        clearInterval(state.autoInterval);
        state.autoInterval = null;
        elAutoBtn.textContent = "Auto Roll: Off";
      } else {
        state.auto = true;
        elAutoBtn.textContent = "Auto Roll: On";
        state.autoInterval = setInterval(() => {
          if (elAutoBtn.disabled) { toggleAuto(); return; }
          rollOnce();
        }, 120);
      }
      saveState();
    }

    // -----------------------------
    // UI ELEMENTS
    const elLuckBanner = document.getElementById("luckBanner");
    const elNewBanner = document.getElementById("newBanner");
    const elResult = document.getElementById("resultText");
    const elRarity = document.getElementById("rarityText");
    const elRollArea = document.getElementById("rollArea");
    const elRollBtn = document.getElementById("btnRoll");
    const elAutoBtn = document.getElementById("btnAuto");
    const elIndexBtn = document.getElementById("btnIndex");
    const elInventoryBtn = document.getElementById("btnInventory");
    const elIndexPanel = document.getElementById("indexPanel");
    const elIndexGrid = document.getElementById("indexGrid");
    const elInventoryPanel = document.getElementById("inventoryPanel");
    const elInventoryStats = document.getElementById("inventoryStats");
    const elInventoryList = document.getElementById("inventoryList");

    function renderLuckBanner(mult) {
      if (mult > 1) {
        elLuckBanner.textContent = `${mult}x luck activated`;
      } else {
        elLuckBanner.textContent = "";
      }
    }

    function renderNewBanner(isNew, tierName, itemName) {
      if (isNew) {
        elNewBanner.textContent = `NEW collected: [${tierName}] ${itemName}`;
        // fade message after 2.5s
        setTimeout(() => { elNewBanner.textContent = ""; }, 2500);
      } else {
        elNewBanner.textContent = "";
      }
    }

    function renderResult(tierItem, itemName) {
      elResult.textContent = itemName ? `${tierItem.name} — ${itemName}` : tierItem.name;
      elRarity.textContent = tierItem.key.toUpperCase();
      elRarity.className = "rarity badge " + tierItem.colorClass;
    }

    function renderStats() {
      // No header stats texts as requested; keep auto button state only
      const upcoming = state.rolls + 1;
      const mult = luckForUpcomingRoll(upcoming);
      renderLuckBanner(mult);
      if (state.rolls >= 50 && elAutoBtn.disabled) {
        elAutoBtn.disabled = false;
        elAutoBtn.textContent = state.auto ? "Auto Roll: On" : "Auto Roll: Off";
      }
      if (state.rolls < 50) {
        elAutoBtn.disabled = true;
        elAutoBtn.textContent = "Auto Roll (locked)";
      }
    }

    function renderIndex() {
      elIndexGrid.innerHTML = "";
      TIERS.forEach(tier => {
        const section = document.createElement("div");
        section.className = "index-section";
        section.innerHTML = `<h4><span class="badge ${tier.colorClass}">${tier.name}</span></h4>`;
        const ul = document.createElement("ul");
        ul.className = "index-list";

        const items = INDEX_ITEMS[tier.key] || [];
        if (!items.length) {
          const li = document.createElement("li");
          li.className = "index-item";
          li.innerHTML = `<span class="locked">No items defined</span>`;
          ul.appendChild(li);
        } else {
          items.forEach(name => {
            const li = document.createElement("li");
            li.className = "index-item";
            const unlocked = !!state.unlocks[tier.key][name];
            // Show name only (no "Tier - Name" prefix). Blur if locked.
            li.innerHTML = `
              <span class="${unlocked ? "" : "locked"}">${name}</span>
              <span class="${unlocked ? "unlocked" : "locked"}">${unlocked ? "Unlocked" : "Locked"}</span>
            `;
            ul.appendChild(li);
          });
        }
        section.appendChild(ul);
        elIndexGrid.appendChild(section);
      });
    }

    function renderInventory() {
      elInventoryStats.textContent = `Rolls: ${state.rolls} • Items: ${state.inventory.length}`;
      elInventoryList.innerHTML = "";
      // Most recent first
      const items = [...state.inventory].reverse();
      for (const entry of items) {
        const tier = TIERS.find(t => t.key === entry.tier);
        const badgeClass = tier ? tier.colorClass : "";
        const li = document.createElement("li");
        li.style.display = "flex";
        li.style.justifyContent = "space-between";
        li.style.borderBottom = "1px solid #242a38";
        li.style.padding = "6px 0";
        const left = document.createElement("div");
        left.innerHTML = `<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"}`;
        const right = document.createElement("div");
        right.style.color = "var(--muted)";
        right.textContent = `Roll #${entry.roll}${entry.luck > 1 ? ` • ${entry.luck}x` : ""}`;
        li.appendChild(left);
        li.appendChild(right);
        elInventoryList.appendChild(li);
      }
    }

    function showGlow() {
      const glow = document.createElement("div");
      glow.className = "glow";
      elRollArea.appendChild(glow);
      setTimeout(() => glow.remove(), 1100);
    }

    // -----------------------------
    // HOOKS
    elRollBtn.addEventListener("click", rollOnce);
    elAutoBtn.addEventListener("click", toggleAuto);
    elIndexBtn.addEventListener("click", () => {
      const visible = elIndexPanel.style.display !== "none";
      elIndexPanel.style.display = visible ? "none" : "block";
      if (!visible) renderIndex();
    });
    elInventoryBtn.addEventListener("click", () => {
      const visible = elInventoryPanel.style.display !== "none";
      elInventoryPanel.style.display = visible ? "none" : "block";
      if (!visible) renderInventory();
    });

    // -----------------------------
    // INIT
    loadState();
    renderStats();
    // Restore auto if eligible
    if (state.auto && state.rolls >= 50) {
      elAutoBtn.disabled = false;
      elAutoBtn.textContent = "Auto Roll: On";
      state.autoInterval = setInterval(() => {
        if (elAutoBtn.disabled) { toggleAuto(); return; }
        rollOnce();
      }, 120);
    } else {
      elAutoBtn.textContent = state.rolls >= 50 ? "Auto Roll: Off" : "Auto Roll (locked)";
    }
  </script>
</body>
</html>
