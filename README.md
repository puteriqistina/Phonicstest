# Phonicstest

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phonics City: Digraph Drive</title>
    <style>
        :root {
            --road-color: #444;
            --grass-color: #27ae60;
            --sky-color: #87CEEB;
            --accent-blue: #3498db;
        }

        body {
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--sky-color);
            text-align: center;
            overflow-x: hidden;
        }

        #header {
            background: white;
            padding: 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }

        /* THE GAME WORLD */
        #game-world {
            position: relative;
            height: 350px;
            background-color: var(--grass-color);
            margin-top: 30px;
            border-top: 12px solid #2ecc71;
            overflow: hidden;
            display: flex;
            align-items: center;
        }

        #road {
            position: absolute;
            bottom: 60px;
            width: 100%;
            height: 120px;
            background-color: var(--road-color);
            border-top: 5px dashed rgba(255,255,255,0.5);
            border-bottom: 5px dashed rgba(255,255,255,0.5);
        }

        /* THE CAR - Facing Right + Vibration */
        #car {
            position: absolute;
            bottom: 75px;
            left: 50px;
            font-size: 70px;
            transition: left 1.2s cubic-bezier(0.45, 0.05, 0.55, 0.95);
            z-index: 10;
            transform: scaleX(-1); /* Flips emoji to face right */
            animation: driveVibration 0.15s infinite;
        }

        @keyframes driveVibration {
            0% { transform: scaleX(-1) translateY(0px); }
            50% { transform: scaleX(-1) translateY(-3px); }
            100% { transform: scaleX(-1) translateY(0px); }
        }

        .obstacle {
            position: absolute;
            bottom: 80px;
            font-size: 55px;
            transition: all 0.5s ease;
        }

        /* UI ELEMENTS */
        #ui-layer { padding: 30px; }

        #sentence-box {
            background: white;
            display: inline-block;
            padding: 25px 50px;
            border-radius: 60px;
            font-size: 36px;
            margin-bottom: 20px;
            border: 5px solid var(--accent-blue);
            min-width: 450px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }

        .highlight { 
            color: #e74c3c; 
            text-decoration: underline; 
            font-weight: 900; 
        }

        .btn-container { margin-top: 20px; }

        button {
            padding: 18px 45px;
            font-size: 22px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 50px;
            border: none;
            color: white;
            transition: 0.2s;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        #mic-btn {
            background: #e67e22;
            box-shadow: 0 6px #d35400;
        }

        #mic-btn:active {
            box-shadow: 0 2px #d35400;
            transform: translateY(4px);
        }

        #restart-btn {
            background: #3498db;
            box-shadow: 0 6px #2980b9;
            display: none;
        }

        #restart-btn:hover { background: #2980b9; }

        #feedback { 
            font-size: 24px; 
            margin-top: 20px; 
            font-weight: bold; 
            min-height: 40px; 
        }
        
        .success { color: #1e8449; }
        .error { color: #c0392b; }

    </style>
</head>
<body>

    <div id="header">
        <h1>🏙️ Phonics City: Digraph Drive</h1>
        <p>Listen and Speak: Pronounce the <b>sh, ch, th, ph</b> sentences!</p>
        <h2>Score: <span id="score">0</span></h2>
    </div>

    <div id="game-world">
        <div id="road"></div>
        <div id="car">🚗</div>
        <div id="obstacles"></div>
    </div>

    <div id="ui-layer">
        <div id="sentence-box">Starting Engine...</div>
        
        <div class="btn-container">
            <button id="mic-btn">🎤 Hold to Speak</button>
            <button id="restart-btn">🔄 Play Again</button>
        </div>
        
        <div id="feedback"></div>
    </div>

    <script>
        // Phonics Data Configuration
        const levels = [
            { digraph: "sh", sentence: "The ship is in the shop", words: ["ship", "shop"] },
            { digraph: "ch", sentence: "The chicken has a chin", words: ["chicken", "chin"] },
            { digraph: "th", sentence: "The moth is on the path", words: ["moth", "path"] },
            { digraph: "ph", sentence: "The dolphin is on the phone", words: ["dolphin", "phone"] }
        ];

        let currentLevel = 0;
        let score = 0;

        const car = document.getElementById('car');
        const sentenceBox = document.getElementById('sentence-box');
        const feedback = document.getElementById('feedback');
        const scoreDisplay = document.getElementById('score');
        const micBtn = document.getElementById('mic-btn');
        const restartBtn = document.getElementById('restart-btn');
        const obstacleContainer = document.getElementById('obstacles');

        // Speech Recognition Setup
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (!SpeechRecognition) {
            alert("Speech recognition not supported in this browser. Please use Google Chrome.");
        }
        const recognition = new SpeechRecognition();
        recognition.lang = 'en-US';

        function loadLevel() {
            if (currentLevel >= levels.length) {
                endGame();
                return;
            }
            
            const data = levels[currentLevel];
            // Visual feedback: underline the digraphs
            let displayHeader = data.sentence.replace(new RegExp(data.digraph, 'gi'), 
                (match) => `<span class="highlight">${match}</span>`);
            
            sentenceBox.innerHTML = displayHeader;
            createObstacle();
        }

        function createObstacle() {
            obstacleContainer.innerHTML = '';
            const cone = document.createElement('div');
            cone.className = 'obstacle';
            cone.innerHTML = '🚧';
            
            // Positioning obstacle 300px ahead of current car position
            const carPos = parseInt(car.style.left) || 50;
            cone.style.left = (carPos + 300) + 'px';
            obstacleContainer.appendChild(cone);
        }

        // Mouse/Touch Events
        micBtn.onmousedown = () => {
            recognition.start();
            feedback.innerHTML = "Listening...";
            feedback.className = "";
        };

        micBtn.onmouseup = () => {
            recognition.stop();
        };

        recognition.onresult = (event) => {
            const transcript = event.results[0][0].transcript.toLowerCase();
            const data = levels[currentLevel];
            
            // Logic: Check if the transcript includes the sentence OR the key digraph words
            const saidKeywords = data.words.every(word => transcript.includes(word));

            if (transcript.includes(data.sentence.toLowerCase()) || saidKeywords) {
                feedback.innerHTML = "<span class='success'>Excellent! Driving forward... 🚗💨</span>";
                winLevel();
            } else {
                feedback.innerHTML = `<span class='error'>I heard: "${transcript}". Try again!</span>`;
            }
        };

        function winLevel() {
            score += 10;
            scoreDisplay.innerText = score;
            
            // Car Drive Animation
            let currentPos = parseInt(car.style.left) || 50;
            car.style.left = (currentPos + 300) + "px";
            
            // Remove barrier
            if(obstacleContainer.firstChild) {
                obstacleContainer.firstChild.style.opacity = "0";
                obstacleContainer.firstChild.style.transform = "translateY(-50px)";
            }

            currentLevel++;
            setTimeout(loadLevel, 1600);
        }

        function endGame() {
            sentenceBox.innerHTML = "🏆 YOU ARE A PHONICS PRO!";
            feedback.innerHTML = "Final Score: " + score;
            micBtn.style.display = "none";
            restartBtn.style.display = "inline-block";
        }

        restartBtn.onclick = () => {
            currentLevel = 0;
            score = 0;
            scoreDisplay.innerText = "0";
            car.style.left = "50px";
            micBtn.style.display = "inline-block";
            restartBtn.style.display = "none";
            feedback.innerText = "";
            loadLevel();
        };

        // Initialize Game
        loadLevel();
    </script>
</body>
</html>
