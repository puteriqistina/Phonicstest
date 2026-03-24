<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phonics City: Digraph Drive</title>
    <style>
        :root {
            --road-color: #444;
            --grass-color: #27ae60;
            --sky-color: #87CEEB;
        }

        body {
            margin: 0; padding: 0;
            font-family: 'Arial', sans-serif;
            background-color: var(--sky-color);
            text-align: center;
            overflow-x: hidden;
        }

        #header { background: white; padding: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }

        #game-world {
            position: relative; height: 300px;
            background-color: var(--grass-color);
            margin-top: 20px; border-top: 10px solid #2ecc71;
            overflow: hidden;
        }

        #road {
            position: absolute; bottom: 50px; width: 100%; height: 100px;
            background-color: var(--road-color);
            border-top: 4px dashed white; border-bottom: 4px dashed white;
        }

        #car {
            position: absolute; bottom: 60px; left: 50px;
            font-size: 60px; z-index: 10;
            transition: left 1.5s ease-in-out;
            transform: scaleX(-1); /* Pusing ke kanan */
            animation: driveVibration 0.2s infinite;
        }

        @keyframes driveVibration {
            0% { transform: scaleX(-1) translateY(0px); }
            50% { transform: scaleX(-1) translateY(-2px); }
            100% { transform: scaleX(-1) translateY(0px); }
        }

        .obstacle { position: absolute; bottom: 65px; font-size: 50px; transition: 0.5s; }

        #sentence-box {
            background: white; display: inline-block;
            padding: 20px 40px; border-radius: 50px;
            font-size: 28px; margin: 20px;
            border: 4px solid #3498db; min-width: 350px;
        }

        .highlight { color: #e74c3c; font-weight: bold; text-decoration: underline; }

        button {
            padding: 15px 40px; font-size: 18px; cursor: pointer;
            border-radius: 50px; border: none; background: #e67e22;
            color: white; box-shadow: 0 5px #d35400; font-weight: bold;
        }

        button:disabled { background: #95a5a6; box-shadow: none; cursor: not-allowed; }

        #feedback { font-size: 20px; margin-top: 15px; font-weight: bold; min-height: 30px; }
        .success { color: #27ae60; }
        .error { color: #e74c3c; }
    </style>
</head>
<body>

    <div id="header">
        <h1>🏙️ Phonics City Drive</h1>
        <p>Sebut ayat dengan betul untuk gerakkan kereta!</p>
        <h3>Mata: <span id="score">0</span></h3>
    </div>

    <div id="game-world">
        <div id="road"></div>
        <div id="car">🚗</div>
        <div id="obstacles"></div>
    </div>

    <div id="ui-layer">
        <div id="sentence-box">Sedia...</div>
        <div>
            <button id="mic-btn">🎤 KLIK & SEBUT</button>
            <button id="restart-btn" style="display:none; background:#3498db; box-shadow:0 5px #2980b9;">🔄 MAIN LAGI</button>
        </div>
        <div id="feedback"></div>
    </div>

    <script>
        const levels = [
            { digraph: "sh", sentence: "The ship is in the shop", words: ["ship", "shop"] },
            { digraph: "ch", sentence: "The chicken has a chin", words: ["chicken", "chin"] },
            { digraph: "th", sentence: "The moth is on the path", words: ["moth", "path"] },
            { digraph: "ph", sentence: "The dolphin is on the phone", words: ["dolphin", "phone"] }
        ];

        let currentLevel = 0;
        let score = 0;
        let isListening = false;

        const car = document.getElementById('car');
        const sentenceBox = document.getElementById('sentence-box');
        const feedback = document.getElementById('feedback');
        const scoreDisplay = document.getElementById('score');
        const micBtn = document.getElementById('mic-btn');
        const restartBtn = document.getElementById('restart-btn');
        const obstacleContainer = document.getElementById('obstacles');

        // Setup Speech Recognition
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (!SpeechRecognition) {
            alert("Browser anda tidak menyokong Speech Recognition. Sila gunakan Google Chrome.");
        }
        const recognition = new SpeechRecognition();
        recognition.lang = 'en-US';
        recognition.interimResults = false;

        function loadLevel() {
            if (currentLevel >= levels.length) {
                endGame();
                return;
            }
            const data = levels[currentLevel];
            let htmlText = data.sentence.replace(new RegExp(data.digraph, 'gi'), 
                (match) => `<span class="highlight">${match}</span>`);
            
            sentenceBox.innerHTML = htmlText;
            createObstacle();
            feedback.innerHTML = "Klik butang oren dan baca ayat di atas.";
            micBtn.disabled = false;
        }

        function createObstacle() {
            obstacleContainer.innerHTML = '';
            const cone = document.createElement('div');
            cone.className = 'obstacle';
            cone.innerHTML = '🚧';
            let carPos = parseInt(car.style.left || 50);
            cone.style.left = (carPos + 250) + 'px';
            obstacleContainer.appendChild(cone);
        }

        micBtn.onclick = () => {
            if (!isListening) {
                recognition.start();
                isListening = true;
                micBtn.innerHTML = "🛑 BERHENTI...";
                feedback.innerHTML = "<span style='color:blue'>Mendengar... Sila sebut sekarang.</span>";
            } else {
                recognition.stop();
                isListening = false;
                micBtn.innerHTML = "🎤 KLIK & SEBUT";
            }
        };

        recognition.onresult = (event) => {
            isListening = false;
            micBtn.innerHTML = "🎤 KLIK & SEBUT";
            const transcript = event.results[0][0].transcript.toLowerCase();
            const data = levels[currentLevel];
            
            // Logik semakan: ayat penuh ATAU kata kunci digraph
            const saidKeywords = data.words.every(word => transcript.includes(word));

            if (transcript.includes(data.sentence.toLowerCase()) || saidKeywords) {
                feedback.innerHTML = `<span class='success'>Bagus! Saya dengar: "${transcript}"</span>`;
                winLevel();
            } else {
                feedback.innerHTML = `<span class='error'>Saya dengar: "${transcript}". Cuba lagi!</span>`;
            }
        };

        recognition.onerror = (event) => {
            isListening = false;
            micBtn.innerHTML = "🎤 KLIK & SEBUT";
            if(event.error === 'not-allowed') {
                feedback.innerHTML = "<span class='error'>Sila 'Allow' mikrofon di bahagian atas browser!</span>";
            } else {
                feedback.innerHTML = "<span class='error'>Ralat: " + event.error + "</span>";
            }
        };

        function winLevel() {
            micBtn.disabled = true;
            score += 10;
            scoreDisplay.innerText = score;
            
            // Gerakkan kereta
            let currentPos = parseInt(car.style.left || 50);
            car.style.left = (currentPos + 250) + "px";
            
            // Hilangkan kon
            if(obstacleContainer.firstChild) obstacleContainer.firstChild.style.opacity = "0";

            currentLevel++;
            
            // Auto-Next selepas 2 saat
            setTimeout(loadLevel, 2000);
        }

        function endGame() {
            sentenceBox.innerHTML = "🏆 TAHNIAH! ANDA TAMAT!";
            feedback.innerHTML = "Skor Akhir: " + score;
            micBtn.style.display = "none";
            restartBtn.style.display = "inline-block";
        }

        restartBtn.onclick = () => location.reload();

        loadLevel();
    </script>
</body>
</html>
