---
title: "動かして理解するGit | addとcommit編"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git]
published: true
---
この記事では、Gitの`add`, `commit` コマンドについて深掘りした内容をまとめます。

## addコマンド
addは選択したファイルだけをコミットするための仕組みです。
具体的には、以下のことをしています。
- ファイルの内容を圧縮してBlobオブジェクトを作成し`.git/objects/`に保存
- ステージングエリア(`.git/index`)に登録。

### addの内部的な動作を追ってみる
#### 下準備
以下のようにファイル・フォルダを配置し、a.txtに変更を加えました。
```
/folder1
├── /folder-1-1
| 	└── c.txt
└── b.txt
a.txt // ← ここを変更・addしてみる。
```
```diff: a.txtの変更内容
Tree・Blobオブジェクトの確認用。

ここは`a.txt`です。
+ 
+ ここを変更して、addの挙動を見てみる。
```

以下のコマンドでindexに登録されたファイルと、そのBlobオブジェクトのハッシュを見ることができます。

```
git ls-files --stage
```

#### addしていないときのindex
まずは、addしていない状態でindexを見てみます。

```: ls-filesの結果
PS C:\Users\...\git-playground> git ls-files --stage
100644 d8038ab201a19d961c0be971a06479e1b857490a 0       a.txt
100644 6b2641653428bc52ebadf30821061fe6b148b039 0       folder1/b.txt
100644 b993a127140d50bab9c349674730d160a264f29a 0       folder1/folder1-1/c.txt
```

`cat-file` コマンドでa.txtの中身を見てみると、変更前の内容であることが分かります。

```: a.txtの中身
PS C:\Users\...\git-playground> git cat-file -p d8038ab201a19d961c0be971a06479e1b857490a
Tree・Blobオブジェクトの確認用。

ここは`a.txt`です。
```

#### addした後のindex
今度は、`git add a.txt` をした後のindexを見てみます。

```diff: ls-filesの結果
PS C:\Users\...\git-playground> git ls-files --stage
- 100644 d8038ab201a19d961c0be971a06479e1b857490a 0       a.txt
+ 100644 8b36dadad0a62065b636af374e9c8d4c923924cb 0       a.txt
  100644 6b2641653428bc52ebadf30821061fe6b148b039 0       folder1/b.txt
  100644 b993a127140d50bab9c349674730d160a264f29a 0       folder1/folder1-1/c.txt
```

a.txtのハッシュが先ほどと変わっているのが分かります。
中身を見てみると、変更後の内容になっています。

```: a.txtの中身
PS C:\Users\...\git-playground> git cat-file -p  8b36dadad0a62065b636af374e9c8d4c923924cb
Tree・Blobオブジェクトの確認用。

ここは`a.txt`です。

ここを変更して、addの挙動を見てみる。
```

add をすると、選択したファイルの新しいBlobオブジェクトが作成され、indexに上書きされることが分かりました。
add しても、変更前のBlobオブジェクトは削除されずに`.git/objects/`にあり、過去のコミットから参照されます。


:::message
**🤔 addしたファイル以外もステージングエリアに登録されているのはなぜ？**
Gitはコミットごとに「プロジェクト全体のファイル状態」を丸ごと保存しています。
ステージングエリアは、次のコミットに含める「プロジェクト全体のファイル状態」を表しています。
※ ある時点でのプロジェクト全体のファイル群の状態のことを「スナップショット」といいます。
:::

## commit
今後まとめます。