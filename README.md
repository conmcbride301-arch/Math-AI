<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Connor McBride – AI Preview System</title>

<!-- Fonts -->
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Inter&display=swap" rel="stylesheet">

<style>
/* =====================
   GLOBAL STYLES
===================== */
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: Inter, sans-serif;
  background: linear-gradient(120deg,#005eff,#00ff9d,#ff0033);
  background-size: 400% 400%;
  animation: bg 12s infinite alternate;
  min-height: 100vh;
}

@keyframes bg { to { background-position: 100%; } }

/* =====================
   ADMIN BADGE
===================== */
.admin-badge {
  position: fixed;
  top: 20px;
  left: 20px;
  background: #2ecc71;
  padding: 10px 18px;
  border-radius: 20px;
  color: white;
  font-family: Orbitron, sans-serif;
  display: none;
  z-index: 1000;
}
.admin-badge.active { display: block; }

/* =====================
   LOCK SCREEN
===================== */
.lock {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.95);
  color: white;
  display: none;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  font-family: Orbitron, sans-serif;
  z-index: 999;
}
.lock.active { display: flex; }

.lock h1 {
  color: red;
  font-size: 4em;
  margin-bottom: 10px;
}
.timer {
  font-size: 3em;
  margin-top: 10px;
}

/* =====================
   CHAT UI
===================== */
.chat-container {
  max-width: 900px;
  margin: 40px auto 120px;
  background: white;
  border-radius: 20px;
  height: 75vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 50px rgba(0,0,0,.3);
}

.chat-header {
  background: #111;
  color: white;
  padding: 18px;
  text-align: center;
  font-family: Orbitron, sans-serif;
}

.chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.msg {
  padding: 12px 18px;
  border-radius: 18px;
  max-width: 80%;
  line-height: 1.6;
}

.user {
  align-self: flex-end;
  background: #005eff;
  color: white;
}

.ai {
  align-self: flex-start;
  background: #f1f1f1;
}

.system {
  align-self: center;
  background: #fff3cd;
  font-size: 14px;
}

/* =====================
   INPUT BAR
===================== */
.input-area {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  padding: 15px;
}

.input-wrap {
  max-width: 900px;
  margin: auto;
  display: flex;
  gap: 10px;
}

textarea {
  flex: 1;
  resize: none;
  padding: 12px;
  border-radius: 20px;
  border: 1px solid #ccc;
  font-family: Inter, sans-serif;
}

button {
  width: 50px;
  border-radius: 50%;
  border: none;
  background: #005eff;
  color: white;
  font-size: 20px;
  cursor: pointer;
}

button:disabled {
  background: #aaa;
  cursor: not-allowed;
}
</style>
</head>

<body>

<!-- ADMIN INDICATOR -->
<div class="admin-badge" id="adminBadge">👑 ADMIN MODE</div>

<!-- LOCK SCREEN -->
<div class="lock" id="lock">
  <h1>EXPOSED</h1>
  <p>System Locked</p>
  <div class="timer" id="timer">30:00</div>
</div>

<!-- CHAT -->
<div class="chat-container">
  <div class="chat-header">Connor’s AI – Preview Mode</div>
  <div class="chat-messages" id="log">
    <div class="msg ai">Hello Connor. This is a fully offline preview.</div>
  </div>
</div>

<!-- INPUT -->
<div class="input-area">
  <div class="input-wrap">
    <textarea id="input" rows="1" placeholder="Try /admin, /lock, /nasa"></textarea>
    <button id="send">➤</button>
  </div>
</div>

<script>
/* =====================
   STATE
===================== */
const log = document.getElementById("log");
const input = document.getElementById("input");
const send = document.getElementById("send");
const lock = document.getElementById("lock");
const timerEl = document.getElementById("timer");
const adminBadge = document.getElementById("adminBadge");

let isAdmin = false;
let lockUntil = null;

/* =====================
   HELPERS
===================== */
function addMessage(text, cls) {
  const d = document.createElement("div");
  d.className = "msg " + cls;
  d.textContent = text;
  log.appendChild(d);
  log.scrollTop = log.scrollHeight;
}

async function mockAI(text) {
  if (text.toLowerCase().includes("math")) return "Let’s solve it step by step.";
  if (text.toLowerCase().includes("hello")) return "Hey Connor 👋";
  return `Mock AI response: "${text}"`;
}

/* =====================
   NASA (PUBLIC GOV API)
===================== */
async function nasa() {
  const res = await fetch(
    "https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY"
  );
  const data = await res.json();
  addMessage("NASA APOD: " + data.title, "ai");
  addMessage(data.explanation.slice(0, 250) + "...", "ai");
}

/* =====================
   MAIN HANDLER
===================== */
async function handleSend() {
  const text = input.value.trim();
  if (!text) return;
  input.value = "";
  addMessage(text, "user");

  if (text === "/admin") {
    isAdmin = !isAdmin;
    adminBadge.classList.toggle("active", isAdmin);
    addMessage("Admin mode toggled.", "system");
    return;
  }

  if (text === "/lock" && isAdmin) {
    lockUntil = Date.now() + 30 * 60000;
    lock.classList.add("active");
    return;
  }

  if (text === "/unlock" && isAdmin) {
    lockUntil = null;
    lock.classList.remove("active");
    return;
  }

  if (text === "/nasa") {
    await nasa();
    return;
  }

  const reply = await mockAI(text);
  addMessage(reply, "ai");
}

/* =====================
   EVENTS
===================== */
send.onclick = handleSend;

input.onkeydown = e => {
  if (e.key === "Enter" && !e.shiftKey) {
    e.preventDefault();
    handleSend();
  }
};

/* =====================
   LOCK TIMER
===================== */
setInterval(() => {
  if (!lockUntil) return;
  const remaining = Math.max(0, lockUntil - Date.now());
  const m = Math.floor(remaining / 60000);
  const s = Math.floor((remaining % 60000) / 1000);
  timerEl.textContent = `${m}:${String(s).padStart(2, "0")}`;
  if (remaining === 0) lock.classList.remove("active");
}, 1000);
</script>

</body>
</html>

