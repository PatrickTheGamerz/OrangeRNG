<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol's RNG (browser skeleton)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0e0f13;
      --panel: #151822;
      --text: #e7e9ee;
      --muted: #9aa0ab;
      --accent: #6ea8fe;
      --good: #64d98b;
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
    .content { display: grid; grid-template-columns: 2fr 1fr; gap: 18px; padding: 20px; }
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
    .prob-table { width: 100%; border-collapse: collapse; font-size: 14px; }
    .prob-table th, .prob-table td { padding: 8px 10px; border-bottom: 1px solid #242a38; }
    .prob-table th { text-align: left; color: var(--muted); font-weight: 600; }
    .badge { display: inline-block; padding: 2px 8px; border-radius: 999px; font-size: 12px; font-weight: 700; }
    .b-common { background: #1f2635; color: #c7d1e5; }
    .b-rare { background: #261f35; color: var(--rare); }
    .b-legendary { background: #262018; color: var(--mythic); }
    .b-divine { background: #26240f; color: var(--divine); }
    .footer { padding: 12px 16px; border-top: 1px solid #242a38; font-size: 12px; color: var(--muted); }
    /* Simple glow effect for roll animation */
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
      <div class="brand">Sol’s RNG — browser skeleton</div>
      <div class="stats">
        <div id="stat-rolls">Rolls: 0</div>
        <div id="stat-pity">Pity: 0</div>
        <div id="stat-streak">Streak: 0</div>
      </div>
    </div>
    <div class="content">
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
        </div>
        <div class="controls">
          <button id="btnRoll">Roll</button>
          <button id="btnAuto">Auto (x100)</button>
          <button id="btnReset">Reset</button>
        </div>
      </div>
      <div class="panel">
        <h3 style="margin:0 0 10px;">Rarities and weights</h3>
        <table class="prob-table" id="probTable">
          <thead><tr><th>Rarity</th><th>Weight</th><th>Chance</th></tr></thead>
          <tbody></tbody>
        </table>
        <div style="margin-top:10px; font-size:12px; color:var(--muted);">
          Edit weights in code to match Sol’s exact data. Supports pity, streaks, and conditional boosts.
        </div>
      </div>
    </div>
    <div class="footer">Plug in accurate tier data and rules for a 1:1 match. This is intentionally minimal and modular.</div>
  </div>

  <script>
    // -----------------------------
    // CONFIG: Define rarities & weights
    // Replace these with Sol’s exact values for a 1:1 build.
    // weight is relative; final chance = weight / sum(weights), unless modified by rules below.
    const BASE_RARITIES = [
      { key: "common",     name: "Common",     weight: 980000, colorClass: "b-common" },
      { key: "uncommon",   name: "Uncommon",   weight: 18000,  colorClass: "b-common" },
      { key: "rare",       name: "Rare",       weight: 1800,   colorClass: "b-rare" },
      { key: "epic",       name: "Epic",       weight: 180,    colorClass: "b-rare" },
      { key: "legendary",  name: "Legendary",  weight: 18,     colorClass: "b-legendary" },
      { key: "divine",     name: "Divine",     weight: 1,      colorClass: "b-divine" },
    ];
    // Rules toggles — enable once you know exact mechanics
    const RULES = {
      pityEnabled: true,
      pityTargetKeys: ["legendary", "divine"], // which rarities pity affects
      pityStep: 0.000002,  // additive chance per roll (placeholder)
      pityMaxBoost: 0.001, // cap boost (placeholder)
      streakEnabled: true,
      streakWindow: 10,       // track last N rolls
      streakBoostKey: "rare", // boost target when dry
      streakBoost: 0.10,      // +10% weight when dry (placeholder)
    };

    // -----------------------------
    // STATE
    const state = {
      rolls: 0,
      pity: 0,      // accumulated pity factor (not percent — an additive chance)
      streak: 0,    // count of rolls since last hit of >= streakThresholdTier
      lastHitTierIndex: null,
      history: [],
    };

    // Tier order is array index; higher index = rarer tier.
    // Define streak threshold (e.g., hitting epic or above resets streak)
    const STREAK_THRESHOLD_INDEX = 3; // epic or higher resets streak

    // -----------------------------
    // UTILS
    function clone(obj) { return JSON.parse(JSON.stringify(obj)); }

    function sumWeights(arr) { return arr.reduce((s, r) => s + r.weight, 0); }

    function applyRules(base) {
      const tiers = clone(base);

      // Streak boost: if dry (no epic+ for N rolls), boost a target tier
      if (RULES.streakEnabled && state.streak >= RULES.streakWindow) {
        const idx = tiers.findIndex(t => t.key === RULES.streakBoostKey);
        if (idx >= 0) tiers[idx].weight = tiers[idx].weight * (1 + RULES.streakBoost);
      }

      // Convert to chances
      const total = sumWeights(tiers);
      let chances = tiers.map(t => ({ ...t, chance: t.weight / total }));

      // Pity: distribute additive chance across target rarities (normalized)
      if (RULES.pityEnabled && state.pity > 0) {
        const targets = chances.filter(c => RULES.pityTargetKeys.includes(c.key));
        const baseSum = targets.reduce((s, t) => s + t.chance, 0);
        if (baseSum > 0) {
          const boost = Math.min(state.pity, RULES.pityMaxBoost);
          // Apportion boost proportional to each target's base chance
          chances = chances.map(c => {
            if (!RULES.pityTargetKeys.includes(c.key)) return c;
            const proportion = c.chance / baseSum;
            return { ...c, chance: c.chance + boost * proportion };
          });
          // Re-normalize by shaving the boost from the non-targets proportionally
          const surplus = chances.reduce((s, c) => s + c.chance, 0) - 1;
          if (surplus > 1e-12) {
            const nonTargets = chances.filter(c => !RULES.pityTargetKeys.includes(c.key));
            const nonSum = nonTargets.reduce((s, c) => s + c.chance, 0);
            chances = chances.map(c => {
              if (RULES.pityTargetKeys.includes(c.key)) return c;
              const proportion = c.chance / nonSum;
              return { ...c, chance: c.chance - surplus * proportion };
            });
          }
        }
      }
      return chances;
    }

    function pickTier(chances) {
      const r = Math.random();
      let acc = 0;
      for (let i = chances.length - 1; i >= 0; i--) {
        // iterate from rarest to common to reduce floating errors near 1
        acc += chances[i].chance;
        if (r <= acc) return { index: i, item: chances[i], roll: r, acc };
      }
      // Fallback
      return { index: 0, item: chances[0], roll: r, acc };
    }

    function formatChance(p) {
      if (p >= 0.01) return (p * 100).toFixed(2) + "%";
      if (p >= 0.0001) return (p * 100).toFixed(4) + "%";
      return (p * 100).toFixed(6) + "%";
    }

    // -----------------------------
    // ROLL LOGIC
    function rollOnce() {
      const chances = applyRules(BASE_RARITIES);
      const result = pickTier(chances);
      state.rolls++;
      // Update pity
      if (RULES.pityEnabled) {
        state.pity = Math.min(state.pity + RULES.pityStep, RULES.pityMaxBoost);
      }
      // Update streak: reset if epic+ hit
      if (result.index >= STREAK_THRESHOLD_INDEX) {
        state.streak = 0;
        state.lastHitTierIndex = result.index;
      } else {
        state.streak++;
      }
      // Persist history
      state.history.push({ key: result.item.key, index: result.index });

      // UI update
      showGlow();
      renderResult(result.item);
      renderStats();
    }

    function autoRoll(times = 100) {
      for (let i = 0; i < times; i++) rollOnce();
    }

    // -----------------------------
    // UI
    const elResult = document.getElementById("resultText");
    const elRarity = document.getElementById("rarityText");
    const elRolls  = document.getElementById("stat-rolls");
    const elPity   = document.getElementById("stat-pity");
    const elStreak = document.getElementById("stat-streak");
    const elProbTableBody = document.querySelector("#probTable tbody");
    const elRollArea = document.getElementById("rollArea");

    function renderResult(item) {
      elResult.textContent = item.name;
      elRarity.textContent = item.key.toUpperCase();
      elRarity.className = "rarity badge " + (
        item.key === "divine" ? "b-divine" :
        item.key === "legendary" ? "b-legendary" :
        item.key === "epic" || item.key === "rare" ? "b-rare" :
        "b-common"
      );
    }

    function renderStats() {
      elRolls.textContent = "Rolls: " + state.rolls;
      elPity.textContent  = "Pity: " + (state.pity * 100).toFixed(4) + "%";
      elStreak.textContent= "Streak: " + state.streak;
      renderTable();
    }

    function renderTable() {
      const chances = applyRules(BASE_RARITIES);
      elProbTableBody.innerHTML = chances.map(c => `
        <tr>
          <td>${c.name}</td>
          <td>${c.weight.toLocaleString()}</td>
          <td>${formatChance(c.chance)}</td>
        </tr>
      `).join("");
    }

    function showGlow() {
      const glow = document.createElement("div");
      glow.className = "glow";
      elRollArea.appendChild(glow);
      setTimeout(() => glow.remove(), 1100);
    }

    // -----------------------------
    // HOOKS
    document.getElementById("btnRoll").addEventListener("click", rollOnce);
    document.getElementById("btnAuto").addEventListener("click", () => autoRoll(100));
    document.getElementById("btnReset").addEventListener("click", () => {
      state.rolls = 0; state.pity = 0; state.streak = 0; state.history = []; state.lastHitTierIndex = null;
      elResult.textContent = "Ready to roll"; elRarity.textContent = "";
      renderStats();
    });

    // init
    renderStats();
  </script>
</body>
</html>
