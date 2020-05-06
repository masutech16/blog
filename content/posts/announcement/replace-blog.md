---
title: "ブログを移転しました + GitHub Actionsを使ったHugo製サイトの自動デプロイ"
date: 2020-05-06T23:10:35+09:00
slug: replace-blog
draft: false
tags: [Development, other]
---

こんにちは。この度ますぐれメモを移転しました。
GitHub Actionsの方に興味がある方は序盤は適当に読み飛ばしてください。

# 事の始まり
移転のきっかけは先日上げた[ブログ記事](../../compro/automate2)でした。online-judge-toolsを使った精進環境の自動化記事を書いたので、競技プログラミング用のツイッターアカウントで宣伝したんです。そしてリンクが正しいかをチェックするために自分で踏んでみると、なんとTwitterから安全でないリンクであると判定されてまともに流入できなくなっていました。

(今スクショを取るためにもう一回踏んでみたら、もう大丈夫になっていました。Twitterに直訴すると効くようです)

これではせっかくブログを書いても誰も見てくれない！　困った！　となりました。

そこで、初めはドメインを変更しようとしました。Twitterのブラックリストの方式が分かりませんが、IPで弾いていなければドメインを変更するだけである程度の対策になると考えたからです。

[このブログ最初の記事](../opening)に書いた通り、このブログは以前はNetlifyというところにホスティングしていました。これは自分で取得したドメインを使えるので、この機能を利用してドメインを変更しようとしたんですね。ここでblog.masutech.workを割り当てようとしたのですが、Netlify上でサブドメインだけを扱う場合のHTTPS化がよくわからず、挫折しました。

で、じゃあもうせっかくサーバー借りてるしそこに載せちゃうか！　というノリで始めたのが今回の移転企画です。

# 移転先とそれに纏わる技術
現在このブログはConoHaのVPSに載っています。ConoHaなのは大学のサークルでよく使っていたため勝手をよくわかっているからです。料金体系が分かりやすく、月額で固定の金額しかかからないので何も考えなくてOKなのがいいところです。

以前Netlifyにホスティングしていた理由は、GitHubのpushにhookした自動デプロイができたからでした。というわけで移転したとしてもこの機能は抜きたくありません。

自動デプロイに使われるサービスとしてはGitHubのwebhookやCI系のサービスなどがあると思うのですが、今回は試しにGitHub Actionsを使ってビルド + コンテンツのデプロイを行うようにしてみました。

この方式の利点は2つあります。1つはGitHubと自分のサーバーだけで完結することです。現状テストもなく外部のCIツールを導入するほどの規模ではないので、GitHubだけで完結するのは嬉しいです。

もう1つは自鯖にHugo環境を作らなくてもいいことです。Dockerが入っているので入れるだけといえば入れるだけなんですが、コンテンツ配信サーバーでビルドするのはなんとなく気持ち悪いので、その責任をGitHub側に寄せられるのはいいなとなりました。

## GitHub Actionsについて
GitHub ActionsはGitHub上のイベントに紐づいてCI/CDをサポートしてくれる仕組みです。Actionと呼ばれる一塊の操作を組み合わせることで柔軟な操作ができます。例えばTypeScriptのコードをテストするアクション、コンパイルするアクション、コンテンツをデプロイするアクション、みたいな感じで組み合わせます。どんな操作をするのかというのはレポジトリの`.github/workflows`以下に記述されているyamlファイルを読むことで確認できます。

これの良い点としては、このActionの詳細をユーザーは知る必要がないという点が挙げられます。基本的にはドキュメントを読んで設定や必要な変数を渡す記述をするだけで使えます。Actionの詳細自体は別のレポジトリで管理されており、内部が気になれば覗きに行くことも出来ます。更にActionは誰でも作成することができるので、自分で作ったActionを自分で使う、みたいなことも可能です。ドキュメントが結構充実していたので、覗きに行けば大体したいことはできるかな～という印象でした。

今回は[Hugo Setup](https://github.com/peaceiris/actions-hugo)というActionと[rsync-deployments](https://github.com/oncletom/rsync-deployments)をforkして編集した[自作Action](https://github.com/masutech16/rsync-deployments)を利用しています。workflowを記述したファイルはこんな感じです。

```main.yaml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
      with:
        submodules: true
        fetch-depth: 0

    # Build
    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.68.3'

    - name: Build with Hugo
      run: hugo --minify

    - name: Rsync deployments
      uses: masutech16/rsync-deployments@v1
      with:
        USER_AND_HOST: gha@blog.masutech.work
        DEST: /srv/blog/
        SRC: public/
        RSYNC_OPTIONS: -rlOtcv --delete --exclude node_modules --exclude '.git*'
      env:
        DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

使ってみた感想としては、やっぱり自分でActionをサクッと作れるのが魅力的でした。これはほぼ他人のコードなのでmarketplaceには投げていませんが、自作したActionは公開して全ユーザーに使ってもらうこともできます。

一方、まだGitHub Actionsは成熟しているとは言い難く、今後もupdateで色々と壊れる可能性があります。実際、rsyncのActionはそれが原因で動かなかったため自前で編集したという経緯があります。安定性が求められるサービスに対してはまだまだ使えない、というのが個人的な感想です。

# 今後について
最近は競技プログラミングについての話題がメインですし、多分今後もこの傾向は続くと思います。それはそれとして、開発にも興味があるのでこういう話は時々は出てくると思います。ゆるゆると興味のある部分を見ていただければ幸いです～。

コメントなどありましたら[@masutech16](https://twitter.com/masutech16)もしくは[@masutech_compro](https://twitter.com/masutech_compro)までメッセージをいただけると嬉しいです！

