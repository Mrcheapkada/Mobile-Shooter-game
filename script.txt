const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
canvas.width = 800;
canvas.height = 500;

let bombs = [];
let bullets = [];
let explosions = [];
let gunX = canvas.width / 2;
let score = 0;
let speed = 2;
let bombsDestroyed = 0;
let highScore = localStorage.getItem("highScore") || 0;
let wrongAttempts = 0;
const maxWrongAttempts = 10;
let gameOver = false;
const mobileInput = document.getElementById("mobileInput");

// Load sounds
const shootSound = new Audio("shoot.mp3");
const explosionSound = new Audio("explosion.mp3");

// Open keyboard on mobile when tapped
document.addEventListener("click", () => mobileInput.focus());
mobileInput.addEventListener("input", function(event) {
    let keyPressed = event.target.value.toUpperCase();
    event.target.value = "";
    shootBullet(keyPressed);
});

// Function to create popup
function showGameOverPopup() {
    if (score > highScore) {
        highScore = score;
        localStorage.setItem("highScore", highScore);
    }

    let popup = document.createElement("div");
    popup.style = "position:fixed; top:50%; left:50%; transform:translate(-50%, -50%); background:#333; color:white; padding:20px; border-radius:10px; text-align:center; box-shadow:0 0 10px rgba(0,0,0,0.5);";
    popup.innerHTML = `
        <h2>Game Over!</h2>
        <p>Your Score: <b>${score}</b></p>
        <p>High Score: <b>${highScore}</b></p>
        <button id="restartBtn" style="padding:10px 20px; font-size:16px; background:red; color:white; border:none; cursor:pointer; border-radius:5px;">Restart Game</button>
    `;
    document.body.appendChild(popup);
    document.getElementById("restartBtn").addEventListener("click", () => location.reload());
}

// Generate random alphabet
function getRandomLetter() {
    return "ABCDEFGHIJKLMNOPQRSTUVWXYZ"[Math.floor(Math.random() * 26)];
}

// Create bomb
function createBomb() {
    if (gameOver) return;
    let bomb = {
        x: Math.random() * (canvas.width - 30),
        y: 0,
        speed: speed,
        letter: getRandomLetter(),
        powerUp: Math.random() < 0.1 ? true : false // 10% chance for slow-motion power-up
    };
    bombs.push(bomb);
}

// Move bombs down
function updateBombs() {
    for (let i = 0; i < bombs.length; i++) {
        bombs[i].y += bombs[i].powerUp ? bombs[i].speed / 2 : bombs[i].speed;
        if (bombs[i].y > canvas.height - 20) {
            gameOver = true;
            showGameOverPopup();
            return;
        }
    }
}

// Move bullets
function updateBullets() {
    for (let i = 0; i < bullets.length; i++) {
        let bullet = bullets[i];

        if (bullet.target) {
            let dx = bullet.target.x - bullet.x;
            let dy = bullet.target.y - bullet.y;
            let distance = Math.sqrt(dx * dx + dy * dy);
            let bulletSpeed = 7;
            bullet.x += (dx / distance) * bulletSpeed;
            bullet.y += (dy / distance) * bulletSpeed;

            if (distance < 10) {
                explosionSound.play();
                explosions.push({ x: bullet.target.x, y: bullet.target.y, radius: 10, opacity: 1 });
                bombs = bombs.filter(b => b !== bullet.target);
                bullets.splice(i, 1);
                score += 10;
                bombsDestroyed++;

                if (bombsDestroyed % 10 === 0) speed += 0.5;
            }
        } else {
            bullet.x += bullet.dx;
            bullet.y -= 5;
            if (bullet.y < 0 || bullet.x < 0 || bullet.x > canvas.width) bullets.splice(i, 1);
        }
    }
}

// Shooting logic
function shootBullet(key) {
    if (gameOver) return;
    let targetBomb = bombs.find(bomb => bomb.letter === key);
    shootSound.play();

    if (targetBomb) {
        bullets.push({ x: gunX, y: canvas.height - 30, target: targetBomb });
    } else {
        bullets.push({ x: gunX, y: canvas.height - 30, dx: Math.random() * 6 - 3, target: null });
        wrongAttempts++;
        if (wrongAttempts >= maxWrongAttempts) {
            gameOver = true;
            showGameOverPopup();
        }
    }
}

// Draw everything
function drawGame() {
    if (gameOver) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw gun (Triangle)
    ctx.fillStyle = "grey";
    ctx.beginPath();
    ctx.moveTo(gunX, canvas.height - 20);
    ctx.lineTo(gunX - 20, canvas.height);
    ctx.lineTo(gunX + 20, canvas.height);
    ctx.fill();

    // Draw bombs
    bombs.forEach(bomb => {
        ctx.fillStyle = bomb.powerUp ? "blue" : "red";
        ctx.beginPath();
        ctx.arc(bomb.x, bomb.y, 20, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = "white";
        ctx.font = "20px Arial";
        ctx.textAlign = "center";
        ctx.fillText(bomb.letter, bomb.x, bomb.y + 5);
    });

    // Draw bullets
    ctx.fillStyle = "yellow";
    bullets.forEach(bullet => {
        ctx.fillRect(bullet.x - 2, bullet.y, 4, 10);
    });

    // Draw score
    ctx.fillStyle = "white";
    ctx.font = "24px Arial";
    ctx.textAlign = "center";
    ctx.fillText("Score: " + score, canvas.width / 2, 30);
    ctx.fillText("High Score: " + highScore, canvas.width - 80, 30);
    ctx.fillText("Wrong Attempts: " + wrongAttempts + "/" + maxWrongAttempts, 100, 30);
}

// Game loop
function gameLoop() {
    updateBombs();
    updateBullets();
    drawGame();
    if (!gameOver) requestAnimationFrame(gameLoop);
}

setInterval(createBomb, 1000);
gameLoop();
