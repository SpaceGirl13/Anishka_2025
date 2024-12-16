<style>
    body {
        margin: 0;
        overflow: hidden;
    }

    .wrap {
        margin-left: auto;
        margin-right: auto;
    }

    canvas {
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: #FFFFFF;
    }

    canvas:focus {
        outline: none;
    }

    #gameover p,
    #setting p,
    #menu p {
        font-size: 20px;
    }

    #gameover a,
    #setting a,
    #menu a {
        font-size: 30px;
        display: block;
    }

    #gameover a:hover,
    #setting a:hover,
    #menu a:hover {
        cursor: pointer;
    }

    #gameover a:hover::before,
    #setting a:hover::before,
    #menu a:hover::before {
        content: ">";
        margin-right: 10px;
    }

    #menu {
        display: block;
    }

    #gameover {
        display: none;
    }

    #setting {
        display: none;
    }

    #setting input {
        display: none;
    }

    #setting label {
        cursor: pointer;
    }

    #setting input:checked+label {
        background-color: #FFF;
        color: #000;
    }
</style>

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
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="120" checked />
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75" />
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35" />
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked />
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0" />
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
    (function () {
        const canvas = document.getElementById("snake");
        const ctx = canvas.getContext("2d");
        const SCREEN_MENU = -1, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2, SCREEN_SNAKE = 0;
        const screen_snake = document.getElementById("snake");
        const screen_menu = document.getElementById("menu");
        const screen_game_over = document.getElementById("gameover");
        const screen_setting = document.getElementById("setting");
        const ele_score = document.getElementById("score_value");
        const speed_setting = document.getElementsByName("speed");
        const wall_setting = document.getElementsByName("wall");
        const BLOCK = 10;
        let SCREEN = SCREEN_MENU;
        let snake, snake_dir, snake_next_dir, snake_speed, food = { x: 0, y: 0 }, score, wall;

        const showScreen = (screen_opt) => {
            SCREEN = screen_opt;
            screen_snake.style.display = screen_opt === SCREEN_SNAKE || screen_opt === SCREEN_GAME_OVER ? "block" : "none";
            screen_menu.style.display = screen_opt === SCREEN_MENU ? "block" : "none";
            screen_setting.style.display = screen_opt === SCREEN_SETTING ? "block" : "none";
            screen_game_over.style.display = screen_opt === SCREEN_GAME_OVER ? "block" : "none";
        };

        window.onload = function () {
            document.getElementById("new_game").onclick = newGame;
            document.getElementById("new_game1").onclick = newGame;
            document.getElementById("new_game2").onclick = newGame;
            document.getElementById("setting_menu").onclick = () => showScreen(SCREEN_SETTING);
            document.getElementById("setting_menu1").onclick = () => showScreen(SCREEN_SETTING);

            setSnakeSpeed(150);
            for (let i = 0; i < speed_setting.length; i++) {
                speed_setting[i].addEventListener("click", function () {
                    if (speed_setting[i].checked) setSnakeSpeed(speed_setting[i].value);
                });
            }

            setWall(1);
            for (let i = 0; i < wall_setting.length; i++) {
                wall_setting[i].addEventListener("click", function () {
                    if (wall_setting[i].checked) setWall(wall_setting[i].value);
                });
            }

            window.addEventListener("keydown", (evt) => {
                if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(evt.key)) evt.preventDefault();
                if (evt.code === "Space" && SCREEN !== SCREEN_SNAKE) newGame();
            });
        };

        const mainLoop = () => {
            ctx.fillStyle = "royalblue";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = "black";
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

            for (let i = 0; i < snake.length; i++) activeDot(snake[i].x, snake[i].y);
            activeDot(food.x, food.y);

            let _x = snake[0].x, _y = snake[0].y;
            snake_dir = snake_next_dir;
            switch (snake_dir) {
                case 0: _y--; break;
                case 1: _x++; break;
                case 2: _y++; break;
                case 3: _x--; break;
            }
            snake.pop();
            snake.unshift({ x: _x, y: _y });

            if (wall === 1 && (_x < 0 || _x >= canvas.width / BLOCK || _y < 0 || _y >= canvas.height / BLOCK)) return showScreen(SCREEN_GAME_OVER);

            if (wall === 0) {
                snake.forEach((part) => {
                    if (part.x < 0) part.x += canvas.width / BLOCK;
                    if (part.x >= canvas.width / BLOCK) part.x -= canvas.width / BLOCK;
                    if (part.y < 0) part.y += canvas.height / BLOCK;
                    if (part.y >= canvas.height / BLOCK) part.y -= canvas.height / BLOCK;
                });
            }

            for (let i = 1; i < snake.length; i++) if (checkBlock(_x, _y, snake[i].x, snake[i].y)) return showScreen(SCREEN_GAME_OVER);

            if (checkBlock(_x, _y, food.x, food.y)) {
                snake.push({});
                altScore(++score);
                addFood();
            }

            setTimeout(mainLoop, snake_speed);
        };

        const newGame = () => {
            showScreen(SCREEN_SNAKE);
            screen_snake.focus();
            score = 0;
            altScore(score);
            snake = [{ x: 0, y: 15 }];
            snake_next_dir = 1;
            addFood();
            screen_snake.onkeydown = (evt) => changeDir(evt.keyCode);
            mainLoop();
        };

        const activeDot = (x, y) => ctx.fillStyle = "#FFFFFF", ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
        const addFood = () => food.x = Math.floor(Math.random() * (canvas.width / BLOCK)), food.y = Math.floor(Math.random() * (canvas.height / BLOCK));
        const checkBlock = (x, y, _x, _y) => x === _x && y === _y;
        const altScore = (score_val) => ele_score.innerHTML = String(score_val);
        const setSnakeSpeed = (speed_value) => snake_speed = speed_value;
        const setWall = (wall_value) => wall = wall_value, screen_snake.style.borderColor = wall === 1 ? "#FFFFFF" : "#606060";
    })();
</script>
