<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Гра "Змійка"</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            font-family: Arial, sans-serif;
            background-color: #f0ebdc;
        }

        .container {
            display: flex;
            flex-direction: row;
        }

        .menu, .game-area {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .title {
            font-size: 48px;
            font-weight: bold;
            color: #3c281e;
            margin-bottom: 20px;
            text-shadow: 2px 2px #966f46;
        }

        .button {
            padding: 10px 30px;
            font-size: 24px;
            color: #f0ebdc;
            background-color: #64483a;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.2);
            transition: 0.3s;
            margin-top: 10px;
        }

        .button:hover {
            background-color: #805d4a;
            transform: scale(1.05);
            box-shadow: 4px 4px 12px rgba(0, 0, 0, 0.3);
        }

        canvas {
            display: none;
            border: 2px solid #966f46;
            border-radius: 10px;
            box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);
            background-color: #f0ebdc;
            margin-top: 20px;
        }

        #score {
            display: none;
            font-size: 18px;
            color: #3c281e;
            margin: 10px;
        }

        .history {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 10px;
            margin-left: 20px;
            border-left: 2px solid #ccc;
            height: 500px;
        }

        #history-list {
            max-height: 400px;
            overflow-y: auto;
            width: 220px;
            border: 1px solid #ccc;
            padding: 10px;
            margin-top: 10px;
            background-color: #fff;
        }

        .history-title {
            font-weight: bold;
            color: #3c281e;
        }

        /* Модальне вікно */
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .modal-content {
            background-color: #fff;
            padding: 30px;
            border-radius: 10px;
            text-align: center;
            width: 300px;
        }

        .modal button {
            padding: 10px 20px;
            background-color: #64483a;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .modal button:hover {
            background-color: #805d4a;
        }
    </style>
</head>
<body>

<div class="container">
    <!-- Ігрова зона -->
    <div class="game-area">
        <div class="menu" id="menu">
            <div class="title">Гра "Змійка"</div>
            <button class="button" onclick="startGame()">Старт</button>
        </div>
        <div id="score">Ваш рахунок: 0</div>
        <canvas id="gameCanvas" width="500" height="500"></canvas>
    </div>

    <!-- Історія гри -->
    <div class="history">
        <div class="history-title">Історія ігор</div>
        <button class="button" onclick="clearHistory()">Очистити історію</button>
        <div id="history-list"></div>
    </div>
</div>

<!-- Модальне вікно після програшу -->
<div id="gameOverModal" class="modal">
    <div class="modal-content">
        <h2>Ви програли!</h2>
        <p id="finalScore">Ваш рахунок: 0</p>
        <button onclick="restartGame()">Грати знову</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.getElementById('score');
    const historyList = document.getElementById('history-list');
    const gameOverModal = document.getElementById('gameOverModal');
    const finalScore = document.getElementById('finalScore');

    const BACKGROUND_COLOR = "#f0ebdc";
    const SNAKE_COLOR = "#006400";
    const FOOD_COLOR = "#ff6347";
    const SNAKE_BLOCK = 20;
    let snake = [{ x: 100, y: 100 }];
    let direction = { x: 0, y: 0 };
    let food = getRandomPosition();
    let score = 0;

    // Завантажити історію з Local Storage
    loadHistory();

    function startGame() {
        document.getElementById('menu').style.display = 'none';
        canvas.style.display = 'block';
        scoreDisplay.style.display = 'block';
        gameOverModal.style.display = 'none'; // Ховаємо модальне вікно, якщо воно відкрите
    }

    function getRandomPosition() {
        return {
            x: Math.floor(Math.random() * canvas.width / SNAKE_BLOCK) * SNAKE_BLOCK,
            y: Math.floor(Math.random() * canvas.height / SNAKE_BLOCK) * SNAKE_BLOCK
        };
    }

    function drawSnake() {
        ctx.fillStyle = SNAKE_COLOR;
        snake.forEach(segment => {
            ctx.fillRect(segment.x, segment.y, SNAKE_BLOCK, SNAKE_BLOCK);
        });
    }

    function moveSnake() {
        const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };
        snake.unshift(head);
        if (head.x === food.x && head.y === food.y) {
            score++;
            scoreDisplay.textContent = `Ваш рахунок: ${score}`;
            food = getRandomPosition();
        } else {
            snake.pop();
        }
    }

    function drawFood() {
        ctx.fillStyle = FOOD_COLOR;
        ctx.beginPath();
        ctx.arc(food.x + SNAKE_BLOCK / 2, food.y + SNAKE_BLOCK / 2, SNAKE_BLOCK / 2, 0, Math.PI * 2);
        ctx.fill();
    }

    function gameLoop() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        moveSnake();
        drawSnake();
        drawFood();
        if (checkCollision()) {
            showGameOverModal();
            saveScore(score);
            resetGame();
        }
    }

    function checkCollision() {
        const head = snake[0];
        if (head.x < 0 || head.x >= canvas.width || head.y < 0 || head.y >= canvas.height) return true;
        for (let i = 1; i < snake.length; i++) {
            if (snake[i].x === head.x && snake[i].y === head.y) return true;
        }
        return false;
    }

    function resetGame() {
        snake = [{ x: 100, y: 100 }];
        direction = { x: 0, y: 0 };
        food = getRandomPosition();
        score = 0;
        scoreDisplay.textContent = "Ваш рахунок: 0";
    }

    function showGameOverModal() {
        finalScore.textContent = `Ваш рахунок: ${score}`;
        gameOverModal.style.display = 'flex'; // Показуємо модальне вікно
    }

    function restartGame() {
        gameOverModal.style.display = 'none'; // Закриваємо модальне вікно
        startGame(); // Перезапускаємо гру
    }

    document.addEventListener('keydown', event => {
        switch (event.key) {
            case 'ArrowUp':
                if (direction.y === 0) direction = { x: 0, y: -SNAKE_BLOCK }; break;
            case 'ArrowDown':
                if (direction.y === 0) direction = { x: 0, y: SNAKE_BLOCK }; break;
            case 'ArrowLeft':
                if (direction.x === 0) direction = { x: -SNAKE_BLOCK, y: 0 }; break;
            case 'ArrowRight':
                if (direction.x === 0) direction = { x: SNAKE_BLOCK, y: 0 }; break;
        }
    });

    function saveScore(score) {
        const now = new Date();
        const timestamp = now.toLocaleString();
        const history = JSON.parse(localStorage.getItem('gameHistory')) || [];
        history.push({ score, timestamp });
        localStorage.setItem('gameHistory', JSON.stringify(history));
        loadHistory();
    }

    function loadHistory() {
        const history = JSON.parse(localStorage.getItem('gameHistory')) || [];
        historyList.innerHTML = '';
        history.forEach(entry => {
            const historyItem = document.createElement('div');
            historyItem.textContent = `${entry.timestamp} - Рахунок: ${entry.score}`;
            historyList.appendChild(historyItem);
        });
    }

    function clearHistory() {
        localStorage.removeItem('gameHistory');
        loadHistory();
    }

    function gameStart() {
        setInterval(gameLoop, 100);
    }

    gameStart(); // Починаємо гру
</script>

</body>
</html>
