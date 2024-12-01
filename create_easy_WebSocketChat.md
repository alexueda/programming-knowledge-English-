Here is the full implementation guide for the WebSocket-based chat application described:

### Folder Structure
```
websocket-chat/
├── public/
│   ├── index.html
│   ├── main.css
│   ├── chatClient.js
├── server.js
├── package.json
```

### Code Files

+++markdown
#### `public/index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>WebSocket Chat</title>
    <link rel="stylesheet" href="main.css" />
  </head>
  <body>
    <div class="name">
      <fieldset id="name-controls">
        <legend>My Name</legend>
        <input id="my-name" type="text" />
      </fieldset>
    </div>

    <fieldset id="chat-controls" disabled>
      <legend>Chat</legend>
      <input id="new-msg" type="text" />
      <button onclick="sendMessage()">Send</button>
    </fieldset>
    <div id="chat-text"></div>
  </body>
  <script src="chatClient.js"></script>
</html>
```

#### `public/main.css`
```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

fieldset {
  margin: 10px 0;
}

#chat-text {
  width: 90%;
  height: 50%;
  border: 1px solid #ccc;
  overflow-y: scroll;
  padding: 5px;
}
```

#### `public/chatClient.js`
```javascript
const chatControls = document.querySelector('#chat-controls');
const myName = document.querySelector('#my-name');
const input = document.querySelector('#new-msg');
let socket;

myName.addEventListener('keyup', () => {
  chatControls.disabled = myName.value === '';
});

function appendMsg(cls, from, msg) {
  const chatText = document.querySelector('#chat-text');
  chatText.innerHTML = `<div><span class="${cls}">${from}</span>: ${msg}</div>` + chatText.innerHTML;
}

function sendMessage() {
  const msgEl = document.querySelector('#new-msg');
  const msg = msgEl.value;
  if (msg) {
    appendMsg('me', 'me', msg);
    const name = myName.value;
    socket.send(JSON.stringify({ name, msg }));
    msgEl.value = '';
  }
}

input.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    sendMessage();
  }
});

const protocol = window.location.protocol === 'http:' ? 'ws' : 'wss';
socket = new WebSocket(`${protocol}://${window.location.host}/ws`);

socket.onopen = () => {
  appendMsg('system', 'websocket', 'connected');
};

socket.onmessage = async (event) => {
  const text = await event.data.text();
  const chat = JSON.parse(text);
  appendMsg('friend', chat.name, chat.msg);
};

socket.onclose = () => {
  appendMsg('system', 'websocket', 'disconnected');
  chatControls.disabled = true;
  myName.disabled = true;
};
```

#### `server.js`
```javascript
const express = require('express');
const { WebSocketServer } = require('ws');

const app = express();
const port = process.argv[2] || 3000;

app.use(express.static('./public'));

const server = app.listen(port, () => {
  console.log(`Listening on http://localhost:${port}`);
});

const wss = new WebSocketServer({ noServer: true });

let connections = [];

wss.on('connection', (ws) => {
  const connection = { id: connections.length + 1, alive: true, ws };
  connections.push(connection);

  ws.on('message', (data) => {
    connections.forEach((c) => {
      if (c.id !== connection.id) {
        c.ws.send(data);
      }
    });
  });

  ws.on('close', () => {
    connections = connections.filter((c) => c.id !== connection.id);
  });

  ws.on('pong', () => {
    connection.alive = true;
  });
});

server.on('upgrade', (request, socket, head) => {
  wss.handleUpgrade(request, socket, head, (ws) => {
    wss.emit('connection', ws, request);
  });
});

setInterval(() => {
  connections.forEach((connection) => {
    if (!connection.alive) {
      connection.ws.terminate();
    } else {
      connection.alive = false;
      connection.ws.ping();
    }
  });
}, 10000);
```

#### `package.json`
```json
{
  "name": "websocket-chat",
  "version": "1.0.0",
  "description": "WebSocket-based chat application",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "ws": "^8.13.0"
  }
}
```

---

### Instructions to Run
1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd websocket-chat
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the server:
   ```bash
   npm start
   ```

4. Open multiple browser tabs to [http://localhost:3000](http://localhost:3000) to chat.

5. Use browser developer tools to debug WebSocket communication.

---

This setup will provide you with a fully functional WebSocket chat application that you can extend or refine.
