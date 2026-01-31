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
  box-shadow: 0 4px 15px rgba(46, 204, 113, 0.4);
  animation: adminPulse 2s infinite;
}
.admin-badge.active { display: block; }
@keyframes adminPulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }

.lock {
  position: fixed; inset: 0; background: rgba(0,0,0,0.95);
  display: none; justify-content: center; align-items: center;
  z-index: 10000; color: white;
}
.lock.active { display: flex; animation: flashRed 1s infinite; }
@keyframes flashRed {
  0%, 100% { background: rgba(0,0,0,0.95); }
  50% { background: rgba(255,0,0,0.95); }
}

.lock-box {
  text-align: center; padding: 50px; border: 5px solid red;
  border-radius: 20px; box-shadow: 0 0 50px red;
  background: rgba(0,0,0,0.5); backdrop-filter: blur(10px);
}

.lock-icon {
  font-size: 8em; margin-bottom: 20px;
  animation: shake 0.5s infinite;
}
@keyframes shake {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-15deg); }
  75% { transform: rotate(15deg); }
}

.exposed-text {
  font-size: 5em; font-weight: 900; color: #ff0000;
  text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #ff0000;
  margin: 20px 0; animation: exposedGlow 1s infinite;
  letter-spacing: 10px;
}
@keyframes exposedGlow {
  0%, 100% { text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #ff0000; }
  50% { text-shadow: 0 0 20px #fff, 0 0 40px #ff0000, 0 0 80px #ff0000; }
}

.lock-box h1 { font-size: 3em; margin: 20px 0; }
.timer { font-size: 4em; margin: 30px 0; font-weight: 900; font-family: 'Courier New', monospace; }
.music-indicator { font-size: 2em; margin: 20px 0; animation: musicBounce 0.5s infinite; }
@keyframes musicBounce { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.2); } }

.bypass-section { margin-top: 30px; padding-top: 30px; border-top: 2px solid rgba(255,255,255,0.3); }
.bypass-title { font-weight: 900; font-size: 1.3em; margin-bottom: 15px; }
#bypass {
  padding: 12px 20px; font-size: 18px; border-radius: 8px;
  border: 2px solid white; background: rgba(255,255,255,0.1);
  color: white; width: 250px; text-align: center;
}

.chat-container {
  max-width: 800px; margin: 80px auto 140px;
  background: rgba(255,255,255,0.95); border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.25); overflow: hidden;
}

.chat-header {
  background: linear-gradient(135deg, #005eff, #00ff9d);
  color: white; padding: 20px; text-align: center;
}
.chat-header h2 { font-weight: 900; font-size: 2em; margin: 0; }

.chat-messages {
  height: 500px; overflow-y: auto;
  padding: 20px; background: #f8f9fa;
}

.message {
  margin-bottom: 12px; padding: 12px 18px;
  border-radius: 18px; max-width: 70%;
  animation: messageSlide 0.3s ease;
  line-height: 1.6;
}
@keyframes messageSlide {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.message.user {
  background: linear-gradient(135deg, #005eff, #0047cc);
  color: white; margin-left: auto;
  border-bottom-right-radius: 4px;
}

.message.assistant {
  background: white; color: #333;
  border: 2px solid #e0e0e0;
  margin-right: auto;
  border-bottom-left-radius: 4px;
}

.message.system {
  background: #fff3cd; color: #856404;
  border: 2px solid #ffc107;
  text-align: center; max-width: 90%;
  margin: 0 auto; font-weight: 600;
}

.message.image {
  background: white; border: 2px solid #e0e0e0;
  margin-right: auto; padding: 8px;
  max-width: 80%;
}

.message.image img {
  max-width: 100%; border-radius: 8px; display: block;
}

.chat-input-container {
  position: fixed; bottom: 0; left: 0; right: 0;
  background: white; border-top: 1px solid #e5e7eb;
  z-index: 1000; box-shadow: 0 -2px 10px rgba(0,0,0,0.05);
}

.chat-input-wrapper {
  max-width: 800px; margin: 0 auto; padding: 12px;
}

.input-container {
  position: relative; border: 1px solid #d1d5db;
  border-radius: 24px; transition: all 0.2s;
}

.input-container:focus-within {
  border-color: #005eff;
  box-shadow: 0 0 0 3px rgba(0, 94, 255, 0.1);
}

#chatInput {
  width: 100%; border: none; padding: 12px 80px 12px 16px;
  resize: none; font-size: 16px; background: transparent;
  font-family: 'Inter', Arial, sans-serif;
  max-height: 200px; min-height: 24px; line-height: 24px;
}
#chatInput:focus { outline: none; }

.button-group {
  position: absolute; right: 8px; bottom: 8px;
  display: flex; gap: 4px;
}

#sendBtn, #imageBtn {
  width: 32px; height: 32px; border-radius: 8px;
  border: none; cursor: pointer; color: white;
  display: flex; align-items: center; justify-content: center;
  font-size: 18px; transition: all 0.2s;
}

#imageBtn {
  background: #9b59b6;
}
#imageBtn:hover:not(:disabled) {
  background: #8e44ad;
}

