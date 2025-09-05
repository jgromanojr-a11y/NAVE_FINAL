<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Macknave</title>
    <style>
        /* Estilos CSS do jogo */
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #000;
            font-family: 'Arial', sans-serif;
            color: white;
            touch-action: manipulation;
            user-select: none;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
        }

        #game-container {
            position: absolute;
            width: 100%;
            height: 100%;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        #ui {
            position: absolute;
            bottom: 0;
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 10px;
            box-sizing: border-box;
            user-select: none;
            z-index: 5;
            -webkit-user-select: none;
        }

        #stats {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 600px;
            padding: 5px 20px;
            font-size: 1em;
        }

        #lives-bar {
            background-color: #333;
            border-radius: 10px;
            height: 15px;
            width: 120px;
            margin-right: 10px;
        }

        #lives-bar-fill {
            height: 100%;
            background-color: #e55a5a;
            border-radius: 10px;
            transition: width 0.3s ease;
        }

        #controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
            max-width: 600px;
            padding-top: 10px;
        }

        #joystick {
            width: 130px;
            height: 130px;
            background-color: rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            touch-action: none;
        }

        #stick {
            width: 60px;
            height: 60px;
            background-color: rgba(255, 255, 255, 0.5);
            border-radius: 50%;
            position: absolute;
        }

        #pauseBtn, .actionBtn {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            font-size: 1.2em;
            font-weight: bold;
            color: white;
            border: none;
            cursor: pointer;
            touch-action: manipulation;
            text-shadow: 1px 1px 2px black;
        }

        #specialFireBtn {
            background-color: rgba(0, 150, 255, 0.7);
            border: 2px solid white;
            /* Diminui a fonte para caber no botão */
            font-size: 0.9em;
        }

        #pauseBtn {
            position: absolute;
            top: 10px;
            right: 10px;
            width: 50px;
            height: 50px;
            background-color: rgba(50, 50, 50, 0.7);
            z-index: 6;
        }

        #action-buttons {
            display: flex;
            gap: 10px;
        }

        #fireBtn {
            background-color: rgba(255, 0, 0, 0.7);
        }

        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.5s ease;
            z-index: 10;
        }

        .overlay.show {
            opacity: 1;
            visibility: visible;
        }

        .overlay h1 {
            font-size: 3em;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px #00e0ff;
        }

        .overlay p {
            font-size: 1.5em;
        }

        .message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 3em;
            font-weight: bold;
            color: #fff;
            text-shadow: 2px 2px 5px #000;
            opacity: 0;
            transition: opacity 0.5s ease-in-out;
            pointer-events: none;
            z-index: 20;
        }

        /* Diminui a fonte da mensagem quântica */
        #message.quantum-message {
            font-size: 1.5em; 
        }

        .diff-buttons button, #restartBtn {
            font-size: 1.2em;
            padding: 15px 30px;
            margin: 10px;
            cursor: pointer;
            background-color: #333;
            color: white;
            border: 2px solid #555;
            border-radius: 5px;
            transition: background-color 0.3s;
        }

        .diff-buttons button:hover, #restartBtn:hover {
            background-color: #555;
        }

        .diff-buttons {
            display: flex;
            flex-direction: column;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="game"></canvas>
    </div>

    <div id="message" class="message-box"></div>

    <button id="pauseBtn">⏸</button>

    <div id="ui">
        <div id="stats">
            <div id="lives">Vidas: 10</div>
            <div id="score">Score: 0</div>
            <div id="phase">Fase: 1</div>
            <div id="hits">Tiros Recebidos: 0</div>
            <div id="collisions">Colisões: 0</div>
        </div>
        <div id="controls">
            <div id="joystick">
                <div id="stick"></div>
            </div>
            <div id="action-buttons">
              <button id="fireBtn" class="actionBtn">FOGO</button>
              <button id="specialFireBtn" class="actionBtn">QUÂNTICO</button>
            </div>
        </div>
    </div>

    <div id="startOverlay" class="overlay show">
        <h1>Macknave</h1>
        <p>A Batalha Estelar de Solaris</p>
        <button id="startBtn">Iniciar Jogo</button>
    </div>

    <div id="gameOver" class="overlay">
        <h1>Game Over</h1>
        <p id="finalScore">Score: 0</p>
        <button id="restartBtn">Tente Novamente</button>
    </div>

    <script>
        (() => {
            // ===== Canvas (HiDPI) =====
            const canvas = document.getElementById('game');
            const ctx = canvas.getContext('2d');
            let W = 0, H = 0, DPR = 1;
            function resize() {
                DPR = Math.max(1, Math.floor(window.devicePixelRatio || 1));
                W = Math.floor(window.innerWidth * DPR);
                H = Math.floor(window.innerHeight * DPR);
                canvas.width = W;
                canvas.height = H;
                canvas.style.width = (W / DPR) + 'px';
                canvas.style.height = (H / DPR) + 'px';
            }
            window.addEventListener('resize', resize, { passive: true });
            resize();

            // ===== UI Elementos =====
            const scoreEl = document.getElementById('score');
            const livesEl = document.getElementById('lives');
            const phaseEl = document.getElementById('phase');
            const hitsEl = document.getElementById('hits');
            const collisionsEl = document.getElementById('collisions');
            const pauseBtn = document.getElementById('pauseBtn');
            const startOverlay = document.getElementById('startOverlay');
            const startBtn = document.getElementById('startBtn');
            const gameOver = document.getElementById('gameOver');
            const finalScore = document.getElementById('finalScore');
            const restartBtn = document.getElementById('restartBtn');
            const messageEl = document.getElementById('message');

            startBtn.addEventListener('click', () => {
                startGame();
            });

            restartBtn.addEventListener('click', () => {
                startOverlay.classList.add('show');
                gameOver.classList.remove('show');
                state.running = false;
            });

            pauseBtn.addEventListener('click', () => {
                if (!state.running) return;
                state.paused = !state.paused;
                pauseBtn.textContent = state.paused ? '▶️' : '⏸';
            });

            // ===== Mensagens na tela =====
            function showMessage(text, duration = 1000, isQuantum = false) {
                messageEl.textContent = text;
                messageEl.style.opacity = '1';
                // Adiciona ou remove a classe para mudar o tamanho da fonte
                if (isQuantum) {
                    messageEl.classList.add('quantum-message');
                } else {
                    messageEl.classList.remove('quantum-message');
                }
                setTimeout(() => {
                    messageEl.style.opacity = '0';
                }, duration);
            }

            // ===== WebAudio (sons) =====
            let audioCtx = null;
            function unlockAudio() {
                if (!audioCtx) {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                    const o = audioCtx.createOscillator();
                    const g = audioCtx.createGain();
                    g.gain.value = 0.0001;
                    o.connect(g).connect(audioCtx.destination);
                    o.start();
                    o.stop(audioCtx.currentTime + 0.01);
                }
            }
            function beep(type = 'square', f = 600, dur = 0.08, vol = 0.2, slide = -300) {
                if (!audioCtx) return;
                const o = audioCtx.createOscillator(), g = audioCtx.createGain();
                o.type = type;
                o.frequency.value = f;
                const t = audioCtx.currentTime;
                g.gain.setValueAtTime(0, t);
                g.gain.linearRampToValueAtTime(vol, t + 0.01);
                g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
                o.connect(g).connect(audioCtx.destination);
                if (slide !== 0) {
                    o.frequency.setValueAtTime(f, t);
                    o.frequency.exponentialRampToValueAtTime(Math.max(60, f + slide), t + dur);
                }
                o.start(t);
                o.stop(t + dur + 0.02);
            }
            const SFX = {
                shot: () => beep('square', 720, 0.07, 0.20, -400),
                specialShot: () => beep('sine', 1200, 0.1, 0.3, -800),
                boom: () => {
                    beep('triangle', 220, 0.20, 0.25, -120);
                    setTimeout(() => beep('triangle', 120, 0.16, 0.18, -30), 60);
                },
                collect: () => beep('sawtooth', 800, 0.05, 0.3, 200),
                playerHit: () => {
                    beep('sine', 180, 0.1, 0.5, 0);
                    setTimeout(() => beep('sine', 100, 0.1, 0.4, 0), 50);
                },
                planetBoom: () => {
                    beep('square', 300, 0.3, 0.5, -200);
                    setTimeout(() => beep('square', 100, 0.2, 0.4, -50), 100);
                }
            };

            // ===== Controles: Joystick + Fogo =====
            const joyBase = document.getElementById('joystick');
            const joyStick = document.getElementById('stick');
            const fireBtn = document.getElementById('fireBtn');
            const specialFireBtn = document.getElementById('specialFireBtn');

            let joy = { active: false, dx: 0, dy: 0, radius: 65 };

            function moveStick(x, y) {
                const r = joy.radius;
                const dx = x - r;
                const dy = y - r;
                const dist = Math.sqrt(dx * dx + dy * dy);

                if (dist > r) {
                    const angle = Math.atan2(dy, dx);
                    joyStick.style.left = `${r + r * Math.cos(angle)}px`;
                    joyStick.style.top = `${r + r * Math.sin(angle)}px`;
                } else {
                    joyStick.style.left = `${x}px`;
                    joyStick.style.top = `${y}px`;
                }
                joy.dx = dx / r;
                joy.dy = dy / r;
            }

            function joyPosFromEvent(e) {
                const t = e.touches ? e.touches[0] : e;
                const r = joyBase.getBoundingClientRect();
                return { x: (t.clientX - r.left), y: (t.clientY - r.top) };
            }
            function joyStart(e) {
                unlockAudio();
                joy.active = true;
                const p = joyPosFromEvent(e);
                moveStick(p.x, p.y);
                e.preventDefault();
            }
            function joyMove(e) {
                if (!joy.active) return;
                const p = joyPosFromEvent(e);
                moveStick(p.x, p.y);
                e.preventDefault();
            }
            function joyEnd() {
                joy.active = false;
                joy.dx = joy.dy = 0;
                joyStick.style.left = '50%';
                joyStick.style.top = '50%';
            }
            joyBase.addEventListener('touchstart', joyStart, { passive: false });
            joyBase.addEventListener('touchmove', joyMove, { passive: false });
            joyBase.addEventListener('touchend', joyEnd, { passive: false });
            joyBase.addEventListener('mousedown', joyStart);
            window.addEventListener('mousemove', joyMove);
            window.addEventListener('mouseup', joyEnd);

            let firing = false;
            let specialFiring = false;
            fireBtn.addEventListener('touchstart', () => { unlockAudio(); firing = true; }, { passive: true });
            fireBtn.addEventListener('touchend', () => { firing = false; }, { passive: true });
            fireBtn.addEventListener('mousedown', () => { unlockAudio(); firing = true; });
            fireBtn.addEventListener('mouseup', () => { firing = false; });
            specialFireBtn.addEventListener('touchstart', () => { unlockAudio(); specialFiring = true; }, { passive: true });
            specialFireBtn.addEventListener('touchend', () => { specialFiring = false; }, { passive: true });
            specialFireBtn.addEventListener('mousedown', () => { unlockAudio(); specialFiring = true; });
            specialFireBtn.addEventListener('mouseup', () => { specialFiring = false; });

            // ===== Configurações de Fases =====
            const PHASE_SETTINGS = {
                1: { label: 'Fase 1', spawn: 1800, speed: 0.04, enemyShotFreq: 4500 },
                2: { label: 'Fase 2', spawn: 1500, speed: 0.05, enemyShotFreq: 3500 },
                3: { label: 'Fase 3', spawn: 1200, speed: 0.06, enemyShotFreq: 3000 },
                4: { label: 'Fase 4', spawn: 900, speed: 0.075, enemyShotFreq: 2500 }
            };
            const ENEMY_TYPES = [
                { name: 'small', w: 24, h: 22, color: '#ff6363', score: 10, speedMul: 1.0, hasShot: false, health: 1 },
                { name: 'medium', w: 32, h: 28, color: '#ffb84a', score: 20, speedMul: 0.9, hasShot: true, health: 2 },
                { name: 'large', w: 42, h: 36, color: '#7cff84', score: 50, speedMul: 0.75, hasShot: true, health: 3 },
            ];

            // ===== Estado do jogo =====
            const state = {
                running: false,
                paused: false,
                score: 0,
                lives: 10,
                enemiesDestroyed: 0,
                planetCollisions: 0,
                enemyCollisions: 0,
                enemyHitsReceived: 0,
                phase: 1,
                spawnTimer: 0,
                lastShot: 0,
                lastSpecialShot: 0,
                lastHeartSpawn: 0,
                lastEnemyShot: 0,
                time: 0
            };

            const player = { x: W / 2, y: H * 0.85, w: 36 * DPR, h: 44 * DPR, speed: 0.6 };
            const bullets = [];
            const enemies = [];
            const particles = [];
            const hearts = [];
            const enemyBullets = [];

            const planets = [];
            const planetImages = [
                "https://upload.wikimedia.org/wikipedia/commons/c/c5/The_Planet_Mercury_in_TrueColor.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/e/e5/Venus-real_color.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/e/e1/FullMoon2010.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/2/2b/Hubble_Views_Mars_in_Opposition_2016_%28cropped%29.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/e/ea/Jupiter_and_its_shrunken_Great_Red_Spot.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c7/Saturn_during_Equinox.jpg/1200px-Saturn_during_Equinox.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Uranus2.jpg/1024px-Uranus2.jpg",
                "https://upload.wikimedia.org/wikipedia/commons/0/05/Neptune_-_Voyager_2_%2829280145217%29_-_Procesed.jpg"
            ];

            function loadPlanetImages() {
                // Diminuindo o número de planetas
                planetImages.slice(0, 3).forEach(src => {
                    const img = new Image();
                    img.src = src;
                    img.onload = () => {
                        planets.push({
                            img: img,
                            x: rand(0, W),
                            y: rand(-H, H * 0.6),
                            speed: rand(0.005, 0.015) * DPR, // Velocidade reduzida
                            size: rand(0.08, 0.25) * W,
                            opacity: rand(0.3, 0.7)
                        });
                    };
                });
            }

            function reset() {
                state.running = true;
                state.paused = false;
                state.score = 0;
                state.lives = 10;
                state.enemiesDestroyed = 0;
                state.planetCollisions = 0;
                state.enemyCollisions = 0;
                state.enemyHitsReceived = 0;
                state.phase = 1;
                state.spawnTimer = 0;
                state.lastShot = 0;
                state.lastSpecialShot = 0;
                state.lastHeartSpawn = 0;
                state.lastEnemyShot = 0;
                state.time = 0;
                bullets.length = 0;
                enemies.length = 0;
                particles.length = 0;
                hearts.length = 0;
                enemyBullets.length = 0;
                player.x = W / 2;
                player.y = H * 0.85;
                updateUI();
                planets.forEach(p => {
                    p.x = rand(0, W);
                    p.y = rand(-H, H * 0.6);
                });
            }

            function updateUI() {
                phaseEl.textContent = 'Fase: ' + state.phase;
                livesEl.textContent = 'Vidas: ' + state.lives;
                scoreEl.textContent = 'Score: ' + state.score;
                hitsEl.textContent = `Tiros Recebidos: ${state.enemyHitsReceived}/10`;
                collisionsEl.textContent = `Colisões: ${state.enemyCollisions}/5`;
            }

            // ===== Helpers =====
            function rand(a, b) { return a + Math.random() * (b - a); }
            function clamp(v, a, b) { return Math.max(a, Math.min(b, v)); }
            function overlap(a, b) {
                return Math.abs(a.x - b.x) * 2 < (a.w + b.w) && Math.abs(a.y - b.y) * 2 < (a.h + b.h);
            }

            // ===== Spawns =====
            function spawnEnemy() {
                const phase = PHASE_SETTINGS[state.phase] || PHASE_SETTINGS[4];
                const t = ENEMY_TYPES[Math.floor(Math.random() * ENEMY_TYPES.length)];
                const e = {
                    type: t.name,
                    score: t.score,
                    color: t.color,
                    w: t.w * DPR,
                    h: t.h * DPR,
                    x: rand(30 * DPR, W - 30 * DPR),
                    y: -40 * DPR,
                    vy: (phase.speed * t.speedMul) * DPR,
                    vx: (Math.random() < 0.5 ? -1 : 1) * rand(0.05, 0.12) * DPR,
                    hasShot: t.hasShot,
                    lastShot: 0,
                    health: t.health
                };
                enemies.push(e);
            }

            function spawnHeart() {
                hearts.push({
                    x: rand(30 * DPR, W - 30 * DPR),
                    y: -40 * DPR,
                    w: 24 * DPR,
                    h: 24 * DPR,
                    vy: (PHASE_SETTINGS[state.phase].speed * 0.8) * DPR
                });
            }

            // ===== Efeitos =====
            function spawnExplosion(x, y, n = 16, color = '#ffa800') {
                for (let i = 0; i < n; i++) {
                    particles.push({
                        x, y,
                        r: rand(1, 3) * DPR,
                        life: rand(220, 480),
                        vx: Math.cos(i / n * Math.PI * 2) * rand(0.08, 0.35) * DPR,
                        vy: Math.sin(i / n * Math.PI * 2) * rand(0.08, 0.35) * DPR,
                        color
                    });
                }
            }
            
            function spawnPlanetExplosion(x, y, size) {
                for (let i = 0; i < 50; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const speed = rand(0.1, 0.6) * DPR;
                    particles.push({
                        x: x + Math.cos(angle) * size * rand(0.2, 0.5),
                        y: y + Math.sin(angle) * size * rand(0.2, 0.5),
                        r: rand(2, 5) * DPR,
                        life: rand(300, 700),
                        vx: Math.cos(angle) * speed,
                        vy: Math.sin(angle) * speed,
                        color: '#ff4d4d' // Cor avermelhada de explosão
                    });
                }
            }

            // ===== Fluxo =====
            function startGame() {
                unlockAudio();
                startOverlay.classList.remove('show');
                gameOver.classList.remove('show');
                reset();
                last = performance.now();
            }

            function gameOverNow() {
                state.running = false;
                finalScore.textContent = 'Score: ' + state.score;
                gameOver.classList.add('show');
            }

            function advancePhase() {
                const levels = Object.keys(PHASE_SETTINGS).map(Number);
                const currentLevelIndex = levels.indexOf(state.phase);
                if (currentLevelIndex < levels.length - 1) {
                    state.phase = levels[currentLevelIndex + 1];
                    showMessage(`FASE ${state.phase}`, 2000);
                }
            }

            // ===== Loop =====
            let last = performance.now();
            function loop(now) {
                const dt = now - last;
                last = now;
                if (!state.running || state.paused) {
                    draw();
                    requestAnimationFrame(loop);
                    return;
                }

                state.time += dt;
                update(dt);
                draw();
                requestAnimationFrame(loop);
            }

            function update(dt) {
                const dx = joy.dx;
                const dy = joy.dy;
                const speed = player.speed * dt * DPR * 0.8;
                player.x += dx * speed * 3.2;
                player.y += dy * speed * 3.2;
                player.x = clamp(player.x, player.w / 2 + 10 * DPR, W - player.w / 2 - 10 * DPR);
                player.y = clamp(player.y, player.h / 2 + 10 * DPR, H - player.h / 2 - 10 * DPR);

                if (firing && state.time - state.lastShot > 160) {
                    bullets.push({ x: player.x, y: player.y - player.h / 2, w: 6 * DPR, h: 14 * DPR, vy: -0.9 * DPR, damage: 1 });
                    state.lastShot = state.time;
                    SFX.shot();
                }
                if (specialFiring && state.time - state.lastSpecialShot > 500) {
                    bullets.push({ x: player.x, y: player.y - player.h / 2, w: 12 * DPR, h: 28 * DPR, vy: -1.2 * DPR, damage: 3, isSpecial: true });
                    state.lastSpecialShot = state.time;
                    showMessage('QUÂNTICO', 1000, true);
                    SFX.specialShot();
                }

                // Atualiza e checa projéteis do jogador
                for (let i = bullets.length - 1; i >= 0; i--) {
                    const b = bullets[i];
                    b.y += b.vy * dt;
                    if (b.y < -30) bullets.splice(i, 1);
                }

                // Atualiza e checa projéteis inimigos
                for (let i = enemyBullets.length - 1; i >= 0; i--) {
                    const eb = enemyBullets[i];
                    eb.y += eb.vy * dt;
                    if (overlap(eb, player)) {
                        state.enemyHitsReceived++;
                        if (state.enemyHitsReceived >= 10) {
                            state.lives--;
                            state.enemyHitsReceived = 0;
                            showMessage('Vida Perdida!', 1500);
                        }
                        spawnExplosion(player.x, player.y, 20, '#ff4d4d');
                        SFX.playerHit();
                        enemyBullets.splice(i, 1);
                        if (state.lives <= 0) {
                            gameOverNow();
                            return;
                        }
                    } else if (eb.y > H + 20) {
                        enemyBullets.splice(i, 1);
                    }
                }

                state.spawnTimer += dt;
                const phaseSettings = PHASE_SETTINGS[state.phase] || PHASE_SETTINGS[4];
                const spawnEvery = phaseSettings.spawn;
                if (state.spawnTimer > spawnEvery) {
                    state.spawnTimer = 0;
                    spawnEnemy();
                }

                if (state.time - state.lastHeartSpawn > 20000) {
                    spawnHeart();
                    state.lastHeartSpawn = state.time;
                }

                // Colisões: balas do jogador vs. inimigos
                for (let i = enemies.length - 1; i >= 0; i--) {
                    const e = enemies[i];
                    let hit = false;
                    for (let j = bullets.length - 1; j >= 0; j--) {
                        const b = bullets[j];
                        if (overlap(b, e)) {
                            bullets.splice(j, 1);
                            e.health = e.health - b.damage;
                            if (e.health <= 0) {
                                enemies.splice(i, 1);
                                spawnExplosion(e.x, e.y, b.isSpecial ? 30 : 18, e.color);
                                SFX.boom();
                                state.score += e.score;
                                state.enemiesDestroyed++;
                                if (state.enemiesDestroyed % 10 === 0) {
                                    state.lives += 3;
                                    showMessage('Ganhou 3 Vidas!', 1500);
                                    advancePhase();
                                }
                                hit = true;
                            }
                            break;
                        }
                    }
                    if (hit) continue;

                    e.y += e.vy * dt;
                    e.x += e.vx * dt;
                    if (e.x < e.w / 2 || e.x > W - e.w / 2) e.vx *= -1;
                    if (e.hasShot && state.time - e.lastShot > phaseSettings.enemyShotFreq) {
                        enemyBullets.push({ x: e.x, y: e.y + e.h / 2, w: 6 * DPR, h: 14 * DPR, vy: 0.5 * DPR });
                        e.lastShot = state.time;
                    }
                }

                // Colisões: inimigos vs. jogador / inimigos vs. borda
                for (let i = enemies.length - 1; i >= 0; i--) {
                    const e = enemies[i];
                    if (overlap(e, player)) {
                        state.enemyCollisions++;
                        if (state.enemyCollisions >= 5) {
                            state.lives--;
                            state.enemyCollisions = 0;
                            showMessage('Vida Perdida!', 1500);
                        }
                        enemies.splice(i, 1);
                        spawnExplosion(player.x, player.y, 24, '#66ffff');
                        SFX.playerHit();
                        if (state.lives <= 0) {
                            gameOverNow();
                            return;
                        }
                    } else if (e.y - e.h / 2 > H + 40) {
                        enemies.splice(i, 1);
                    }
                }

                // Colisões: corações vs. jogador
                for (let i = hearts.length - 1; i >= 0; i--) {
                    const h = hearts[i];
                    h.y += h.vy * dt;
                    if (overlap(h, player)) {
                        hearts.splice(i, 1);
                        state.lives += 3; // Corações dão 3 vidas agora
                        showMessage('3 VIDAS GANHAS!', 1000);
                        SFX.collect();
                    } else if (h.y > H + 20) {
                        hearts.splice(i, 1);
                    }
                }

                // Colisões: tiro quântico vs. planetas
                for (let i = planets.length - 1; i >= 0; i--) {
                    const p = planets[i];
                    p.y += p.speed * dt * 10;
                    if (p.y > H + p.size / 2) {
                        p.y = -p.size / 2;
                        p.x = rand(0, W);
                        p.speed = rand(0.005, 0.015) * DPR; // Velocidade reduzida
                        p.size = rand(0.08, 0.25) * W;
                        p.opacity = rand(0.3, 0.7);
                    }
                    
                    let hit = false;
                    for (let j = bullets.length - 1; j >= 0; j--) {
                        const b = bullets[j];
                        if (b.isSpecial && overlap(b, {x: p.x, y: p.y, w: p.size, h: p.size})) {
                            spawnPlanetExplosion(p.x, p.y, p.size);
                            SFX.planetBoom();
                            state.score += 100;
                            showMessage('+100', 500);
                            bullets.splice(j, 1);
                            p.y = -H; // Reinicia o planeta fora da tela
                            hit = true;
                            break;
                        }
                    }
                    if (hit) continue;

                    if (overlap({x: player.x, y: player.y, w: player.w*0.8, h: player.h*0.8}, {x: p.x, y: p.y, w: p.size*0.8, h: p.size*0.8})) {
                        state.planetCollisions++;
                        if (state.planetCollisions >= 4) {
                            state.lives--;
                            state.planetCollisions = 0;
                            showMessage('Vida Perdida!', 1500);
                        }
                        spawnExplosion(player.x, player.y, 30, '#ff9999');
                        SFX.playerHit();
                        p.y = -H;
                        if (state.lives <= 0) {
                            gameOverNow();
                            return;
                        }
                    }
                }

                // Atualiza partículas
                for (let i = particles.length - 1; i >= 0; i--) {
                    const p = particles[i];
                    p.x += p.vx * dt;
                    p.y += p.vy * dt;
                    p.life -= dt;
                    p.vy += 0.00015 * dt;
                    if (p.life <= 0) particles.splice(i, 1);
                }

                updateUI();
                if (state.lives <= 0) gameOverNow();
            }

            function draw() {
                ctx.clearRect(0, 0, W, H);

                // Desenha estrelas
                for (let i = 0; i < 70; i++) {
                    const sx = (i * 97 + (state.time * 0.05)) % W;
                    const sy = (i * 131 + (state.time * 0.12)) % H;
                    ctx.globalAlpha = 0.35 + ((i % 3) / 10);
                    ctx.fillStyle = '#8ff';
                    ctx.fillRect(sx, sy, 2 * DPR, 2 * DPR);
                }
                ctx.globalAlpha = 1;

                // Apenas desenha um subconjunto de planetas para otimizar
                planets.slice(0, 3).forEach(p => {
                    if (p.img.complete) {
                        ctx.save();
                        ctx.globalAlpha = p.opacity;
                        ctx.drawImage(p.img, p.x - p.size / 2, p.y - p.size / 2, p.size, p.size);
                        ctx.restore();
                    }
                });

                drawPixelShip(player.x, player.y, player.w, player.h, '#66ffff');
                bullets.forEach(b => drawBullet(b, b.isSpecial ? '#00e0ff' : '#9ff'));
                enemyBullets.forEach(eb => drawBullet(eb, '#ff4d4d', true));
                enemies.forEach(e => drawAlien(e.x, e.y, e.w, e.h, e.color));
                hearts.forEach(h => drawHeart(h.x, h.y, h.w, h.h));
                particles.forEach(p => {
                    ctx.globalAlpha = Math.max(0, p.life / 480);
                    ctx.fillStyle = p.color;
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.globalAlpha = 1;
                });
            }

            function drawBullet(b, color, isEnemy = false) {
                ctx.fillStyle = color;
                ctx.shadowColor = color;
                ctx.shadowBlur = b.isSpecial ? 15 : 5;
                roundedRect(b.x - b.w / 2, b.y - b.h / 2, b.w, b.h, 2 * DPR);
                ctx.fill();
                ctx.shadowBlur = 0;
            }

            function roundedRect(x, y, w, h, r) {
                ctx.beginPath();
                ctx.moveTo(x + r, y);
                ctx.arcTo(x + w, y, x + w, y + h, r);
                ctx.arcTo(x + w, y + h, x, y + h, r);
                ctx.arcTo(x, y + h, x, y, r);
                ctx.arcTo(x, y, x + w, y, r);
                ctx.closePath();
            }

            function drawHeart(x, y, w, h) {
                ctx.save();
                ctx.translate(x, y);
                ctx.fillStyle = '#ff4d4d';
                ctx.beginPath();
                ctx.moveTo(0, h * 0.4);
                ctx.bezierCurveTo(w * 0.5, -h * 0.6, w, h * 0.1, 0, h * 0.8);
                ctx.bezierCurveTo(-w, h * 0.1, -w * 0.5, -h * 0.6, 0, h * 0.4);
                ctx.fill();
                ctx.restore();
            }

            function drawPixel(x, y, color) {
                ctx.fillStyle = color;
                ctx.fillRect(x, y, 1, 1);
            }

            function drawPixelShip(x, y, w, h) {
                ctx.save();
                ctx.translate(x - w / 2, y - h / 2);
                ctx.scale(w / 30, h / 29);
                const corAzulClaro = '#b9e8ff';
                const corAzulMedio = '#a0d1eb';
                const corAzulEscuro = '#688c9f';
                const corVermelho = '#e55a5a';
                const corCinza = '#a6a6a6';
                const corCinzaEscuro = '#858585';
                const corLaranja = '#f0c055';
                const px = (x, y, color) => drawPixel(x, y, color);

                const pixelData = [
                    {x: [11,18], y: 2, c: corAzulEscuro},
                    {x: [12,17], y: 3, c: corAzulEscuro},
                    {x: [13,16], y: 4, c: corAzulEscuro},
                    {x: [13,16], y: 2, c: corAzulClaro},
                    {x: [12,17], y: 3, c: corAzulClaro},
                    {x: [11,18], y: 4, c: corAzulClaro},
                    {x: [10,19], y: 5, c: corAzulClaro},
                    {x: [9,20], y: 6, c: corAzulClaro},
                    {x: [8,21], y: 7, c: corAzulClaro},
                    {x: [7,22], y: 8, c: corAzulClaro},
                    {x: [6,23], y: 9, c: corAzulClaro},
                    {x: [11,12,17,18], y: 5, c: corAzulMedio},
                    {x: [13,14,15,16], y: 6, c: corAzulMedio},
                    {x: [12,17], y: 7, c: corAzulMedio},
                    {x: [10,19], y: 8, c: corAzulMedio},
                    {x: [8,21], y: 9, c: corAzulMedio},
                    {x: [4,25], y: 10, c: corCinza},
                    {x: [3,26], y: 11, c: corCinza},
                    {x: [2,27], y: 12, c: corCinza},
                    {x: [1,28], y: 13, c: corCinza},
                    {x: [1,28], y: 14, c: corCinza},
                    {x: [0,29], y: 15, c: corVermelho},
                    {x: [0,29], y: 16, c: corVermelho},
                    {x: [0,29], y: 17, c: corVermelho},
                    {x: [0,29], y: 18, c: corVermelho},
                    {x: [1,28], y: 19, c: corCinza},
                    {x: [1,28], y: 20, c: corCinza},
                    {x: [2,27], y: 21, c: corCinza},
                    {x: [3,26], y: 22, c: corCinza},
                    {x: [4,25], y: 23, c: corCinza},
                    {x: [5,24], y: 10, c: corCinzaEscuro},
                    {x: [4,25], y: 11, c: corCinzaEscuro},
                    {x: [3,26], y: 12, c: corCinzaEscuro},
                    {x: [2,27], y: 13, c: corCinzaEscuro},
                    {x: [2,27], y: 14, c: corCinzaEscuro},
                    {x: [2,27], y: 19, c: corCinzaEscuro},
                    {x: [3,26], y: 20, c: corCinzaEscuro},
                    {x: [4,25], y: 21, c: corCinzaEscuro},
                    {x: [5,24], y: 22, c: corCinzaEscuro},
                    {x: [4,24], y: 24, c: corLaranja},
                    {x: [4,25], y: 25, c: corCinza},
                    {x: [4,25], y: 26, c: corCinza},
                    {x: [4,25], y: 27, c: corCinza},
                    {x: [4,25], y: 28, c: corCinza},
                    {x: [3,5,24,26], y: 28, c: corCinza},
                    {x: [3,5,24,26], y: 27, c: corCinzaEscuro},
                    {x: [14,15], y: 24, c: corLaranja},
                    {x: [14,15], y: 25, c: corCinza},
                    {x: [14,15], y: 26, c: corCinza},
                    {x: [13,14,15,16], y: 27, c: corCinza},
                    {x: [13,16], y: 28, c: corCinzaEscuro}
                ];

                pixelData.forEach(data => {
                    if (Array.isArray(data.x)) {
                        data.x.forEach(x => px(x, data.y, data.c));
                    } else {
                        for (let x = data.x[0]; x <= data.x[1]; x++) px(x, data.y, data.c);
                    }
                });

                ctx.restore();
            }

            function drawAlien(x, y, w, h, color) {
                ctx.save();
                ctx.translate(x, y);
                ctx.fillStyle = color;
                ctx.beginPath();
                ctx.ellipse(0, 0, w / 2, h / 3, 0, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.beginPath();
                ctx.arc(-w / 4, -h / 6, w / 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(w / 4, -h / 6, w / 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#000';
                ctx.beginPath();
                ctx.arc(-w / 4, -h / 6, w / 16, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(w / 4, -h / 6, w / 16, 0, Math.PI * 2);
                ctx.fill();
                ctx.strokeStyle = color;
                ctx.lineWidth = 2 * DPR;
                ctx.beginPath();
                ctx.moveTo(-w / 3, -h / 2);
                ctx.lineTo(-w / 4, -h / 3);
                ctx.moveTo(w / 3, -h / 2);
                ctx.lineTo(w / 4, -h / 3);
                ctx.stroke();
                ctx.restore();
            }

            loadPlanetImages();
            requestAnimationFrame(loop);
        })();
    </script>
</body>
</html>
