# Love-morning-game-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Precision Archer - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #000; position: fixed; font-family: -apple-system, system-ui, sans-serif; }

        #sky { position: fixed; inset: 0; background: linear-gradient(to bottom, #020111 0%, #000000 100%); transition: background 2s; z-index: -2; }
        #sun { position: absolute; bottom: -150px; left: 50%; transform: translateX(-50%); width: 140px; height: 140px; background: radial-gradient(circle, #fff700, #ff8c00); border-radius: 50%; box-shadow: 0 0 80px #ff8c00; z-index: -1; transition: bottom 2s; }
        
        #basket { 
            position: absolute; bottom: -20px; width: 120%; left: -10%; height: 25%; 
            background: #5d4037; border-top: 10px solid #3e2723; border-radius: 50% 50% 0 0; 
            z-index: 10; transition: transform 1.5s ease-in;
        }

        /* Gold Aiming Line */
        #aim-line { position: absolute; bottom: 15%; left: 50%; width: 3px; height: 200px; background: rgba(255, 215, 0, 0.5); transform-origin: bottom center; display: none; z-index: 15; }

        #bow-container { position: absolute; bottom: 5%; left: 50%; transform: translateX(-50%); width: 100px; height: 150px; z-index: 20; transition: transform 1.5s; }
        #bow-visual { font-size: 80px; text-align: center; pointer-events: none; transform-origin: center center; }

        .target { position: absolute; z-index: 5; transition: font-size 0.3s; transform: translate(-50%, -50%); }
        
        /* Metallic Gold Arrow */
        .projectile { 
            position: absolute; width: 5px; height: 50px; 
            background: linear-gradient(to right, #bf953f, #fcf6ba, #b38728, #fbf5b7, #aa771c); 
            z-index: 15; border-radius: 2px; box-shadow: 0 0 10px rgba(255, 215, 0, 0.8);
            transform-origin: center center;
        }

        .sparkle { position: absolute; width: 5px; height: 5px; background: gold; border-radius: 50%; pointer-events: none; z-index: 25; box-shadow: 0 0 5px white; }

        #hud { 
            position: absolute; top: 0; width: 100%; display: flex; 
            justify-content: space-between; padding: 50px 25px 0 25px; 
            z-index: 100; color: white; font-weight: bold; text-shadow: 0 0 10px #000; 
        }
        .screen { position: fixed; inset: 0; z-index: 200; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 30px; }
        #start-screen { background: rgba(0,0,0,0.9); color: white; }
        #win-screen { display: none; background: rgba(255, 255, 255, 0.98); color: #333; }
        #fail-screen { display: none; background: rgba(0,0,0,0.95); color: white; }
        .btn { padding: 15px 35px; border-radius: 50px; border: none; background: #fbc02d; font-weight: bold; margin-top: 15px; color: #000; }
    </style>
</head>
<body>

<div id="sky"></div>
<div id="sun"></div>
<div id="basket"></div>
<div id="aim-line"></div>

<div id="bow-container">
    <div id="bow-visual">🏹</div>
</div>

<div id="start-screen" class="screen">
    <h1 style="color: #fbc02d;">Good Morning, Giselle! ❤️</h1>
    <p style="margin: 15px 0;">Drag and aim the bow. Release to fire gold arrows!<br>The sunbeams shrink as you rise.</p>
    <button class="btn" onclick="startGame()">Start 🎈</button>
</div>

<div id="hud">
    <div>☀️ <span id="score">0</span>/10</div>
    <div>HP: <span id="lives-ui">❤️❤️❤️</span></div>
</div>

<div id="fail-screen" class="screen">
    <h1 style="color: #ff4d4d;">THE BALLOON FELL!</h1>
    <button class="btn" onclick="location.reload()">Try Again</button>
</div>

<div id="win-screen" class="screen">
    <h1 style="color: #f57c00;">Sunrise Secured! ☀️</h1>
    <p>Good morning, my love! You are beautiful and perfect.</p>
    <div id="timerText" style="margin: 15px 0; font-weight: bold; color: #f57c00;"></div>
    <button class="btn" onclick="showLetter()" id="letterBtn">Open My Letter 💌</button>
    <div id="secretLetter" style="display:none; padding: 15px; font-style: italic;">"I can't wait for Seattle! I love you!"</div>
</div>

<audio id="gameMusic" loop><source src="https://www.bensound.com/bensound-music/bensound-tomorrow.mp3" type="audio/mpeg"></audio>

<script>
    let score = 0, lives = 3, gameActive = false;
    let dragStartX, dragStartY, currentAngle = 0;
    
    const bowContainer = document.getElementById('bow-container'), bowVisual = document.getElementById('bow-visual');
    const aimLine = document.getElementById('aim-line'), basket = document.getElementById('basket');
    const sky = document.getElementById('sky'), sun = document.getElementById('sun');
    const music = document.getElementById('gameMusic');

    function startGame() {
        document.getElementById('start-screen').style.display = 'none';
        gameActive = true;
        music.play().catch(e => console.log("Audio blocked"));
        updateCountdown();
        spawnLoop();
    }

    // Precise Aiming Controls
    document.addEventListener('touchstart', (e) => {
        if (!gameActive) return;
        dragStartX = e.touches[0].clientX;
        dragStartY = e.touches[0].clientY;
        aimLine.style.display = 'block';
    });

    document.addEventListener('touchmove', (e) => {
        if (!gameActive) return;
        const touchX = e.touches[0].clientX;
        const touchY = e.touches[0].clientY;
        const dx = touchX - dragStartX;
        const dy = touchY - dragStartY;
        
        // Calculate the angle for the bow to point UP towards the drag direction
        currentAngle = Math.atan2(dx, -dy) * (180 / Math.PI);
        bowVisual.style.transform = `rotate(${currentAngle}deg)`;
        aimLine.style.transform = `translateX(-50%) rotate(${currentAngle}deg)`;
    });

    document.addEventListener('touchend', (e) => {
        if (!gameActive) return;
        aimLine.style.display = 'none';
        fireArrow(currentAngle);
    });

    function createSparkles(x, y) {
        for (let i = 0; i < 15; i++) {
            const sparkle = document.createElement('div');
            sparkle.className = 'sparkle';
            sparkle.style.left = x + 'px';
            sparkle.style.top = y + 'px';
            document.body.appendChild(sparkle);
            const ang = Math.random() * Math.PI * 2;
            const vel = Math.random() * 4 + 2;
            let sx = Math.cos(ang) * vel, sy = Math.sin(ang) * vel, op = 1;
            const move = setInterval(() => {
                sparkle.style.left = (parseFloat(sparkle.style.left) + sx) + 'px';
                sparkle.style.top = (parseFloat(sparkle.style.top) + sy) + 'px';
                op -= 0.05; sparkle.style.opacity = op;
                if (op <= 0) { clearInterval(move); sparkle.remove(); }
            }, 20);
        }
    }

    function fireArrow(angle) {
        const proj = document.createElement('div');
        proj.className = 'projectile';
        proj.style.left = '50%'; proj.style.bottom = '15%';
        proj.style.transform = `translateX(-50%) rotate(${angle}deg)`;
        document.body.appendChild(proj);

        const rad = angle * (Math.PI / 180);
        const vx = Math.sin(rad) * 10; // Speed X
        const vy = Math.cos(rad) * 10; // Speed Y

        let bPos = 15; // bottom %
        let lPos = 50; // left %

        const fly = setInterval(() => {
            bPos += (vy / window.innerHeight) * 100;
            lPos += (vx / window.innerWidth) * 100;
            proj.style.bottom = bPos + '%';
            proj.style.left = lPos + '%';
            checkCollision(proj, fly);
            if (bPos > 110 || bPos < -10 || lPos < -10 || lPos > 110) { clearInterval(fly); proj.remove(); }
        }, 16);
    }

    function spawnItem() {
        if (!gameActive) return;
        const item = document.createElement('div'), rand = Math.random();
        item.className = 'target';
        let type = rand > 0.8 ? 'sun' : (rand > 0.5 ? 'rain' : 'cloud');
        item.innerHTML = (type === 'sun') ? '☀️' : (type === 'cloud' ? '☁️' : '💧');
        item.dataset.type = type;
        const size = type === 'sun' ? Math.max(25, 55 - (score * 3)) : 45;
        item.style.fontSize = size + 'px';
        item.style.left = (Math.random() * 80 + 10) + '%';
        item.style.top = '-50px';
        document.body.appendChild(item);
        let yPos = -50;
        const fall = setInterval(() => {
            yPos += (type === 'sun' ? 2.5 : 5);
            item.style.top = yPos + 'px';
            if (yPos > window.innerHeight) { clearInterval(fall); item.remove(); }
        }, 20);
        item.dataset.intervalId = fall;
    }

    function checkCollision(proj, flyInterval) {
        const pRect = proj.getBoundingClientRect();
        document.querySelectorAll('.target').forEach(t => {
            const tRect = t.getBoundingClientRect();
            if (pRect.left < tRect.right && pRect.right > tRect.left && pRect.top < tRect.bottom && pRect.bottom > tRect.top) {
                if (t.dataset.type === 'sun') {
                    score++; createSparkles(tRect.left + 20, tRect.top + 20); updateEnv();
                } else {
                    lives--; updateLivesUI(); if (lives <= 0) gameFail();
                }
                document.getElementById('score').innerText = score;
                clearInterval(t.dataset.intervalId); t.remove();
                clearInterval(flyInterval); proj.remove();
                if (score >= 10) win();
            }
        });
    }

    function updateLivesUI() { document.getElementById('lives-ui').innerText = "❤️".repeat(Math.max(0, lives)); }
    function updateEnv() {
        const colors = ['#020111', '#1a1a40', '#4d2c5e', '#a34d5d', '#ff9a44', '#ffcf71'];
        sky.style.background = colors[Math.floor(score/2)] || '#ffcf71';
        sun.style.bottom = (-150 + (score * 35)) + 'px';
    }

    function gameFail() {
        gameActive = false; basket.style.transform = "translateY(600px) rotate(20deg)";
        bowContainer.style.transform = "translateY(600px)";
        setTimeout(() => { document.getElementById('fail-screen').style.display = 'flex'; }, 800);
    }

    function win() {
        gameActive = false; document.getElementById('win-screen').style.display = 'flex';
        document.querySelectorAll('.target').forEach(t => t.remove());
    }

    function showLetter() { document.getElementById('secretLetter').style.display = 'block'; document.getElementById('letterBtn').style.display = 'none'; }

    function updateCountdown() {
        const trip = new Date("February 14, 2026 00:00:00").getTime();
        const diff = trip - new Date().getTime();
        const d = Math.floor(diff / (1000*60*60*24)), h = Math.floor((diff % (1000*60*60*24)) / (1000*60*60));
        document.getElementById('timerText').innerText = `Seattle Trip: ${d}d ${h}h Left!`;
    }

    function spawnLoop() { if(gameActive && score < 10) { spawnItem(); setTimeout(() => { if(gameActive) spawnItem(); }, 400); setTimeout(spawnLoop, 850); } }
</script>
</body>
</html>
