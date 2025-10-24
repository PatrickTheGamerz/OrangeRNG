<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Ultra Weather FX, Per-Item Cutscenes, Robust Commands, Eternal+ Origin</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root{
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab; --accent:#6ea8fe;
      --gold:#ffd700; --warn:#ff6666; --fx1:#7bb7ff; --fx2:#caa6ff; --fx3:#ffbf66;
      --good:#79f2ff; --boom:#ff9cff; --aurora:#7affff; --storm:#7bb7ff; --meteor:#ffbf66;
    }
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,'Helvetica Neue',sans-serif;background:var(--bg);color:var(--text);display:grid;place-items:center;min-height:100vh;}
    .app{width:980px;max-width:96vw;background:var(--panel);border:1px solid #252b39;border-radius:14px;box-shadow:0 20px 60px rgba(0,0,0,0.5);overflow:hidden;position:relative;}
    .content{padding:20px;display:grid;gap:18px;}
    .panel{background:#121521;border:1px solid #242a38;border-radius:12px;padding:16px;}

    /* Roll area */
    .roll-area{min-height:380px;display:grid;place-items:center;position:relative;overflow:hidden;}
    .result{font-size:28px;font-weight:700;text-align:center;position:relative;z-index:3;}
    .rarity{margin-top:6px;font-size:14px;font-weight:600;text-transform:uppercase;position:relative;z-index:3;}

    /* Banners */
    .banner{position:absolute;left:50%;transform:translateX(-50%);padding:12px 16px;border-radius:14px;z-index:7;background:rgba(20,25,40,0.92);border:1px solid rgba(255,255,255,0.14);box-shadow:0 12px 28px rgba(0,0,0,0.55), inset 0 0 26px rgba(110,168,254,0.08);backdrop-filter:blur(6px);font-weight:800;text-align:center;color:var(--text);opacity:0;letter-spacing:0.3px;}
    .banner .title{font-size:16px;}
    .banner .fxline{position:absolute;left:50%;transform:translateX(-50%);width:68%;height:2px;background:linear-gradient(90deg, transparent, var(--fx1), var(--fx2), var(--fx3), transparent);opacity:0.6;}
    .banner .fxline.top{top:6px;} .banner .fxline.bot{bottom:6px;}
    .banner.show{animation:slideIn .35s ease-out, hold 2.6s ease, fadeout 1.2s ease forwards;}
    .banner.luck{top:16px;} .banner.new{top:120px;} .banner.weather{top:16px;} .banner.activate{top:64px;} .banner.announce{top:64px;}
    @keyframes slideIn{from{transform:translate(-50%,-20px);opacity:0}to{transform:translate(-50%,0);opacity:1}}
    @keyframes hold{from{opacity:1}to{opacity:1}}
    @keyframes fadeout{to{opacity:0;filter:blur(3px)}}
    .banner-icon{position:absolute;left:10px;top:50%;transform:translateY(-50%);width:26px;height:26px;border-radius:50%;box-shadow:0 0 18px rgba(255,200,120,0.35), inset 0 0 8px rgba(0,0,0,0.6);}
    .icon-eclipse{background:radial-gradient(circle at 50% 50%, rgba(0,0,0,0.9) 50%, rgba(255,200,120,0.3) 52%, transparent 60%);}
    .icon-storm{background:linear-gradient(180deg, rgba(180,200,255,0.8), rgba(180,200,255,0.1));filter:blur(0.2px);}
    .icon-aurora{background:conic-gradient(from 0deg, rgba(120,200,255,0.6), rgba(180,120,255,0.6), rgba(120,255,200,0.6), rgba(120,200,255,0.6));}
    .icon-tempest{background:radial-gradient(circle, rgba(255,255,255,0.9), rgba(255,255,255,0));}
    .icon-meteor{background:radial-gradient(circle at 50% 50%, rgba(255,180,120,0.9), rgba(255,180,120,0));}
    .icon-sunny{background:radial-gradient(circle at 50% 50%, rgba(255,220,140,0.9), rgba(255,220,140,0));}

    /* Active effects tray */
    .active-effects{position:absolute;bottom:10px;right:10px;z-index:6;font-size:12px;max-width:70%;display:flex;flex-direction:column;align-items:flex-end;gap:6px;}
    .effect-entry{display:flex;gap:8px;align-items:center;padding:6px 10px;border-radius:10px;background:rgba(27,34,50,0.9);border:1px solid #2a3449;box-shadow:0 6px 18px rgba(0,0,0,0.25);font-weight:600;}

    /* Weather backdrops (IN-PLACE, ultra FX) */
    .weather-bg{position:absolute;inset:0;z-index:1;pointer-events:none;opacity:0;animation:weatherFadeIn .7s ease-out forwards;}
    @keyframes weatherFadeIn{from{opacity:0}to{opacity:1}}
    .wb-sunny{background:
      radial-gradient(120% 120% at 50% 10%, rgba(255,215,120,0.22), transparent 60%),
      linear-gradient(180deg, rgba(255,232,170,0.08), rgba(0,0,0,0));
      overflow:hidden;}
    .wb-storm{background:
      radial-gradient(100% 120% at 50% 0%, rgba(25,30,55,0.8), rgba(0,0,0,0.9)),
      linear-gradient(180deg, rgba(25,30,55,0.78), rgba(0,0,0,0.86));}
    .wb-blizzard{background:
      linear-gradient(180deg, rgba(200,230,255,0.36), rgba(0,0,0,0.52));}
    .wb-meteor{background:
      radial-gradient(160% 160% at 20% -20%, rgba(255,120,80,0.22), transparent 60%),
      linear-gradient(180deg, rgba(120,50,30,0.28), rgba(0,0,0,0.6));}
    .wb-aurora{background:
      linear-gradient(120deg,rgba(120,200,255,0.28),rgba(180,120,255,0.28),rgba(120,255,200,0.28));background-size:600% 600%;animation:auroraShift 18s ease infinite;}
    @keyframes auroraShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
    .wb-eclipse{background:
      radial-gradient(circle at 50% 50%,rgba(0,0,0,0.93),rgba(0,0,0,0.78)),
      radial-gradient(circle at 50% 50%,rgba(255,200,120,0.08),transparent 70%);}
    .wb-tempest{background:
      radial-gradient(circle at 50% 50%,rgba(120,80,255,0.25),transparent 70%),
      radial-gradient(circle at 70% 30%,rgba(80,200,255,0.25),transparent 70%);}
    .wb-fog{background:radial-gradient(100% 100% at 50% 50%, rgba(185,195,210,0.16), rgba(0,0,0,0.57));}

    /* Weather particles & FX (UPGRADED) */
    .particles{position:absolute;inset:0;overflow:hidden;filter:blur(0.1px);}
    /* Storm rain */
    .rain-drop{position:absolute;width:2px;height:18px;background:linear-gradient(to bottom,rgba(180,200,255,0.95),rgba(180,200,255,0.25));border-radius:1px;opacity:0.7;}
    @keyframes rainFall{0%{transform:translateY(-100px)}100%{transform:translateY(480px)}}
    /* Storm lightning */
    .bolt{position:absolute;width:2px;height:180px;background:linear-gradient(to bottom, rgba(255,255,255,0.95), rgba(255,255,255,0));filter:blur(0.6px);opacity:0;}
    @keyframes flash{0%{opacity:0}2%{opacity:1}4%{opacity:0}100%{opacity:0}}
    /* Charge cloud at top */
    .storm-charge{position:absolute;left:0;top:-10%;width:100%;height:20%;background:radial-gradient(100% 80% at 50% 50%, rgba(200,220,255,0.18), transparent 60%);filter:blur(6px);animation:chargePulse 5s ease-in-out infinite;}
    @keyframes chargePulse{0%{opacity:.2}50%{opacity:.6}100%{opacity:.2}}

    /* Blizzard snow */
    .snow-flake{position:absolute;width:6px;height:6px;background:white;border-radius:50%;opacity:0.9;box-shadow:0 0 6px rgba(255,255,255,0.5);}
    @keyframes snowFall{0%{transform:translateY(-60px) translateX(0)}50%{transform:translateY(260px) translateX(16px)}100%{transform:translateY(480px) translateX(0)}}
    .snow-swirl{position:absolute;left:0;bottom:-10%;width:120%;height:30%;background:radial-gradient(100% 80% at 50% 50%, rgba(255,255,255,0.12), transparent 60%);filter:blur(10px);animation:swirl 10s linear infinite;}
    @keyframes swirl{0%{transform:translateX(-10%)}50%{transform:translateX(10%)}100%{transform:translateX(-10%)}}

    /* Fog */
    .fog-mist{position:absolute;left:-40%;top:0;width:180%;height:120%;background:radial-gradient(circle,rgba(255,255,255,0.08),transparent 70%);filter:blur(8px);opacity:0.75;animation:fogDrift 24s linear infinite;}
    @keyframes fogDrift{0%{transform:translateX(0)}50%{transform:translateX(10%)}100%{transform:translateX(0)}}

    /* Aurora ribbons */
    .ribbon{position:absolute;width:60%;height:16px;left:20%;top:18%;border-radius:999px;filter:blur(2px);opacity:0.7;animation:ribbonWave 12s ease-in-out infinite;}
    @keyframes ribbonWave{0%{transform:translateY(0) skewX(6deg)}50%{transform:translateY(40px) skewX(-6deg)}100%{transform:translateY(0) skewX(6deg)}}
    .aurora-stars{position:absolute;inset:0;}
    .aurora-star{position:absolute;width:3px;height:3px;border-radius:50%;background:radial-gradient(circle,rgba(255,255,255,0.8),transparent);opacity:0.9;animation:twinkle 2.2s ease-in-out infinite;}

    /* Cosmic Tempest glints */
    .cosmic{position:absolute;width:4px;height:4px;border-radius:50%;background:radial-gradient(circle,rgba(255,255,255,0.9),rgba(255,255,255,0));opacity:0.8;}
    @keyframes drift{0%{transform:translate(0,0)}50%{transform:translate(12px,-16px)}100%{transform:translate(0,0)}}

    /* Eclipse corona */
    .corona{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:240px;height:240px;border-radius:50%;box-shadow:0 0 80px rgba(255,200,120,0.24);animation:pulse 5s ease-in-out infinite;}
    @keyframes pulse{0%{box-shadow:0 0 80px rgba(255,200,120,0.18)}50%{box-shadow:0 0 120px rgba(255,200,120,0.4)}100%{box-shadow:0 0 80px rgba(255,200,120,0.18)}}

    /* Meteors */
    .meteor{position:absolute;width:5px;height:16px;background:linear-gradient(180deg, rgba(255,180,120,0.95), rgba(255,180,120,0.1));border-radius:2px;transform:rotate(35deg);opacity:0.9;box-shadow:0 0 8px rgba(255,180,120,0.5);}
    @keyframes meteorFall{0%{transform:translate(0,-100px) rotate(35deg)}100%{transform:translate(-260px,520px) rotate(35deg)}}

    /* Sunny shimmer */
    .sunbeam{position:absolute;width:4px;height:220px;background:linear-gradient(180deg, rgba(255,230,160,0.8), rgba(255,230,160,0));opacity:.5;filter:blur(0.6px);}
    @keyframes beamRise{0%{transform:translateY(220px)}100%{transform:translateY(-40px)}}

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

    /* Mode selector and Auto-Sell */
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
    .b-exclusive{background:conic-gradient(from 0deg, red, orange, yellow, green, blue, indigo, violet, red);animation:exclusiveSpin 8s linear infinite; color:white;}
    @keyframes exclusiveSpin{0%{filter:hue-rotate(0deg)}100%{filter:hue-rotate(360deg)}}

    /* Roll glow */
    .glow{position:absolute;inset:-40%;border-radius:50%;background:radial-gradient(closest-side,rgba(110,168,254,0.25),transparent 65%);filter:blur(12px);animation:glow 1.1s ease-out forwards;z-index:2;}
    @keyframes glow{0%{opacity:0;transform:scale(0.7)}50%{opacity:1}100%{opacity:0;transform:scale(1.2)}}

    /* Inline cutscene container (inside roll area, not fullscreen) */
    .cutscene-inline{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:90%;height:82%;border-radius:16px;overflow:hidden;z-index:5;pointer-events:none;border:1px solid rgba(255,255,255,0.12);box-shadow:0 16px 40px rgba(0,0,0,0.55), inset 0 0 50px rgba(255,255,255,0.04);}
    .cut-title-inline{position:absolute;top:14px;left:50%;transform:translateX(-50%);font-weight:900;color:#fff;text-shadow:0 6px 18px rgba(0,0,0,0.7);z-index:6;font-size:18px;letter-spacing:0.8px;}
    .cut-sub-inline{position:absolute;top:40px;left:50%;transform:translateX(-50%);color:#cfd6ff;z-index:6;text-shadow:0 6px 18px rgba(0,0,0,0.7);font-size:13px;}

    /* Cinematic primitives */
    .star{position:absolute;border-radius:50%;background:radial-gradient(circle, rgba(255,255,255,0.95), rgba(255,255,255,0));filter:blur(0.2px);}
    @keyframes twinkle{0%,100%{opacity:0.3}50%{opacity:1}}
    .ring{position:absolute;border-radius:50%;border:2px solid rgba(255,255,255,0.25);}
    @keyframes pulseRing{0%{transform:scale(0.6);opacity:0.0}50%{opacity:1}100%{transform:scale(1.4);opacity:0}}
    .beam{position:absolute;background:linear-gradient(180deg,rgba(255,255,255,0.9),rgba(255,255,255,0));filter:blur(1.2px);mix-blend-mode:screen;}
    .shard{position:absolute;width:8px;height:24px;background:linear-gradient(180deg,rgba(150,200,255,0.9),rgba(150,200,255,0));transform-origin:center;filter:blur(0.4px);}

    /* Delete confirmation modal (improved) */
    .confirm-wrap{display:none;position:fixed;inset:0;z-index:80;align-items:center;justify-content:center;background:rgba(0,0,0,0.55);backdrop-filter:blur(2px);}
    .confirm-modal{width:480px;max-width:94vw;background:#0f1320;border:1px solid #2a3449;border-radius:16px;box-shadow:0 18px 50px rgba(0,0,0,0.65);overflow:hidden;}
    .confirm-head{padding:14px 16px;background:#121826;border-bottom:1px solid #2a3449;font-weight:800;}
    .confirm-body{padding:16px;display:grid;gap:10px;}
    .confirm-item{padding:10px;border-radius:10px;background:rgba(27,34,50,0.85);border:1px solid #2a3449;display:flex;gap:10px;align-items:center;}
    .confirm-actions{display:flex;gap:12px;justify-content:flex-end;padding:14px;border-top:1px solid #2a3449;background:#101624;}
    .btn-danger{background:#2a1620;border-color:#64324a;color:#ff9fae;}
    .btn-safe{background:#162a1f;border-color:#2c5a40;color:#9cf2c7;}

    /* Commands panel */
    #commandsPanel{display:none;}
    .cmds{display:grid;gap:16px;}
    .cmd-section{border:1px solid #2a3449;border-radius:10px;padding:12px;background:#101624;}
    .cmd-section h4{margin:0 0 8px;font-size:14px;color:#9aa0ab;}
    .cmd-row{display:grid;grid-template-columns:1fr 1fr;gap:10px;align-items:center;}
    .cmd-row.triple{grid-template-columns:1fr 1fr 1fr;}
    .cmd-row.single{grid-template-columns:1fr;}
    select,input[type=text],input[type=number]{background:#1b2232;border:1px solid #2a3449;border-radius:8px;padding:8px;color:var(--text);}
    .cmd-actions{display:flex;justify-content:flex-end;gap:10px;}
    .cmd-actions button{min-width:120px;}
    .cmd-note{font-size:12px;color:#9aa0ab;}
  </style>
</head>
<body>
  <div class="app">
    <div class="content">
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
          <div class="active-effects" id="activeEffects"></div>
        </div>
        <div class="controls" id="controls">
          <button id="btnRoll">Roll</button>
          <button id="btnAuto" disabled>Auto Roll (locked)</button>
          <button id="btnIndex">Index</button>
          <button id="btnInventory">Inventory</button>
          <button id="btnCommands" style="margin-left:auto;">Commands</button>
        </div>
      </div>

      <div class="panel" id="indexPanel" style="display:none;">
        <h3 style="margin:0 0 10px;display:flex;justify-content:space-between;">
          <span>Index</span><span id="indexCompletion"></span>
        </h3>
        <div class="index-grid" id="indexGrid"></div>
      </div>

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
              <button class="mode-arrow" id="autoSellPrev">‹</button>
              <div class="autosell-value" id="autoSellValue">Off</div>
              <button class="mode-arrow" id="autoSellNext">›</button>
            </div>
          </div>
        </div>
        <ul id="inventoryList" class="inv-list"></ul>
      </div>

      <!-- Commands (robust and working) -->
      <div class="panel" id="commandsPanel" style="display:none;">
        <h3>Commands</h3>
        <div class="cmds">
          <div class="cmd-section">
            <h4>Weights & luck</h4>
            <div class="cmd-row">
              <input type="number" step="0.01" id="cmdLuck" placeholder="Luck + (e.g. 0.5 for +50%)" />
              <select id="cmdLuckDuration">
                <option value="30">30s</option><option value="60">60s</option><option value="120">120s</option><option value="180">180s</option>
              </select>
            </div>
            <div class="cmd-row">
              <select id="cmdBiasRarity"></select>
              <input type="number" step="0.01" id="cmdBiasAmount" placeholder="Bias amount (e.g. 0.2 = +20%)" />
            </div>
          </div>

          <div class="cmd-section">
            <h4>Weather</h4>
            <div class="cmd-row">
              <select id="cmdWeather"></select>
              <select id="cmdWeatherDuration">
                <option value="90">90s</option><option value="150">150s</option><option value="240">240s</option>
              </select>
            </div>
          </div>

          <div class="cmd-section">
            <h4>Force next roll</h4>
            <div class="cmd-row triple">
              <select id="cmdNextRarity"></select>
              <select id="cmdNextName"></select>
              <select id="cmdNextClear"><option value="keep">Keep after use</option><option value="clear">Clear after use</option></select>
            </div>
            <div class="cmd-note">If you pick a name, it must exist in that tier’s Index.</div>
          </div>

          <div class="cmd-section">
            <h4>Data</h4>
            <div class="cmd-row">
              <button id="cmdReset">Reset Entire Data</button>
              <button id="cmdClearEffects">Clear Effects</button>
            </div>
          </div>

          <div class="cmd-section">
            <h4>Give item/effect</h4>
            <div class="cmd-row triple">
              <select id="cmdGiveRarity"></select>
              <select id="cmdGiveItem"></select>
              <select id="cmdGiveAction"><option value="add_inv">Add to Items inventory</option><option value="activate">Activate immediately</option></select>
            </div>
          </div>

          <div class="cmd-actions">
            <button id="cmdApply">Apply</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Delete confirmation modal -->
  <div class="confirm-wrap" id="confirmWrap">
    <div class="confirm-modal">
      <div class="confirm-head" id="confirmTitle">Confirm delete</div>
      <div class="confirm-body">
        <div class="confirm-item">
          <span id="confirmBadge" class="badge">RARITY</span>
          <div>
            <div id="confirmName" style="font-weight:800">Item name</div>
            <div id="confirmDesc" style="font-size:12px;color:#9aa0ab">Are you sure you want to delete this very rare item? This action cannot be undone.</div>
          </div>
        </div>
      </div>
      <div class="confirm-actions">
        <button class="btn-safe" id="confirmCancel">Keep</button>
        <button class="btn-danger" id="confirmDelete">Delete</button>
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
      {key:"exclusive",name:"Exclusive",weight:0,colorClass:"b-exclusive"}
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
      omniversal:["Omniversal Heart","All-Crown","Totality Core","Panreality Halo","Absolute Sigil","Everything Rune","Boundless Star","Cosmos Crown","Axis of All","Prime Totality","Eclipse Gem","Cosmic Gem"],
      exclusive:["Gem Of Gem"]
    };

    /* ---------------- Consumables & Totems ---------------- */
    const ITEM_DROPS={
      worthless:[{ name:"Vial of Luck", rarity:"worthless", type:"luck", amount:0.01, duration:40 },{ name:"Cracked Token of Chance", rarity:"worthless", type:"bias", target:"common", amount:0.05, duration:60 }],
      trash:[{ name:"Spoiled Luck Potion", rarity:"trash", type:"luck", amount:0.05, duration:60 },{ name:"Tarnished Coin", rarity:"trash", type:"bias", target:"uncommon", amount:0.05, duration:60 }],
      common:[
        { name:"Luck Potion", rarity:"common", type:"luck", amount:0.25, duration:90 },
        { name:"Charm of Fortune", rarity:"common", type:"bias", target:"rare", amount:0.07, duration:60 },
        { name:"Storm Totem", rarity:"legendary", type:"totem", weather:"Storm" },
        { name:"Blizzard Totem", rarity:"legendary", type:"totem", weather:"Blizzard" },
        { name:"Sun Totem", rarity:"legendary", type:"totem", weather:"Sunny Radiance" },
        { name:"Fog Totem", rarity:"legendary", type:"totem", weather:"Fog" },
        { name:"Random Event Totem", rarity:"divine", type:"totem_random" }
      ],
      uncommon:[{ name:"Strong Luck Potion", rarity:"uncommon", type:"luck", amount:0.50, duration:150 },{ name:"Sigil of Odds", rarity:"uncommon", type:"bias", target:"epic", amount:0.08, duration:60 }],
      rare:[
        { name:"Greater Luck Potion", rarity:"rare", type:"luck", amount:1.00, duration:60 },
        { name:"Greater Luck Charm", rarity:"rare", type:"bias", target:"legendary", amount:0.10, duration:60 },
        { name:"Meteor Storm Totem", rarity:"divine", type:"totem", weather:"Meteor Storm" },
        { name:"Aurora Veil Totem", rarity:"divine", type:"totem", weather:"Aurora Veil" }
      ],
      epic:[{ name:"Epic Luck Potion", rarity:"epic", type:"luck", amount:1.75, duration:40 },{ name:"Crystal of Providence", rarity:"epic", type:"bias", target:"mythic", amount:0.12, duration:60 }],
      legendary:[
        { name:"Elixir of Destiny", rarity:"legendary", type:"luck", amount:2.50, duration:30 },
        { name:"Relic of Destiny", rarity:"legendary", type:"bias", target:"divine", amount:0.15, duration:60 },
        { name:"Speed Potion", rarity:"legendary", type:"speed", amount:0.25, duration:60 },
        { name:"Eternal Eclipse Totem", rarity:"transcendent", type:"totem", weather:"Eternal Eclipse" },
        { name:"Cosmic Tempest Totem", rarity:"transcendent", type:"totem", weather:"Cosmic Tempest" }
      ],
      mythic:[{ name:"Fatebinder’s Draught", rarity:"mythic", type:"luck", amount:3.75, duration:30 },{ name:"Fate‑Twister’s Seal", rarity:"mythic", type:"bias", target:"celestial", amount:0.18, duration:60 }],
      divine:[{ name:"Elixir of Fortune", rarity:"divine", type:"luck", amount:5.00, duration:20 },{ name:"Blessed Talisman", rarity:"divine", type:"bias", target:"transcendent", amount:0.22, duration:60 },{ name:"Elixir of Quickening", rarity:"divine", type:"speed", amount:0.50, duration:45 }],
      celestial:[{ name:"Starlight Elixir", rarity:"celestial", type:"luck", amount:7.50, duration:20 },{ name:"Starlight Sigil", rarity:"celestial", type:"bias", target:"eternal", amount:0.26, duration:60 }],
      transcendent:[{ name:"Paradox Brew", rarity:"transcendent", type:"luck", amount:10.00, duration:15 },{ name:"Paradox Shard", rarity:"transcendent", type:"bias", target:"omniversal", amount:0.30, duration:60 },{ name:"Godly Potion of Haste", rarity:"transcendent", type:"speed", amount:1.00, duration:30 }],
      eternal:[{ name:"Godly Potion", rarity:"eternal", type:"luck", amount:15.00, duration:15 },{ name:"Godly Relic", rarity:"eternal", type:"bias", target:"omniversal", amount:0.50, duration:60 }],
      omniversal:[
        { name:"Origin Draught", rarity:"omniversal", type:"luck", amount:25.00, duration:20 },
        { name:"Origin Draught of Speed", rarity:"omniversal", type:"speed", amount:2.50, duration:25 },
        { name:"Origin Crystal", rarity:"omniversal", type:"guarantee", amount:1, duration:0 }
      ],
      exclusive:[]
    };

    const LUCK_TARGET_KEYS=["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal","exclusive"];

    /* ---------------- Weather definitions ---------------- */
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
      mode:"sol_rng_mode",
      cmd:"sol_rng_commands",
      guarantees:"sol_rng_guarantees"
    };

    function buildInitialUnlocks(def){ const o={}; for(const k in def){ o[k]={}; def[k].forEach(n=>o[k][n]=false); } return o; }
    function reviveUnlocks(u){ const f=buildInitialUnlocks(INDEX_ITEMS); for(const k in f){ for(const n of Object.keys(f[k])){ f[k][n]=u&&u[k]&&typeof u[k][n]==="boolean"?u[k][n]:false; } } return f; }

    /* ---------------- State ---------------- */
    const ROLLED_MAX=10, ITEMS_MAX=50, BASE_AUTO_INTERVAL=140, MAX_EFFECT_SECONDS=250;
    const state={
      rolls:0, unlocks:buildInitialUnlocks(INDEX_ITEMS), auto:false, autoInterval:null,
      inventoryRolled:[], inventoryItems:[], autoSellRolled:"off", autoSellItems:"off",
      fullAnnouncedRolled:false, fullAnnouncedItems:false,
      activeEffects:{ luck:0, speed:0, bias:{}, weatherBiasItem:null },
      effectInstances:[], guarantees:[],
      mode:"Rolled",
      commands:{ nextRoll:null }, // {rarity:'', name:'', applied:false, clearAfter:true}
      autoWasOnBeforeCut:false, cutscenePlaying:false
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
      const cmdRaw=localStorage.getItem(STORAGE_KEYS.cmd);
      const guarRaw=localStorage.getItem(STORAGE_KEYS.guarantees);

      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      const inv = invRaw?JSON.parse(invRaw):{rolled:[],items:[]};
      state.inventoryRolled = inv.rolled || [];
      state.inventoryItems = inv.items || [];
      state.autoSellRolled=autoSellRolledRaw||"off";
      state.autoSellItems=autoSellItemsRaw||"off";
      const parsedEffects = effInstRaw ? JSON.parse(effInstRaw) : [];
      const now=Date.now();
      state.effectInstances = parsedEffects.filter(e=>e.expiresAt>now);
      state.guarantees = guarRaw ? JSON.parse(guarRaw) : [];
      deriveEffectsTotals();
      state.mode = modeRaw || "Rolled";
      state.commands = cmdRaw ? JSON.parse(cmdRaw) : { nextRoll:null };
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
      localStorage.setItem(STORAGE_KEYS.cmd,JSON.stringify(state.commands));
      localStorage.setItem(STORAGE_KEYS.guarantees,JSON.stringify(state.guarantees));
    }

    /* ---------------- Utils ---------------- */
    function clone(o){ return JSON.parse(JSON.stringify(o)); }
    function sumWeights(a){ return a.reduce((s,r)=>s+r.weight,0); }
    function milestoneLuckMultiplier(n){ if(n>0 && n%200===0) return 10; if(n>0 && n%50===0) return 2; return 1; }

    function spawnBanner(text,type,colorClass,withIcon){
      const rollArea=document.getElementById("rollArea");
      const div=document.createElement("div");
      div.className=`banner ${type} show ${colorClass?colorClass:''}`;
      const title=document.createElement("div"); title.className="title"; title.textContent=text;
      const fxTop=document.createElement("div"); fxTop.className="fxline top";
      const fxBot=document.createElement("div"); fxBot.className="fxline bot";
      if(withIcon){ const icon=document.createElement("div"); icon.className="banner-icon " + withIcon; div.appendChild(icon); }
      div.appendChild(fxTop); div.appendChild(title); div.appendChild(fxBot);
      div.addEventListener("animationend",()=>div.remove());
      rollArea.appendChild(div);
    }

    function shouldAutoSellRolled(tierKey){ const order=TIERS.map(t=>t.key); const thr=state.autoSellRolled; if(thr==="off") return false; if(tierKey==="exclusive") return false; return order.indexOf(tierKey) <= order.indexOf(thr); }
    function shouldAutoSellItems(tierKey){ const order=TIERS.map(t=>t.key); const thr=state.autoSellItems; if(thr==="off") return false; if(tierKey==="exclusive") return false; return order.indexOf(tierKey) <= order.indexOf(thr); }

    function applyWeightModifiers(baseTiers,mMult){
      const tiers=clone(baseTiers);
      if(mMult>1){ for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= mMult; } }
      if(state.activeEffects.luck>0){ const mult=1+state.activeEffects.luck; for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight *= mult; } }
      for(const key in state.activeEffects.bias){ const amt=state.activeEffects.bias[key]; const t=tiers.find(x=>x.key===key); if(t){ t.weight *= (1+amt); } }
      return tiers;
    }
    function toChances(tiers){ const total=sumWeights(tiers); return tiers.map(t=>({ ...t, chance: total>0 ? t.weight/total : 0 })); }
    function pickTier(chances){ const r=Math.random(); let acc=0; for(let i=0;i<chances.length;i++){ acc+=chances[i].chance; if(r<=acc) return chances[i]; } return chances[chances.length-1]; }
    function pickIndexItem(tierKey){ const list=INDEX_ITEMS[tierKey]||[]; if(!list.length) return null; return list[Math.floor(Math.random()*list.length)]; }

    function buildItemTierWeightsFromIndex(baseTiers){
      const order=baseTiers.map(t=>t.key);
      return baseTiers.map(t=>{ const idx=order.indexOf(t.key); const penalKey=order[Math.min(idx+2,order.length-1)]; const penalTier=baseTiers.find(x=>x.key===penalKey); const weight=(penalTier?.weight || t.weight)/8000; return { ...t, weight }; });
    }
    function pickConsumableFromTier(tierKey){ const list=ITEM_DROPS[tierKey]||[]; if(!list.length) return null; return list[Math.floor(Math.random()*list.length)]; }

    /* ---------------- Weather visual (ULTRA FX) ---------------- */
    function renderWeatherBackdrop(){
      const area=document.getElementById("rollArea");
      const old=area.querySelector(".weather-bg"); if(old) old.remove();
      const now=Date.now();
      const weather=state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      if(!weather) return;
      const map={"Sunny Radiance":"wb-sunny","Storm":"wb-storm","Blizzard":"wb-blizzard","Meteor Storm":"wb-meteor","Aurora Veil":"wb-aurora","Eternal Eclipse":"wb-eclipse","Cosmic Tempest":"wb-tempest","Fog":"wb-fog"};
      const cls=map[weather.name] || "wb-sunny";
      const bg=document.createElement("div"); bg.className="weather-bg "+cls;
      const particles=document.createElement("div"); particles.className="particles";

      // Sunny: beams and drifting dust motes
      if(cls==="wb-sunny"){
        for(let i=0;i<10;i++){
          const b=document.createElement("div"); b.className="sunbeam";
          b.style.left=(5+i*9)+"%"; b.style.top=(20+Math.random()*20)+"%";
          b.style.animation=`beamRise ${4+Math.random()*2}s ease-in-out infinite`;
          b.style.opacity=0.35+Math.random()*0.4;
          particles.appendChild(b);
        }
        for(let i=0;i<40;i++){
          const mote=document.createElement("span"); mote.className="cosmic";
          mote.style.left=Math.floor(Math.random()*100)+"%";
          mote.style.top=Math.floor(Math.random()*100)+"%";
          mote.style.width=mote.style.height=(Math.random()*2+1)+"px";
          mote.style.animation=`drift ${6+Math.random()*6}s ease-in-out infinite`;
          mote.style.opacity=0.5;
          particles.appendChild(mote);
        }
      }

      // Storm: rain + timed lightning + charge cloud
      if(cls==="wb-storm"){
        const charge=document.createElement("div"); charge.className="storm-charge"; particles.appendChild(charge);
        for(let i=0;i<160;i++){
          const p=document.createElement("span"); p.className="rain-drop";
          p.style.left=Math.floor(Math.random()*100)+"%";
          p.style.top=(-100-Math.random()*160)+"px";
          p.style.animation=`rainFall ${0.7+Math.random()*0.6}s linear infinite`;
          p.style.opacity=0.55+Math.random()*0.35;
          particles.appendChild(p);
        }
        // Lightning generation timer
        function spawnLightning(){
          const bolt=document.createElement("div"); bolt.className="bolt";
          bolt.style.left=(10+Math.random()*80)+"%"; bolt.style.top=(-20+Math.random()*20)+"px";
          bolt.style.animation="flash 1.8s ease-in-out 1";
          particles.appendChild(bolt);
          setTimeout(()=>bolt.remove(), 1800);
        }
        const lightningInterval=setInterval(()=>{ if(Math.random()<0.7) spawnLightning(); }, 2000);
        bg.addEventListener("DOMNodeRemoved", ()=>clearInterval(lightningInterval));
      }

      // Blizzard: heavy snow + swirling ground mist
      if(cls==="wb-blizzard"){
        for(let i=0;i<120;i++){
          const s=document.createElement("span"); s.className="snow-flake";
          s.style.left=Math.floor(Math.random()*100)+"%"; s.style.top=(-80-Math.random()*180)+"px";
          s.style.animation=`snowFall ${3.2+Math.random()*2.6}s linear infinite`;
          s.style.opacity=0.6+Math.random()*0.4;
          particles.appendChild(s);
        }
        const swirl=document.createElement("div"); swirl.className="snow-swirl"; particles.appendChild(swirl);
      }

      // Fog: layered drifting mists
      if(cls==="wb-fog"){
        const fog1=document.createElement("div"); fog1.className="fog-mist"; fog1.style.opacity="0.7";
        const fog2=document.createElement("div"); fog2.className="fog-mist"; fog2.style.opacity="0.45"; fog2.style.animationDuration="32s";
        const fog3=document.createElement("div"); fog3.className="fog-mist"; fog3.style.opacity="0.3"; fog3.style.animationDuration="40s";
        particles.appendChild(fog1); particles.appendChild(fog2); particles.appendChild(fog3);
      }

      // Aurora: multiple ribbons + starfield shimmer
      if(cls==="wb-aurora"){
        const r1=document.createElement("div"); r1.className="ribbon"; r1.style.background="linear-gradient(90deg, rgba(160,255,220,0.7), rgba(150,120,255,0.7))";
        const r2=document.createElement("div"); r2.className="ribbon"; r2.style.top="30%"; r2.style.animationDuration="16s"; r2.style.background="linear-gradient(90deg, rgba(120,200,255,0.7), rgba(120,255,200,0.7))";
        const r3=document.createElement("div"); r3.className="ribbon"; r3.style.top="46%"; r3.style.animationDuration="20s"; r3.style.background="linear-gradient(90deg, rgba(180,140,255,0.7), rgba(120,255,220,0.7))";
        particles.appendChild(r1); particles.appendChild(r2); particles.appendChild(r3);
        const stars=document.createElement("div"); stars.className="aurora-stars";
        for(let i=0;i<60;i++){
          const st=document.createElement("div"); st.className="aurora-star";
          st.style.left=Math.random()*100+"%"; st.style.top=Math.random()*100+"%";
          st.style.animationDuration=(1.8+Math.random()*1.6)+"s";
          stars.appendChild(st);
        }
        particles.appendChild(stars);
      }

      // Eclipse: corona glow
      if(cls==="wb-eclipse"){ const corona=document.createElement("div"); corona.className="corona"; particles.appendChild(corona); }

      // Tempest: dense glints
      if(cls==="wb-tempest"){
        for(let i=0;i<42;i++){
          const c=document.createElement("span"); c.className="cosmic";
          c.style.left=Math.floor(Math.random()*100)+"%";
          c.style.top=Math.floor(Math.random()*100)+"%";
          c.style.width=c.style.height=(Math.random()*3+2)+"px";
          c.style.animation=`drift ${8+Math.random()*6}s ease-in-out infinite`;
          particles.appendChild(c);
        }
      }

      // Meteor storm: glints + timed meteor showers
      if(cls==="wb-meteor"){
        for(let i=0;i<24;i++){
          const c=document.createElement("span"); c.className="cosmic";
          c.style.left=Math.floor(Math.random()*100)+"%";
          c.style.top=Math.floor(Math.random()*40)+"%";
          c.style.width=c.style.height=(Math.random()*3+2)+"px";
          c.style.animation=`drift ${8+Math.random()*6}s ease-in-out infinite`;
          particles.appendChild(c);
        }
        function spawnMeteorBurst(){
          const count = 10 + Math.floor(Math.random()*8);
          for(let i=0;i<count;i++){
            const m=document.createElement("div"); m.className="meteor";
            const startX = 20 + Math.random()*80;
            m.style.left = startX + "%";
            m.style.top = (-100 - Math.random()*80) + "px";
            m.style.animation = `meteorFall ${1.2+Math.random()*0.8}s linear 1`;
            particles.appendChild(m);
            setTimeout(()=>m.remove(), 2200);
          }
        }
        spawnMeteorBurst();
        const meteorInterval = setInterval(spawnMeteorBurst, 3000);
        bg.addEventListener("DOMNodeRemoved", ()=>clearInterval(meteorInterval));
      }

      bg.appendChild(particles); area.appendChild(bg);
    }

    /* ---------------- Effects ---------------- */
    function deriveEffectsTotals(){
      const totals={ luck:0, speed:0, bias:{}, weatherBiasItem:null };
      const now=Date.now();
      for(const e of state.effectInstances){
        if(e.expiresAt<=now) continue;
        if(e.type==="luck") totals.luck+=e.amount;
        else if(e.type==="speed") totals.speed+=e.amount;
        else if(e.type==="bias"){ totals.bias[e.target]=(totals.bias[e.target]||0)+e.amount; }
        else if(e.type==="weather" && e.meta){ totals.weatherBiasItem=e.meta.biasItem||null; totals.luck += (e.meta.luck||0); }
      }
      state.activeEffects=totals;
    }
    function addEffect(effect){
      if(effect.type==="guarantee" && effect.name==="Origin Crystal"){
        state.guarantees.push({ name:effect.name, type:"guarantee" }); saveState();
        spawnBanner(`Stored: ${effect.name}`,"activate","b-omniversal"); renderActiveEffects(); return;
      }
      const now=Date.now(), durMs=(effect.duration||0)*1000, capMs=MAX_EFFECT_SECONDS*1000;
      const keyMatch=e=>e.name===effect.name && e.type===effect.type && (e.target||null)===(effect.target||null);
      const existingIdx=state.effectInstances.findIndex(keyMatch);
      if(existingIdx>=0){
        const ex=state.effectInstances[existingIdx]; const remaining=Math.max(0,ex.expiresAt-now); const extended=Math.min(capMs, remaining+durMs);
        ex.expiresAt = now + extended;
        if(effect.type!=="weather"){ ex.amount += (effect.amount||0); } else { ex.meta = effect.meta || ex.meta; }
        ex.rarityKey = effect.rarity || ex.rarityKey;
      } else {
        state.effectInstances.push({ name:effect.name, type:effect.type, amount:effect.amount||0, target:effect.target, expiresAt: durMs ? now+Math.min(capMs,durMs) : now+durMs, rarityKey:effect.rarity||"common", weather: effect.type==="weather", meta: effect.meta||null });
      }
      deriveEffectsTotals(); saveState(); renderActiveEffects(); renderWeatherBackdrop(); updateAutoInterval();
    }
    setInterval(()=>{ const now=Date.now(); const before=state.effectInstances.length; state.effectInstances=state.effectInstances.filter(e=>e.expiresAt>now); if(state.effectInstances.length!==before){ deriveEffectsTotals(); saveState(); renderWeatherBackdrop(); updateAutoInterval(); } renderActiveEffects(); },1000);

    function formatSecondsLeft(ms){ const s=Math.max(0,Math.ceil(ms/1000)); return `${s}s`; }
    function renderActiveEffects(){
      const el=document.getElementById("activeEffects"); el.innerHTML="";
      const now=Date.now(); const others=state.effectInstances.filter(e=>e.expiresAt>now && !e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      const weathers=state.effectInstances.filter(e=>e.expiresAt>now && e.weather).sort((a,b)=>a.expiresAt-b.expiresAt);
      [...others,...weathers].forEach(e=>{
        const left=formatSecondsLeft(e.expiresAt-now); const colorClass=TIERS.find(t=>t.key===e.rarityKey)?.colorClass || "";
        const div=document.createElement("div"); div.className=`effect-entry ${colorClass}`; div.textContent=`${e.name}: ${left}`; el.appendChild(div);
      });
      state.guarantees.forEach(g=>{ const div=document.createElement("div"); div.className=`effect-entry ${"b-omniversal"}`; div.textContent=`${g.name}: ready`; el.appendChild(div); });
    }
    function colorClassForWeather(name){ const w=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].find(x=>x.name===name); return w?.colorClass || ""; }
    function showGlow(){ const rollArea=document.getElementById("rollArea"); const g=document.createElement("div"); g.className="glow"; rollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Mode & Auto-Sell ---------------- */
    const autoSellOptions=["off","worthless","trash","common","uncommon","rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal","exclusive"];
    function setAutoSell(value){ if(state.mode==="Items") state.autoSellItems=value; else state.autoSellRolled=value; saveState(); renderButtonsState(); }
    const modes=["Rolled","Items"];
    function setMode(value){ state.mode=value; saveState(); renderButtonsState(); renderInventory(); }
    function cycle(list,current,dir){ const idx=list.indexOf(current); if(dir<0) return list[idx<=0?list.length-1:idx-1]; return list[idx>=list.length-1?0:idx+1]; }

    /* ---------------- Auto clicker ---------------- */
    function updateAutoInterval(){
      if(state.autoInterval){ clearInterval(state.autoInterval); state.autoInterval=null; }
      if(state.auto){
        const mult=1+(state.activeEffects.speed||0);
        const interval=Math.max(60,Math.round(BASE_AUTO_INTERVAL/mult));
        state.autoInterval=setInterval(()=>{
          const btn=document.getElementById("btnAuto");
          if(btn.disabled || state.cutscenePlaying){ toggleAuto(false); return; }
          rollOnce();
        },interval);
      }
    }
    function toggleAuto(forceState){
      const btn=document.getElementById("btnAuto"); if(btn.disabled && typeof forceState!=="boolean") return;
      const nextState = typeof forceState==="boolean" ? forceState : !state.auto;
      if(nextState){ state.auto=true; btn.textContent="Auto Roll: On"; updateAutoInterval(); }
      else { state.auto=false; clearInterval(state.autoInterval); state.autoInterval=null; btn.textContent="Auto Roll: Off"; }
      saveState();
    }

    /* ---------------- Weather schedule ---------------- */
    function triggerRandomWeather(){
      const r=Math.random(); let pool=WEATHERS.normal;
      if(r>=0.70 && r<0.95) pool=WEATHERS.rare; else if(r>=0.95) pool=WEATHERS.super;
      const w=pool[Math.floor(Math.random()*pool.length)];
      const dur=100+Math.floor(Math.random()*201);
      const meta={ luck:(w.effects?.luck)||0, biasItem:(w.effects?.biasItem)||null };
      addEffect({ name:w.name, type:"weather", duration:dur, rarity: classToTierKey(w.colorClass), meta });
      const icon = w.name==="Eternal Eclipse" ? "icon-eclipse" : w.name==="Storm" ? "icon-storm" : w.name==="Aurora Veil" ? "icon-aurora" : w.name==="Cosmic Tempest" ? "icon-tempest" : w.name==="Meteor Storm" ? "icon-meteor" : w.name==="Sunny Radiance" ? "icon-sunny" : "";
      spawnBanner(`${w.name} started`,"weather",w.colorClass,icon);
    }
    function scheduleNextWeather(){ const delayMs=(240+Math.random()*480)*1000; setTimeout(()=>{ triggerRandomWeather(); scheduleNextWeather(); },delayMs); }
    function classToTierKey(colorClass){ const t=TIERS.find(x=>x.colorClass===colorClass); return t?t.key:"common"; }

    /* ---------------- Inline Cinematic cutscenes per REAL Index item ---------------- */
    function variantForName(name, max=12){
      let h=0; for(let i=0;i<name.length;i++){ h=(h*31 + name.charCodeAt(i))>>>0; }
      return h % max;
    }

    function playInlineCutscene(tierKey,itemName){
      const rollArea=document.getElementById("rollArea");
      // pause auto-roll while cutscene plays
      state.autoWasOnBeforeCut = state.auto;
      state.cutscenePlaying = true;
      if(state.auto) toggleAuto(false);

      // container inside roll area
      const container=document.createElement("div");
      container.className="cutscene-inline";
      container.style.background = ({
        divine:"radial-gradient(circle at 50% 50%, rgba(255,215,120,0.18), rgba(0,0,0,0.92))",
        celestial:"linear-gradient(120deg, rgba(120,255,255,0.18), rgba(0,0,0,0.92))",
        transcendent:"linear-gradient(120deg, rgba(160,140,255,0.22), rgba(0,0,0,0.92))",
        eternal:"radial-gradient(circle at 50% 50%, rgba(140,255,200,0.18), rgba(0,0,0,0.92))",
        omniversal:"conic-gradient(from 0deg, rgba(255,160,220,0.18), rgba(120,255,220,0.18), rgba(180,120,255,0.18), rgba(255,160,220,0.18))"
      }[tierKey] || "rgba(0,0,0,0.85)");
      rollArea.appendChild(container);

      const title=document.createElement("div"); title.className="cut-title-inline"; title.textContent=itemName;
      const sub=document.createElement("div"); sub.className="cut-sub-inline"; sub.textContent=tierKey.toUpperCase();
      container.appendChild(title); container.appendChild(sub);

      // stage
      const stage=document.createElement("div");
      stage.style.position="absolute"; stage.style.inset="0"; stage.style.pointerEvents="none";
      container.appendChild(stage);

      // build distinct sequence
      runProceduralCutscene(itemName, tierKey, stage);

      // end timing & cleanup
      const end=()=>{
        container.remove();
        state.cutscenePlaying=false;
        if(state.autoWasOnBeforeCut) toggleAuto(true);
      };
      setTimeout(end, 7200);
    }

    function runProceduralCutscene(name, tierKey, stage){
      const v = variantForName(name, 12);
      // enriched intro starfield
      for(let i=0;i<140;i++){
        const s=document.createElement("div");
        s.className="star";
        s.style.left=Math.random()*100+"%";
        s.style.top=Math.random()*100+"%";
        s.style.width=s.style.height=(Math.random()*2+1)+"px";
        s.style.animation=`twinkle ${1.6+Math.random()*1.6}s ease-in-out infinite`;
        stage.appendChild(s);
      }
      // choose style family with more drama
      const styles = [
        styleGalacticSpiral, styleCrownRings, styleAuroraWeave, styleShardBurst,
        styleBeamConvergence, styleVortexCollapse, styleSigilEngrave, styleGridAxis,
        stylePrismWave, styleHeartPulse, styleConicBloom, styleHaloShatter
      ];
      styles[v](stage,tierKey,name);

      // dramatic BOOM
      setTimeout(()=>boom(stage, tierKey), 4700);
    }

    function boom(stage, tierKey){
      const tint={
        divine:"rgba(255,215,120,0.9)",
        celestial:"rgba(120,255,255,0.9)",
        transcendent:"rgba(160,140,255,0.9)",
        eternal:"rgba(140,255,200,0.9)",
        omniversal:"rgba(255,160,220,0.9)"
      }[tierKey] || "rgba(255,255,255,0.9)";
      for(let i=0;i<16;i++){
        const ring=document.createElement("div");
        ring.className="ring";
        ring.style.left="50%"; ring.style.top="50%";
        ring.style.borderColor=tint;
        ring.style.width=80+i*36+"px"; ring.style.height=80+i*36+"px";
        ring.style.animation=`pulseRing ${1.4+i*0.1}s ease-out forwards`;
        stage.appendChild(ring);
      }
      for(let i=0;i<54;i++){
        const shard=document.createElement("div");
        shard.className="shard";
        shard.style.left="50%"; shard.style.top="50%";
        shard.style.transform=`translate(-50%,-50%) rotate(${Math.random()*360}deg)`;
        shard.style.animation="explodeShard 1.7s ease-out forwards";
        stage.appendChild(shard);
      }
      injectKeyframesOnce("explodeShard","0%{transform:translate(-50%,-50%) scale(0.35) rotate(0);opacity:.95}100%{transform:translate(-50%,-50%) scale(2.6) rotate(280deg);opacity:.0}");
    }

    // Distinct style families (expanded)
    function styleGalacticSpiral(stage,tier,name){
      for(let i=0;i<20;i++){
        const r=document.createElement("div"); r.className="ring";
        r.style.left="50%"; r.style.top="50%";
        r.style.width=40+i*26+"px"; r.style.height=40+i*26+"px";
        r.style.animation=`spiral ${1.0+i*0.12}s ease-in-out infinite`;
        stage.appendChild(r);
      }
      injectKeyframesOnce("spiral","0%{transform:translate(-50%,-50%) rotate(0) scale(0.8);opacity:.5}50%{opacity:1}100%{transform:translate(-50%,-50%) rotate(200deg) scale(1.4);opacity:.2}");
    }
    function styleCrownRings(stage,tier,name){
      for(let i=0;i<8;i++){
        const ring=document.createElement("div"); ring.className="ring";
        ring.style.left="50%"; ring.style.top="50%"; ring.style.borderColor="rgba(255,200,120,"+(0.35-0.03*i)+")";
        ring.style.width=120+i*38+"px"; ring.style.height=120+i*38+"px";
        ring.style.animation=`crown ${2+i*0.2}s ease-out infinite`; stage.appendChild(ring);
      }
      injectKeyframesOnce("crown","0%{transform:translate(-50%,-50%) scale(0.6);opacity:.3}50%{opacity:1}100%{transform:translate(-50%,-50%) scale(1.7);opacity:.0}");
    }
    function styleAuroraWeave(stage,tier,name){
      for(let i=0;i<5;i++){
        const ribbon=document.createElement("div"); ribbon.className="ribbon";
        ribbon.style.left="10%"; ribbon.style.top=(12+i*12)+"%";
        ribbon.style.animationDuration = 9+i*2 + "s";
        ribbon.style.background="linear-gradient(90deg, rgba(120,255,255,0.7), rgba(255,160,220,0.7))";
        stage.appendChild(ribbon);
      }
      const core=document.createElement("div");
      core.className="corona";
      stage.appendChild(core);
    }
    function styleShardBurst(stage,tier,name){
      for(let i=0;i<160;i++){
        const s=document.createElement("div"); s.className="shard";
        s.style.left=(45+Math.random()*10)+"%"; s.style.top=(45+Math.random()*10)+"%";
        s.style.transform=`rotate(${Math.random()*360}deg)`;
        s.style.animation="shardDance 2.2s ease-in-out infinite";
        stage.appendChild(s);
      }
      injectKeyframesOnce("shardDance","0%{transform:rotate(0) translateY(0);opacity:.25}50%{transform:rotate(180deg) translateY(10px);opacity:.95}100%{transform:rotate(360deg) translateY(0);opacity:.25}");
    }
    function styleBeamConvergence(stage,tier,name){
      for(let i=0;i<10;i++){
        const b=document.createElement("div"); b.className="beam";
        b.style.width="4px"; b.style.height="320px";
        b.style.left=(10+i*9.5)+"%"; b.style.top="0%";
        b.style.animation=`beamDown ${1.6+i*0.1}s linear infinite`;
        stage.appendChild(b);
      }
      injectKeyframesOnce("beamDown","0%{transform:translateY(-160px);opacity:.0}50%{opacity:1}100%{transform:translateY(280px);opacity:.0}");
    }
    function styleVortexCollapse(stage,tier,name){
      for(let i=0;i<22;i++){
        const r=document.createElement("div"); r.className="ring";
        r.style.left="50%"; r.style.top="50%";
        r.style.width=60+i*22+"px"; r.style.height=60+i*22+"px";
        r.style.animation=`vortex ${1.0+i*0.12}s ease-in-out infinite`;
        stage.appendChild(r);
      }
      injectKeyframesOnce("vortex","0%{transform:translate(-50%,-50%) scale(1) rotate(0);opacity:.5}50%{transform:translate(-50%,-50%) scale(0.6) rotate(90deg);opacity:1}100%{transform:translate(-50%,-50%) scale(1.3) rotate(200deg);opacity:.2}");
    }
    function styleSigilEngrave(stage,tier,name){
      const sig=document.createElement("div"); sig.className="ring";
      sig.style.left="50%"; sig.style.top="50%"; sig.style.width="300px"; sig.style.height="300px";
      sig.style.border="4px double rgba(255,255,255,0.4)";
      sig.style.animation="sigPulse 3s ease-in-out infinite";
      stage.appendChild(sig);
      injectKeyframesOnce("sigPulse","0%{transform:translate(-50%,-50%) scale(1);opacity:.65}50%{transform:translate(-50%,-50%) scale(1.12);opacity:1}100%{transform:translate(-50%,-50%) scale(1);opacity:.65}");
    }
    function styleGridAxis(stage,tier,name){
      for(let i=0;i<16;i++){
        const line=document.createElement("div");
        line.style.position="absolute"; line.style.left=i*6+"%"; line.style.top="0";
        line.style.width="1px"; line.style.height="100%"; line.style.background="rgba(255,255,255,0.18)";
        line.style.animation="axisSpin 6s linear infinite";
        stage.appendChild(line);
      }
      injectKeyframesOnce("axisSpin","0%{transform:rotate(0)}100%{transform:rotate(360deg)}");
    }
    function stylePrismWave(stage,tier,name){
      for(let i=0;i<8;i++){
        const cover=document.createElement("div");
        cover.style.position="absolute"; cover.style.left="0"; cover.style.top="0"; cover.style.right="0"; cover.style.bottom="0";
        cover.style.background=`linear-gradient(120deg, rgba(255,160,220,${0.06+i*0.06}), rgba(120,255,220,${0.06+i*0.06}), rgba(180,120,255,${0.06+i*0.06}))`;
        cover.style.filter="blur(6px)";
        cover.style.animation=`sweep ${2.6+i*0.2}s ease-in-out infinite`;
        stage.appendChild(cover);
      }
      injectKeyframesOnce("sweep","0%{transform:translateX(-6%)}50%{transform:translateX(6%)}100%{transform:translateX(-6%)}");
    }
    function styleHeartPulse(stage,tier,name){
      const core=document.createElement("div");
      core.style.position="absolute"; core.style.left="50%"; core.style.top="50%";
      core.style.transform="translate(-50%,-50%)";
      core.style.width="140px"; core.style.height="120px";
      core.style.background="radial-gradient(circle at 30% 30%, rgba(255,160,220,0.9), transparent 60%)";
      core.style.clipPath="polygon(50% 5%, 61% 16%, 73% 28%, 80% 42%, 80% 60%, 66% 78%, 50% 90%, 34% 78%, 20% 60%, 20% 42%, 27% 28%, 39% 16%)";
      core.style.boxShadow="0 0 60px rgba(255,160,220,0.5)";
      core.style.animation="heartPulse 2.4s ease-in-out infinite";
      stage.appendChild(core);
      injectKeyframesOnce("heartPulse","0%{transform:translate(-50%,-50%) scale(1)}50%{transform:translate(-50%,-50%) scale(1.18)}100%{transform:translate(-50%,-50%) scale(1)}");
    }
    function styleConicBloom(stage,tier,name){
      for(let i=0;i<5;i++){
        const cover=document.createElement("div");
        cover.style.position="absolute"; cover.style.left="0"; cover.style.top="0"; cover.style.right="0"; cover.style.bottom="0";
        cover.style.background="conic-gradient(from 0deg, rgba(255,160,220,0.12), rgba(120,255,220,0.12), rgba(180,120,255,0.12), rgba(255,160,220,0.12))";
        cover.style.animation=`spinConic ${6+i*1.2}s linear infinite`;
        stage.appendChild(cover);
      }
      injectKeyframesOnce("spinConic","0%{transform:rotate(0)}100%{transform:rotate(360deg)}");
    }
    function styleHaloShatter(stage,tier,name){
      for(let i=0;i<12;i++){
        const r=document.createElement("div"); r.className="ring";
        r.style.left="50%"; r.style.top="50%"; r.style.borderColor="rgba(255,255,255,"+(0.3-0.02*i)+")";
        r.style.width=80+i*30+"px"; r.style.height=80+i*30+"px";
        r.style.animation=`haloPulse ${1.6+i*0.12}s ease-in-out infinite`;
        stage.appendChild(r);
      }
      injectKeyframesOnce("haloPulse","0%{transform:translate(-50%,-50%) scale(0.7);opacity:.4}50%{transform:translate(-50%,-50%) scale(1.05);opacity:1}100%{transform:translate(-50%,-50%) scale(1.4);opacity:.0}");
    }

    // Ensure keyframes only injected once
    const injectedKeyframes = new Set();
    function injectKeyframesOnce(name, body){
      if(injectedKeyframes.has(name)) return;
      injectedKeyframes.add(name);
      const style=document.createElement("style");
      style.textContent=`@keyframes ${name}{${body}}`;
      document.head.appendChild(style);
    }

    /* ---------------- Rolling ---------------- */
    function weatherCategoryForName(name){
      if(WEATHERS.normal.find(w=>w.name===name)) return "normal";
      if(WEATHERS.rare.find(w=>w.name===name)) return "rare";
      if(WEATHERS.super.find(w=>w.name===name)) return "super";
      return "normal";
    }
    function processIndexItem(tierKey,tierName,itemName,milestone){
      const toKeep=!shouldAutoSellRolled(tierKey);
      if(toKeep){
        if(state.inventoryRolled.length<ROLLED_MAX){
          state.inventoryRolled.push({ type:"index", tier:tierKey, tierName, name:itemName, roll:state.rolls, milestone });
        } else { if(!state.fullAnnouncedRolled){ spawnBanner(`Rolled inventory is full ${ROLLED_MAX}/${ROLLED_MAX}`,"announce"); state.fullAnnouncedRolled=true; } }
      }
    }
    function markNew(tierKey,itemName){ if(!state.unlocks[tierKey][itemName]){ state.unlocks[tierKey][itemName]=true; return true; } return false; }

    // Origin Crystal guarantee: REAL Index item from Eternal+ (Eternal or Omniversal)
    function consumeGuaranteeIfAnyAndPick(){
      if(!state.guarantees.length) return null;
      const idx=state.guarantees.findIndex(g=>g.name==="Origin Crystal");
      if(idx>=0){
        state.guarantees.splice(idx,1); saveState();
        spawnBanner(`Guarantee consumed: Origin Crystal`,"announce","b-omniversal");
        const highKeys=["eternal","omniversal"];
        const tierKey = highKeys[Math.random()<0.65 ? 0 : 1]; // Eternal a bit more likely
        const items=INDEX_ITEMS[tierKey];
        const itemName = items[Math.floor(Math.random()*items.length)];
        return { forcedTierKey:tierKey, forcedTierName:TIERS.find(t=>t.key===tierKey).name, forcedItemName:itemName };
      }
      return null;
    }

    function rollOnce(){
      const upcoming=state.rolls+1; const surge=milestoneLuckMultiplier(upcoming);
      if(surge>1){ spawnBanner(`Luck Surge x${surge} (1 roll)`,"luck", surge===10?"b-omniversal":"b-divine"); }

      let tiers=TIERS.filter(t=>t.key!=="exclusive");
      tiers=applyWeightModifiers(tiers,surge);

      const now=Date.now(); const wEff=state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      const exclusiveActive=wEff && (wEff.name==="Eternal Eclipse" || wEff.name==="Cosmic Tempest");

      const forced = state.commands.nextRoll && !state.commands.nextRoll.applied ? state.commands.nextRoll : null;
      const guaranteePick=consumeGuaranteeIfAnyAndPick();

      let tierKey,tierName, prePickedItem=null;
      if(guaranteePick){
        tierKey=guaranteePick.forcedTierKey; tierName=guaranteePick.forcedTierName; prePickedItem=guaranteePick.forcedItemName;
      } else if(forced && forced.rarity){
        tierKey=forced.rarity; tierName=TIERS.find(t=>t.key===tierKey)?.name || forced.rarity;
        if(forced.name){ prePickedItem=forced.name; }
        state.commands.nextRoll.applied=true; if(forced.clearAfter){ setTimeout(()=>{ state.commands.nextRoll=null; saveState(); }, 0); }
        saveState();
      } else {
        const chances=toChances(tiers); const pickedTier=pickTier(chances); tierKey=pickedTier.key; tierName=pickedTier.name;
        if(exclusiveActive){
          const secretGateBase=0.00002; const luckBoostSecret=Math.min(0.00002,(state.activeEffects.luck||0)*0.000004);
          const passExclusive=Math.random()<(secretGateBase+luckBoostSecret);
          if(passExclusive){ tierKey="exclusive"; tierName="Exclusive"; }
        }
      }

      const itemTiers=buildItemTierWeightsFromIndex(TIERS.filter(t=>t.key!=="exclusive"));
      const itemWeighted=applyWeightModifiers(itemTiers,surge);
      const itemChances=toChances(itemWeighted);
      const baseItemChance=0.02; const luckBoost=Math.min(0.03,(state.activeEffects.luck||0)*0.015);
      const rollItem=(!prePickedItem) && Math.random()<(baseItemChance+luckBoost);

      state.rolls++;

      let isNew=false, displayName=null, displayRarityClass=null;

      function tryWeatherItemBias(defaultTierKey){
        const bias=state.activeEffects.weatherBiasItem; if(!bias) return null;
        const items=INDEX_ITEMS[defaultTierKey]||[]; if(!items.includes(bias.name)) return null;
        const pass=Math.random()<(bias.boost||0); return pass?bias.name:null;
      }

      if(prePickedItem){
        displayName=prePickedItem; displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
        processIndexItem(tierKey,tierName,prePickedItem,surge); isNew=markNew(tierKey,prePickedItem);
      } else if(rollItem){
        const itemTier=pickTier(itemChances);
        const drop=pickConsumableFromTier(itemTier.key);
        if(drop){
          const isTotem = drop.type==="totem" || drop.type==="totem_random";
          let allow=true;
          if(isTotem){
            if(drop.type==="totem_random"){ allow=Math.random()<(1/350); }
            else { const wName=drop.weather||""; const cat=weatherCategoryForName(wName); const mult=cat==="normal"?220:cat==="rare"?350:500; allow=Math.random()<(1/mult); }
          }
          if(allow){
            displayName=drop.name; displayRarityClass=TIERS.find(t=>t.key===drop.rarity)?.colorClass || "";
            const autosell=shouldAutoSellItems(itemTier.key);
            if(!autosell){
              if(state.inventoryItems.length<ITEMS_MAX){
                state.inventoryItems.push({ type:drop.type, tier:itemTier.key, tierName:itemTier.name, name:drop.name, roll:state.rolls, effect:drop });
              } else { if(!state.fullAnnouncedItems){ spawnBanner(`Items inventory is full ${ITEMS_MAX}/${ITEMS_MAX}`,"announce",""); state.fullAnnouncedItems=true; } }
            }
          } else {
            const biasPick=tryWeatherItemBias(tierKey); const itemName=biasPick || pickIndexItem(tierKey);
            displayName=itemName || tierName; displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
            processIndexItem(tierKey,tierName,itemName,surge); if(itemName) isNew=markNew(tierKey,itemName);
          }
        } else {
          const biasPick=tryWeatherItemBias(tierKey); const itemName=biasPick || pickIndexItem(tierKey);
          displayName=itemName || tierName; displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey,tierName,itemName,surge); if(itemName) isNew=markNew(tierKey,itemName);
        }
      } else {
        if(tierKey==="exclusive"){
          const itemName=(wEff && (wEff.name==="Eternal Eclipse" || wEff.name==="Cosmic Tempest"))?"Gem Of Gem":null;
          displayName=itemName || "Exclusive"; displayRarityClass="b-exclusive";
          processIndexItem("exclusive","Exclusive",displayName==="Exclusive"?null:displayName,surge);
          if(displayName && displayName!=="Exclusive") isNew=markNew("exclusive",displayName);
        } else {
          const chosenName=tryWeatherItemBias(tierKey) || pickIndexItem(tierKey);
          displayName=chosenName || tierName; displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey,tierName,chosenName,surge); if(chosenName) isNew=markNew(tierKey,chosenName);
        }
      }

      saveState();
      showGlow();
      renderResult(displayName,tierKey,displayRarityClass);
      if(isNew) spawnBanner(`NEW collected: [${tierName}] ${displayName}`,"new",displayRarityClass);

      const highOrder=["divine","celestial","transcendent","eternal","omniversal"];
      if(highOrder.includes(tierKey)){ playInlineCutscene(tierKey, displayName); }

      renderButtonsState(); renderIndex(); renderInventory();
    }

    /* ---------------- UI ---------------- */
    const elResult=document.getElementById("resultText");
    const elRarity=document.getElementById("rarityText");
    const elIndexPanel=document.getElementById("indexPanel");
    const elIndexGrid=document.getElementById("indexGrid");
    const elIndexCompletion=document.getElementById("indexCompletion");
    const elInventoryPanel=document.getElementById("inventoryPanel");
    const elInventoryList=document.getElementById("inventoryList");
    const elCommandsPanel=document.getElementById("commandsPanel");

    function renderResult(name,tierKey,rarityClass){
      elResult.textContent=name || "Unknown";
      elRarity.textContent=tierKey ? tierKey.toUpperCase() : "";
      elRarity.className="rarity badge "+(rarityClass||"");
      renderWeatherBackdrop();
    }
    function renderButtonsState(){
      const elAutoBtn=document.getElementById("btnAuto");
      if(state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)"; }
      const elAutoSellValue=document.getElementById("autoSellValue");
      const elModeValue=document.getElementById("modeValue");
      const currentThreshold=state.mode==="Items"?state.autoSellItems:state.autoSellRolled;
      elAutoSellValue.textContent=currentThreshold==="off" ? "Off" : labelForAutoSell(currentThreshold);
      elModeValue.textContent=state.mode;
    }
    function labelForAutoSell(val){ const t=TIERS.find(x=>x.key===val); return t?`${t.name}+`:"Off"; }
    function tierCompletion(key){ const items=INDEX_ITEMS[key]||[]; const unlocked=items.filter(n=>state.unlocks[key][n]).length; const pct=items.length?Math.round(unlocked/items.length*100):0; return { unlocked, total:items.length, percent:pct }; }
    function totalCompletion(){ let u=0,t=0; for(const k in INDEX_ITEMS){ const c=tierCompletion(k); u+=c.unlocked; t+=c.total; } return { unlocked:u,total:t,percent:t?Math.round(u/t*100):0 }; }

    function renderIndex(){
      const tot=totalCompletion(); elIndexCompletion.textContent=`Total ${tot.percent}%`;
      elIndexGrid.innerHTML="";
      TIERS.forEach(tier=>{
        const section=document.createElement("div"); section.className="index-section";
        const comp=tierCompletion(tier.key); const isExclusive=tier.key==="exclusive";
        const badgeClasses=`badge ${tier.colorClass} ${isExclusive?'locked':''}`;
        section.innerHTML=`<h4><span class="${badgeClasses}">${tier.name}</span></h4><div class="completion ${isExclusive?'locked':''}">${isExclusive?'—':comp.percent+'%'}</div>`;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(isExclusive){
          const name="Gem Of Gem"; const li=document.createElement("li"); li.className="index-item";
          const unlocked=!!state.unlocks[tier.key][name];
          li.innerHTML=`<span class="${unlocked?'':'locked'}">${name}</span><span class="${unlocked?'unlocked':'locked'}">${unlocked?'Unlocked':'Locked'}</span>`;
          ul.appendChild(li);
        } else {
          items.forEach(name=>{
            const li=document.createElement("li"); li.className="index-item";
            const unlocked=!!state.unlocks[tier.key][name];
            li.innerHTML=`<span class="${unlocked?'':'locked'}">${name}</span><span class="${unlocked?'unlocked':'locked'}">${unlocked?'Unlocked':'Locked'}</span>`;
            ul.appendChild(li);
          });
        }
        section.appendChild(ul); elIndexGrid.appendChild(section);
      });
    }

    /* ---------------- Delete confirmation (improved) ---------------- */
    const confirmWrap=document.getElementById("confirmWrap");
    const confirmTitle=document.getElementById("confirmTitle");
    const confirmBadge=document.getElementById("confirmBadge");
    const confirmName=document.getElementById("confirmName");
    const confirmDesc=document.getElementById("confirmDesc");
    const confirmCancel=document.getElementById("confirmCancel");
    const confirmDelete=document.getElementById("confirmDelete");

    let pendingDelete = null; // {type:'rolled'|'item', entry}
    function askDelete(entry, isRolled){
      const tier=TIERS.find(t=>t.key===entry.tier);
      const badgeClass=tier?tier.colorClass:"";
      confirmTitle.textContent="Confirm delete";
      confirmBadge.className=`badge ${badgeClass}`;
      confirmBadge.textContent=tier ? tier.name : "RARITY";
      confirmName.textContent=entry.name || "(Unknown)";
      const veryRare = ["divine","celestial","transcendent","eternal","omniversal","exclusive"].includes(entry.tier);
      confirmDesc.textContent = veryRare ? "This is a very rare item. Deleting it is permanent. Are you absolutely sure?" : "This action cannot be undone. Continue?";
      confirmWrap.style.display="flex";
      pendingDelete = { type:isRolled?'rolled':'item', entry };
    }
    confirmCancel.addEventListener("click",()=>{ confirmWrap.style.display="none"; pendingDelete=null; });
    confirmDelete.addEventListener("click",()=>{
      if(!pendingDelete){ confirmWrap.style.display="none"; return; }
      if(pendingDelete.type==="rolled"){ performDeleteRolledEntry(pendingDelete.entry); }
      else { performDeleteItemEntry(pendingDelete.entry); }
      confirmWrap.style.display="none"; pendingDelete=null;
    });

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
          const del=document.createElement("button"); del.textContent="Delete"; del.addEventListener("click",()=>askDelete(entry,true));
          right.appendChild(del); li.appendChild(left); li.appendChild(right); elInventoryList.appendChild(li);
        });
        const cap=document.createElement("div"); cap.className="inv-stats"; cap.style.marginTop="8px"; cap.textContent=`Rolled capacity ${state.inventoryRolled.length}/${ROLLED_MAX}`; elInventoryList.appendChild(cap);
      } else {
        const items=[...state.inventoryItems].reverse();
        items.forEach(entry=>{
          const tier=TIERS.find(t=>t.key===entry.tier); const badgeClass=tier?tier.colorClass:"";
          const li=document.createElement("li");
          const left=document.createElement("div"); const e=entry.effect;
          const effDesc = e?.type==="luck" ? `Luck +${Math.round(e.amount*100)}%`
                        : e?.type==="speed" ? `Speed +${Math.round(e.amount*100)}%`
                        : e?.type==="bias" ? `Bias → ${e.target?.toUpperCase()||''} +${Math.round(e.amount*100)}%`
                        : e?.type==="guarantee" ? `Guarantee Eternal+ next roll`
                        : e?.type==="totem" ? `Summons ${e.weather}`
                        : e?.type==="totem_random" ? `Summons random weather` : "";
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name}${effDesc?` (${effDesc})`:''} • #${entry.roll}`;
          const right=document.createElement("div"); right.className="inv-actions";
          const use=document.createElement("button"); use.textContent="Use"; use.addEventListener("click",()=>useItemEntry(entry));
          const del=document.createElement("button"); del.textContent="Delete"; del.addEventListener("click",()=>askDelete(entry,false));
          right.appendChild(use); right.appendChild(del); li.appendChild(left); li.appendChild(right); elInventoryList.appendChild(li);
        });
        const cap=document.createElement("div"); cap.className="inv-stats"; cap.style.marginTop="8px"; cap.textContent=`Items capacity ${state.inventoryItems.length}/${ITEMS_MAX}`; elInventoryList.appendChild(cap);
      }
    }
    function performDeleteRolledEntry(entry){
      const idx=state.inventoryRolled.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){ state.inventoryRolled.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedRolled=false; }
    }
    function performDeleteItemEntry(entry){
      const idx=state.inventoryItems.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){ state.inventoryItems.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedItems=false; }
    }

    function useItemEntry(entry){
      if(entry.effect){
        const eff=entry.effect;
        if(eff.type==="totem"){
          const metaSrc=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].find(w=>w.name===eff.weather);
          const dur=100+Math.floor(Math.random()*201);
          const meta={ luck:(metaSrc?.effects?.luck)||0, biasItem:(metaSrc?.effects?.biasItem)||null };
          addEffect({ name:eff.weather, type:"weather", duration:dur, rarity: classToTierKey(colorClassForWeather(eff.weather)), meta });
          const cl=colorClassForWeather(eff.weather);
          const icon=eff.weather==="Eternal Eclipse" ? "icon-eclipse" : eff.weather==="Storm" ? "icon-storm" : eff.weather==="Aurora Veil" ? "icon-aurora" : eff.weather==="Cosmic Tempest" ? "icon-tempest" : eff.weather==="Meteor Storm" ? "icon-meteor" : eff.weather==="Sunny Radiance" ? "icon-sunny" : "";
          spawnBanner(`Activated ${entry.name}`,"activate",cl); spawnBanner(`${eff.weather} started`,"weather",cl,icon);
        } else if(eff.type==="totem_random"){
          const r=Math.random(); let pool=WEATHERS.normal; if(r>=0.70 && r<0.95) pool=WEATHERS.rare; else if(r>=0.95) pool=WEATHERS.super;
          const w=pool[Math.floor(Math.random()*pool.length)]; const dur=100+Math.floor(Math.random()*201);
          const meta={ luck:(w.effects?.luck)||0, biasItem:(w.effects?.biasItem)||null };
          addEffect({ name:w.name, type:"weather", duration:dur, rarity: classToTierKey(w.colorClass), meta });
          const icon=w.name==="Eternal Eclipse" ? "icon-eclipse" : w.name==="Storm" ? "icon-storm" : w.name==="Aurora Veil" ? "icon-aurora" : w.name==="Cosmic Tempest" ? "icon-tempest" : w.name==="Meteor Storm" ? "icon-meteor" : w.name==="Sunny Radiance" ? "icon-sunny" : "";
          spawnBanner(`Activated ${entry.name}`,"activate",w.colorClass); spawnBanner(`${w.name} started`,"weather",w.colorClass,icon);
        } else if(eff.type==="guarantee" && eff.name==="Origin Crystal"){
          addEffect(eff); // store guarantee
        } else {
          addEffect({ name:eff.name, type:eff.type, amount:eff.amount, target:eff.target, duration:eff.duration, rarity:eff.rarity });
          const cl=TIERS.find(t=>t.key===eff.rarity)?.colorClass || ""; spawnBanner(`Activated ${entry.name}`,"activate",cl);
        }
      }
      performDeleteItemEntry(entry); renderActiveEffects();
    }

    /* ---------------- Commands (ROBUST, working) ---------------- */
    const cmdLuck=document.getElementById("cmdLuck");
    const cmdLuckDuration=document.getElementById("cmdLuckDuration");
    const cmdBiasRarity=document.getElementById("cmdBiasRarity");
    const cmdBiasAmount=document.getElementById("cmdBiasAmount");

    const cmdWeatherSel=document.getElementById("cmdWeather");
    const cmdWeatherDuration=document.getElementById("cmdWeatherDuration");

    const cmdNextRaritySel=document.getElementById("cmdNextRarity");
    const cmdNextNameSel=document.getElementById("cmdNextName");
    const cmdNextClearSel=document.getElementById("cmdNextClear");

    const cmdGiveRaritySel=document.getElementById("cmdGiveRarity");
    const cmdGiveItemSel=document.getElementById("cmdGiveItem");
    const cmdGiveActionSel=document.getElementById("cmdGiveAction");

    const cmdResetBtn=document.getElementById("cmdReset");
    const cmdClearEffectsBtn=document.getElementById("cmdClearEffects");
    const cmdApplyBtn=document.getElementById("cmdApply");

    document.getElementById("btnCommands").addEventListener("click",()=>{
      const vis=elCommandsPanel.style.display!=="none";
      elCommandsPanel.style.display=vis?"none":"block";
    });

    function populateCommandSelectors(){
      // bias rarity, next rarity, give rarity
      const rarityOptions = TIERS.map(t=>`<option value="${t.key}">${t.name}</option>`).join("");
      cmdBiasRarity.innerHTML = rarityOptions;
      cmdNextRaritySel.innerHTML = rarityOptions;
      cmdGiveRaritySel.innerHTML = rarityOptions;

      // weather names
      const weatherNames=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].map(w=>w.name);
      cmdWeatherSel.innerHTML = weatherNames.map(n=>`<option>${n}</option>`).join("");

      // next names for selected rarity
      function refreshNamesForRarity(selKey,targetSel){
        const items=INDEX_ITEMS[selKey]||[];
        targetSel.innerHTML = ['<option value="">(none)</option>', ...items.map(n=>`<option value="${n}">${n}</option>`)].join("");
      }
      refreshNamesForRarity(cmdNextRaritySel.value||"common", cmdNextNameSel);
      cmdNextRaritySel.addEventListener("change",()=>refreshNamesForRarity(cmdNextRaritySel.value, cmdNextNameSel));

      // give items for rarity (consumables list)
      function refreshGiveItemsForRarity(key){
        const items=(ITEM_DROPS[key]||[]);
        cmdGiveItemSel.innerHTML = items.length
          ? items.map(d=>`<option value="${encodeURIComponent(JSON.stringify(d))}">${d.name}</option>`).join("")
          : `<option value="">(none)</option>`;
      }
      refreshGiveItemsForRarity(cmdGiveRaritySel.value||"common");
      cmdGiveRaritySel.addEventListener("change",()=>refreshGiveItemsForRarity(cmdGiveRaritySel.value));
    }

    cmdResetBtn.addEventListener("click",()=>{
      if(!confirm("Reset all data (rolls, unlocks, inventory, effects, commands)?")) return;
      localStorage.clear();
      location.reload();
    });
    cmdClearEffectsBtn.addEventListener("click",()=>{
      state.effectInstances=[]; state.guarantees=[]; deriveEffectsTotals(); saveState(); renderActiveEffects(); renderWeatherBackdrop(); spawnBanner("Effects & guarantees cleared","announce","b-common");
    });
    cmdApplyBtn.addEventListener("click",()=>{
      // Luck
      const luckAmt = parseFloat(cmdLuck.value);
      if(!Number.isNaN(luckAmt) && luckAmt!==0){
        const dur = parseInt(cmdLuckDuration.value,10)||30;
        addEffect({ name:`Command Luck +${Math.round(luckAmt*100)}%`, type:"luck", amount:luckAmt, duration:dur, rarity: luckAmt>=1 ? "legendary" : luckAmt>=0.5 ? "epic" : "rare" });
        spawnBanner(`CMD: Luck +${Math.round(luckAmt*100)}% (${dur}s)`,"activate","b-epic");
      }

      // Bias
      const biasKey = cmdBiasRarity.value;
      const biasAmt = parseFloat(cmdBiasAmount.value);
      if(biasKey && !Number.isNaN(biasAmt) && biasAmt>0){
        addEffect({ name:`Bias → ${biasKey.toUpperCase()} +${Math.round(biasAmt*100)}%`, type:"bias", target:biasKey, amount:biasAmt, duration:60, rarity:"epic" });
        spawnBanner(`CMD: Bias → ${biasKey.toUpperCase()} +${Math.round(biasAmt*100)}%`,"activate","b-epic");
      }

      // Weather
      const wName = cmdWeatherSel.value;
      if(wName){
        const dur = parseInt(cmdWeatherDuration.value,10)||150;
        const metaSrc=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].find(w=>w.name===wName);
        const meta={ luck:(metaSrc?.effects?.luck)||0, biasItem:(metaSrc?.effects?.biasItem)||null };
        addEffect({ name:wName, type:"weather", duration:dur, rarity: classToTierKey(metaSrc?.colorClass||""), meta });
        const icon=wName==="Eternal Eclipse" ? "icon-eclipse" : wName==="Storm" ? "icon-storm" : wName==="Aurora Veil" ? "icon-aurora" : wName==="Cosmic Tempest" ? "icon-tempest" : wName==="Meteor Storm" ? "icon-meteor" : wName==="Sunny Radiance" ? "icon-sunny" : "";
        spawnBanner(`CMD: ${wName} (${dur}s)`,"weather",metaSrc?.colorClass||"",icon);
      }

      // Next roll
      const nextRarity = cmdNextRaritySel.value;
      const nextName = cmdNextNameSel.value || null;
      const clearAfter = cmdNextClearSel.value==="clear";
      if(nextRarity){
        if(nextName && !(INDEX_ITEMS[nextRarity]||[]).includes(nextName)){
          alert("Selected name does not exist in the chosen rarity’s Index.");
        } else {
          state.commands.nextRoll={ rarity:nextRarity, name: nextName || null, applied:false, clearAfter };
          saveState();
          spawnBanner(`CMD: Next roll → ${nextRarity.toUpperCase()} ${nextName?`(${nextName})`:''}`,"announce",TIERS.find(t=>t.key===nextRarity)?.colorClass||"");
        }
      }

      // Give effect/item
      const effStr = cmdGiveItemSel.value;
      const action = cmdGiveActionSel.value;
      const giveRarity = cmdGiveRaritySel.value;
      if(effStr){
        const eff = JSON.parse(decodeURIComponent(effStr));
        if(action==="activate"){
          addEffect(eff);
          spawnBanner(`CMD: Activated ${eff.name}`,"activate",TIERS.find(t=>t.key===eff.rarity)?.colorClass||"");
        } else {
          if(state.inventoryItems.length<ITEMS_MAX){
            state.inventoryItems.push({ type:eff.type, tier:giveRarity, tierName:TIERS.find(t=>t.key===giveRarity).name, name:eff.name, roll:state.rolls, effect:eff });
            saveState(); renderInventory();
            spawnBanner(`CMD: Added ${eff.name} to Items`,"announce",TIERS.find(t=>t.key===giveRarity)?.colorClass||"");
          } else {
            spawnBanner(`Items inventory is full ${ITEMS_MAX}/${ITEMS_MAX}`,"announce","");
          }
        }
      }
    });

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
    document.getElementById("btnAuto").addEventListener("click",()=>toggleAuto());
    document.getElementById("btnIndex").addEventListener("click",()=>{ const vis=elIndexPanel.style.display!=="none"; if(vis){ elIndexPanel.style.display="none"; } else { elIndexPanel.style.display="block"; elInventoryPanel.style.display="none"; renderIndex(); } });
    document.getElementById("btnInventory").addEventListener("click",()=>{ const vis=elInventoryPanel.style.display!=="none"; if(vis){ elInventoryPanel.style.display="none"; } else { elInventoryPanel.style.display="block"; elIndexPanel.style.display="none"; renderInventory(); } });
    document.getElementById("autoSellPrev").addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),-1)));
    document.getElementById("autoSellNext").addEventListener("click",()=>setAutoSell(cycle(autoSellOptions,(state.mode==="Items"?state.autoSellItems:state.autoSellRolled),1)));
    document.getElementById("modePrev").addEventListener("click",()=>setMode(cycle(modes,state.mode,-1)));
    document.getElementById("modeNext").addEventListener("click",()=>setMode(cycle(modes,state.mode,1)));

    /* ---------------- Init ---------------- */
    loadState();
    renderButtonsState();
    renderActiveEffects();
    renderWeatherBackdrop();
    renderInventory();
    renderIndex();
    populateCommandSelectors();
    const elAutoBtn=document.getElementById("btnAuto");
    if(state.auto && state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On"; updateAutoInterval(); }
    else { elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)"; }
    scheduleNextWeather();
  </script>
</body>
</html>
