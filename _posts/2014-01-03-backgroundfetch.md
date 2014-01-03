---
layout: post
title:  "iOS 7 の Background Fetch を Titanium で使う"
date:   2014-01-03
categories: titanium
---

Titanium 3.2 は iOS 7 対応が強化されていて、 NSURLSession のブリッジ API である **URL Session Module** や新しいバックグラウンドモードの1つである **Background Fetch** が使えるようになりました。 Background Fetch はメッセージングアプリや SNS アプリとの相性が良い API です。名前の通り、アプリがバックグラウンド状態にあるときでもアプリ中の適当なメソッドを呼び出すことができます。

これからのアプリ開発で結構重要な API だと思うのですが、 Titanium 3.2 のドキュメントではサラリと対応したことだけが書かれていて、詳細については[公式ドキュメントのガイド](http://docs.appcelerator.com/titanium/latest/#!/guide/iOS_Background_Services)の方に置かれているぐらいなので目立っていません。今回はこの API を紹介します。

### Background Fetch を使う前に

Background Fetch の大前提ですが、**メソッド呼び出しのタイミングを自分で完全制御することはできません**。 Background Fetch は iOS がアプリの起動時刻や利用頻度を確かめながら、自動的にメソッドを呼び出すものです。 Titanium 側では最小間隔での呼び出しと、呼び出さないの2種類のタイミングが定数として用意されています。

* `Ti.App.iOS.BACKGROUNDFETCHINTERVAL_MIN`<br>
<small>最小間隔で呼び出すための定数</small>
* `Ti.App.iOS.BACKGROUNDFETCHINTERVAL_NEVER`<br>
<small>Background Fetch を無効にするための定数</small>

これらの定数を `Ti.App.iOS.setMinimumBackgroundFetchInterval` メソッドの引数に渡します。メソッドのデフォルトの引数は `Ti.App.iOS.BACKGROUNDFETCHINTERVAL_NEVER` なので、空で呼び出すと Background Fetch は無効になります。このメソッドには `Number` 型の値を渡すことができます。もしも任意の時間をベースにしたい場合には秒数の値を渡します。

Qiita の[こちらの記事を参照](http://qiita.com/griffin_stewie/items/8371c09059b3ba7bb202)すると、

> UIApplicationBackgroundFetchIntervalMinimum を指定していると  
> 最短 10 分程度  
> 最長 5 時間  
> くらいのインターバルで動いている感じだった。  
> インターバルを 2 時間に広げた場合には  
> 最短 2 時間前後 (2 時間切っている場合もあった)  
> 最長 22.5 時間  
> くらいのインターバルで動いている感じだった。

とありますので、時間を指定したとしても、かなりの差があるということは頭に入れておきましょう。

### 準備

まずはアプリに Background Fetch モードを許可させる必要があります。 Titanium 3.2 以降、プラットフォーム固有の設定は `tiapp.xml` に多くを書きます。 iOS であれば info.plist 、 Android であれば AndroidManifest.xml の内容を `tiapp.xml` 中に書く事になります。

まずは `tiapp.xml` の `<ios>` タグの `<plist>` セクション中に

```xml
<ios>
  <plist>
    <dict>
      <key>UIBackgroundModes</key>
      <array>
        <string>fetch</string>
      </array>
    </dict>
  </plist>
</ios>
```

このような形で値を追加します。これで準備完了です。 Titanium 側でコードを書いていきましょう。

### Background Fetch のイベントを購読する

Background Fetch を使うには `Ti.App.iOS` オブジェクトの `addEventListener` メソッドで `backgroundfetch` イベントを購読します。グローバルイベントなので、同じイベントを何度も登録するとその分メモリにオブジェクトが残り続けます。

特に理由が無ければアプリケーション起動中に1回だけ登録するようにしましょう。

```javascript
Ti.App.iOS.setMinimumBackgroundFetchInterval(
    Ti.App.iOS.BACKGROUNDFETCHINTERVAL_MIN
);
Ti.App.iOS.addEventListener(
    'backgroundfetch',
    function (e) {
        // バックグラウンドフェッチ時に行いたい処理を書く
        console.log('----- [Background Fetch] -----');
        console.log(e);
        
        // イベントハンドラの最後で呼び出し必須
        Ti.App.iOS.endBackgroundHandler(e.handlerId);
    }
);
```

これで Titanium 側で Background Fetch を使うための設定は終わりました。

### Background Fetch をデバッグする

Titanium で Background Fetch を使う場合、困るのはデバッグです。まさか iOS に任せてメソッドを呼び出してくれるのを待つわけにはいきません。幸い Xcode には Background Fetch をシミュレートするためのデバッグ機能があるので、これを活用します。

まずは Titanium CLI を使ってビルドオンリーオプション付きでビルドします。

```
ti build -p ios -b
```

次にプロジェクトディレクトリの `build/iphone/<プロジェクト名>.xcodeproj` を Xcode で開きます。今回は Xcode 5.0.2 を使っています。 Xcode 上から iOS 7 のシミュレータまたは実機でプロジェクトを起動します。起動後、 Titanium のコンソールと同様に Xcode 上のコンソールにログが流れ始めます。その後、シミュレータであれば `cmd + shift + h` を押して、実機であればホームボタンを押してアプリをバックグラウンド状態にしてください。

![Background Fetch](/image/bgfetch.png)

次に Xcode のメニューから Debug → Simulate Background Fetch を選択すると、擬似的に backgroundfetch イベントを呼び出すことができます。上の画像で使ったアプリケーションのサンプルコードを下に掲載します。

```javascript
(function () {
    'use strict';

    // Background Fetch の登録
    Ti.App.iOS.setMinimumBackgroundFetchInterval(
        Ti.App.iOS.BACKGROUNDFETCHINTERVAL_MIN
    );
    Ti.App.iOS.addEventListener(
        'backgroundfetch',
        function (e) {
            // バックグラウンドフェッチ時に行いたい処理を書く
            console.log('----- [Background Fetch] -----');
            console.log(e);

            // イベントハンドラの最後で呼び出し必須
            Ti.App.iOS.endBackgroundHandler(e.handlerId);
        }
    );

    // UI
    var navigation = Ti.UI.iOS.createNavigationWindow({
        window: Ti.UI.createWindow({
            backgroundColor: '#FFFFFF',
            title: 'Background Fetch'
        })
    });
    navigation.open();
}());
```

アプリを iOS 6 にも対応させる場合はコード中で分岐させましょう。

```javascript
var OS_MAJOR = parseInt(Ti.App.version.split('.')[0], 10);
if (OS_MAJOR >= 7) {
    // iOS 7 以上
} else {
    // iOS 6 以下
}
```

のように OS のバージョン判定を行ってあげると良いと思います。