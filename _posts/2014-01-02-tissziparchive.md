---
layout: post
title:  "TiSSZipArchive (SSZipArchive ラッパー) の紹介"
date:   2014-01-02
categories: titanium
---

Titanium ユーザー会サポート BBS を見ていると、 [ZIP モジュールが動かないという報告](http://support.titanium-mobile.jp/questions/3560)がありました。 GitHub に置いてある [ZipFile](https://github.com/TermiT/ZipFile) モジュールを見てみるとしばらくメンテナンスされていないようです。なので、 Titanium 3.2 以降向けに、モジュールを1つこしらえてみました。

**[TiSSZipArchive](https://github.com/ryugoo/TiSSZipArchive)** という名前のモジュールで、 [SSZipArchive](https://github.com/soffes/ssziparchive) という ZIP ファイルを操作するための iOS 向けライブラリを利用しています。このライブラリ自体は [nanapi TechBlog](http://nanapi.co.jp/blog/2013/12/11/qaapp_iphone_library/) で知りましたが、 ARC に対応していてメンテナンスもされているようなので Titanium 3.2 にはピッタリだと思いました。

詳しくは GitHub の README を読んでいただければと思いますが、

```javascript
var zip = require('net.imthinker.ti.ssziparchive');

// "archive" event handler
zip.addEventListener('archive', function (e) {
    if (e.success) {
        console.log('ZIP archive is success');
        console.log(e);
    } else {
        console.error('ZIP archive is failure');
        console.error(e);
    }
});

// Processing
zip.archive({
    target: '作りたい ZIP ファイル名 (フルパス)',
    files: [
        'a.png',
        'b.png',
        'c.png'
    ],
    overwrite: false
});

// "extract" event listener
zip.addEventListener('extract', function (e) {
    if (e.success) {
        console.log('ZIP extract is success');
        console.log(e);
    } else {
        console.error('ZIP extract is failure');
        console.error(e);
    }
});

// Processing
zip.extract({
    target: '解凍したい ZIP ファイル名 (フルパス)',
    destination: '展開先 (フルパス)',
    password: '',
    overwrite: false
});
```

こんな感じで使います。圧縮と解凍時に `addEventListener` でイベントを設定できるので、アクティビティインジケータなどの表示・非表示の切り替えに使えるのではないでしょうか？

例によって MIT ライセンスで、**[今回はバイナリファイルも GitHub に置いてあります](https://github.com/ryugoo/TiSSZipArchive/tree/master/dist)**ので、興味がある方は自身の責任でお使いください。