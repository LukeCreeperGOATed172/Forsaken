<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Forsaken: Skate & Survive</title>
    <style>
        body {
            background-color: #0a0a0a;
            color: #fff;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start; /* Aligns to top so it's safely scrollable */
            min-height: 100vh;
            margin: 0;
            padding: 20px 10px; /* Gives nice breathing room */
            box-sizing: border-box;
            overflow-y: auto; /* Allows scrolling if window is too small */
            user-select: none;
        }
        h1 { color: #800; margin: 0 0 5px 0; text-shadow: 0 0 10px #f00; letter-spacing: 2px; text-align: center;}
        
        /* Layout wrapper to hold everything tight */
        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 800px;
            width: 100%;
        }

        #gameCanvas {
            border: 4px solid #300;
            background-color: #141414;
            box-shadow: 0 0 30px #000;
            max-width: 100%;
            height: auto;
        }
        #ui { width: 100%; display: flex; justify-content: space-between; margin-bottom: 10px; font-weight: bold; font-size: 18px;}
        .bars-container { display: flex; flex-direction: column; gap: 5px; width: 100%; margin-top: 8px; }
        .bar-row { display: flex; justify-content: space-between; font-size: 14px; gap: 10px; }
        .stamina-bar, .health-bar { width: 140px; height: 12px; background: #333; border: 1px solid #555; display: inline-block; vertical-align: middle;}
        .fill { height: 100%; transition: width 0.1s; }
        #p1-stamina-fill { background: #00bcff; width: 100%; }
        #p2-stamina-fill { background: #ff0000; width: 100%; }
        #p1-health-fill { background: #00ff00; width: 100%; }
        .instructions { color: #aaa; font-size: 13px; margin: 15px 0 0 0; text-align: center; line-height: 1.5; background: #111; padding: 10px; border: 1px solid #222; border-radius: 4px; width: 100%; box-sizing: border-box; }
        #timer { color: #ffcc00; font-size: 22px; }
    </style>
</head>
<body>

    <div class="game-container">
        <h1>FORSAKEN: SKATE & SURVIVE</h1>
        
        <div id="ui">
            <div id="status" style="color: #0f0;">SURVIVE THE MATCH!</div>
            <div id="timer">60s</div>
        </div>

        <!-- Shrunk height from 500 to 450 to fit screens much better -->
        <canvas id="gameCanvas" width="800" height="450"></canvas>

        <div class="bars-container">
            <div class="bar-row">
                <div>
                    <span style="color: #00bcff;">P1 Skater Stamina:</span>
                    <div class="stamina-bar"><div id="p1-stamina-fill" class="fill"></div></div>
                </div>
                <div>
                    <span style="color: #ff0000;">P2 Killer Stamina:</span>
                    <div class="stamina-bar"><div id="p2-stamina-fill" class="fill"></div></div>
                </div>
            </div>
            <div class="bar-row">
                <div>
                    <span style="color: #00ff00;">P1 Skater Health:</span>
                    <div class="health-bar"><div id="p1-health-fill" class="fill"></div></div>
                </div>
                <div style="color: #888; font-size: 12px; font-weight: bold;">Press [R] to Restart Match</div>
            </div>
        </div>

        <div class="instructions">
            <strong>P1 SURVIVOR (Skater):</strong> W/A/S/D to Skate | Hold SHIFT to Sprint<br>
            <strong>P2 KILLER (Guest 666):</strong> ARROW KEYS to Hunt | Hold SPACEBAR to Sprint | Press <strong>'/' or Numpad Enter</strong> to Attack
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        let timeLeft, gameOver, gameWon, countdown, p1, p2;

        const obstacles = [
            { x: 180, y: 100, w: 80, h: 80 },
            { x: 540, y: 270, w: 80, h: 80 },
            { x: 360, y: 170, w: 80, h: 100 },
            { x: 180, y: 310, w: 120, h: 40 },
            { x: 500, y: 100, w: 120, h: 40 }
        ];

        const keys = {};

        function initGame() {
            if (countdown) clearInterval(countdown);
            
            timeLeft = 60;
            gameOver = false;
            gameWon = false;

            document.getElementById("status").innerText = "SURVIVE THE MATCH!";
            document.getElementById("status").style.color = "#0f0";
            document.getElementById("timer").innerText = "60s";

            p1 = {
                x: 100, y: 225, size: 15,
                baseSpeed: 4.5, sprintSpeed: 7,
                stamina: 100, health: 100,
                color: "#00bcff", iFrames: 0
            };

            p2 = {
                x: 700, y: 225, size: 22,
                baseSpeed: 3.5, sprintSpeed: 6.5,
                stamina: 100, color: "#ff0000",
                attackCooldown: 0
            };

            countdown = setInterval(() => {
                if (gameOver || gameWon) return;
                timeLeft--;
                document.getElementById("timer").innerText = `${timeLeft}s`;

                if (timeLeft <= 0) {
                    gameWon = true;
                    document.getElementById("status").innerText = "SKATER SURVIVED! SURVIVORS WIN!";
                    document.getElementById("status").style.color = "#00ff00";
                }
            }, 1000);
        }

        window.addEventListener("keydown", e => {
            if ([" ", "/", "arrowup", "arrowdown", "arrowleft", "arrowright"].includes(e.key.toLowerCase())) {
                e.preventDefault();
            }
            keys[e.key.toLowerCase()] = true;
            keys[e.key] = true; 

            if (e.key.toLowerCase() === 'r') {
                initGame();
            }
        }, { passive: false });

        window.addEventListener("keyup", e => {
            keys[e.key.toLowerCase()] = false;
            keys[e.key] = false;
        });

        function checkCollision(x, y, size) {
            for (let obs of obstacles) {
                if (x + size > obs.x && x - size < obs.x + obs.w &&
                    y + size > obs.y && y - size < obs.y + obs.h) {
                    return true;
                }
            }
            return false;
        }

        function update() {
            if (gameOver || gameWon) return;

            if (p1.iFrames > 0) p1.iFrames--;
            if (p2.attackCooldown > 0) p2.attackCooldown--;

            // --- P1 (SKATER) ---
            let p1Speed = p1.baseSpeed;
            if (keys["shift"] && p1.stamina > 0 && (keys["w"] || keys["a"] || keys["s"] || keys["d"])) {
                p1Speed = p1.sprintSpeed;
                p1.stamina = Math.max(0, p1.stamina - 0.6);
            } else {
                p1.stamina = Math.min(100, p1.stamina + 0.2);
            }

            let nextP1X = p1.x;
            let nextP1Y = p1.y;
            if (keys["w"]) nextP1Y -= p1Speed;
            if (keys["s"]) nextP1Y += p1Speed;
            if (keys["a"]) nextP1X -= p1Speed;
            if (keys["d"]) nextP1X += p1Speed;

            if (nextP1X > p1.size && nextP1X < canvas.width - p1.size && !checkCollision(nextP1X, p1.y, p1.size)) p1.x = nextP1X;
            if (nextP1Y > p1.size && nextP1Y < canvas.height - p1.size && !checkCollision(p1.x, nextP1Y, p1.size)) p1.y = nextP1Y;

            // --- P2 (KILLER) ---
            let p2Speed = p2.baseSpeed;
            if (keys[" "] && p2.stamina > 0 && (keys["arrowup"] || keys["arrowdown"] || keys["arrowleft"] || keys["arrowright"])) {
                p2Speed = p2.sprintSpeed;
                p2.stamina = Math.max(0, p2.stamina - 0.8);
            } else {
                p2.stamina = Math.min(100, p2.stamina + 0.15);
            }

            let nextP2X = p2.x;
            let nextP2Y = p2.y;
            if (keys["arrowup"]) nextP2Y -= p2Speed;
            if (keys["arrowdown"]) nextP2Y += p2Speed;
            if (keys["arrowleft"]) nextP2X -= p2Speed;
            if (keys["arrowright"]) nextP2X += p2Speed;

            if (nextP2X > p2.size && nextP2X < canvas.width - p2.size && !checkCollision(nextP2X, p2.y, p2.size)) p2.x = nextP2X;
            if (nextP2Y > p2.size && nextP2Y < canvas.height - p2.size && !checkCollision(p2.x, nextP2Y, p2.size)) p2.y = nextP2Y;

            // --- ATTACK SYSTEM ---
            let dx = p1.x - p2.x;
            let dy = p1.y - p2.y;
            let distance = Math.sqrt(dx * dx + dy * dy);
            let touching = distance < p1.size + p2.size + 8; 

            if ((keys["/"] || keys["enter"]) && p2.attackCooldown === 0) {
                p2.attackCooldown = 25; 

                if (touching && p1.iFrames === 0) {
                    p1.health = Math.max(0, p1.health - 34); 
                    p1.iFrames = 45; 
                    
                    if (p1.health <= 0) {
                        gameOver = true;
                        document.getElementById("status").innerText = "DOWNED! KILLER WINS!";
                        document.getElementById("status").style.color = "#ff0000";
                    } else {
                        document.getElementById("status").innerText = "SKATER WAS HIT!";
                        document.getElementById("status").style.color = "#ffaa00";
                        setTimeout(() => {
                            if (!gameOver && !gameWon) {
                                document.getElementById("status").innerText = "SURVIVE THE MATCH!";
                                document.getElementById("status").style.color = "#0f0";
                            }
                        }, 800);
                    }
                }
            }

            document.getElementById("p1-stamina-fill").style.width = p1.stamina + "%";
            document.getElementById("p2-stamina-fill").style.width = p2.stamina + "%";
            document.getElementById("p1-health-fill").style.width = p1.health + "%";
        }

        function draw() {
            ctx.fillStyle = "#141414";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            obstacles.forEach(obs => {
                ctx.fillStyle = "#252525";
                ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
                ctx.strokeStyle = "#3a3a3a";
                ctx.lineWidth = 3;
                ctx.strokeRect(obs.x, obs.y, obs.w, obs.h);
            });

            // Skateboard
            ctx.fillStyle = "#ffaa00";
            ctx.fillRect(p1.x - 22, p1.y - 6, 44, 12);
            ctx.fillStyle = "#fff";
            ctx.fillRect(p1.x - 16, p1.y - 9, 6, 3);
            ctx.fillRect(p1.x - 16, p1.y + 6, 6, 3);
            ctx.fillRect(p1.x + 10, p1.y - 9, 6, 3);
            ctx.fillRect(p1.x + 10, p1.y + 6, 6, 3);

            // Skater
            if (p1.iFrames > 0 && Math.floor(p1.iFrames / 4) % 2 === 0) {
                ctx.fillStyle = "rgba(255, 0, 0, 0.6)";
            } else {
                ctx.fillStyle = p1.color;
            }
            ctx.beginPath();
            ctx.arc(p1.x, p1.y, p1.size, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = "#000";
            ctx.fillRect(p1.x + 1, p1.y - 5, 8, 3);
            ctx.fillRect(p1.x + 1, p1.y + 1, 8, 3);

            // Killer
            ctx.fillStyle = p2.color;
            ctx.beginPath();
            ctx.arc(p2.x, p2.y, p2.size, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = "#fff";
            ctx.fillRect(p2.x - 6, p2.y - 6, 4, 4);
            ctx.fillRect(p2.x - 6, p2.y + 2, 4, 4);

            // Attack Swing Ring
            if (p2.attackCooldown > 12) {
                ctx.strokeStyle = "rgba(255, 0, 0, 0.6)";
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.arc(p2.x, p2.y, p2.size + 15, 0, Math.PI * 2);
                ctx.stroke();
            }
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        initGame();
        gameLoop();
    </script>
</body>
</html>
