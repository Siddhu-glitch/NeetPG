<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Focus Space | Pro Study OS</title>

  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">

  <style>
    body {
      margin: 0;
      overflow-x: hidden;
      font-family: 'Poppins', sans-serif;
      min-height: 100vh;
      color: #333;
      transition: background 2s ease-in-out;
    }

    /* Dynamic Themes */
    .theme-morning { background: linear-gradient(to bottom, #a1c4fd, #c2e9fb); color: #1e3a8a; }
    .theme-afternoon { background: linear-gradient(to bottom, #4facfe, #00f2fe); color: #0c4a6e; }
    .theme-sunset { background: linear-gradient(to bottom, #ff7e5f, #feb47b, #ffecd2); color: #5c1404; }
    .theme-night { background: linear-gradient(to bottom, #0f2027, #203a43, #2c5364); color: #e2e8f0; }

    /* Glass Effect */
    .glass {
      background: rgba(255, 255, 255, 0.15);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border: 1px solid rgba(255, 255, 255, 0.2);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    }

    /* Interactive Sun/Moon */
    .celestial-body {
      width: 70px; height: 70px;
      border-radius: 50%; cursor: pointer;
      transition: all 0.5s ease;
      box-shadow: 0 0 40px rgba(255, 255, 255, 0.5);
      animation: pulse 4s infinite alternate;
    }
    .sun { background: radial-gradient(circle, #fff7b0, #ffcc00); }
    .moon { background: radial-gradient(circle, #f4f6f0, #d7d7d7); box-shadow: 0 0 40px rgba(255, 255, 255, 0.2); }

    @keyframes pulse { 0% { transform: scale(1); } 100% { transform: scale(1.05); box-shadow: 0 0 60px rgba(255, 255, 255, 0.8); } }

    /* Ocean Waves */
    .waves { position: fixed; bottom: 0; left: 0; width: 100%; height: 15vh; min-height: 100px; max-height: 150px; z-index: 0; pointer-events: none; }
    .parallax > use { animation: move-forever 25s cubic-bezier(.55,.5,.45,.5) infinite; }
    .parallax > use:nth-child(1) { animation-delay: -2s; animation-duration: 7s; fill: rgba(255,255,255,0.3); }
    .parallax > use:nth-child(2) { animation-delay: -3s; animation-duration: 10s; fill: rgba(255,255,255,0.2); }
    .parallax > use:nth-child(3) { animation-delay: -4s; animation-duration: 13s; fill: rgba(255,255,255,0.1); }
    .parallax > use:nth-child(4) { animation-delay: -5s; animation-duration: 20s; fill: rgba(255,255,255,0.4); }
    @keyframes move-forever { 0% { transform: translate3d(-90px,0,0); } 100% { transform: translate3d(85px,0,0); } }

    /* Deep Focus Fade */
    .distraction { transition: opacity 0.8s ease, filter 0.8s ease; }
    .deep-focus-on .distraction { opacity: 0 !important; pointer-events: none; filter: blur(10px); }

    /* Flashcard 3D Flip */
    .perspective { perspective: 1000px; }
    .flip-card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
    .is-flipped .flip-card-inner { transform: rotateY(180deg); }
    .flip-card-front, .flip-card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; display: flex; align-items: center; justify-content: center; text-align: center; padding: 1rem; }
    .flip-card-back { transform: rotateY(180deg); }
    
    /* Scrollbars */
    ::-webkit-scrollbar { width: 4px; }
    ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.4); border-radius: 10px; }
  </style>
</head>

<body id="body-theme" class="theme-sunset pb-24 relative">

  <div class="fixed top-6 right-6 z-50 flex flex-col items-center gap-4">
    <div class="flex flex-col items-center">
      <p class="text-[10px] uppercase tracking-widest mb-1 font-bold opacity-70">Tap</p>
      <div id="celestial" class="celestial-body sun" onclick="toggleNotification()"></div>
    </div>
    <button onclick="toggleDeepFocus()" class="glass rounded-full w-10 h-10 flex items-center justify-center hover:bg-white/30 transition shadow-lg tooltip group relative">
      <i id="focus-icon" class="fa-solid fa-eye text-sm"></i>
      <span class="absolute right-12 text-[10px] uppercase tracking-widest font-bold opacity-0 group-hover:opacity-100 transition whitespace-nowrap bg-black/40 text-white px-2 py-1 rounded">Focus Mode</span>
    </button>
  </div>

  <div id="notification-modal" class="opacity-0 pointer-events-none transition-opacity duration-300 fixed top-28 right-6 z-50 glass rounded-2xl p-5 w-64 shadow-2xl">
    <p class="text-xs uppercase tracking-widest opacity-70 mb-1">Daily Note</p>
    <h3 id="daily-reminder" class="text-md font-bold leading-tight">Keep pushing!</h3>
  </div>

  <div class="max-w-7xl mx-auto relative z-10 pt-10 px-6 md:px-10">
    
    <div class="text-center md:text-left mb-10">
      <div class="flex flex-wrap justify-center md:justify-start gap-3 mb-4 text-sm font-medium">
        <div class="glass rounded-full px-4 py-2 flex items-center">
          <i class="fa-solid fa-location-dot mr-2"></i> <span id="weather">Fetching...</span>
        </div>
        <div class="glass rounded-full px-4 py-2 flex items-center gap-2 text-orange-500 font-bold bg-white/20">
          <i class="fa-solid fa-fire"></i> <span id="streak-counter">0 Day Streak</span>
        </div>
        <div class="glass rounded-full px-4 py-2 flex items-center gap-2 font-bold bg-white/20">
          <i class="fa-solid fa-hourglass-half"></i> <span id="exam-countdown">Loading...</span>
        </div>
      </div>
      
      <h1 id="clock" class="text-6xl md:text-[8rem] font-black tracking-tighter drop-shadow-lg tabular-nums">00:00</h1>
      <p id="caption" class="text-xl md:text-3xl font-light mt-2 opacity-90 tracking-wide">Good evening, Anjali.</p>
      <p id="motivation-quote" class="text-md md:text-lg font-medium mt-3 opacity-80 max-w-2xl italic border-l-4 border-current pl-4 distraction">"Loading motivation..."</p>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 relative z-20">

      <div class="glass rounded-[2rem] p-6 flex flex-col">
        <div class="flex justify-between items-center mb-4">
          <p class="uppercase tracking-[0.3em] text-xs opacity-70 font-bold">Study Tracker</p>
          <button onclick="toggleTimerMode()" class="text-[10px] font-bold uppercase tracking-widest bg-white/20 px-3 py-1 rounded-full hover:bg-white/30 transition">
            <span id="mode-text">Switch to Pomodoro</span>
          </button>
        </div>
        
        <div class="flex justify-between items-end mb-4">
          <div>
            <p class="text-sm opacity-80 mb-1 font-semibold">Total Today</p>
            <h2 id="total-time" class="text-4xl font-black tabular-nums leading-none">00:00:00</h2>
          </div>
          <div class="text-right">
            <p id="session-label" class="text-sm opacity-80 mb-1 font-semibold">Stopwatch</p>
            <h2 id="session-time" class="text-2xl font-bold tabular-nums opacity-90 leading-none">00:00</h2>
          </div>
        </div>

        <div class="flex gap-2 mb-6">
          <button id="start-btn" onclick="startTimer()" class="flex-1 bg-white/20 hover:bg-white/30 backdrop-blur-md border border-white/40 px-4 py-3 rounded-2xl font-bold transition-all text-sm"><i class="fa-solid fa-play mr-1"></i> Start</button>
          <button id="stop-btn" onclick="pauseTimer()" class="flex-1 bg-red-500/50 hover:bg-red-500/70 text-white backdrop-blur-md border border-white/40 px-4 py-3 rounded-2xl font-bold transition-all text-sm hidden"><i class="fa-solid fa-pause mr-1"></i> Pause</button>
          <button onclick="resetSession()" class="bg-black/10 hover:bg-black/20 backdrop-blur-md border border-white/20 px-4 py-3 rounded-2xl font-bold transition-all text-sm"><i class="fa-solid fa-rotate-right"></i></button>
        </div>

        <div class="mt-auto">
          <p class="text-[10px] uppercase tracking-widest opacity-60 font-bold mb-2">Past 7 Days</p>
          <div id="chart-container" class="flex items-end gap-2 h-16 w-full border-b border-white/20 pb-1">
            </div>
        </div>
      </div>

      <div class="glass rounded-[2rem] p-6 flex flex-col max-h-[450px] distraction">
        
        <div class="mb-5 bg-white/10 p-3 rounded-2xl border border-white/20 flex justify-between items-center">
          <p class="font-bold text-sm">Hydration 💧</p>
          <div class="flex gap-1 px-2" id="water-grid"></div>
        </div>

        <p class="uppercase tracking-[0.3em] text-xs opacity-70 font-bold mb-3">Daily Targets</p>
        <form onsubmit="addTodo(event)" class="flex gap-2 mb-3">
          <input id="todo-input" type="text" autocomplete="off" class="flex-1 bg-white/20 border border-white/30 p-2 rounded-xl text-current placeholder-current/60 outline-none text-sm font-medium" placeholder="Add target..." />
          <button type="submit" class="bg-white/30 hover:bg-white/40 px-4 rounded-xl font-bold transition"><i class="fa-solid fa-plus"></i></button>
        </form>
        <ul id="todo-list" class="space-y-2 overflow-y-auto todo-container flex-1 pr-2"></ul>
      </div>

    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-6 relative z-20 mt-6 distraction">
      
      <div class="lg:col-span-1 space-y-6 flex flex-col">
        <div class="glass rounded-[2rem] p-5">
          <p class="uppercase tracking-[0.3em] text-xs opacity-70 font-bold mb-3">Focus Audio</p>
          <div class="space-y-2">
            <div class="bg-white/10 p-2 rounded-xl border border-white/20">
              <p class="font-bold text-[10px] mb-1">Ocean Vibes</p>
              <audio controls class="w-full h-7 outline-none opacity-80"><source src="audio1.mp3" type="audio/mpeg"></audio>
            </div>
            <div class="bg-white/10 p-2 rounded-xl border border-white/20">
              <p class="font-bold text-[10px] mb-1">Lo-Fi</p>
              <audio controls class="w-full h-7 outline-none opacity-80"><source src="audio2.mp3" type="audio/mpeg"></audio>
            </div>
          </div>
        </div>

        <div class="glass rounded-[2rem] p-5 flex-1 flex flex-col">
          <p class="uppercase tracking-[0.3em] text-xs opacity-70 font-bold mb-2 flex items-center justify-between">
            Brain Dump <i class="fa-solid fa-pen-to-square"></i>
          </p>
          <textarea id="scratchpad" oninput="saveScratchpad()" class="w-full flex-1 bg-transparent border-none outline-none resize-none text-sm font-medium placeholder-current/50 mt-1" placeholder="Type random thoughts, mnemonics, or things to buy here..."></textarea>
        </div>
      </div>

      <div class="lg:col-span-3 glass rounded-[2rem] p-6">
        <div class="flex justify-between items-center mb-4">
          <p class="uppercase tracking-[0.3em] text-xs opacity-70 font-bold">Quick-Flip Medical Facts 📇</p>
          <span class="text-[10px] font-bold opacity-60">Tap to flip</span>
        </div>

        <form onsubmit="addFlashcard(event)" class="flex flex-col md:flex-row gap-2 mb-4">
          <input id="fc-front" type="text" class="flex-1 bg-white/20 border border-white/30 p-2 rounded-xl outline-none text-sm font-medium placeholder-current/60" placeholder="Concept (Front)" required />
          <input id="fc-back" type="text" class="flex-1 bg-white/20 border border-white/30 p-2 rounded-xl outline-none text-sm font-medium placeholder-current/60" placeholder="Answer (Back)" required />
          <button type="submit" class="bg-white/30 hover:bg-white/40 px-4 rounded-xl font-bold transition text-sm">Add Card</button>
        </form>

        <div id="flashcard-grid" class="grid grid-cols-2 md:grid-cols-3 gap-3 overflow-y-auto max-h-[300px] todo-container pr-2">
          </div>
      </div>

    </div>
  </div>

  <svg class="waves" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 24 150 28" preserveAspectRatio="none">
    <defs><path id="gentle-wave" d="M-160 44c30 0 58-18 88-18s 58 18 88 18 58-18 88-18 58 18 88 18 v44h-352z" /></defs>
    <g class="parallax">
      <use xlink:href="#gentle-wave" x="48" y="0" />
      <use xlink:href="#gentle-wave" x="48" y="3" />
      <use xlink:href="#gentle-wave" x="48" y="5" />
      <use xlink:href="#gentle-wave" x="48" y="7" />
    </g>
  </svg>

  <script>
    // --- 1. NEET PG COUNTDOWN ---
    const examDate = new Date("2026-08-15T00:00:00").getTime();
    function updateCountdown() {
      const now = new Date().getTime();
      const distance = examDate - now;
      if (distance < 0) { document.getElementById('exam-countdown').innerText = "Exam Day!"; return; }
      const days = Math.floor(distance / (1000 * 60 * 60 * 24));
      const hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      const minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
      document.getElementById('exam-countdown').innerText = `${days}d ${hours}h ${minutes}m to NEET PG`;
    }
    setInterval(updateCountdown, 60000); updateCountdown();

    // --- 2. 7-DAY STUDY GRAPH ---
    let dailyHistory = JSON.parse(localStorage.getItem('studyHistory')) || {};
    
    function renderGraph() {
      const container = document.getElementById('chart-container');
      container.innerHTML = '';
      const today = new Date();
      
      // Calculate max time for scaling (min 4 hours to avoid giant bars for 5 mins)
      let maxSeconds = 14400; // 4 hours baseline
      for (let i = 6; i >= 0; i--) {
        let d = new Date(today); d.setDate(today.getDate() - i);
        let dateStr = d.toLocaleDateString('en-IN');
        if (dailyHistory[dateStr] > maxSeconds) maxSeconds = dailyHistory[dateStr];
      }

      for (let i = 6; i >= 0; i--) {
        let d = new Date(today); d.setDate(today.getDate() - i);
        let dateStr = d.toLocaleDateString('en-IN');
        let dayLabel = d.toLocaleDateString('en-US', { weekday: 'short' }).charAt(0);
        let seconds = (i === 0) ? totalSeconds : (dailyHistory[dateStr] || 0); // Today is live
        
        let heightPercent = Math.min((seconds / maxSeconds) * 100, 100);
        
        const barWrap = document.createElement('div');
        barWrap.className = "flex-1 flex flex-col items-center justify-end h-full gap-1 group relative";
        barWrap.innerHTML = `
          <div class="absolute -top-6 text-[9px] font-bold opacity-0 group-hover:opacity-100 transition bg-black/50 text-white px-1 rounded">${(seconds/3600).toFixed(1)}h</div>
          <div class="w-full bg-white/40 rounded-t-sm transition-all duration-500 hover:bg-white/60" style="height: ${Math.max(heightPercent, 2)}%"></div>
          <span class="text-[9px] font-bold opacity-60">${dayLabel}</span>
        `;
        container.appendChild(barWrap);
      }
    }

    // --- 3. SCRATCHPAD (Brain Dump) ---
    const scratchpad = document.getElementById('scratchpad');
    scratchpad.value = localStorage.getItem('scratchpadText') || '';
    function saveScratchpad() { localStorage.setItem('scratchpadText', scratchpad.value); }

    // --- 4. EYE-CARE & POSTURE GUARD (20-20-20 Rule) ---
    setInterval(() => {
      const modal = document.getElementById('notification-modal');
      document.getElementById('daily-reminder').innerText = "👀 Look 20 feet away for 20 seconds. Fix your posture!";
      modal.classList.replace('opacity-0', 'opacity-100');
      setTimeout(() => modal.classList.replace('opacity-100', 'opacity-0'), 8000);
    }, 20 * 60 * 1000); // 20 minutes

    // --- 5. DEEP FOCUS (Zen Mode) ---
    let isDeepFocus = false;
    function toggleDeepFocus() {
      isDeepFocus = !isDeepFocus;
      const icon = document.getElementById('focus-icon');
      if (isDeepFocus) {
        document.body.classList.add('deep-focus-on');
        icon.classList.replace('fa-eye', 'fa-eye-slash');
      } else {
        document.body.classList.remove('deep-focus-on');
        icon.classList.replace('fa-eye-slash', 'fa-eye');
      }
    }

    // --- CORE LOGIC (Timer, Reset, Theme, Weather) ---
    let totalSeconds = parseInt(localStorage.getItem('totalSeconds')) || 0;
    let timerInterval = null;
    let isPomodoro = false; let sessionSeconds = 0; let pomodoroState = 'work';
    let streak = parseInt(localStorage.getItem('studyStreak')) || 0;
    let lastStreakDate = localStorage.getItem('lastStreakDate');

    function checkDateReset() {
      const todayStr = new Date().toLocaleDateString('en-IN', { timeZone: 'Asia/Kolkata' });
      const savedDate = localStorage.getItem('studyDate');
      
      if (savedDate !== todayStr) {
        if(savedDate) dailyHistory[savedDate] = totalSeconds; // Save yesterday's final time
        localStorage.setItem('studyHistory', JSON.stringify(dailyHistory));
        
        localStorage.setItem('studyDate', todayStr);
        localStorage.setItem('totalSeconds', 0);
        localStorage.setItem('todos', JSON.stringify([]));
        localStorage.setItem('waterCount', 0);
        
        totalSeconds = 0; todos = []; waterCount = 0;
        updateTotalTimeDisplay(); renderTodos(); renderWater(); renderGraph();

        let yesterday = new Date(); yesterday.setDate(yesterday.getDate() - 1);
        const yStr = yesterday.toLocaleDateString('en-IN', { timeZone: 'Asia/Kolkata' });
        if (lastStreakDate !== todayStr && lastStreakDate !== yStr) {
          streak = 0; localStorage.setItem('studyStreak', streak); updateStreakUI();
        }
      }
    }

    function updateStreakUI() {
      const counter = document.getElementById('streak-counter');
      counter.innerText = `${streak} Day Streak`;
      counter.classList.toggle('opacity-70', streak === 0);
    }
    updateStreakUI();

    function formatTime(sec, hr = true) {
      const h = Math.floor(sec / 3600).toString().padStart(2, '0');
      const m = Math.floor((sec % 3600) / 60).toString().padStart(2, '0');
      const s = (sec % 60).toString().padStart(2, '0');
      return hr ? `${h}:${m}:${s}` : `${m}:${s}`;
    }
    function updateTotalTimeDisplay() { document.getElementById('total-time').innerText = formatTime(totalSeconds); renderGraph(); }
    function updateSessionTimeDisplay() { document.getElementById('session-time').innerText = formatTime(sessionSeconds, !isPomodoro); }
    updateTotalTimeDisplay(); updateSessionTimeDisplay();

    function toggleTimerMode() {
      pauseTimer();
      isPomodoro = !isPomodoro;
      document.getElementById('mode-text').innerText = isPomodoro ? "Switch to Stopwatch" : "Switch to Pomodoro";
      document.getElementById('session-label').innerText = isPomodoro ? "Pomodoro (Work)" : "Stopwatch";
      sessionSeconds = isPomodoro ? 25 * 60 : 0;
      updateSessionTimeDisplay();
    }

    function startTimer() {
      document.getElementById('start-btn').classList.add('hidden');
      document.getElementById('stop-btn').classList.remove('hidden');
      timerInterval = setInterval(() => {
        if (!isPomodoro) { sessionSeconds++; totalSeconds++; } 
        else {
          sessionSeconds--;
          if (pomodoroState === 'work') totalSeconds++;
          if (sessionSeconds <= 0) {
            if (pomodoroState === 'work') {
              alert("Take a 5-minute break.");
              pomodoroState = 'break'; document.getElementById('session-label').innerText = "Pomodoro (Break)"; sessionSeconds = 5 * 60;
            } else {
              alert("Ready to focus?");
              pomodoroState = 'work'; document.getElementById('session-label').innerText = "Pomodoro (Work)"; sessionSeconds = 25 * 60; pauseTimer();
            }
          }
        }
        
        // Streak Logic (2h)
        if (totalSeconds === 7200) {
          const todayStr = new Date().toLocaleDateString('en-IN');
          if(lastStreakDate !== todayStr) {
             streak++; lastStreakDate = todayStr;
             localStorage.setItem('studyStreak', streak); localStorage.setItem('lastStreakDate', todayStr);
             updateStreakUI();
             const modal = document.getElementById('notification-modal');
             document.getElementById('daily-reminder').innerText = "🔥 2 Hours hit! Streak increased!";
             modal.classList.replace('opacity-0', 'opacity-100'); setTimeout(() => modal.classList.replace('opacity-100', 'opacity-0'), 4000);
          }
        }
        localStorage.setItem('totalSeconds', totalSeconds);
        updateTotalTimeDisplay(); updateSessionTimeDisplay();
      }, 1000);
    }

    function pauseTimer() { clearInterval(timerInterval); document.getElementById('stop-btn').classList.add('hidden'); document.getElementById('start-btn').classList.remove('hidden'); }
    function resetSession() { pauseTimer(); if (isPomodoro) { pomodoroState = 'work'; document.getElementById('session-label').innerText = "Pomodoro (Work)"; sessionSeconds = 25 * 60; } else { sessionSeconds = 0; } updateSessionTimeDisplay(); }

    function updateClockAndTheme() {
      const timeString = new Date().toLocaleTimeString('en-IN', { timeZone: 'Asia/Kolkata', hour: '2-digit', minute: '2-digit', hour12: false });
      document.getElementById('clock').innerText = timeString;
      const hour = parseInt(timeString.split(':')[0]);
      const body = document.getElementById('body-theme'); const celestial = document.getElementById('celestial');
      
      body.className = `pb-24 relative transition-all duration-1000 ${isDeepFocus ? 'deep-focus-on' : ''}`;
      
      if (hour >= 5 && hour < 12) { body.classList.add('theme-morning'); celestial.className = 'celestial-body sun'; }
      else if (hour >= 12 && hour < 16) { body.classList.add('theme-afternoon'); celestial.className = 'celestial-body sun'; }
      else if (hour >= 16 && hour < 19) { body.classList.add('theme-sunset'); celestial.className = 'celestial-body sun'; }
      else { body.classList.add('theme-night'); celestial.className = 'celestial-body moon'; }
      
      checkDateReset();
    }
    setInterval(updateClockAndTheme, 1000); updateClockAndTheme();

    async function fetchWeather() {
      try {
        const res = await fetch("https://api.open-meteo.com/v1/forecast?latitude=16.30&longitude=80.43&current_weather=true");
        const data = await res.json();
        document.getElementById('weather').innerText = `${data.current_weather.temperature}°C in Guntur`;
      } catch (e) { document.getElementById('weather').innerText = `Guntur`; }
    }
    fetchWeather(); setInterval(fetchWeather, 3600000);

    function toggleNotification() {
      const modal = document.getElementById('notification-modal');
      document.getElementById('daily-reminder').innerText = "You're doing great. Keep going!";
      modal.classList.replace('opacity-0', 'opacity-100');
      setTimeout(() => modal.classList.replace('opacity-100', 'opacity-0'), 4000);
    }

    // Hydration, Todos, Flashcards boilerplate
    let waterCount = parseInt(localStorage.getItem('waterCount')) || 0;
    function renderWater() {
      const g = document.getElementById('water-grid'); g.innerHTML = '';
      for (let i = 1; i <= 8; i++) {
        g.innerHTML += `<i class="fa-solid fa-droplet text-xl cursor-pointer transition-all hover:scale-110 ${i <= waterCount ? 'text-blue-300 drop-shadow-[0_0_8px_rgba(147,197,253,0.8)]' : 'text-white/20'}" onclick="updateWater(${i})"></i>`;
      }
    }
    function updateWater(n) { waterCount = n; localStorage.setItem('waterCount', n); renderWater(); } renderWater();

    let todos = JSON.parse(localStorage.getItem('todos')) || [];
    function renderTodos() {
      const l = document.getElementById('todo-list'); l.innerHTML = '';
      todos.forEach((t, i) => {
        l.innerHTML += `<li class="flex justify-between items-center bg-white/10 p-2 rounded-xl border border-white/20 transition ${t.done ? 'opacity-50' : ''}">
          <div class="flex items-center gap-2 overflow-hidden"><input type="checkbox" ${t.done ? 'checked' : ''} onchange="toggleTodo(${i})" class="w-4 h-4 cursor-pointer"><span class="text-[13px] font-medium truncate ${t.done ? 'line-through' : ''}">${t.text}</span></div>
          <button onclick="deleteTodo(${i})" class="text-xs opacity-60 hover:opacity-100"><i class="fa-solid fa-xmark"></i></button></li>`;
      });
      localStorage.setItem('todos', JSON.stringify(todos));
    }
    function addTodo(e) { e.preventDefault(); const i = document.getElementById('todo-input'); if(i.value.trim()) { todos.push({text: i.value.trim(), done: false}); i.value=''; renderTodos(); } }
    function toggleTodo(i) { todos[i].done = !todos[i].done; renderTodos(); }
    function deleteTodo(i) { todos.splice(i, 1); renderTodos(); } renderTodos();

    let flashcards = JSON.parse(localStorage.getItem('flashcards')) || [{ front: "Torsades de pointes treatment", back: "Magnesium Sulfate" }];
    function renderFlashcards() {
      const g = document.getElementById('flashcard-grid'); g.innerHTML = '';
      flashcards.forEach((c, i) => {
        g.innerHTML += `<div class="perspective h-24"><div class="flip-card-inner cursor-pointer" onclick="this.parentElement.classList.toggle('is-flipped')">
          <div class="flip-card-front bg-white/10 border border-white/20 rounded-xl shadow-lg"><p class="font-semibold text-[11px] leading-tight">${c.front}</p></div>
          <div class="flip-card-back bg-white/30 border border-white/40 rounded-xl shadow-lg backdrop-blur-md relative"><p class="font-bold text-[11px] leading-tight">${c.back}</p><button onclick="event.stopPropagation(); deleteFlashcard(${i})" class="absolute top-1 right-1 text-[10px] opacity-50 hover:opacity-100"><i class="fa-solid fa-trash"></i></button></div>
        </div></div>`;
      });
      localStorage.setItem('flashcards', JSON.stringify(flashcards));
    }
    function addFlashcard(e) { e.preventDefault(); const f = document.getElementById('fc-front'); const b = document.getElementById('fc-back'); flashcards.push({front: f.value, back: b.value}); f.value=''; b.value=''; renderFlashcards(); }
    function deleteFlashcard(i) { flashcards.splice(i, 1); renderFlashcards(); } renderFlashcards();
  </script>
</body>
</html>
