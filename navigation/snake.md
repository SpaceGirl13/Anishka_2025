---
layout: base
title: Snake
permalink: /snake/
---

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <style>
        /* General body and background styling */
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background-color: #D8BFD8; /* Light purple background */
            color: #fff;
            text-align: center; /* Center all content */
        }
        /* Canvas styling */
        canvas {
            display: none;
            border: 10px solid #FFFFFF;
            background-color: #000000;
            margin: 20px auto; /* Center the canvas horizontally */
            display: block;
        }
        /* General screens styling */
        #menu, #gameover, #setting {
            text-align: center;
            margin-top: 50px;
        }
        a {
            font-size: 24px;
            text-decoration: none;
            color: #fff;
            margin: 10px;
            cursor: pointer;
        }
        a:hover::before {
            content: ">";
            margin-right: 10px;
        }
        /* Settings buttons */
        input[type="radio"] {
            display: none;
        }
        label {
            cursor: pointer;
            padding: 5px 10px;
            border: 1px solid #fff;
            margin: 0 5px;
            display: inline-block;
            background-color: transparent;
            color: #fff;
        }
        input[type="radio"]:checked + label {
            background-color: #FFFFFF;
            color: #000;
        }
        /* Footer styling */
        footer {
            margin-top: 50px;
            font-size: 14px;
            color: #fff;
            text-align: center;
        }
        footer a {
            color: #fff;
            text-decoration: underline;
        }
    </style>

