<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            background-color: #87CEEB; /* Ciel bleu */
        }
        #moto {
            position: absolute;
            width: 50px;
            height: 30px;
            background-color: #333;
            bottom: 0;
            left: 375px;
        }
        #obstacle {
            position: absolute;
            width: 30px;
            height: 30px;
            background-color: #FF0000; /* Rouge */
            top: 0;
            left: 375px;
        }
        #parametres {
            display: none;
        }
    </style>
</head>
<body>
    <div id="moto"></div>
    <div id="obstacle"></div>
    <span id="score">Score: 0</span>
    <span id="niveau">Niveau: 1</span>
    <span id="time">Temps restant: 60</span> <!-- Ajout du temps restant -->
    <button onclick="startGame()">Lancer le jeu</button>
    <button onclick="showParametres()">Paramètres</button>
    <div id="parametres">
        <h2>Paramètres</h2>
        <label for="vitesse">Vitesse du jeu :</label>
        <input type="number" id="vitesse" name="vitesse" min="1" max="10" value="5">
        <br>
        <label for="luminosite">Luminosité :</label>
        <input type="range" id="luminosite" name="luminosite" min="0" max="100" value="100" oninput="updateLuminosite()">
        <br>
        <label for="volume">Volume :</label>
        <input type="range" id="volume" name="volume" min="0" max="100" value="100" oninput="updateVolume()">
        <br>
        <button onclick="pauseGame()">Pause</button>
        <button onclick="resumeGame()">Reprendre</button>
        <button onclick="restartGame()">Recommencer</button>
        <button onclick="hideParametres()">Fermer</button>
    </div>

    <audio id="moveSound" src="move.mp3"></audio>
    <audio id="collisionSound" src="collision.mp3"></audio>
    <audio id="gameOverSound" src="gameover.mp3"></audio>

    <script>
        let gameInterval;
        let timerInterval;
        let currentLevel = 1;

        const levels = [
            { speed: 5, obstacleFrequency: 2000 }, // Niveau 1
            { speed: 7, obstacleFrequency: 1500 }, // Niveau 2
            { speed: 9, obstacleFrequency: 1000 }  // Niveau 3
        ];

        const moto = document.getElementById("moto");
        const obstacle = document.getElementById("obstacle");
        const scoreDisplay = document.getElementById("score");
        const niveauDisplay = document.getElementById("niveau");
        const timeDisplay = document.getElementById("time"); // Ajout du temps restant
        const moveSound = document.getElementById("moveSound");
        const collisionSound = document.getElementById("collisionSound");
        const gameOverSound = document.getElementById("gameOverSound");
        let score = 0;
        let vitesseJeu = 5;
        let gameStarted = false;
        let remainingTime = 60; // Temps initial en secondes
        let pausedTime = 0;

        niveauDisplay.textContent = "Niveau: " + currentLevel;

        function updateGameArea() {
            let obstacleTop = parseInt(obstacle.style.top);
            obstacle.style.top = (obstacleTop + levels[currentLevel - 1].speed) + "px";

            if (obstacleTop > window.innerHeight) {
                obstacle.style.top = "0px";
                obstacle.style.left = Math.random() * (window.innerWidth - 30) + "px";
            }

            if (
                obstacleTop >= window.innerHeight - 30 &&
                parseInt(obstacle.style.left) >= parseInt(moto.style.left) &&
                parseInt(obstacle.style.left) <= (parseInt(moto.style.left) + 50)
            ) {
                collisionSound.play();
                alert("Game Over. Score: " + score);
                endGame();
            }

            score++;
            scoreDisplay.textContent = "Score: " + score;
        }

        function handleKeydown(event) {
            if (!gameStarted) return;

            if (event.keyCode === 37 && parseInt(moto.style.left) > 0) {
                moto.style.left = (parseInt(moto.style.left) - 10) + "px";
                score--;
                moveSound.play();
            } else if (event.keyCode === 39 && parseInt(moto.style.left) < (window.innerWidth - 50)) {
                moto.style.left = (parseInt(moto.style.left) + 10) + "px";
                score++;
                moveSound.play();
            }
        }

        window.addEventListener("keydown", handleKeydown);

        function startGame() {
            if (gameStarted) return;
            gameStarted = true;
            score = 0;
            scoreDisplay.textContent = "Score: " + score;
            moto.style.left = "375px";
            obstacle.style.top = "0px";
            obstacle.style.left = Math.random() * (window.innerWidth - 30) + "px";

            gameInterval = setInterval(updateGameArea, 20);
            startTimer(remainingTime);
        }

        function showParametres() {
            document.getElementById("parametres").style.display = "block";
        }

        function hideParametres() {
            document.getElementById("parametres").style.display = "none";
            vitesseJeu = parseInt(document.getElementById("vitesse").value);
        }

        function updateLuminosite() {
            const luminosite = document.getElementById("luminosite").value;
            document.body.style.backgroundColor = `hsl(207, 50%, ${luminosite}%)`;
        }

        function updateVolume() {
            const volume = document.getElementById("volume").value / 100;
            moveSound.volume = volume;
            collisionSound.volume = volume;
            gameOverSound.volume = volume;
        }

        function startTimer(duration) {
            let timer = duration;
            timeDisplay.textContent = "Temps restant: " + timer;
            timerInterval = setInterval(function () {
                timer--;
                if (timer < 0) {
                    clearInterval(timerInterval);
                    alert("Temps écoulé. Score: " + score);
                    endGame();
                } else {
                    timeDisplay.textContent = "Temps restant: " + timer;
                }
            }, 1000);
        }

        function pauseGame() {
            clearInterval(gameInterval);
            clearInterval(timerInterval);
            pausedTime = parseInt(timeDisplay.textContent.split(": ")[1]);
        }

        function resumeGame() {
            if (!gameStarted) return;
            gameInterval = setInterval(updateGameArea, 20);
            startTimer(pausedTime);
        }

        function restartGame() {
            clearInterval(gameInterval);
            clearInterval(timerInterval);
            gameStarted = false;
            remainingTime = 60; // Reset time
            startGame();
        }

        function endGame() {
            clearInterval(gameInterval);
            clearInterval(timerInterval);
            gameStarted = false;
            gameOverSound.play();
        }
    </script>
    <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7211339036856121"
     crossorigin="anonymous"></script>
</body>
</html>

