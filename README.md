# Gta
Shooter
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Уличный Шутер: 128-бит Стиль</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        // Обязательные импорты для работы в среде Canvas (оставлены для совместимости)
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Глобальные переменные, предоставленные средой
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let app, db, auth, userId;

        if (Object.keys(firebaseConfig).length > 0) {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            setLogLevel('Debug');
            
            // Аутентификация
            window.onload = async () => {
                try {
                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                    userId = auth.currentUser?.uid || 'anonymous-user';
                    startGameAfterInit();
                } catch (error) {
                    console.error("Firebase Auth failed:", error);
                    startGameAfterInit();
                }
            };
        } else {
            window.onload = () => {
                startGameAfterInit();
            };
        }
    </script>
    <style>
        /* Шрифт, более современный, но все еще агрессивный */
        @import url('https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@700&display=swap');
        
        body {
            font-family: 'Roboto Mono', monospace;
            background-color: #1a1a2e; /* Темный фон для gritty атмосферы */
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 10px;
        }

        #game-container {
            width: 100%;
            max-width: 700px; /* Увеличиваем размер для "высокого разрешения" */
            background: #000;
            border: 4px solid #333; 
            border-radius: 10px;
            box-shadow: 0 0 40px rgba(0, 0, 0, 0.9); /* Более глубокая тень */
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        #game-info {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 20px;
            background: #111;
            border-bottom: 2px solid #5a5a5a;
            font-size: 1.2rem;
            font-weight: 700;
            color: #ccc;
        }

        #weapon-display {
            font-size: 1.1rem;
            color: #FFEA00;
            text-shadow: 0 0 8px #FFEA00; /* Эффект свечения */
            padding: 4px 8px;
            border: 1px solid #FFEA00;
            border-radius: 3px;
            text-align: center;
        }

        #health-display span {
            color: #C00000; 
            text-shadow: 0 0 6px #FF0000;
        }

        canvas {
            display: block;
            background-color: #000000;
            touch-action: none; 
            border-radius: 0 0 6px 6px;
            /* Сглаживание и контраст для 128-бит */
            filter: contrast(1.2); 
        }
        
        /* Стили для меню */
        #menu-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.95);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            text-align: center;
        }

        #menu-content {
            background: #111111;
            padding: 40px 50px;
            border-radius: 15px;
            border: 3px solid #666; 
            box-shadow: 0 0 30px rgba(255, 255, 255, 0.1);
        }

        #start-button {
            background-color: #38a169; /* Современный, глубокий зеленый */
            color: #fff;
            padding: 15px 40px;
            margin-top: 25px;
            border: none;
            border-radius: 8px;
            font-size: 1.6rem;
            font-weight: 700;
            cursor: pointer;
            transition: background-color 0.2s, transform 0.1s;
            box-shadow: 0 5px #278d58; 
            text-transform: uppercase;
        }

        #start-button:hover { background-color: #48b179; }
        #start-button:active { box-shadow: 0 2px #278d58; transform: translateY(3px); }
    </style>
