<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>لعبة السيارات</title>
    <style>
        * { box-sizing: border-box; }
        body {
            margin: 0;
            font-family: 'Segoe UI', sans-serif;
            background: #111;
            color: white;
            text-align: center;
            overflow: hidden;
        }

        /* شاشة البداية */
        #startScreen {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 10;
            padding: 40px;
            background: rgba(0,0,0,0.9);
            border-radius: 15px;
        }

        #startBtn {
            padding: 15px 30px;
            font-size: 20px;
            background: gold;
            border: none;
            border-radius: 10px;
            cursor: pointer;
        }

        /* شاشة اختيار السيارة */
        #carSelection {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 10;
            padding: 20px;
            background: rgba(0,0,0,0.9);
            border-radius: 15px;
        }

        .car-option {
            display: inline-block;
            margin: 15px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .car-option img {
            width: 100px;
            border-radius: 10px;
            border: 3px solid transparent;
            transition: all 0.3s ease;
        }

        .car-option.selected img {
            outline: 4px solid yellow;
        }

        #confirmCar {
            margin-top: 15px;
            padding: 10px 20px;
            font-size: 16px;
            background-color: gold;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }

        /* Canvas اللعبة */
        #gameCanvas {
            display: none;
            position: absolute;
            top: 0;
            left: 0;
        }

        /* HUD (النقاط والوقت) */
        #hud {
            display: none;
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 5;
            background: rgba(0,0,0,0.6);
            padding: 10px 20px;
            border-radius: 10px;
        }
    </style>
</head>
<body>

<!-- شاشة البداية -->
<div id="startScreen">
    <h1>لعبة السيارات</h1>
    <p>اختر سيارتك وابدأ المغامرة!</p>
    <button id="startBtn">ابدأ اللعبة</button>
</div>

<!-- شاشة اختيار السيارة -->
<div id="carSelection">
    <h2>اختر سيارتك</h2>
    
    <div class="car-option" data-car="red">
        <img src="car_red.png" alt="Red Car">
        <p>السيارة الحمراء (سرعة عادية)</p>
    </div>
    <div class="car-option" data-car="blue">
        <img src="car_blue.png" alt="Blue Car">
        <p>السيارة الزرقاء (سرعة أعلى)</p>
    </div>
    <div class="car-option" data-car="green">
        <img src="car_green.png" alt="Green Car">
        <p>السيارة الخضراء (تحكم أفضل)</p>
    </div>

    <button id="confirmCar">تأكيد الاختيار</button>
</div>

<!-- HUD -->
<div id="hud">
    <span id="timer">الوقت: 0 ثانية</span> |
    <span id="score">النقاط: 0</span>
</div>

<!-- Canvas اللعبة -->
<canvas id="gameCanvas" width="800" height="600"></canvas>

<script>
    const startScreen = document.getElementById('startScreen');
    const startBtn = document.getElementById('startBtn');
    const carSelection = document.getElementById('carSelection');
    const gameCanvas = document.getElementById('gameCanvas');
    const ctx = gameCanvas.getContext('2d');
    const hud = document.getElementById('hud');

    const timerDisplay = document.getElementById('timer');
    const scoreDisplay = document.getElementById('score');

    let selectedCar = null;
    let carImage = new Image();
    let carX = gameCanvas.width / 2 - 25;
    let carY = gameCanvas.height - 100;
    let carSpeed = 5;
    let speedMultiplier = 1;

    let keys = {};
    let startTime, timerInterval, score = 0;

    // بيانات السيارات
    const cars = {
        red: { speed: 5, handling: 1, image: 'car_red.png' },
        blue: { speed: 8, handling: 0.7, image: 'car_blue.png' },
        green: { speed: 4, handling: 1.5, image: 'car_green.png' }
    };

    // أحداث الأزرار
    document.addEventListener('keydown', e => keys[e.key] = true);
    document.addEventListener('keyup', e => keys[e.key] = false);

    // أحداث اختيار السيارة
    document.querySelectorAll('.car-option').forEach(option => {
        option.addEventListener('click', () => {
            document.querySelectorAll('.car-option').forEach(opt => opt.classList.remove('selected'));
            option.classList.add('selected');
            selectedCar = option.getAttribute('data-car');
            carImage.src = cars[selectedCar].image;
            carSpeed = cars[selectedCar].speed;
        });
    });

    // بدء اللعبة
    startBtn.onclick = () => {
        startScreen.style.display = 'none';
        carSelection.style.display = 'block';
    };

    document.getElementById('confirmCar').onclick = () => {
        if (!selectedCar) return alert("اختر سيارة أولاً");
        carSelection.style.display = 'none';
        gameCanvas.style.display = 'block';
        hud.style.display = 'block';

        carX = gameCanvas.width / 2 - 25;
        carY = gameCanvas.height - 100;

        startTime = Date.now();
        timerInterval = setInterval(() => {
            let elapsed = Math.floor((Date.now() - startTime) / 1000);
            timerDisplay.textContent = `الوقت: ${elapsed} ثانية`;
            score = elapsed;
            scoreDisplay.textContent = `النقاط: ${score}`;
        }, 1000);

        requestAnimationFrame(gameLoop);
    };

    function gameLoop() {
        ctx.clearRect(0, 0, gameCanvas.width, gameCanvas.height);

        // تسارع عند الضغط الطويل
        if (keys['ArrowLeft']) carX -= carSpeed * speedMultiplier;
        if (keys['ArrowRight']) carX += carSpeed * speedMultiplier;
        if (keys['ArrowUp']) carY -= carSpeed * speedMultiplier;
        if (keys['ArrowDown']) carY += carSpeed * speedMultiplier;

        // زيادة السرعة أثناء الحركة
        if (keys['ArrowLeft'] || keys['ArrowRight'] || keys['ArrowUp'] || keys['ArrowDown']) {
            speedMultiplier = Math.min(speedMultiplier + 0.001, 2);
        } else {
            speedMultiplier = 1;
        }

        // حدود Canvas
        carX = Math.max(0, Math.min(gameCanvas.width - 50, carX));
        carY = Math.max(0, Math.min(gameCanvas.height - 70, carY));

        // رسم السيارة
        ctx.drawImage(carImage, carX, carY, 50, 70);

        requestAnimationFrame(gameLoop);
    }
</script>

</body>
</html># Car-game
