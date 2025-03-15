<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ball Game</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #70c5ce;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        #game {
            position: relative;
            width: 400px;
            height: 600px;
            background-color: #70c5ce;
            border: 2px solid #000;
            overflow: hidden;
        }
        #ball {
            position: absolute;
            top: 250px;
            left: 50px;
            width: 40px;
            height: 40px;
            background-color: red;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
        }
        .pipe {
            position: absolute;
            width: 60px;
            background-color: green;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
        }
        #score {
            text-align: center;
            font-size: 24px;
            margin-top: 20px;
            color: #000;
        }
        #startButton {
            padding: 10px 20px;
            font-size: 18px;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
        }
        #gameOver {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 24px;
            color: red;
            text-align: center;
        }
        .power-up {
            position: absolute;
            width: 20px;
            height: 20px;
            background-color: gold;
            border-radius: 50%;
            animation: blink 1s infinite;
        }
        @keyframes blink {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }
    </style>
</head>
<body>
    <div id="game">
        <div id="ball"></div>
        <div id="gameOver">Game Over!<br>Score: <span id="finalScore">0</span><br><button id="restartButton">Restart</button></div>
    </div>
    <div id="score">Score: 0</div>
    <button id="startButton">Start Game</button>

    <script>
        const ball = document.getElementById("ball");
        const game = document.getElementById("game");
        const scoreDisplay = document.getElementById("score");
        const startButton = document.getElementById("startButton");
        const gameOverDiv = document.getElementById("gameOver");
        const finalScore = document.getElementById("finalScore");
        const restartButton = document.getElementById("restartButton");

        let ballY = 250;
        let gravity = 2;
        let jump = -30;
        let score = 0;
        let pipeInterval = 2000;
        let gameOver = false;
        let gameStarted = false;

        // Start game
        startButton.addEventListener("click", () => {
            startGame();
        });

        // Restart game
        restartButton.addEventListener("click", () => {
            location.reload();
        });

        function startGame() {
            gameStarted = true;
            startButton.style.display = "none";
            applyGravity();
            setInterval(createPipe, pipeInterval);
            setInterval(createPowerUp, 5000); // Power-up every 5 seconds
        }

        // Ball movement
        function applyGravity() {
            if (!gameOver && gameStarted) {
                ballY += gravity;
                ball.style.top = ballY + "px";
                checkCollision();
                requestAnimationFrame(applyGravity);
            }
        }

        // Jump on spacebar
        document.addEventListener("keydown", function (event) {
            if (event.code === "Space" && !gameOver && gameStarted) {
                ballY += jump;
                ball.style.top = ballY + "px";
                playSound("jump");
            }
        });

        // Create pipes
        function createPipe() {
            if (gameOver || !gameStarted) return;

            const pipeGap = 150;
            const pipeHeight = Math.random() * (game.clientHeight - pipeGap);
            const topPipe = document.createElement("div");
            const bottomPipe = document.createElement("div");

            topPipe.classList.add("pipe");
            bottomPipe.classList.add("pipe");

            topPipe.style.height = pipeHeight + "px";
            topPipe.style.top = "0";
            topPipe.style.left = game.clientWidth + "px";

            bottomPipe.style.height = (game.clientHeight - pipeHeight - pipeGap) + "px";
            bottomPipe.style.bottom = "0";
            bottomPipe.style.left = game.clientWidth + "px";

            game.appendChild(topPipe);
            game.appendChild(bottomPipe);

            movePipes(topPipe, bottomPipe);
        }

        // Move pipes
        function movePipes(topPipe, bottomPipe) {
            let pipeX = game.clientWidth;
            const pipeSpeed = 2;

            const pipeIntervalId = setInterval(() => {
                if (gameOver || !gameStarted) {
                    clearInterval(pipeIntervalId);
                    return;
                }

                pipeX -= pipeSpeed;
                topPipe.style.left = pipeX + "px";
                bottomPipe.style.left = pipeX + "px";

                if (pipeX < -60) {
                    game.removeChild(topPipe);
                    game.removeChild(bottomPipe);
                    clearInterval(pipeIntervalId);
                    score++;
                    scoreDisplay.textContent = "Score: " + score;
                    increaseDifficulty();
                }

                checkCollisionWithPipes(topPipe, bottomPipe);
            }, 20);
        }

        // Create power-ups
        function createPowerUp() {
            if (gameOver || !gameStarted) return;

            const powerUp = document.createElement("div");
            powerUp.classList.add("power-up");
            powerUp.style.top = Math.random() * (game.clientHeight - 20) + "px";
            powerUp.style.left = game.clientWidth + "px";
            game.appendChild(powerUp);

            movePowerUp(powerUp);
        }

        // Move power-ups
        function movePowerUp(powerUp) {
            let powerUpX = game.clientWidth;
            const powerUpSpeed = 2;

            const powerUpIntervalId = setInterval(() => {
                if (gameOver || !gameStarted) {
                    clearInterval(powerUpIntervalId);
                    return;
                }

                powerUpX -= powerUpSpeed;
                powerUp.style.left = powerUpX + "px";

                if (powerUpX < -20) {
                    game.removeChild(powerUp);
                    clearInterval(powerUpIntervalId);
                }

                checkCollisionWithPowerUp(powerUp);
            }, 20);
        }

        // Check collision with power-ups
        function checkCollisionWithPowerUp(powerUp) {
            const ballRect = ball.getBoundingClientRect();
            const powerUpRect = powerUp.getBoundingClientRect();

            if (
                ballRect.left < powerUpRect.right &&
                ballRect.right > powerUpRect.left &&
                ballRect.top < powerUpRect.bottom &&
                ballRect.bottom > powerUpRect.top
            ) {
                game.removeChild(powerUp);
                score += 5; // Bonus points
                scoreDisplay.textContent = "Score: " + score;
                playSound("powerUp");
            }
        }

        // Increase difficulty
        function increaseDifficulty() {
            if (score % 5 === 0) {
                pipeInterval -= 100; // Pipes faster
                gravity += 0.2; // Gravity increases
            }
        }

        // Check collision with pipes
        function checkCollisionWithPipes(topPipe, bottomPipe) {
            const ballRect = ball.getBoundingClientRect();
            const topPipeRect = topPipe.getBoundingClientRect();
            const bottomPipeRect = bottomPipe.getBoundingClientRect();

            if (
                ballRect.left < topPipeRect.right &&
                ballRect.right > topPipeRect.left &&
                (ballRect.top < topPipeRect.bottom || ballRect.bottom > bottomPipeRect.top)
            ) {
                endGame();
            }
        }

        // Check collision with ground or sky
        function checkCollision() {
            if (ballY > game.clientHeight - 40 || ballY < 0) {
                endGame();
            }
        }

        // End game
        function endGame() {
            gameOver = true;
            gameOverDiv.style.display = "block";
            finalScore.textContent = score;
            playSound("gameOver");
        }

        // Play sound
        function playSound(sound) {
            const audio = new Audio();
            if (sound === "jump") {
                audio.src = "https://www.soundjay.com/button/beep-07.mp3"; // Jump sound
            } else if (sound === "gameOver") {
                audio.src = "https://www.soundjay.com/misc/sounds/fail-trombone-01.mp3"; // Game over sound
            } else if (sound === "powerUp") {
                audio.src = "https://www.soundjay.com/misc/sounds/cash-register-01.mp3"; // Power-up sound
            }
            audio.play();
        }
    </script>
</body>
</html>
