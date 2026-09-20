<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ben Racing</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    background: #111;
    font-family: Arial, sans-serif;
    text-align: center;
    color: white;
}

h1 {
    margin: 15px 0 5px;
}

#score {
    font-size: 20px;
    margin-bottom: 10px;
}

#game {
    position: relative;
    width: 360px;
    height: 600px;
    max-width: 95vw;
    margin: auto;
    overflow: hidden;
    background: #444;
    border-left: 8px solid #222;
    border-right: 8px solid #222;
}

/* Route */
#road {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #444;
}

/* Lignes de la route */
.line {
    position: absolute;
    width: 8px;
    height: 80px;
    background: white;
    left: 50%;
    transform: translateX(-50%);
}

/* Voiture du joueur */
#car {
    position: absolute;
    width: 55px;
    height: 90px;
    background: red;
    border-radius: 12px;
    bottom: 30px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 5;
}

/* Pare-brise */
#car::before {
    content: "";
    position: absolute;
    width: 35px;
    height: 25px;
    background: #87ceeb;
    top: 12px;
    left: 10px;
    border-radius: 5px;
}

/* Roues */
#car::after {
    content: "⚫   ⚫";
    position: absolute;
    font-size: 20px;
    left: 4px;
    bottom: 8px;
}

/* Obstacles */
.enemy {
    position: absolute;
    width: 55px;
    height: 90px;
    background: blue;
    border-radius: 12px;
    z-index: 4;
}

/* Boutons */
.controls {
    margin-top: 15px;
}

button {
    font-size: 25px;
    padding: 12px 25px;
    margin: 5px;
    border: none;
    border-radius: 10px;
    cursor: pointer;
}

#start {
    background: limegreen;
    color: white;
    font-size: 18px;
}

#left, #right {
    background: #333;
    color: white;
}

#gameOver {
    display: none;
    position: absolute;
    top: 40%;
    width: 100%;
    background: rgba(0,0,0,0.85);
    padding: 25px;
    z-index: 10;
}

#gameOver h2 {
    color: red;
}
</style>
</head>

<body>

<h1>🏎️ BEN RACING</h1>

<div id="score">Score : 0</div>

<div id="game">

    <div id="road"></div>

    <div id="car"></div>

    <div id="gameOver">
        <h2>💥 GAME OVER</h2>
        <p id="finalScore"></p>
        <button onclick="restart()">Rejouer</button>
    </div>

</div>

<button id="start" onclick="startGame()">▶ COMMENCER</button>

<div class="controls">
    <button id="left">⬅️</button>
    <button id="right">➡️</button>
</div>

<script>

const game = document.getElementById("game");
const car = document.getElementById("car");
const scoreText = document.getElementById("score");

let carX = 152;
let score = 0;
let speed = 5;
let playing = false;

let enemies = [];
let lines = [];

/* Création des lignes */
for(let i = 0; i < 6; i++) {

    let line = document.createElement("div");

    line.className = "line";

    line.style.top = (i * 120) + "px";

    game.appendChild(line);

    lines.push(line);
}

/* Déplacement de la voiture */
function moveLeft() {

    if(!playing) return;

    carX -= 25;

    if(carX < 10) {
        carX = 10;
    }

    car.style.left = carX + "px";
    car.style.transform = "none";
}

function moveRight() {

    if(!playing) return;

    carX += 25;

    if(carX > 295) {
        carX = 295;
    }

    car.style.left = carX + "px";
    car.style.transform = "none";
}

/* Clavier */
document.addEventListener("keydown", function(event) {

    if(event.key === "ArrowLeft") {
        moveLeft();
    }

    if(event.key === "ArrowRight") {
        moveRight();
    }

});

/* Boutons téléphone */
document.getElementById("left")
.addEventListener("touchstart", moveLeft);

document.getElementById("right")
.addEventListener("touchstart", moveRight);

document.getElementById("left")
.addEventListener("click", moveLeft);

document.getElementById("right")
.addEventListener("click", moveRight);


/* Démarrage */
function startGame() {

    if(playing) return;

    playing = true;

    score = 0;

    speed = 5;

    scoreText.innerHTML = "Score : 0";

    document.getElementById("gameOver").style.display = "none";

    createEnemy();

    gameLoop();
}


/* Créer une voiture ennemie */
function createEnemy() {

    if(!playing) return;

    let enemy = document.createElement("div");

    enemy.className = "enemy";

    let position = Math.floor(Math.random() * 6);

    enemy.style.left = (20 + position * 55) + "px";

    enemy.style.top = "-100px";

    game.appendChild(enemy);

    enemies.push(enemy);

    setTimeout(createEnemy, 1200);
}


/* Vérifier collision */
function collision(a, b) {

    let r1 = a.getBoundingClientRect();
    let r2 = b.getBoundingClientRect();

    return !(
        r1.bottom < r2.top ||
        r1.top > r2.bottom ||
        r1.right < r2.left ||
        r1.left > r2.right
    );
}


/* Boucle du jeu */
function gameLoop() {

    if(!playing) return;

    /* Animation de la route */

    lines.forEach(line => {

        let y = parseInt(line.style.top);

        y += speed;

        if(y > 600) {
            y = -80;
        }

        line.style.top = y + "px";

    });


    /* Déplacement des ennemis */

    enemies.forEach((enemy, index) => {

        let y = parseInt(enemy.style.top);

        y += speed;

        enemy.style.top = y + "px";


        /* Collision */

        if(collision(car, enemy)) {

            endGame();

        }


        /* Ennemi sorti de l'écran */

        if(y > 650) {

            enemy.remove();

            enemies.splice(index, 1);

            score++;

            scoreText.innerHTML = "Score : " + score;

            /* Augmenter progressivement la difficulté */

            if(score % 10 === 0) {
                speed += 1;
            }
        }

    });


    requestAnimationFrame(gameLoop);
}


/* Fin du jeu */
function endGame() {

    playing = false;

    document.getElementById("gameOver").style.display = "block";

    document.getElementById("finalScore").innerHTML =
        "Ton score : " + score;
}


/* Recommencer */
function restart() {

    enemies.forEach(enemy => enemy.remove());

    enemies = [];

    carX = 152;

    car.style.left = carX + "px";

    car.style.transform = "translateX(-50%)";

    score = 0;

    speed = 5;

    document.getElementById("gameOver").style.display = "none";

    startGame();
}

</script>

</body>
</html># jeu-de-voiture
