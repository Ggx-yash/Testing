// game.js

// Setup Canvas
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Game constants
const CANVAS_WIDTH = 800;
const CANVAS_HEIGHT = 600;
const TANK_SIZE = 40;

// Tank class
class Tank {
    constructor(x, y, color, speed, health) {
        this.x = x;
        this.y = y;
        this.color = color;
        this.speed = speed;
        this.health = health;
        this.angle = 0;
    }

    // Draw tank
    draw() {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.angle);
        ctx.fillStyle = this.color;
        ctx.fillRect(-TANK_SIZE / 2, -TANK_SIZE / 2, TANK_SIZE, TANK_SIZE);
        ctx.restore();
    }

    // Move the tank
    move(direction) {
        const radians = (Math.PI / 180) * this.angle;
        if (direction === 'forward') {
            this.x += Math.cos(radians) * this.speed;
            this.y += Math.sin(radians) * this.speed;
        } else if (direction === 'backward') {
            this.x -= Math.cos(radians) * this.speed;
            this.y -= Math.sin(radians) * this.speed;
        }
    }

    // Rotate the tank
    rotate(angleDelta) {
        this.angle += angleDelta;
    }

    // Check if tank is alive
    isAlive() {
        return this.health > 0;
    }

    // Take damage
    takeDamage(damage) {
        this.health -= damage;
    }
}

// Scrap Collection Class
class Scrap {
    constructor(x, y, value) {
        this.x = x;
        this.y = y;
        this.value = value;
    }

    // Draw the scrap
    draw() {
        ctx.fillStyle = 'gray';
        ctx.fillRect(this.x - 5, this.y - 5, 10, 10);
    }
}

// Enemy AI Tank
class EnemyTank extends Tank {
    constructor(x, y, color) {
        super(x, y, color, 2, 100); // Speed 2, Health 100
    }

    // Basic AI behavior: move towards the player
    moveTowards(player) {
        const angleToPlayer = Math.atan2(player.y - this.y, player.x - this.x);
        this.angle = angleToPlayer * (180 / Math.PI);
        this.move('forward');
    }
}

// Initialize player and enemy tanks
let playerTank = new Tank(100, 100, 'green', 4, 100);
let enemyTank = new EnemyTank(600, 400, 'red');
let scrap = new Scrap(400, 300, 50);

// Game loop
let lastTime = 0;
function gameLoop(timestamp) {
    const deltaTime = timestamp - lastTime;
    lastTime = timestamp;

    // Clear the canvas
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

    // Update player tank movement
    if (keys['ArrowUp']) playerTank.move('forward');
    if (keys['ArrowDown']) playerTank.move('backward');
    if (keys['ArrowLeft']) playerTank.rotate(-5);
    if (keys['ArrowRight']) playerTank.rotate(5);

    // Update enemy AI movement
    enemyTank.moveTowards(playerTank);

    // Draw player and enemy tanks
    playerTank.draw();
    enemyTank.draw();

    // Draw scrap
    scrap.draw();

    // Check if player collects scrap
    if (Math.abs(playerTank.x - scrap.x) < TANK_SIZE && Math.abs(playerTank.y - scrap.y) < TANK_SIZE) {
        playerTank.health += scrap.value;  // Increase player health by scrap value (simplified for now)
        console.log("Scrap Collected!");
    }

    // Request the next frame
    requestAnimationFrame(gameLoop);
}

// Keyboard input handling
let keys = {};
window.addEventListener('keydown', (e) => {
    keys[e.key] = true;
});
window.addEventListener('keyup', (e) => {
    keys[e.key] = false;
});

// Start game loop
requestAnimationFrame(gameLoop);