</head>
<body>
    <h2>Snake</h2>
    <div class="container">
        <header class="pb-3 mb-4 border-bottom border-primary text-dark">
            <p class="fs-4">Score: <span id="score_value">0</span></p>
        </header>
        <div class="container bg-secondary" style="text-align:center;">
            <!-- Main Menu -->
            <div id="menu" class="py-4 text-light">
                <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
                <a id="new_game" class="link-alert">new game</a>
                <a id="setting_menu" class="link-alert">settings</a>
            </div>
            <!-- Game Over -->
            <div id="gameover" class="py-4 text-light" style="display:none;">
                <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
                <a id="new_game1" class="link-alert">new game</a>
                <a id="setting_menu1" class="link-alert">settings</a>
                <p>High Score: <span id="high_score_value">0</span></p>
            </div>
            <!-- Play Screen -->
            <canvas id="snake" class="wrap" width = "320" height="320" tabindex="1"></canvas>
            <!-- Settings Screen -->
            <div id="setting" class="py-4 text-light" style="display:none;">
                <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
                <a id="new_game2" class="link-alert">new game</a>
                <br>
                <p>Speed:
                    <input id="speed1" type="radio" name="speed" value="120" checked/>
                    <label for="speed1">Slow</label>
                    <input id="speed2" type="radio" name="speed" value="75"/>
                    <label for="speed2">Normal</label>
                    <input id="speed3" type="radio" name="speed" value="35"/>
                    <label for="speed3">Fast</label>
                </p>
                <p>Wall:
                    <input id="wallon" type="radio" name="wall" value="1" checked/>
                    <label for="wallon">On</label>
                    <input id="walloff" type="radio" name="wall" value="0"/>
                    <label for="walloff">Off</label>
                </p>
                <p>Block Size:
                    <input id="block_small" type="radio" name="blocksize" value="10" checked />
                    <label for="block_small">Small</label>
                    <input id="block_medium" type="radio" name="blocksize" value="20" />
                    <label for="block_medium">Medium</label>
                    <input id="block_large" type="radio" name="blocksize" value="30" />
                    <label for="block_large">Large</label>
                </p>
            </div>
        </div>
    </div>
    <script>
        (function () {
            /* Game Variables */
            const canvas = document.getElementById("snake");
            const ctx = canvas.getContext("2d");
            let BLOCK = 10;
            let SCREEN;
            let snake = [];
            let snake_dir, snake_next_dir;
            let snake_speed = 120;
            let food = {
                x: 0,
                y: 0,
                image: new Image(),
                width: 512, // Set the width in pixels
                height: 512 // Set the height in pixels
            };
            let score = 0;
            let highScore = localStorage.getItem("highScore") || 0;
            /* Screen Constants */
            const SCREEN_MENU = -1, SCREEN_GAME_OVER = 1, SCREEN_SNAKE = 0, SCREEN_SETTING = 2;
            const screen_menu = document.getElementById("menu");
            const screen_game_over = document.getElementById("gameover");
            const screen_setting = document.getElementById("setting");
            const ele_score = document.getElementById("score_value");
            const ele_high_score = document.getElementById("high_score_value"); // Element to display high score
            /* Utility Functions */
            const showScreen = (screen) => {
                SCREEN = screen;
                screen_menu.style.display = screen === SCREEN_MENU ? "block" : "none";
                screen_game_over.style.display = screen === SCREEN_GAME_OVER ? "block" : "none";
                screen_setting.style.display = screen === SCREEN_SETTING ? "block" : "none";
                canvas.style.display = screen === SCREEN_SNAKE ? "block" : "none";
            };
            const addFood = () => {
                const fruitEmojis = ["🍎", "🍌", "🍉", "🍇", "🍒", "🍓", "🍍", "🥭"];
                food.x = Math.floor(Math.random() * (canvas.width / BLOCK)) * BLOCK / BLOCK;
                food.y = Math.floor(Math.random() * (canvas.height / BLOCK)) * BLOCK / BLOCK;
                foodemoji = fruitEmojis[Math.floor(Math.random() * fruitEmojis.length)];
            };
            // Update activeDot to use emoji for snake
            const activeDot = (x, y, emoji) => {
                ctx.font = `${BLOCK}px Arial`; // Set font size to match BLOCK size
                ctx.textAlign = "center"; // Center the emoji in the block
                ctx.textBaseline = "middle"; // Center the emoji vertically
                ctx.fillText(emoji, x * BLOCK + BLOCK / 2, y * BLOCK + BLOCK / 2); // Draw the emoji
            };
            const gameOver = () => {
                // Update high score if the current score is higher than the stored high score
                if (score > highScore) {
                    highScore = score;
                    localStorage.setItem("highScore", highScore);  // Store the new high score in localStorage
                }
                showScreen(SCREEN_GAME_OVER);
            };
            // Add the applyBlockSize function here
            const applyBlockSize = (newSize) => {
                BLOCK = newSize;
                canvas.width = 320
                canvas.height = 320
                addFood();
                clearTimeout(gameLoop);
                newGame(); // Restart the game with the new block size
            };
            let gameLoop;
            const mainLoop = () => {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                // Draw grid
                ctx.strokeStyle = "#444";
                for (let x = 0; x < canvas.width; x += BLOCK) {
                    ctx.beginPath();
                    ctx.moveTo(x, 0);
                    ctx.lineTo(x, canvas.height);
                    ctx.stroke();
                }
                for (let y = 0; y < canvas.height; y += BLOCK) {
                    ctx.beginPath();
                    ctx.moveTo(0, y);
                    ctx.lineTo(canvas.width, y);
                    ctx.stroke();
                }
                // Move the snake
                let head = { ...snake[0] };
                if (snake_next_dir !== undefined) snake_dir = snake_next_dir;
                if (snake_dir === 0) head.y--;
                if (snake_dir === 1) head.x++;
                if (snake_dir === 2) head.y++;
                if (snake_dir === 3) head.x--;
                // Check for collisions
                if (head.x < 0 || head.y < 0 || head.x >= canvas.width / BLOCK || head.y >= canvas.height / BLOCK)
                    return gameOver();
                snake.unshift(head);
                // Check for collisions
                if (wall_on) {
                    // Snake hits the wall
                    if (head.x < 0 || head.y < 0 || head.x >= canvas.width / BLOCK || head.y >= canvas.height / BLOCK) {
                        return gameOver();
                    }
                } else {
                    // Snake wraps around when wall is off
                    if (head.x < 0) head.x = canvas.width / BLOCK - 1;
                    if (head.y < 0) head.y = canvas.height / BLOCK - 1;
                    if (head.x >= canvas.width / BLOCK) head.x = 0;
                    if (head.y >= canvas.height / BLOCK) head.y = 0;
                }
                // Check for self-collision
                for (let i = 1; i < snake.length; i++) {
                    if (snake[i].x === head.x && snake[i].y === head.y) {
                        return gameOver(); // End game if snake collides with itself
                    }
                }
                if (head.x === food.x && head.y === food.y) {
                    score++;
                    ele_score.innerText = score;
                    addFood();
                } else snake.pop();
                // Draw snake and food
                activeDot(food.x, food.y, foodemoji);
                snake.forEach(part => activeDot(part.x, part.y,"🐍"));
                gameLoop = setTimeout(mainLoop, snake_speed);
            };
            const newGame = () => {
                showScreen(SCREEN_SNAKE);
                score = 0;
                ele_score.innerText = score;
                snake = [{ x: Math.floor(canvas.width / (2 * BLOCK)), y: Math.floor(canvas.height / (2 * BLOCK)) }];
                snake_dir = 1;
                snake_next_dir = undefined;
                addFood();
                mainLoop();
            };
            /* Event Listeners */
            window.addEventListener("keydown", (e) => {
                e.preventDefault();
                if (e.code === "Space" && SCREEN !== SCREEN_SNAKE) newGame();
                if (e.code === "ArrowUp" && snake_dir !== 2) snake_next_dir = 0;
                if (e.code === "ArrowRight" && snake_dir !== 3) snake_next_dir = 1;
                if (e.code === "ArrowDown" && snake_dir !== 0) snake_next_dir = 2;
                if (e.code === "ArrowLeft" && snake_dir !== 1) snake_next_dir = 3;
            });
            document.getElementById("new_game").onclick = newGame;
            document.getElementById("new_game1").onclick = newGame;
            // Settings menu toggle
            document.getElementById("setting_menu").onclick = () => showScreen(SCREEN_SETTING);
            document.getElementById("setting_menu1").onclick = () => showScreen(SCREEN_SETTING);
            // Apply settings
            document.getElementById("speed1").onclick = () => snake_speed = 120;
            document.getElementById("speed2").onclick = () => snake_speed = 75;
            document.getElementById("speed3").onclick = () => snake_speed = 35;
            let wall_on = true
            document.getElementById("wallon").onclick = () => wall_on = true;
            document.getElementById("walloff").onclick = () => wall_on = false;
            document.getElementById("block_small").onclick = () => applyBlockSize(10);
            document.getElementById("block_medium").onclick = () => applyBlockSize(20);
            document.getElementById("block_large").onclick = () => applyBlockSize(30);
            document.getElementById("wallon").addEventListener("click", () => {
            wall_on = true;
                console.log("Wall is ON");
            });
            document.getElementById("walloff").addEventListener("click", () => {
                wall_on = false;
                console.log("Wall is OFF");
            });
            ele_high_score.innerText = highScore;
        })();

    </script>

</body>
