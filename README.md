<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol's RNG — Index + Luck milestones</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0e0f13;
      --panel: #151822;
      --text: #e7e9ee;
      --muted: #9aa0ab;
      --accent: #6ea8fe;
      --rare: #caa6ff;
      --mythic: #ffbf66;
      --divine: #ffd700;
    }
    body {
      margin: 0;
      font-family: system-ui, Segoe UI, Roboto, Arial, sans-serif;
      background: radial-gradient(1200px 600px at 50% -200px, #1a2030 0%, var(--bg) 50%, var(--bg) 100%);
      color: var(--text);
      min-height: 100vh; display: grid; place-items: center;
    }
    .app {
      width: 900px; max-width: 96vw;
      background: linear-gradient(180deg, #141821 0%, var(--panel) 100%);
      border: 1px solid #252b39; border-radius: 14px;
      box-shadow: 0 20px 60px rgba(0,0,0,0.5);
      overflow: hidden;
    }
    .header {
      padding: 16px 20px; border-bottom: 1px solid #242a38;
      display: flex; align-items: center; justify-content: space-between;
    }
    .brand { font-weight: 700; letter-spacing: 0.2px; }
    .stats { font-size: 14px; color: var(--muted); display: flex; gap: 18px; }
    .content { display: grid; grid-template-columns: 1fr; gap: 18px; padding: 20px; }
    .panel {
      background: #121521; border: 1px solid #242a38; border-radius: 12px; padding: 16px;
    }
    .roll-area { height: 240px; display: grid; place-items: center; position: relative; overflow: hidden; }
    .result {
      font-size: 28px; font-weight: 700; text-align: center;
      text-shadow: 0 0 12px rgba(255,255,255,0.15);
    }
    .rarity {
      margin-top: 6px; font-size: 14px; font-weight: 600; letter-spacing: 0.3px; text-transform: uppercase;
    }
    .controls { display: flex; gap: 12px; padding-top: 12px; }
    button {
      background: #1b2232; color: var(--text); border: 1px solid #2a3449;
      padding: 12px 16px; border-radius: 10px; cursor: pointer; font-weight: 600;
      transition: transform 0.06s ease, background 0.2s ease, box-shadow 0.2s ease;
    }
    button:hover { background: #232c41; box-shadow: 0 6px 18px rgba(110,168,254,0.12); }
    button:active { transform: translateY(1px); }
    button:disabled { opacity: 0.45; cursor: not-allowed; }
    .footer { padding: 12px 16px; border-top: 1px solid #242a38; font-size: 12px; color: var(--muted); }

    /* Index styles */
    .index-grid {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 12px;
    }
    .index-section {
      background: #0f1320; border: 1px solid #252b39; border-radius: 10px; padding: 12px;
    }
    .index-section h4 { margin: 0 0 8px; font-size: 14px; color: var(--muted); }
    .index-list { list-style: none; padding: 0; margin: 0; }
    .index-item {
      display: flex; justify-content: space-between; align-items: center;
      padding: 6px 0; border-bottom: 1px solid #242a38; font-size: 14px;
    }
    .badge { display: inline-block; padding: 2px 8px; border-radius: 999px; font-size: 12px; font-weight: 700; }
    .b-common { background: #1f2635; color: #c7d1e5; }
    .b-rare { background: #261f35; color: var(--rare); }
    .b-legendary { background: #262018; color: var(--mythic); }
    .b-divine { background: #26240f; color: var(--divine); }
    .locked { color: var(--muted); }
    .unlocked { color: var(--accent); font-weight: 600; }

    /* Glow effect for roll animation */
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
    <div class="header">
      <div class="brand">Sol’s RNG — Index + Luck milestones</div>
      <div class="stats">
        <div id="stat-rolls">Rolls: 0</div>
        <div id="stat-pity">Pity: 1x</div>
        <div id="stat-luck">Luck: 1x (milestone)</div>
        <div id="stat-index">Index: Locked</div>
      </div>
    </div>

    <div class="content">
      <!-- Rolling panel -->
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
        </div>
        <div class="controls">
          <button id="btnRoll">Roll</button>
          <!-- Index button sits where the old Auto button was, and is unlockable -->
          <button id="btnIndex" disabled>INDEX (locked)</button>
        </div>
      </div>

      <!-- Index panel (always visible below rolling; button toggles highlight/open state if desired) -->
      <div class="panel">
        <h3 style="margin:0 0 10px;">Index</h3>
        <div class="index-grid" id="indexGrid"></div>
        <div style="margin-top:10px; font-size:12px; color:var(--muted);">
          Roll to unlock items. Index button unlocks at 100 rolls. Luck milestones: every 50 rolls → 2x for that roll; every 250 rolls → 10x for that roll.
        </div>
      </div>
    </div>

    <div class="footer">Pity is fixed at 1x display-only. Luck applies only to the single milestone roll.</div>
  </div>

  <script>
    // -----------------------------
    // CONFIG: Define tiers, weights, and item names per rarity
    // Higher index = rarer tier
    const TIERS = [
      { key: "common",     name: "Common",     weight: 980000, colorClass: "b-common" },
      { key: "uncommon",   name: "Uncommon",   weight: 18000,  colorClass: "b-common" },
      { key: "rare",       name: "Rare",       weight: 1800,   colorClass: "b-rare" },
      { key: "epic",       name: "Epic",       weight: 180,    colorClass: "b-rare" },
      { key: "legendary",  name: "Legendary",  weight: 18,     colorClass: "b-legendary" },
      { key: "divine",     name: "Divine",     weight: 1,      colorClass: "b-divine" },
    ];

    // Items per rarity (expand freely)
    const INDEX_ITEMS = {
      common:    ["Stone Pebble", "Wood Stick", "Rusty Nail"],
      uncommon:  ["Copper Charm", "Traveler's Map", "Old Compass"],
      rare:      ["Crystal Shard", "Moonflower", "Ancient Coin"],
      epic:      ["Dragon Scale", "Starlight Sigil"],
      legendary: ["Phoenix Feather", "Eternal Hourglass"],
      divine:    ["Celestial Orb", "Sol's Tear"]
    };

    // Luck affects higher tiers only
    const LUCK_TARGET_KEYS = ["rare","epic","legendary","divine"];

    // -----------------------------
    // STATE
    const state = {
      rolls: 0,
      history: [],
      unlocks: buildInitialUnlocks(INDEX_ITEMS), // {rarityKey: {itemName: false}}
      indexUnlocked: false, // button unlock gated by rolls >= 100
    };

    function buildInitialUnlocks(indexDef) {
      const out = {};
      for (const key in indexDef) {
        out[key] = {};
        indexDef[key].forEach(name => out[key][name] = false);
      }
      return out;
    }

    // -----------------------------
    // UTILS
    function clone(obj) { return JSON.parse(JSON.stringify(obj)); }
    function sumWeights(arr) { return arr.reduce((s, r) => s + r.weight, 0); }

    // Luck milestones: apply to the current roll only
    function rollLuckMultiplierForUpcomingRoll(upcomingRollNumber) {
      // 250-roll milestone overrides 50-roll milestone
      if (upcomingRollNumber % 250 === 0) return 10;
      if (upcomingRollNumber % 50 === 0) return 2;
      return 1;
    }

    function applyMilestoneLuckToWeights(baseTiers, tempLuck) {
      const tiers = clone(baseTiers);
      if (tempLuck > 1) {
        for (const t of tiers) {
          if (LUCK_TARGET_KEYS.includes(t.key)) {
            t.weight *= tempLuck;
          }
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
      const list = INDEX_ITEMS[tierKey];
      if (!list || list.length === 0) return null;
      const i = Math.floor(Math.random() * list.length);
      return list[i];
    }

    function formatMilestoneLuckText(upcomingRollNumber) {
      const mult = rollLuckMultiplierForUpcomingRoll(upcomingRollNumber);
      return mult + "x (milestone)";
    }

    // -----------------------------
    // ROLL LOGIC
    function rollOnce() {
      // Determine luck for THIS roll from milestone (based on upcoming roll number)
      const upcoming = state.rolls + 1;
      const tempLuck = rollLuckMultiplierForUpcomingRoll(upcoming);

      // Compute chances with temp luck applied to higher tiers
      const weighted = applyMilestoneLuckToWeights(TIERS, tempLuck);
      const chances = toChances(weighted);
      const tierPick = pickTier(chances);

      // Pick specific item name within the chosen tier
      const pickedTierKey = tierPick.item.key;
      const pickedName = pickItemNameForTier(pickedTierKey);

      // Increment rolls AFTER computing milestone
      state.rolls++;

      // Unlock in index if not already
      if (pickedName) {
        if (!state.unlocks[pickedTierKey][pickedName]) {
          state.unlocks[pickedTierKey][pickedName] = true;
        }
      }

      // Persist history (tier + item)
      state.history.push({ tier: pickedTierKey, name: pickedName });

      // UI updates
      showGlow();
      renderResult(tierPick.item, pickedName);
      renderStats();
      renderIndex(); // reflect unlocked states

      // Unlock the Index button at 100 rolls (replacing old auto unlock)
      if (!state.indexUnlocked && state.rolls >= 100) {
        state.indexUnlocked = true;
        elIndexBtn.disabled = false;
        elIndexBtn.textContent = "INDEX";
        renderStats();
      }
    }

    // -----------------------------
    // UI
    const elResult = document.getElementById("resultText");
    const elRarity = document.getElementById("rarityText");
    const elRolls  = document.getElementById("stat-rolls");
    const elPity   = document.getElementById("stat-pity");
    const elLuck   = document.getElementById("stat-luck");
    const elIndexStat = document.getElementById("stat-index");
    const elRollArea = document.getElementById("rollArea");
    const elIndexGrid = document.getElementById("indexGrid");
    const elRollBtn = document.getElementById("btnRoll");
    const elIndexBtn = document.getElementById("btnIndex");

    function renderResult(tierItem, itemName) {
      elResult.textContent = itemName ? `${tierItem.name} — ${itemName}` : tierItem.name;
      elRarity.textContent = tierItem.key.toUpperCase();
      elRarity.className = "rarity badge " + (
        tierItem.key === "divine" ? "b-divine" :
        tierItem.key === "legendary" ? "b-legendary" :
        tierItem.key === "epic" || tierItem.key === "rare" ? "b-rare" :
        "b-common"
      );
    }

    function renderStats() {
      elRolls.textContent = "Rolls: " + state.rolls;
      elPity.textContent  = "Pity: 1x";
      elLuck.textContent  = "Luck: " + formatMilestoneLuckText(state.rolls + 1);
      elIndexStat.textContent = "Index: " + (state.indexUnlocked ? "Unlocked" : "Locked");
    }

    function renderIndex() {
      // Build sections per rarity, showing names with lock state (no manual toggles)
      elIndexGrid.innerHTML = "";
      TIERS.forEach(tier => {
        const section = document.createElement("div");
        section.className = "index-section";
        section.innerHTML = `<h4><span class="badge ${tier.colorClass}">${tier.name}</span></h4>`;
        const ul = document.createElement("ul");
        ul.className = "index-list";

        const items = INDEX_ITEMS[tier.key] || [];
        if (items.length === 0) {
          const li = document.createElement("li");
          li.className = "index-item";
          li.innerHTML = `<span class="locked">No items defined</span>`;
          ul.appendChild(li);
        } else {
          items.forEach(name => {
            const li = document.createElement("li");
            li.className = "index-item";
            const isUnlocked = !!state.unlocks[tier.key][name];
            li.innerHTML = `
              <span>${tier.name} - ${name}</span>
              <span class="${isUnlocked ? "unlocked" : "locked"}">${isUnlocked ? "Unlocked" : "Locked"}</span>
            `;
            ul.appendChild(li);
          });
        }
        section.appendChild(ul);
        elIndexGrid.appendChild(section);
      });
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

    // Index button is simply present in controls (replacing the old auto button),
    // but locked until 100 rolls. Here it can act as a quick scroll-to-index or
    // visual cue; we’ll scroll to the index panel when clicked.
    elIndexBtn.addEventListener("click", () => {
      if (elIndexBtn.disabled) return;
      // Scroll to index panel
      document.querySelector(".content .panel:nth-child(2)").scrollIntoView({ behavior: "smooth", block: "start" });
    });

    // -----------------------------
    // INIT
    renderStats();
    renderIndex();
  </script>
</body>
</html>
