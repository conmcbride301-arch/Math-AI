<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Connor Classroom Lock System</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
/* 🌈 BACKGROUND */
body {
  margin: 0;
  font-family: Inter, Arial, sans-serif;
  background: linear-gradient(120deg, #005eff, #00ff9d, #ff0033);
  background-size: 400% 400%;
  animation: bg 10s infinite alternate;
}
@keyframes bg {
  0% { background-position: 0%; }
  100% { background-position: 100%; }
}

/* 🔒 LOCK SCREEN */
.lock {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.95);
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  color: white;
}
.lock.active { display: flex; }

.lock-box {
  text-align: center;
  padding: 40px;
  border: 4px solid red;
  box-shadow: 0 0 40px red;
}

.timer {
  font-size: 3em;
  margin: 20px 0;
}

/* 🎓 BYPASS */
.bypass-title {
  font-weight: 900;
  margin-top: 15px;
}
#bypass {
  padding: 10px;
  font-size: 18px;
  margin-top: 10px;
}

/* ⌨️ TYPE AREA */
.type-area {
  max-width: 900px;
  margin: 80px auto;
  padding: 25px;
  background: rgba(255,255,255,0.95);
  border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.25);
  text-align: center;
}
.type-area h2 {
  font-weight: 900;
}
#mainInput {
  width: 100%;
  height: 220px;
  padding: 15px;
  font-size: 18px;
  border-radius: 12px;
  border: 3px solid #005eff;
  resize: none;
}
#mainInput:disabled {
  background: #eee;
  opacity: 0.6;
}
</style>
</head>

<body>

<!-- 🔒 LOCK SCREEN -->
<div class="lock" id="lock">
  <div class="lock-box">
    <h1>🔒 SYSTEM LOCKED</h1>
    <div class="timer" id="time"></div>
    <p>🎵 🤠 Country Mode Active 🤠 🎵</p>

    <div class="bypass-title"><b>BYPASS CODE</b></div>
    <input id="bypass" placeholder="Teacher only">
  </div>
</div>

<!-- ⌨️ TYPE AREA -->
<div class="type-area">
  <h2>Type Below</h2>
  <textarea id="mainInput" placeholder="Start typing here..."></textarea>
</div>

<!-- 🎵 MUSIC -->
<audio id="music" src="country.mp3" loop></audio>

<script>
/* 🔥 FIREBASE CONFIG (REPLACE THESE) */
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

let endTime = null;

/* 🌍 GLOBAL LOCK LISTENER */
ref.on("value", snap => {
  const d = snap.val();
  if (!d || !d.locked) return unlock();

  if (Date.now() < d.until) lockSite(d.until);
  else unlock();
});

/* 🔒 LOCK */
function lockSite(until) {
  endTime = until;
  lock.classList.add("active");
  inputBox.disabled = true;
  music.play().catch(()=>{});
}

/* 🔓 UNLOCK */
function unlock() {
  lock.classList.remove("active");
  inputBox.disabled = false;
  music.pause();
  music.currentTime = 0;
  endTime = null;
}

/* ⏱ TIMER */
setInterval(() => {
  if (!endTime) return;
  const t = Math.max(0, endTime - Date.now());
  const m = Math.floor(t / 60000);
  const s = Math.floor((t % 60000) / 1000);
  timer.textContent = `${m}:${s.toString().padStart(2,"0")}`;
}, 1000);

/* 🔐 HIDDEN COMMANDS */
let buffer = "";
document.addEventListener("keydown", e => {
  buffer += e.key;
  buffer = buffer.slice(-6);

  if (buffer === "/lock") {
    ref.set({ locked: true, until: Date.now() + 30 * 60 * 1000 });
  }

  if (buffer === "/unlock") {
    ref.set({ locked: false });
  }
});

/* 🎓 TEACHER BYPASS */
document.getElementById("bypass").addEventListener("keydown", e => {
  if (e.key === "Enter" && e.target.value === "202031") {
    ref.set({ locked: false });
    e.target.value = "";
  }
});
</script>

</body>
</html>
