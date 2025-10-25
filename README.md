<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Cinematics, Weather FX, Commands (Overhauled)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root{
      --bg:#0e0f13; --panel:#151822; --text:#e7e9ee; --muted:#9aa0ab; --accent:#6ea8fe;
      --gold:#ffd700; --warn:#ff6666;
      --fx1:#7bb7ff; --fx2:#caa6ff; --fx3:#ffbf66;
    }
    body{margin:0;font-family:system-ui,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:var(--bg);color:var(--text);display:grid;place-items:center;min-height:100vh;}
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
    .icon-bh{background:radial-gradient(circle at 50% 50%, rgba(0,0,0,0.95) 60%, rgba(80,80,80,0.35) 62%, transparent 70%);}

    /* Active effects tray */
    .active-effects{position:absolute;bottom:10px;right:10px;z-index:6;font-size:12px;max-width:70%;display:flex;flex-direction:column;align-items:flex-end;gap:6px;}
    .effect-entry{display:flex;gap:8px;align-items:center;padding:6px 10px;border-radius:10px;background:rgba(27,34,50,0.9);border:1px solid #2a3449;box-shadow:0 6px 18px rgba(0,0,0,0.25);font-weight:600;}

    /* Weather backdrops */
    .weather-bg{position:absolute;inset:0;z-index:1;pointer-events:none;opacity:0;animation:weatherFadeIn .7s ease-out forwards;}
    @keyframes weatherFadeIn{from{opacity:0}to{opacity:1}}
    .wb-sunny{background:
      radial-gradient(140% 140% at 50% 0%, rgba(255,215,120,0.22), transparent 60%),
      linear-gradient(180deg, rgba(255,232,170,0.10), rgba(0,0,0,0.0));
    }
    .wb-storm{background:linear-gradient(180deg, rgba(25,30,55,0.72), rgba(0,0,0,0.78));}
    .wb-blizzard{background:linear-gradient(180deg, rgba(210,235,255,0.38), rgba(0,0,0,0.60));}
    .wb-meteor{background:radial-gradient(160% 160% at 20% -20%, rgba(255,120,80,0.22), transparent 60%), linear-gradient(180deg, rgba(120,50,30,0.28), rgba(0,0,0,0.6));}
    .wb-aurora{background:linear-gradient(120deg,rgba(120,200,255,0.20),rgba(180,120,255,0.20),rgba(120,255,200,0.20));background-size:600% 600%;animation:auroraShift 18s ease infinite;}
    @keyframes auroraShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
    .wb-eclipse{background:radial-gradient(circle at 50% 50%,rgba(0,0,0,0.93),rgba(0,0,0,0.78)),radial-gradient(circle at 50% 50%,rgba(255,200,120,0.08),transparent 70%);}
    .wb-blackhole{
      background:
        radial-gradient(closest-side, rgba(0,0,0,0.98) 42%, rgba(0,0,0,0.85) 55%, rgba(0,0,0,0.65) 75%),
        radial-gradient(circle at 50% 50%, rgba(60,40,90,0.25), transparent 70%);
    }
    .wb-tempest{background:radial-gradient(circle at 50% 50%,rgba(120,80,255,0.25),transparent 70%),radial-gradient(circle at 70% 30%,rgba(80,200,255,0.25),transparent 70%);}
    .wb-fog{background:radial-gradient(100% 100% at 50% 50%, rgba(185,195,210,0.16), rgba(0,0,0,0.57));}
    .particles{position:absolute;inset:0;overflow:hidden;filter:blur(0.15px);pointer-events:none;}

    /* Weather particles and effects (overhauled) */
    /* Storm */
    .rain-drop{position:absolute;width:2px;height:20px;background:linear-gradient(to bottom,rgba(180,200,255,0.95),rgba(180,200,255,0.25));border-radius:1px;}
    @keyframes rainFall{0%{transform:translateY(-60px)}100%{transform:translateY(460px)}}
    .bolt{position:absolute;width:3px;height:220px;background:linear-gradient(to bottom,rgba(255,255,255,0.96),transparent);filter:blur(0.6px);opacity:0;transform-origin:top left;}
    .bolt.glow{box-shadow:0 0 24px rgba(255,255,255,0.5);}
    .bolt:before{content:"";position:absolute;left:-22px;top:60px;width:2px;height:120px;background:linear-gradient(to bottom,rgba(255,255,255,0.95),transparent);transform:rotate(-18deg);opacity:0.9;}
    .bolt:after{content:"";position:absolute;left:16px;top:120px;width:2px;height:90px;background:linear-gradient(to bottom,rgba(255,255,255,0.9),transparent);transform:rotate(24deg);opacity:0.8;}

    /* Blizzard */
    .snow-flake{position:absolute;width:6px;height:6px;background:white;border-radius:50%;opacity:0.9;box-shadow:0 0 6px rgba(255,255,255,0.55);}
    @keyframes snowFall{0%{transform:translateY(-40px) translateX(0)}50%{transform:translateY(220px) translateX(18px)}100%{transform:translateY(480px) translateX(0)}}
    .blizzard-gust{position:absolute;left:-50%;top:20%;width:220%;height:40%;background:linear-gradient(90deg,rgba(255,255,255,0.08),transparent);filter:blur(6px);opacity:0.0;}

    /* Fog */
    .fog-mist{position:absolute;left:-40%;top:0;width:180%;height:120%;background:radial-gradient(circle,rgba(255,255,255,0.08),transparent 70%);filter:blur(8px);opacity:0.75;animation:fogDrift 24s linear infinite;}
    @keyframes fogDrift{0%{transform:translateX(0)}50%{transform:translateX(10%)}100%{transform:translateX(0)}}

    /* Aurora */
    .ribbon{position:absolute;width:64%;height:20px;left:18%;border-radius:999px;filter:blur(2px);opacity:0.75;animation:ribbonWave 9s ease-in-out infinite;}
    @keyframes ribbonWave{0%{transform:translateY(0) skewX(6deg)}50%{transform:translateY(44px) skewX(-6deg)}100%{transform:translateY(0) skewX(6deg)}}
    .aurora-veil{position:absolute;left:0;right:0;top:0;bottom:0;mix-blend-mode:screen;opacity:0.65;animation:veilShift 18s ease-in-out infinite;}
    @keyframes veilShift{0%{background:radial-gradient(120% 120% at 20% 30%, rgba(120,255,255,0.35), transparent 50%)}50%{background:radial-gradient(120% 120% at 70% 60%, rgba(255,160,220,0.35), transparent 50%)}100%{background:radial-gradient(120% 120% at 20% 30%, rgba(120,255,255,0.35), transparent 50%)}}

    /* Sunny */
    .sun-core{position:absolute;left:50%;top:10%;transform:translateX(-50%);width:120px;height:120px;border-radius:50%;background:radial-gradient(circle,rgba(255,230,150,0.95),rgba(255,200,100,0.35),transparent 65%);box-shadow:0 0 60px rgba(255,210,140,0.45);animation:pulseSun 3.6s ease-in-out infinite;}
    @keyframes pulseSun{0%{transform:translateX(-50%) scale(1)}50%{transform:translateX(-50%) scale(1.08)}100%{transform:translateX(-50%) scale(1)}}
    .sun-ray{position:absolute;left:50%;top:10%;transform:translateX(-50%) rotate(0deg);width:2px;height:160px;background:linear-gradient(180deg,rgba(255,235,160,0.9),rgba(255,235,160,0));opacity:0.55;transform-origin:bottom center;animation:raySpin 8s linear infinite;}
    @keyframes raySpin{0%{transform:translateX(-50%) rotate(0deg)}100%{transform:translateX(-50%) rotate(360deg)}}

    /* Cosmic Tempest + Meteor */
    .shooting-star{position:absolute;width:2px;height:2px;border-radius:50%;background:white;box-shadow:0 0 10px rgba(255,255,255,0.65);opacity:0.0;}
    .meteor{position:absolute;width:3px;height:26px;background:linear-gradient(to bottom,rgba(255,160,120,0.95),rgba(255,160,120,0));border-radius:2px;filter:blur(0.25px);opacity:0.95;}

    /* Black hole accretion disk */
    .bh-core{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:220px;height:220px;border-radius:50%;background:#000;box-shadow:0 0 40px rgba(0,0,0,0.8);z-index:2;}
    .bh-disk{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%) rotate(0deg);width:360px;height:360px;border-radius:50%;background:
      conic-gradient(from 0deg, rgba(255,220,160,0.0) 0deg, rgba(255,220,160,0.75) 80deg, rgba(255,160,120,0.85) 160deg, rgba(200,120,255,0.7) 220deg, rgba(120,160,255,0.75) 300deg, rgba(255,220,160,0.0) 360deg);
      filter:blur(0.6px);box-shadow:0 0 40px rgba(255,200,140,0.25);animation:diskSpin 18s linear infinite;z-index:1;}
    @keyframes diskSpin{0%{transform:translate(-50%,-50%) rotate(0deg)}100%{transform:translate(-50%,-50%) rotate(360deg)}}
    .bh-lens{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:520px;height:520px;border-radius:50%;box-shadow:inset 0 0 80px rgba(150,120,255,0.12), inset 0 0 140px rgba(255,220,160,0.12);opacity:0.9;}

    /* Cinematic: inline stage */
    .cinema-stage-inline{position:absolute;inset:0;overflow:hidden;pointer-events:none;z-index:5;}
    .star{position:absolute;border-radius:50%;background:radial-gradient(circle, rgba(255,255,255,0.95), rgba(255,255,255,0));filter:blur(0.2px);}
    @keyframes twinkle{0%,100%{opacity:0.3}50%{opacity:1}}
    .ring{position:absolute;border-radius:50%;border:2px solid rgba(255,255,255,0.25);}
    @keyframes pulseRing{0%{transform:scale(0.6);opacity:0.0}50%{opacity:1}100%{transform:scale(1.4);opacity:0}}
    .beam{position:absolute;background:linear-gradient(180deg,rgba(255,255,255,0.9),rgba(255,255,255,0));filter:blur(1.2px);mix-blend-mode:screen;}
    .shard{position:absolute;width:8px;height:24px;background:linear-gradient(180deg,rgba(150,200,255,0.9),rgba(150,200,255,0));transform-origin:center;filter:blur(0.4px);}

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
    /* Dark Matter style for inventory only (exclusive item specific) */
    .b-exclusive-dark{background:radial-gradient(circle at 50% 50%, #0a0a10, #1a1a28); color:#d9ccff; box-shadow:inset 0 0 18px rgba(160,120,255,0.25), 0 0 20px rgba(60,0,140,0.25);}

    /* Glow */
    .glow{position:absolute;inset:-40%;border-radius:50%;background:radial-gradient(closest-side,rgba(110,168,254,0.25),transparent 65%);filter:blur(12px);animation:glow 1.1s ease-out forwards;z-index:2;}
    @keyframes glow{0%{opacity:0;transform:scale(0.7)}50%{opacity:1}100%{opacity:0;transform:scale(1.2)}}

    /* Secret modal */
    #secretBtn{position:absolute;top:6px;right:6px;width:28px;height:28px;opacity:0;cursor:pointer;z-index:40;}
    .modal{display:none;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:#121521;padding:16px;border:1px solid #2a3449;border-radius:12px;box-shadow:0 10px 30px rgba(0,0,0,0.5);z-index:50;width:360px;}
    .modal h3{margin:0 0 10px;}
    .modal .row{display:flex;gap:8px;align-items:center;margin:8px 0;}
    .modal input[type=password]{flex:1;background:#1b2232;border:1px solid #2a3449;border-radius:8px;padding:8px;color:var(--text);}
    .modal button{width:100%}

    /* Commands panel (overhauled) */
    #commandsPanel{display:none;}
    .cmds{display:grid;gap:16px;}
    .cmd-section{border:1px solid #2a3449;border-radius:10px;padding:12px;background:#101624;}
    .cmd-section h4{margin:0 0 8px;font-size:14px;color:#cfd6ff;font-weight:800;}
    .cmd-row{display:grid;grid-template-columns:1fr 1fr;gap:10px;align-items:center;}
    .cmd-row.triple{grid-template-columns:1fr 1fr 1fr;}
    .cmd-row.single{grid-template-columns:1fr;}
    .cmd-hint{font-size:12px;color:#9aa0ab;}
    select,input[type=text]{background:#1b2232;border:1px solid #2a3449;border-radius:8px;padding:8px;color:var(--text);}
    .cmd-actions{display:flex;justify-content:space-between;gap:10px;}
    .cmd-actions-left{display:flex;gap:10px;}
    .cmd-actions-right{display:flex;gap:10px;}
    .cmd-actions button{min-width:140px;}

    /* Delete confirmation modal */
    .confirm-modal{display:none;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:#0f1320;padding:16px;border:1px solid #2a3449;border-radius:12px;box-shadow:0 12px 36px rgba(0,0,0,0.6);z-index:60;width:480px;}
    .confirm-title{margin:0 0 10px;font-size:16px;color:#cfd6ff;font-weight:800;}
    .confirm-body{font-size:14px;color:#c7d1e5;margin-bottom:12px;}
    .confirm-actions{display:flex;justify-content:flex-end;gap:10px;}
    .confirm-actions .danger{background:#2a1f12;border-color:#4a371e;}
    .confirm-item-row{display:flex;align-items:center;gap:8px;}
    .confirm-badge{display:inline-block;padding:2px 8px;border-radius:999px;font-size:12px;font-weight:700;}
  </style>
</head>
<body>
  <div class="app">
    <div id="secretBtn" aria-hidden="true"></div>
    <div id="passwordModal" class="modal" aria-hidden="true">
      <h3>Enter password</h3>
      <div class="row"><input type="password" id="secretInput" autocomplete="off" /></div>
      <button id="secretConfirm">Confirm</button>
    </div>

    <div id="deleteConfirm" class="confirm-modal" aria-hidden="true">
      <div class="confirm-title">Confirm deletion</div>
      <div class="confirm-body">
        <div class="confirm-item-row">
          <span id="confirmBadge" class="confirm-badge"></span>
          <span id="confirmText"></span>
        </div>
      </div>
      <div class="confirm-actions">
        <button id="confirmCancel">Cancel</button>
        <button id="confirmDeleteBtn" class="danger">Delete</button>
      </div>
    </div>

    <div class="content">
      <div class="panel">
        <div class="roll-area" id="rollArea">
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
          <div class="active-effects" id="activeEffects"></div>
          <div class="cinema-stage-inline" id="cinemaStageInline"></div>
        </div>
        <div class="controls" id="controls">
          <button id="btnRoll">Roll</button>
          <button id="btnAuto" disabled>Auto Roll (locked)</button>
          <button id="btnIndex">Index</button>
          <button id="btnInventory">Inventory</button>
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

      <div class="panel" id="commandsPanel" style="display:none;">
        <h3>Commands</h3>
        <div class="cmds">
          <div class="cmd-section">
            <h4>Boost effects</h4>
            <div class="cmd-row triple">
              <input type="text" id="cmdLuck" placeholder="Luck (e.g. 0.5 = +50%)" />
              <input type="text" id="cmdSpeed" placeholder="Speed (e.g. 0.5 = +50%)" />
              <select id="cmdLuckScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
          </div>
          <div class="cmd-section">
            <h4>Weather</h4>
            <div class="cmd-row">
              <select id="cmdWeather"></select>
              <select id="cmdWeatherScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
          </div>
          <div class="cmd-section">
            <h4>Force next roll</h4>
            <div class="cmd-row triple">
              <select id="cmdNextRarity"></select>
              <select id="cmdNextName"></select>
              <select id="cmdNextScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
          </div>
          <div class="cmd-section">
            <h4>Data</h4>
            <div class="cmd-row">
              <button id="cmdReset">Reset Entire Data</button>
              <select id="cmdResetScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
            <div class="cmd-row">
              <button id="cmdClear">Clear Effects</button>
              <select id="cmdClearScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
          </div>
          <div class="cmd-section">
            <h4>Give effect/item</h4>
            <div class="cmd-row triple">
              <select id="cmdGiveRarity"></select>
              <select id="cmdGiveItem"></select>
              <select id="cmdGiveScope"><option>GLOBAL</option><option>LOCAL</option></select>
            </div>
          </div>
          <div class="cmd-actions">
            <div class="cmd-actions-left">
              <button id="cmdPresetBoost">Preset: Lucky Burst</button>
              <button id="cmdPresetEclipse">Preset: Eclipse</button>
            </div>
            <div class="cmd-actions-right">
              <button id="cmdConfirm">Apply</button>
            </div>
          </div>
        </div>
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
      exclusive:["Dark Matter Shard"] /* inventory styled dark, index rainbow */
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
        { name:"Cosmic Tempest Totem", rarity:"transcendent", type:"totem", weather:"Cosmic Tempest" },
        { name:"Black Hole Totem", rarity:"transcendent", type:"totem", weather:"Black Hole" }
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
        {name:"Fog", colorClass:"b-uncommon", effects:{luck:+0.08}},
        {name:"Aurora Veil", colorClass:"b-mythic", effects:{luck:+0.20}}
      ],
      rare: [
        {name:"Meteor Storm", colorClass:"b-legendary", effects:{luck:+0.26}}
      ],
      super: [
        {name:"Eternal Eclipse", colorClass:"b-divine", effects:{luck:+0.80, biasItem:{name:"Timeweaver Crest", boost:+0.50}}},
        {name:"Cosmic Tempest", colorClass:"b-omniversal", effects:{luck:+0.65}},
        {name:"Black Hole", colorClass:"b-transcendent", effects:{luck:+0.52, speed:+0.35}}
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
      secretUnlocked:"sol_rng_secret_unlocked",
      cmdQueue:"sol_rng_commands_queue"
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
      effectInstances:[], persistentGuarantees:[],
      mode:"Rolled", secretUnlocked:false, cmdQueue:null,
      cutsceneActive:false, cutscenePausedAuto:false
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
      const secretRaw=localStorage.getItem(STORAGE_KEYS.secretUnlocked);
      const cmdRaw=localStorage.getItem(STORAGE_KEYS.cmdQueue);

      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      const inv = invRaw?JSON.parse(invRaw):{rolled:[],items:[]};
      state.inventoryRolled = inv.rolled || [];
      state.inventoryItems = inv.items || [];
      state.autoSellRolled=autoSellRolledRaw||"off";
      state.autoSellItems=autoSellItemsRaw||"off";
      const parsedEffects = effInstRaw ? JSON.parse(effInstRaw) : { timed:[], guarantees:[] };
      const now=Date.now();
      state.effectInstances = (parsedEffects.timed||[]).filter(e=>e.expiresAt>now);
      state.persistentGuarantees = parsedEffects.guarantees || [];
      deriveEffectsTotals();
      state.mode = modeRaw || "Rolled";
      state.secretUnlocked = secretRaw==="true";
      state.cmdQueue = cmdRaw ? JSON.parse(cmdRaw) : null;
    }
    function saveState(){
      localStorage.setItem(STORAGE_KEYS.rolls,String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks,JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto,state.auto?"true":"false");
      localStorage.setItem(STORAGE_KEYS.inv,JSON.stringify({rolled:state.inventoryRolled,items:state.inventoryItems}));
      localStorage.setItem(STORAGE_KEYS.autoSellRolled,state.autoSellRolled);
      localStorage.setItem(STORAGE_KEYS.autoSellItems,state.autoSellItems);
      localStorage.setItem(STORAGE_KEYS.effects,JSON.stringify({ timed:state.effectInstances, guarantees:state.persistentGuarantees }));
      localStorage.setItem(STORAGE_KEYS.mode,state.mode);
      localStorage.setItem(STORAGE_KEYS.secretUnlocked,state.secretUnlocked?"true":"false");
      if(state.cmdQueue) localStorage.setItem(STORAGE_KEYS.cmdQueue,JSON.stringify(state.cmdQueue)); else localStorage.removeItem(STORAGE_KEYS.cmdQueue);
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

    /* ---------------- Weather visual (overhauled) ---------------- */
    let weatherEventTimers = [];
    function clearWeatherEvents(){ weatherEventTimers.forEach(t=>clearInterval(t)); weatherEventTimers=[]; }

    function renderWeatherBackdrop(){
      const area=document.getElementById("rollArea");
      const old=area.querySelector(".weather-bg"); if(old) old.remove();
      clearWeatherEvents();

      const now=Date.now();
      const weather=state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      if(!weather) return;
      const map={"Sunny Radiance":"wb-sunny","Storm":"wb-storm","Blizzard":"wb-blizzard","Meteor Storm":"wb-meteor","Aurora Veil":"wb-aurora","Eternal Eclipse":"wb-eclipse","Cosmic Tempest":"wb-tempest","Fog":"wb-fog","Black Hole":"wb-blackhole"};
      const cls=map[weather.name] || "wb-sunny";
      const bg=document.createElement("div"); bg.className="weather-bg "+cls;
      const particles=document.createElement("div"); particles.className="particles";

      if(cls==="wb-storm"){
        // Rain density tuned; randomized speed; cinematic bolts with glow/flicker
        for(let i=0;i<100;i++){
          const p=document.createElement("span"); p.className="rain-drop";
          p.style.left=Math.floor(Math.random()*100)+"%";
          p.style.top=(-60-Math.random()*160)+"px";
          p.style.animation=`rainFall ${0.7+Math.random()*0.6}s linear infinite`;
          p.style.opacity=(0.5+Math.random()*0.5).toFixed(2);
          particles.appendChild(p);
        }
        weatherEventTimers.push(setInterval(()=>{
          const bolt=document.createElement("div");
          bolt.className="bolt glow";
          const x = 10+Math.random()*80;
          bolt.style.left = x+"%";
          bolt.style.top = (-8)+"%";
          bolt.style.transform = `rotate(${(Math.random()*12-6).toFixed(1)}deg)`;
          bolt.style.transition="opacity 0.06s";
          particles.appendChild(bolt);
          requestAnimationFrame(()=>bolt.style.opacity="1");
          setTimeout(()=>bolt.style.opacity="0", 140);
          setTimeout(()=>bolt.remove(), 360);
        }, 2000 + Math.random()*2200));
      } else if(cls==="wb-blizzard"){
        // Layered flakes + occasional gust sweeps
        for(let i=0;i<120;i++){
          const s=document.createElement("span"); s.className="snow-flake";
          s.style.left=Math.floor(Math.random()*100)+"%";
          s.style.top=(-60-Math.random()*180)+"px";
          s.style.animation=`snowFall ${3.2+Math.random()*2.6}s linear infinite`;
          s.style.opacity=(0.6+Math.random()*0.4).toFixed(2);
          particles.appendChild(s);
        }
        weatherEventTimers.push(setInterval(()=>{
          const gust=document.createElement("div"); gust.className="blizzard-gust";
          particles.appendChild(gust);
          gust.style.transition="opacity 0.2s, transform 1.2s ease-out";
          requestAnimationFrame(()=>{ gust.style.opacity="1"; gust.style.transform="translateX(6%)"; });
          setTimeout(()=>{ gust.style.opacity="0"; }, 900);
          setTimeout(()=>gust.remove(), 1400);
        }, 4200 + Math.random()*2800));
      } else if(cls==="wb-fog"){
        const fog1=document.createElement("div"); fog1.className="fog-mist"; fog1.style.opacity="0.7";
        const fog2=document.createElement("div"); fog2.className="fog-mist"; fog2.style.opacity="0.45"; fog2.style.animationDuration="32s";
        particles.appendChild(fog1); particles.appendChild(fog2);
      } else if(cls==="wb-aurora"){
        const veil=document.createElement("div"); veil.className="aurora-veil"; particles.appendChild(veil);
        for(let i=0;i<3;i++){
          const r=document.createElement("div"); r.className="ribbon";
          r.style.left=(10+i*20)+"%";
          r.style.top=(12+i*16)+"%";
          r.style.animationDuration=(8+i*3)+"s";
          r.style.background=`linear-gradient(90deg, rgba(${120+i*10},${200-i*20},255,0.65), rgba(255,160,220,0.65))`;
          particles.appendChild(r);
        }
      } else if(cls==="wb-sunny"){
        const sun=document.createElement("div"); sun.className="sun-core"; particles.appendChild(sun);
        for(let i=0;i<10;i++){
          const ray=document.createElement("div"); ray.className="sun-ray";
          ray.style.transform=`translateX(-50%) rotate(${i*36}deg)`;
          ray.style.height=(140+Math.random()*60)+"px";
          ray.style.opacity=(0.4+Math.random()*0.25).toFixed(2);
          particles.appendChild(ray);
        }
      } else if(cls==="wb-tempest"){
        // Ambient shooting stars
        weatherEventTimers.push(setInterval(()=>{
          const s=document.createElement("div"); s.className="shooting-star";
          s.style.left=(Math.random()*40)+"%"; s.style.top=(10+Math.random()*30)+"%";
          particles.appendChild(s);
          s.style.transition="transform 1.8s linear, opacity 0.2s ease";
          setTimeout(()=>{ s.style.opacity="1"; s.style.transform=`translate(${160+Math.random()*140}px, ${220+Math.random()*120}px)`; }, 20);
          setTimeout(()=>{ s.style.opacity="0"; }, 1600);
          setTimeout(()=>s.remove(), 2000);
        }, 3800 + Math.random()*2200));
      } else if(cls==="wb-meteor"){
        // Meteors from top-right corner path, fewer but punchier
        weatherEventTimers.push(setInterval(()=>{
          const count = 3 + Math.floor(Math.random()*3);
          for(let i=0;i<count;i++){
            const m=document.createElement("div"); m.className="meteor";
            const startX = 85 + Math.random()*12; const startY = 6 + Math.random()*10;
            const dx = -(180 + Math.random()*160); const dy = 300 + Math.random()*180;
            const dur = 0.9 + Math.random()*0.6;
            m.style.left=startX+"%"; m.style.top=startY+"%";
            m.style.transition=`transform ${dur}s linear, opacity ${dur}s linear`;
            particles.appendChild(m);
            setTimeout(()=>{ m.style.transform=`translate(${dx}px, ${dy}px)`; m.style.opacity="0.0"; }, 20);
            setTimeout(()=>m.remove(), (dur*1000)+420);
          }
        }, 2600 + Math.random()*1800));
      } else if(cls==="wb-eclipse"){
        const corona=document.createElement("div"); corona.className="corona"; particles.appendChild(corona);
      } else if(cls==="wb-blackhole"){
        // Event horizon + accretion disk + gravitational lensing halo (like the image)
        const core=document.createElement("div"); core.className="bh-core";
        const disk=document.createElement("div"); disk.className="bh-disk";
        const lens=document.createElement("div"); lens.className="bh-lens";
        particles.appendChild(lens); particles.appendChild(disk); particles.appendChild(core);
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
        else if(e.type==="weather" && e.meta){
          totals.weatherBiasItem=e.meta.biasItem||null;
          totals.luck += (e.meta.luck||0);
          totals.speed += (e.meta.speed||0);
        }
      }
      state.activeEffects=totals;
    }
    function addEffect(effect){
      if(effect.type==="guarantee" && effect.name==="Origin Crystal"){
        state.persistentGuarantees.push({ name:effect.name, type:"guarantee", rarity:effect.rarity || "omniversal", grantPool:["eternal","omniversal"] });
        saveState(); spawnBanner(`Stored: ${effect.name}`,"activate","b-omniversal"); renderActiveEffects(); return;
      }
      const now=Date.now(), durMs=(effect.duration||0)*1000, capMs=MAX_EFFECT_SECONDS*1000;
      const keyMatch=e=>e.name===effect.name && (effect.type==="weather"?e.type==="weather":e.type===effect.type) && (e.target||null)===(effect.target||null);
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
      state.persistentGuarantees.forEach(g=>{ const colorClass=TIERS.find(t=>t.key===g.rarity)?.colorClass || ""; const div=document.createElement("div"); div.className=`effect-entry ${colorClass}`; div.textContent=`${g.name}: ready`; el.appendChild(div); });
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
      if(state.auto && !state.cutsceneActive){
        const mult=1+(state.activeEffects.speed||0);
        const interval=Math.max(60,Math.round(BASE_AUTO_INTERVAL/mult));
        state.autoInterval=setInterval(()=>{ const btn=document.getElementById("btnAuto"); if(btn.disabled){ toggleAuto(); return; } rollOnce(); },interval);
      }
    }
    function toggleAuto(){
      const btn=document.getElementById("btnAuto"); if(btn.disabled) return;
      if(state.auto){ state.auto=false; clearInterval(state.autoInterval); state.autoInterval=null; btn.textContent="Auto Roll: Off"; }
      else { state.auto=true; btn.textContent="Auto Roll: On"; updateAutoInterval(); }
      saveState();
    }

    /* ---------------- Weather schedule ---------------- */
    function triggerRandomWeather(){
      const r=Math.random(); let pool=WEATHERS.normal;
      if(r>=0.70 && r<0.95) pool=WEATHERS.rare; else if(r>=0.95) pool=WEATHERS.super;
      const w=pool[Math.floor(Math.random()*pool.length)];
      const dur=100+Math.floor(Math.random()*201);
      const meta={ luck:(w.effects?.luck)||0, speed:(w.effects?.speed)||0, biasItem:(w.effects?.biasItem)||null };
      addEffect({ name:w.name, type:"weather", duration:dur, rarity: classToTierKey(w.colorClass), meta });
      const icon = w.name==="Eternal Eclipse" ? "icon-eclipse" : w.name==="Storm" ? "icon-storm" : w.name==="Aurora Veil" ? "icon-aurora" : w.name==="Cosmic Tempest" ? "icon-tempest" : w.name==="Black Hole" ? "icon-bh" : "";
      spawnBanner(`${w.name} started`,"weather",w.colorClass,icon);
    }
    function scheduleNextWeather(){ const delayMs=(240+Math.random()*480)*1000; setTimeout(()=>{ triggerRandomWeather(); scheduleNextWeather(); },delayMs); }
    function classToTierKey(colorClass){ const t=TIERS.find(x=>x.colorClass===colorClass); return t?t.key:"common"; }

    /* ---------------- Cinematic cutscenes per REAL Index item (inline, no name overlay) ---------------- */

    function variantForName(name, max=10){
      let h=0; for(let i=0;i<name.length;i++){ h=(h*31 + name.charCodeAt(i))>>>0; }
      return h % max;
    }

    function playCutsceneInline(tierKey,itemName){
      const stage=document.getElementById("cinemaStageInline");
      const overlay=document.createElement("div");
      const bgTint={
        divine:"radial-gradient(circle at 50% 50%, rgba(255,215,120,0.18), rgba(0,0,0,0))",
        celestial:"linear-gradient(120deg, rgba(120,255,255,0.18), rgba(0,0,0,0))",
        transcendent:"linear-gradient(120deg, rgba(160,140,255,0.22), rgba(0,0,0,0))",
        eternal:"radial-gradient(circle at 50% 50%, rgba(140,255,200,0.18), rgba(0,0,0,0))",
        omniversal:"conic-gradient(from 0deg, rgba(255,160,220,0.18), rgba(120,255,220,0.18), rgba(180,120,255,0.18), rgba(255,160,220,0.18))"
      };
      stage.innerHTML="";
      overlay.style.position="absolute"; overlay.style.inset="0";
      overlay.style.background=bgTint[tierKey] || "transparent";
      overlay.style.opacity="0.0"; overlay.style.transition="opacity .35s ease-out";
      stage.appendChild(overlay);

      for(let i=0;i<90;i++){
        const s=document.createElement("div");
        s.className="star";
        s.style.left=Math.random()*100+"%";
        s.style.top=Math.random()*100+"%";
        s.style.width=s.style.height=(Math.random()*2+1)+"px";
        s.style.animation=`twinkle ${1.8+Math.random()*1.6}s ease-in-out infinite`;
        stage.appendChild(s);
      }

      const v=variantForName(itemName,10);
      if(v===0) styleGalacticSpiral(stage,tierKey);
      else if(v===1) styleCrownRings(stage,tierKey);
      else if(v===2) styleAuroraWeave(stage,tierKey);
      else if(v===3) styleShardBurst(stage,tierKey);
      else if(v===4) styleBeamConvergence(stage,tierKey);
      else if(v===5) styleVortexCollapse(stage,tierKey);
      else if(v===6) styleSigilEngrave(stage,tierKey);
      else if(v===7) styleGridAxis(stage,tierKey);
      else if(v===8) stylePrismWave(stage,tierKey);
      else styleHeartPulse(stage,tierKey);

      overlay.style.opacity="1";
      setTimeout(()=>boom(stage,tierKey),4200);

      // Pause auto-roll and normal roll during cutscene
      state.cutsceneActive=true;
      const rollBtn=document.getElementById("btnRoll");
      rollBtn.disabled=true;
      const autoBtn=document.getElementById("btnAuto");
      const wasAuto=state.auto;
      if(wasAuto){ state.cutscenePausedAuto=true; toggleAuto(); }
      setTimeout(()=>{
        overlay.style.opacity="0";
        setTimeout(()=>{
          stage.innerHTML="";
          state.cutsceneActive=false;
          rollBtn.disabled=false;
          if(state.cutscenePausedAuto){ state.cutscenePausedAuto=false; if(state.rolls>=50){ autoBtn.disabled=false; } toggleAuto(); }
        }, 350);
      }, 6500);
    }

    function boom(stage, tierKey){
      const tint={
        divine:"rgba(255,215,120,0.75)",
        celestial:"rgba(120,255,255,0.75)",
        transcendent:"rgba(160,140,255,0.75)",
        eternal:"rgba(140,255,200,0.75)",
        omniversal:"rgba(255,160,220,0.75)"
      }[tierKey] || "rgba(255,255,255,0.75)";
      for(let i=0;i<14;i++){
        const ring=document.createElement("div");
        ring.className="ring";
        ring.style.left="50%"; ring.style.top="50%";
        ring.style.borderColor=tint;
        ring.style.width=80+i*34+"px"; ring.style.height=80+i*34+"px";
        ring.style.animation=`pulseRing ${1.6+i*0.12}s ease-out forwards`;
        stage.appendChild(ring);
      }
      for(let i=0;i<48;i++){
        const shard=document.createElement("div");
        shard.className="shard";
        shard.style.left="50%"; shard.style.top="50%";
        shard.style.transform=`translate(-50%,-50%) rotate(${Math.random()*360}deg)`;
        shard.style.animation="explodeShard 1.6s ease-out forwards";
        stage.appendChild(shard);
      }
      injectKeyframesOnce("explodeShard","0%{transform:translate(-50%,-50%) scale(0.4) rotate(0);opacity:.95}100%{transform:translate(-50%,-50%) scale(2.2) rotate(240deg);opacity:.0}");
    }

    /* Style families */
    function styleGalacticSpiral(stage,tier){
      for(let i=0;i<16;i++){
        const r=document.createElement("div"); r.className="ring";
        r.style.left="50%"; r.style.top="50%";
        r.style.width=40+i*26+"px"; r.style.height=40+i*26+"px";
        r.style.animation=`spiral ${1.2+i*0.12}s ease-in-out infinite`;
        stage.appendChild(r);
      }
      injectKeyframesOnce("spiral","0%{transform:translate(-50%,-50%) rotate(0) scale(0.8);opacity:.5}50%{opacity:1}100%{transform:translate(-50%,-50%) rotate(180deg) scale(1.3);opacity:.2}");
    }
    function styleCrownRings(stage,tier){
      for(let i=0;i<6;i++){
        const ring=document.createElement("div"); ring.className="ring";
        ring.style.left="50%"; ring.style.top="50%"; ring.style.borderColor="rgba(255,200,120,"+(0.35-0.04*i)+")";
        ring.style.width=120+i*36+"px"; ring.style.height=120+i*36+"px";
        ring.style.animation=`crown ${2+i*0.2}s ease-out infinite`; stage.appendChild(ring);
      }
      injectKeyframesOnce("crown","0%{transform:translate(-50%,-50%) scale(0.6);opacity:.3}50%{opacity:1}100%{transform:translate(-50%,-50%) scale(1.5);opacity:.0}");
    }
    function styleAuroraWeave(stage,tier){
      for(let i=0;i<4;i++){
        const ribbon=document.createElement("div"); ribbon.className="ribbon";
        ribbon.style.left="12%"; ribbon.style.top=(14+i*14)+"%";
        ribbon.style.animationDuration = (7+i*2) + "s";
        ribbon.style.background="linear-gradient(90deg, rgba(120,255,255,0.7), rgba(255,160,220,0.7))";
        stage.appendChild(ribbon);
      }
      const core=document.createElement("div");
      core.className="corona";
      stage.appendChild(core);
    }
    function styleShardBurst(stage,tier){
      for(let i=0;i<120;i++){
        const s=document.createElement("div"); s.className="shard";
        s.style.left=(45+Math.random()*10)+"%"; s.style.top=(45+Math.random()*10)+"%";
        s.style.transform=`rotate(${Math.random()*360}deg)`;
        s.style.animation="shardDance 2.2s ease-in-out infinite";
        stage.appendChild(s);
      }
      injectKeyframesOnce("shardDance","0%{transform:rotate(0) translateY(0);opacity:.3}50%{transform:rotate(180deg) translateY(8px);opacity:1}100%{transform:rotate(360deg) translateY(0);opacity:.3}");
    }
    function styleBeamConvergence(stage,tier){
      for(let i=0;i<8;i++){
        const b=document.createElement("div"); b.className="beam";
        b.style.width="4px"; b.style.height="280px";
        b.style.left=(10+i*12)+"%"; b.style.top="0%";
        b.style.animation=`beamDown ${1.6+i*0.1}s linear infinite`;
        stage.appendChild(b);
      }
      injectKeyframesOnce("beamDown","0%{transform:translateY(-120px);opacity:.0}50%{opacity:1}100%{transform:translateY(240px);opacity:.0}");
    }
    function styleVortexCollapse(stage,tier){
      for(let i=0;i<18;i++){
        const r=document.createElement("div"); r.className="ring";
        r.style.left="50%"; r.style.top="50%";
        r.style.width=60+i*22+"px"; r.style.height=60+i*22+"px";
        r.style.animation=`vortex ${1.1+i*0.12}s ease-in-out infinite`;
        stage.appendChild(r);
      }
      injectKeyframesOnce("vortex","0%{transform:translate(-50%,-50%) scale(1) rotate(0);opacity:.5}50%{transform:translate(-50%,-50%) scale(0.6) rotate(90deg);opacity:.9}100%{transform:translate(-50%,-50%) scale(1.2) rotate(180deg);opacity:.2}");
    }
    function styleSigilEngrave(stage,tier){
      const sig=document.createElement("div"); sig.className="ring";
      sig.style.left="50%"; sig.style.top="50%"; sig.style.width="260px"; sig.style.height="260px";
      sig.style.border="4px double rgba(255,255,255,0.35)";
      sig.style.animation="sigPulse 3s ease-in-out infinite";
      stage.appendChild(sig);
      injectKeyframesOnce("sigPulse","0%{transform:translate(-50%,-50%) scale(1);opacity:.7}50%{transform:translate(-50%,-50%) scale(1.08);opacity:1}100%{transform:translate(-50%,-50%) scale(1);opacity:.7}");
    }
    function styleGridAxis(stage,tier){
      for(let i=0;i<12;i++){
        const line=document.createElement("div");
        line.style.position="absolute"; line.style.left=i*8+"%"; line.style.top="0";
        line.style.width="1px"; line.style.height="100%"; line.style.background="rgba(255,255,255,0.2)";
        line.style.animation="axisSpin 6s linear infinite";
        stage.appendChild(line);
      }
      injectKeyframesOnce("axisSpin","0%{transform:rotate(0)}100%{transform:rotate(360deg)}");
    }
    function stylePrismWave(stage,tier){
      for(let i=0;i<6;i++){
        const cover=document.createElement("div");
        cover.style.position="absolute"; cover.style.left="0"; cover.style.top="0"; cover.style.right="0"; cover.style.bottom="0";
        cover.style.background=`linear-gradient(120deg, rgba(255,160,220,${0.06+i*0.06}), rgba(120,255,220,${0.06+i*0.06}), rgba(180,120,255,${0.06+i*0.06}))`;
        cover.style.filter="blur(6px)";
        cover.style.animation=`sweep ${2.6+i*0.2}s ease-in-out infinite`;
        stage.appendChild(cover);
      }
      injectKeyframesOnce("sweep","0%{transform:translateX(-6%)}50%{transform:translateX(6%)}100%{transform:translateX(-6%)}");
    }
    function styleHeartPulse(stage,tier){
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
    function consumeGuaranteeIfAny(){
      if(!state.persistentGuarantees.length) return null;
      const idx=state.persistentGuarantees.findIndex(g=>g.name==="Origin Crystal");
      if(idx>=0){ const g=state.persistentGuarantees[idx]; state.persistentGuarantees.splice(idx,1); saveState(); spawnBanner(`Guarantee consumed: ${g.name}`,"announce","b-omniversal"); return g; }
      return null;
    }

    function rollOnce(){
      if(state.cutsceneActive) return; // disable manual while cutscene
      const upcoming=state.rolls+1; const surge=milestoneLuckMultiplier(upcoming);
      if(surge>1){ spawnBanner(`Luck Surge x${surge} (1 roll)`,"luck", surge===10?"b-omniversal":"b-divine"); }

      let tiers=TIERS.filter(t=>t.key!=="exclusive");
      tiers=applyWeightModifiers(tiers,surge);

      const now=Date.now(); const wEff=state.effectInstances.find(e=>e.type==="weather" && e.expiresAt>now);
      const exclusiveActive=wEff && (wEff.name==="Eternal Eclipse" || wEff.name==="Cosmic Tempest");

      const forced=(state.cmdQueue && state.cmdQueue.nextRoll && !state.cmdQueue.nextRoll.applied) ? state.cmdQueue.nextRoll : null;
      const guarantee=consumeGuaranteeIfAny();

      let tierKey,tierName, guaranteedItemName=null;
      if(guarantee){
        const poolTiers = guarantee.grantPool || ["eternal","omniversal"];
        const chosenTier = poolTiers[Math.floor(Math.random()*poolTiers.length)];
        const items = INDEX_ITEMS[chosenTier] || [];
        const pick = items.length ? items[Math.floor(Math.random()*items.length)] : null;
        tierKey = chosenTier; tierName = TIERS.find(t=>t.key===tierKey)?.name || "Unknown";
        guaranteedItemName = pick;
      } else if(forced && forced.rarity){
        tierKey=forced.rarity; tierName=TIERS.find(t=>t.key===tierKey)?.name || forced.rarity; state.cmdQueue.nextRoll.applied=true; saveState();
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
      const rollItem=(!forced?.name && !guaranteedItemName) && Math.random()<(baseItemChance+luckBoost);

      state.rolls++;

      let isNew=false, displayName=null, displayRarityClass=null;

      function tryWeatherItemBias(defaultTierKey){
        const bias=state.activeEffects.weatherBiasItem; if(!bias) return null;
        const items=INDEX_ITEMS[defaultTierKey]||[]; if(!items.includes(bias.name)) return null;
        const pass=Math.random()<(bias.boost||0); return pass?bias.name:null;
      }

      if(guaranteedItemName){
        displayName=guaranteedItemName;
        displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
        processIndexItem(tierKey,tierName,guaranteedItemName,surge);
        isNew=markNew(tierKey,guaranteedItemName);
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
          const itemName=null;
          displayName=forced?.name || itemName || "Exclusive"; displayRarityClass="b-exclusive";
          processIndexItem("exclusive","Exclusive",displayName==="Exclusive"?null:displayName,surge);
          if(displayName && displayName!=="Exclusive") isNew=markNew("exclusive",displayName);
        } else {
          const chosenName=forced?.name || tryWeatherItemBias(tierKey) || pickIndexItem(tierKey);
          displayName=chosenName || tierName; displayRarityClass=TIERS.find(t=>t.key===tierKey)?.colorClass || "";
          processIndexItem(tierKey,tierName,chosenName,surge); if(chosenName) isNew=markNew(tierKey,chosenName);
        }
      }

      saveState();
      showGlow();
      renderResult(displayName,tierKey,displayRarityClass);
      if(isNew) spawnBanner(`NEW collected: [${tierName}] ${displayName}`,"new",displayRarityClass);

      const highOrder=["divine","celestial","transcendent","eternal","omniversal"];
      if(highOrder.includes(tierKey) && displayName && INDEX_ITEMS[tierKey]?.includes(displayName)){
        playCutsceneInline(tierKey, displayName);
      }

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

    function renderResult(name,tierKey,rarityClass){
      elResult.textContent=name || "Unknown";
      elRarity.textContent=tierKey ? tierKey.toUpperCase() : "";
      elRarity.className="rarity badge "+(rarityClass||"");
      renderWeatherBackdrop();
    }
    function renderButtonsState(){
      const elAutoBtn=document.getElementById("btnAuto");
      if(state.rolls>=50){
        elAutoBtn.disabled=false;
        elAutoBtn.textContent = state.auto ? "Auto Roll: On" : "Auto Roll: Off";
      } else {
        elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)";
      }
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
          const name="Dark Matter Shard"; const li=document.createElement("li"); li.className="index-item";
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

    /* -------- Improved deletion confirmation -------- */
    let pendingDelete = null;
    function openDeleteConfirm(entry, tier){
      const modal=document.getElementById("deleteConfirm");
      const badge=document.getElementById("confirmBadge");
      const text=document.getElementById("confirmText");
      const badgeClass=tier?.colorClass || "";
      const dmClass = (entry.name==="Dark Matter Shard") ? "b-exclusive-dark" : badgeClass;
      badge.className = `confirm-badge ${dmClass}`;
      badge.textContent = tier?.name || "Item";
      text.textContent = `Are you sure you want to delete "${entry.name || '(Unknown)'}"? This action cannot be undone.`;
      pendingDelete = entry;
      modal.style.display="block";
    }
    function closeDeleteConfirm(){ const modal=document.getElementById("deleteConfirm"); modal.style.display="none"; pendingDelete=null; }

    document.getElementById("confirmCancel").addEventListener("click", closeDeleteConfirm);
    document.getElementById("confirmDeleteBtn").addEventListener("click", ()=>{
      if(!pendingDelete) return;
      if(pendingDelete._type==="rolled"){
        const idx=state.inventoryRolled.findIndex(i=>i.roll===pendingDelete.roll && i.name===pendingDelete.name);
        if(idx>=0){ state.inventoryRolled.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedRolled=false; }
      } else if(pendingDelete._type==="item"){
        const idx=state.inventoryItems.findIndex(i=>i.roll===pendingDelete.roll && i.name===pendingDelete.name);
        if(idx>=0){ state.inventoryItems.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedItems=false; }
      }
      closeDeleteConfirm();
    });

    function renderInventory(){
      elInventoryList.innerHTML="";
      if(state.mode==="Rolled"){
        const items=[...state.inventoryRolled].reverse();
        items.forEach(entry=>{
          const tier=TIERS.find(t=>t.key===entry.tier); let badgeClass=tier?tier.colorClass:"";
          if(entry.tier==="exclusive" && entry.name==="Dark Matter Shard"){ badgeClass="b-exclusive-dark"; }
          const li=document.createElement("li");
          const left=document.createElement("div");
          left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"} • #${entry.roll}${entry.milestone>1?` • ${entry.milestone}x`:``}`;
          const right=document.createElement("div"); right.className="inv-actions";
          const del=document.createElement("button"); del.textContent="Delete";
          del.addEventListener("click",()=>{
            const order=TIERS.map(t=>t.key);
            const isVeryRare = order.indexOf(entry.tier) >= order.indexOf("legendary");
            if(isVeryRare){
              const enriched = {...entry, _type:"rolled"}; openDeleteConfirm(enriched, tier);
            } else {
              const idx=state.inventoryRolled.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
              if(idx>=0){ state.inventoryRolled.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedRolled=false; }
            }
          });
          right.appendChild(del); li.appendChild(left); li.appendChild(right); elInventoryList.appendChild(li);
        });
        const cap=document.createElement("div"); cap.className="inv-stats"; cap.style.marginTop="8px"; cap.textContent=`Rolled capacity ${state.inventoryRolled.length}/${ROLLED_MAX}`; elInventoryList.appendChild(cap);
      } else {
        const items=[...state.inventoryItems].reverse();
        items.forEach(entry=>{
          const tier=TIERS.find(t=>t.key===entry.tier); let badgeClass=tier?tier.colorClass:"";
          if(entry.tier==="exclusive" && entry.name==="Dark Matter Shard"){ badgeClass="b-exclusive-dark"; }
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
          const del=document.createElement("button"); del.textContent="Delete";
          del.addEventListener("click",()=>{
            const order=TIERS.map(t=>t.key);
            const isVeryRare = order.indexOf(entry.tier) >= order.indexOf("legendary");
            if(isVeryRare){
              const enriched = {...entry, _type:"item"}; openDeleteConfirm(enriched, tier);
            } else {
              const idx=state.inventoryItems.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
              if(idx>=0){ state.inventoryItems.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedItems=false; }
            }
          });
          right.appendChild(use); right.appendChild(del); li.appendChild(left); li.appendChild(right); elInventoryList.appendChild(li);
        });
        const cap=document.createElement("div"); cap.className="inv-stats"; cap.style.marginTop="8px"; cap.textContent=`Items capacity ${state.inventoryItems.length}/${ITEMS_MAX}`; elInventoryList.appendChild(cap);
      }
    }

    function deleteRolledEntry(entry){
      const idx=state.inventoryRolled.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){ state.inventoryRolled.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedRolled=false; }
    }
    function deleteItemEntry(entry){
      const idx=state.inventoryItems.findIndex(i=>i.roll===entry.roll && i.name===entry.name);
      if(idx>=0){ state.inventoryItems.splice(idx,1); saveState(); renderInventory(); state.fullAnnouncedItems=false; }
    }
    function useItemEntry(entry){
      if(entry.effect){
        const eff=entry.effect;
        if(eff.type==="totem"){
          const metaSrc=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].find(w=>w.name===eff.weather);
          const dur=100+Math.floor(Math.random()*201);
          const meta={ luck:(metaSrc?.effects?.luck)||0, speed:(metaSrc?.effects?.speed)||0, biasItem:(metaSrc?.effects?.biasItem)||null };
          addEffect({ name:eff.weather, type:"weather", duration:dur, rarity: classToTierKey(colorClassForWeather(eff.weather)), meta });
          const cl=colorClassForWeather(eff.weather);
          const icon=eff.weather==="Eternal Eclipse" ? "icon-eclipse" : eff.weather==="Storm" ? "icon-storm" : eff.weather==="Aurora Veil" ? "icon-aurora" : eff.weather==="Cosmic Tempest" ? "icon-tempest" : eff.weather==="Black Hole" ? "icon-bh" : "";
          spawnBanner(`Activated ${entry.name}`,"activate",cl); spawnBanner(`${eff.weather} started`,"weather",cl,icon);
        } else if(eff.type==="totem_random"){
          const r=Math.random(); let pool=WEATHERS.normal; if(r>=0.70 && r<0.95) pool=WEATHERS.rare; else if(r>=0.95) pool=WEATHERS.super;
          const w=pool[Math.floor(Math.random()*pool.length)]; const dur=100+Math.floor(Math.random()*201);
          const meta={ luck:(w.effects?.luck)||0, speed:(w.effects?.speed)||0, biasItem:(w.effects?.biasItem)||null };
          addEffect({ name:w.name, type:"weather", duration:dur, rarity: classToTierKey(w.colorClass), meta });
          const icon=w.name==="Eternal Eclipse" ? "icon-eclipse" : w.name==="Storm" ? "icon-storm" : w.name==="Aurora Veil" ? "icon-aurora" : w.name==="Cosmic Tempest" ? "icon-tempest" : w.name==="Black Hole" ? "icon-bh" : "";
          spawnBanner(`Activated ${entry.name}`,"activate",w.colorClass); spawnBanner(`${w.name} started`,"weather",w.colorClass,icon);
        } else if(eff.type==="guarantee" && eff.name==="Origin Crystal"){
          addEffect(eff);
        } else {
          addEffect({ name:eff.name, type:eff.type, amount:eff.amount, target:eff.target, duration:eff.duration, rarity:eff.rarity });
          const cl=TIERS.find(t=>t.key===eff.rarity)?.colorClass || ""; spawnBanner(`Activated ${entry.name}`,"activate",cl);
        }
      }
      deleteItemEntry(entry); renderActiveEffects();
    }

    /* ---------------- Secret & Commands ---------------- */
    const secretBtn=document.getElementById("secretBtn");
    const passwordModal=document.getElementById("passwordModal");
    const secretInput=document.getElementById("secretInput");
    const secretConfirm=document.getElementById("secretConfirm");

    function injectCommandsButton(){
      const controls=document.getElementById("controls");
      let existing=document.getElementById("btnCommands"); if(existing) return;
      const btn=document.createElement("button"); btn.id="btnCommands"; btn.textContent="Commands"; btn.style.marginLeft="auto";
      btn.addEventListener("click",()=>{ const panel=document.getElementById("commandsPanel"); const vis=panel.style.display!=="none"; panel.style.display=vis?"none":"block"; });
      controls.appendChild(btn);
    }
    secretBtn.addEventListener("click",()=>{ if(state.secretUnlocked) return; passwordModal.style.display="block"; });
    secretConfirm.addEventListener("click",()=>{
      const val=secretInput.value||"";
      if(val==="Orange_Toaster"){ state.secretUnlocked=true; saveState(); passwordModal.style.display="none"; secretBtn.remove(); injectCommandsButton(); }
      else { passwordModal.style.display="none"; secretInput.value=""; }
    });

    const cmdWeatherSel=document.getElementById("cmdWeather");
    const cmdNextRaritySel=document.getElementById("cmdNextRarity");
    const cmdNextNameSel=document.getElementById("cmdNextName");
    const cmdGiveRaritySel=document.getElementById("cmdGiveRarity");
    const cmdGiveItemSel=document.getElementById("cmdGiveItem");

    function populateCommandsSelectors(){
      const weatherNames=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].map(w=>w.name);
      cmdWeatherSel.innerHTML=weatherNames.map(n=>`<option>${n}</option>`).join("");
      cmdNextRaritySel.innerHTML=TIERS.map(t=>`<option value="${t.key}">${t.name}</option>`).join("");
      cmdGiveRaritySel.innerHTML=TIERS.map(t=>`<option value="${t.key}">${t.name}</option>`).join("");

      function refreshNamesForRarity(selKey,targetSel){ const items=INDEX_ITEMS[selKey]||[]; targetSel.innerHTML=items.length?items.map(n=>`<option>${n}</option>`).join(""):`<option>(none)</option>`; }
      refreshNamesForRarity(cmdNextRaritySel.value || "common", cmdNextNameSel);
      cmdNextRaritySel.addEventListener("change",()=>refreshNamesForRarity(cmdNextRaritySel.value, cmdNextNameSel));

      function refreshGiveItemsForRarity(key){ const items=(ITEM_DROPS[key]||[]); cmdGiveItemSel.innerHTML=items.length?items.map(d=>`<option value="${encodeURIComponent(JSON.stringify(d))}">${d.name}</option>`).join(""):`<option>(none)</option>`; }
      refreshGiveItemsForRarity(cmdGiveRaritySel.value || "common");
      cmdGiveRaritySel.addEventListener("change",()=>refreshGiveItemsForRarity(cmdGiveRaritySel.value));
    }

    const cmdLuck=document.getElementById("cmdLuck");
    const cmdSpeed=document.getElementById("cmdSpeed");
    const cmdLuckScope=document.getElementById("cmdLuckScope");
    const cmdWeatherScope=document.getElementById("cmdWeatherScope");
    const cmdNextScope=document.getElementById("cmdNextScope");
    const cmdReset=document.getElementById("cmdReset");
    const cmdResetScope=document.getElementById("cmdResetScope");
    const cmdClear=document.getElementById("cmdClear");
    const cmdClearScope=document.getElementById("cmdClearScope");
    const cmdConfirm=document.getElementById("cmdConfirm");
    const cmdGiveScope=document.getElementById("cmdGiveScope");
    const cmdPresetBoost=document.getElementById("cmdPresetBoost");
    const cmdPresetEclipse=document.getElementById("cmdPresetEclipse");

    cmdReset.addEventListener("click",()=>{
      const scope=cmdResetScope.value;
      const ok=confirm("Are you sure you want to reset all data? This cannot be undone.");
      if(!ok) return;
      state.cmdQueue={ reset:{ scope } };
      saveState(); applyCmdQueueIfAny();
    });
    cmdClear.addEventListener("click",()=>{
      const scope=cmdClearScope.value;
      state.cmdQueue={ clearEffects:{ scope } };
      saveState(); applyCmdQueueIfAny();
    });
    cmdPresetBoost.addEventListener("click",()=>{
      cmdLuck.value="1.0"; // +100%
      cmdSpeed.value="0.5"; // +50%
      cmdWeather.value="Sunny Radiance";
      spawnBanner("Preset loaded: Lucky Burst","announce","b-epic");
    });
    cmdPresetEclipse.addEventListener("click",()=>{
      cmdLuck.value="0.8";
      cmdSpeed.value="0.3";
      cmdWeather.value="Eternal Eclipse";
      spawnBanner("Preset loaded: Eclipse","announce","b-divine");
    });

    cmdConfirm.addEventListener("click",()=>{
      const q={};
      const luckVal=cmdLuck.value.trim(); if(luckVal!==""){ const num=Number(luckVal); if(!Number.isNaN(num)) q.setLuck={ amount:num, scope:cmdLuckScope.value }; }
      const speedVal=cmdSpeed.value.trim(); if(speedVal!==""){ const num=Number(speedVal); if(!Number.isNaN(num)) q.setSpeed={ amount:num, scope:cmdLuckScope.value }; }
      const wName=cmdWeatherSel.value; if(wName) q.setWeather={ name:wName, scope:cmdWeatherScope.value };
      const rKey=cmdNextRaritySel.value; const rName=cmdNextNameSel.value; if(rKey) q.nextRoll={ rarity:rKey, name:rName && rName!=="(none)" ? rName : null, scope:cmdNextScope.value };
      const effStr=cmdGiveItemSel.value; if(effStr && effStr!=="(none)"){ const eff=JSON.parse(decodeURIComponent(effStr)); q.giveEffect={ effect:eff, scope:cmdGiveScope.value }; }
      state.cmdQueue=q; saveState(); applyCmdQueueIfAny(); spawnBanner("Commands applied","announce","b-epic");
    });

    function applyCmdQueueIfAny(){
      if(!state.cmdQueue) return;
      const q=state.cmdQueue;
      if(q.setLuck){ const amount=Number(q.setLuck.amount)||0; addEffect({ name:`Command Luck +${Math.round(amount*100)}%`, type:"luck", amount, duration:30, rarity:"divine" }); }
      if(q.setSpeed){ const amount=Number(q.setSpeed.amount)||0; addEffect({ name:`Command Speed +${Math.round(amount*100)}%`, type:"speed", amount, duration:30, rarity:"divine" }); }
      if(q.setWeather){
        const wName=q.setWeather.name; const metaSrc=[...WEATHERS.normal,...WEATHERS.rare,...WEATHERS.super].find(w=>w.name===wName);
        const dur=150; const meta={ luck:(metaSrc?.effects?.luck)||0, speed:(metaSrc?.effects?.speed)||0, biasItem:(metaSrc?.effects?.biasItem)||null };
        addEffect({ name:wName, type:"weather", duration:dur, rarity: classToTierKey(metaSrc?.colorClass||""), meta });
        const icon=wName==="Eternal Eclipse" ? "icon-eclipse" : wName==="Storm" ? "icon-storm" : wName==="Aurora Veil" ? "icon-aurora" : wName==="Cosmic Tempest" ? "icon-tempest" : wName==="Black Hole" ? "icon-bh" : "";
        spawnBanner(`${wName} started (Command)`,"weather",metaSrc?.colorClass||"",icon);
      }
      if(q.reset){
        if(q.reset.scope==="GLOBAL"){ localStorage.clear(); location.reload(); }
        else { state.inventoryRolled=[]; state.inventoryItems=[]; state.effectInstances=[]; state.persistentGuarantees=[]; deriveEffectsTotals(); saveState(); renderActiveEffects(); renderInventory(); renderIndex(); spawnBanner("Local data cleared","announce","b-trash"); }
      }
      if(q.clearEffects){ state.effectInstances=[]; state.persistentGuarantees=[]; deriveEffectsTotals(); saveState(); renderActiveEffects(); renderWeatherBackdrop(); spawnBanner("All effects cleared","announce","b-common"); }
      if(q.giveEffect){ const eff=q.giveEffect.effect; if(eff){ addEffect(eff); const colorClass=TIERS.find(t=>t.key===eff.rarity)?.colorClass || ""; spawnBanner(`Activated ${eff.name}`,"activate",colorClass); } }
      if(q.nextRoll){ state.cmdQueue={ nextRoll:q.nextRoll }; } else { state.cmdQueue=null; }
      saveState();
    }

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
    document.getElementById("btnAuto").addEventListener("click",toggleAuto);
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
    populateCommandsSelectors();
    if(state.secretUnlocked){ injectCommandsButton(); document.getElementById("secretBtn")?.remove(); document.getElementById("passwordModal").style.display="none"; }
    const elAutoBtn=document.getElementById("btnAuto");
    if(state.auto && state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On"; updateAutoInterval(); }
    else { elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)"; }
    scheduleNextWeather();
  </script>
</body>
</html>
