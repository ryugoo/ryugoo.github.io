---
layout: post
title: "第2回 かわいい Kotlin 勉強会に行ってきた #jkug"
date: 2014-7-06
categories: kotlin
---

「[第2回 かわいい Kotlin 勉強会](http://www.zusaar.com/event/12417003)」に行ってきました。 Kotlin (ことりん) とは IntelliJ IDEA で有名な JetBrains が開発を行っている JVM 言語で、 Scala や Groovy っぽさを出しつつも、速さや安全性を意識したとても「かわいい」言語です。後発の JVM 言語なので、他の JVM 言語のいいとこ取り + α な感じになってます。 Android 開発にも使える言語ということで最近ちょっとずつ手を出していて、勉強会が開催されるとのことだったので参加してきました。

### Kotlin の由来

Wikipedia によると、 Kotlin の名前の由来は

> コトリン島にちなんで命名された。コトリンは、開発の地サンクトペテルブルクに近いバルト海フィンランド湾にあり、全長約12kmの細長い島である。もともと Kotlin というのは湯沸かしのやかんを表すフィンランド語であり、Kotlin 言語のロゴマークもやかんである。

らしいのですが、日本語におけるその語感から「かわいい」感じの響きがあり、結果、かわいいのです (気にすんな) 。実際の言語としても Java の冗長なところを良い感じに取り除いてモダンな感じになっていたり、 [IntelliJ IDEA や Android Studio とサクッと連携](http://qiita.com/shoma2da/items/52567a7be21992c7abbf)できたりして使い始めの障壁が低めです。

### Hello, world

```kotlin
// Kotlin で Hello, world
package sample
fun main(args: Array<String>) {
    println("Hello, world!")
}
```

Kotlin の Hello, world はとてもシンプルですが、この中に Kotlin のかわいさが凝縮されているそうです。例えば Package 直下に関数が書けたり、 `main` という関数が基点となったり、仮引数の型宣言は後置だったり、式の最後にセミコロンがいらなかったり……です。

### Kotlin のかわいいところ (抜粋)

#### SAM 変換

勉強会の当日に簡単に予習をしたときに「お、これは良いな！」と思って、勉強会で正式名称を知ったのが SAM 変換です。例えば Android アプリを書く場合、 `Button` に対してクリックしたときのイベントハンドラを設定するには Java では

```java
// Java, Android のクリックイベントハンドラ
Button button = (Button) findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Log.d("Test", "Hello, world");
    }
});
```

こんな感じで書きます。 IntelliJ IDEA や Android Studio を使っていれば実際にはコードを一気に展開してくれるのでタイプ自体は面倒ではありませんが、コードを読むときに本質的ではない部分が多いことに辟易してしまいます。

上の処理の本質的な部分を言葉にすれば「ボタンをクリックしたらログに Hello, world が流れる」というものですが、動きとしては分かりきっているのに書かなければいけないコードが多いことが分かります。これを何とかしてくれるのが SAM 変換というらしいです。

```kotlin
// Kotlin, Android のクリックイベントハンドラ
val button = findViewById(R.id.button) as Button
button.setOnClickListener { view ->
    Log.d("Test", "Hello, world")
}
```

なんということでしょう。とんでもなくスッキリしてしまいました。 JavaScript でエレメントに `addEventListener` をしている感覚で書けてしまいますね。

```javascript
// Titanium, JavaScript のクリックイベントハンドラ
var button = Ti.UI.createButton({ title: 'Button' });
button.addEventListener('click', function (e) {
    console.log('Hello, world');
});
```

SAM とは Single Abstract Method の略で、 Android アプリ開発をしているとよく見かける、1つだけの抽象メソッドを持つインターフェースは SAM インターフェースと呼ぶそうです。この SAM についての詳しくは、勉強会で LT をされていた [@RyotaMurohoshi](https://twitter.com/RyotaMurohoshi) さんの [Qiita](http://qiita.com/RyotaMurohoshi/items/01b370f34a4bf96f5c39) に詳しく掲載されています。

この SAM 変換によって書いたコードがスッキリして、あとで読み返すときに辛くなさそうな点がかわいいですね。

#### null 安全

最近 Apple が Swift を発表して、コードの中に ? があったり ! があったりして「なんぞ!?」と思われた方も多いと思いますが、私はあれを見た瞬間「Kotlin のかわいさが認められたか」と思いました。

```kotlin
// 非 null 型は null を持てない
val str: String = null // ダメ
val str: String? = null // 大丈夫
```

実際に Kotlin で Android アプリのコードを書き始めると null 安全機能が若干煩わしく感じたりもするのですが、ぬるぽと極力お友達にならないためには有効に使っていきたい機能です。 Kotlin の場合、 [null を許容する型にアクセスする](https://sites.google.com/site/tarokotlin/chap5/sec54)には

```kotlin
// null チェック
val str: String? = "こんにちは"
if (str != null) {
    println(str.size) // 5
}

// 安全呼び出し
println(str?.size) // 5

// !! 演算子
println(str!!.size) // 5
```

があるそうです。 null チェックは良くある条件で確認する形ですが、安全呼び出しはぬるぽを発生させずに、ダメならば null を返してくれるアクセス手段で、 !! 演算子は「おれ null じゃないから！」と宣言してアクセスする手段ですが、 !! 演算子を使う場合にアクセス元が null だと、やっぱりぬるぽが発生するので気をつけなきゃですね。かわいいです。

### 勉強会に参加して

会社で Java を使ったネイティブ開発をはじめて、これまでの Titanium = JavaScript の世界とは違う良さ・悪さを感じていたところに出会った Kotlin は大変魅力的なものなのですが、如何せんどれぐらいのユーザ規模が居て、どれぐらい盛り上がっていて、どれぐらい使われているのかが判っていなかったところに勉強会が開催され、実際にいろんな方と話をしてみることで Kotlin の現状を何となくレベルの雰囲気で感じ取ることができました。

勉強会前日に Kotlin M8 が出て、いろんな問題が解決したり、まだ仕様にあるのに未実装な部分があったりと、これからの発展 **も** 楽しみな言語だと思います。特に個人的に楽しみなのが Android アプリ開発サポートを一緒に進めてくれているところで、 Android Studio との連携が簡単だったりするので Groovy 以上に Android アプリ開発の良い選択肢になるんじゃないかなという点です。

会場にいた方と話をしてみて「まだプロダクションに投入するには早いけど、Android SDK と紐付かないライブラリを書くにはもう使っても良いかも」という言葉に納得しました。まだまだ発展途上な言語ですが、 Android と一緒にさらに進化していくんじゃないかなと感じられたので、勉強会に参加できて良かったです。

### 参考リンク

* [第2回 かわいいKotlin勉強会を開催しました #jkug](http://taro.hatenablog.jp/entry/2014/07/06/103339)
* [第2回 かわいいKotlin勉強会 #jkug でLTしてきた](http://clomie.hatenablog.com/entry/2014/07/06/120107)
* [goat-reader-2-android-prototype](https://github.com/sys1yagi/goat-reader-2-android-prototype)
* [Kotlin (Wikipedia)](http://ja.wikipedia.org/wiki/Kotlin)
* [Kotlin (JetBrains)](http://kotlin.jetbrains.org/)
* [プログラミング言語 Kotlin 解説](https://sites.google.com/site/tarokotlin/)
* [Kotlin でリスナーやコールバックをスッキリと書く【関数リテラルと SAM 変換】](http://qiita.com/RyotaMurohoshi/items/01b370f34a4bf96f5c39)
* [Kotlin で Android アプリを Hello World!!](http://qiita.com/shoma2da/items/52567a7be21992c7abbf)