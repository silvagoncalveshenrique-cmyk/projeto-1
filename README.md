<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Estrada Brasil</title>
    <style>
        body { display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; background: #1a1a1a; font-family: 'Segoe UI', sans-serif; overflow: hidden; }
        canvas { border: 5px solid #444; background: #333; box-shadow: 0 0 30px rgba(0,0,0,0.7); display: none; }
        
        /* Interface do Menu */
        #menu { width: 400px; height: 600px; background: linear-gradient(to bottom, #2c3e50, #000); border: 5px solid #f1c40f; display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; text-align: center; }
        h1 { font-size: 45px; color: #f1c40f; text-shadow: 3px 3px #c0392b; margin-bottom: 10px; }
        .color-selector { margin: 20px 0; }
        .color-btn { width: 40px; height: 40px; border: 3px solid white; cursor: pointer; margin: 5px; border-radius: 5px; transition: 0.2s; }
        .color-btn:hover { transform: scale(1.2); }
        #start-btn { padding: 15px 40px; font-size: 20px; background: #27ae60; color: white; border: none; cursor: pointer; border-radius: 50px; font-weight: bold; margin-top: 20px; }
        #start-btn:hover { background: #2ecc71; }
        
        .ui { position: absolute; top: 20px; color: white; text-align: center; pointer-events: none; display: none; }
        #pauseMsg { position: absolute; color: white; font-size: 40px; font-weight: bold; display: none; background: rgba(0,0,0,0.6); width: 400px; height: 600px; justify-content: center; align-items: center; z-index: 10; }
    </style>
</head>
<body>

<div id="menu">
    <h1>ESTRADA BRASIL</h1>
    <p>Escolha a cor do seu possante:</p>
    <div class="color-selector">
        <button class="color-btn" style="background: #e74c3c;" onclick="selectColor('#e74c3c')"></button>
        <button class="color-btn" style="background: #3498db;" onclick="selectColor('#3498db')"></button>
        <button class="color-btn" style="background: #f1c40f;" onclick="selectColor('#f1c40f')"></button>
        <button class="color-btn" style="background: #ffffff;" onclick="selectColor('#ffffff')"></button>
        <button class="color-btn" style="background: #9b59b6;" onclick="selectColor('#9b59b6')"></button>
    </div>
    <p>Comandos: Setas para mover<br>Tecla <strong>P</strong> para pausar</p>
    <button id="start-btn" onclick="startGame()">INICIAR CORRIDA</button>
</div>

<div class="ui" id="gameUI">
    <div id="scoreDisplay" style="font-size: 24px; color: #f1c40f;">Pontos: 0</div>
</div>
<div id="pauseMsg">PAUSADO</div>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const menu = document.getElementById("menu");
    const gameUI = document.getElementById("gameUI");
    const pauseMsg = document.getElementById("pauseMsg");

    let player = { x: 175, y: 480, w: 45, h: 75, speed: 7, color: '#e74c3c' };
    let obstacles = [];
    let score = 0;
    let gameActive = false;
    let isPaused = false;
    let roadSpeed = 6;
    let lineOffset = 0;
    let keys = {};

    function selectColor(c) {
        player.color = c;
        // Feedback visual simples
        document.querySelectorAll('.color-btn').forEach(b => b.style.borderColor = 'white');
        event.target.style.borderColor = '#27ae60';
    }

    function startGame() {
        menu.style.display = "none";
        canvas.style.display = "block";
        gameUI.style.display = "block";
        gameActive = true;
        gameLoop();
    }

    window.addEventListener("keydown", (e) => {
        keys[e.code] = true;
        if (e.code === "KeyP" && gameActive) {
            isPaused = !isPaused;
            pauseMsg.style.display = isPaused ? "flex" : "none";
        }
    });
    window.addEventListener("keyup", (e) => keys[e.code] = false);

    function spawnObstacle() {
        if (!gameActive || isPaused) return;
        let xPos = Math.random() * (canvas.width - 70) + 15;
        let isHole = Math.random() < 0.15;
        
        if (isHole) {
            obstacles.push({ type: 'hole', x: xPos, y: -100, radius: 22 });
        } else {
            let colors = ['#555', '#9b59b6', '#2ecc71', '#e67e22'];
            obstacles.push({ type: 'car', x: xPos, y: -100, w: 45, h: 75, color: colors[Math.floor(Math.random()*colors.length)] });
        }
    }

    function update() {
        if (!gameActive || isPaused) return;

        lineOffset += roadSpeed;
        if (lineOffset > 40) lineOffset = 0;

        if (keys["ArrowLeft"] && player.x > 15) player.x -= player.speed;
        if (keys["ArrowRight"] && player.x < canvas.width - player.w - 15) player.x += player.speed;
        if (keys["ArrowUp"] && player.y > 20) player.y -= player.speed;
        if (keys["ArrowDown"] && player.y < canvas.height - player.h - 20) player.y += player.speed;

        for (let i = 0; i < obstacles.length; i++) {
            obstacles[i].y += roadSpeed;

            // Colisão
            let hit = false;
            let o = obstacles[i];
            if (o.type === 'car') {
                if (player.x < o.x + o.w && player.x + player.w > o.x && player.y < o.y + o.h && player.y + player.h > o.y) hit = true;
            } else {
                let dx = Math.abs((o.x + o.radius) - (player.x + player.w/2));
                let dy = Math.abs((o.y + o.radius) - (player.y + player.h/2));
                if (dx < (player.w/2 + o.radius) && dy < (player.h/2 + o.radius)) hit = true;
            }

            if (hit) {
                gameActive = false;
                alert("GAME OVER! Pontos: " + score);
                location.reload();
            }

            if (o.y > canvas.height) {
                obstacles.splice(i, 1);
                score += 10;
                document.getElementById("scoreDisplay").innerText = "Pontos: " + score;
                if (score % 200 === 0) roadSpeed += 0.5;
            }
        }
        if (Math.random() < 0.02 && obstacles.length < 4) spawnObstacle();
    }

    function draw() {
        ctx.fillStyle = "#333"; ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#27ae60"; ctx.fillRect(0, 0, 15, canvas.height);
        ctx.fillRect(canvas.width - 15, 0, 15, canvas.height);

        ctx.strokeStyle = "#fff"; ctx.setLineDash([30, 30]);
        ctx.lineDashOffset = -lineOffset; ctx.lineWidth = 4;
        ctx.beginPath(); ctx.moveTo(canvas.width/2, 0); ctx.lineTo(canvas.width/2, canvas.height); ctx.stroke();

        obstacles.forEach(o => {
            if (o.type === 'hole') {
                ctx.fillStyle = "#1a1a1a"; ctx.beginPath(); ctx.arc(o.x+o.radius, o.y+o.radius, o.radius, 0, Math.PI*2); ctx.fill();
            } else {
                ctx.fillStyle = o.color; ctx.fillRect(o.x, o.y, o.w, o.h);
                ctx.fillStyle = "rgba(0,0,0,0.3)"; ctx.fillRect(o.x+5, o.y+45, o.w-10, 15);
            }
        });

        // Desenhar Jogador com a cor escolhida
        ctx.fillStyle = player.color;
        ctx.fillRect(player.x, player.y, player.w, player.h);
        ctx.fillStyle = "#34495e"; ctx.fillRect(player.x+5, player.y+15, player.w-10, 20); // Parabrisa
        ctx.fillStyle = "#f1c40f"; ctx.fillRect(player.x+5, player.y, 8, 4); ctx.fillRect(player.x+player.w-13, player.y, 8, 4); // Faróis

        if (gameActive) requestAnimationFrame(gameLoop);
    }

    function gameLoop() {
        update();
        draw();
    }
</script>
</body>
</html>