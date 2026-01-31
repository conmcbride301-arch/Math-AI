<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Connor McBride's Ultimate AI System</title>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
  background: linear-gradient(120deg, #005eff, #00ff9d, #ff0033);
  background-size: 400% 400%;
  animation: bg 10s infinite alternate;
  overflow-x: hidden;
  padding-bottom: 100px;
}
@keyframes bg {
  0% { background-position: 0%; }
  100% { background-position: 100%; }
}

/* --- ADMIN / LOCK UI --- */
.admin-badge {
  position: fixed; top: 20px; left: 20px;
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  color: white; padding: 12px 24px; border-radius: 25px;
  font-weight: 700; font-size: 14px; display: none; z-index: 10001;
}
.admin-badge.active { display: block; }

.lock {
  position: fixed; inset: 0; background: rgba(0,0,0,0.95);
  display: none; justify-content: center; align-items: center;
  z-index: 10000; color: white;
}
.lock.active { display: flex; }

.lock-box {
  text-align: center; padding: 50px; border: 5px solid red;
  border-radius: 20px; box-shadow: 0 0 50px red;
  background: rgba(0,0,0,0.5);
}

.exposed-text {
  font-size: 4em; font-weight: 900; color: red;
  margin: 20px 0;
}

.chat-container {
  max-width: 800px; margin: 80px auto 140px;
  background: rgba(255,255,255,0.95); border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.25);
}

.chat-header {
  background: linear-gradient(135deg, #005eff, #00ff9d);
  color: white; padding: 20px; text-align: center;
}

.chat-messages {
  height: 500px; overflow-y: auto;
  padding: 20px; background: #f8f9fa;
}

.message {
  margin-bottom: 12px; padding: 12px 18px;
  border-radius: 18px; max-width: 70%;
}

.message.user {
  background: #005eff; color: white;
  margin-left: auto;
}

.message.assistant {
  background: white; border: 1px solid #ddd;
}

.message.system {
  background: #fff3cd; border: 1px solid #ffc107;
  text-align: center; max-width: 90%;
}

.message.image img {
  max-width: 100%; border-radius: 8px;
}

.chat-input-container {
  position: fixed; bottom: 0; left: 0; right: 0;
  background: white; border-top: 1px solid #e5e7eb;
}

.chat-input-wrapper {
  max-width: 800px; margin: 0 auto; padding: 12px;
}

.input-container {
  position: relative; border: 1px solid #ccc;
  border-radius: 24px;
}

#chatInput {
  width: 100%; border: none; padding: 12px 80px 12px 16px;
  resize: none; font-size: 16px;
}

.button-group {
  position: absolute; right: 8px; bottom: 8px;
  display: flex; gap: 6px;
}

#sendBtn, #imageBtn {
  width: 32px; height: 32px; border-radius: 8px;
  border: none; cursor: pointer;
}
</style>
</head>

<body>

<div class="admin-badge" id="adminBadge">👑 ADMIN</div>

<div class="lock" id="lock">
  <div class="lock-box">
    <div class="exposed-text">EXPOSED</div>
    <h1>SYSTEM LOCKED</h1>
    <div id="time">30:00</div>
    <input id="bypass" placeholder="Enter code">
  </div>
</div>

<div class="chat-container">
  <div class="chat-header">
    <h2>🤖 Connor's AI System</h2>
  </div>
  <div class="chat-messages" id="chatMessages"></div>
</div>

<div class="chat-input-container">
  <div class="chat-input-wrapper">
    <div class="input-container">
      <textarea id="chatInput" rows="1"
        placeholder="Ask anything or type /image ..."></textarea>
      <div class="button-group">
        <button id="imageBtn">🎨</button>
        <button id="sendBtn" disabled>➤</button>
      </div>
    </div>
  </div>
</div>

<audio id="music" loop>
  <source src="country.mp3">
</audio>

<script>
firebase.initializeApp({
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT"
});

const db = firebase.database();
const ref = db.ref("lockState");

const lock = document.getElementById("lock");
const adminBadge = document.getElementById("adminBadge");
const chatMessages = document.getElementById("chatMessages");
const chatInput = document.getElementById("chatInput");
const sendBtn = document.getElementById("sendBtn");
const imageBtn = document.getElementById("imageBtn");

let isAdmin = false;

/* ---------- UI ---------- */
function addMessage(text, type) {
  const div = document.createElement("div");
  div.className = `message ${type}`;

  if (type === "image") {
    const img = document.createElement("img");
    img.src = text;
    div.appendChild(img);
  } else {
    div.textContent = text;
  }

  chatMessages.appendChild(div);
  chatMessages.scrollTop = chatMessages.scrollHeight;
}

/* ---------- AI ---------- */
async function getAIResponse(msg) {
  const r = await fetch("http://localhost:3000/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message: msg })
  });
  const j = await r.json();
  return j.reply;
}

async function generateImage(prompt) {
  const r = await fetch("http://localhost:3000/api/generate-image", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ prompt })
  });
  const j = await r.json();
  return j.image;
}

/* ---------- SEND ---------- */
async function sendMessage() {
  const text = chatInput.value.trim();
  if (!text) return;

  // Admin
  if (text === "/admin") {
    isAdmin = !isAdmin;
    adminBadge.classList.toggle("active", isAdmin);
    addMessage(`Admin ${isAdmin ? "ON" : "OFF"}`, "system");
    chatInput.value = "";
    return;
  }

  // Image
  if (text.startsWith("/image ")) {
    const prompt = text.replace("/image ", "");
    addMessage("🎨 Generating image...", "system");
    chatInput.value = "";
    sendBtn.disabled = true;

    const img = await generateImage(prompt);
    addMessage(img, "image");

    sendBtn.disabled = false;
    return;
  }

  // Chat
  addMessage(text, "user");
  chatInput.value = "";
  sendBtn.disabled = true;

  const thinking = document.createElement("div");
  thinking.className = "message system";
  thinking.textContent = "🤔 Thinking...";
  chatMessages.appendChild(thinking);

  const reply = await getAIResponse(text);
  chatMessages.removeChild(thinking);
  addMessage(reply, "assistant");

  sendBtn.disabled = false;
}

/* ---------- EVENTS ---------- */
sendBtn.addEventListener("click", sendMessage);

chatInput.addEventListener("input", () => {
  sendBtn.disabled = chatInput.value.trim().length === 0;
});

chatInput.addEventListener("keydown", e => {
  if (e.key === "Enter" && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
});

imageBtn.addEventListener("click", () => {
  chatInput.value = "/image ";
  chatInput.focus();
});
</script>

</body>
</html>
