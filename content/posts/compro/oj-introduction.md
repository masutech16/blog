---
title: "online-judge-toolsを導入しよう！"
date: 2020-05-09T12:19:45+09:00
draft: false
tags: [compro]
---

以前から何度か紹介している[online-judge-tools](https://github.com/online-judge-tools/oj)の布教をしていきたいと思います。

基本的には[公式のドキュメント](https://online-judge-tools.readthedocs.io/en/master/introduction.ja.html)を読んだ方が情報がたくさん得られます。この記事では初めて`oj`を初めて使う人が最低限抑えておくべき内容を紹介しています。使っている人には物足りない記事だと思います。

# 導入と基本的な使い方
## 導入
基本的には公式のリファレンスが整備されているのでそれ通りに導入しましょう。python3が動く環境なら簡単に入れられると思います。動作確認はお使いのシェル上で`oj`を打てばOKです。色々なサブコマンドの一覧が出力されればインストールできています。具体的には以下のような出力が得られます。(ちょっと編集しています)

```
$ oj
usage: oj [-h] [-v] [-c COOKIE] [--version] {download,d,dl,login,l,submit,s,test,t,generate-output,g/o,generate-input,g/i,test-reactive,t/r} ...

Tools for online judge services

positional arguments:
  {download,d,dl,login,l,submit,s,test,t,generate-output,g/o,generate-input,g/i,test-reactive,t/r}
                        for details, see "oj COMMAND --help"
    download (d, dl)    download sample cases
    login (l)           login to a service
    submit (s)          submit your solution
    test (t)            test your code
    generate-output (g/o)
                        generate output files from input and reference implementation
    generate-input (g/i)
                        generate input files from given generator
    test-reactive (t/r)
                        test for reactive problem

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose
  -c COOKIE, --cookie COOKIE
                        path to cookie.
  --version             print the online-judge-tools version number
```

こんな感じの出力が得られなかった場合はインストールに失敗しています。もしそうなったらGitHubのissue等を確認してみると良いです。また、それでも解決しない場合はTwitterなどでHelpを求めると有識者が答えてくれるかもです。

## 基本の使い方
### ダウンロード
ここからは`oj`の基本的な使い方について説明していきます。これだけでも十分便利なので、まずはここに書かれている機能を使いこなせるようになりましょう！　

`oj`は基本的に1問題に1フォルダを使います。フォルダを切ることをしてこなかった人にとっては最初は不便かもしれませんが、これのおかげで様々な恩恵を受けられます。慣れれば自然に感じるのでここは慣れましょう。ぼくは`workspace`という競プロの問題を解く用のディレクトリを作っているんですが、その下に毎回問題用のフォルダを作成しています。

具体例として、ABC166のA問題を`oj`を利用するとどんな感じになるかということをチェックしていきましょう。

まずフォルダを作成します。名前が他の問題と被ると面倒なことが起きるので、ここで作るフォルダの名前は問題と一対一に対応するようなものにしておきましょう。

そうしたら作成したフォルダの中に入り`oj d {問題のURL}`というコマンドを打ちます。

```
$ mkdir abc_166 a
$ cd abc166_a
$ oj d https://atcoder.jp/contests/abc166/tasks/abc166_a
[x] clear the downloading history for this directory: /home/masutech/.cache/online-judge-tools/download-history.jsonl
[x] append the downloading history: /home/masutech/.cache/online-judge-tools/download-history.jsonl

[*] sample 0
[x] input: sample-1
ABC

[+] saved to: test/sample-1.in
[x] output: sample-1
ARC

[+] saved to: test/sample-1.out
```

これはサンプルケースをダウンロードしてくれるコマンドです。ダウンロードされたサンプルケースは基本的には`test`以下のフォルダに格納されます。

この中を見てみると、`sample-1.in`や`sample-1.out`というファイルが見つけられると思います。これらがAtCoder上で公開されているテストケースの入力例/出力例に対応しています。

### テスト
それらを確認したところでコードを書いていきます。ojがちゃんと機能していることを確認するために、まずはWAを出すコードを書いてテストしてみましょう。以下のコードを書きました。

```cpp:abc166_a.cpp
#include <iostream>
#include <string>

int main() {
  std::string s;
  std::cin >> s;
  std::cout << "ABC" << std::endl;
}
```

これをコンパイルしてテストしてみます。C++を使っている場合は`a.out`というバイナリを生成すると`oj t`だけで実行できます。そのほかの場合は`oj t -c {実行コマンド}`と書くとできます。ここら辺の詳細はドキュメントを読みましょう！

実行結果はこんな感じです。

```
$ g++ abc166_a.cpp && oj t
[*] 1 cases found

[*] sample-1
[x] time: 0.032876 sec
[-] WA
output:
ABC

expected:
ARC


[x] slowest: 0.032876 sec  (for sample-1)
[x] max memory: 1.712000 MB  (for sample-1)
[-] test failed: 0 AC / 1 cases
```

このコマンドは`g++`を用いて`abc166_a.cpp`というファイルをコンパイルし`a.out`という実行ファイルを作成した後に、`oj t`を実行することを意味しています。

この間僅か1秒もありません。コピペが異常に高速な人以外にとって、これは自分でテストするよりも高速です。特にABCの簡単な問題とかは手作業だと面倒くさがってサンプル通さない場合とかあると思うんですが、コマンド一つでできるならサボりませんよね。というわけで低難易度の誤爆とかを防げます。レート帯によってはABCのペナは十分重いので、こうやって保証できると嬉しいですよね。

ちなみに、ちゃんとサンプルをパスするコードをテストするとこんな感じになります。

```cpp
#include <iostream>
#include <string>

int main() {
  std::string s;
  std::cin >> s;
  std::cout << "ARC" << std::endl;
}
```

```
$ g++ abc166_a.cpp && oj t
[*] 1 cases found

[*] sample-1
[x] time: 0.028122 sec
[+] AC

[x] slowest: 0.028122 sec  (for sample-1)
[x] max memory: 1.712000 MB  (for sample-1)
[+] test success: 1 cases
```

#### サンプルケースの追加
上のコードをよく見てみると、なんとこれは`ABC`と出力していた所を`ARC`に書き換えただけです。今回はサンプルが一つしかないためにすり抜けてしまいますが、これではACを得ることができません！　`oj`を用いた自動テストではこういう場合に自作テストを突っ込むことができます。

こんな簡単な場合は不要かもしれませんが、例えばサンプルにはないけど考えているときに発見したコーナーケースや、実装上バグりそうなケースなどを突っ込んでおけば、`oj t`だけでサンプルに加えてこれらについても簡単にテストを行うことができます。

使い方は簡単で、`test`のフォルダ以下に入力が書かれた`hoge.in`というファイルと、出力が書かれた`hoge.out`というファイルを追加するだけです。これだけで自動テストが行えるようになります。試しに`test`以下に`abc-1.in`と`abc-1.out`というファイルを追加してテストを実行してみます。

```
$ oj t
[*] 2 cases found

[*] abc-1
[x] time: 0.024885 sec
[-] WA
output:
ARC

expected:
ABC


[*] sample-1
[x] time: 0.024794 sec
[+] AC

[x] slowest: 0.024885 sec  (for abc-1)
[x] max memory: 1.712000 MB  (for abc-1)
[-] test failed: 1 AC / 2 cases
```

追加したテストが実行され、ちゃんと落ちているのが分かります。

### 提出
なんと`oj`を利用すると1コマンドでコードを提出することができます。最近、AtCoderのコンテストで言語選択が変になっていてCEが無限に出たみたいな話がありましたが、これを使えばなんと**自動で言語を識別してくれる**ため、そのような事故はなくなります。更に、テキストをコピペしたりファイルを選択する手間も省け、その際に間違って別のコードを提出してしまった……といった事故もなくなります。

操作も超簡単で`oj s`を打つだけです。URLとか打たなくていいの？　って思うかもしれませんが、作成したフォルダの名前とURLが紐づいているために`oj`側がよしなにして正しい場所に出してくれます。言語の方も推測してくれます。マジで便利です。

ちなみに、フォルダ名が問題と一対一である必要はここの推測のためです。そうでない場合はここでもURLを指定する必要があります。

#### WSLを使っている人向けの話
`oj`では提出後、提出した際に自分の提出ページを自動的に開いてくれます。ですが、WSL側にちゃんとしたブラウザが入っていないと普段使わないようなブラウザが勝手に起動します。これを避けるためには`--no-open`オプションを付けると良いです。

`BROWSER`変数にちゃんとしたものを登録しておくとOKっぽいので、そちらの解決策でも構いません。詳細は[このissue](https://github.com/online-judge-tools/oj/issues/537)に書いてあります。ぼくもそのうちこっちの解決策に乗り換えます。

### まとめ
というわけで、`oj`の基本の使い方をまとめるとこのようになります。

* 問題用のフォルダを作成
* そのフォルダの中で`oj d URL`を叩いてサンプルをダウンロード
* 問題を解く
* `oj t`で確認
  * 不安なケースは自分で例を作って追加
* サンプルが通ったら`oj s`で提出

ブラウザと行き来する必要がほとんどなくなるので、かなり快適な競プロ環境になります。

`oj`はテスト・提出の自動化だけに留まらず、これらの環境をより便利にするような機能がいくつか含まれています。具体的には、特殊な出力例の検証機能やストレステスト機能が挙げられます。これらについては次に投稿する予定の記事で紹介したいと思います。