#sendBtn {
  background: #d1d5db;
}
#sendBtn.active {
  background: #005eff;
}
#sendBtn.active:hover {
  background: #0047cc;
}
#sendBtn:disabled, #imageBtn:disabled {
  background: #e5e7eb;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .chat-container { margin: 20px 10px 140px; }
  .exposed-text { font-size: 3em; }
}

.chat-messages::-webkit-scrollbar { width: 8px; }
.chat-messages::-webkit-scrollbar-thumb { background: #888; border-radius: 4px; }
</style>
</head>

<body>

<div class="admin-badge" id="adminBadge">👑 ADMIN - Connor McBride</div>

<div class="lock" id="lock">
  <div class="lock-box">
    <div class="lock-icon">🔒</div>
    <div class="exposed-text">EXPOSED</div>
    <h1>SYSTEM LOCKED</h1>
    <div class="timer" id="time">30:00</div>
    <div class="music-indicator">🎵 🤠 Country Mode Active 🤠 🎵</div>
    <div class="bypass-section">
      <div class="bypass-title">TEACHER BYPASS CODE</div>
      <input id="bypass" type="password" placeholder="Enter code...">
    </div>
  </div>
</div>

<div class="chat-container">
  <div class="chat-header">
    <h2>🤖 Connor's AI System - Chat & Images</h2>
  </div>
  <div class="chat-messages" id="chatMessages"></div>
</div>

<div class="chat-input-container">
  <div class="chat-input-wrapper">
    <div class="input-container">
      <textarea id="chatInput" rows="1"
        placeholder="Ask anything or type /image ..."></textarea>
      <div class="button-group">
        <button id="imageBtn" title="Generate Image">🎨</button>
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
const timer = document.getElementById("time");
const music = document.getElementById("music");
const adminBadge = document.getElementById("adminBadge");
const chatMessages = document.getElementById("chatMessages");
const chatInput = document.getElementById("chatInput");
const sendBtn = document.getElementById("sendBtn");
const imageBtn = document.getElementById("imageBtn");
const bypassInput = document.getElementById("bypass");

let isAdmin = false;
let endTime = null;

music.volume = 0.7;

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
  try {
    const r = await fetch("http://localhost:3000/api/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: msg })
    });
    const j = await r.json();
    return j.reply || "No response received.";
  } catch (error) {
    console.error('AI Error:', error);
    return "⚠️ Server offline! Run: node server.js";
  }
}

async function generateImage(prompt) {
  try {
    const r = await fetch("http://localhost:3000/api/generate-image", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ prompt })
    });
    const j = await r.json();
    return j.image;
  } catch (error) {
    console.error('Image Error:', error);
    return null;
  }
}

/* ---------- SEND ---------- */
async function sendMessage() {
  const text = chatInput.value.trim();
  if (!text) return;

  // 👑 ADMIN COMMAND
  if (text === "/admin") {
    isAdmin = !isAdmin;
    adminBadge.classList.toggle("active", isAdmin);
    addMessage(`✅ Admin mode: ${isAdmin ? "ENABLED" : "DISABLED"}`, "system");
    chatInput.value = "";
    return;
  }

  // 🔒 LOCK COMMAND
  if (text === "/lock" && isAdmin) {
    ref.set({ locked: true, until: Date.now() + 30 * 60 * 1000 });
    addMessage("🔒 LOCKDOWN ACTIVATED - 30 minutes", "system");
    chatInput.value = "";
    return;
  }

  // 🔓 UNLOCK COMMAND
  if (text === "/unlock" && isAdmin) {
    ref.set({ locked: false });
    addMessage("🔓 System unlocked", "system");
    chatInput.value = "";
    return;
  }

  // 🎨 IMAGE GENERATION
  if (text.startsWith("/image ")) {
    const prompt = text.replace("/image ", "").trim();
    addMessage(`🎨 Generating: "\${prompt}"`, "system");
    chatInput.value = "";
    sendBtn.disabled = true;
    imageBtn.disabled = true;

    const img = await generateImage(prompt);
    
    if (img) {
      addMessage(img, "image");
    } else {
      addMessage("❌ Image generation failed", "system");
    }

    sendBtn.disabled = false;
    imageBtn.disabled = false;
    return;
  }

  // 💬 AI CHAT
  addMessage(text, "user");
  chatInput.value = "";
  sendBtn.disabled = true;

  const thinking = document.createElement("div");
  thinking.className = "message system";
  thinking.textContent = "🤔 

