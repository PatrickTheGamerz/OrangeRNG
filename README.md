<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG 1:1 Style Simulator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0b0f17;
      --panel: #121826;
      --text: #e8ecf1;
      --muted: #a8b0bf;
      --accent: #6ee7ff;
      --good: #9cff6e;
      --mid: #ffd56e;
      --rare: #ff6ee7;
      --ultra: #a46eff;
      --border: #222b3a;
    }

    html, body {
      background: radial-gradient(1200px at 50% -200px, #162036, var(--bg));
      color: var(--text);
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
      margin: 0;
      padding: 0;
    }

    .app {
      max-width: 960px;
      margin: 36px auto;
      padding: 0 16px;
    }

    header {
      display: flex;
      align-items: baseline;
      justify-content: space-between;
      margin-bottom: 24px;
    }

    h1 {
      font-size: 1.6rem;
      margin: 0;
      letter-spacing: 0.2px;
    }

    .subtitle {
      color: var(--muted);
      font-size: 0.95rem;
    }

    .grid {
      display: grid;
      grid-template-columns: 1.25fr 1fr;
      gap: 16px;
    }

    .panel {
      background: linear-gradient(180deg, #121826, #0e1523);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 16px;
      box-shadow: 0 6px 30px rgba(0,0,0,0.35);
    }

    .panel h2 {
      font-size: 1.1rem;
      margin: 0 0 10px 0;
      color: var(--accent);
      font-weight: 600;
      letter-spacing: 0.2px;
    }

    .controls {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 16px;
      margin-bottom: 12px;
    }

    label {
      display: block;
      font-size: 0.9rem;
      color: var(--muted);
      margin-bottom: 6px;
    }

    input[type="number"], select {
      width: 100%;
      border: 1px solid var(--border);
      background: #0b101a;
      color: var(--text);
      padding: 10px 12px;
      border-radius: 10px;
      outline: none;
      font-size: 0.95rem;
    }

    .buttons {
      display: flex;
      gap: 10px;
      margin: 10px 0 16px;
      flex-wrap: wrap;
    }

    button {
      background: #182238;
      color: var(--text);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 14px;
      font-size: 0.95rem;
      cursor: pointer;
      transition: transform 0.06s ease, background 0.2s ease;
    }

    button:hover { background: #1b2744; }
    button:active { transform: translateY(1px); }

    .result {
      margin-top: 8px;
      padding: 14px;
      border-radius: 10px;
      border: 1px dashed var(--border);
      background: #0c1320;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 10px;
    }

    .badge {
      display: inline-block;
      padding: 6px 10px;
      border-radius: 999px;
      font-weight: 600;
      letter-spacing: 0.2px;
      font-size: 0.9rem;
      border: 1px solid var(--border);
    }

    .tier-common     { background: #10161f; color: #b8c2d6; }
    .tier-uncommon   { background: #132019; color: var(--good); }
    .tier-rare       { background: #1f1a10; color: var(--mid); }
    .tier-epic       { background: #1d1220; color: var(--rare); }
    .tier-legendary  { background: #161126; color: var(--ultra); }
    .tier-exotic     { background: #231321; color: #ffa7ef; }
    .tier-divine     { background: #0f1f26; color: #83e8ff; }
    .tier-celestial  { background: #0d1b24; color: #a6e4ff; }
    .tier-eternal    { background: #0b1521; color: #c9f0ff; }

    .rarity-table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 8px;
    }

    .rarity-table th, .rarity-table td {
      border-bottom: 1px solid var(--border);
      text-align: left;
      padding: 8px;
      font-size: 0.92rem;
    }

    .small {
      color: var(--muted);
      font-size: 0.86rem;
    }

    .flex {
      display: flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
    }

    .history {
      max-height: 280px;
      overflow: auto;
      border: 1px solid var(--border);
      border-radius: 10px;
      background: #0c1320;
    }

    .history-item {
      display: flex;
      justify-content: space-between;
      padding: 10px 12px;
      border-bottom: 1px solid #0f172a;
    }

    .stats {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      margin-top: 12px;
    }

    .stat {
      background: #0c1320;
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 12px;
    }

    .stat .label { color: var(--muted); font-size: 0.85rem; }
    .stat .value { font-size: 1.1rem; font-weight: 600; margin-top: 6px; }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div>
        <h1>Sol’s RNG style simulator</h1>
        <div class="subtitle">Configurable rarity odds, luck multiplier, and soft pity. Minimal, readable, and modular.</div>
      </div>
      <div class="subtitle">Client-side only</div>
    </header>

    <div class="grid">
      <section class="panel">
        <h2>Controls</h2>
        <div class="controls">
          <div>
            <label>Luck multiplier (1–10)</label>
            <input type="number" id="luck" min="1" max="10" step="0.1" value="1" />
          </div>
          <div>
            <label>Soft pity threshold</label>
            <input type="number" id="pityThreshold" min="0" step="10" value="200" />
          </div>
          <div>
            <label>Pity boost per roll (%)</label>
            <input type="number" id="pityBoostPct" min="0" step="0.1" value="0.25" />
          </div>
          <div>
            <label>RNG seed (optional)</label>
            <input type="number" id="seed" placeholder="e.g., 12345" />
          </div>
        </div>

        <div class="buttons">
          <button id="roll1">Roll ×1</button>
          <button id="roll10">Roll ×10</button>
          <button id="roll100">Roll ×100</button>
          <button id="reset">Reset stats</button>
        </div>

        <div id="result" class="result">
          <span class="badge tier-common">Waiting to roll…</span>
          <span class="small">No result yet</span>
        </div>

        <div class="stats">
          <div class="stat">
            <div class="label">Total rolls</div>
            <div id="statRolls" class="value">0</div>
          </div>
          <div class="stat">
            <div class="label">Pity counter</div>
            <div id="statPity" class="value">0</div>
          </div>
          <div class="stat">
            <div class="label">Best obtained</div>
            <div id="statBest" class="value">—</div>
          </div>
        </div>
      </section>

      <section class="panel">
        <h2>Rarity table</h2>
        <table class="rarity-table" id="rarityTable">
          <thead>
            <tr>
              <th>Tier</th>
              <th>Base odds</th>
              <th>Effective odds</th>
            </tr>
          </thead>
          <tbody id="rarityBody"></tbody>
        </table>
        <p class="small">Edit the rarity config in code to match exact 1:1 odds or add more tiers.</p>

        <h2 style="margin-top:14px;">History</h2>
        <div class="history" id="history"></div>
      </section>
    </div>
  </div>

  <script>
    // =============== CONFIG ===============

    // Rarity tiers, ordered from least rare to most rare.
    // Use probability in decimal (e.g., 0.2 = 20%). The system normalizes and rolls from top to bottom.
    // For a closer Sol’s feel, keep many rarities with ultra-low odds.
    const RARITIES = [
      { key: "common",     name: "Common",     prob: 0.80,   colorClass: "tier-common"    },
      { key: "uncommon",   name: "Uncommon",   prob: 0.18,   colorClass: "tier-uncommon"  },
      { key: "rare",       name: "Rare",       prob: 0.015,  colorClass: "tier-rare"      },
      { key: "epic",       name: "Epic",       prob: 0.003,  colorClass: "tier-epic"      },
      { key: "legendary",  name: "Legendary",  prob: 0.001,  colorClass: "tier-legendary" },
      { key: "exotic",     name: "Exotic",     prob: 0.0004, colorClass: "tier-exotic"    },
      { key: "divine",     name: "Divine",     prob: 0.00015,colorClass: "tier-divine"    },
      { key: "celestial",  name: "Celestial",  prob: 0.00005,colorClass: "tier-celestial" },
      { key: "eternal",    name: "Eternal",    prob: 0.00001,colorClass: "tier-eternal"   }
    ];

    // Soft pity targets higher tiers by scaling their weight after many misses.
    const PITY_APPLIES_TO_KEYS = new Set([
      "rare","epic","legendary","exotic","divine","celestial","eternal"
    ]);

    // =============== STATE ===============
    const state = {
      rolls: 0,
      pityCounter: 0,
      seed: null,
      bestIndex: null,
      // tally by key
      counts: Object.fromEntries(RARITIES.map(r => [r.key, 0]))
    };

    // =============== RNG ===============
    // Mulberry32 PRNG for optional deterministic rolls.
    function mulberry32(seed) {
      return function() {
        let t = seed += 0x6D2B79F5;
        t = Math.imul(t ^ (t >>> 15), t | 1);
        t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
        return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
      }
    }

    let rng = Math.random;
    function reseed(value) {
      if (value === null || value === undefined || value === "") {
        rng = Math.random;
        state.seed = null;
      } else {
        const s = Number(value);
        if (!Number.isNaN(s)) {
          rng = mulberry32(s >>> 0);
          state.seed = s >>> 0;
        }
      }
    }

    // =============== PITY + LUCK ===============
    function getEffectiveProbs() {
      const luck = clamp(Number(document.getElementById("luck").value) || 1, 1, 10);
      const pityThreshold = Math.max(0, Number(document.getElementById("pityThreshold").value) || 0);
      const pityBoostPct = Math.max(0, Number(document.getElementById("pityBoostPct").value) || 0);

      // Base weights = prob * luckFactorForRares
      const weights = RARITIES.map(r => {
        const base = r.prob;
        const isRareUp = PITY_APPLIES_TO_KEYS.has(r.key);
        const luckFactor = isRareUp ? luck : 1;
        let w = base * luckFactor;

        // Soft pity: after threshold, each miss increases rare-tier weights by pityBoostPct% per roll past threshold.
        if (isRareUp && state.pityCounter > pityThreshold && pityThreshold > 0) {
          const over = state.pityCounter - pityThreshold;
          const boost = 1 + (over * (pityBoostPct / 100));
          w *= boost;
        }
        return w;
      });

      // Normalize to sum = 1
      const sum = weights.reduce((a, b) => a + b, 0);
      const eff = weights.map(w => w / (sum || 1));
      return eff;
    }

    function clamp(x, min, max) { return Math.max(min, Math.min(max, x)); }

    // =============== ROLL LOGIC ===============
    function rollOnce() {
      const eff = getEffectiveProbs();
      let x = rng();
      let acc = 0;
      let index = 0;
      for (let i = 0; i < eff.length; i++) {
        acc += eff[i];
        if (x <= acc) { index = i; break; }
      }
      const hit = RARITIES[index];

      // Update state
      state.rolls++;
      state.counts[hit.key]++;
      // Pity resets on hitting any rare-up tier; otherwise increments
      if (PITY_APPLIES_TO_KEYS.has(hit.key)) {
        state.pityCounter = 0;
      } else {
        state.pityCounter++;
      }
      if (state.bestIndex === null || index > state.bestIndex) {
        state.bestIndex = index;
      }

      renderResult(hit, eff[index]);
      pushHistory(hit, eff[index]);
      renderStats();
      renderTable();
      return hit;
    }

    function rollMany(n) {
      for (let i = 0; i < n; i++) rollOnce();
    }

    // =============== UI ===============
    const els = {
      history: document.getElementById("history"),
      rarityBody: document.getElementById("rarityBody"),
      result: document.getElementById("result"),
      statRolls: document.getElementById("statRolls"),
      statPity: document.getElementById("statPity"),
      statBest: document.getElementById("statBest")
    };

    function renderTable() {
      const eff = getEffectiveProbs();
      const rows = RARITIES.map((r, i) => {
        const basePct = (r.prob * 100).toFixed(r.prob < 0.001 ? 5 : 3) + "%";
        const effPct = (eff[i] * 100).toFixed(eff[i] < 0.001 ? 5 : 3) + "%";
        return `
          <tr>
            <td><span class="badge ${r.colorClass}">${r.name}</span></td>
            <td>${basePct}</td>
            <td>${effPct}</td>
          </tr>
        `;
      }).join("");
      els.rarityBody.innerHTML = rows;
    }

    function renderResult(hit, effProb) {
      els.result.innerHTML = `
        <span class="badge ${hit.colorClass}">${hit.name}</span>
        <span class="small">Effective odds: ${(effProb * 100).toFixed(effProb < 0.001 ? 5 : 3)}%</span>
      `;
    }

    function pushHistory(hit, effProb) {
      const item = document.createElement("div");
      item.className = "history-item";
      item.innerHTML = `
        <span class="badge ${hit.colorClass}">${hit.name}</span>
        <span class="small">roll #${state.rolls} • ${(effProb * 100).toFixed(effProb < 0.001 ? 5 : 3)}%</span>
      `;
      els.history.prepend(item);
      // Limit history size
      const maxItems = 400;
      while (els.history.children.length > maxItems) {
        els.history.removeChild(els.history.lastChild);
      }
    }

    function renderStats() {
      els.statRolls.textContent = state.rolls;
      els.statPity.textContent = state.pityCounter;
      els.statBest.textContent = state.bestIndex == null ? "—" : RARITIES[state.bestIndex].name;
    }

    function resetAll() {
      state.rolls = 0;
      state.pityCounter = 0;
      state.bestIndex = null;
      state.counts = Object.fromEntries(RARITIES.map(r => [r.key, 0]));
      els.history.innerHTML = "";
      renderStats();
      renderTable();
      els.result.innerHTML = `
        <span class="badge tier-common">Ready</span>
        <span class="small">Adjust luck/pity and roll</span>
      `;
    }

    // =============== EVENTS ===============
    document.getElementById("roll1").addEventListener("click", () => rollOnce());
    document.getElementById("roll10").addEventListener("click", () => rollMany(10));
    document.getElementById("roll100").addEventListener("click", () => rollMany(100));
    document.getElementById("reset").addEventListener("click", resetAll);

    document.getElementById("luck").addEventListener("input", renderTable);
    document.getElementById("pityThreshold").addEventListener("input", renderTable);
    document.getElementById("pityBoostPct").addEventListener("input", renderTable);
    document.getElementById("seed").addEventListener("change", (e) => reseed(e.target.value));

    // =============== INIT ===============
    renderTable();
    renderStats();
  </script>
</body>
</html>
```
