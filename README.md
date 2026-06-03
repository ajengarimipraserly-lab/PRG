<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Dragon Quest: Legacy of the Ancient Flame | RPG Fantasi 2D</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
        }

        body {
            background: #0a0f1e;
            margin: 0;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Cinema', 'Courier New', monospace;
            padding: 20px;
        }

        /* Game Container */
        .game-container {
            background: #1e2a2e;
            border-radius: 32px;
            box-shadow: 0 20px 35px rgba(0,0,0,0.5), inset 0 1px 2px rgba(255,255,200,0.1);
            padding: 16px;
        }

        .game-canvas {
            background: linear-gradient(145deg, #2c4c3b, #1d3b2a);
            border-radius: 24px;
            box-shadow: inset 0 0 0 2px #ecd9a2;
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 20px;
            cursor: pointer;
            background: #2d3e2b;
        }

        /* UI Panel */
        .ui-panel {
            margin-top: 12px;
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            background: #2b2b1f;
            border-radius: 24px;
            padding: 12px;
            color: #f9e7b3;
            text-shadow: 1px 1px 0 #2c2b1a;
        }

        .stats {
            flex: 2;
            background: #0f2125;
            border-radius: 20px;
            padding: 6px 12px;
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            align-items: center;
            font-size: 0.8rem;
            font-weight: bold;
        }

        .stats div {
            background: #2a3f36;
            padding: 4px 12px;
            border-radius: 40px;
        }

        .action-buttons {
            flex: 3;
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
            justify-content: center;
        }

        button {
            background: #ae7e4b;
            border: none;
            color: white;
            font-weight: bold;
            padding: 6px 14px;
            border-radius: 36px;
            cursor: pointer;
            font-family: monospace;
            transition: 0.07s linear;
            box-shadow: 0 3px 0 #5a3a1a;
        }

        button:active {
            transform: translateY(2px);
            box-shadow: 0 1px 0 #5a3a1a;
        }

        .log-area {
            background: #1d2822;
            border-radius: 18px;
            padding: 8px;
            margin-top: 10px;
            max-height: 100px;
            overflow-y: auto;
            font-size: 0.7rem;
            color: #e7dbaa;
        }

        @media (max-width: 700px) {
            .stats div { font-size: 0.7rem; padding: 2px 8px;}
            button { padding: 4px 10px; font-size: 0.7rem;}
        }
    </style>
</head>
<body>
<div>
    <div class="game-container">
        <div class="game-canvas">
            <canvas id="gameCanvas" width="800" height="450" style="width:100%; height:auto; max-width:800px; aspect-ratio:800/450"></canvas>
        </div>
        <div class="ui-panel" id="uiPanel"></div>
        <div class="log-area" id="logMessages">✨ Selamat datang, petualang! Pilih kelas dan taklukkan naga kuno!</div>
    </div>
</div>

<script>
    (function(){
        // ---------- GAME STATE RPG ----------
        let gameState = {
            // pemain
            player: {
                name: "Hero",
                class: "Warrior",    // Warrior, Mage, Archer
                level: 1,
                xp: 0,
                xpToNext: 100,
                hp: 100,
                maxHp: 100,
                mp: 50,
                maxMp: 50,
                attack: 20,
                defense: 10,
                magic: 15,
                agility: 12,
                gold: 150,
                inventory: []      // item {id, name, type, value, qty}
            },
            // dunia & progres
            region: "Forest",     // Forest, Desert, Mountain, Castle
            mainQuestsDone: 0,    // 0-10
            sideQuestsDone: 0,    // 0-15
            currentQuest: null,
            questLog: [],
            defeatedBoss: { Forest: false, Desert: false, Mountain: false, Castle: false },
            location: "Forest_Outskirts",
            // pertarungan
            inBattle: false,
            currentEnemy: null,
            battleMessage: "",
            // pilihan cerita sementara
            storyChoice: null,
        };

        // ---------- DATA DUNIA ----------
        const REGIONS = ["Forest", "Desert", "Mountain", "Castle"];
        const REGION_NAMES = { Forest: "🌲 Hutan Purba", Desert: "🏜️ Gurun Pasir", Mountain: "⛰️ Pegunungan Es", Castle: "🏰 Kastel Naga" };

        // Musuh berdasarkan region & progres
        const ENEMIES = {
            Forest: [
                { name: "Slime Hijau", level: 1, hp: 25, attack: 8, defense: 3, exp: 35, gold: 20, icon: "🟢" },
                { name: "Serigala Hutan", level: 2, hp: 45, attack: 14, defense: 5, exp: 55, gold: 35, icon: "🐺" },
                { name: "Treant Kecil", level: 3, hp: 70, attack: 12, defense: 8, exp: 80, gold: 45, icon: "🌳" }
            ],
            Desert: [
                { name: "Scorpion", level: 5, hp: 80, attack: 20, defense: 12, exp: 110, gold: 70, icon: "🦂" },
                { name: "Mummy Guard", level: 6, hp: 110, attack: 24, defense: 15, exp: 140, gold: 90, icon: "🫅" },
                { name: "Efreet Kecil", level: 7, hp: 130, attack: 28, defense: 10, exp: 180, gold: 120, icon: "🔥" }
            ],
            Mountain: [
                { name: "Golem Batu", level: 9, hp: 180, attack: 32, defense: 22, exp: 220, gold: 150, icon: "🗿" },
                { name: "Yeti", level: 10, hp: 200, attack: 36, defense: 18, exp: 260, gold: 180, icon: "❄️" },
                { name: "Griffin", level: 11, hp: 230, attack: 40, defense: 20, exp: 310, gold: 230, icon: "🦅" }
            ],
            Castle: [
                { name: "Dark Knight", level: 13, hp: 280, attack: 48, defense: 30, exp: 400, gold: 300, icon: "⚔️" },
                { name: "Wizard Bayangan", level: 14, hp: 240, attack: 52, defense: 20, exp: 450, gold: 350, icon: "🔮" },
                { name: "Naga Muda", level: 15, hp: 350, attack: 60, defense: 35, exp: 550, gold: 500, icon: "🐉" }
            ]
        };

        // BOSS tiap region
        const BOSSES = {
            Forest: { name: "Raja Hutan", hp: 200, attack: 28, defense: 15, exp: 400, gold: 300, icon: "👑🐗", story: "Penguasa Hutan" },
            Desert: { name: "Naga Pasir", hp: 380, attack: 42, defense: 25, exp: 700, gold: 600, icon: "🐉🏜️" },
            Mountain: { name: "Roc Raksasa", hp: 520, attack: 55, defense: 30, exp: 1000, gold: 900, icon: "🦅⛰️" },
            Castle: { name: "Naga Kuno", hp: 850, attack: 75, defense: 40, exp: 2500, gold: 2000, icon: "🐲🔥", final: true }
        };

        // item database
        const ITEMS = {
            hpotion: { name: "Ramuan Penyembuh", type: "heal", value: 60, price: 40, emoji: "🧪" },
            mpotion: { name: "Ramuan Mana", type: "mana", value: 40, price: 35, emoji: "💙" },
            sword1: { name: "Pedang Besi", type: "weapon", attackBonus: 8, price: 150, emoji: "⚔️" },
            bow1: { name: "Busur Panjang", type: "weapon", attackBonus: 9, price: 160, emoji: "🏹" },
            staff1: { name: "Tongkat Sihir", type: "weapon", magicBonus: 10, price: 170, emoji: "🔱" },
            armor1: { name: "Zirah Kulit", type: "armor", defenseBonus: 6, price: 120, emoji: "🛡️" }
        };

        // misi utama (10) & sampingan (15) terintegrasi dengan progres
        let mainQuests = Array(10).fill().map((_,i) => ({ id: i, title: `Misi Utama ${i+1}`, done: false, regionReq: i<2?"Forest": i<5?"Desert": i<8?"Mountain":"Castle" }));
        let sideQuests = Array(15).fill().map((_,i) => ({ id: i, title: `Sampingan: ${["Bantu Elf","Koleksi Barang","Hilangkan Monster","Ramuan Langka","Perlindungan Desa"][i%5]} ${i+1}`, done: false }));

        // UI elements
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const uiPanel = document.getElementById('uiPanel');
        const logDiv = document.getElementById('logMessages');

        function addLog(msg) { logDiv.innerHTML += `<div>⚔️ ${msg}</div>`; logDiv.scrollTop = logDiv.scrollHeight; if(logDiv.children.length>12) logDiv.removeChild(logDiv.firstChild); }

        // update panel UI & tombol aksi
        function refreshUI() {
            const p = gameState.player;
            let regionBossDone = gameState.defeatedBoss[gameState.region];
            let regionIdx = REGIONS.indexOf(gameState.region);
            uiPanel.innerHTML = `
                <div class="stats">
                    <div>🧙 ${p.class} Lv.${p.level}</div>
                    <div>❤️ ${p.hp}/${p.maxHp}</div>
                    <div>✨ XP ${p.xp}/${p.xpToNext}</div>
                    <div>⚔️ ATK ${p.attack}</div>
                    <div>🛡️ DEF ${p.defense}</div>
                    <div>💰 ${p.gold}</div>
                    <div>🌍 ${REGION_NAMES[gameState.region]}</div>
                    <div>📜 Misi Utama: ${gameState.mainQuestsDone}/10 | Samping: ${gameState.sideQuestsDone}/15</div>
                </div>
                <div class="action-buttons">
                    <button id="exploreBtn">🌲 Jelajahi</button>
                    <button id="restBtn">🏕️ Istirahat</button>
                    <button id="invBtn">🎒 Inventaris</button>
                    <button id="questBtn">📜 Misi</button>
                    <button id="changeRegionBtn">🗺️ Pindah Wilayah</button>
                </div>
            `;
            document.getElementById('exploreBtn')?.addEventListener('click', () => exploreRegion());
            document.getElementById('restBtn')?.addEventListener('click', () => { if(!gameState.inBattle){ p.hp = p.maxHp; p.mp = p.maxMp; addLog("Beristirahat, pulih penuh!"); refreshUI(); drawWorldMap(); } else addLog("Tidak bisa saat bertarung!");});
            document.getElementById('invBtn')?.addEventListener('click', showInventory);
            document.getElementById('questBtn')?.addEventListener('click', showQuests);
            document.getElementById('changeRegionBtn')?.addEventListener('click', changeRegionMenu);
            drawWorldMap();
        }

        // Pindah wilayah
        function changeRegionMenu() {
            if(gameState.inBattle) { addLog("Sedang bertarung!"); return; }
            let msg = "Pilih wilayah:\n";
            REGIONS.forEach((reg, idx) => { msg += `${idx+1}. ${REGION_NAMES[reg]} ${gameState.defeatedBoss[reg]?"✅":""}\n`; });
            let choice = prompt(msg + "Masukkan nomor (1-4):");
            let idx = parseInt(choice)-1;
            if(idx>=0 && idx<4) {
                gameState.region = REGIONS[idx];
                addLog(`Berpindah ke ${REGION_NAMES[gameState.region]}`);
                refreshUI();
                drawWorldMap();
            } else addLog("Pilihan salah");
        }

        // Jelajahi => random encounter / boss jika belum kalah
        function exploreRegion() {
            if(gameState.inBattle) return;
            let region = gameState.region;
            if(!gameState.defeatedBoss[region]) {
                // boss battle
                let boss = BOSSES[region];
                if(confirm(`⚠️ BOSS ${boss.name} menghadang! Lawan?`)) {
                    startBattle({ ...boss, isBoss: true, exp: boss.exp, gold: boss.gold, hp: boss.hp, maxHp: boss.hp, attack: boss.attack, defense: boss.defense, name: boss.name, icon: boss.icon });
                } else addLog("Melarikan diri...");
                return;
            }
            // random musuh biasa
            let enemies = ENEMIES[region];
            let idx = Math.floor(Math.random() * enemies.length);
            let enemy = { ...enemies[idx], isBoss: false, maxHp: enemies[idx].hp };
            startBattle(enemy);
        }

        // sistem pertarungan
        function startBattle(enemy) {
            gameState.inBattle = true;
            gameState.currentEnemy = { ...enemy, currentHp: enemy.hp };
            addLog(`⚔️ Bertarung dengan ${enemy.name} (Lv.${enemy.level || 'BOSS'})!`);
            drawBattleScreen();
        }

        function attackAction() {
            let p = gameState.player;
            let e = gameState.currentEnemy;
            let dmg = Math.max(1, p.attack + (Math.floor(Math.random()*8)) - e.defense);
            e.currentHp -= dmg;
            addLog(`💥 Seranganmu memberi ${dmg} damage ke ${e.name}!`);
            if(e.currentHp <= 0) { winBattle(); return; }
            enemyTurn();
        }

        function skillAction() {
            let p = gameState.player;
            let e = gameState.currentEnemy;
            let mpCost = 12;
            if(p.mp < mpCost) { addLog("Mana tidak cukup!"); return; }
            let skillDmg = Math.max(5, (p.magic + p.attack/2) + Math.floor(Math.random()*12) - e.defense);
            if(p.class === "Mage") skillDmg += 10;
            if(p.class === "Archer") skillDmg += 5;
            e.currentHp -= skillDmg;
            p.mp -= mpCost;
            addLog(`✨ Skill Hero! Damage ${skillDmg} ke ${e.name}!`);
            if(e.currentHp <= 0) { winBattle(); return; }
            enemyTurn();
        }

        function healAction() {
            let p = gameState.player;
            // cari item penyembuh di inventaris
            let healItem = p.inventory.find(i => i.id === 'hpotion' && i.qty > 0);
            if(!healItem) { addLog("Tidak punya ramuan penyembuh!"); return; }
            healItem.qty--;
            let healAmt = ITEMS.hpotion.value;
            p.hp = Math.min(p.maxHp, p.hp+healAmt);
            addLog(`🧪 Menggunakan Ramuan, pulih ${healAmt} HP!`);
            refreshUI();
            enemyTurn();
        }

        function enemyTurn() {
            let p = gameState.player;
            let e = gameState.currentEnemy;
            let dmg = Math.max(1, e.attack + Math.floor(Math.random()*6) - p.defense);
            p.hp -= dmg;
            addLog(`😈 ${e.name} menyerangmu! ${dmg} damage.`);
            if(p.hp <= 0) { loseBattle(); return; }
            drawBattleScreen();
        }

        function winBattle() {
            let e = gameState.currentEnemy;
            let gainExp = e.exp;
            let gainGold = e.gold;
            gameState.player.gold += gainGold;
            gameState.player.xp += gainExp;
            addLog(`🏆 Menang! Dapat ${gainExp} XP dan ${gainGold} gold!`);
            if(e.isBoss) {
                gameState.defeatedBoss[gameState.region] = true;
                addLog(`🎉 Boss ${e.name} dikalahkan! Wilayah ${REGION_NAMES[gameState.region]} aman.`);
                if(gameState.region === "Castle") addLog("👑 Naga Kuno tumbang! Kerajaan Selamat! Permainan selesai. 🏅");
            }
            checkLevelUp();
            gameState.inBattle = false;
            gameState.currentEnemy = null;
            refreshUI();
            drawWorldMap();
        }

        function loseBattle() {
            addLog("💀 Kalah... kembali ke kota dengan setengah emas.");
            gameState.player.gold = Math.floor(gameState.player.gold/2);
            gameState.player.hp = gameState.player.maxHp;
            gameState.player.mp = gameState.player.maxMp;
            gameState.inBattle = false;
            gameState.currentEnemy = null;
            refreshUI();
            drawWorldMap();
        }

        function checkLevelUp() {
            let p = gameState.player;
            while(p.xp >= p.xpToNext) {
                p.xp -= p.xpToNext;
                p.level++;
                p.maxHp += 15; p.hp = p.maxHp;
                p.maxMp += 8; p.mp = p.maxMp;
                p.attack += 5;
                p.defense += 3;
                p.magic += 4;
                p.xpToNext = Math.floor(100 + p.level * 25);
                addLog(`🎉 LEVEL UP! Sekarang Level ${p.level}, stat meningkat!`);
                // misi progres otomatis berdasarkan level & region
                if(p.level >= 3 && gameState.mainQuestsDone < 2 && !gameState.defeatedBoss.Forest) { gameState.mainQuestsDone = Math.min(10, gameState.mainQuestsDone+1); addLog("📜 Misi Utama: Hutan Terlindungi +1");}
                if(p.level >= 6 && gameState.mainQuestsDone < 5 && gameState.defeatedBoss.Desert) gameState.mainQuestsDone++;
                if(p.level >= 9 && gameState.mainQuestsDone < 8 && gameState.defeatedBoss.Mountain) gameState.mainQuestsDone++;
                if(p.level >= 12 && gameState.mainQuestsDone < 10 && gameState.defeatedBoss.Castle) gameState.mainQuestsDone++;
            }
            refreshUI();
        }

        function drawBattleScreen() {
            let p = gameState.player;
            let e = gameState.currentEnemy;
            ctx.clearRect(0,0,800,450);
            ctx.fillStyle = "#201e1a";
            ctx.fillRect(0,0,800,450);
            ctx.font = "bold 18px monospace";
            ctx.fillStyle = "#ffd966";
            ctx.fillText(`⚔️ VS ${e.name} ${e.icon || ""}`, 40, 50);
            ctx.font = "14px monospace";
            ctx.fillStyle = "#fff0b5";
            ctx.fillText(`❤️ ${e.currentHp}/${e.hp}  ⚔️ ${e.attack}`, 40, 90);
            ctx.fillStyle = "#b9f6ca";
            ctx.fillText(`${p.name} [${p.class}] Lv.${p.level}`, 500, 50);
            ctx.fillStyle = "#ffaaaa";
            ctx.fillText(`HP: ${p.hp}/${p.maxHp}  MP: ${p.mp}/${p.maxMp}`, 500, 90);
            // tombol aksi gambar interaktif?
            document.querySelectorAll('.battle-btn').forEach(b=>b.remove());
            let btnDiv = document.createElement('div');
            btnDiv.className = "action-buttons battle-btn";
            btnDiv.style.marginTop = "8px";
            btnDiv.style.display = "flex"; btnDiv.style.gap="8px"; btnDiv.style.justifyContent="center";
            btnDiv.innerHTML = `<button id="attackBtn">⚔️ Serang</button><button id="skillBtn">✨ Skill Hero</button><button id="healBtn">🧪 Ramuan</button>`;
            uiPanel.appendChild(btnDiv);
            document.getElementById('attackBtn')?.addEventListener('click', ()=>{ attackAction(); drawBattleScreen(); });
            document.getElementById('skillBtn')?.addEventListener('click', ()=>{ skillAction(); drawBattleScreen(); });
            document.getElementById('healBtn')?.addEventListener('click', ()=>{ healAction(); drawBattleScreen(); refreshUI(); });
        }

        function drawWorldMap() {
            if(gameState.inBattle) return;
            ctx.clearRect(0,0,800,450);
            let grad = ctx.createLinearGradient(0,0,800,450);
            grad.addColorStop(0,"#347a4a");
            grad.addColorStop(1,"#1e5631");
            ctx.fillStyle=grad; ctx.fillRect(0,0,800,450);
            ctx.font = "bold 24px 'Segoe UI'";
            ctx.fillStyle = "#ffefb0";
            ctx.fillText(`${REGION_NAMES[gameState.region]} | Petualang ${gameState.player.class}`, 40, 60);
            ctx.font = "16px monospace";
            ctx.fillStyle = "#ffeecc";
            ctx.fillText("🗺️ Gunakan tombol bawah untuk aksi: Jelajahi / Istirahat / Inventori", 40, 110);
            // gambar icon wilayah sederhana
            ctx.font = "40px monospace";
            if(gameState.region === "Forest") ctx.fillText("🌳🏕️", 350, 250);
            if(gameState.region === "Desert") ctx.fillText("🏜️🐪", 350, 250);
            if(gameState.region === "Mountain") ctx.fillText("⛰️❄️", 350, 250);
            if(gameState.region === "Castle") ctx.fillText("🏰🐲", 350, 250);
    
