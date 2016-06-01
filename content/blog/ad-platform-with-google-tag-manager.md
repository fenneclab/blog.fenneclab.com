---
title: Google Tag Managerを使って画面サイズに応じた動的広告配信を実装する
date: '2015-12-25T17:45:26.000Z'
draft: true
type: post
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
categories:
  - Development
author: satoshun00
---

satoshun00です。

皆さん、GoogleTagManager使ってますか。


# GoogleTagManagerで広告配信タグを作成

GoogleTagMangerを構成する主な設定項目は「`変数`、`トリガー`、`タグ`」です。順に設定していきましょう。

## \#-1. ブレークポイント種別`変数`の作成

GoogleTagManagerで画面サイズの種類(デスクトップ、スマートフォン・タブレット)を判別するための変数を作成します。

メニューから`変数 > 新規`を選択し、下記画像のように項目を設定します。

![Variable](/img/2015/12/google-tag-managerwoshi-tutehua-mian-saizuniying-zitadong-de-guang-gao-pei-xin-woshe-ding-suru-1.png)

GoogleTagManagerにおける「[データレイヤー変数](https://support.google.com/tagmanager/answer/6106899?hl=ja&ref_topic=3441647)」は、クライアントのJavaScriptとGoogleTagManagerが`dataLayer`変数を通じてやり取り出来るJavaScriptオブジェクトのことです。

例では`dataLayer.push({breakpoint: 'desktop'})`のようにクライアントから送信された変数の中身を、GoogleTagManagerにて参照し、タグ配信条件を分岐させるのに利用します。

また、変数には他にページの参照元(リファラー)やページのURLといったよく使う変数や、CSSセレクターを使ってクライアントのタグ内の値を参照出来るDOM変数など便利な変数が用意されています。(詳しくは[変数](https://support.google.com/tagmanager/answer/6106899?hl=ja&ref_topic=3441647)リファレンスを参照して下さい)

## \#-2. 画面サイズ変更イベント`トリガー`の作成

GoogleTagManagerで画面サイズの変更イベントによって発火するトリガーを作成します。

メニューから`トリガー > 新規`を選択し、下記画像のように項目を設定します。

![Trigger1](/img/2015/12/google-tag-managerwoshi-tutehua-mian-saizuniying-zitadong-de-guang-gao-pei-xin-woshe-ding-suru-2.png)

スマートフォンの画面サイズになった時に発火するイベントも同様に作成します。

![Trigger2](/img/2015/12/google-tag-managerwoshi-tutehua-mian-saizuniying-zitadong-de-guang-gao-pei-xin-woshe-ding-suru-3.png)

GoogleTagManagerにおける「[カスタムイベント](https://support.google.com/tagmanager/answer/6106961?hl=ja&ref_topic=3441647)」は`dataLayer.push()`でクライアントからイベントが送信されると発火するトリガーを作成出来ます。

例では`dataLayer.push({event: 'media-screen-changed', breakpoint: 'desktop'})`のように、#-1で設定したブレークポイント種別変数を同時に送信し、GoogleTagManagerで発火条件として指定しています。

また、トリガーには他にユーザーのクリックイベントやページビュー(window.onLoadやDOMContentLoadedイベント)といった便利なトリガーが用意されています。(詳しくは[トリガー](https://support.google.com/tagmanager/answer/6106961?hl=ja&ref_topic=3441647)リファレンスを参照して下さい)

## \#-3. 広告配信`タグ`の作成

GoogleTagManagerでトリガーを条件に配信されるタグを作成します。

メニューから`タグ > 新規`を選択し、下記画像のように項目を設定します。

「配信するタイミング」の項目に#-2で作成した「イベント - 画面サイズ変更 - デスクトップ」トリガーを指定します。

カスタムHTMLとして仮に下記のコードを実行しています。

```html
<script>
  (function(d) {
    var element = d.getElementById('koukoku-waku-1');
    if (element) {
      // 配置先の広告枠を空にする
      while(element.hasChildNodes()){
        element.removeChild(element.lastChild);
      }
      // 広告を枠に設定
      var ad = d.createElement('div');
      ad.textContent = 'デスクトップの広告';
      element.appendChild(ad);
    }
  })(document);
</script>
```

![Tag1](/img/2015/12/google-tag-managerwoshi-tutehua-mian-saizuniying-zitadong-de-guang-gao-pei-xin-woshe-ding-suru-4.png)

スマートフォンの画面サイズになった時に配信するタグも同様に作成してください。

GoogleTagManagerにおける「[カスタムHTML](https://support.google.com/tagmanager/answer/6107167?hl=ja&ref_topic=3281056)」は任意のHTMLタグを配信出来ます。

例では`<script>`タグを作成し、広告枠の要素(#koukoku-waku-1がふられた要素)に広告要素を挿入しています。(実践では広告スクリプトのロードを行うことになるでしょう)

また、タグには他にGoogleAnalyticsへのイベント送信などのタグが用意されています。(詳しくは[タグ](https://support.google.com/tagmanager/answer/3281060?hl=ja&ref_topic=3281056)リファレンスを参照して下さい)

# GoogleTagManagerスニペットの追加

[設定とインストール](https://support.google.com/tagmanager/answer/6103696?hl=ja&ref_topic=3441530)を参考に、HTMLのBodyタグ内にGoogleTagManagerスニペットを追加して下さい。

# ブレークポイントイベントの設定

作成したGoogleTagManagerタグの配信をJavaScriptのソースからトリガーします。

画面サイズの変更に応じて動的にイベントを送信するため、それに適したモジュールを使用します。

ここでは[js-breakpoints](https://www.npmjs.com/package/js-breakpoints)モジュールを使い、CSSに定義されたブレークポイントに応じて画面サイズ変更のイベントが発火するようにします。

以下、ほぼ[モジュールのドキュメント](https://github.com/14islands/js-breakpoints#javascript-breakpoints)通りです。

※今回、sassやJavaScriptのimport部分は本質でないので、説明を端折っています。環境に応じて適宜読み変えて下さい。

## \#-1. [js-breakpoints](https://www.npmjs.com/package/js-breakpoints)モジュールをインストール
```sh
npm install --save --save-exact js-breakpoints
# もしくは
bower install --save --save-exact js-breakpoints
```

## \#-2. CSSにメディアクエリーブレークポイントを設定

`defineBreakpoint`ミックスインをincludeすることにより、要素の`:after`擬似要素に、`content: 設定した名前;`でコンテンツプロパティが設定されるようです。

```scss
@import '../../node_modules/js-breakpoints/breakpoints';

// ブレークポイント。960px以上をデスクトップとする
$desktop-breakpoint: 960px;

// デスクトップより小さいサイズのメディアクエリー
@mixin phone-and-tablet() {
  @media screen and (max-width: $desktop-breakpoint - 1) {
    @content;
  }
}

// デスクトップ以上のサイズのメディアクエリー
@mixin desktop() {
  @media screen and (min-width: $desktop-breakpoint) {
    @content;
  }
}

body {
  // 適当なブレークポイントの名称を設定する
  @include desktop {
    @include defineBreakpoint('DESKTOP_SCREEN');
  }
  @include phone-and-tablet {
    @include defineBreakpoint('PHONE_AND_TABLET_SCREEN');
  }
}
```

## \#-3. メディアクエリーブレークポイントイベントを発火させる

例によってBrowserifyでrequireを解決していますが、ここもほぼ[モジュールのドキュメント](https://github.com/14islands/js-breakpoints#javascript-breakpoints)通りです。(Browserifyを使用しない場合、`<script>`タグでモジュールをロードして下さい)

先のCSSで設定したブレークポイントの名称に対応する、nameプロパティのブレークポイントイベントが発火します。

※`dataLayer`にデータをpushするため、先で設定したGoogleTagManagerスニペットよりも後で下記JavaScriptをロードします

```js
require('js-breakpoints');

var bodyTag = document.body;
// 画面サイズがブレークポイントに到達した時に発火するイベント
// GoogleTagManagerで作成したイベント名、及びブレークポイント名をdataLayerオブジェクトにpushする
var pushMediaScreenChangedEvent = function(breakpoint) {
  dataLayer.push({
    event: 'media-screen-changed',
    breakpoint: breakpoint
  });
};

Breakpoints.on({
  // 先で設定したブレークポイントの名称
  name: 'DESKTOP_SCREEN',
  // 先でブレークポイントを設定した要素を指定
  el: bodyTag,
  matched: function() {
    pushMediaScreenChangedEvent('desktop');
  }
});
Breakpoints.on({
  name: 'PHONE_AND_TABLET_SCREEN',
  el: bodyTag,
  matched: function() {
    pushMediaScreenChangedEvent('phone-and-tablet');
  }
});
```

# 配信テスト

GoogleTagManagerの画面に戻って、「プレビューモード」でタグ配信をデバッグします。
プレビューモードはプレビューを実行したブラウザーにのみタグ配信が行われる便利機能です


