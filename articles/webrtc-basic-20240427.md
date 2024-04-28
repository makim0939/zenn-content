---
title: "WebRTCで最もシンプルなビデオチャットを実装する"
emoji: "🤙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript, webrtc,]
published: false
---
この記事では、WebRTCによるP2P通信で、できるだけシンプルにビデオチャットを実装する方法を紹介します。
WebRTCについて実際にコードを書いて理解を深めたい方は参考にしてみてください。
## デモ
PC1台でのデモ
![demo1](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/webrtc-basic-20240427-demo1.gif)
Https環境を用意して、スマホ・タブレットで行ったデモ(iPad・iPhone / Safari)
![demo2](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/webrtc-basic-20240427-demo2.gif)
## 実装
### 開発環境
別デバイスからもアクセスしてテストする場合は、httpsで動作す環境を用意します。
:::details Viteでhttps環境を用意
```
npm vite create@latest
```
```
npm i @vitejs/plugin-basic-ssl
```
```js: vite.config.jsを作成
import basicSsl from '@vitejs/plugin-basic-ssl'

export default {
  plugins: [
    basicSsl()
  ]
}
```
:::
### ビデオチャットを実装する
```html: index.html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="./style.css">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>WebRTC ビデオチャット</title>
  </head>
  <body>
    <div id="app">
      <div class="videoContainer">
        <div>
          Local
          <video id="localVideo" autoplay playsinline></video>
        </div>
        <div>
          Remote
          <video id="remoteVideo" autoplay playsinline></video>
        </div>
      </div>
      <div class="buttonWrapper">
        <div class="buttonContainer">
          <p><b>| offer側の操作</b></p>
          <button id="createOfferButton">① Offerを生成</button>
          <button id="setAnswerButton">④ Answerを登録</button>
          <button id="copyIceCandidateButton">⑤ ICE経路情報をコピー</button>
        </div>
        <div class="buttonContainer">
          <p><b>| answer側の操作</b></p>
          <button id="setOfferButton">② Offerを登録</button>
          <button id="createAnswerButton">③ Answerを生成</button>
          <button id="setIceCandidateButton">⑥ ICE経路情報を登録</button>
        </div>
      </div>
      <div class="infoContainer">
        <textarea id="input"></textarea>
        <textarea id="textContainer"></textarea>
      </div>
    </div>
    <script type="module" src="./main.js"></script>
  </body>
</html>
```
```js: main.js
// ===== user1, user2共通 ===== //

//html要素の取得
const textContainer = document.getElementById('textContainer');
const input = document.getElementById('input');
input.placeholder = '手動接続用の入力欄';

const localVideo = document.getElementById('localVideo');
const remoteVideo = document.getElementById('remoteVideo'); 

const createOfferButton = document.getElementById('createOfferButton');
const setAnswerButton = document.getElementById('setAnswerButton');
const copyIceCandidateButton = document.getElementById('copyIceCandidateButton');
const createAnswerButton = document.getElementById('createAnswerButton');
const setOfferButton = document.getElementById('setOfferButton');
const setIceCandidateButton = document.getElementById('setIceCandidateButton');

//user1とuser2のRTCPeerConnectionを生成。
const user1Connection = new RTCPeerConnection();
const user2Connection = new RTCPeerConnection();

//mediaStreamを取得し、トラックをRTCPeerConnectionに登録
navigator.mediaDevices
  .getUserMedia({ audio: true, video: true })
  .then((stream) => {
    localVideo.srcObject = stream;
    //音声トラックとビデオトラックをRTCPeerConnectionに登録
    stream.getTracks().forEach((track) => {
      user1Connection.addTrack(track, stream);
      user2Connection.addTrack(track, stream);
    });
  });

const copyToClipboard = (data, message={success, fail})=> {
  //文字列をデバイスのクリップボードにコピー
  navigator.clipboard
    .writeText(data)
    .then(() => textContainer.innerHTML = message.success)
    .catch((e) => textContainer.innerHTML = message.fail + data);
}

// ===== user1 ===== //

createOfferButton.onclick = () => {
  //Offerを生成
  user1Connection
    .createOffer()
    .then((offer) => user1Connection.setLocalDescription(offer))
    .then(() => {
      console.log(user1Connection.localDescription);
      copyToClipboard(
        JSON.stringify(user1Connection.localDescription),
        {
          success: 'SDP Offerをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「② Offerを登録」を押してください。',
          fail: '● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「② Offerを登録」を押してください。\n\n'
        }
      )
    });
};

setAnswerButton.onclick = async () => {
  //Answerを登録
  const answer = input.value;
  try {
    await user1Connection .setRemoteDescription(JSON.parse(answer)).then(()=>{
      textContainer.innerHTML = 'Answerを登録しました。';
    });
  } catch (e) {
    textContainer.innerHTML = "Answerの登録に失敗しました。\n正しくSDP Answerがコピー・ペーストできているか確認してください。";
  }
  
};

//setLocalDescription()で発火するイベント. ICE Candidateを配列に格納しておく
const candidates = [];
user1Connection.onicecandidate = (e) => candidates.push(e.candidate);
  
copyIceCandidateButton.onclick = () => {
  //保存しておいたICE Candidatesをクリップボードにコピー
  copyToClipboard(
    JSON.stringify(candidates),
    {
      success: 'ICE Candidateをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「⑥ ICE経路情報を登録」を押してください。',
      fail: '● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「⑥ ICE経路情報を登録」を押してください。\n\n'
    }
  )
};

//相手からのトラックを受信したら、remoteVideoにセット
user1Connection.ontrack = e => remoteVideo.srcObject = e.streams[0];

// ===== user2 ===== //

createAnswerButton.onclick = async () => {
  //SDP Answerを生成
  try {
    await user2Connection
    .createAnswer()
    .then((answer) => user2Connection.setLocalDescription(answer))
    .then(() => {
      console.log(user2Connection.localDescription);
      copyToClipboard(
        JSON.stringify(user2Connection.localDescription),
        {
          success: 'SDP Answerをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「④ Answerを登録」を押してください。',
          fail: '● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「④ Answerを登録」を押してください。\n\n'
        }
      )
    });
  } catch (e) {
    console.error(e);
    textContainer.innerHTML = "Answerの生成に失敗しました。\n正しくOfferが登録できているか確認してください。";
  }
  
};

setOfferButton.onclick = async () => {
  //Offerを登録
  const offer = input.value;
  try {
    await user2Connection.setRemoteDescription(JSON.parse(offer)).then(()=>{
      textContainer.innerHTML = 'Offerを登録しました。';
    });
  } catch (e) {
    textContainer.innerHTML = "Offerの登録に失敗しました。\n正しくSDP Offerがコピー・ペーストできているか確認してください。";
  }
};

setIceCandidateButton.onclick = () => {
  //ICE Candidatesを登録
  const candidatesStr = input.value;
  try {
    const senderCandidates = JSON.parse(candidatesStr);
    senderCandidates.forEach(async (candidate) => {
      if (candidate === null) return;
      await user2Connection.addIceCandidate(candidate)
    });
    textContainer.innerHTML = 'ICE経路情報を登録しました。';
  } catch (e) {
    textContainer.innerHTML = "ICE経路情報の登録に失敗しました。\n正しくICE経路情報がコピー・ペーストできているか確認してください。";
  }
};

//相手からのトラックを受信したら、remoteVideoにセット
user2Connection.ontrack = e => remoteVideo.srcObject = e.streams[0];
```
:::details 見やすいようにスタイリング
```css: style.css
html {
    font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
    line-height: 1.5;
}
#app {
    max-width: 1280px;
    width: 100%;
    margin: 0 auto;
  }
  .videoContainer {
    width: 100%;
    display: flex;
  }
  .videoContainer > div {
    width: 48%;
    margin: 16px 1%;
  }
  video {
    width: 100%;
    border-radius: 8px;
  }
  .buttonWrapper {
    width: 100%;
    display: flex;
    justify-content: center;
    margin-top: 8px;
  }
  .buttonContainer {
    width: 48%;
    margin: 16px 1%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    margin-top: 8px;
  }
  .infoContainer {
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    margin-top: 16px;
  }
  #textContainer {
    max-width: 100%;
    height: 108px;
    padding: 8px;
    margin: 8px;
    font-size: 16px;
    border: 1px solid #d9d9d9;
    border-radius: 8px;
  }
  #textContainer:empty {
    width: 0;
    opacity: 0;
    border: none;
  }
  #input {
    max-width: 100%;
    height: fit-content;
    padding: 8px;
    margin: 8px;
    font-size: 16px;
    border: 1px solid #d9d9d9;
    border-radius: 8px;
  }
  button {
    border-radius: 8px;
    border: 1px solid transparent;
    margin: 4px;
    padding: 8px 16px;
    font-size: 16px;
    background-color: #f0f0f0;
    cursor: pointer;
    transition: border-color 0.25s; 
  }
  button:hover {
    border-color: #646cff;
  }
  button:focus,
  button:focus-visible {
    outline: 4px auto -webkit-focus-ring-color;
  }
```
:::

### コードの解説

## WebRTCとは