<!doctype html>
<html lang="uk">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Тест — Симулятор (Оновлено)</title>
<style>
  body{font-family:system-ui; background:#f3f4f6; margin:0; padding:24px}
  .container{max-width:900px; margin:0 auto; background:#fff; padding:20px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.1)}
  h1{margin:0 0 12px; font-size:22px}
  .question{margin:18px 0; padding:14px; border-radius:6px; border:1px solid #e5e7eb}
  .options{display:grid; grid-template-columns:1fr 1fr; gap:8px}
  label.opt{padding:10px; border:1px solid #ddd; border-radius:6px; cursor:pointer; display:block}
  .controls{display:flex; justify-content:space-between; margin-top:16px}
  button{background:#0ea5a4; color:white; padding:8px 12px; border:none; border-radius:6px; cursor:pointer}
  button.secondary{background:#e5e7eb; color:#111}
  .timer{font-size:18px; font-weight:600; color:#b91c1c; margin-bottom:12px}
</style>
</head>
<body>
<div class="container">
  <h1>Тест — Симулятор</h1>

  <div class="timer">Час: <span id="time">30:00</span></div>

  <div style="margin-bottom:15px">
    <label><strong>Введіть ПІБ:</strong></label><br>
    <input id="username" type="text" style="width:100%; padding:8px; margin-top:6px; border:1px solid #ccc; border-radius:6px" />
  </div>

  <p>Питань: <span id="total">0</span></p>
  <div id="app"></div>
</div>

<script>
// використано питання та відповіді як у вихідному файлі (скорочено синтаксис для прикладу)

const questions = [
  {id:1,text:"Що означає попереджувальний сигнал 'Увага всім!'?",options:{A:"Потрібно негайно евакуюватися",B:"Увімкнути радіо/телебачення",C:"Виходити на вулицю",D:"Чекати інструкцій"}},
];

const answers = {1:"B"};

let state = { index:0, choices:{}, showResults:false };

// ------------------ ТАЙМЕР 30 хв ------------------
let timeLeft = 30 * 60;
const timerEl = document.getElementById('time');
const timer = setInterval(()=>{
  const m = Math.floor(timeLeft/60);
  const s = timeLeft%60;
  timerEl.textContent = `${m}:${s.toString().padStart(2,'0')}`;
  if (timeLeft <= 0){
    clearInterval(timer);
    finishTest();
  }
  timeLeft--;
},1000);

// --------------------------------------------------

function render(){
  const app = document.getElementById('app');
  document.getElementById('total').textContent = questions.length;

  if (state.showResults){
      const score = grade();
      const percent = Math.round(score/questions.length*100);
      const username = document.getElementById('username').value || 'Невідомо';

      const passed = percent >= 50;

      const resultText = passed ? "Тест СКЛАДЕНО" : "ТЕСТ НЕ СКЛАДЕНО";

      const body = `ПІБ: ${username}\nРезультат: ${score} з ${questions.length} (${percent}%)\nСтатус: ${resultText}`;

      sendEmail(body);

      app.innerHTML = `
        <div>
          <h2>${resultText}</h2>
          <p>Балів: <strong>${score}</strong> / ${questions.length}</p>
          <button onclick="location.reload()">Пройти знову</button>
        </div>`;
      return;
  }

  const q = questions[state.index];

  app.innerHTML = `
    <div class='question'>
      <div><strong>Питання ${state.index+1}:</strong> ${q.text}</div>
      <div class="options">
        ${Object.entries(q.options).map(([k,v])=>`
           <label class='opt'><input type='radio' name='q${q.id}' value='${k}' ${state.choices[q.id]===k?'checked':''}> <strong>${k}.</strong> ${v}</label>
        `).join('')}
      </div>

      <div class="controls">
        <button class="secondary" onclick="prev()" ${state.index===0?'disabled':''}>Назад</button>
        <button class="secondary" onclick="next()" ${state.index===questions.length-1?'disabled':''}>Далі</button>
        <button onclick="finishTest()">Завершити</button>
      </div>
    </div>
  `;

  document.querySelectorAll('input[type=radio]').forEach(r=>{
    r.addEventListener('change', e =>{
      state.choices[q.id] = e.target.value;
    });
  });
}

function prev(){ state.index--; render(); }
function next(){ state.index++; render(); }

function finishTest(){
  state.showResults = true;
  render();
}

function grade(){
  let sc = 0;
  for (const q of questions){ if (state.choices[q.id]===answers[q.id]) sc++; }
  return sc;
}

// ------------------ ВІДПРАВКА РЕЗУЛЬТАТІВ ------------------
function sendEmail(text){
  fetch('https://formsubmit.co/ajax/k.karpynets@esbu.gov.ua',{
    method:'POST',
    headers:{ 'Content-Type':'application/json' },
    body: JSON.stringify({ message:text })
  });
}
// ------------------------------------------------------------

render();
</script>
</body>
</html>
