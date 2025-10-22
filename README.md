<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Sol’s RNG — Expanded with Inventory, Index, Auto-Sell</title>
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
    .banner.new  { top:120px; color:var(--accent); font-size:16px; } /* directly under "Ready to roll" */
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

    /* Auto-sell chooser anchored to right corner of inventory; scrollable menu */
    .autosell-wrap{position:relative;}
    .chooser-menu{
      position:absolute; right:0; top:110%; background:#0f1320; border:1px solid #2a3449; border-radius:8px;
      box-shadow:0 12px 24px rgba(0,0,0,0.35); min-width:200px; max-height:220px; overflow-y:auto; display:none; z-index:10;
    }
    .chooser-menu button{width:100%;text-align:left;background:transparent;border:none;padding:10px 12px;color:var(--text);}
    .chooser-menu button:hover{background:#1b2232;}

    /* Rarity colors */
    .b-worthless{background:#1a1a1a;color:#b3b3b3;} .b-trash{background:#211616;color:#d28f8f;}
    .b-common{background:#1f2635;color:#c7d1e5;} .b-uncommon{background:#14261d;color:#7ae08f;}
    .b-rare{background:#221a35;color:#caa6ff;} .b-epic{background:#10233a;color:#7bb7ff;}
    .b-legendary{background:#2a1f12;color:#ffbf66;} .b-mythic{background:#2a142a;color:#ff7ee6;}
    .b-divine{background:#2a2a12;color:#ffd700;} .b-celestial{background:#142a2a;color:#7affff;}
    .b-transcendent{background:#1a1a2f;color:#a0a7ff;} .b-eternal{background:#1a2f2a;color:#9cf2c7;}
    .b-omniversal{background:#2f1a2f;color:#ff9cff;}

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
          <!-- Banners are now ephemeral elements appended per-event; no single IDs to be overwritten -->
          <div class="result" id="resultText">Ready to roll</div>
          <div class="rarity" id="rarityText"></div>
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
          <div class="inv-title">
            <h3 style="margin:0;">Inventory</h3>
            <span class="inv-stats" id="inventoryStats"></span>
          </div>
          <div class="autosell-wrap">
            <button id="btnAutoSell">Auto-Sell: Off</button>
            <div class="chooser-menu" id="autoSellMenu"></div>
          </div>
          <span class="inv-warning" id="invWarning" style="display:none;"></span>
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
    };
    const LUCK_TARGET_KEYS=["rare","epic","legendary","mythic","divine","celestial","transcendent","eternal","omniversal"];
    const INVENTORY_MAX=10;

    /* ---------------- Persistence ---------------- */
    const STORAGE_KEYS={ rolls:"sol_rng_rolls", unlocks:"sol_rng_unlocks", auto:"sol_rng_auto", inv:"sol_rng_inventory", autoSell:"sol_rng_autosell" };
    function buildInitialUnlocks(indexDef){ const out={}; for(const k in indexDef){ out[k]={}; indexDef[k].forEach(n=>out[k][n]=false); } return out; }
    function reviveUnlocks(unlocks){
      const fresh=buildInitialUnlocks(INDEX_ITEMS);
      for(const k in fresh){ for(const n of Object.keys(fresh[k])){ fresh[k][n]=unlocks[k]&&typeof unlocks[k][n]==="boolean"?unlocks[k][n]:false; } }
      return fresh;
    }
    function loadState(){
      const rolls=parseInt(localStorage.getItem(STORAGE_KEYS.rolls)||"0",10);
      const unlocksRaw=localStorage.getItem(STORAGE_KEYS.unlocks);
      const autoRaw=localStorage.getItem(STORAGE_KEYS.auto);
      const invRaw=localStorage.getItem(STORAGE_KEYS.inv);
      const autoSellRaw=localStorage.getItem(STORAGE_KEYS.autoSell);
      state.rolls=Number.isFinite(rolls)?rolls:0;
      state.unlocks=unlocksRaw?reviveUnlocks(JSON.parse(unlocksRaw)):buildInitialUnlocks(INDEX_ITEMS);
      state.auto=autoRaw==="true";
      state.inventory=invRaw?JSON.parse(invRaw):[];
      state.autoSell=autoSellRaw||"off";
    }
    function saveState(){
      localStorage.setItem(STORAGE_KEYS.rolls,String(state.rolls));
      localStorage.setItem(STORAGE_KEYS.unlocks,JSON.stringify(state.unlocks));
      localStorage.setItem(STORAGE_KEYS.auto,state.auto?"true":"false");
      localStorage.setItem(STORAGE_KEYS.inv,JSON.stringify(state.inventory));
      localStorage.setItem(STORAGE_KEYS.autoSell,state.autoSell);
    }

    /* ---------------- State ---------------- */
    const state={ rolls:0, unlocks:buildInitialUnlocks(INDEX_ITEMS), auto:false, autoInterval:null, inventory:[], autoSell:"off", fullAnnounced:false };

    /* ---------------- Utils ---------------- */
    function clone(o){ return JSON.parse(JSON.stringify(o)); }
    function sumWeights(a){ return a.reduce((s,r)=>s+r.weight,0); }
    function luckForUpcomingRoll(n){ if(n===250) return 10; if(n===50) return 2; return 1; }
    function applyMilestoneLuckToWeights(base,luck){
      const tiers=clone(base);
      if(luck>1){ for(const t of tiers){ if(LUCK_TARGET_KEYS.includes(t.key)) t.weight*=luck; } }
      return tiers;
    }
    function toChances(tiers){ const total=sumWeights(tiers); return tiers.map(t=>({ ...t, chance:t.weight/total })); }
    function pickTier(chances){
      const r=Math.random(); let acc=0;
      for(let i=chances.length-1;i>=0;i--){ acc+=chances[i].chance; if(r<=acc) return { index:i, item:chances[i], roll:r, acc }; }
      return { index:0, item:chances[0], roll:r, acc };
    }
    function pickItemNameForTier(key){
      const list=INDEX_ITEMS[key]||[]; if(!list.length) return null;
      return list[Math.floor(Math.random()*list.length)];
    }
    function shouldAutoSell(key){
      const order=TIERS.map(t=>t.key);
      const thr=state.autoSell;
      if(thr==="off") return false;
      return order.indexOf(key) <= order.indexOf(thr);
    }

    /* ---------------- Ephemeral banners (persist through quick re-rolls) ---------------- */
    const rollArea=document.getElementById("rollArea");
    function spawnBanner(text,type){
      const div=document.createElement("div");
      div.className=`banner ${type}`;
      div.textContent=text;
      div.addEventListener("animationend",()=>div.remove());
      rollArea.appendChild(div);
      // Force reflow to ensure animation starts even if another banner just spawned
      void div.offsetWidth;
      div.style.animation="fadeout 3.6s forwards";
    }
    function showLuckBanner(mult){ if(mult>1) spawnBanner(`${mult}x luck activated`,"luck"); }
    function showNewBanner(tierName,itemName){ spawnBanner(`NEW collected: [${tierName}] ${itemName}`,"new"); }
    function showAnnouncement(text){ spawnBanner(text,"announce"); }

    /* ---------------- Roll Logic ---------------- */
    function rollOnce(){
      const upcoming=state.rolls+1;
      const tempLuck=luckForUpcomingRoll(upcoming);
      showLuckBanner(tempLuck);

      const chances=toChances(applyMilestoneLuckToWeights(TIERS,tempLuck));
      const tierPick=pickTier(chances);
      const key=tierPick.item.key, tierName=tierPick.item.name;
      const itemName=pickItemNameForTier(key);

      state.rolls++;

      // Index unlock tracking
      let isNew=false;
      if(itemName && !state.unlocks[key][itemName]){
        state.unlocks[key][itemName]=true;
        isNew=true;
      }

      // Inventory handling with auto-sell and capacity
      if(!shouldAutoSell(key)){
        if(state.inventory.length<INVENTORY_MAX){
          state.inventory.push({ tier:key, tierName, name:itemName, roll:state.rolls, luck:tempLuck });
        } else {
          if(!state.fullAnnounced){ showAnnouncement(`Inventory is full ${INVENTORY_MAX}/${INVENTORY_MAX}`); state.fullAnnounced=true; }
        }
      }
      if(state.inventory.length<INVENTORY_MAX) state.fullAnnounced=false;

      saveState();

      showGlow();
      renderResult(tierPick.item,itemName);
      if(isNew) showNewBanner(tierName,itemName);
      renderButtonsState();
      renderIndex();
      renderInventory();
    }

    /* ---------------- Auto Clicker ---------------- */
    function toggleAuto(){
      if(elAutoBtn.disabled) return;
      if(state.auto){
        state.auto=false; clearInterval(state.autoInterval); state.autoInterval=null; elAutoBtn.textContent="Auto Roll: Off";
      } else {
        state.auto=true; elAutoBtn.textContent="Auto Roll: On";
        state.autoInterval=setInterval(()=>{ if(elAutoBtn.disabled){ toggleAuto(); return; } rollOnce(); },120);
      }
      saveState();
    }

    /* ---------------- UI Elements ---------------- */
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
    const elAutoSellBtn=document.getElementById("btnAutoSell");
    const elAutoSellMenu=document.getElementById("autoSellMenu");
    const elRollBtn=document.getElementById("btnRoll");

    function renderResult(tierItem,itemName){
      elResult.textContent=itemName ? `${tierItem.name} — ${itemName}` : tierItem.name;
      elRarity.textContent=tierItem.key.toUpperCase();
      elRarity.className="rarity badge "+tierItem.colorClass;
    }
    function renderButtonsState(){
      if(state.rolls>=50){ elAutoBtn.disabled=false; elAutoBtn.textContent=state.auto?"Auto Roll: On":"Auto Roll: Off"; }
      else { elAutoBtn.disabled=true; elAutoBtn.textContent="Auto Roll (locked)"; }
      elAutoSellBtn.textContent = `Auto-Sell: ${state.autoSell==="off"?"Off":labelForAutoSell(state.autoSell)}`;
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
        section.innerHTML=`
          <h4><span class="badge ${tier.colorClass}">${tier.name}</span></h4>
          <div class="completion">${comp.percent}%</div>
        `;
        const ul=document.createElement("ul"); ul.className="index-list";
        const items=INDEX_ITEMS[tier.key]||[];
        if(!items.length){
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
        left.innerHTML=`<span class="badge ${badgeClass}">${entry.tierName}</span> — ${entry.name || "(Unknown)"} • #${entry.roll}${entry.luck>1?` • ${entry.luck}x`:``}`;
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
      if(high){ const ok=confirm(`Delete "${entry.name}" [${entry.tierName}]?`); if(!ok) return; }
      const idx=state.inventory.findIndex(i=>i.roll===entry.roll);
      if(idx>=0){
        state.inventory.splice(idx,1);
        saveState(); renderInventory();
        if(state.inventory.length<INVENTORY_MAX) state.fullAnnounced=false;
      }
    }
    function showGlow(){ const g=document.createElement("div"); g.className="glow"; rollArea.appendChild(g); setTimeout(()=>g.remove(),1100); }

    /* ---------------- Auto-Sell chooser (menu populated and scrollable) ---------------- */
    const autoSellOptions=[
      {val:"off",label:"Off"},
      {val:"worthless",label:"Worthless+"},{val:"trash",label:"Trash+"},{val:"common",label:"Common+"},{val:"uncommon",label:"Uncommon+"},
      {val:"rare",label:"Rare+"},{val:"epic",label:"Epic+"},{val:"legendary",label:"Legendary+"},{val:"mythic",label:"Mythic+"},
      {val:"divine",label:"Divine+"},{val:"celestial",label:"Celestial+"},{val:"transcendent",label:"Transcendent+"},
      {val:"eternal",label:"Eternal+"},{val:"omniversal",label:"Omniversal+"},
    ];
    function populateAutoSellMenu(){
      elAutoSellMenu.innerHTML="";
      autoSellOptions.forEach(opt=>{
        const b=document.createElement("button"); b.textContent=opt.label; b.dataset.val=opt.val;
        b.addEventListener("click",()=>{
          state.autoSell=opt.val; elAutoSellMenu.style.display="none"; renderButtonsState(); saveState();
        });
        elAutoSellMenu.appendChild(b);
      });
    }

    /* ---------------- Hooks ---------------- */
    document.getElementById("btnRoll").addEventListener("click",rollOnce);
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

    elAutoSellBtn.addEventListener("click",()=>{
      const open=elAutoSellMenu.style.display==="block";
      elAutoSellMenu.style.display=open?"none":"block";
    });
    document.addEventListener("click",(e)=>{
      if(!e.target.closest(".autosell-wrap")) elAutoSellMenu.style.display="none";
    });

    /* ---------------- Init ---------------- */
    loadState();
    populateAutoSellMenu();
    renderButtonsState();
    if(state.auto && state.rolls>=50){
      elAutoBtn.disabled=false; elAutoBtn.textContent="Auto Roll: On";
      state.autoInterval=setInterval(()=>{ if(elAutoBtn.disabled){ toggleAuto(); return; } rollOnce(); },120);
    } else {
      elAutoBtn.textContent=state.rolls>=50?"Auto Roll: Off":"Auto Roll (locked)";
    }
  </script>
</body>
</html>
