---
layout: post
title:  "TiExtendNW (MKNetworkKit ラッパー) の紹介"
date:   2013-12-16
categories: titanium
---

[Titanium™ Advent Calendar](http://www.adventar.org/calendars/78) 16 日目の記事です。 15 日目は @donayama さんの [Ti Weekly Clips 2013年総集編 (前編)](http://ti-news.hatenablog.jp/entry/2013/12/15/000018) でした。ポエムとか書いてないで少しは役立つ情報を書こうということで、作ってる途中のモジュールを紹介したいと思います。

### どんなものか

Titanium には HTTPClient というシンプルな HTTP 通信用の API が用意されています。 XMLHttpRequest のような形で書く事ができて、 URI からリソースを取得したり、テキストや画像をポンポンと送信できる意外と高機能な API です。

Objective-C で開発する場合、 [AFNetworking](http://afnetworking.com/) や [MKNetworkKit](https://github.com/MugunthKumar/MKNetworkKit) 、 [RestKit](http://restkit.org/) などの強力な HTTP 通信用のライブラリが有名で、最近では iOS 7 から [NSURLSession](http://dev.classmethod.jp/references/ios-nsurlsession-1/) という新しい API が提供されるなど、どんどんと進化を続けています。

一方で、 iOS 版の Titanium では古くから [ASIHTTPRequest](http://allseeing-i.com/ASIHTTPRequest/) という iOS にある [CFNetwork](https://developer.apple.com/library/mac/documentation/networking/conceptual/cfnetwork/Concepts/Concepts.html) のラッパーライブラリを、 Titanium 向けにさらに糖衣して提供しているわけですが、この ASIHTTPRequest は既に 2 年以上更新されていません。

Titanium 3.2 では iOS 向けのモジュールに ARC が使えるようになった (らしい) ので、 ARC 対応をかかげている HTTP 通信系のライブラリをモジュール化することが現実的になりました。そこで、 MKNetworkKit をラップしてみました。

### どこにあるのか

[GitHub にリポジトリを用意しました](https://github.com/ryugoo/TiExtendNW)。 MIT ライセンスで公開しています。 [Titanium 3.2 の RC 版](http://www.appcelerator.com/blog/2013/12/3-2-0-rc-of-sdkstudio-now-available/)で開発していて、 GA 版が登場したらそちらに差し替える予定です。まだまだ開発途上なので、プレビュー版ぐらいの気持ちで見てください。

Titanium 3.2 以降を想定して作っています。なので、 Titanium 3.1.x 以下でこのモジュールを動作させた場合、そもそも動くのか、動いたとしても ARC 非対応でメモリをガンガン食いつぶして大変なことになるのか、それも分かりません。

### どう使うのか

Titanium SDK 3.2 RC がインストールされた環境でリポジトリからコードをクローンしてきて、プロジェクトディレクトリにある `build.py` を叩けばモジュールがビルドされるはずです。バイナリ版を提供する予定は今のところありません。

`example/app.js` がサンプルのコードです。基本的には Ti.Network.HTTPClient と同じような感覚で使うことができます。

```javascript
var TiExtendNW = require('net.imthinker.ti.extendnw'),
    http = TiExtendNW.createHTTPClient();

http.open('GET', 'http://httpbin.org/get', {
    freezable: false,
    forceReload: false
});

http.onload = function (e) {
    Ti.API.debug('responseJSON');
    Ti.API.debug(http.responseJSON);
};

http.onerror = function (e) {
    Ti.API.error(e);
};

http.setRequestHeader('User-Agent', 'MKNetworkKit wrapper for Titanium (iOS) / Version 1.0');
http.setRequestHeader('X-ApplicationVersion', Ti.App.version);
http.send();
```

MKNetworkKit のラッパーですので、ネットワーク凍結機能 (freezable) と GET メソッドの自動キャッシュ機能を使うことができます (**が、作りたてほやほやのモジュールなので全くテストしてません**) 。

`open` メソッドの第 3 引数にオブジェクトで `freezable` と `forceReload` という真偽値型のプロパティをセットすることができます。 POST や PUT リクエストをするときに、圏外になるなどして通信が途切れた場合、自動的に凍結して、通信が回復したら自動で再リクエストしてくれる素晴らしい機能です。これを Titanium で使いたいがために、 MKNetworkKit をラップしようと思ったぐらいです。

`forceReload` は GET リクエストを行うときに、キャッシュを無視して新しいリクエストを投げたい場合に使います。わざわざクエリパラメータに `?rnd=<Date.now()>` な値を付加する必要はありません。

そして `onload` メソッド内では JSON を取得した際にデシリアライズした結果を得られる `responseJSON` というプロパティを使うことができます。 JavaScript 中で `JSON.parse` メソッドを叩かなくてもオブジェクトにアクセスできます。

**繰り返しますが、コード中ではこれらの設定を反映できるようにモジュールを組んでいますが、作りたてほやほやなので、全くテストしていません。なので、ネットワーク凍結やキャッシュ無視が動作するかはこれから確かめるところです。**

### やりたいけど、まだできてないところ

* `send` メソッドにオブジェクトを渡して、それを GET リクエスト時のクエリパラメータに変換する機能は実装途中です。一度作ったのですが、正しく動作しないことがあったので外しました。
* `send` メソッドに文字列や `Ti.Blob` 、 `Ti.Filesystem.File` オブジェクトを渡してそのまま POST する機能は未実装です。どう作るかもまだ考えてません。オブジェクトだけじゃダメですか？
* `responseXML` プロパティは libxml を読み込めない罠にはまったので保留しています。機能実装するかは悩んでいます。 XML 欲しいです？
* その他沢山、まだ Ti.Network.HTTPClient からしたら機能不足です。

### できないところ

* `timeout` プロパティは設定できません。 MKNetworkKit 自体にタイムアウト設定する API が無いっぽいのです。

### 〆

ユニットテストやパフォーマンステストなど、まだまだ信頼性を確保する面でも足りてませんが、強力な通信ライブラリを Titanium で使えるようになれば、 Titanium が得意とする Web API と連携するタイプのアプリ開発が捗りそうですよね。

Ti.Next が登場したら、このモジュールはそもそもお役御免になるのかもしれませんが、それもまだ先の話。是非ともこのモジュールを使ってみてください。まぁ、 Titanium 自体が NSURLConnection とか使ってくれれば嬉しいんだけどね！

### 17日目は？

まだ未定なんですよね。もしかしたら、私が [ChatWork Advent Calendar](http://www.adventar.org/calendars/210) と共作で書くかも!? そしたら 17/18 日目は私ということに…。