---
title: "競プロコンテスト環境などを自動化しました"
date: 2020-03-04T18:08:55+09:00
draft: false
tags: [compro, Development]
slug: automate
mathjax: true
---

お久しぶりです。ますぐれです。卒論が終わり、現在は一人暮らしの準備中です。
本日は競技プログラミングのあれこれを色々自動化したので、その環境構築の内容について書いていきたいと思います。

## 自動化したこと
* `#include`した自作ライブラリを提出用コードに展開する
* ライブラリの自動テスト環境
* コンテスト中のサンプルテスト/提出

手元のOSはwindows10ですが、基本的にはWSLを使っています。競プロ環境もWSLに載せています。
では、早速順番に自動化の中身について見に行きましょう。

## `#include`した自作ライブラリを提出用コードに展開する
そもそも、自動化を始めたきっかけはこれです。従来の競プロ環境では、ライブラリとは別にVSCodeのコードスニペット用のファイルも管理していたのですが、これが気持ち悪いのでやめたいなあというところがスタート地点でした。

そんなことをツイッターでつぶやいたところ、熨斗袋さんが[online-judge-verify-helper](https://github.com/kmyk/online-judge-verify-helper)の存在を教えてくれました。

<blockquote class="twitter-tweet" data-cards="hidden" data-partner="tweetdeck"><p lang="ja" dir="ltr">online judge verify helper がありますね</p>&mdash; 熨斗袋 (@noshi91) <a href="https://twitter.com/noshi91/status/1231465966837518336?ref_src=twsrc%5Etfw">February 23, 2020</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

これは本当はライブラリのCIを回すためのツールで、この中に`oj-bundle`というプログラムが入っています。これがまさにほしいものだったので導入しました。コマンド一発で簡単です。

これを入れて`oj-bundle hoge.cpp > fuga.cpp`とすると、`hoge`側で`#include`された自作ライブラリが`fuga`にババーンと展開されるというわけです。最高です。やっとスニペットとダブルで管理する苦しみから解放されました。

……と思ったのですが、これだけでは不十分です。というのも、自分はライブラリを書いているファイルと同じところにmainを作ってverifyしていたので、これも付いてきてしまうわけですね。そのためライブラリ自体に手を入れる必要が出てきました。

## ライブラリの自動テスト環境
その問題を解決するため、このままテスト環境もいい感じにすることを決意しました。幸いさっきインストールしてきたonline-judge-verify-helperはまさしくそれのためのツールなのでこのまま利用します。

CIの環境としてはGithub Actionsを選択しました。今まで使ったことがなかったのでテストも兼ねています。
詳しい使い方は公式サイトを見るとわかると思います。

特にハマるところはないんですが、手元で普通にテストを回すとclang++とg++両方が必要というところでぼくはハマりました。clang++使ってないのにclang++入れるんかみたいな気持ちになっていたのですが、環境変数`CXX`に自分の使うコンパイラを指定してあげることで解消できることを開発者の[@kmyk](https://github.com/kmyk)さんに教えていただきました。

[Skip execution with clang++ if clang++ doesn't exist · Issue #173 · kmyk/online-judge-verify-helper](https://github.com/kmyk/online-judge-verify-helper/issues/173)

これぼくがツイッターでつぶやいた数時間後にはissueできててめっちゃ感動しました。

閑話休題。ということで、手元で実行するときは`CXX=g++ oj-verify run`みたいにしてあげると良さそうです。使い勝手もよくて楽しいです。

oj-verifyの大まかな使い方は、`***.test.cpp`みたいなファイルを用意して、そこに`#define PROBLEM ${問題のURL}`と書いておくと、そのファイルのプログラムがその問題に対して実行されるという感じです。C++以外にも対応しています。詳しいことは[公式レポジトリ](https://github.com/kmyk/online-judge-verify-helper)へ。

GitHub Actionsについても公式に書いてありますが、ぼくはこんな感じにして運用しています。こっちもかなり使いやすくてよいですね。
```
name: verify

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v1
      with:
        python-version: 3.8

    - name: Install dependencies
      run: pip3 install -U git+https://github.com/kmyk/online-judge-verify-helper.git@master

    - name: Run tests
      env:
        CXX: g++
      run: oj-verify run --tle 10
```

## コンテスト中のサンプルテスト/提出
人間だんだんと欲が出てくるもので、ここまできたらコンテスト中にコマンドラインから提出できたらいいなあとなりました。bundleとか忘れそうですし、自動化したいわけです。

ここら辺についても調べていると、[online-judge-tools](https://github.com/kmyk/online-judge-tools)というものと[atcoder-cli](https://github.com/Tatamo/atcoder-cli)というツールを発見します。前者はonline-judge-verify-helperと同じ方が開発しているものです。後者はそれをAtCoder用に使いやすくしたWrapperです。(でもnodeで動いているのでnodeの環境が別途必要です)

これ使えば一瞬で出来るなと思ってやってみたんですが、WSLだと提出後にテキストブラウザが起動したり、提出の度に確認を求められたりしてかなり面倒でした。なので、atcoder-cliはコンテストフォルダ作成用と割り切って、提出/テスト部分は自前で作ることにしました。

とはいっても、雑にやればかなり簡単なシェルスクリプトを書くだけでサクッと自動化できます。atcoder-cliが作成するフォルダはいい感じにカスタマイズできるので、作成するたびにそれ用のシェルスクリプトとテンプレートファイルを置くように設定しました。

シェルスクリプトはこんな感じです。コンテストフォルダは`abc***/a/template.cpp`みたいな感じで1問題1フォルダみたいな構成になっています。`abc***`の中に入ってそれぞれ提出したい問題番号(aとかbとか)を引数に渡して実行します。

```test.sh
#!/bin/bash

basename `pwd`
cd $1
g++ template.cpp && oj test
```

```submit.sh
#!/bin/bash

contest=$(basename `pwd`)
cd $1
oj-bundle template.cpp > submit.cpp
oj s "https://atcoder.jp/contests/${contest}/tasks/${contest}_$1" submit.cpp --no-open -y
```

こうすることでコンテスト中はもうコマンドラインだけですべてが完結するようになりました。`--no-open`を付けることでブラウザが立ち上がるのを拒否し、`-y`で毎回提出の際に間違っていないか確認を求められることがなくなります。

## 終わりに
実は、ここで書いたシェルスクリプトはatcoder-cliが内部的に呼ぶ`oj`にオプションを渡せるようにしてくれると不要になります。ちょっとコードリーディングしてきたのですが、渡せなさそうだったので仕方がなくこういうスクリプトを書いているという感じです。アプデを期待しています。

上のシェルスクリプトなんですが、フォルダの名前から提出先URLを推測しているので、ARCに併設されていたABCではうまく動きません。最近はこういうコンテストもないし大丈夫かなあと思ってこんな感じで妥協しています。

こういう自動化とか楽しいので、まだやったことないという方がいらっしゃれば是非取り組んでみましょう！　身近な作業の自動化は労力の割にめっちゃ幸せになれることが多いのでぜひに。

