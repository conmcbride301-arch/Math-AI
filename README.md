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
  padding-bottom: 100px;
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

/* 💬 CHAT AREA */
.chat-container {
  max-width: 900px;
  margin: 80px auto 120px;
  background: rgba(255,255,255,0.95);
  border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.25);
  overflow: hidden;
}

.chat-header {
  background: linear-gradient(135deg, #005eff, #00ff9d);
  color: white;
  padding: 20px 30px;
  text-align: center;
}

.chat-header h2 {
  font-weight: 900;
  font-size: 2em;
  margin: 0;
}

.chat-messages {
  height: 500px;
  overflow-y: auto;
  padding: 20px;
  background: #f8f9fa;
}

.message {
  margin-bottom: 15px;
  padding: 12px 18px;
  border-radius: 18px;
  max-width: 70%;
  word-wrap: break-word;
  animation: messageSlide 0.3s ease;
}

@keyframes messageSlide {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message.user {
  background: linear-gradient(135deg, #005eff, #0047cc);
  color: white;
  margin-left: auto;
  border-bottom-right-radius: 4px;
}

.message.assistant {
  background: white;
  color: #333;
  border: 2px solid #e0e0e0;
  margin-right: auto;
  border-bottom-left-radius: 4px;
}

.message.system {
  background: #fff3cd;
  color: #856404;
  border: 2px solid #ffc107;
  margin: 0 auto;
  text-align: center;
  font-weight: 600;
  max-width: 90%;
}

/* 💬 CHATGPT-STYLE INPUT BAR */
.chat-input-container {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(255,255,255,0.98);
  backdrop-filter: blur(10px);
  border-top: 2px solid #e0e0e0;
  padding: 15px;
  box-shadow: 0 -5px 20px rgba(0,0,0,0.1);
  z-index: 1000;
}

.chat-input-wrapper {
  max-width: 900px;
  margin: 0 auto;
  display: flex;
  gap: 10px;
  align-items: center;
}

#chatInput {
  flex: 1;
  padding: 15px 20px;
  font-size: 16px;
  border: 2px solid #d0d0d0;
  border-radius: 25px;
  background: white;
  font-family: 'Inter', Arial, sans-serif;
  resize: none;
  max-height: 150px;
  transition: all 0.3s ease;
}

#chatInput:focus {
  outline: none;
  border-color: #005eff;
  box-shadow: 0 0 0 3px rgba(0, 94, 255, 0.1);
}

#chatInput:disabled {
  background: #f5f5f5;
  cursor: not-allowed;
  opacity: 0.6;
}

#sendBtn {
  background: linear-gradient(135deg, #005eff, #0047cc);
  color: white;
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 94, 255, 0.3);
}

#sendBtn:hover:not(:disabled) {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(0, 94, 255, 0.5);
}

#sendBtn:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

#sendBtn:active:not(:disabled) {
  transform: scale(0.95);
}

/* 📱 RESPONSIVE */
@media (max-width: 768px) {
  .chat-container {
    margin: 40px 10px 120px;
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
  .chat-input-wrapper {
    padding: 0 10px;
  }
  #chatInput {
    font-size: 14px;
    padding: 12px 16px;
  }
  #sendBtn {
    width: 45px;
    height: 45px;
    font-size: 18px;
  }
}

/* Scrollbar styling */
.chat-messages::-webkit-scrollbar {
  width: 8px;
}

.chat-messages::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.chat-messages::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}

.chat-messages::-webkit-scrollbar-thumb:hover {
  background: #555;
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

<!-- 💬 CHAT AREA -->
<div class="chat-container">
  <div class="chat-header">
    <h2>💬 Connor's Chat System</h2>
  </div>
  <div class="chat-messages" id="chatMessages">
    <div class="message system">
      🔥 System loaded. Type /admin to enable admin mode.
    </div>
  </div>
</div>

<!-- 💬 CHATGPT-STYLE INPUT BAR -->
<div class="chat-input-container">
  <div class="chat-input-wrapper">
    <textarea 
      id="chatInput" 
      placeholder="Message Connor's system..." 
      rows="1"
      autocomplete="off"
    ></textarea>
    <button id="sendBtn">➤</button>
  </div>
</div>

<!-- 🎵 MUSIC -->
<audio id="music" loop preload="auto">
  <source src="country.mp3" type="audio/mpeg">
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
const adminBadge = document.getElementById("adminBadge");
const chatMessages = document.getElementById("chatMessages");
const chatInput = document.getElementById("chatInput");
const sendBtn = document.getElementById("sendBtn");

let endTime = null;
let isAdmin = false;

// Lock volume at 70% and prevent changes
music.volume = 0.7;
music.addEventListener('volumechange', () => {
  if (lock.classList.contains('active')) {
    music.volume = 0.7;
  }
});

/* 💬 CHAT FUNCTIONS */
function addMessage(text, type = 'user') {
  const messageDiv = document.createElement('div');
  messageDiv.className = `message \${type}`;
  messageDiv.textContent = text;
  chatMessages.appendChild(messageDiv);
  chatMessages.scrollTop = chatMessages.scrollHeight;
}

function handleCommand(command) {
  const cmd = command.toLowerCase().trim();
  
  if (cmd === '/admin') {
    isAdmin = !isAdmin;
    adminBadge.classList
