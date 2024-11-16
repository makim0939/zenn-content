---
title: "Biomeのセットアップ"
emoji: "⛰️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Biome]
published: true
---

ESlintの設定が苦しくなったのでBiomeを導入してみました。

## 1. VSCodeの設定
- **VSCodeの拡張機能のインストール**
[Biome VS Code extension](https://marketplace.visualstudio.com/items?itemName=biomejs.biome)
![VSCode拡張機能](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/biome-setup-20241116/vscode-plugin.png =420x)
*VSCode拡張機能*
- **Formatterの設定**
右下の歯車マークから「設定」を開いて「format」で検索すると「Editor:DefaultFormatter」が出てきます。
![フォーマッターの設定](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/biome-setup-20241116/formatter-settings.png  =420x)
*Formatterの設定*
:::message
この時点ではまだフォーマットは適用されません
:::
## 2. プロジェクトにBiomeをインストール
[ドキュメント Get started](https://biomejs.dev/ja/guides/getting-started/)
- **npmパッケージのインストール**
```
npm install --save-dev --save-exact @biomejs/biome
```
```
npx @biomejs/biome init
```
設定ファイル`biome.json`が作成されます。
ここまでの設定を終えれば、VSCodeで警告・フォーマットされるようになります。
![警告が出る様子](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/biome-setup-20241116/alert.png =540x)
警告文からドキュメントに飛べて、ドキュメントがわかりやすいのでなんのエラーかすぐにわかります。
- **ignoreファイルを設定する**
```:json biome.json
{
  ...
  files: {
    "ignore": ["**/node_modules/**/]
  }
}
```
## 3. コマンド
- **lint**
```
npx @biomejs/biome lint --write ファイル名
```
- **format**
```
npx @biomejs/biome format --write ファイル名
```
- **lint + format + import文のソート**
```
 npx @biomejs/biome check --write ファイル名 
```
import文**ファイルからの距離**によってソートされるみたいです。
(距離が同じ場合はアルファベット順)
- **試しに`check --write`してみる**
![ソート前](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/biome-setup-20241116/before-sort.png =360x)
*ソート前*
![ソート後](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/biome-setup-20241116/after-sort.png =360x)
*ソート後*

## 4. Huskyでコミット前にlint, format, import文のソートを適用
- **HuskyとLint-stagedのインストール**
```
npm i --save-dev husky lint-staged
```
```
npx husky install
```
これで`.husky/`ディレクトリが作成されます。
```: .husky/pre-commit
npx lint-staged
```
- **package.jsonの編集**
```:json package.json
...
"scripts": {
  ...
  "prepare": "husky"
},
...
"lint-staged": {
"*.{js,jsx,ts,tsx}": [
  "biome check --write --no-errors-on-unmatched" //お好みのコマンド
]
}
```
lint-stagedに設定するコマンド例

- `"biome check --files-ignore-unknown=true "`
フォーマットのチェックとリント
- `"biome check --write --no-errors-on-unmatched "`
フォーマット、インポート文のソート、リント、安全な修正の適用
- `"biome check --write --organize-imports-enabled=false --no-errors-on-unmatched "`
 フォーマットと安全な修正の適用
- `"biome check --write --unsafe --no-errors-on-unmatched "`
フォーマット、インポート文のソート、リント、安全及び安全ではない修正の適用
- `"biome format --write --no-errors-on-unmatched"`
フォーマット
- `"biome lint --write --no-errors-on-unmatched "`
リントと安全な修正の適用