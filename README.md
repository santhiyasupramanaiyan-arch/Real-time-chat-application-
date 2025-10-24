# Real-time-chat-application-
step1:Install Node.js (v14 or later).

macOS / Linux: use your package manager or download from nodejs.org

Windows: download installer from nodejs.org

   A text editor (VS Code, Notepad, etc.) and a terminal/command prompt.

Files to create

 Create a folder, e.g. realtime-        chat. Inside it create two files:

index.html — your client (use the client HTML you pasted; I’ll include a cleaned version below).

server.js — the Node/WebSocket server (pick option A or B).

Minimal ws server (fastest)

1. Open terminal in realtime-chat.

2. Initialize and install ws:


npm init -y
npm install ws

3. Create server.js with this content:


// server.js - minimal WebSocket broadcast server using 'ws'
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 3000 }, () => {
  console.log('WebSocket server listening on ws://localhost:3000');
});

wss.on('connection', (ws) => {
  console.log('Client connected');

  ws.on('message', (data) => {
    // Assume JSON { name, text }
    let msg;
    try { msg = JSON.parse(data); } catch (e) { return; }

    // Broadcast to everyone
    const out = JSON.stringify(msg);
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) client.send(out);
    });
  });

  ws.on('close', () => console.log('Client disconnected'));
});

Put index.html (client) in same folder (see client sample below).


5. Start server:

node server.js

Terminal will show WebSocket server listening on ws://localhost:3000.

6. Open index.html in your browser (double-click or File → Open). Open 2 browser tabs or 2 different browsers, set different names, then send messages. They should appear across tabs.


   express + ws server (serves the HTML at http://localhost:3000)

1. Install packages:


npm init -y
npm install express ws

2. server.js:

// server.js - serve static files + WebSocket server
const express = require('express');
const path = require('path');
const http = require('http');
const WebSocket = require('ws');

const app = express();
app.use(express.static(path.join(__dirname))); // serves index.html

const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

wss.on('connection', ws => {
  console.log('Client connected');
  ws.on('message', message => {
    // Broadcast received message to all clients
    wss.clients.forEach(c => {
      if (c.readyState === WebSocket.OPEN) c.send(message);
    });
  });
  ws.on('close', () => console.log('Client disconnected'));
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`App and WS running at http://localhost:${PORT}`);
});

3. Start:

node server.js

4. Open http://localhost:3000 in browser tabs and test.

Client HTML (cleaned single file)

Save as index.html (use this if you want a ready file):

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Real-Time Chat</title>
<style>
  body{font-family:Arial,system-ui;background:#f3f4f6;display:flex;justify-content:center;align-items:center;height:100vh;margin:0}
  .chat-container{width:360px;background:white;border-radius:10px;box-shadow:0 0 10px rgba(0,0,0,0.08);padding:18px}
  .chat-box{height:300px;border:1px solid #e6e6e6;padding:10px;border-radius:6px;overflow-y:auto;background:#fafafa;margin-bottom:10px}
  .message{background:#e8f0ff;padding:8px;border-radius:6px;margin:6px 0}
  .input-row{display:flex;gap:8px}
  input[type="text"]{flex:1;padding:8px;border-radius:6px;border:1px solid #ccc}
  button{padding:8px 12px;border-radius:6px;border:none;background:#1976d2;color:#fff;cursor:pointer}
  button:disabled{opacity:0.6}
  #status{margin-top:8px;font-size:13px;color:#666}
</style>
</head>
<body>
  <div class="chat-container">
    <h3>💬 Real-Time Chat</h3>
    <div>
      <label>Your name: <input id="name" type="text" value="Guest" style="width:120px"></label>
    </div>
    <div id="chatBox" class="chat-box"></div>
    <div class="input-row">
      <input id="msg" type="text" placeholder="Type your message...">
      <button id="send">Send</button>
    </div>
    <div id="status">Status: <span id="statText">Disconnected</span></div>
  </div>

<script>
  const wsUrl = (location.protocol === 'https:' ? 'wss://' : 'ws://') + (location.hostname || 'localhost') + ':3000';
  // If you opened index.html directly (file://), use ws://localhost:3000
  const chatBox = document.getElementById('chatBox');
  const nameInput = document.getElementById('name');
  const msgInput = document.getElementById('msg');
  const sendBtn = document.getElementById('send');
  const statText = document.getElementById('statText');

  let socket;
  function connect() {
    try {
      socket = new WebSocket(wsUrl);
    } catch (e) {
      statText.textContent = 'Error creating WebSocket';
      return;
    }

    socket.onopen = () => { statText.textContent = 'Connected'; };
    socket.onclose = () => { statText.textContent = 'Disconnected'; };
    socket.onerror = () => { statText.textContent = 'Error'; };

    socket.onmessage = (evt) => {
      const data = JSON.parse(evt.data);
      const d = document.createElement('div');
      d.className = 'message';
      d.textContent = data.name + ': ' + data.text;
      chatBox.appendChild(d);
      chatBox.scrollTop = chatBox.scrollHeight;
    };
  }

  connect(); // try to connect immediately

  sendBtn.addEventListener('click', sendMessage);
  msgInput.addEventListener('keydown', (e) => { if (e.key === 'Enter') sendMessage(); });

  function sendMessage() {
    const text = msgInput.value.trim();
    const name = nameInput.value.trim() || 'Guest';
    if (!text) return;
    if (!socket || socket.readyState !== WebSocket.OPEN) {
      alert('Socket not connected.');
      return;
    }
    const payload = { name, text };
    socket.send(JSON.stringify(payload));
    msgInput.value = '';
  }
</script>
</body>
</html>
Quick troubleshooting

If node server.js fails: ensure Node is installed and you ran npm install for packages.

If connection shows Disconnected:

Check server logs — did it say “listening”?

Port 3000 may be in use. Change to another port in server.js (and update client wsUrl).

Firewall or antivirus blocking port.


If messages don't appear across tabs: make sure each tab connected to same ws server and wss.clients broadcast code is running.


Stop server with Ctrl+C in terminal.


Technologies used (simple list)

Client: HTML, CSS, JavaScript (browser WebSocket API)

Server: Node.js (JavaScript runtime)

WebSocket library: ws (npm package) — provides WebSocket server in Node

Optional: express (npm) to serve static files (index.html) and combine with ws


If you want, I can:

give a one-line script to run everything with npm scripts, or

convert this into a single docker-compose + Dockerfile, or

walk you through debugging any error you see (just paste the terminal error).


Which would you like next?


