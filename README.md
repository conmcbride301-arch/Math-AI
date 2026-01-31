<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Connor's Classroom Lock System</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
/* 🌈 ANIMATED BACKGROUND */
body {
  margin: 0;
  font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
  background: linear-gradient(120deg, #005eff, #00ff9d, #ff0033);
  background-size: 400% 400%;
  animation: bg 10s infinite alternate;
  overflow-x: hidden;
}
@keyframes bg {
  0% { background-position: 0%; }
  100% { background-position: 100%; }
}

/* 👑 ADMIN BADGE */
.admin-badge {
  position: fixed;
  top: 20px;
  left: 20px;
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  color: white;
  padding: 12px 24px;
  border-radius: 25px;
  font-weight: 700;
  font-size: 14px;
  display: none;
  z-index: 10001;
  box-shadow: 0 4px 15px rgba(46, 204, 113, 0.4);
  animation: adminPulse 2s infinite;
}
.admin-badge.active {
  display: block;
}
@keyframes adminPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

/* 🔒 LOCK SCREEN */
.lock {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.95);
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  color: white;
}
.lock.active { 
  display: flex;
  animation: flashRed 1s infinite;
}
@keyframes flashRed {
  0%, 100% { background: rgba(0,0,0,0.95); }
  50% { background: rgba(255,0,0,0.95); }
}

.lock-box {
  text-align: center;
  padding: 50px;
  border: 5px solid red;
  border-radius: 20px;
  box-shadow: 0 0 50px red;
  background: rgba(0,0,0,0.5);
  backdrop-filter: blur(10px);
}

.lock-icon {
  font-size: 8em;
  margin-bottom: 20px;
  animation: shake 0.5s infinite;
}
@keyframes shake {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-15deg); }
  75% { transform: rotate(15deg); }
}

.exposed-text {
  font-size: 5em;
  font-weight: 900;
  color: #ff0000;
  text-shadow: 
    0 0 10px #fff,
    0 0 20px #fff,
    0 0 30px #ff0000,
    0 0 40px #ff0000,
    0 0 50px #ff0000;
  margin: 20px 0;
  animation: exposedGlow 1s infinite, exposedShake 0.3s infinite;
  letter-spacing: 10px;
}
@keyframes exposedGlow {
  0%, 100% { 
    text-shadow: 
      0 0 10px #fff,
      0 0 20px #fff,
      0 0 30px #ff0000,
      0 0 40px #ff0000;
  }
  50% { 
    text-shadow: 
      0 0 20px #fff,
      0 0 30px #fff,
      0 0 40px #ff0000,
      0 0 60px #ff0000,
      0 0 80px #ff0000;
  }
}
@keyframes exposedShake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.lock-box h1 {
  font-size: 3em;
  margin: 20px 0;
  text-shadow: 0 0 20px rgba(255,255,255,0.8);
}

.timer {
  font-size: 4em;
  margin: 30px 0;
  font-weight: 900;
  font-family: 'Courier New', monospace;
  text-shadow: 0 0 20px rgba(255,255,255,0.8);
}

.music-indicator {
  font-size: 2em;
  margin: 20px 0;
  animation: musicBounce 0.5s infinite;
}
@keyframes musicBounce {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.2); }
}

/* 🎓 BYPASS */
.bypass-section {
  margin-top: 30px;
  padding-top: 30px;
  border-top: 2px solid rgba(255,255,255,0.3);
}
.bypass-title {
  font-weight: 900;
  font-size: 1.3em;
  margin-bottom: 15px;
}
#bypass {
  padding: 12px 20px;
  font-size: 18px;
  border-radius: 8px;
  border: 2px solid white;
  background: rgba(255,255,255,0.1);
  color: white;
  width: 250px;
  text-align: center;
}
#bypass::placeholder {
  color: rgba(255,255,255,0.6);
}

/* ⌨️ TYPE AREA */
.type-area {
  max-width: 900px;
  margin: 80px auto 40px;
  padding: 30px;
  background: rgba(255,255,255,0.95);
  border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.25);
  text-align: center;
}
.type-area h2 {
  font-weight: 900;
  font-size: 2em;
  margin-bottom: 10px;
  color: #333;
}
.type-area p {
  color: #666;
  margin-bottom: 20px;
  font-size: 1.1em;
}
#mainInput {
  width: 100%;
  height: 250px;
  padding: 15px;
  font-size: 18px;
  border-radius: 12px;
  border: 3px solid #005eff;
  resize: none;
  font-family: 'Inter', Arial, sans-serif;
  line-height: 1.6;
}
#mainInput:focus {
  outline: none;
  border-color: #00ff9d;
  box-shadow: 0 0 20px rgba(0, 255, 157, 0.3);
}
#mainInput:disabled {
  background: #eee;
  opacity: 0.6;
  cursor: not-allowed;
}

/* 📱 RESPONSIVE */
@media (max-width: 768px) {
  .type-area {
    margin: 40px 20px;
    padding: 20px;
  }
  .lock-box {
    padding: 30px 20px;
  }
  .exposed-text {
    font-size: 3em;
  }
  .timer {
    font-size: 2.5em;
  }
}
</style>
</head>

<body>

<!-- 👑 ADMIN BADGE -->
<div class="admin-badge" id="adminBadge">
  👑 ADMIN MODE - Connor McBride
</div>

