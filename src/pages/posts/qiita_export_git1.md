---
layout: ../../layouts/MarkdownPostLayout.astro
title: "git基礎（理論）"
pubDate: 2022-12-03
description: "git基礎（理論）"
author: "@ekusiadadus"
image:
  url: "https://avatars.githubusercontent.com/u/70436490?s=400&u=a714da7802c65046265c6848887eecddfc58b5c0&v=4"
  alt: "git基礎（理論）"
tags: ["Git", "勉強会"]
---

# git 基礎（理論）

環境:Windows10,Git2.28.0,(Backlog git)
この記事では、git のデータモデルを見ていきます。
実際に動かしたいという方は、[実践編](https://qiita.com/drafts/4dc0eb8bb65e0fd2dc1a/)をみてください。

## §1 git とは

git は、分散型バージョンコントロールシステムです。
例えば、あるプロジェクトを複数人で取り組むときに、"誰が","いつ","どのような"変更をコードにしたのかを管理ます。各個人が、ローカルで作成した部品を統合したり、また、ある時点でエラーが発生した場合、それ以前の状態に簡単に戻すことができます。

git を使うには git の理解しづらいインターフェースを覚えなければなりませんが、その根底にあるデザインとアイデアは非常に美しいです。ここでは、git のデータモデルから始めて、コマンドラインインターフェースについて実際に触って学びます。git のデータモデルを理解すると、コマンドがどのようにデータを操作しているかという点をよりよく理解できるようになっています。

git は、ローカル環境で変更履歴を保存することができるのでリモートサーバーに接続していなくても大丈夫です。
GitHub(ここでは、Backlog を使います)とは、リモートサーバーの役割を果たしていて、世界中の人々と git の仕組みを利用してコラボレーションしてプロジェクトに取り組めるようなプラットフォームです。

それでは、git を始めてみましょう！

## §2 git を始める

Windows10 で Git を始めてみましょう！
まずは、[Git](https://gitforwindows.org/)から、Windows バージョンをダウンロードしましょう。

## §3 データモデル

バージョン管理をする方法は実は、git 以外にもたくさん存在します。
その中でも、git はとてもよく機能するデータモデルを採用していて、変更履歴やサポートブランチ等がチーム開発を潤滑に行えるようにしています。(\*ここでは、ディレクトリとフォルダは同じ意味と捉えて下さい)

git の用語として、

```
"tree":フォルダ(ディレクトリ)
"blob":ファイル
"commit":コミット
"snapshot":スナップショット
```

を使いながら git のデータ構造を見ていきます。

### 1.スナップショット

git はファイルとフォルダの集合の変更履歴を、一連のスナップショットとして親ディレクトリに作ります。git の用語では、ファイルは"blob"と呼ばれ、ディレクトリは"tree"と呼ばれます。ディレクトリは、"blob"や"tree"のを持つことができます。（親ディレクトリの中に存在する子ディレクトリも含まれます）スナップショットは、親ディレクトリで、変更履歴を常に保管します。例えば、下のような"tree"があったとします。

```
\<root(親ツリー(ディレクトリ))>(tree)
|
+- foo(tree)
|  |
|  +-bar.txt(blob, contents="gitは楽しい(マイナビ))"
|
+-baz.txt(blob, contents="gitへようこそ(マイナビ))"
```

親ツリーは、2 つの要素を持ちます。tree:"foo"(1 つの要素 blob:"bar.txt"をそれ自身に含む)と、blob:"baz.txt"です。

### 2.変更履歴

git のバージョン管理は、どのようにスナップショットと関係しているのでしょうか？
一つの簡単なモデルは、変更履歴を時間系列で保管することでしょう。

```
〇<----〇<----〇 (〇は日付時点でのファイル状態を表す,矢印は親を表す)
```

しかし、git はこのような簡単なモデルでバージョン管理をしていません。
git は、変更履歴を DAG(Directed Acyclic Graph,有向非巡回グラフ)として持ちます。すべてのスナップショットは点(Edge)の集合として、その親(変更前のスナップショット)を必ず持っています。必ずしも、親はユニークとは限らず、複数の親集合を持つような子も存在します(マージされたとき等)

```
〇<----〇<----〇(新機能追加)<------〇(バグ修正と新機能追加をマージ)
　　　　↑　　　　　　　　　　　　　　|
　　　　 ------〇(バグ修正)<-------|
```

### 3.データモデル(疑似コード)

git のデータモデルを疑似コード（雰囲気だけ）で書くとこんな感じです。

```
// ファイルはbyteの集合です
type blob = array<byte>

//ディレクトリは、ファイルとディレクトリの辞書集合です
type tree = map<string, tree | blob>

// コミットは、親集合（コミットの）、メタデータ（作成者,メッセージ等）、親ツリーを持ちます。
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

### 4.オブジェクト、アドレス

git の"オブジェクト"は、"blob"(ファイル),"tree"(フォルダ),"commit"(コミット ↑)です.

```
type object = blob | tree | commit
```

git のデータは、[SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1)（Secure Hash Algorithm シリーズの暗号学的ハッシュ関数）というハッシュ関数で暗号化されたアドレスで保管されています。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

blob,tree,commit は上のように統一されています。
実際にコミットをしたときにすべてのファイルを、そのまま保管しているわけではなく、ハッシュ関数によってアドレス(40 文字の 16 進数)を参照しています。例えば、こんな感じです(環境によって違います)

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

ツリーは、その要素のポインタを持っています。

```
\<root(親ツリー(ディレクトリ))>(tree)
|
+- foo(tree)(ポインタを親ツリー(root)が持つ)
|  |
|  +-bar.txt(blob, contents="gitは楽しい(マイナビ)")(ポインタをツリー(foo)が持つ)
|
+-baz.txt(blob, contents="gitへようこそ(マイナビ)")(ポインタを親ツリー(root)が持つ)
```

例えば、

```
git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85(baz.txtのポインタ)
```

とすると "git へようこそ(マイナビ)"が出力されます。

### 5.参照

すべてのスナップショットが(ハッシュ関数で暗号化された)アドレス(40 文字の 16 進数)で識別されますが、人間にとってこれを読むのは困難です。git は、この問題に"参照"で対処しています。参照とは、コミットのポインタです。git、人間にとって識別しやすい文字列で、スナップショットを参照できるようにしています。オブジェクト自体("blob","tree","commit")と違って、参照は変更可能です。例えば、ポインタ"master"は通常最新のコミットを参照しています。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

上のような疑似コードで、git は人間にとって読みやすい名前を使って参照しています。
よく、今どのリビジョン（コミットの状態の単位）にいるのかを確認したいことがありますが、これはしばしば,"HEAD"で参照されます。

### 6.リポジトリ

データモデル最後です
リポジトリとは、大雑把にオブジェクトと参照の集合です。
git が保管しているのは、オブジェクトと参照の集合です。git コマンドが、マッピングしているのはコミット DAG(ファイルを追加したり、参照を更新したり)の操作です。

これから、git のコマンドを打つときは、そのコマンドが一体どんな操作をしているのかを考えましょう。
逆に、何かしらの操作をコミット DAG に行うときには、どのような git コマンドを打てばいいか推測しましょう。
例えば、"未コミットの変化を削除して、参照をコミットポイント 5d83f9e にもどしたい"とき、

```
git checkout master; git reset --hard 5d83f9e
```

というコマンドになります。

##

git 基礎です。初心者向けから、実際どのように動いているのかを解説します。

https://www.youtube.com/watch?reload=9&v=2sjqTHE0zok&ab_channel=MissingSemester \*参考
https://www.youtube.com/watch?reload=9&v=2sjqTHE0zok&ab_channel=MissingSemester
