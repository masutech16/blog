---
title: "Kotlinで簡単なタイマーアプリを作った"
date: 2018-12-05T23:54:06+09:00
draft: false
slug: timerapp
tags: [Development]
---

# Kotlin で簡単なタイマーアプリを作った

こんにちは。ますぐれです。本日はこの前の休日でさくっと簡単なアプリを作った話をします。

タイマーアプリなんて巷にあふれているので作らなくてもよかったのですが、 Kotlin に触るのがほぼ初めてだったので練習ということで作りました。なんでタイマーアプリなのかというと、アルバイトで塾講師をしているので、複数人の生徒を見るときに使えるかなあと思ったからです。

こんな感じのアプリです。



{{< figure src="/images/doubleTimer.jpg" title="Screenshot" class="center" width="80" >}}


この記事はアプリのソースコードを見返しながら適当にコメントをしている記事です。 Kotlin のいいところをまとめるとか、 Android アプリの開発のときに気づいたことなども多少は書きますがそれは主題ではないです。

ソースコード自体は[ここ](https://github.com/masutech16/DoubleTimer)に公開しているので、気になった人は見てみてください。バグとかも多分残っていると思いますが、個人で利用する分には好き勝手してもらって大丈夫です。設計とかなんもわからんのでダメ出しとかあれば [Twitter](https://twitter.com/masutech16) か GitHub でお願いします。



## 環境

* Windows 10 home ( 64 bit )
* Android Studio
* Kotlin 1.3



## 実装

核となるクラスは実際にタイマーを動かす `MyCountDownTimer` と、実際の時間を管理する `TimeKeeper` です。



### MyCountDownTimer

Android の方で `CountDownTimer` というクラスがあることを知っている人は察せると思うのですが、それを継承していたクラスです。最終的には内部に継承したクラスを持ち、それを管理するクラスになったので名前が体を表していません。

関数を引数に取れたりしたため `onTick()` 等で呼ぶメソッドを外部から登録できる形にしました。結果として画面の描画とかを分離できてよかったです。下が `onTick()` の実装です。 `tickListenerFuncList`というのが `Long -> Unit` のリストで、こんな感じで登録したメソッドを呼び出せていいなあって感じでした。

```Kotlin
override fun onTick(millisUntilFinished: Long) = tickListenerFuncList.forEach {it(millisUntilFinished)} // デフォルトの引数の名前がitなので、特にそれを明示しなくてもよい
```



### TimeKeeper

時間の管理をする人です。そんな大それたことはしていません。一回 Android 開発でもテストコードを書いてみたかったので、 Unit テストしやすそうな形に切り出したらこうなりました。

`import android.widget.TextView` をしていますが、これも過去の名残ですね。消し忘れです( Android Studio にここら辺をよしなにしてくれる機能ありそう)

Kotlin は Scala と同じで if 式なのでコードがとっても綺麗になってよいです。これは時間を追加する `addTimeBySeconds(Int) : Unit` のコードですが、下のようになります。

```Kotlin
    fun addTimeBySeconds(addingSeconds : Int) {
        val addingMillSeconds = addingSeconds * 1000
        countTime = if (addingMillSeconds + countTime < 0) 0 else countTime + addingMillSeconds
    }
```

楽で良いです。 Java は最初に学習した言語だったのですが、後続の奴を触ってしまうともう Java には戻りたくなくなりますね……。

テストも難なく導入出来ました。最近 JS のプロジェクトに本格的にテストを導入しようとして四苦八苦していたのでめっちゃ感動しました。



### その他

関数をトップレベルで宣言できるため、 Utils みたいなクラスを作らなくてよくなったのは個人的にはとてもうれしいです。

Util ファイル自体は存在しているのですがクラスではないため、呼び出しの時が結構スマートに決まってよいです。

```kotlin
package com.example.masutech.doubletimer

fun formatTime(num: Long): String { // formatTime(hoge)で呼べる
    val minute = num / (1000 * 60)
    val minuteStr = formatNumber(minute)

    val second = (num / 1000) % 60
    val secondStr = formatNumber(second)

    val mill = (num / 10) % 100
    val millStr = formatNumber(mill)

    return minuteStr.plus(":").plus(secondStr).plus(":").plus(millStr)
}

private fun formatNumber(num: Long) : String = if (num < 10) "0$num" else "$num"
```



## 感想

Android 開発は分からなさ過ぎてすごい昔に挫折したんですよね～。従ってこれを始めるときも若干苦手意識があったのですが、詰まることなくスムーズに行けたので成長を感じました。

あといくつかアプリとして作りたいものがあるので、日曜エンジニアリングでぼちぼち進めていきたいと思います。ちょっと形になったら今度は報告だけではなく制作過程も記事に書いていこうかなあと思います。


