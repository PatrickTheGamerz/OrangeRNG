<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol's RNG — Index Demo</title>
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
    .footer { padding: 12px 16px; border-top: 1px solid #242a38; font-size: 12px; color: var(--muted); }
    .index-list { list-style: none; padding: 0; margin: 0; }
    .index-list li {
      display: flex; justify-content: space-between; align-items: center;
      padding: 6px 0; border-bottom: 1px solid #242a38;
    }
    .locked { color: var(--muted); }
    .unlocked { color: var(--accent); font-weight: 600; }
  </style>
</head>
<body>
  <div class="app">
    <div class="header">
      <div class="brand">Sol’s RNG — Index Demo</div>
      <div class="stats">
        <div id="stat-rolls">Rolls: 0</div>
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
          <button id="btnIndex">INDEX</button>
        </div>
      </div>
      <div class="panel" id="indexPanel" style="display:none;">
        <h3 style="margin:0 0 10px;">Index</h3>
        <ul class="index-list" id="indexList"></ul>
      </div>
    </div>
    <div class="footer">Click INDEX to view all rarities and toggle unlocked status.</div>
  </div>

  <script>
    // Example rarities with names
    const RARITY_ITEMS = [
      { rarity: "Common", name: "Stone Pebble", unlocked: false },
      { rarity: "Common", name: "Wood Stick", unlocked: false },
      { rarity: "Rare", name: "Crystal Shard", unlocked: false },
      { rarity: "Epic", name: "Dragon Scale", unlocked: false },
      { rarity: "Legendary", name: "Phoenix Feather", unlocked: false },
      { rarity: "Divine", name: "Celestial Orb", unlocked: false }
    ];

    const state = { rolls: 0 };

    const elResult = document.getElementById("resultText");
    const elRarity = document.getElementById("rarityText");
    const elRolls  = document.getElementById("stat-rolls");
    const elIndexPanel = document.getElementById("indexPanel");
    const elIndexList = document.getElementById("indexList");

    function renderIndex() {
      elIndexList.innerHTML = "";
      RARITY_ITEMS.forEach((item, idx) => {
        const li = document.createElement("li");
        li.innerHTML = `<span>${item.rarity} - ${item.name}</span>
                        <button>${item.unlocked ? "Unlocked" : "Locked"}</button>`;
        const btn = li.querySelector("button");
        btn.className = item.unlocked ? "unlocked" : "locked";
        btn.addEventListener("click", () => {
          item.unlocked = !item.unlocked;
          renderIndex();
        });
        elIndexList.appendChild(li);
      });
    }

    function rollOnce() {
      state.rolls++;
      elResult.textContent = "You rolled something!";
      elRarity.textContent = "";
      elRolls.textContent = "Rolls: " + state.rolls;
    }

    document.getElementById("btnRoll").addEventListener("click", rollOnce);
    document.getElementById("btnIndex").addEventListener("click", () => {
      elIndexPanel.style.display = elIndexPanel.style.display === "none" ? "block" : "none";
      renderIndex();
    });

    // init
    renderIndex();
  </script>
</body>
</html>
