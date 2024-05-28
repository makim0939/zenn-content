---
title: "TextAliveAppApiで歌詞・コード・ビートをリアルタイムに取得する。"
emoji: "🎺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript, typescript, textaliveappapi]
published: false
---
## デモ
![デモ](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/textalive-app-basic-20240527-demo1.gif =480x)
## 実装
### 環境構築
TextAliveAppApiを使用するには、[開発者登録](https://developer.textalive.jp/profile)をしてTextAliveアプリを作成し、アプリトークンを取得する必要があります。

viteプロジェクトの作成、npmパッケージのインストール
```
npm create vite@latest プロジェクト名 -- --template vanilla-ts
npm i
```
```
npm i textalive-app-api
```
### コードの全体像
```ts: main.ts
import './style.css';
import { IPlayerApp, Player, Timer } from 'textalive-app-api';

// ===== ボタンと表示部分の準備 ===== //
//HTML要素を取得します。
const playButton = document.querySelector('#play_button') as HTMLButtonElement;
const pauseButton = document.querySelector('#pause_button') as HTMLButtonElement;
const rewindButton = document.querySelector('#rewind_button') as HTMLButtonElement;
//再生する準備が整うまで無効にしておきます。
playButton.disabled = true;
pauseButton.disabled = true;
rewindButton.disabled = true;

const charDisplay = document.querySelector('#char') as HTMLSpanElement;
const wordDisplay = document.querySelector('#word') as HTMLSpanElement;
const phraseDisplay = document.querySelector('#phrase') as HTMLSpanElement;
const chordDisplay = document.querySelector('#chord') as HTMLSpanElement;
const beatDisplay = document.querySelector('#beat') as HTMLSpanElement;

//ボタンに再生・停止・巻き戻しをセットします。
const addButtonEventListeners = () => {
  playButton.addEventListener('click', () => player.requestPlay());
  pauseButton.addEventListener('click', () => player.requestPause());
  rewindButton.addEventListener('click', () => player.requestMediaSeek(0));
};

// ===== 楽曲情報を取得、表示するためのメソッドを用意 ===== //
//ループしてその都度データを取得、表示します。prevBeatStartTimeは
const loop = (prevBeatStartTime?: number) => {
  const nowPosition = player.timer.position;
  //歌詞の取得。文字・単語・フレーズ単位で取得できます。
  const char = player.video.findChar(nowPosition)?.text;
  const word = player.video.findWord(nowPosition)?.text;
  const phrase = player.video.findPhrase(nowPosition)?.text;
  //歌詞があれば表示します。
  char && (charDisplay.textContent = char);
  word && (wordDisplay.textContent = word);
  phrase && (phraseDisplay.textContent = phrase);

  //コードの取得
  chordDisplay.textContent = player.findChord(nowPosition).name;

  //ビートの取得
  const beat = player.findBeat(nowPosition);
  if (beat && beat.startTime !== prevBeatStartTime) {
    //取得したビートが小節中の何拍目かに応じて「*」を表示します。
    prevBeatStartTime = beat.startTime;
    let beatText = '';
    for (let i = 0; i < beat.position; i++) {
      beatText += '* ';
    }
    beatDisplay.textContent = beatText;
  }
  requestAnimationFrame(() => loop(prevBeatStartTime));
};

//Playerに設定するイベントリスナです。(onAppReady, onTimerReady)
const onAppReady = (app: IPlayerApp) => {
  //好きな楽曲のURLを指定します。//URLの取得方法は記事の後半で解説しています。
  player.createFromSongUrl('https://piapro.jp/t/fnhJ/20230131212038');
  addButtonEventListeners();
};
const onTimerReady = (t: Timer) => {
  //再生の準備が整った時に呼ばれます。
  playButton.disabled = false;
  pauseButton.disabled = false;
  rewindButton.disabled = false;
  requestAnimationFrame(loop);
};

// ===== Playerを作成してリスナを登録することで順次処理を開始　===== //
const player = new Player({
  app: { token: 'ここにアプリトークン' },
  mediaElement: document.querySelector('#media') as HTMLDivElement,
});
player.addListener({
  onAppReady,
  onTimerReady,
});
```
```html: index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>TextAliveApp Basic</title>
  </head>
  <body>
    <div id="app">
      <div class="controls">
        <div id="media"></div>
        <div id="buttons">
          <button id="play_button">Play</button>
          <button id="pause_button">Pause</button>
          <button id="rewind_button">ReWind</button>
        </div>
      </div>
      <div>
        <h3>Lyric</h3>
        <div class="div"></div>
        <p>Char: <span id="char"></span></p>
        <p>Word: <span id="word"></span></p>
        <p>Phrase: <span id="phrase"></span></p>
        <h3>Chord</h3>
        <p>Chord: <span id="chord"></span></p>
        <h3>Beat</h3>
        <p>Beat: <span id="beat"></span></p>
      </div>
    </div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```
:::details cssでスタイリング
```css style.css
:root {
  color: #252528;
}
body {
  margin: 0;
  display: flex;
  place-items: center;
  min-width: 320px;
  min-height: 100vh;
}
#app {
  min-width: 480px;
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
}
.controls {
  width: fit-content;
  margin: 0 auto;
}
h3 {
  border-bottom: gray 1px solid;
}
button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
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
### Playerの準備
リスナーの設定
楽曲URLの取得方法
### ループを実装
### 歌詞を取得
### コードを取得
### ビートを取得
