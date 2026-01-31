<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Connor Lock System</title>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
body {
  margin: 0;
  font-family: Arial;
  background: linear-gradient(120deg, #005eff, #00ff9d, #ff0033);
  background-size: 400% 400%;
  animation: bg 10s infinite alternate;
}
@keyframes bg {
  0% {background-position: 0%}
  100% {background-position: 100%}
}

.lock {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.95);
  display: none;
  justify-content: center;
  align-items: center;
  color: white;
  z-index: 9999;
}

.lock.active { display: flex; }

.box {
  text-align: center;
  padding: 40px;
  border: 4px solid red;
}

.timer { font-size: 3em; margin: 20px 0; }

input {
  padding: 10px;
  font-size: 18px;
}
</style>
</head>

<body>

<div class="lock" id="lock">
  <div class="box">
    <h1>🔒 SYSTEM LOCKED</h1>
    <div class="timer" id="time"></div>

    <h2><b>BYPASS CODE</b></h2>
    <input id="bypass" placeholder="Teacher only">

    <p>🎵 🤠 Country Mode Active 🤠 🎵</p>
  </div>
</div>

<audio id="music" src="country.mp3" loop></audio>

<script>
/* 🔥 FIREBASE CONFIG — REPLACE WITH YOURS */
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

let endTime = null;

/* 🌍 GLOBAL LISTENER */
ref.on("value", snap => {
  const d = snap.val();
  if (!d || !d.locked) return unlock();

  if (Date.now() < d.until) lockSite(d.until);
  else unlock();
});

function lockSite(until) {
  endTime = until;
  lock.classList.add("active");
  music.play().catch(()=>{});
}

function unlock() {
  lock.classList.remove("active");
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
    ref.set({
      locked: true,
      until: Date.now() + 30 * 60 * 1000
    });
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
