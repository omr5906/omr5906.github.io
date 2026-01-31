<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مؤقت الكابتن يوسف</title>
    <style>
        :root {
            --primary: #fbbf24; /* أصفر ذهبي للأبطال */
            --secondary: #d97706;
            --accent: #34d399;
            --danger: #f87171;
            --bg: #0a0a0a;
            --card: #171717;
        }
        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg);
            color: #e5e5e5;
            display: flex;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .app-container {
            width: 100%;
            max-width: 450px;
            background: var(--card);
            border-radius: 25px;
            padding: 25px;
            border: 1px solid #262626;
            box-shadow: 0 20px 40px rgba(0,0,0,0.6);
        }
        .header { text-align: center; margin-bottom: 25px; }
        .header h1 { color: var(--primary); font-size: 1.8rem; margin: 0; }
        .header p { color: #737373; font-size: 0.9rem; }

        .section-title { font-size: 1rem; color: var(--primary); margin-bottom: 10px; display: block; border-right: 3px solid var(--primary); padding-right: 10px; }

        /* Round Styling */
        .round-card {
            background: #262626;
            padding: 12px;
            border-radius: 12px;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            gap: 8px;
            transition: 0.3s;
        }
        .round-card:hover { border-color: var(--primary); }
        
        input {
            background: #000;
            border: 1px solid #404040;
            color: white;
            padding: 8px;
            border-radius: 6px;
            outline: none;
        }
        .name-input { flex-grow: 1; font-weight: bold; }
        .time-input { width: 50px; text-align: center; color: var(--primary); }

        /* Timer Display */
        .timer-screen {
            text-align: center;
            padding: 40px 0;
        }
        .circle {
            width: 220px; height: 220px;
            border-radius: 50%;
            border: 6px solid #262626;
            margin: 0 auto 20px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: radial-gradient(circle, #171717 0%, #000 100%);
            border-top-color: var(--primary);
            animation: pulse 2s infinite;
        }
        @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(251, 191, 36, 0.2); } 70% { box-shadow: 0 0 0 15px rgba(251, 191, 36, 0); } 100% { box-shadow: 0 0 0 0 rgba(251, 191, 36, 0); } }
        
        .timer-val { font-size: 4rem; font-weight: 900; font-family: 'Courier New', Courier, monospace; }
        .current-task-name { font-size: 1.4rem; color: var(--primary); font-weight: bold; }

        /* Action Buttons */
        .btn {
            width: 100%;
            padding: 16px;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
        }
        .btn-start { background: var(--primary); color: #000; font-size: 1.1rem; }
        .btn-reset { background: #404040; color: #fff; margin-top: 15px; }
        .btn-add { background: transparent; border: 1px solid #404040; color: #a3a3a3; margin-top: 10px; padding: 8px; }

        .warmup-box { background: #1c1917; padding: 15px; border-radius: 12px; margin-bottom: 20px; display: flex; align-items: center; justify-content: space-between; }
    </style>
</head>
<body>

<div class="app-container">
    <div id="setup-ui">
        <div class="header">
            <h1>مؤقت الكابتن يوسف ⚡</h1>
            <p>جاهز لحرق الدهون وتحطيم الأرقام؟</p>
        </div>

        <div class="warmup-box">
            <span>🔥 تشغيل جولة إحماء؟</span>
            <div>
                <input type="number" id="warmup-time" class="time-input" value="2"> د
                <input type="checkbox" id="warmup-check" checked>
            </div>
        </div>

        <span class="section-title">خطة الجولات (8 جولات جري)</span>
        <div id="rounds-list"></div>

        <button class="btn btn-add" onclick="addRoundRow()">+ إضافة مرحلة</button>
        <button class="btn btn-start" onclick="startWorkout()" style="margin-top: 25px;">انطلق يا يوسف 🚀</button>
    </div>

    <div id="timer-ui" style="display: none;">
        <div class="header">
            <h2 id="round-count" style="color: #737373;">المرحلة 1</h2>
        </div>
        
        <div class="timer-screen">
            <div class="circle">
                <div id="current-name" class="current-task-name">استعد</div>
                <div id="time-display" class="timer-val">00:00</div>
            </div>
        </div>

        <button class="btn btn-reset" onclick="stopWorkout()">إيقاف وإعادة ضبط ⛔</button>
    </div>
</div>

<audio id="beep" src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg"></audio>

<script>
    let timerInterval = null;
    let workoutPlan = [];
    let currentIndex = 0;
    let timeLeft = 0;
    const beep = document.getElementById('beep');

    // الإعداد الافتراضي ليوسف (8 جولات)
    const defaultRounds = [
        {n: 'جري سريع', m: 1, s: 30}, {n: 'مشي خفيف', m: 1, s: 0},
        {n: 'جري سريع', m: 1, s: 30}, {n: 'مشي خفيف', m: 1, s: 0},
        {n: 'جري سريع', m: 1, s: 30}, {n: 'مشي خفيف', m: 1, s: 0},
        {n: 'جري سريع', m: 1, s: 30}, {n: 'مشي خفيف', m: 1, s: 0}
    ];

    defaultRounds.forEach(r => addRoundRow(r.n, r.m, r.s));

    function addRoundRow(name = 'جولة جديدة', m = 1, s = 0) {
        const div = document.createElement('div');
        div.className = 'round-card';
        div.innerHTML = `
            <input type="text" class="name-input" value="${name}">
            <input type="number" class="time-input min" value="${m}">د
            <input type="number" class="time-input sec" value="${s}">ث
        `;
        document.getElementById('rounds-list').appendChild(div);
    }

    function startWorkout() {
        workoutPlan = [];
        if(document.getElementById('warmup-check').checked) {
            const wMin = parseInt(document.getElementById('warmup-time').value) || 0;
            workoutPlan.push({ name: 'إحماء وتمدد', total: wMin * 60 });
        }
        document.querySelectorAll('.round-card').forEach(card => {
            const name = card.querySelector('.name-input').value;
            const m = parseInt(card.querySelector('.min').value) || 0;
            const s = parseInt(card.querySelector('.sec').value) || 0;
            workoutPlan.push({ name: name, total: (m * 60) + s });
        });

        document.getElementById('setup-ui').style.display = 'none';
        document.getElementById('timer-ui').style.display = 'block';
        currentIndex = 0;
        nextStep();
    }

    function nextStep() {
        if(currentIndex >= workoutPlan.length) {
            document.getElementById('current-name').innerText = "عاش يا بطل!";
            document.getElementById('time-display').innerText = "DONE";
            playTone(3);
            return;
        }
        timeLeft = workoutPlan[currentIndex].total;
        document.getElementById('current-name').innerText = workoutPlan[currentIndex].name;
        document.getElementById('round-count').innerText = `مرحلة ${currentIndex + 1} من ${workoutPlan.length}`;
        runTimer();
    }

    function runTimer() {
        updateDisplay();
        timerInterval = setInterval(() => {
            timeLeft--;
            updateDisplay();
            if(timeLeft <= 3 && timeLeft > 0) playTone(1);
            if(timeLeft <= 0) {
                clearInterval(timerInterval);
                playTone(2);
                currentIndex++;
                setTimeout(nextStep, 1500);
            }
        }, 1000);
    }

    function updateDisplay() {
        const m = Math.floor(timeLeft / 60);
        const s = timeLeft % 60;
        document.getElementById('time-display').innerText = `${m.toString().padStart(2,'0')}:${s.toString().padStart(2,'0')}`;
    }

    function playTone(count) {
        let i = 0;
        const intr = setInterval(() => {
            beep.currentTime = 0;
            beep.play().catch(e => {});
            i++;
            if(i >= count) clearInterval(intr);
        }, 500);
    }

    function stopWorkout() {
        clearInterval(timerInterval);
        document.getElementById('setup-ui').style.display = 'block';
        document.getElementById('timer-ui').style.display = 'none';
    }
</script>
</body>
</html>
