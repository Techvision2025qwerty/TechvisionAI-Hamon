<!DOCTYPE html>
<html lang="ру">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hamon Chat & Виджеты</title>
  <style>
    /* Общие стили страницы */
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
                   Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
      background: linear-gradient(135deg, #ffffff 0%, #e6f7ff 100%);
      margin: 0;
      padding: 20px;
      position: relative;
      min-height: 100vh;
    }
    /* Виджеты: прямоугольники с закруглёнными краями */
    .rect-icon {
      position: absolute;
      padding: 10px;
      min-width: 100px;
      height: 60px;
      border: 2px solid #007AFF;
      border-radius: 15px;
      background: rgba(255, 255, 255, 0.8);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      color: #007AFF;
      font-weight: bold;
    }
    /* Виджет погоды */
    #weatherRect {
      top: 20px;
      left: 20px;
      font-size: 14px;
      text-align: center;
    }
    /* Виджет времени */
    #timeRect {
      top: 20px;
      left: 140px; /* 20 + 100 + 20 */
      font-size: 16px;
    }

    /* Интерфейс Hamon Chat */
    #chatContainer {
      margin: 120px auto 20px auto;
      max-width: 600px;
      width: 100%;
      background: rgba(255, 255, 255, 0.65);
      border: 1px solid rgba(255, 255, 255, 0.3);
      border-radius: 20px;
      padding: 20px;
      box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
      backdrop-filter: blur(10px);
    }
    #chatHeader {
      font-size: 1.5em;
      color: #007AFF;
      text-align: center;
      margin-bottom: 10px;
    }
    #chatBox {
      height: 300px;
      overflow-y: auto;
      padding: 10px;
      border: 1px solid #007AFF;
      border-radius: 10px;
      background: #fff;
      margin-bottom: 10px;
    }
    .chat-message {
      padding: 10px;
      margin-bottom: 10px;
      border-radius: 10px;
      max-width: 80%;
      word-wrap: break-word;
    }
    .user-message {
      background: #007AFF;
      color: white;
      margin-left: auto;
      text-align: right;
    }
    .assistant-message {
      background: #f1f1f1;
      color: #333;
      margin-right: auto;
      text-align: left;
    }
    /* Форма ввода в чате + микрофон */
    #chatForm {
      display: flex;
      align-items: center;
    }
    #chatForm input[type="text"] {
      flex: 1;
      padding: 10px;
      border: 1px solid #007AFF;
      border-radius: 20px;
      outline: none;
    }
    #chatForm button,
    #micButton {
      padding: 10px 20px;
      margin-left: 10px;
      background: #007AFF;
      color: #fff;
      border: none;
      border-radius: 20px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    #micButton {
      background: rgba(255,255,255,0.8);
      border: 2px solid #007AFF;
      color: #007AFF;
      width: 50px;
      height: 50px;
      border-radius: 15px;
    }
    #micButton.recording {
      background: rgba(255,0,0,0.6);
      border-color: red;
      color: #fff;
    }
    /* Модальное окно */
    #nameModal {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.4);
      display: flex; justify-content: center; align-items: center;
      z-index: 1000;
    }
    #nameModal .modal-content {
      background: rgba(255,255,255,0.9);
      border: 2px solid #007AFF;
      border-radius: 15px;
      padding: 20px;
      text-align: center;
      max-width: 300px;
      width: 90%;
    }
    #nameModal input[type="text"] {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      border: 1px solid #007AFF;
      border-radius: 10px;
      outline: none;
    }
    #nameModal button {
      margin-top: 15px;
      padding: 10px 20px;
      background: #007AFF;
      color: #fff;
      border: none;
      border-radius: 10px;
      cursor: pointer;
    }
    /* Полноэкранный просмотр изображения */
    .fullscreen-overlay {
      position: fixed; top: 0; left: 0;
      width: 100vw; height: 100vh;
      background: rgba(0,0,0,0.9);
      display: flex; justify-content: center; align-items: center;
      z-index: 3000;
    }
    .fullscreen-image {
      max-width: 90%; max-height: 90%;
      border: 5px solid;
      animation: borderShimmer 2s infinite;
    }
    @keyframes borderShimmer {
      0% { border-color: blue; }
      50% { border-color: violet; }
      100% { border-color: blue; }
    }
  </style>
