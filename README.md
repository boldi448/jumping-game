<!DOCTYPE html>
<html lang="hu">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Egyszerű Platformer Játék</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }
        canvas {
            display: block;
            background-color: #87CEEB;
        }
        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 20px;
            font-family: Arial, sans-serif;
            color: black;
        }
    </style>
</head>
<body>
    <div id="score">Platform Magasság: 0</div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');

        canvas.width = 800;
        canvas.height = 600;

        const gravity = 0.5;
        const platformSpacing = 100;
        const maxFallDistance = platformSpacing * 4;

        const player = {
            x: canvas.width / 2,
            y: canvas.height - 50,
            radius: 25,
            color: 'blue',
            velocityY: 0,
            speed: 5,
            jumping: false,
            doubleJumpAvailable: true,
            maxHeight: 0
        };

        let platforms = [
            { x: 0, y: canvas.height - 50, width: canvas.width, height: 50, color: 'green' }
        ];

        let keys = {};

        window.addEventListener('keydown', (e) => keys[e.key] = true);
        window.addEventListener('keyup', (e) => keys[e.key] = false);

        function resetGame() {
            player.x = canvas.width / 2;
            player.y = canvas.height - 50;
            player.velocityY = 0;
            player.doubleJumpAvailable = true;
            player.maxHeight = 0;
            platforms = [
                { x: 0, y: canvas.height - 50, width: canvas.width, height: 50, color: 'green' }
            ];
            generatePlatform(canvas.height - 150);
            generatePlatform(canvas.height - 250);
            generatePlatform(canvas.height - 350);
        }

        function generatePlatform(y) {
            const width = Math.random() * 200 + 100;
            const x = Math.random() * (canvas.width - width);
            platforms.push({ x, y, width, height: 10, color: 'red' });
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            platforms = platforms.filter(p => p.y < canvas.height + 50);
            for (let platform of platforms) {
                ctx.fillStyle = platform.color;
                ctx.fillRect(platform.x, platform.y, platform.width, platform.height);
            }

            if (keys['ArrowRight'] || keys['d']) {
                player.x += player.speed;
            }
            if (keys['ArrowLeft'] || keys['a']) {
                player.x -= player.speed;
            }

            if ((keys['ArrowUp'] || keys['w'])) {
                if (!player.jumping) {
                    player.velocityY = -10;
                    player.jumping = true;
                } else if (player.doubleJumpAvailable) {
                    player.velocityY = -10;
                    player.doubleJumpAvailable = false;
                }
            }

            player.velocityY += gravity;
            player.y += player.velocityY;

            let onPlatform = false;
            for (let platform of platforms) {
                if (player.y + player.radius > platform.y &&
                    player.y - player.radius < platform.y + platform.height &&
                    player.x + player.radius > platform.x &&
                    player.x - player.radius < platform.x + platform.width) {
                    player.y = platform.y - player.radius;
                    player.jumping = false;
                    player.doubleJumpAvailable = true;
                    player.velocityY = 0;
                    player.fallDistance = 0;
                    onPlatform = true;
                    break;
                }
            }

            if (!onPlatform) {
                player.fallDistance += Math.abs(player.velocityY);
                if (player.fallDistance > maxFallDistance) {
                    resetGame();
                    return;
                }
            }

            const upperThirdY = canvas.height / 3;
            if (player.y < upperThirdY) {
                const offset = upperThirdY - player.y;
                player.y = upperThirdY;
                player.maxHeight += offset;
                platforms.forEach(p => p.y += offset);
            }

            if (player.y > canvas.height) {
                resetGame();
                return;
            }

            if (player.x > canvas.width) player.x = 0;
            if (player.x < 0) player.x = canvas.width;

            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
            ctx.fill();

            if (platforms.length > 0 && platforms[platforms.length - 1].y > platformSpacing) {
                generatePlatform(platforms[platforms.length - 1].y - platformSpacing);
            }

            scoreDisplay.textContent = `Platform Magasság: ${Math.floor(player.maxHeight / platformSpacing)}`;

            requestAnimationFrame(gameLoop);
        }

        resetGame();
        gameLoop();
    </script>
</body>
</html>
