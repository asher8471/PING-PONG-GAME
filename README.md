<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Super Fun Ping Pong</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #1a1a1a;
            font-family: 'Arial', sans-serif;
            color: #fff;
        }
        canvas {
            border: 2px solid #00ffcc;
            border-radius: 10px;
            box-shadow: 0 0 20px #00ffcc;
        }
        .modal {
            display: none;
            position: fixed;
            z-index: 1;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
        }
        .modal-content {
            background-color: #333;
            margin: 15% auto;
            padding: 20px;
            border: 2px solid #00ffcc;
            border-radius: 10px;
            width: 300px;
            text-align: center;
            box-shadow: 0 0 20px #00ffcc;
        }
        .close {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close:hover,
        .close:focus {
            color: #fff;
            text-decoration: none;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            margin: 10px 0;
            padding: 10px;
            background-color: #444;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        li:hover {
            background-color: #555;
        }
        button {
            padding: 10px 20px;
            background-color: #00ffcc;
            border: none;
            border-radius: 5px;
            color: #000;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #00cc99;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <div id="upgradeModal" class="modal">
        <div class="modal-content">
            <span class="close">×</span>
            <h2>Upgrades</h2>
            <p>Points: <span id="pointsDisplay">0</span></p>
            <ul id="upgradeList"></ul>
        </div>
    </div>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Game objects
        let playerPaddle = { x: 10, y: 250, width: 10, height: 100, dy: 0, color: '#00ffcc' };
        let aiPaddle = { x: 780, y: 250, width: 10, height: 100, dy: 0, color: '#ff00cc' };
        let ball = { x: 400, y: 300, radius: 10, dx: 5, dy: 5, speed: 5, color: '#ffff00' };
        let playerScore = 0;
        let aiScore = 0;
        let playerPoints = parseInt(localStorage.getItem('playerPoints')) || 0;
        let purchasedUpgrades = JSON.parse(localStorage.getItem('purchasedUpgrades')) || [];

        // Rain effect
        let rainDrops = [];
        for (let i = 0; i < 100; i++) {
            rainDrops.push({
                x: Math.random() * canvas.width,
                y: -Math.random() * canvas.height,
                speed: Math.random() * 5 + 5
            });
        }

        // Power-ups
        const POWER_UP_TYPES = {
            BIG_PADDLE: 'big_paddle',
            FAST_BALL: 'fast_ball',
            SLOW_BALL: 'slow_ball'
        };
        let powerUps = [];
        let nextPowerUpTime = Date.now() + Math.random() * 5000 + 5000;

        // Upgrades
        const UPGRADES = [
            { name: 'Increase Paddle Size', cost: 100, effect: () => playerPaddle.height += 10 },
            { name: 'Decrease Ball Speed', cost: 150, effect: () => { if (ball.speed > 1) ball.speed -= 1; } },
            { name: 'Change Paddle Color', cost: 50, effect: () => playerPaddle.color = '#ff00cc' }
        ];

        // Apply saved upgrades
        for (const upgrade of purchasedUpgrades) {
            UPGRADES.find(u => u.name === upgrade).effect();
        }

        // Sound effects (replace paths with actual audio files)
        const ballHitSound = new Audio('path/to/ball_hit.mp3');
        const wallBounceSound = new Audio('path/to/wall_bounce.mp3');
        const scoreSound = new Audio('path/to/score.mp3');

        // Player controls
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowUp') playerPaddle.dy = -5;
            if (e.key === 'ArrowDown') playerPaddle.dy = 5;
        });
        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowUp' || e.key === 'ArrowDown') playerPaddle.dy = 0;
        });

        // Modal controls
        const modal = document.getElementById('upgradeModal');
        const closeModal = document.getElementsByClassName('close')[0];
        closeModal.onclick = () => modal.style.display = 'none';
        window.onclick = (event) => {
            if (event.target == modal) modal.style.display = 'none';
        };

        function displayUpgrades() {
            const pointsDisplay = document.getElementById('pointsDisplay');
            const upgradeList = document.getElementById('upgradeList');
            pointsDisplay.textContent = playerPoints;
            upgradeList.innerHTML = '';
            for (const upgrade of UPGRADES) {
                if (!purchasedUpgrades.includes(upgrade.name)) {
                    const li = document.createElement('li');
                    li.textContent = `${upgrade.name} - ${upgrade.cost} points`;
                    li.onclick = function() {
                        if (playerPoints >= upgrade.cost) {
                            playerPoints -= upgrade.cost;
                            purchasedUpgrades.push(upgrade.name);
                            upgrade.effect();
                            saveProgress();
                            displayUpgrades();
                        }
                    };
                    upgradeList.appendChild(li);
                }
            }
            modal.style.display = 'block';
        }

        function saveProgress() {
            localStorage.setItem('playerPoints', playerPoints);
            localStorage.setItem('purchasedUpgrades', JSON.stringify(purchasedUpgrades));
        }

        // Drawing functions
        function drawRain() {
            ctx.strokeStyle = 'rgba(0, 255, 255, 0.5)';
            ctx.lineWidth = 1;
            for (const drop of rainDrops) {
                ctx.beginPath();
                ctx.moveTo(drop.x, drop.y);
                ctx.lineTo(drop.x, drop.y + 10);
                ctx.stroke();
                drop.y += drop.speed;
                if (drop.y > canvas.height) drop.y = -Math.random() * canvas.height;
            }
        }

        function drawPaddle(paddle) {
            ctx.fillStyle = paddle.color;
            ctx.fillRect(paddle.x, paddle.y, paddle.width, paddle.height);
        }

        function drawBall() {
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fillStyle = ball.color;
            ctx.fill();
            ctx.closePath();
        }

        function drawScore() {
            ctx.fillStyle = '#fff';
            ctx.font = '24px Arial';
            ctx.fillText(`Player: ${playerScore}`, 50, 30);
            ctx.fillText(`AI: ${aiScore}`, 700, 30);
        }

        // Movement logic
        function movePaddles() {
            playerPaddle.y += playerPaddle.dy;
            if (playerPaddle.y < 0) playerPaddle.y = 0;
            if (playerPaddle.y + playerPaddle.height > canvas.height) playerPaddle.y = canvas.height - playerPaddle.height;

            // Basic AI for right paddle
            if (ball.y < aiPaddle.y + aiPaddle.height / 2) aiPaddle.dy = -3;
            else aiPaddle.dy = 3;
            aiPaddle.y += aiPaddle.dy;
            if (aiPaddle.y < 0) aiPaddle.y = 0;
            if (aiPaddle.y + aiPaddle.height > canvas.height) aiPaddle.y = canvas.height - aiPaddle.height;
        }

        function moveBall() {
            ball.x += ball.dx;
            ball.y += ball.dy;

            // Wall bounce
            if (ball.y - ball.radius < 0 || ball.y + ball.radius > canvas.height) {
                ball.dy = -ball.dy;
                wallBounceSound.play();
            }

            // Paddle collision
            if (isBallCollidingWithPaddle(ball, playerPaddle)) {
                ball.dx = ball.speed;
                ballHitSound.play();
            }
            if (isBallCollidingWithPaddle(ball, aiPaddle)) {
                ball.dx = -ball.speed;
                ballHitSound.play();
            }

            // Scoring
            if (ball.x - ball.radius < 0) {
                aiScore++;
                playerPoints += 10;
                resetBall();
                scoreSound.play();
            }
            if (ball.x + ball.radius > canvas.width) {
                playerScore++;
                playerPoints += 10;
                resetBall();
                scoreSound.play();
            }
        }

        function resetBall() {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.dx = ball.speed * (Math.random() > 0.5 ? 1 : -1);
            ball.dy = ball.speed * (Math.random() > 0.5 ? 1 : -1);
        }

        // Power-up logic
        function spawnPowerUp() {
            if (Date.now() > nextPowerUpTime) {
                const type = getRandomPowerUpType();
                const x = Math.random() * canvas.width;
                const y = -10;
                powerUps.push({ type, x, y, width: 10, height: 10, duration: 10000, startTime: Date.now() });
                nextPowerUpTime = Date.now() + Math.random() * 5000 + 5000;
            }
        }

        function getRandomPowerUpType() {
            const types = Object.values(POWER_UP_TYPES);
            return types[Math.floor(Math.random() * types.length)];
        }

        function updatePowerUps() {
            for (let i = powerUps.length - 1; i >= 0; i--) {
                const powerUp = powerUps[i];
                powerUp.y += 1;
                if (powerUp.y > canvas.height) {
                    powerUps.splice(i, 1);
                } else if (isCollision(playerPaddle, powerUp)) {
                    applyPowerUpEffect(powerUp.type);
                    powerUps.splice(i, 1);
                } else if (Date.now() - powerUp.startTime > powerUp.duration) {
                    powerUps.splice(i, 1);
                } else {
                    ctx.fillStyle = 'yellow';
                    ctx.fillRect(powerUp.x, powerUp.y, powerUp.width, powerUp.height);
                }
            }
        }

        function applyPowerUpEffect(type) {
            switch (type) {
                case POWER_UP_TYPES.BIG_PADDLE:
                    playerPaddle.height *= 1.5;
                    setTimeout(() => playerPaddle.height /= 1.5, 10000);
                    break;
                case POWER_UP_TYPES.FAST_BALL:
                    ball.speed *= 1.5;
                    setTimeout(() => ball.speed /= 1.5, 10000);
                    break;
                case POWER_UP_TYPES.SLOW_BALL:
                    ball.speed /= 1.5;
                    setTimeout(() => ball.speed *= 1.5, 10000);
                    break;
            }
        }

        // Collision detection
        function isBallCollidingWithPaddle(ball, paddle) {
            return ball.x - ball.radius < paddle.x + paddle.width &&
                   ball.x + ball.radius > paddle.x &&
                   ball.y - ball.radius < paddle.y + paddle.height &&
                   ball.y + ball.radius > paddle.y;
        }

        function isCollision(paddle, obj) {
            return paddle.x < obj.x + obj.width &&
                   paddle.x + paddle.width > obj.x &&
                   paddle.y < obj.y + obj.height &&
                   paddle.y + paddle.height > obj.y;
        }

        // Game loop
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawRain();
            drawPaddle(playerPaddle);
            drawPaddle(aiPaddle);
            drawBall();
            drawScore();
            movePaddles();
            moveBall();
            spawnPowerUp();
            updatePowerUps();
            requestAnimationFrame(gameLoop);
        }

        // Initialize and start
        resetBall();
        gameLoop();

        // Upgrade button
        const upgradeButton = document.createElement('button');
        upgradeButton.textContent = 'Upgrades';
        upgradeButton.style.position = 'absolute';
        upgradeButton.style.top = '10px';
        upgradeButton.style.left = '10px';
        upgradeButton.onclick = displayUpgrades;
        document.body.appendChild(upgradeButton);
    </script>
</body>
</html>
