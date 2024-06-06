---
title: "TextAliveAppApiで歌詞・コード・ビートをリアルタイムに取得する。"
emoji: "🎺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript, typescript, textaliveappapi]
published: true
---
この記事では[TextAliveAppAPI](https://developer.textalive.jp/)を使って、楽曲から歌詞・コード・ビートを取得するシンプルな実装を紹介します。

## デモ
![デモ](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/textalive-app-basic-20240527/demo.gif =480x)
## 実装
### 環境構築
**TextAliveAppAPIの事前準備**
TextAliveAppApiを使用するには、[開発者登録](https://developer.textalive.jp/profile)をしてTextAliveアプリを作成し、アプリトークンを取得する必要があります。
![アプリトークンの取得](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/textalive-app-basic-20240527/apptoken.png =480x)

**viteプロジェクトの作成、npmパッケージのインストール**
```
npm create vite@latest プロジェクト名 -- --template vanilla-ts
npm i
```
```
npm i textalive-app-api
```
### 歌詞・コード・ビート情報を取得し表示する
```ts: main.ts
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
//Playerに設定するイベントリスナです。(onAppReady, onTimerReady, onTimeUpdate)
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
};

let prevBeatPosition = -1;
const onTimeUpdate = (position: number) => {
  //歌詞の取得。文字・単語・フレーズ単位で取得できます。
  const char = player.video.findChar(position)?.text;
  const word = player.video.findWord(position)?.text;
  const phrase = player.video.findPhrase(position)?.text;
  //歌詞があれば表示します。
  char && (charDisplay.textContent = char);
  word && (wordDisplay.textContent = word);
  phrase && (phraseDisplay.textContent = phrase);

  //コードの取得
  chordDisplay.textContent = player.findChord(position).name;

  //ビートの取得
  const beat = player.findBeat(position);
  if (beat.position === prevBeatPosition) return;
  let beatText = '';
  for (let i = 0; i < beat.position; i++) {
    beatText += '* ';
  }
  beatDisplay.textContent = beatText;
  prevBeatPosition = beat.position;
};

// ===== Playerを作成してリスナを登録することで順次処理を開始　===== //
const player = new Player({
  app: { token: 'ここにアプリトークン' },
  mediaElement: document.querySelector('#media') as HTMLDivElement,
});
player.addListener({
  onAppReady,
  onTimerReady,
  onTimeUpdate,
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
### 実行方法
```
npm run dev
```
## 解説
### リスナーの設定
`onAppReady`：ホスト・楽曲URLの有無の確認はここに記述します。\
`onTimerReady`：発火したら、再生の準備は完了です。\
`onTimeUpdates`：再生位置が変化するたびに発火します。ここに楽曲情報の処理を記述します。
[公式ドキュメント→](https://developer.textalive.jp/app/life-cycle/)

### 楽曲情報の取得
楽曲情報の取得はonTimerUpdates()に記述すると良いと思います。
- **歌詞の取得**
歌詞は文字・単語・フレーズの3つの異なる単位で取得できます。
```
const char = player.video.findChar(position).text
const word = player.video.findWord(position).text
const phrase = player.video.findPhrase(position).text
```
<!-- - **コードの取得**
- **ビートの取得** -->

### 楽曲URLの取得方法
使用できる楽曲は https://textalive.jp/songs で検索できます。
![楽曲検索](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/textalive-app-basic-20240527/search-song.png =480x)

ローカル環境ではニコニコ動画のURLの楽曲は動作しません。ngrokで公開しHTTPS経由でアクセス可能にすると正常に動作しました。
![エラー](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/textalive-app-basic-20240527/nikoniko-error.png =360x)

https://developer.textalive.jp/app/