---
title: "WebSocketで特定のクライアントにデータを送信する"
emoji: "🪐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "websocket", "socketio"]
published: true
---

この記事ではWebSocketによるサーバ・ブラウザ間双方向通信で、指定したクライアントにのみメッセージを送信する方法を紹介します。
## 実装
### 開発構築
クライアント側ではブラウザAPIである[WebSocketAPI](https://developer.mozilla.org/ja/docs/Web/API/WebSockets_API)を使います。
サーバ側では[ws](http://localhost:8000/articles/websocket-bacic-20240424)というNode.jsライブラリを使います。
```
npm install ws
```
-----
### メッセージをブロードキャストする
```js: server.js
const Server = require('ws').Server;

const server = new Server({ port: 3000 });

server.on('connection', (ws) => {
  ws.on('message', (message) => {
    console.log('Received: ' + message);
    server.clients.forEach((client) => {
      if (client.readyState === ws.OPEN) client.send('' + message);
    });
  });
});
```
```html: client.html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>WebSocket Basic</title>
  </head>
  <body>
    <div>
      <input type="text" id="message" />
      <button type="button" onclick="sendMessage()">送信</button>
    </div>
    <ul id="message_list"></ul>

    <script>
      const socket = new WebSocket('ws://localhost:3000');
   
      socket.addEventListener('message', (e) => {
        if (typeof e.data === 'string') {
          console.log(e.data);
          const li = document.createElement('li');
          li.textContent = e.data;
          document.getElementById('message_list').appendChild(li);
        }
      });

      const sendMessage = (e) => {
        socket.send(document.getElementById('message').value);
      };
    </script>
  </body>
</html>
```
-----
### 特定のクライアントのみにメッセージを送信する。
各クライアントに対してIDを割り振ることで実装します。
```js: server.js
const Server = require('ws').Server;

const server = new Server({ port: 3000 });

//接続時、クライアントにIDを割り当てる
const assignClientId = (ws) => {
  const clientId = server.clients.size - 1;
  ws.send('id' + clientId);
};
//切断時、各クライアントにIDを再割り当てする
const reAssignClientId = (ws) => {
  let i = 0;
  server.clients.forEach((client) => {
    if (client.readyState === ws.OPEN) {
      client.send('id' + i);
      i++;
    }
  });
};

server.on('connection', (ws) => {
  assignClientId(ws);
  ws.on('message', (messageData) => {
    const [message, id] = ('' + messageData).split(',');
    //指定のidを持つクライアントに送信
    let i = 0;
    server.clients.forEach((client) => {
      if (client.readyState === ws.OPEN && (id === '' + i || id === '')) {
        client.send('me' + message);
      }
      i++;
    });
  });
  ws.on('close', () => {
    console.log('Closed.');
    reAssignClientId(ws);
  });
});

```

```html: client.html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>WebSocket Basic</title>
  </head>
  <body>
    <h1>WebSocket Basic</h1>
    <b>ClientID: <span id="clientId"></span></b>
    <div>
      <input type="text" id="id" style="width: 16px" />
      <input type="text" id="message" />
      <button type="button" onclick="sendMessage()">送信</button>
    </div>
    <ul id="message_list"></ul>

    <script>
      const socket = new WebSocket('ws://localhost:3000');

      socket.addEventListener('message', (e) => {
        if (typeof e.data === 'string') {
          console.log(e.data);
          if (e.data.slice(0, 2) === 'id') {
            //自身のidを受信した時の処理
            document.getElementById('clientId').textContent = e.data.slice(2);
          } else if (e.data.slice(0, 2) === 'me') {
            //メッセージを受信した時の処理
            const li = document.createElement('li');
            li.textContent = e.data.slice(2);
            document.getElementById('message_list').appendChild(li);
          }
        }
      });
    //ボタンを押した際に、データを送信
    const sendMessage = (e) => {
        const id = document.getElementById('id').value;
        const message = document.getElementById('message').value;
        socket.send(`${message},${id}`);
      };
    </script>
  </body>
</html>
```
![demo](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/websocket-bacic-20240424-demo2.gif)

#### 実行方法
サーバ側
```
node server.js
```
クライアント側はVSCodeの拡張機能LiveServerなどを使いサーバを起動してください。
## WebSocketとは
WebSocketとはHTTPと同様通信プロトコルの一種です。
HTTPとは異なり、ブラウザからのリクエストなしにサーバからブラウザにデータを送信できます。

## WebSocketとSocket.ioの違いと使い分け
Socket.ioはブラウザのWebSocketへの対応が不十分だった時代にそれを補う目的で使用されました。現在はほとんどのブラウザがWebSocketをサポートしているため用途に応じて使い分ける必要があります。
https://developer.mozilla.org/ja/docs/Web/API/WebSockets_API
https://socket.io/docs/v4/
