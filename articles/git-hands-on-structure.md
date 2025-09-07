---
title: "動かして理解するGit | Gitの構成要素編"
emoji: "🔩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git]
published: true
---

会社でGitを使っていると、よくエラーに出くわします。
エラーメッセージが理解できず、原因もわからないので、いつも先輩に丸投げ状態です。
先輩社員が、なにやらカタカタして直してくれるのを見て、自分でもできるようになりたい！
…と思い、Gitについて知ろうと思いました。

Gitについて、記事やAIや実践から得た情報を、いくつかの章に分けてまとめようと思います。
この記事では、gitの仕組みを理解するにあたって、.gitの中身について調べた内容をまとめます。

## .gitディレクトリ
.gitディレクトリの中には、gitの仕組みを実現するための情報が記録されています。

## オブジェクト(.git/objects/)
Gitは4種類のオブジェクトを都度作成して、変更履歴を記録しています。
オブジェクトを理解することで、Gitがどのようにコミットとコミット時のプロジェクト全体を記録しているかが分かります。

オブジェクトが作成されるとき、内容に基づいてSHA-1ハッシュが計算されます。
オブジェクトは`.git/objects/ハッシュの先頭2桁/残りの38桁` に格納されます。

この性質から、データが正しいことを検証できます。

以下のコマンドでオブジェクトの中身を見ることができます。
```json
git cat-file -p ハッシュ
```

ここからは、4種類のオブジェクトについて、説明します。
1. **Blobオブジェクト**
ファイルの中身そのものを保存するオブジェクト。
`git add` したときに選択したファイルを圧縮して作成される。
ファイル名やディレクトリ構造の情報は持たない単なるバイト列。
    
    ```:cat-fileでblobオブジェクトを見た例
    PS C:\Users\...\git-playground> git cat-file -p d8038ab201a19d961c0be971a06479e1b857490a
    Tree・Blobオブジェクトの確認用。

    ここは`a.txt`です。
    ```
    
2. **Treeオブジェクト**
ディレクトリを表すオブジェクト。
コミット時に作成される。
TreeオブジェクトとBlobオブジェクトが入れ子の構造を持ち、プロジェクト全体のディレクトリ構造とファイル中身の状態を記録しています。
    
    ```:cat-fileでtreeオブジェクトを見た例
    PS C:\Users\...\git-playground> git cat-file -p fedbef5902a945e098f3fa00f72c037e3e1167e1
    100644 blob d8038ab201a19d961c0be971a06479e1b857490a    a.txt
    040000 tree d8b465f99cbd8047e700b7a5843dccaf685efb77    folder1
    ```
    
3. **Commitオブジェクト**
    コミットしたときに作成される変更履歴を記録するためのオブジェクト。
    以下の情報を持つ。
    
    - ルートディレクトリのTreeオブジェクトのハッシュ
    - 親コミットのハッシュ
    - 著者とタイムスタンプ
    - コミッターとタイムスタンプ
    - コミットメッセージ
    
    ```:cat-fileでcommitオブジェクトを見た例
    PS C:\Users\...\git-playground> git cat-file -p c175cc4b295c544d3b80662f46cf06383171a191
    tree fedbef5902a945e098f3fa00f72c037e3e1167e1
    parent 0aa70a1401af7a4f10ef5a99368763182aede522
    author makim0939 <makim0939@gmail.com> 1757221279 +0900
    committer makim0939 <makim0939@gmail.com> 1757221279 +0900
    
    オブジェクト確認用のフォルダ・ファイルを配置
    ```
    
4. **Tagオブジェクト**
    今後まとめます。
    

### 実際にオブジェクトの中身を見てみる
ここからは実際に動かしながら見ていきます。
そのために以下のようにフォルダとファイルを配置しコミットしました。

```
/folder1
├── /folder-1-1
| 	└── c.txt
└── b.txt
a.txt
```

まずはコミットのハッシュを得るためにログを見ます。

```
PS C:\Users\...\git-playground> git log
commit c175cc4b295c544d3b80662f46cf06383171a191 (HEAD -> write-article, origin/write-article)
Author: makim0939 <makim0939@gmail.com>
Date:   Sun Sep 7 14:01:19 2025 +0900

    オブジェクト確認用のフォルダ・ファイルを配置

以下省略
```

`c175cc4b295c544d3b80662f46cf06383171a191`が最新のコミットのハッシュです。

上で得たハッシュから、`cat-file` コマンドでCommitオブジェクトの中身を見てみます。

```
PS C:\Users\...\git-playground> git cat-file -p c175cc4b295c544d3b80662f46cf06383171a191
tree fedbef5902a945e098f3fa00f72c037e3e1167e1
parent 0aa70a1401af7a4f10ef5a99368763182aede522
author makim0939 <makim0939@gmail.com> 1757221279 +0900
committer makim0939 <makim0939@gmail.com> 1757221279 +0900

オブジェクト確認用のフォルダ・ファイルを配置
```

Treeオブジェクト・親コミット・著者・コミッター・コミットメッセージの情報を持っていることが分かりますね。

コミットオブジェクトに記載されているTreeオブジェクトの中身を見てみます。

```
PS C:\Users\...\git-playground> git cat-file -p fedbef5902a945e098f3fa00f72c037e3e1167e1
100644 blob d8038ab201a19d961c0be971a06479e1b857490a    a.txt
040000 tree d8b465f99cbd8047e700b7a5843dccaf685efb77    folder1
```

a.txtとfolder1があることから、ルートディレクトリのTreeオブジェクトであることが分かります。また、Treeオブジェクトが子階層のTreeオブジェクトを持つ構造になっています。

最後にBlobオブジェクトの中身を見てみます。

```
PS C:\Users\...\git-playground> git cat-file -p d8038ab201a19d961c0be971a06479e1b857490a
Tree・Blobオブジェクトの確認用。

ここは`a.txt`です。
```

これは実際にa.txtに書いてある内容です。

コミットは、親へ親へと辿っていくことができ、各コミットではその時点でのディレクトリ構造とファイルの内容がTree・Blobオブジェクトによって紐づけられていることがわかりました。

addやcommitについて知るときに、オブジェクトを知っていると腑に落ちると思います。

## 参照(.git/refs/)
今後まとめます。