<!-- 🔒 LOCK SCREEN -->
<div class="lock" id="lock">
  <div class="lock-box">
    <div class="lock-icon">🔒</div>
    <div class="exposed-text">EXPOSED</div>
    <h1>SYSTEM LOCKED</h1>
    <div class="timer" id="time">30:00</div>
    <div class="music-indicator">🎵 🤠 Country Mode Active 🤠 🎵</div>
    
    <div class="bypass-section">
      <div class="bypass-title">TEACHER BYPASS CODE</div>
      <input id="bypass" type="password" placeholder="Enter code..." autocomplete="off">
    </div>
  </div>
</div>

<!-- ⌨️ TYPE AREA -->
<div class="type-area">
  <h2>Connor's Typing Area</h2>
  <p>Start typing below. Admin can lock this remotely.</p>
  <textarea id="mainInput" placeholder="Start typing here..." spellcheck="true"></textarea>
</div>

<!-- 🎵 MUSIC -->
<audio id="music" loop preload="auto">
  <!-- Add your country music file here -->
  <source src="country.mp3" type="audio/mpeg">
  <!-- Fallback to online source if local file not found -->
  <source src="https://www.bensound.com/bensound-music/bensound-country.mp3" type="audio/mpeg">
</audio>

<script>
/* 🔥 FIREBASE CONFIG - REPLACE WITH YOUR OWN */
firebase.initializeApp({
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT"
});

const db = firebase.database();
const ref = db.ref("lockState");

const lock = document.getElementById("lock");
const timer = document.getElementById("time");
const music = document.getElementById("music");
const inputBox = document.getElementById("mainInput");
const adminBadge = document.getElementById("adminBadge");

let endTime = null;
let isAdmin = false;

// Lock volume at 70% and prevent changes
music.volume = 0.7;
music.addEventListener('volumechange', () => {
  if (lock.classList.contains('active')) {
    music.volume = 0.7;
  }
});

/* 🌍 GLOBAL LOCK LISTENER */
ref.on("value", snap => {
  const data = snap.val();
  
  // If admin, don't lock them
  if (isAdmin) {
    unlock();
    return;
  }
  
  if (!data || !data.locked) {
    unlock();
    return;
  }

  if (Date.now() < data.until) {
    lockSite(data.until);
  } else {
    unlock();
  }
});

/* 🔒 LOCK FUNCTION */
function lockSite(until) {
  endTime = until;
  lock.classList.add("active");
  inputBox.disabled = true;
  
  // Try to play music (may be blocked by browser)
  music.play().catch(err => {
    console.log("Autoplay blocked, user interaction needed");
  });
}

/* 🔓 UNLOCK FUNCTION */
function unlock() {
  lock.classList.remove("active");
  inputBox.disabled = false;
  music.pause();
  music.currentTime = 0;
  endTime = null;
}

/* ⏱ TIMER UPDATE */
setInterval(() => {
  if (!endTime) return;
  
  const remaining = Math.max(0, endTime - Date.now());
  const minutes = Math.floor(remaining / 60000);
  const seconds = Math.floor((remaining % 60000) / 1000);
  
  timer.textContent = `${minutes}:${seconds.toString().padStart(2, "0")}`;
  
  // Auto-unlock when time's up
  if (remaining === 0) {
    ref.set({ locked: false });
  }
}, 1000);

/* 🔐 HIDDEN COMMANDS */
let buffer = "";
document.addEventListener("keydown", e => {
  buffer += e.key;
  buffer = buffer.slice(-7); // Track last 7 characters

  // Admin mode toggle
  if (buffer.includes("/admin")) {
    isAdmin = !isAdmin;
    adminBadge.classList.toggle("active", isAdmin);
    unlock(); // Unlock admin immediately
    buffer = "";
    console.log("Admin mode:", isAdmin ? "ON" : "OFF");
  }

  // Lock command (only works if admin)
  if (buffer.includes("/lock") && isAdmin) {
    ref.set({ 
      locked: true, 
      until: Date.now() + 30 * 60 * 1000 // 30 minutes
    });
    buffer = "";
    console.log("🔒 LOCK ACTIVATED");
  }

  // Unlock command (only works if admin)
  if (buffer.includes("/unlock") && isAdmin) {
    ref.set({ locked: false });
    buffer = "";
    console.log("🔓 UNLOCKED");
  }
});

/* 🎓 TEACHER BYPASS */
document.getElementById("bypass").addEventListener("keydown", e => {
  if (e.key === "Enter") {
    const code = e.target.value.trim();
    
    // Teacher bypass code
    if (code === "202031") {
      ref.set({ locked: false });
      e.target.value = "";
      console.log("✅ Teacher bypass successful");
    } else if (code !== "") {
      e.target.value = "";
      e.target.placeholder = "❌ Wrong code";
      setTimeout(() => {
        e.target.placeholder = "Enter code...";
      }, 2000);
    }
  }
});

/* 🎵 CLICK TO ENABLE AUDIO (for browsers that block autoplay) */
document.body.addEventListener('click', () => {
  if (lock.classList.contains('active') && music.paused) {
    music.play().catch(() => {});
  }
}, { once: true });

console.log("🔥 Connor's Lock System Loaded");
console.log("Commands: /admin (toggle), /lock (admin only), /unlock (admin only)");
console.log("Teacher bypass: 202031");
</script>

</body>
</html>