</head>
<body>
    <div id="game-container">
        <!-- Информационная панель -->
        <div id="game-info">
            <div id="weapon-display">ОРУЖИЕ: УЗИ</div>
            <div id="score-display">ОЧКИ: 0</div>
            <div id="health-display">ЖИЗНИ: <span id="health-value">3</span></div>
        </div>
        
        <!-- Холст для игры -->
        <canvas id="gameCanvas"></canvas>

        <!-- Меню (Начало/Конец игры) -->
        <div id="menu-overlay">
            <div id="menu-content">
                <h1 class="text-5xl font-extrabold mb-4 text-gray-200">GANG WARS 128</h1>
                <p class="mb-4 text-xl text-[#38a169]">ЭРА PS2: ДЕТАЛЬНЫЕ МОДЕЛИ</p>
                <p class="mb-8 text-lg text-gray-400">ВЫСОКАЯ ДЕТАЛИЗАЦИЯ И ГРЯЗНЫЕ УЛИЦЫ</p>
                <ul class="list-disc list-inside text-left mx-auto max-w-xs mb-8 text-white">
                    <li>МОБИЛЬНЫЙ: ПЕРЕТАСКИВАНИЕ (СВАЙП).</li>
                    <li>DESKTOP: КЛАВИАТУРА ← → ИЛИ МЫШЬ.</li>
                    <li>СТРЕЛЬБА АВТОМАТИЧЕСКАЯ.</li>
                </ul>
                <button id="start-button">НАЧАТЬ МИССИЮ</button>
                <div id="final-score" class="mt-8 text-3xl font-extrabold hidden text-yellow-400"></div>
                <div id="message" class="mt-4 text-xl hidden text-red-500"></div>
            </div>
        </div>
    </div>

    <script>
        // === Глобальные переменные и константы ===
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score-display');
        const healthValueDisplay = document.getElementById('health-value');
        const weaponDisplay = document.getElementById('weapon-display');
        const menuOverlay = document.getElementById('menu-overlay');
        const startButton = document.getElementById('start-button');
        const finalScoreDisplay = document.getElementById('final-score');
        const messageDisplay = document.getElementById('message');

        // 128-битная палитра (более глубокие и реалистичные тона)
        const C_GROVE_GREEN = '#1F803A'; // Темно-зеленый
        const C_GROVE_SHADOW = '#0E401D';
        const C_BALLAS_PURPLE = '#6F008F'; // Глубокий фиолетовый
        const C_BALLAS_SHADOW = '#400055';
        const C_SKIN_128 = '#C7906E'; // Более сложный тон кожи
        const ENEMY_SPAWN_INTERVAL = 1200; 
        const WEAPON_UPGRADE_SCORE = 20;

        // Конфигурации оружия
        const WEAPONS = [
            { name: "ПИСТОЛЕТ-ПУЛЕМЕТ", type: "UZI", bulletConfig: { width: 8, height: 30, speed: 20, color: '#FFD700', fireCooldown: 150 } },
            { name: "ДРОБОВИК", type: "SHOTGUN", bulletConfig: { width: 6, height: 20, speed: 17, color: '#C0C0C0', fireCooldown: 400 } },
            { name: "ГРАНАТОМЕТ", type: "ROCKET", bulletConfig: { width: 30, height: 45, speed: 12, color: '#FF4400', fireCooldown: 1200 } }
        ];

        let game;
        let lastSpawnTime = 0;
        let keys = {};
        let touchLastX = null; 

        // === Классы игровых объектов ===

        class Player {
            constructor() {
                this.width = 80; // Более крупная модель
                this.height = 90;
                this.x = canvas.width / 2 - this.width / 2;
                this.y = canvas.height - this.height - 50;
                this.speed = 10; 
                this.health = 3;
                this.canFire = true;
                this.currentWeaponIndex = 0;
                this.weapon = WEAPONS[this.currentWeaponIndex];
            }
            
            // Смена оружия
            upgradeWeapon(score) {
                this.currentWeaponIndex = Math.floor(score / WEAPON_UPGRADE_SCORE) % WEAPONS.length;
                this.weapon = WEAPONS[this.currentWeaponIndex];
                weaponDisplay.textContent = `ОРУЖИЕ: ${this.weapon.name}`;
            }

            // Вспомогательный метод для отрисовки оружия игрока
            drawWeapon(x, y, w, h) {
                ctx.fillStyle = '#444444'; 
                const weaponX = x + w * 0.55;
                const weaponY = y + h * 0.45;

                if (this.weapon.type === 'UZI') {
                    // ПП: Ствол + Магазин + Рукоятка
                    ctx.fillRect(weaponX, weaponY, 40, 8); // Ствол
                    ctx.fillRect(weaponX + 5, weaponY + 8, 5, 15); // Магазин
                    ctx.fillRect(weaponX - 5, weaponY + 3, 10, 10); // Рукоятка
                } else if (this.weapon.type === 'SHOTGUN') {
                    // Дробовик: Длинный ствол + Помпа
                    ctx.fillRect(weaponX - 5, weaponY, 50, 12); // Ствол
                    ctx.fillStyle = '#666';
                    ctx.fillRect(weaponX + 15, weaponY + 12, 10, 5); // Помпа
                } else if (this.weapon.type === 'ROCKET') {
                    // Гранатомет: Объемный + прицел
                    ctx.fillStyle = '#222';
                    ctx.fillRect(weaponX - 10, weaponY - 10, 60, 20);
                    ctx.fillStyle = '#666'; // Прицел
                    ctx.fillRect(weaponX + 10, weaponY - 15, 5, 5); 
                    ctx.fillRect(weaponX + 10, weaponY + 10, 5, 5); 
                }
            }

            // Отрисовка Игрока (детализированная 128-битная модель)
            draw() {
                const x = this.x;
                const y = this.y;
                const w = this.width;
                const h = this.height;

                // --- ЦВЕТА И ТЕНИ ---
                const primaryColor = C_GROVE_GREEN;
                const shadowColor = C_GROVE_SHADOW;
                const detailColor = '#111'; // Черный для деталей

                // --- 1. НОГИ (LEGS) ---
                const legW = w * 0.3;
                const legH = h * 0.3;
                const legY = y + h * 0.65;
                
                // Левая нога (shadowed)
                ctx.fillStyle = shadowColor;
                ctx.fillRect(x + w * 0.2, legY, legW, legH);
                // Правая нога (lighter/primary)
                ctx.fillStyle = primaryColor;
                ctx.fillRect(x + w * 0.5, legY, legW, legH);

                // --- 2. ОБУВЬ (SHOES/BOOTS) ---
                ctx.fillStyle = detailColor;
                ctx.fillRect(x + w * 0.15, y + h * 0.9, legW * 1.2, h * 0.1);
                ctx.fillRect(x + w * 0.45, y + h * 0.9, legW * 1.2, h * 0.1);

                // --- 3. ТОРС (TORSO/SHIRT) ---
                const torsoW = w * 0.8;
                const torsoH = h * 0.4;
                const torsoY = y + h * 0.3;
                
                // Основа торса (Градиент для объема)
                const bodyGradient = ctx.createLinearGradient(x, torsoY, x + w, torsoY);
                bodyGradient.addColorStop(0, shadowColor);
                bodyGradient.addColorStop(0.5, primaryColor);
                bodyGradient.addColorStop(1, shadowColor);
                ctx.fillStyle = bodyGradient;
                ctx.fillRect(x + w * 0.1, torsoY, torsoW, torsoH); 

                // --- 4. РУКИ (ARMS) ---
                const armW = 15;
                const armH = 35;
                const armY = torsoY + 5;

                // Левая рука (тянущаяся вперед)
                ctx.fillStyle = C_SKIN_128; // Кожа
                ctx.fillRect(x + w * 0.1, armY, armW, armH);
                ctx.fillStyle = detailColor; // Рукав
                ctx.fillRect(x + w * 0.1, armY, armW, 10);

                // Правая рука (на оружии)
                ctx.fillStyle = C_SKIN_128; // Кожа
                ctx.fillRect(x + w - armW * 1.5, armY, armW, armH);
                ctx.fillStyle = detailColor; // Рукав
                ctx.fillRect(x + w - armW * 1.5, armY, armW, 10);

                // --- 5. ГОЛОВА и ШЕЯ (HEAD AND NECK) ---
                ctx.fillStyle = C_SKIN_128; 
                
                // Шея
                ctx.fillRect(x + w * 0.45, y + h * 0.25, w * 0.1, h * 0.05);

                // Голова (более круглая/сглаженная)
                ctx.beginPath();
                ctx.ellipse(x + w * 0.5, y + h * 0.18, w * 0.3, h * 0.12, 0, 0, 2 * Math.PI);
                ctx.fill();
                
                // --- 6. КЕПКА/ВОЛОСЫ (CAP/HAIR) ---
                ctx.fillStyle = primaryColor;
                ctx.fillRect(x + w * 0.2, y, w * 0.6, h * 0.1); // Верх
                ctx.fillRect(x + w * 0.15, y + h * 0.08, w * 0.7, h * 0.02); // Козырек

                // --- 7. ОРУЖИЕ (WEAPON) ---
                this.drawWeapon(x, y, w, h);
            }

            update() {
                // Обновление движения... (логика та же)
                if (keys['ArrowLeft'] || keys['a']) {
                    this.x -= this.speed;
                }
                if (keys['ArrowRight'] || keys['d']) {
                    this.x += this.speed;
                }
                
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvas.width) this.x = canvas.width - this.width;

                this.shoot(); 
            }

            shoot() {
                // Логика стрельбы... (та же)
                if (this.canFire) {
                    const bulletConfig = this.weapon.bulletConfig;
                    const centerX = this.x + this.width * 0.65;
                    const startY = this.y + this.height * 0.4;
                    
                    if (this.weapon.type === 'UZI') {
                        game.bullets.push(new Bullet(centerX, startY, bulletConfig));
                    } else if (this.weapon.type === 'SHOTGUN') {
                        // Стрельба веером (3 пули)
                        game.bullets.push(new Bullet(centerX - 10, startY, bulletConfig, -0.4));
                        game.bullets.push(new Bullet(centerX, startY, bulletConfig, 0));
                        game.bullets.push(new Bullet(centerX + 10, startY, bulletConfig, 0.4));
                    } else if (this.weapon.type === 'ROCKET') {
                        game.bullets.push(new Bullet(centerX - 10, startY - 10, bulletConfig));
                    }

                    this.canFire = false;
                    setTimeout(() => {
                        this.canFire = true;
                    }, bulletConfig.fireCooldown);
                }
            }
        }

        class Bullet {
            constructor(x, y, config, angleOffset = 0) {
                this.x = x;
                this.y = y;
                this.width = config.width; 
                this.height = config.height;
                this.speed = config.speed; 
                this.color = config.color;
                this.angleOffset = angleOffset;
            }

            // Отрисовка Снаряда (с эффектом свечения Bloom)
            draw() {
                // 1. Эффект свечения (Bloom / Glow)
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;

                // 2. Основное тело снаряда
                const gradient = ctx.createLinearGradient(this.x, this.y, this.x, this.y + this.height);
                gradient.addColorStop(0, '#FFFFFF'); // Блик
                gradient.addColorStop(1, this.color);
                ctx.fillStyle = gradient;
                
                if (this.width > 20) { // Ракета
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                    // Дым/Огонь сзади
                    ctx.shadowBlur = 5;
                    ctx.fillStyle = '#FF8800';
                    ctx.beginPath();
                    ctx.arc(this.x + this.width / 2, this.y + this.height, this.width * 0.6, 0, Math.PI * 2);
                    ctx.fill();
                } else { // Пули
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                }
                
                // Сброс теней
                ctx.shadowBlur = 0;
                ctx.shadowColor = 'transparent';
            }

            update() {
                this.y -= this.speed;
                this.x += this.angleOffset * 5; 
            }
        }

        class Enemy {
            constructor() {
                this.width = 75;
                this.height = 85;
                this.x = Math.random() * (canvas.width - this.width);
                this.y = -this.height;
                this.baseSpeed = 2.5;
                this.speed = this.baseSpeed + Math.floor(game.score / 10) * 0.3;
            }

            // Отрисовка Врага (детализированная 128-битная модель - Ballas)
            draw() {
                const x = this.x;
                const y = this.y;
                const w = this.width;
                const h = this.height;

                // --- ЦВЕТА И ТЕНИ ---
                const primaryColor = C_BALLAS_PURPLE;
                const shadowColor = C_BALLAS_SHADOW;
                const detailColor = '#111'; // Черный для деталей
                const accessoryColor = '#FF0000'; // Красная бандана

                // --- 1. НОГИ (LEGS) ---
                const legW = w * 0.3;
                const legH = h * 0.3;
                const legY = y + h * 0.65;
                
                // Левая нога (shadowed)
                ctx.fillStyle = shadowColor;
                ctx.fillRect(x + w * 0.2, legY, legW, legH);
                // Правая нога (lighter/primary)
                ctx.fillStyle = primaryColor;
                ctx.fillRect(x + w * 0.5, legY, legW, legH);

                // --- 2. ОБУВЬ (SHOES/BOOTS) ---
                ctx.fillStyle = detailColor;
                ctx.fillRect(x + w * 0.15, y + h * 0.9, legW * 1.2, h * 0.1);
                ctx.fillRect(x + w * 0.45, y + h * 0.9, legW * 1.2, h * 0.1);

                // --- 3. ТОРС (TORSO/SHIRT) ---
                const torsoW = w * 0.8;
                const torsoH = h * 0.4;
                const torsoY = y + h * 0.3;
                
                // Основа торса (Градиент для объема)
                const bodyGradient = ctx.createLinearGradient(x, torsoY, x + w, torsoY);
                bodyGradient.addColorStop(0, shadowColor);
                bodyGradient.addColorStop(0.5, primaryColor);
                bodyGradient.addColorStop(1, shadowColor);
                ctx.fillStyle = bodyGradient;
                ctx.fillRect(x + w * 0.1, torsoY, torsoW, torsoH); 

                // --- 4. РУКИ (ARMS) ---
                const armW = 15;
                const armH = 35;
                const armY = torsoY + 5;

                // Левая рука (тянущаяся вперед)
                ctx.fillStyle = C_SKIN_128; // Кожа
                ctx.fillRect(x + w * 0.1, armY, armW, armH);
                ctx.fillStyle = detailColor; // Рукав
                ctx.fillRect(x + w * 0.1, armY, armW, 10);

                // Правая рука (на оружии)
                ctx.fillStyle = C_SKIN_128; // Кожа
                ctx.fillRect(x + w - armW * 1.5, armY, armW, armH);
                ctx.fillStyle = detailColor; // Рукав
                ctx.fillRect(x + w - armW * 1.5, armY, armW, 10);

                // --- 5. ГОЛОВА и ШЕЯ (HEAD AND NECK) ---
                ctx.fillStyle = C_SKIN_128; 
                
                // Шея
                ctx.fillRect(x + w * 0.45, y + h * 0.25, w * 0.1, h * 0.05);

                // Голова (более круглая/сглаженная)
                ctx.beginPath();
                ctx.ellipse(x + w * 0.5, y + h * 0.18, w * 0.3, h * 0.12, 0, 0, 2 * Math.PI);
                ctx.fill();
                
                // --- 6. БАНДАНА/ГОЛОВНОЙ УБОР (BANDANA/HEADGEAR) ---
                ctx.fillStyle = accessoryColor; 
                ctx.beginPath();
                ctx.moveTo(x + w * 0.1, y + h * 0.1);
                ctx.lineTo(x + w * 0.9, y + h * 0.1);
                ctx.lineTo(x + w * 0.5, y + h * 0.3);
                ctx.closePath();
                ctx.fill();

                // --- 7. ОРУЖИЕ (WEAPON - Enemy UZI) ---
                ctx.fillStyle = '#444444'; 
                const weaponX = x + w * 0.55;
                const weaponY = y + h * 0.45;
                ctx.fillRect(weaponX, weaponY, 40, 8); // Ствол
                ctx.fillRect(weaponX + 5, weaponY + 8, 5, 15); // Магазин
                ctx.fillRect(weaponX - 5, weaponY + 3, 10, 10); // Рукоятка
            }

            update() {
                this.y += this.speed;
            }
        }

        // === Основной объект игры ===
        class Game {
            constructor() {
                this.isRunning = false;
                this.score = 0;
                this.player = new Player();
                this.enemies = [];
                this.bullets = [];
                this.lastUpgradeScore = -1;
            }

            start() {
                this.isRunning = true;
                this.score = 0;
                this.player = new Player();
                this.player.upgradeWeapon(0); 
                this.enemies = [];
                this.bullets = [];
                lastSpawnTime = 0;
                this.updateUI();
                menuOverlay.style.display = 'none';
                this.loop(0);
            }

            gameOver(message) {
                this.isRunning = false;
                messageDisplay.textContent = message;
                finalScoreDisplay.textContent = `ФИНАЛЬНЫЙ СЧЕТ: ${this.score}`;
                finalScoreDisplay.classList.remove('hidden');
                messageDisplay.classList.remove('hidden');
                startButton.textContent = 'НАЧАТЬ СНОВА';
                menuOverlay.style.display = 'flex';
                touchLastX = null;
            }

            updateUI() {
                scoreDisplay.textContent = `ОЧКИ: ${this.score}`;
                healthValueDisplay.textContent = this.player.health;
            }

            spawnEnemy(timestamp) {
                if (timestamp - lastSpawnTime > ENEMY_SPAWN_INTERVAL - Math.min(this.score * 5, 1000)) {
                    this.enemies.push(new Enemy());
                    lastSpawnTime = timestamp;
                }
            }

            update(timestamp) {
                if (!this.isRunning) return;

                this.spawnEnemy(timestamp);

                this.player.update();
                
                // Проверка бонуса оружия
                const currentUpgradeLevel = Math.floor(this.score / WEAPON_UPGRADE_SCORE);
                if (currentUpgradeLevel !== this.lastUpgradeScore) {
                     this.player.upgradeWeapon(this.score);
                     this.lastUpgradeScore = currentUpgradeLevel;
                }

                this.bullets.forEach(bullet => bullet.update());
                this.bullets = this.bullets.filter(bullet => bullet.y > -50 && bullet.x > -50 && bullet.x < canvas.width + 50);

                this.enemies.forEach((enemy, enemyIndex) => {
                    enemy.update();

                    if (enemy.y > canvas.height - 50) { 
                        this.enemies.splice(enemyIndex, 1);
                        this.player.health -= 1;
                        this.updateUI();
                        if (this.player.health <= 0) {
                            this.gameOver('ВАС ПЕРЕИГРАЛИ! ВЫ ПРОДЕРЖАЛИСЬ ДО ' + this.score + ' ОЧКОВ');
                        }
                    }

                    this.bullets.forEach((bullet, bulletIndex) => {
                        if (
                            bullet.x < enemy.x + enemy.width &&
                            bullet.x + bullet.width > enemy.x &&
                            bullet.y < enemy.y + enemy.height &&
                            bullet.y + bullet.height > enemy.y
                        ) {
                            this.enemies.splice(enemyIndex, 1);
                            this.bullets.splice(bulletIndex, 1);
                            this.score += 1;
                            this.updateUI();
                            return; 
                        }
                    });
                });
            }

            // Рисование 128-битного фона (детализированный, сглаженный)
            drawBackground() {
                // 1. Небо (темное, но с градиентом)
                const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height * 0.3);
                skyGradient.addColorStop(0, '#101015');
                skyGradient.addColorStop(1, '#202025');
                ctx.fillStyle = skyGradient;
                ctx.fillRect(0, 0, canvas.width, canvas.height * 0.4);
                
                // 2. Здания (более детализированные силуэты, имитация текстур)
                ctx.fillStyle = '#333333';
                ctx.beginPath();
                ctx.moveTo(0, canvas.height * 0.4);
                ctx.lineTo(0, canvas.height * 0.6);
                ctx.lineTo(canvas.width * 0.15, canvas.height * 0.6);
                ctx.lineTo(canvas.width * 0.25, canvas.height * 0.4);
                ctx.closePath();
                ctx.fill();

                ctx.fillStyle = '#282828';
                ctx.beginPath();
                ctx.moveTo(canvas.width, canvas.height * 0.4);
                ctx.lineTo(canvas.width, canvas.height * 0.65);
                ctx.lineTo(canvas.width * 0.85, canvas.height * 0.65);
                ctx.lineTo(canvas.width * 0.75, canvas.height * 0.4);
                ctx.closePath();
                ctx.fill();

                // 3. Дорога (Мягкие градиенты, имитация асфальта)
                const roadGradient = ctx.createLinearGradient(0, canvas.height * 0.4, 0, canvas.height);
                roadGradient.addColorStop(0, '#303030');
                roadGradient.addColorStop(1, '#151515');
                ctx.fillStyle = roadGradient;
                ctx.fillRect(0, canvas.height * 0.4, canvas.width, canvas.height * 0.6);

                // 4. Тротуар/Бордюр (Мягкое освещение)
                ctx.fillStyle = '#404040';
                ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
                
                // 5. Центральная разметка (Сглаженная перспектива)
                ctx.fillStyle = '#E0E000';
                ctx.shadowBlur = 5;
                ctx.shadowColor = '#E0E000';
                for (let i = 0; i < 6; i++) {
                    const yStart = canvas.height * 0.6 + i * 50;
                    const w = 10 + i * 2; // Расширение к низу
                    const h = 20;
                    ctx.fillRect(canvas.width / 2 - w / 2, yStart, w, h);
                }
                ctx.shadowBlur = 0; // Сброс
            }

            draw() {
                this.drawBackground(); 

                this.player.draw();
                this.bullets.forEach(bullet => bullet.draw());
                this.enemies.forEach(enemy => enemy.draw());
            }

            loop(timestamp) {
                if (!this.isRunning) return;

                this.update(timestamp);
                this.draw();
                requestAnimationFrame(this.loop.bind(this));
            }
        }

        // === Обработчики свайп-управления ===
        // (Оставлены для мобильной совместимости)

        function handleTouchStart(e) {
            if (!game.isRunning) return;
            const touch = e.touches ? e.touches[0] : e;
            touchLastX = touch.clientX;
            e.preventDefault(); 
        }

        function handleTouchMove(e) {
            if (!game.isRunning || touchLastX === null) return;
            const touch = e.touches ? e.touches[0] : e;
            const rect = canvas.getBoundingClientRect();
            
            const currentX = touch.clientX - rect.left;
            const targetX = currentX - game.player.width / 2;
            
            const dampingFactor = 0.4;
            game.player.x += (targetX - game.player.x) * dampingFactor;
            
            if (game.player.x < 0) game.player.x = 0;
            if (game.player.x + game.player.width > canvas.width) game.player.x = canvas.width - game.player.width;

            touchLastX = touch.clientX;
            e.preventDefault(); 
        }

        function handleTouchEnd(e) {
            touchLastX = null;
        }


        // === Инициализация и обработчики событий ===

        function resizeCanvas() {
            const container = document.getElementById('game-container');
            canvas.width = container.clientWidth - 8; 
            canvas.height = Math.min(canvas.width * 1.5, 750); 
            
            if (game && game.player) {
                game.player.y = canvas.height - game.player.height - 50;
                game.player.x = canvas.width / 2 - game.player.width / 2;
            }
        }

        function startGameAfterInit() {
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);

            game = new Game();
            
            document.addEventListener('keydown', (e) => { keys[e.key] = true; });
            document.addEventListener('keyup', (e) => { keys[e.key] = false; });

            startButton.addEventListener('click', () => {
                finalScoreDisplay.classList.add('hidden');
                messageDisplay.classList.add('hidden');
                game.start();
            });

            canvas.addEventListener('touchstart', handleTouchStart);
            canvas.addEventListener('touchmove', handleTouchMove);
            canvas.addEventListener('touchend', handleTouchEnd);
            canvas.addEventListener('touchcancel', handleTouchEnd); 

            canvas.addEventListener('mousedown', handleTouchStart);
            canvas.addEventListener('mousemove', handleTouchMove);
            canvas.addEventListener('mouseup', handleTouchEnd);
            canvas.addEventListener('mouseleave', handleTouchEnd);

            menuOverlay.style.display = 'flex';
        }
    </script>
</body>
</html>

