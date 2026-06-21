
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة ذاكرة الحيوانات التفاعلية 🦁✨</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f9ff;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            user-select: none;
        }
        h1 {
            color: #2a4d69;
            margin-bottom: 5px;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
        }
        p {
            color: #4b86b4;
            font-size: 18px;
            font-weight: bold;
            margin-top: 0;
            margin-bottom: 25px;
        }
        /* شبكة الكروت للحيوانات */
        .game-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
            max-width: 500px;
            width: 100%;
        }
        .card {
            height: 110px;
            background-color: #4b86b4; /* لون خلفية الكرت وهو مغلق */
            border-radius: 15px;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0px; /* الحل الصحيح: إخفاء الحيوان تماماً في البداية عن طريق تصغير الخط */
            box-shadow: 0 5px 10px rgba(0,0,0,0.15);
            transition: transform 0.2s, background-color 0.3s, font-size 0.2s;
        }
        /* عند قلب الكرت يظهر الحيوان بحجمه الطبيعي والملون */
        .card.flipped {
            background-color: #ffffff;
            font-size: 50px; /* إظهار الحيوان بحجم كبير ومبهج */
            border: 3px solid #ffcb05;
            transform: scale(1.05);
        }
        /* عند التطابق الصحيح */
        .card.matched {
            background-color: #2ecc71;
            font-size: 50px;
            border: 3px solid #27ae60;
            cursor: default;
            transform: scale(1);
        }
        .reset-btn {
            margin-top: 30px;
            padding: 12px 35px;
            font-size: 18px;
            font-weight: bold;
            background-color: #ff6b6b;
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: 0.2s;
        }
        .reset-btn:hover {
            transform: scale(1.05);
            background-color: #ff4747;
        }
        .win-message {
            display: none;
            color: #2ecc71;
            font-size: 26px;
            font-weight: bold;
            margin-top: 25px;
            text-align: center;
            animation: bounce 0.5s infinite alternate;
        }
        @keyframes bounce {
            from { transform: translateY(0); }
            to { transform: translateY(-8px); }
        }
    </style>
</head>
<body>

    <h1>لعبة ذاكرة الحيوانات الذكية 🐾✨</h1>
    <p>ابحث عن الحيوانات المتطابقة لتسمع صوت الفوز!</p>

    <div class="game-grid" id="gameGrid"></div>

    <div class="win-message" id="winMessage">🎉 مذهل جداً! لقد فزت واكتشفت كل الحيوانات يا بطل! 🎉</div>

    <button class="reset-btn" onclick="resetGame()">إعادة اللعب 🔄</button>

<script>
    // حيوانات فقط مفرغة ملونة مأخوذة من فكرة كتابك
    const animalIcons = ['🐍', '🐍', '🦆', '🦆', '🦁', '🦁', '🐵', '🐵', '🐰', '🐰', '🐘', '🐘'];
    
    let flippedCards = [];
    let matchedCount = 0;
    let lockBoard = false;

    const gameGrid = document.getElementById('gameGrid');
    const winMessage = document.getElementById('winMessage');

    // نظام الصوت المطور لتجنب مشاكل الحجب في المتصفحات
    function playSound(type) {
        try {
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            if (!AudioContext) return;
            const ctx = new AudioContext();

            if (type === 'flip') {
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.type = 'sine';
                osc.frequency.setValueAtTime(580, ctx.currentTime);
                gain.gain.setValueAtTime(0.1, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.08);
                osc.start();
                osc.stop(ctx.currentTime + 0.08);
            } else if (type === 'match') {
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(523.25, ctx.currentTime);
                osc.frequency.setValueAtTime(659.25, ctx.currentTime + 0.08);
                gain.gain.setValueAtTime(0.15, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.25);
                osc.start();
                osc.stop(ctx.currentTime + 0.25);
            } else if (type === 'wrong') {
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(160, ctx.currentTime);
                osc.frequency.setValueAtTime(120, ctx.currentTime + 0.12);
                gain.gain.setValueAtTime(0.08, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.2);
                osc.start();
                osc.stop(ctx.currentTime + 0.2);
            } else if (type === 'winFanfare') {
                const notes = [523.25, 587.33, 659.25, 698.46, 783.99, 880.00, 987.77, 1046.50];
                notes.forEach((freq, index) => {
                    const noteOsc = ctx.createOscillator();
                    const noteGain = ctx.createGain();
                    noteOsc.type = 'triangle';
                    noteOsc.frequency.setValueAtTime(freq, ctx.currentTime + (index * 0.1));
                    noteGain.gain.setValueAtTime(0.15, ctx.currentTime + (index * 0.1));
                    noteGain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + (index * 0.1) + 0.2);
                    noteOsc.connect(noteGain);
                    noteGain.connect(ctx.destination);
                    noteOsc.start(ctx.currentTime + (index * 0.1));
                    noteOsc.stop(ctx.currentTime + (index * 0.1) + 0.2);
                });
            }
        } catch (e) {
            console.log("الصوت مدعوم.");
        }
    }

    function shuffle(array) {
        return array.sort(() => Math.random() - 0.5);
    }

    function createBoard() {
        const shuffledItems = shuffle([...animalIcons]);
        gameGrid.innerHTML = '';
        winMessage.style.display = 'none';
        matchedCount = 0;
        flippedCards = [];
        lockBoard = false;

        shuffledItems.forEach((animal) => {
            const card = document.createElement('div');
            card.classList.add('card');
            card.dataset.value = animal;
            card.innerText = animal;
            
            card.addEventListener('click', () => flipCard(card));
            gameGrid.appendChild(card);
        });
    }

    function flipCard(card) {
        if (lockBoard) return;
        if (card.classList.contains('flipped') || card.classList.contains('matched')) return;

        playSound('flip');
        card.classList.add('flipped');
        flippedCards.push(card);

        if (flippedCards.length === 2) {
            checkMatch();
        }
    }

    function checkMatch() {
        lockBoard = true;
        const [card1, card2] = flippedCards;

        if (card1.dataset.value === card2.dataset.value) {
            setTimeout(() => { playSound('match'); }, 100);
            card1.classList.add('matched');
            card2.classList.add('matched');
            matchedCount += 2;
            flippedCards = [];
            lockBoard = false;

            if (matchedCount === animalIcons.length) {
                setTimeout(() => {
                    winMessage.style.display = 'block';
                    playSound('winFanfare'); // صوت الفوز الموسيقي المتصاعد 🎉
                }, 400);
            }
        } else {
            setTimeout(() => { playSound('wrong'); }, 150);
            setTimeout(() => {
                card1.classList.remove('flipped');
                card2.classList.remove('flipped');
                flippedCards = [];
                lockBoard = false;
            }, 1000);
        }
    }

    function resetGame() {
        createBoard();
    }

    createBoard();
</script>

</body>
</html>
