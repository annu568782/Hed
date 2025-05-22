car racing #
<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Red Green Road Run Car Racing</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            position: relative;
            width: 100vw;
            height: 100vh;
            background-color: #333;
        }
        #road { running car 
            position: absolute;
            width: 300px;
            height: 100%;
            background-color: #555;
            left: 50%;
            transform: translateX(-50%);
            overflow: hidden;
        }
        #laneMarkers { stop 
            position: absolute;
            width: 100%;
            height: 200%;
            background: repeating-linear-gradient(
                to bottom,
                #fff,
                #fff 50px,
                transparent 50px,
                transparent 100px
            );
            animation: moveRoad 2s linear infinite;
        }
        #playerCar {
            position: absolute;
            width: 40px;
            height: 70px;
            background-color: red;
            bottom: 50px;
            left: 50%;
            transform: translateX(-50%);
        }
        .obstacle {
            position: absolute;
            width: 40px;
            height: 70px;
            background-color: blue;
        }
        #controls {
            position: fixed;
            bottom: 20px;
            width: 100%;
            display: flex;
            justify-content: center;
            gap: 20px;
        }
        .control-btn {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            border: none;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
        }
        #leftBtn {
            background-color: red;
            color: white;
        }
        #rightBtn {
            background-color: green;
            color: white;
        }
        #scoreDisplay {
            position: fixed;
            top: 20px;
            left: 20px;
            color: white;
            font-size: 24px;
        }
        #startBtn {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 15px 30px;
            font-size: 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="road">
            <div id="laneMarkers"></div>
            <div id="playerCar"></div>
        </div>
        <div id="scoreDisplay">Score: 0</div>
        <div id="controls">
            <button class="control-btn" id="leftBtn">LEFT</button>
            <button class="control-btn" id="rightBtn">RIGHT</button>
        </div>
        <button id="startBtn">START RACE</button>
    </div>

    <script>
        const playerCar = document.getElementById('playerCar');
        const road = document.getElementById('road');
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const startBtn = document.getElementById('startBtn');
        const gameContainer = document.getElementById('gameContainer');
        
        let gameActive = false;
        let score = 0;
        let playerPosition = 1; // 0: left, 1: center, 2: right
        let obstacles = [];
        let gameSpeed = 2;
        let gameInterval;
        
        // Lane positions
        const lanePositions = [
            road.offsetLeft + 50, 
            road.offsetLeft + road.offsetWidth/2 - playerCar.offsetWidth/2, 
            road.offsetLeft + road.offsetWidth - 50 - playerCar.offsetWidth
        ];
        
        // Initialize player car position
        playerCar.style.left = lanePositions[1] + 'px';
        
        // Start game
        startBtn.addEventListener('click', startGame);
        
        function startGame() {
            startBtn.style.display = 'none';
            gameActive = true;
            score = 0;
            playerPosition = 1;
            obstacles = [];
            gameSpeed = 2;
            updateScore();
            
            // Clear existing obstacles
            document.querySelectorAll('.obstacle').forEach(obs => obs.remove());
            
            // Set up game loop
            if (gameInterval) clearInterval(gameInterval);
            gameInterval = setInterval(updateGame, 20);
            
            // Start generating obstacles
            setTimeout(generateObstacle, 1000);
        }
        
        function updateGame() {
            if (!gameActive) return;
            
            // Move obstacles
            obstacles.forEach((obstacle, index) => {
                const top = parseInt(obstacle.style.top) + gameSpeed;
                obstacle.style.top = top + 'px';
                
                // Check collision
                if (top + obstacle.offsetHeight > playerCar.offsetTop && 
                    top < playerCar.offsetTop + playerCar.offsetHeight &&
                    parseInt(obstacle.style.left) === parseInt(playerCar.style.left)) {
                    endGame();
                }
                
                // Remove obstacle when off screen
                if (top > gameContainer.offsetHeight) {
                    obstacle.remove();
                    obstacles.splice(index, 1);
                    score += 10;
                    updateScore();
                    
                    // Increase speed every 100 points
                    if (score % 100 === 0) {
                        gameSpeed += 0.5;
                    }
                }
            });
        }
        
        function generateObstacle() {
            if (!gameActive) return;
            
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle';
            
            // Random lane (0, 1, or 2)
            const lane = Math.floor(Math.random() * 3);
            obstacle.style.left = lanePositions[lane] + 'px';
            obstacle.style.top = '-70px';
            
            road.appendChild(obstacle);
            obstacles.push(obstacle);
            
            // Schedule next obstacle
            const delay = Math.max(500, 2000 - score * 5); // Decrease delay as score increases
            setTimeout(generateObstacle, delay);
        }
        
        function updateScore() {
            scoreDisplay.textContent = `Score: ${score}`;
        }
        
        function endGame() {
            gameActive = false;
            clearInterval(gameInterval);
            startBtn.style.display = 'block';
            startBtn.textContent = 'TRY AGAIN';
        }
        
        // Control buttons
        leftBtn.addEventListener('click', () => {
            if (!gameActive) return;
            playerPosition = Math.max(0, playerPosition - 1);
            playerCar.style.left = lanePositions[playerPosition] + 'px';
        });
        
        rightBtn.addEventListener('click', () => {
            if (!gameActive) return;
            playerPosition = Math.min(2, playerPosition + 1);
            playerCar.style.left = lanePositions[playerPosition] + 'px';
        });
        
        // Keyboard controls
        document.addEventListener('keydown', (e) => {
            if (!gameActive) return;
            
            if (e.key === 'ArrowLeft' || e.key === 'a') {
                playerPosition = Math.max(0, playerPosition - 1);
                playerCar.style.left = lanePositions[playerPosition] + 'px';
            } else if (e.key === 'ArrowRight' || e.key === 'd') {
                playerPosition = Math.min(2, playerPosition + 1);
                playerCar.style.left = lanePositions[playerPosition] + 'px';
            }
        });
    </script>
</body>
</html>
