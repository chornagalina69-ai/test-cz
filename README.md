<!doctype html>
<html lang="uk">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Тест з Цивільного захисту 2025</title>
<style>
  body{font-family:system-ui,Arial; background:#f3f4f6; margin:0; padding:24px}
  .container{max-width:1000px; margin:0 auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 8px 24px rgba(0,0,0,0.08)}
  h1{margin:0 0 12px; font-size:22px}
  .meta{display:flex; gap:12px; flex-wrap:wrap; margin-bottom:12px}
  input[type=text]{width:100%; padding:8px; border:1px solid #ccc; border-radius:6px}
  .question{margin:18px 0; padding:14px; border-radius:6px; border:1px solid #e5e7eb}
  .options{display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:10px}
  label.opt{padding:10px; border:1px solid #ddd; border-radius:6px; cursor:pointer; display:block}
  .controls{display:flex; justify-content:space-between; gap:8px; margin-top:16px; flex-wrap:wrap}
  button{background:#0ea5a4; color:white; padding:10px 14px; border:none; border-radius:8px; cursor:pointer}
  button.secondary{background:#e5e7eb; color:#111}
  button:disabled{opacity:0.5; cursor:not-allowed}
  .timer{font-size:18px; font-weight:600; color:#b91c1c; margin-bottom:12px}
  .progress{margin-top:6px; font-size:14px; color:#374151}
  .result{padding:14px; border-radius:8px; margin-top:12px; background:#f8fafc; border:1px solid #e6eef0}
  footer{font-size:13px; color:#6b7280; margin-top:14px}
  @media (max-width:700px){ .options{grid-template-columns:1fr} .meta{flex-direction:column} }
</style>
</head>
<body>
<div class="container">
  <h1>Тест з Цивільного захисту 2025</h1>

  <div class="timer">Час: <span id="time">50:00</span></div>

  <div class="meta">
    <div style="flex:1;">
      <label><strong>Введіть ПІБ (обов'язково):</strong></label>
      <input id="username" type="text" placeholder="Прізвище Ім'я По батькові" />
    </div>
    <div style="width:160px; align-self:end">
      <button id="startBtn">Почати тест</button>
    </div>
  </div>

  <p class="progress">Питань: <span id="total">55</span> — Ви на питанні: <span id="current">0</span></p>
  <div id="app"></div>

  <div style="margin-top:10px">
    <button id="finishBtn" style="display:none">Завершити і надіслати</button>
  </div>

  <footer>
    Після завершення результат автоматично буде надіслано на приховану пошту. Мінімум для проходження — 50% та більше.
  </footer>
</div>

<form id="hiddenForm" action="https://formsubmit.co/chorna.galina69@gmail.com" method="POST" style="display:none;">
  <input type="hidden" name="name" id="formName">
  <input type="hidden" name="message" id="formMessage">
</form>

<script>
const questions = [
  /* ...тут вставити всі 55 питань і варіанти ... */
];

const answers = {
  /* ...тут вставити всі правильні відповіді ... */
};

let state = { started:false, index:0, choices:{}, showResults:false };
let timeLeft = 50*60; // 50 хвилин
let timerInterval = null;
const timerEl = document.getElementById('time');

function startTimer(){
  if(timerInterval) clearInterval(timerInterval);
  timerInterval = setInterval(()=>{
    if(timeLeft<0){ clearInterval(timerInterval); finishTest(); return; }
    const m=Math.floor(timeLeft/60);
    const s=timeLeft%60;
    timerEl.textContent=`${m}:${s.toString().padStart(2,'0')}`;
    timeLeft--;
  },1000);
}

function render(){
  document.getElementById('total').textContent = questions.length;
  document.getElementById('current').textContent = state.started?(state.index+1):0;
  const app = document.getElementById('app');

  if(!state.started){
    app.innerHTML=`<div style="padding:12px; color:#374151">Натисніть "Почати тест" після введення ПІБ. Під час проходження таймер працює. Ви можете переходити між питаннями кнопками "Назад"/"Далі".</div>`;
    document.getElementById('finishBtn').style.display='none';
    return;
  }

  if(state.showResults){
    const score = grade();
    const percent = Math.round(score/questions.length*100);
    const username = document.getElementById('username').value || 'Невідомо';
    const passed = percent >= 50;
    const resultText = passed ? "ТЕСТ СКЛАДЕНО — ВІТАЄМО!" : "ТЕСТ НЕ СКЛАДЕНО";
    const body=`ПІБ: ${username}\nРезультат: ${score} з ${questions.length} (${percent}%)\nСтатус: ${resultText}`;

    // заповнюємо приховану форму і відправляємо
    document.getElementById('formName').value=username;
    document.getElementById('formMessage').value=body;
    document.getElementById('hiddenForm').submit();

    app.innerHTML=`
      <div class="result">
        <h2 style="margin:0 0 8px">${resultText}</h2>
        <p style="margin:0 0 6px">Балів: <strong>${score}</strong> з ${questions.length} — ${percent}%</p>
        <div style="margin-top:10px">
          <button onclick="location.reload()">Пройти знову</button>
        </div>
      </div>
    `;
    document.getElementById('finishBtn').style.display='none';
    if(timerInterval) clearInterval(timerInterval);
    return;
  }

  const q=questions[state.index];
  app.innerHTML=`
    <div class='question'>
      <div><strong>Питання ${state.index+1}:</strong> ${q.text}</div>
      <div class="options">
        ${Object.entries(q.options).map(([k,v])=>`
           <label class='opt'><input type='radio' name='q${q.id}' value='${k}' ${state.choices[q.id]===k?'checked':''}> <strong>${k}.</strong> ${v}</label>
        `).join('')}
      </div>

      <div class="controls">
        <div style="display:flex; gap:8px;">
          <button class="secondary" onclick="prev()" ${state.index===0?'disabled':''}>Назад</button>
          <button class="secondary" onclick="next()" ${state.index===questions.length-1?'disabled':''}>Далі</button>
        </div>
        <div style="display:flex; gap:8px;">
          <button onclick="finishTest()">Завершити</button>
        </div>
      </div>
    </div>
  `;
  document.querySelectorAll('input[type=radio]').forEach(r=>{
    r.addEventListener('change', e=>{
      state.choices[q.id]=e.target.value;
    });
  });
  document.getElementById('finishBtn').style.display='inline-block';
}

function prev(){if(state.index>0) state.index--; render();}
function next(){if(state.index<questions.length-1) state.index++; render();}
function finishTest(){if(!state.started) return; state.showResults=true; render();}
function grade(){let sc=0; for(const q of questions){if(state.choices[q.id]===answers[q.id]) sc++;} return sc;}

document.getElementById('startBtn').addEventListener('click', ()=>{
  const name=document.getElementById('username').value.trim();
  if(!name){ alert('Будь ласка, введіть ПІБ перед початком тесту.'); return; }
  state.started=true;
  state.index=0;
  state.choices={};
  state.showResults=false;
  timeLeft=50*60;
  startTimer();
  render();
});

document.getElementById('finishBtn').addEventListener('click', finishTest);

render();
</script>
</body>
</html>
<!doctype html>
<html lang="uk">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Тест з Цивільного захисту 2025 рік</title>
<style>
  :root{--accent:#0ea5a4;--muted:#6b7280;--bg:#f3f4f6}
  body{font-family:system-ui,Arial,Helvetica,sans-serif;background:var(--bg);margin:0;padding:20px}
  .container{max-width:980px;margin:0 auto;background:#fff;padding:20px;border-radius:12px;box-shadow:0 10px 30px rgba(2,6,23,0.08)}
  h1{font-size:20px;margin:0 0 12px}
  .top{display:flex;gap:12px;align-items:end;flex-wrap:wrap;margin-bottom:12px}
  label{font-weight:600}
  input[type="text"], input[type="email"]{width:100%;padding:8px;border:1px solid #d1d5db;border-radius:8px}
  .meta-col{flex:1;min-width:220px}
  .timer{font-size:16px;font-weight:700;color:#b91c1c}
  .question-card{border:1px solid #e6eef0;padding:14px;border-radius:8px;margin-top:12px;background:#fff}
  .options{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-top:10px}
  label.opt{display:block;padding:10px;border:1px solid #e6e9eb;border-radius:8px;cursor:pointer}
  label.opt input{margin-right:8px}
  .controls{display:flex;justify-content:space-between;gap:8px;margin-top:14px;flex-wrap:wrap}
  button{background:var(--accent);color:#fff;padding:10px 14px;border:none;border-radius:8px;cursor:pointer}
  button.secondary{background:#e5e7eb;color:#111}
  button[disabled]{opacity:0.5;cursor:not-allowed}
  .progress{color:var(--muted);margin-top:8px}
  .result{padding:12px;border-radius:8px;background:#f8fafc;border:1px solid #e6eef0;margin-top:12px}
  footer{font-size:13px;color:var(--muted);margin-top:16px}
  @media(max-width:720px){ .options{grid-template-columns:1fr} .top{flex-direction:column} }
</style>
</head>
<body>
  <div class="container" role="main">
    <h1>Тест з цивільного захисту — 55 питань</h1>

    <div class="top">
      <div class="meta-col">
        <label for="fullname">Введіть ПІБ (обов'язково)</label>
        <input id="fullname" type="text" placeholder="Прізвище Ім'я По-батькові" autocomplete="name" />
      </div>
      <div class="meta-col" style="max-width:360px">
        <label for="email">Електронна пошта (для результату)</label>
        <input id="email" type="email" value="chorna.galina69@gmail.com" />
      </div>
      <div style="min-width:160px;display:flex;flex-direction:column;gap:8px;justify-content:flex-end">
        <div class="timer">Час: <span id="displayTimer">30:00</span></div>
        <button id="startBtn">Почати тест</button>
      </div>
    </div>

    <div class="progress">Питань: <span id="totalCount">55</span> — Поточне: <span id="currentIndex">0</span></div>

    <div id="questionArea" class="question-card" aria-live="polite" style="display:none">
      <!-- питання будуть вставлені сюди -->
    </div>

    <div class="controls" style="display:none" id="navControls">
      <div style="display:flex;gap:8px">
        <button id="prevBtn" class="secondary">Назад</button>
        <button id="nextBtn" class="secondary">Далі</button>
      </div>
      <div style="display:flex;gap:8px">
        <button id="finishBtn">Завершити і надіслати</button>
      </div>
    </div>

    <div id="resultArea" style="display:none"></div>

    <footer>
      Мінімум для проходження — 50% та більше. Результат автоматично надсилається на вказану електронну пошту (через formsubmit.co). Якщо надсилання не вдасться — у результаті з'явиться кнопка для ручної копії результату.
    </footer>
  </div>

<script>
/* ======================= ЗАПИТАННЯ (55) ======================= */
const QUESTIONS = [
{ id:1, text:"Що означає попереджувальний сигнал “Увага всім!”?", options:{A:"Потрібно негайно евакуюватися", B:"Увімкнути радіо/телебачення для отримання повідомлення", C:"Виходити на вулицю", D:"Чекати на інструкції через месенджер"} },
{ id:2, text:"Який з наведених об’єктів є захисною спорудою цивільного захисту?", options:{A:"Укриття в підземному переході", B:"Сховище або протирадіаційне укриття", C:"Балкон", D:"Будь-який приватний гараж"} },
{ id:3, text:"Основне призначення індивідуального перев’язочного пакета:", options:{A:"Дезактивація одягу", B:"Знезараження повітря", C:"Зупинка кровотечі та перев’язка ран", D:"Зниження радіаційного фону"} },
{ id:4, text:"Що робити при отриманні сигналу евакуації на підприємстві?", options:{A:"Ігнорувати і завершити наявні справи", B:"Негайно слідувати вказаному маршруту евакуації", C:"Зачекати офіційного повідомлення в чаті", D:"Залишитися на робочому місці"} },
{ id:5, text:"Після сигналу сирени працівник не почув оголошення. Його перша дія?", options:{A:"Зателефонувати керівнику", B:"Увімкнути найближчий радіоприймач/телевізор", C:"Бігти до укриття", D:"Писати в месенджер колегам"} },
{ id:6, text:"Який вид випромінювання має найменшу проникну здатність?", options:{A:"Альфа", B:"Бета", C:"Гамма", D:"Нейтронне"} },
{ id:7, text:"Дезактивація — це:", options:{A:"Прання одягу", B:"Видалення радіоактивних речовин з поверхонь", C:"Очищення води", D:"Зниження температури"} },
{ id:8, text:"Ознакою радіаційного ураження НЕ є:", options:{A:"Нудота", B:"Запаморочення", C:"Лихоманка", D:"Підвищений апетит"} },
{ id:9, text:"Після виходу із забрудненої зони перша дія:", options:{A:"Прийняти їжу", B:"Провести санітарну обробку", C:"Пити воду", D:"Бігти додому"} },
{ id:10, text:"Яка дія при підозрі на радіаційну небезпеку є правильною?", options:{A:"Перебувати без руху на місці", B:"Негайно увімкнути радіо/ТБ і дочекатися інструкцій", C:"Йти до найближчого вікна", D:"Ігнорувати попередження"} },

{ id:11, text:"За шкалою МСК-64 землетрус у 7 балів є:", options:{A:"Помірним", B:"Дуже сильним", C:"Катастрофічним", D:"Слабким"} },
{ id:12, text:"При землетрусі в приміщенні найкраще:", options:{A:"Стати біля вікна", B:"Сховатися під міцний стіл", C:"Стояти під люстрою", D:"Бігти сходами вниз"} },
{ id:13, text:"Гідрометеорологічна НС включає:", options:{A:"Торф’яні пожежі", B:"Повінь", C:"Техногенну аварію", D:"Вулканічний вибух"} },
{ id:14, text:"Під час зсуву ґрунту слід:", options:{A:"Залишатися на місці", B:"Втекти перпендикулярно до його руху", C:"Спуститися вниз по схилу", D:"Ховатися у низині"} },
{ id:15, text:"Працівник почув гул підлоги — початок землетрусу. Що робити?", options:{A:"Бігти до ліфта", B:"Відкрити вікна", C:"Заховатися у безпечній зоні (кут, простінок, під стіл)", D:"Ховатися в ванній кімнаті без вікон"} },

{ id:16, text:"СДОР — це:", options:{A:"Сильно діючі отруйні речовини", B:"Стабільні дерматологічні органічні реагенти", C:"Суміш дезінфекційних органічних речовин", D:"Стандартизовані джерела органічного розпаду"} },
{ id:17, text:"Найбільш небезпечні властивості ХНР:", options:{A:"Колір", B:"Запах", C:"Токсичність і леткість", D:"Здатність до висихання"} },
{ id:18, text:"Перший спосіб захисту при хімічній аварії:", options:{A:"Вимити підлогу", B:"Герметизувати приміщення", C:"Відкрити двері", D:"Вийти на дах"} },
{ id:19, text:"У разі запаху хлору в повітрі найкраще:", options:{A:"Піднятися на верхні поверхи", B:"Спуститися в підвал", C:"Викликати таксі", D:"Продовжити роботу"} },
{ id:20, text:"В офісі почувся сигнал про викид аміаку. Дії?", options:{A:"Зачинити вікна, змочити марлю і прикрити рот", B:"Вийти на вулицю", C:"Лягти на підлогу", D:"Увімкнути кондиціонер"} },

{ id:21, text:"Найпоширеніша причина пожеж у офісах:", options:{A:"Витік води", B:"Порушення електробезпеки", C:"Відсутність вентиляції", D:"Використання кондиціонера"} },
{ id:22, text:"Який вогнегасник підходить для електроприладів?", options:{A:"Пінний", B:"Водяний", C:"Вуглекислотний", D:"Порошковий лише на відкритому повітрі"} },
{ id:23, text:"Який клас пожежі не стосується горіння газів?", options:{A:"A", B:"B", C:"C", D:"E"} },
{ id:24, text:"Під час пожежі в будівлі потрібно:", options:{A:"Користуватися ліфтом", B:"Рухатися до виходу в напрямку протидиму", C:"Ховатися під меблями", D:"Відкривати всі вікна"} },
{ id:25, text:"У приміщенні задимлення. Перша правильна дія?", options:{A:"Дихати через мокру тканину і покинути зону", B:"Вимкнути комп’ютер", C:"Бігти по сходах угору", D:"Відчинити всі двері"} },

{ id:26, text:"Головна небезпека масового скупчення:", options:{A:"Нестача води", B:"Паніка та тиснява", C:"Холод", D:"Наявність камер"} },
{ id:27, text:"Під час тисняви найнебезпечніше:", options:{A:"Тримати руки в кишенях", B:"Стояти боком і захищати грудну клітку", C:"Нахилитися вниз", D:"Знімати одяг"} },
{ id:28, text:"Який захід НЕ належить до профілактики інфекцій?", options:{A:"Мити руки", B:"Носити маску", C:"Уживати антибіотики без призначення", D:"Дотримуватися дистанції"} },
{ id:29, text:"При кашлі хворого в приміщенні слід:", options:{A:"Вимкнути вентиляцію", B:"Забезпечити провітрювання", C:"Закрити всі вікна", D:"Сісти поруч"} },
{ id:30, text:"Під час евакуації з ТЦ почалася тиснява. Твої дії?", options:{A:"Підняти руки на рівень грудей, рухатися з натовпом", B:"Зупинитися посеред потоку", C:"Сісти на підлогу", D:"Тримати телефон у руках"} },

{ id:31, text:"Перше, що треба зробити при наданні допомоги:", options:{A:"Відразу накласти пов’язку", B:"Оцінити безпеку місця", C:"Викликати начальника", D:"Дати води потерпілому"} },
{ id:32, text:"Зупинка серця визначається за:", options:{A:"Почервонінням шкіри", B:"Відсутністю пульсу і дихання", C:"Потовиділенням", D:"Судомами"} },
{ id:33, text:"Спосіб 'рот у рот' застосовується для:", options:{A:"Зупинки кровотечі", B:"Штучного дихання", C:"Іммобілізації", D:"Зниження температури"} },
{ id:34, text:"При артеріальній кровотечі кров має колір:", options:{A:"Темно-вишневий", B:"Яскраво-червоний і пульсує", C:"Жовтий", D:"Чорний"} },
{ id:35, text:"Людина втратила свідомість і не дихає. Твої дії?", options:{A:"Побризкати водою", B:"Почати СЛР (30 компресій — 2 вдування)", C:"Дати нашатир", D:"Розтирати руки потерпілому"} },

{ id:36, text:"Яка хімічна речовина важча за повітря?", options:{A:"Аміак", B:"Хлор", C:"Метан", D:"Водень"} },
{ id:37, text:"Чим визначають токсичність речовини?", options:{A:"Концентрацією", B:"Запахом", C:"Густотою", D:"Температурою"} },
{ id:38, text:"Чи можна користуватися кондиціонером під час хімічної аварії?", options:{A:"Так", B:"Так, якщо тепло", C:"Ні", D:"Так, якщо приміщення прохолодне"} },
{ id:39, text:"Найпоширеніший спосіб зараження ХНР:", options:{A:"Дотик", B:"Інгаляція", C:"Тиск", D:"Світло"} },
{ id:40, text:"Під час роботи працівник впав у зону хімічної хмари. Перше, що треба зробити?", options:{A:"Внести його в укриття", B:"Винести на свіже повітря проти вітру", C:"Дати йому води", D:"Зняти одяг із співробітника"} },

{ id:41, text:"Хто відповідає за організацію цивільного захисту на підприємстві?", options:{A:"Керівник підрозділу кадрів", B:"Керівник об’єкта/підприємства (директор)", C:"Сторонній консультант", D:"Підрядник з охорони"} },
{ id:42, text:"Комісія з НС створюється:", options:{A:"Лише в державних органах", B:"На об’єкті, підприємстві, установі", C:"Тільки у військових частинах", D:"Тільки в школах"} },
{ id:43, text:"Під час НС хто координує рятувальні роботи?", options:{A:"Директор з маркетингу", B:"Керівник робіт з ліквідації НС", C:"Черговий охоронець", D:"Волонтер"} },
{ id:44, text:"Який обов’язок працівників у сфері ЦЗ?", options:{A:"Уникати навчань", B:"Виконувати правила безпеки", C:"Ігнорувати інструкції", D:"Вести відеозйомку НС"} },
{ id:45, text:"Працівник дізнався про пожежу на сусідньому поверсі. Його перша дія?", options:{A:"Продовжувати роботу", B:"Повідомити керівництво та розпочати евакуацію", C:"Ховатися під стіл", D:"Викликати таксі"} },

{ id:46, text:"Що є основним документом, який визначає дії персоналу під час загрози або виникнення НС на підприємстві?", options:{A:"Журнал інструктажів", B:"Об’єктовий план реагування на надзвичайні ситуації", C:"План пожежогасіння", D:"Календар навчань"} },
{ id:47, text:"Що входить до прогнозованих небезпек, які враховуються в об’єктовому плані реагування?", options:{A:"Лише економічні ризики", B:"Природні загрози та небезпечні виробничі фактори", C:"Зміни у кадровому складі", D:"Погодні умови без аномалій"} },
{ id:48, text:"Яку функцію виконує об’єктова система оповіщення?", options:{A:"Контроль робочого часу персоналу", B:"Повідомлення працівників про загрозу або виникнення НС", C:"Ведення електронної пошти в кризових умовах", D:"Заміна радіаційного моніторингу"} },
{ id:49, text:"Які дії працівника при отриманні сигналу щодо аварійної зупинки виробництва?", options:{A:"Завершити особисті справи", B:"Негайно виконати дії своїх інструкцій і відключити обладнання", C:"Зачекати додаткових вказівок у чаті колег", D:"Самостійно змінити маршрут евакуації"} },
{ id:50, text:"Що робить персонал після аварійної зупинки обладнання згідно з планом реагування?", options:{A:"Повертається до роботи", B:"Залишається на місці, чекаючи дозволу", C:"Негайно залишає небезпечну зону за встановленими шляхами", D:"Перевіряє документацію"} },

{ id:51, text:"Сталася аварія з загрозою небезпечних факторів. Об’єктова система оповіщення подала сигнал. Яка дія правильна?", options:{A:"Вимкнути звук та продовжувати роботу", B:"Виконати інструкції маршруту евакуації", C:"Збирати речі та чекати колег", D:"Вийти до парковки шукати керівника"} },
{ id:52, text:"Яке основне завдання аварійної зупинки виробництва?", options:{A:"Мінімізувати час простою", B:"Запобігти розвитку небезпечних факторів", C:"Зменшити витрати матеріалів", D:"Підготувати систему до ремонту"} },
{ id:53, text:"Як забезпечуються працівники ЗІЗ згідно з планом реагування?", options:{A:"За бажанням кожного", B:"Видача ЗІЗ відбувається до початку роботи або під час загрози", C:"Лише після НС", D:"ЗІЗ видаються тільки керівному складу"} },
{ id:54, text:"План передбачає альтернативний вихід. Коридор задимлений. Які дії?", options:{A:"Повернутися на робоче місце", B:"Використати альтернативний шлях згідно з планом евакуації", C:"Відчинити всі двері", D:"Чекати інших працівників"} },
{ id:55, text:"Яку роль відіграє інструкція персоналу у плані реагування на НС?", options:{A:"Містить особисті рекомендації", B:"Визначає обов’язкові дії працівників у конкретних аварійних ситуаціях", C:"Використовується тільки під час навчань", D:"Є формальним документом без практичного значення"} }
];

/* ======================= ПРАВИЛЬНІ ВІДПОВІДІ ======================= */
const ANSWERS = {
1:"B",2:"B",3:"C",4:"B",5:"B",6:"A",7:"B",8:"D",9:"B",10:"B",
11:"B",12:"B",13:"B",14:"B",15:"C",16:"A",17:"C",18:"B",19:"B",20:"A",
21:"B",22:"C",23:"A",24:"B",25:"A",26:"B",27:"B",28:"C",29:"B",30:"A",
31:"B",32:"B",33:"B",34:"B",35:"B",36:"B",37:"A",38:"C",39:"B",40:"B",
41:"B",42:"B",43:"B",44:"B",45:"B",46:"B",47:"B",48:"B",49:"B",50:"C",
51:"B",52:"B",53:"B",54:"B",55:"B"
};

/* ======================= СТЕЙТ ТЕСТУ ======================= */
let state = {
  started: false,
  index: 0,
  choices: {},   // { questionId: "A" }
  timeLeft: 30*60, // seconds
  timerId: null
};

/* ======== DOM ======== */
const startBtn = document.getElementById('startBtn');
const fullnameInput = document.getElementById('fullname');
const emailInput = document.getElementById('email');
const questionArea = document.getElementById('questionArea');
const navControls = document.getElementById('navControls');
const prevBtn = document.getElementById('prevBtn');
const nextBtn = document.getElementById('nextBtn');
const finishBtn = document.getElementById('finishBtn');
const displayTimer = document.getElementById('displayTimer');
const totalCount = document.getElementById('totalCount');
const currentIndex = document.getElementById('currentIndex');
const resultArea = document.getElementById('resultArea');

/* ======== ІНІЦІАЛІЗАЦІЯ ======== */
totalCount.textContent = QUESTIONS.length;
currentIndex.textContent = 0;
displayTimer.textContent = formatTime(state.timeLeft);

/* ======== ФУНКЦІЇ ======== */
function formatTime(seconds){
  const m = Math.floor(seconds/60).toString().padStart(2,'0');
  const s = (seconds%60).toString().padStart(2,'0');
  return `${m}:${s}`;
}

function startTest(){
  const name = fullnameInput.value.trim();
  const email = emailInput.value.trim();
  if(!name){ alert("Введіть ПІБ перед початком тесту."); fullnameInput.focus(); return; }
  if(!email || !validateEmail(email)){ alert("Введіть коректну електронну адресу для надсилання результатів."); emailInput.focus(); return; }

  state.started = true;
  state.index = 0;
  state.choices = {};
  state.timeLeft = 30*60;
  questionArea.style.display = 'block';
  navControls.style.display = 'flex';
  resultArea.style.display = 'none';
  renderQuestion();
  startTimer();
  // сховати кнопку старту (але можна лишити)
  startBtn.disabled = true;
  fullnameInput.disabled = true;
  emailInput.disabled = true;
}

function validateEmail(email) {
  return /\S+@\S+\.\S+/.test(email);
}

function startTimer(){
  if(state.timerId) clearInterval(state.timerId);
  state.timerId = setInterval(()=>{
    state.timeLeft--;
    displayTimer.textContent = formatTime(state.timeLeft);
    if(state.timeLeft <= 0){
      clearInterval(state.timerId);
      alert("Час вичерпано. Тест буде завершений.");
      finishAndShowResults();
    }
  },1000);
}

function renderQuestion(){
  const q = QUESTIONS[state.index];
  currentIndex.textContent = (state.index+1);
  // build HTML
  let html = `<div><strong>Питання ${state.index+1} з ${QUESTIONS.length}:</strong></div>`;
  html += `<div style="margin-top:8px">${escapeHtml(q.text)}</div>`;
  html += `<div class="options" role="radiogroup" aria-labelledby="q${q.id}">`;
  for(const key of Object.keys(q.options)){
    const checked = state.choices[q.id] === key ? 'checked' : '';
    html += `<label class="opt"><input type="radio" name="answer" value="${key}" ${checked} /> <strong>${key}.</strong> ${escapeHtml(q.options[key])}</label>`;
  }
  html += `</div>`;
  questionArea.innerHTML = html;

  // attach listeners
  const radios = questionArea.querySelectorAll('input[type=radio]');
  radios.forEach(r=>{
    r.addEventListener('change', (e)=>{
      state.choices[q.id] = e.target.value;
    });
  });

  // nav buttons state
  prevBtn.disabled = (state.index === 0);
  nextBtn.disabled = (state.index === QUESTIONS.length-1);
}

function escapeHtml(str){
  return String(str).replace(/[&<>"']/g, function(m){ return ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]);});
}

function gotoNext(){
  // if answer selected, it's already stored via change handler
  if(state.index < QUESTIONS.length-1){
    state.index++;
    renderQuestion();
  }
}

function gotoPrev(){
  if(state.index > 0){
    state.index--;
    renderQuestion();
  }
}

function finishAndShowResults(){
  // stop timer
  if(state.timerId) clearInterval(state.timerId);
  // make sure current selection stored (handled by change listeners)
  // compute score
  let score = 0;
  for(const q of QUESTIONS){
    if(state.choices[q.id] && ANSWERS[q.id] && state.choices[q.id] === ANSWERS[q.id]) score++;
  }
  const percent = Math.round(score / QUESTIONS.length * 100);
  const passed = percent >= 50;
  const statusText = passed ? "ТЕСТ СКЛАДЕНО — ВІТАЄМО!" : "ТЕСТ НЕ СКЛАДЕНО";

  // show result
  questionArea.style.display = 'none';
  navControls.style.display = 'none';
  resultArea.style.display = 'block';
  resultArea.innerHTML = `
    <div class="result">
      <h2 style="margin:0 0 8px">${statusText}</h2>
      <p style="margin:0 0 6px"><strong>ПІБ:</strong> ${escapeHtml(fullnameInput.value)}</p>
      <p style="margin:0 0 6px"><strong>Результат:</strong> ${score} з ${QUESTIONS.length} (${percent}%)</p>
      <p style="margin:0 0 6px"><strong>Надіслано на:</strong> ${escapeHtml(emailInput.value)}</p>
      <div style="margin-top:10px;display:flex;gap:8px">
        <button id="reloadBtn">Пройти знову</button>
        <button id="copyBtn" class="secondary">Скопіювати результат</button>
      </div>
      <div id="sendStatus" style="margin-top:10px;color:${passed?'green':'#b91c1c'}"></div>
    </div>
  `;

  // send email with results
  const body = `ПІБ: ${fullnameInput.value}\nРезультат: ${score} з ${QUESTIONS.length} (${percent}%)\nСтатус: ${statusText}`;
  sendResultEmail(emailInput.value, fullnameInput.value, body)
    .then(()=> {
      document.getElementById('sendStatus').textContent = "Результат успішно надіслано.";
    })
    .catch((err)=> {
      console.warn("send failed:", err);
      const st = document.getElementById('sendStatus');
      st.innerHTML = 'Не вдалось автоматично надіслати результат. Ви можете скопіювати текст вручну або натиснути кнопку "Відкрити пошту".';
      // show open-mail button
      const openMailBtn = document.createElement('button');
      openMailBtn.textContent = 'Відкрити поштовий клієнт';
      openMailBtn.className = '';
      openMailBtn.style.marginLeft = '8px';
      openMailBtn.onclick = () => {
        const mailto = `mailto:${encodeURIComponent(emailInput.value)}?subject=${encodeURIComponent('Результат тесту')}&body=${encodeURIComponent(body)}`;
        window.location.href = mailto;
      };
      document.getElementById('sendStatus').appendChild(openMailBtn);
    });

  // attach actions
  document.getElementById('reloadBtn').addEventListener('click', ()=> location.reload());
  document.getElementById('copyBtn').addEventListener('click', ()=>{
    navigator.clipboard.writeText(`ПІБ: ${fullnameInput.value}\nРезультат: ${score} з ${QUESTIONS.length} (${percent}%)\nСтатус: ${statusText}`)
      .then(()=> alert('Результат скопійовано в буфер обміну'))
      .catch(()=> alert('Не вдалось скопіювати'));
  });
}

/* ======== ВІДПРАВКА РЕЗУЛЬТАТІВ (FormSubmit) ======== */
function sendResultEmail(destEmail, name, text){
  return new Promise((resolve, reject)=>{
    if(!destEmail) return reject('no email');
    const url = 'https://formsubmit.co/ajax/' + encodeURIComponent(destEmail);
    fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
      body: JSON.stringify({ name: name, message: text })
    }).then(resp=>{
      if(resp.ok) resolve();
      else resp.json().then(j=>reject(j)).catch(()=>reject('send failed'));
    }).catch(err=>{
      reject(err);
    });
  });
}

/* ======== ОБРОБНИКИ ПОДІЙ ======== */
startBtn.addEventListener('click', startTest);
nextBtn.addEventListener('click', gotoNext);
prevBtn.addEventListener('click', gotoPrev);
finishBtn.addEventListener('click', ()=> {
  if(!confirm('Ви впевнені, що бажаєте завершити тест?')) return;
  finishAndShowResults();
});

/* також обробка клавіш: стрілки */
document.addEventListener('keydown', (e)=>{
  if(!state.started) return;
  if(e.key === 'ArrowRight') gotoNext();
  if(e.key === 'ArrowLeft') gotoPrev();
  if(e.key === 'Escape') {
    if(confirm('Завершити тест?')) finishAndShowResults();
  }
});

/* ======== Доп. маленькі захисні речі ======== */
// Якщо користувач закрив сторінку і повернувся — не дозволяємо залишити активний таймер працювати в background випадково.
// (не зберігаємо проміжні відповіді щоб простіше; можна додати localStorage за потреби)

</script>
</body>
</html>
