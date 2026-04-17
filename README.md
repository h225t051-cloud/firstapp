# firstapp
はじめてのアプリ
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>レトロ妖怪格闘アクション</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- ドット風フォントの読み込み -->
    <link href="https://fonts.googleapis.com/css2?family=DotGothic16&display=swap" rel="stylesheet">
    <style>
        canvas:focus { outline: none; }
        body {
            font-family: 'DotGothic16', sans-serif;
            background-color: #0f172a;
        }
        /* レトロゲーム風のテキストシャドウ */
        .pixel-text {
            text-shadow: 2px 2px 0 #000, -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000;
        }
    </style>
</head>
<body class="flex items-center justify-center h-screen m-0 overflow-hidden text-white relative select-none">

    <!-- タイトル画面（ホーム画面） -->
    <div id="title-screen" class="absolute inset-0 flex flex-col items-center justify-center bg-gray-900 z-40 transition-opacity duration-500 overflow-y-auto py-10 pixel-text">
        <!-- 背景の装飾（薄いグリッド） -->
        <div class="absolute inset-0 opacity-10" style="background-image: linear-gradient(#333 1px, transparent 1px), linear-gradient(90deg, #333 1px, transparent 1px); background-size: 30px 30px;"></div>
        
        <h1 class="text-7xl md:text-9xl text-red-600 font-black mb-2 tracking-widest text-center relative z-10" style="text-shadow: 4px 4px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 2px 2px 0 #000, 0px 0px 20px rgba(220,38,38,0.8);">妖魔覇戦</h1>
        <p class="text-xl md:text-2xl text-yellow-400 mb-12 tracking-[0.5em] relative z-10">YOMA HASEN</p>

        <!-- 操作方法パネル -->
        <div class="bg-gray-800 border-4 border-gray-500 p-6 rounded text-center mb-12 max-w-lg w-full shadow-2xl relative z-10">
            <h2 class="text-2xl text-white mb-6 border-b-2 border-gray-600 pb-2">【 操作方法 】</h2>
            <div class="grid grid-cols-2 gap-y-4 gap-x-8 text-left pl-4 md:pl-10 text-lg md:text-xl">
                <div class="text-gray-300"><span class="text-yellow-300 inline-block w-16">[A][D]</span>: 移動</div>
                <div class="text-gray-300"><span class="text-yellow-300 inline-block w-12">[W]</span>: ジャンプ</div>
                <div class="text-gray-300"><span class="text-yellow-300 inline-block w-16">[J]</span>: パンチ</div>
                <div class="text-gray-300"><span class="text-red-400 inline-block w-12 font-bold">[K]</span>: 必殺技 <span class="text-xs text-gray-400">(MP消費)</span></div>
            </div>
        </div>

        <button id="to-select-btn" class="px-12 py-4 bg-red-700 text-white font-bold text-3xl rounded border-4 border-red-900 hover:bg-red-500 hover:scale-110 transition-all animate-pulse relative z-10 shadow-[0_0_15px_rgba(220,38,38,0.6)]">
            GAME START
        </button>
    </div>

    <!-- キャラクター選択画面 -->
    <div id="character-select" class="absolute inset-0 flex flex-col items-center justify-center bg-gray-900 z-30 transition-opacity duration-500 overflow-y-auto py-10 hidden">
        <h1 class="text-4xl md:text-5xl text-yellow-400 font-bold mb-8 tracking-widest text-center pixel-text">キャラクター選択</h1>
        <div id="char-list" class="grid grid-cols-2 md:grid-cols-3 gap-6 mb-12 max-w-5xl px-4">
            <!-- JSで動的に生成します -->
        </div>
        <button id="start-btn" class="px-10 py-4 bg-gray-600 text-gray-300 font-bold text-2xl rounded border-4 border-gray-500 cursor-not-allowed opacity-50 transition-all pixel-text" disabled>
            キャラクターを選択...
        </button>
    </div>

    <!-- UI レイヤー -->
    <div id="game-ui" class="absolute top-4 left-0 w-full flex justify-between px-8 z-10 pointer-events-none hidden">
        
        <!-- プレイヤー側 UI -->
        <div class="w-5/12 flex flex-col">
            <div id="player-name" class="text-lg font-bold text-blue-400 mb-1 pixel-text">PLAYER</div>
            <div class="w-full bg-red-900 h-6 border-4 border-gray-400 relative flex justify-end">
                <div id="player-hp" class="bg-green-400 h-full w-full transition-all duration-200"></div>
            </div>
            <div class="w-full bg-gray-700 h-3 border-x-4 border-b-4 border-gray-400 relative">
                <div id="player-mp" class="bg-cyan-400 h-full w-full transition-all duration-75"></div>
            </div>
        </div>

        <!-- タイマー -->
        <div id="timer" class="text-4xl font-bold text-white flex items-center justify-center w-24 bg-gray-800 border-4 border-gray-400 pb-1 pixel-text">
            60
        </div>

        <!-- 敵側 UI -->
        <div class="w-5/12 flex flex-col items-end">
            <div id="enemy-name" class="text-lg font-bold text-red-500 mb-1 pixel-text">ENEMY</div>
            <div class="w-full bg-red-900 h-6 border-4 border-gray-400 relative flex justify-start">
                <div id="enemy-hp" class="bg-green-400 h-full w-full transition-all duration-200"></div>
            </div>
            <!-- 敵にMPバーは表示しないが内部で消費する -->
        </div>

    </div>

    <!-- リザルト画面 -->
    <div id="result" class="absolute inset-0 flex flex-col items-center justify-center z-20 hidden bg-black bg-opacity-80 tracking-widest pixel-text">
        <div id="result-text" class="text-5xl md:text-7xl font-bold text-yellow-400 mb-8"></div>
        <button id="restart-btn" class="px-8 py-3 bg-gray-700 text-white font-bold text-xl rounded border-4 border-gray-500 hover:bg-gray-600 hover:border-white transition-all hidden">
            キャラクター選択に戻る
        </button>
    </div>

    <!-- 操作説明 -->
    <div class="absolute bottom-4 left-0 w-full text-center text-sm text-gray-300 z-10 pointer-events-none opacity-90 pixel-text">
        [A][D] 移動  |  [W] ジャンプ  |  [J] パンチ  |  [K] 必殺技（超能力）
    </div>

    <!-- ゲームキャンバス -->
    <canvas id="gameCanvas" width="1024" height="576" class="bg-gray-800 shadow-2xl rounded border-4 border-gray-600" tabindex="0"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const c = canvas.getContext('2d');
        window.onload = () => canvas.focus();

        canvas.width = 1024;
        canvas.height = 576;
        const gravity = 0.8;
        const groundLevel = canvas.height - 50;

        // --- ピクセルアート（ドット絵）システム ---
        
        // カラーパレット
        const palette = {
            'K': '#000000', 'W': '#ffffff', 'R': '#ef4444', 'G': '#22c55e', 
            'B': '#3b82f6', 'Y': '#eab308', 'P': '#a855f7', 'S': '#fcd34d', 
            'C': '#92400e', 'D': '#78350f', 'L': '#38bdf8', 'M': '#9ca3af',
            'O': '#b91c1c' // 暗い赤
        };

        // 12x12のスプライトデータ（_は透明）
        const sprites = {
            tengu: [
                "___KKK______",
                "___KKK______",
                "__RRRR______",
                "_RRWRWR_____",
                "_RRRRRRRRR__", // 長い鼻
                "__RRRR______",
                "__MMMM______",
                "__WM_MW_____",
                "__MMMM______",
                "__KKKK______",
                "__K__K______",
                "__K__K______"
            ],
            oni: [
                "___W________", // 角
                "__RRRR______",
                "_RRWRWR_____",
                "_RRRRRR_____",
                "__RRRR______",
                "__RRRR______",
                "__YKYK______", // 虎パンツ
                "__KYKY______",
                "__RRRR______",
                "__R__R______",
                "__R__R______",
                "__K__K______"
            ],
            vampire: [
                "____KKKK____",
                "___KWWWWK___",
                "___WWRWRW___",
                "___WWWWWW___",
                "____WWWW____",
                "____KKKK____",
                "_KKKRRRRKKK_", // マント
                "KKKKRKKRKKKK",
                "KKKKKKKKKKKK",
                "KKKK____KKKK",
                "___K____K___",
                "___K____K___"
            ],
            werewolf: [
                "__C___C_____", // 耳
                "_CCCCCCC____",
                "_CCWCWCCC___",
                "_CCCCCCCCC__", // 鼻先
                "__CCWWCC____",
                "__DDDDDD____",
                "_DDDDDDDD___",
                "_DDDDDDDD___",
                "__CCCCCC____",
                "__CC__CC____",
                "__C____C____",
                "__C____C____"
            ],
            wukong: [
                "____________",
                "____YYYY____", // 輪
                "___CWCWC____",
                "___CCCCCCC__",
                "___SSSSSS___",
                "____RRRR____",
                "__RRRRRRRR__",
                "YYYRRYYRRYYY", // 如意棒構え
                "___YYYYYY___",
                "____K__K____",
                "____K__K____",
                "____K__K____"
            ],
            kappa: [
                "____________",
                "___GLLLG____", // 皿
                "__GGGGGGG___",
                "_GGGWWGGG___",
                "_GGGYYYGG___", // くちばし
                "___YYGG_____",
                "__GMMMMG____", // 甲羅
                "_GGMMMMGG___",
                "_GGMMMMGG___",
                "__GG__GG____",
                "__G____G____",
                "__G____G____"
            ]
        };

        // スプライト描画関数
        function drawPixelArt(ctx, spriteData, x, y, scale, facingRight) {
            for (let row = 0; row < spriteData.length; row++) {
                for (let col = 0; col < spriteData[row].length; col++) {
                    const colorCode = spriteData[row][col];
                    if (colorCode === '_') continue;
                    
                    ctx.fillStyle = palette[colorCode] || 'magenta';
                    // 向きに合わせてX軸を反転させる
                    const drawCol = facingRight ? col : (spriteData[row].length - 1 - col);
                    ctx.fillRect(x + drawCol * scale, y + row * scale, scale, scale);
                }
            }
        }

        // --- ゲーム状態とキャラクターデータ ---
        let gameState = 'TITLE'; // 初期状態をタイトル画面に変更
        const keys = { a: { pressed: false }, d: { pressed: false }, w: { pressed: false } };

        const characterData = {
            'tengu': {
                name: '天狗', color: '#ef4444', speed: 7, jump: -20, attackDamage: 8, spCost: 30, spType: 'tornado',
                desc: '空中戦が得意。巨大な竜巻を放つ。', sprite: sprites.tengu
            },
            'oni': {
                name: '鬼', color: '#b91c1c', speed: 4, jump: -14, attackDamage: 15, spCost: 40, spType: 'wave',
                desc: '圧倒的パワー。地を這う衝撃波を放つ。', sprite: sprites.oni
            },
            'vampire': {
                name: '吸血鬼', color: '#7e22ce', speed: 6, jump: -16, attackDamage: 10, spCost: 35, spType: 'bat',
                desc: '攻撃ヒット時に少しHPを吸収する。', sprite: sprites.vampire
            },
            'werewolf': {
                name: '人狼', color: '#92400e', speed: 10, jump: -15, attackDamage: 12, spCost: 20, spType: 'claw',
                desc: '超スピード。前方に鋭い爪の連撃。', sprite: sprites.werewolf
            },
            'wukong': {
                name: '孫悟空', color: '#eab308', speed: 8, jump: -18, attackDamage: 9, spCost: 30, spType: 'pole',
                desc: '通常リーチ長め。如意棒を一気に伸ばす。', sprite: sprites.wukong
            },
            'kappa': {
                name: '河童', color: '#22c55e', speed: 6, jump: -16, attackDamage: 8, spCost: 15, spType: 'water',
                desc: '燃費が良く、水球を放物線状に連射可能。', sprite: sprites.kappa
            }
        };

        // --- クラス定義 ---

        class Projectile {
            constructor({ position, velocity, facingRight, type, owner }) {
                this.position = position;
                this.velocity = velocity;
                this.facingRight = facingRight;
                this.type = type;
                this.owner = owner; // 誰が撃ったか
                this.markedForDeletion = false;
                this.hasHit = false; // 当たったかどうかのフラグ（貫通してエフェクトを残すため）
                this.frames = 0;
                this.life = 0; // 一部の技用の寿命タイマー
                this.history = []; // 残像エフェクト用の軌跡履歴

                // 技ごとの初期設定
                switch(this.type) {
                    case 'tornado':
                        this.width = 70; this.height = 140;
                        this.position.y = this.owner.position.y - 20; // 少し上から発生
                        this.damage = 15;
                        break;
                    case 'wave':
                        this.width = 30; this.height = 80;
                        this.position.y = groundLevel - this.height; // 地面を這う
                        this.damage = 25;
                        break;
                    case 'bat':
                        this.width = 30; this.height = 20;
                        this.damage = 12;
                        break;
                    case 'claw':
                        // 弾ではなく、一瞬前方に発生する判定
                        this.width = 90; this.height = 100;
                        this.velocity.x = 0; // 進まない
                        this.life = 15; // 少し長めにしてエフェクトを見せる
                        this.damage = 18;
                        break;
                    case 'pole':
                        // 長い棒
                        this.width = 350; this.height = 20;
                        this.velocity.x = 0;
                        this.position.y += 20;
                        this.life = 15; // 少し長めにしてエフェクトを見せる
                        this.damage = 15;
                        // 向きによる位置調整
                        if(!this.facingRight) this.position.x = this.owner.position.x - this.width;
                        break;
                    case 'water':
                        this.width = 25; this.height = 25;
                        this.velocity.y = -5; // 少し上に撃ち上げる
                        this.damage = 10;
                        break;
                }
            }

            draw() {
                c.save();
                // 全体的に発光（グロウ）効果を追加してかっこよく
                c.shadowBlur = 15;
                
                if (this.type === 'tornado') {
                    c.shadowColor = '#86efac';
                    c.fillStyle = 'rgba(134, 239, 172, 0.7)';
                    
                    // 竜巻を複数の楕円と動きで表現
                    for(let i=0; i<6; i++) {
                        const yOff = i * 25;
                        const wOff = i * 6;
                        const sOff = Math.sin(this.frames * 0.4 + i) * 12;
                        
                        c.beginPath();
                        c.ellipse(
                            this.position.x + this.width/2 + sOff, 
                            this.position.y + this.height - yOff, 
                            this.width/2 - wOff + 10, 
                            15, 0, 0, Math.PI*2
                        );
                        c.fill();
                        
                        // 風の線を重ねる
                        c.strokeStyle = 'rgba(255, 255, 255, 0.6)';
                        c.lineWidth = 2;
                        c.stroke();
                    }
                } 
                else if (this.type === 'wave') {
                    c.shadowColor = '#ef4444';
                    
                    // グラデーションで燃えるような斬撃波
                    const grad = c.createLinearGradient(this.position.x, 0, this.position.x + this.width, 0);
                    grad.addColorStop(0, 'rgba(239, 68, 68, 0)');
                    grad.addColorStop(0.5, '#ef4444');
                    grad.addColorStop(1, '#fca5a5');
                    c.fillStyle = grad;
                    
                    c.beginPath();
                    if(this.facingRight) {
                        c.moveTo(this.position.x - 20, this.position.y - 10);
                        c.quadraticCurveTo(this.position.x + this.width + 30, this.position.y + this.height/2, this.position.x - 20, this.position.y + this.height + 10);
                        c.quadraticCurveTo(this.position.x + this.width/2, this.position.y + this.height/2, this.position.x - 20, this.position.y - 10);
                    } else {
                        c.moveTo(this.position.x + this.width + 20, this.position.y - 10);
                        c.quadraticCurveTo(this.position.x - 30, this.position.y + this.height/2, this.position.x + this.width + 20, this.position.y + this.height + 10);
                        c.quadraticCurveTo(this.position.x + this.width/2, this.position.y + this.height/2, this.position.x + this.width + 20, this.position.y - 10);
                    }
                    c.fill();
                }
                else if (this.type === 'bat') {
                    c.shadowColor = '#a855f7';
                    
                    // 残像（トレイル）を描画
                    this.history.forEach((h, index) => {
                        const alpha = index / this.history.length;
                        c.fillStyle = `rgba(168, 85, 247, ${alpha * 0.4})`;
                        c.beginPath();
                        c.arc(h.x + 15, h.y + 10, 8, 0, Math.PI*2);
                        c.fill();
                    });

                    const yOffset = Math.sin(this.frames * 0.5) * 8;
                    c.fillStyle = '#7e22ce';
                    
                    // コウモリの本体
                    c.beginPath();
                    c.arc(this.position.x + 15, this.position.y + 10 + yOffset, 8, 0, Math.PI*2);
                    c.fill();
                    
                    // なめらかに動く翼
                    const wingSway = Math.cos(this.frames * 0.5) * 18;
                    c.beginPath();
                    c.moveTo(this.position.x + 15, this.position.y + 10 + yOffset);
                    c.quadraticCurveTo(this.position.x - 15, this.position.y - wingSway + yOffset, this.position.x - 10, this.position.y - 5 + yOffset);
                    c.lineTo(this.position.x + 15, this.position.y + 10 + yOffset);
                    c.fill();
                    
                    c.beginPath();
                    c.moveTo(this.position.x + 15, this.position.y + 10 + yOffset);
                    c.quadraticCurveTo(this.position.x + 45, this.position.y - wingSway + yOffset, this.position.x + 40, this.position.y - 5 + yOffset);
                    c.lineTo(this.position.x + 15, this.position.y + 10 + yOffset);
                    c.fill();
                    
                    // 光る赤い目
                    c.shadowBlur = 5;
                    c.shadowColor = 'red';
                    c.fillStyle = '#fca5a5';
                    if(this.facingRight) {
                        c.fillRect(this.position.x + 18, this.position.y + 8 + yOffset, 3, 3);
                    } else {
                        c.fillRect(this.position.x + 9, this.position.y + 8 + yOffset, 3, 3);
                    }
                }
                else if (this.type === 'claw') {
                    c.shadowColor = '#fbbf24';
                    c.strokeStyle = '#fef08a';
                    const alpha = this.life / 15; // 徐々に消える
                    c.globalAlpha = alpha;
                    c.lineCap = 'round';
                    
                    // 鋭い3本の斬撃痕
                    for(let i=0; i<3; i++) {
                        c.lineWidth = 8 - i;
                        c.beginPath();
                        if(this.facingRight) {
                            c.moveTo(this.position.x + 10 + i*20, this.position.y + 10 - i*10); 
                            c.quadraticCurveTo(this.position.x + 60 + i*15, this.position.y + 50, this.position.x + 30 + i*20, this.position.y + 100 - i*10);
                        } else {
                            c.moveTo(this.position.x + 80 - i*20, this.position.y + 10 - i*10); 
                            c.quadraticCurveTo(this.position.x + 30 - i*15, this.position.y + 50, this.position.x + 60 - i*20, this.position.y + 100 - i*10);
                        }
                        c.stroke();
                    }
                    c.globalAlpha = 1.0;
                }
                else if (this.type === 'pole') {
                    c.shadowColor = '#eab308';
                    c.shadowBlur = 20;
                    
                    // 立体的な金属のグラデーション
                    const grad = c.createLinearGradient(0, this.position.y, 0, this.position.y + this.height);
                    grad.addColorStop(0, '#7f1d1d');
                    grad.addColorStop(0.5, '#ef4444');
                    grad.addColorStop(1, '#7f1d1d');
                    c.fillStyle = grad;
                    
                    c.fillRect(this.position.x, this.position.y, this.width, this.height);
                    
                    // 両端の金の装飾
                    c.fillStyle = '#fde047'; 
                    c.fillRect(this.position.x, this.position.y - 4, 30, this.height + 8);
                    c.fillRect(this.position.x + this.width - 30, this.position.y - 4, 30, this.height + 8);
                    
                    // 伸びた瞬間の衝撃波リング
                    if(this.life > 10) {
                        c.beginPath();
                        c.strokeStyle = `rgba(253, 224, 71, ${(this.life - 10) / 5})`;
                        c.lineWidth = 6;
                        const endX = this.facingRight ? this.position.x + this.width : this.position.x;
                        c.arc(endX, this.position.y + this.height/2, (15 - this.life) * 15, 0, Math.PI*2);
                        c.stroke();
                    }
                }
                else if (this.type === 'water') {
                    c.shadowColor = '#38bdf8';
                    
                    // 水滴の残像
                    this.history.forEach((h, index) => {
                        const alpha = index / this.history.length;
                        c.fillStyle = `rgba(56, 189, 248, ${alpha * 0.5})`;
                        c.beginPath();
                        c.arc(h.x + this.width/2, h.y + this.height/2, (this.width/2) * alpha, 0, Math.PI*2);
                        c.fill();
                    });

                    // 立体的な水球のグラデーション
                    const grad = c.createRadialGradient(
                        this.position.x + this.width/3, this.position.y + this.height/3, 2,
                        this.position.x + this.width/2, this.position.y + this.height/2, this.width/2
                    );
                    grad.addColorStop(0, '#e0f2fe'); // ハイライト
                    grad.addColorStop(0.5, '#0ea5e9'); // メイン
                    grad.addColorStop(1, '#0369a1'); // 影
                    
                    c.fillStyle = grad;
                    c.beginPath();
                    c.arc(this.position.x + this.width/2, this.position.y + this.height/2, this.width/2, 0, Math.PI*2);
                    c.fill();
                }
                
                c.shadowBlur = 0; // リセット
                c.restore();
            }

            update() {
                // 残像用に位置を記録
                this.history.push({x: this.position.x, y: this.position.y});
                if(this.history.length > 6) this.history.shift();

                this.draw();
                this.position.x += this.velocity.x;
                this.position.y += this.velocity.y;
                this.frames++;

                // 技固有の特殊な動き
                if (this.type === 'bat') {
                    this.position.y += Math.sin(this.frames * 0.5) * 8;
                } else if (this.type === 'water') {
                    this.velocity.y += 0.4; // 重力で落ちる
                    if (this.position.y + this.height > groundLevel) {
                        this.markedForDeletion = true; 
                    }
                } else if (this.type === 'claw' || this.type === 'pole') {
                    this.life--;
                    if(this.life <= 0) this.markedForDeletion = true;
                    
                    // キャラに追従させる
                    if (this.type === 'pole') {
                        this.position.y = this.owner.position.y + 60;
                        if(this.facingRight) {
                            this.position.x = this.owner.position.x + 30;
                        } else {
                            this.position.x = this.owner.position.x - this.width + 30;
                        }
                    } else if (this.type === 'claw') {
                        this.position.y = this.owner.position.y + 10;
                        if(this.facingRight) {
                            this.position.x = this.owner.position.x + 40;
                        } else {
                            this.position.x = this.owner.position.x - this.width + 20;
                        }
                    }
                }

                // 画面外に大きく出たら消す（長めに残す）
                if(this.position.x < -800 || this.position.x > canvas.width + 800) {
                    this.markedForDeletion = true;
                }
            }
        }

        const projectiles = [];

        class Fighter {
            constructor({ position, velocity, charId, isPlayer }) {
                this.position = position;
                this.velocity = velocity;
                // 当たり判定のベースサイズ
                this.width = 60;
                this.height = 110;
                this.isPlayer = isPlayer;
                this.charId = charId;
                
                const data = characterData[charId];
                this.sprite = data.sprite;
                this.speed = data.speed;
                this.jumpPower = data.jump;
                this.attackDamage = data.attackDamage;
                this.spCost = data.spCost;
                this.spType = data.spType;

                this.health = 100;
                this.mana = 100;
                this.facingRight = isPlayer;
                
                // 孫悟空は通常攻撃のリーチが長い
                const attackW = (charId === 'wukong') ? 110 : 70;
                this.attackBox = {
                    position: { x: this.position.x, y: this.position.y },
                    width: attackW,
                    height: 50
                };
                
                this.isAttacking = false;
                this.attackCooldown = false;
            }

            draw() {
                // スプライト描画 (scale: 10) -> 120x120のサイズになる
                // 少しX座標を左にずらして描画し、当たり判定(幅60)の中央に見えるようにする
                drawPixelArt(c, this.sprite, this.position.x - 30, this.position.y - 10, 10, this.facingRight);

                // 攻撃判定の可視化（パンチ中のみ白枠）
                if (this.isAttacking) {
                    c.strokeStyle = 'white';
                    c.lineWidth = 2;
                    c.strokeRect(this.attackBox.position.x, this.attackBox.position.y, this.attackBox.width, this.attackBox.height);
                }
            }

            update() {
                this.draw();

                // 攻撃判定ボックスの追従
                this.attackBox.position.y = this.position.y + 30;
                if (this.facingRight) {
                    this.attackBox.position.x = this.position.x + this.width;
                } else {
                    this.attackBox.position.x = this.position.x - this.attackBox.width;
                }

                // 移動と重力
                this.position.x += this.velocity.x;
                this.position.y += this.velocity.y;

                if (this.position.y + this.height + this.velocity.y >= groundLevel) {
                    this.velocity.y = 0;
                    this.position.y = groundLevel - this.height;
                } else {
                    this.velocity.y += gravity;
                }

                // 画面端
                if(this.position.x < 0) this.position.x = 0;
                if(this.position.x + this.width > canvas.width) this.position.x = canvas.width - this.width;

                // MP回復
                if(this.mana < 100) {
                    this.mana += 0.25;
                    if(this.mana > 100) this.mana = 100;
                    if(this.isPlayer) {
                        document.querySelector('#player-mp').style.width = this.mana + '%';
                    }
                }
            }

            attack() {
                if(this.attackCooldown) return;
                this.isAttacking = true;
                this.attackCooldown = true;
                
                setTimeout(() => { this.isAttacking = false; }, 100);
                setTimeout(() => { this.attackCooldown = false; }, 400); // 攻撃間隔
            }

            useSuperPower() {
                if(this.mana >= this.spCost) {
                    this.mana -= this.spCost;
                    if(this.isPlayer) document.querySelector('#player-mp').style.width = this.mana + '%';
                    
                    let projVelX = this.facingRight ? 10 : -10;
                    
                    // 発動位置を少し内側（キャラ寄り）にして近距離でもエフェクトが発生・当たるようにする
                    let startX = this.facingRight ? this.position.x + this.width / 2 : this.position.x - 30 + this.width / 2;
                    
                    // 技によって弾の速度を変える
                    if(this.spType === 'tornado') projVelX = this.facingRight ? 5 : -5;
                    if(this.spType === 'wave') projVelX = this.facingRight ? 15 : -15;

                    projectiles.push(new Projectile({
                        position: { x: startX, y: this.position.y + 20 },
                        velocity: { x: projVelX, y: 0 },
                        facingRight: this.facingRight,
                        type: this.spType,
                        owner: this // 誰が放ったかを記録
                    }));
                }
            }

            // 吸血鬼のパッシブスキル（HP回復）
            heal(amount) {
                if(this.charId === 'vampire') {
                    this.health += amount;
                    if(this.health > 100) this.health = 100;
                    const uiId = this.isPlayer ? '#player-hp' : '#enemy-hp';
                    document.querySelector(uiId).style.width = this.health + '%';
                }
            }
        }

        // --- キー操作イベント ---
        window.addEventListener('keydown', (event) => {
            if(gameState !== 'PLAYING') return;
            switch (event.key.toLowerCase()) {
                case 'd': keys.d.pressed = true; player.facingRight = true; break;
                case 'a': keys.a.pressed = true; player.facingRight = false; break;
                case 'w': if(player.velocity.y === 0) player.velocity.y = player.jumpPower; break;
                case 'j': player.attack(); break;
                case 'k': player.useSuperPower(); break;
            }
        });
        window.addEventListener('keyup', (event) => {
            switch (event.key.toLowerCase()) {
                case 'd': keys.d.pressed = false; break;
                case 'a': keys.a.pressed = false; break;
            }
        });

        // --- ゲームの進行管理 ---
        let player, enemy, timer = 60, timerId;
        const titleScreen = document.getElementById('title-screen');
        const selectScreen = document.getElementById('character-select');
        const gameUI = document.getElementById('game-ui');
        const resultScreen = document.getElementById('result');

        // タイトル画面からキャラクター選択への遷移
        document.getElementById('to-select-btn').addEventListener('click', () => {
            titleScreen.classList.add('opacity-0', 'pointer-events-none');
            setTimeout(() => {
                titleScreen.classList.add('hidden');
                selectScreen.classList.remove('hidden');
                gameState = 'SELECT';
            }, 500);
        });

        function initSelectScreen() {
            const container = document.getElementById('char-list');
            Object.keys(characterData).forEach(id => {
                const char = characterData[id];
                const card = document.createElement('div');
                card.className = 'p-4 border-4 border-gray-600 bg-gray-800 rounded cursor-pointer hover:border-gray-300 transition-all transform hover:-translate-y-2 flex flex-col items-center';
                
                // サムネイル用キャンバス
                const thumbCanvasId = `thumb-${id}`;
                card.innerHTML = `
                    <div class="bg-gray-700 rounded mb-3 p-2 w-full flex justify-center shadow-inner border-2 border-gray-900">
                        <canvas id="${thumbCanvasId}" width="80" height="80"></canvas>
                    </div>
                    <h2 class="text-xl font-bold mb-1 text-yellow-300 pixel-text">${char.name}</h2>
                    <p class="text-[10px] text-gray-300 mb-3 h-8 text-center leading-tight">${char.desc}</p>
                    <div class="w-full text-xs">
                        <div class="flex justify-between bg-gray-900 p-1 mb-1"><span class="text-gray-400">移動</span><span class="text-green-400">${char.speed}</span></div>
                        <div class="flex justify-between bg-gray-900 p-1 mb-1"><span class="text-gray-400">攻撃</span><span class="text-red-400">${char.attackDamage}</span></div>
                        <div class="flex justify-between bg-gray-900 p-1"><span class="text-gray-400">必殺技</span><span class="text-cyan-400 text-[10px]">${char.spType.toUpperCase()}</span></div>
                    </div>
                `;
                
                card.addEventListener('click', () => {
                    document.querySelectorAll('#char-list > div').forEach(c => {
                        c.classList.remove('border-yellow-400', 'scale-105', 'bg-gray-700');
                        c.classList.add('border-gray-600', 'bg-gray-800');
                    });
                    card.classList.add('border-yellow-400', 'scale-105', 'bg-gray-700');
                    card.classList.remove('border-gray-600', 'bg-gray-800');
                    
                    selectedCharId = id;
                    const btn = document.getElementById('start-btn');
                    btn.disabled = false;
                    btn.classList.remove('bg-gray-600', 'text-gray-300', 'border-gray-500', 'cursor-not-allowed', 'opacity-50');
                    btn.classList.add('bg-red-600', 'hover:bg-red-500', 'text-white', 'border-red-800');
                    btn.innerText = 'バトル開始！';
                });
                
                container.appendChild(card);

                // サムネイルキャンバスにドット絵を描画
                const tCanvas = document.getElementById(thumbCanvasId);
                const tCtx = tCanvas.getContext('2d');
                // 12x12をscale=6で描画すると72x72。中央に配置。
                drawPixelArt(tCtx, char.sprite, 4, 4, 6, true);
            });

            document.getElementById('start-btn').addEventListener('click', startGame);
        }

        let selectedCharId = null;

        function startGame() {
            gameState = 'PLAYING';
            selectScreen.classList.add('opacity-0', 'pointer-events-none');
            setTimeout(() => selectScreen.classList.add('hidden'), 500);
            gameUI.classList.remove('hidden');
            canvas.focus();

            player = new Fighter({ position: { x: 150, y: 0 }, velocity: { x: 0, y: 0 }, charId: selectedCharId, isPlayer: true });
            
            const charIds = Object.keys(characterData);
            // 敵は自分以外のランダム
            let randomCharId;
            do { randomCharId = charIds[Math.floor(Math.random() * charIds.length)]; } while (randomCharId === selectedCharId);
            
            enemy = new Fighter({ position: { x: 800, y: 0 }, velocity: { x: 0, y: 0 }, charId: randomCharId, isPlayer: false });

            // UIテキスト更新
            document.getElementById('player-name').innerText = characterData[selectedCharId].name;
            document.getElementById('enemy-name').innerText = characterData[randomCharId].name;

            decreaseTimer();
        }

        function decreaseTimer() {
            if (timer > 0 && gameState === 'PLAYING') {
                timerId = setTimeout(decreaseTimer, 1000);
                timer--;
                document.getElementById('timer').innerText = timer;
            }
            if (timer === 0) determineWinner();
        }

        function determineWinner() {
            clearTimeout(timerId);
            gameState = 'GAMEOVER';
            resultScreen.style.display = 'flex';
            const resultText = document.getElementById('result-text');
            const restartBtn = document.getElementById('restart-btn');
            
            if (player.health === enemy.health) {
                resultText.innerHTML = 'DRAW';
                resultText.className = 'text-5xl md:text-7xl font-bold text-yellow-400 mb-8';
            } else if (player.health > enemy.health) { 
                resultText.innerHTML = 'YOU WIN!'; 
                resultText.className = 'text-5xl md:text-7xl font-bold text-blue-400 mb-8';
            } else { 
                resultText.innerHTML = 'YOU LOSE...'; 
                resultText.className = 'text-5xl md:text-7xl font-bold text-red-500 mb-8';
            }

            // 少し余韻を残すために1秒後に戻るボタンを表示
            setTimeout(() => {
                restartBtn.classList.remove('hidden');
            }, 1000);
        }

        function resetGame() {
            // リザルトとゲームUIを隠す
            resultScreen.style.display = 'none';
            document.getElementById('restart-btn').classList.add('hidden');
            gameUI.classList.add('hidden');
            
            // キャラクター選択画面を再表示
            selectScreen.classList.remove('hidden', 'opacity-0', 'pointer-events-none');
            
            // ゲームの状態とタイマーを初期化
            gameState = 'SELECT';
            timer = 60;
            document.getElementById('timer').innerText = timer;
            
            // キャラクターや画面上の飛び道具を削除
            player = null;
            enemy = null;
            projectiles.length = 0;

            // UIのゲージをすべて100%に戻す
            document.getElementById('player-hp').style.width = '100%';
            document.getElementById('player-mp').style.width = '100%';
            document.getElementById('enemy-hp').style.width = '100%';
        }
        
        // ボタンにリセット処理を紐付け
        document.getElementById('restart-btn').addEventListener('click', resetGame);

        function rectangularCollision({ rect1, rect2 }) {
            return (
                rect1.position.x + rect1.width >= rect2.position.x &&
                rect1.position.x <= rect2.position.x + rect2.width &&
                rect1.position.y + rect1.height >= rect2.position.y &&
                rect1.position.y <= rect2.position.y + rect2.height
            );
        }

        function enemyAI() {
            if(gameState !== 'PLAYING') { if(enemy) enemy.velocity.x = 0; return; }

            const distanceX = player.position.x - enemy.position.x;
            enemy.facingRight = distanceX > 0;

            if (Math.abs(distanceX) > (enemy.charId === 'wukong' ? 100 : 70)) {
                enemy.velocity.x = enemy.facingRight ? enemy.speed * 0.5 : -enemy.speed * 0.5;
            } else {
                enemy.velocity.x = 0;
                if(Math.random() < 0.05) enemy.attack();
            }
            
            // 敵もたまに必殺技を使う
            if(Math.random() < 0.01 && enemy.mana >= enemy.spCost) {
                enemy.useSuperPower();
            }

            if(Math.random() < 0.01 && enemy.velocity.y === 0) enemy.velocity.y = enemy.jumpPower;
        }

        // --- メインループ ---
        function animate() {
            window.requestAnimationFrame(animate);

            // 背景描画（夜の森風）
            c.fillStyle = '#0f172a'; c.fillRect(0, 0, canvas.width, canvas.height);
            // 簡易的な月
            c.fillStyle = '#fef08a'; c.beginPath(); c.arc(800, 150, 60, 0, Math.PI*2); c.fill();
            // 地面
            c.fillStyle = '#1e293b'; c.fillRect(0, groundLevel, canvas.width, canvas.height - groundLevel);
            c.strokeStyle = '#475569'; c.lineWidth = 5; c.beginPath(); c.moveTo(0, groundLevel); c.lineTo(canvas.width, groundLevel); c.stroke();

            if (gameState === 'SELECT' || gameState === 'TITLE') return;

            if (player) player.update();
            if (enemy) enemy.update();

            player.velocity.x = 0;
            if(gameState === 'PLAYING') {
                if (keys.a.pressed && player.position.x > 0) player.velocity.x = -player.speed;
                else if (keys.d.pressed && player.position.x + player.width < canvas.width) player.velocity.x = player.speed;
            }

            enemyAI();

            // 飛び道具の処理
            for (let i = projectiles.length - 1; i >= 0; i--) {
                const p = projectiles[i];
                p.update();

                // 敵と味方、どちらに当たるべきかの判定
                const target = p.owner.isPlayer ? enemy : player;

                // hasHitフラグを見て、1発の弾につき1回だけダメージを与える（貫通してエフェクトは残る）
                if (!p.hasHit && !p.markedForDeletion && rectangularCollision({
                    rect1: { position: p.position, width: p.width, height: p.height },
                    rect2: target
                })) {
                    p.hasHit = true; // 当たったフラグを立てる（消さない）

                    if(gameState === 'PLAYING') {
                        target.health -= p.damage;
                        document.querySelector(target.isPlayer ? '#player-hp' : '#enemy-hp').style.width = Math.max(0, target.health) + '%';
                        p.owner.heal(5); // 飛び道具ヒット時も吸血鬼は回復
                    }
                }

                if (p.markedForDeletion) projectiles.splice(i, 1);
            }

            if(gameState === 'GAMEOVER') return;

            // 近接攻撃の当たり判定
            if (player.isAttacking && rectangularCollision({ rect1: player.attackBox, rect2: enemy })) {
                player.isAttacking = false; 
                enemy.health -= player.attackDamage;
                document.querySelector('#enemy-hp').style.width = Math.max(0, enemy.health) + '%';
                player.heal(3); // 吸血鬼なら回復
            }

            if (enemy.isAttacking && rectangularCollision({ rect1: enemy.attackBox, rect2: player })) {
                enemy.isAttacking = false;
                player.health -= enemy.attackDamage;
                document.querySelector('#player-hp').style.width = Math.max(0, player.health) + '%';
                enemy.heal(3);
            }

            if (enemy.health <= 0 || player.health <= 0) determineWinner();
        }

        initSelectScreen();
        animate();
    </script>
</body>
</html>
