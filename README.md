<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>85-kg-Challenge Tracker</title>
<style>
    body{font-family:Arial,Helvetica,sans-serif;margin:0;padding:20px;background:#f5f5f5;color:#222;}
    h1,h2{margin-top:0;color:#0a4d68;}
    section{background:#fff;padding:20px;margin-bottom:20px;border-radius:8px;box-shadow:0 2px 5px rgba(0,0,0,0.1);}
    label, textarea{display:block;margin:6px 0;}
    input, textarea{padding:4px;}
    input[type="date"], input[type="datetime-local"], input[type="number"], input[type="text"]{width:200px;}
    textarea{width:100%;height:80px;}
    table{border-collapse:collapse;width:100%;margin-top:10px;}
    th,td{border:1px solid #ccc;padding:6px;text-align:center;vertical-align:top;}
    th{background:#e0f3ff;}
    .btn{padding:6px 12px;margin:6px 0;border:none;border-radius:4px;background:#0a4d68;color:#fff;cursor:pointer;}
    .btn:disabled{opacity:.5;cursor:not-allowed;}
    .points-box{font-size:1.1em;margin:10px 0;font-weight:bold;}
    .reward{background:#d9ffd9;padding:4px 8px;border-radius:4px;display:inline-block;margin:4px;}
    #habitList div{margin:4px 0;}
    #habitList button{margin-left:8px;background:#d9534f;color:#fff;border:none;padding:2px 6px;border-radius:3px;}
    .info{margin-top:8px;font-weight:bold;}
</style>
</head>
<body>

<h1>85-kg-Challenge – persönlicher Online-Tracker</h1>

<section id="periodSection">
    <h2>Challenge-Zeitraum & Ziel</h2>
    <label>Startdatum:</label>
    <input type="date" id="periodStart">
    <label>Enddatum:</label>
    <input type="date" id="periodEnd">
    <button id="periodSave" class="btn">Speichern</button>
    <p id="periodDisplay" class="info"></p>
    <label>Startgewicht (kg):</label>
    <input type="number" step="0.1" id="startWeight">
    <label>Zielgewicht (kg):</label>
    <input type="number" step="0.1" id="goalWeight">
    <p id="goalEval" class="info"></p>
</section>

<section id="manageHabits">
    <h2>Gewohnheiten verwalten</h2>
    <input type="text" id="habitName" placeholder="Neue Gewohnheit (Label)">
    <button id="addHabit" class="btn">Hinzufügen</button>
    <div id="habitList"></div>
</section>

<section id="dailySection">
    <h2>Täglicher Habit-Tracker</h2>
    <form id="dailyForm">
        <label>Datum:</label>
        <input type="date" id="dailyDate" required>
        <div id="habits"></div>
        <label>Journal-Eintrag:</label>
        <textarea id="journalText" placeholder="Notizen..."></textarea>
        <button type="submit" class="btn">Tag speichern</button>
    </form>
    <table id="dailyTable"></table>
    <div class="points-box">Wöchentliche Punkte: <span id="weeklyPoints">0</span> / <span id="weeklyMax">0</span></div>
</section>

<section id="measureSection">
    <h2>Wöchentliche Messung</h2>
    <form id="measureForm">
        <label>Mess-Datum (Montag):</label>
        <input type="date" id="measureDate" required>
        <label>Gewicht (kg): <input type="number" step="0.1" id="weight" required></label>
        <label>Bauch (cm): <input type="number" step="0.1" id="belly" required></label>
        <label>Brust (cm): <input type="number" step="0.1" id="chest"></label>
        <label>Bizeps (cm): <input type="number" step="0.1" id="biceps"></label>
        <label>Oberschenkel (cm): <input type="number" step="0.1" id="thigh"></label>
        <label>Unterschenkel (cm): <input type="number" step="0.1" id="calf"></label>
        <button type="submit" class="btn">Messung speichern</button>
    </form>
    <table id="measureTable">
        <thead>
            <tr>
                <th>Datum</th><th>Gewicht</th><th>Δ kg</th><th>Bauch</th><th>Δ cm</th><th>Punkte</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
</section>

<section id="journalSection">
    <h2>Tagebuch Einträge</h2>
    <ul id="journalList"></ul>
</section>

<section id="totalSection">
    <h2>Gesamt-Punktestand</h2>
    <div class="points-box">Aktuell: <span id="totalPoints">0</span> / <span id="totalMax">0</span></div>
    <div id="rewards"></div>
</section>

<script>
(function(){
    const storeKey = 'abnehmtoolData';
    const data = JSON.parse(localStorage.getItem(storeKey) || '{}');
    data.period = data.period || {};
    data.habitNames = data.habitNames || ['kcal','protein','sugarFree','water','rennrad','stretching','kraft','hrv','gebet','sleep'];
    data.habitLabels = data.habitLabels || {
        kcal:'Kalorien-Budget eingehalten',
        protein:'≥ 150 g Protein',
        sugarFree:'Zucker 0 g',
        water:'3 L Wasser',
        rennrad:'Rennradtraining',
        stretching:'Tägliches Stretching',
        kraft:'Krafttraining',
        hrv:'Atemübungen',
        gebet:'Gebet',
        sleep:'≥ 7 h Schlaf'
    };
    data.daily = data.daily || [];
    data.measure = data.measure || [];

    // DOM
    const periodStart = document.getElementById('periodStart');
    const periodEnd = document.getElementById('periodEnd');
    const periodSave = document.getElementById('periodSave');
    const periodDisplay = document.getElementById('periodDisplay');
    const startWeight = document.getElementById('startWeight');
    const goalWeight = document.getElementById('goalWeight');
    const goalEval = document.getElementById('goalEval');

    function save(){ localStorage.setItem(storeKey, JSON.stringify(data)); render(); }
    
    function calcDuration(s,e){
        const start = new Date(s), end = new Date(e);
        const diff = end - start;
        const days = Math.floor(diff/86400000);
        const weeks = (days/7).toFixed(1);
        const months = (days/30).toFixed(1);
        return {days,weeks,months};
    }
    function evaluateGoal(){
        const s = parseFloat(startWeight.value), g = parseFloat(goalWeight.value);
        if(!data.period.start || !data.period.end || isNaN(s) || isNaN(g)) return goalEval.textContent = ''; 
        const d = calcDuration(data.period.start, data.period.end);
        const targetLoss = s - g;
        const maxSafe = parseFloat(d.weeks); // 1 kg per week
        let msg = '';
        if(targetLoss <= maxSafe) msg = 'Das Ziel ist realistisch (' + targetLoss.toFixed(1) + ' kg in ' + d.weeks + ' Wochen).';
        else msg = 'Vorsicht: ' + targetLoss.toFixed(1) + ' kg in ' + d.weeks + ' Wochen sind ambitioniert!';
        goalEval.textContent = msg;
    }

    periodSave.onclick = () => {
        const s = periodStart.value, e = periodEnd.value;
        if(!s || !e) return alert('Bitte Start und Ende wählen');
        if(new Date(s) >= new Date(e)) return alert('Ende muss nach Start liegen');
        data.period.start = s; data.period.end = e; save();
    };

    function renderPeriod(){
        if(data.period.start){ 
            periodStart.value = data.period.start; 
            periodEnd.value = data.period.end; 
            const duration = calcDuration(data.period.start, data.period.end);
            periodDisplay.textContent = 'Dauer: ' + duration.days + ' Tage (' + duration.weeks + ' Wochen / ' + duration.months + ' Monate)';
        }
        if(data.period.startWeight){ startWeight.value = data.period.startWeight; }
        if(data.period.goalWeight){ goalWeight.value = data.period.goalWeight; }

        startWeight.onchange = () => { data.period.startWeight = startWeight.value; save(); };
        goalWeight.onchange = () => { data.period.goalWeight = goalWeight.value; save(); };
        evaluateGoal();
    }

    function render(){ renderPeriod(); /* Weitere Render-Funktionen für Habits, Daily, Measure, Journal, Punkte... */ }

    render();
})();
</script>

</body>
</html>