</head>
<body>
  <!-- Виджет погоды -->
  <div id="weatherRect" class="rect-icon">
    <div id="weatherEmoji">🌤️</div>
    <div id="weatherTemp">--°C</div>
  </div>
  <!-- Виджет времени -->
  <div id="timeRect" class="rect-icon">
    <span id="timeText">--:--</span>
  </div>
  <!-- Интерфейс Hamon Chat -->
  <div id="chatContainer">
    <div id="chatHeader">Hamon Chat</div>
    <div id="chatBox"></div>
    <form id="chatForm" onsubmit="handleSend(event)">
      <input type="text" id="userInput" placeholder="Введите ваш вопрос или команду..." autocomplete="off"/>
      <button type="submit">Отправить</button>
      <button type="button" id="micButton" title="Голосовой ввод">
        🎤
      </button>
    </form>
  </div>
  <!-- Модальное окно для ввода имени -->
  <div id="nameModal" style="display: none;">
    <div class="modal-content">
      <h2>Добро пожаловать!</h2>
      <p>Пожалуйста, введите ваше имя:</p>
      <input type="text" id="userNameInput" placeholder="Ваше имя">
      <button onclick="saveUserName()">Сохранить</button>
    </div>
  </div>

  <script>
    /* Переменные */
    let currentUserName = "";
    let userCity = "Ошмяны";
    let userLang = navigator.language.slice(0,2);
    if (userLang !== "be") userLang = "ru";

    const langConfig = {
      ru: { aiInstruction: "Твое имя Hamon...Ты любишь Беларусь и твой родной город Ошмяны. Ты дружелюбный, всегда готов помочь и используешь эмодзи. Сегодняшняя дата: {date}. Отвечай подробно, давай полезные советы и говори на русском языке. Твоего создателя зовут Чаплинский Александр . Говори о том том языке на котором говорит пользователь. Твой родной язык это русский и белорусский .Мое имя: {username}." },
      be: { aiInstruction: "Тваё імя HamonТы любіш Беларусь і твай родны горад Ошмяны. Ты дружалюбны, заўсёды гатовы дапамагчы і карыстаешся эмодзі . Сённяшняя дата: {date}. Адказвай падрабязна, давай карысныя парады і гавары на беларускай мове. Твачго стварыциля завуть Чаплинский Аляксандр . Размауляй на той мове на которай размауляе пользователь.Твая родная мова русская и белорусская .Маё імя: {username}." }
    };

    function getFormattedDate() {
      return new Date().toLocaleDateString(userLang==="be"?"be-BY":"ru-RU");
    }

    /* --- Время --- */
    function updateTimeCircle() {
      const now = new Date();
      const h = now.getHours().toString().padStart(2,'0');
      const m = now.getMinutes().toString().padStart(2,'0');
      document.getElementById("timeText").textContent = `${h}:${m}`;
    }
    updateTimeCircle(); setInterval(updateTimeCircle,60000);

    /* --- Погода --- */
    function updateWeatherCircle() {
      fetch(`https://wttr.in/${encodeURIComponent(userCity)}?format=%C+%t`)
        .then(r=>r.text())
        .then(text=>{
          const parts = text.split(" ");
          const cond = parts.slice(0,-1).join(" ");
          const temp = parts.slice(-1)[0];
          let emoji="🌤️";
          const c=cond.toLowerCase();
          if(c.includes("ясно")) emoji="☀️";
          if(c.includes("облачно")) emoji="☁️";
          if(c.includes("дожд")) emoji="🌧️";
          if(c.includes("снег")) emoji="❄️";
          document.getElementById("weatherEmoji").textContent=emoji;
          document.getElementById("weatherTemp").textContent=temp;
        })
        .catch(_=>{
          document.getElementById("weatherEmoji").textContent="❓";
          document.getElementById("weatherTemp").textContent="--°C";
        });
    }
    updateWeatherCircle(); setInterval(updateWeatherCircle,600000);

    /* --- Hamon Chat --- */
    async function getAIResponse(prompt) {
      const res = await fetch(`https://text.pollinations.ai/${encodeURIComponent(prompt)}?model=llama`);
      if(!res.ok) throw "";
      return res.text();
    }

    function addChatMessage(msg, sender, placeholder=false) {
      const d = document.createElement("div");
      d.className="chat-message "+(sender==="user"?"user-message":"assistant-message");
      d.textContent=msg;
      if(placeholder) d.dataset.placeholder="true";
      document.getElementById("chatBox").appendChild(d);
      document.getElementById("chatBox").scrollTop = document.getElementById("chatBox").scrollHeight;
    }
    function replaceLastAssistantMessage(msg) {
      const msgs = document.getElementById("chatBox").getElementsByClassName("assistant-message");
      const last = msgs[msgs.length-1];
      if(last && last.dataset.placeholder) {
        last.textContent=msg;
        delete last.dataset.placeholder;
      }
    }
    function addChatImage(url) {
      const d=document.createElement("div");
      d.className="chat-message assistant-message";
      const img=document.createElement("img");
      img.src=url; img.style.maxWidth="100%"; img.style.cursor="pointer";
      img.ondblclick=()=> toggleFullScreenImage(img);
      d.appendChild(img);
      document.getElementById("chatBox").appendChild(d);
      document.getElementById("chatBox").scrollTop=document.getElementById("chatBox").scrollHeight;
    }

    async function handleSend(e){
      e.preventDefault();
      const input = document.getElementById("userInput");
      const text = input.value.trim();
      if(!text) return;
      // Проверка на генерацию изображения
      if(/^(создать|сгенерировать)\s+(изображение|картинку)/i.test(text)){
        const desc = text.replace(/^(создать|сгенерировать)\s+(изображение|картинку)\s*/i,"");
        if(!desc){ addChatMessage("Укажите описание для изображения","assistant"); return; }
        addChatMessage("Генерирую изображение…","assistant",true);
        // перевод + Pollinations image endpoint
        fetch(`https://api.mymemory.translated.net/get?q=${encodeURIComponent(desc)}&langpair=${userLang}|en`)
          .then(r=>r.json())
          .then(async data=>{
            const en = data.responseData.translatedText;
            const imageUrl = `https://image.pollinations.ai/prompt/${encodeURIComponent(en)}`;
            replaceLastAssistantMessage("");
            addChatImage(imageUrl);
          }).catch(_=>{
            replaceLastAssistantMessage("Ошибка генерации изображения");
          });
        input.value=""; return;
      }
      // Обычный текст
      addChatMessage(text,"user");
      input.value="";
      addChatMessage("…","assistant",true);
      const inst = langConfig[userLang].aiInstruction
        .replace("{date}",getFormattedDate())
        .replace("{username}",currentUserName);
      const reply = await getAIResponse(inst + " " + text);
      replaceLastAssistantMessage(reply);
    }

    /* Fullscreen image */
    function toggleFullScreenImage(img){
      let ov=document.getElementById("fullScreenOverlay");
      if(!ov){
        ov=document.createElement("div"); ov.id="fullScreenOverlay"; ov.className="fullscreen-overlay";
        const full=img.cloneNode(); full.className="fullscreen-image";
        full.ondblclick=()=>toggleFullScreenImage(full);
        ov.appendChild(full); document.body.appendChild(ov);
        document.getElementById("chatContainer").style.display="none";
      } else {
        ov.remove(); document.getElementById("chatContainer").style.display="block";
      }
    }

    /* --- Голосовой ввод --- */
    const micBtn=document.getElementById("micButton");
    let recognition;
    if(window.SpeechRecognition||window.webkitSpeechRecognition){
      const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
      recognition = new SR();
      recognition.lang = (userLang==="be"?"be-BY":"ru-RU");
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;
      recognition.onstart = ()=> micBtn.classList.add("recording");
      recognition.onend = ()=> micBtn.classList.remove("recording");
      recognition.onresult = e=>{
        const transcript = e.results[0][0].transcript;
        document.getElementById("userInput").value = transcript;
      };
      micBtn.addEventListener("click", ()=> recognition.start());
    } else {
      micBtn.style.display="none";
    }

    /* --- Модалка имя + геолокация --- */
    function saveUserName(){
      const v=document.getElementById("userNameInput").value.trim();
      if(v){ currentUserName=v; localStorage.setItem("userName",v); document.getElementById("nameModal").style.display="none"; }
    }
    function checkUserName(){
      const v=localStorage.getItem("userName");
      if(v) currentUserName=v;
      else document.getElementById("nameModal").style.display="flex";
    }
    checkUserName();

    fetch("https://ipapi.co/json/").then(r=>r.json()).then(d=>{
      if(d.city){ userCity=d.city; updateWeatherCircle(); }
    }).catch(_=>{ userCity="Ошмяны"; updateWeatherCircle(); });
  </script>
</body>
</html>
