---
title: FacebookGraphAPIを使った投稿パターンと見え方まとめ
date: '2015-03-19T19:01:14.000Z'
draft: false
type: post
description: |-
  FacebookGraphAPIを使うとFacebookユーザーフィードへコンテンツの投稿を行うことが出来ます。
  Facebookフィードへの主な投稿パターンを整理しました。
featured: posting-facebook-feed-patterns-featured.jpg
featuredalt: FacebookGraphAPIを使った投稿パターンと見え方まとめ
featuredpath: date
categories:
  - Development
aliases:
  - /posting-facebook-feed-patterns
author: satoshun00
---

satoshun00です。

[Facebook Graph API v2.2](https://developers.facebook.com/docs/graph-api)を使うと、Facebookユーザーフィードへコンテンツの投稿を行うことが出来ます。

Facebookにはフィード投稿の方法がいくつかあり、ユーザーへのフィード上の見え方がそれぞれ異なります。

主な投稿パターンを以下3つに整理しました。

\#-1. フィードポスト パターン

<img src="/img/2015/03/posting-facebook-feed-patterns-5.png" width="50%" />


\#-2. Custom Open Graph パターン

<img src="/img/2015/03/posting-facebook-feed-patterns-13.png" width="50%" />

\#-3. Common Open Graph パターン

<img src="/img/2015/03/posting-facebook-feed-patterns-15.png" width="50%" />

※この記事では実装・SDKの利用には触れていません

<!--more-->

# \#-1. フィードポスト パターン

ブログ記事やWebサイトの一般的なシェアはこれが簡単でよいでしょう。

APIドキュメント: [POST /v2.2/me/feed](https://developers.facebook.com/docs/graph-api/reference/v2.2/user/feed#publish)

この方法では以下２種類の表示を得ることが出来ます。

- アクションリンク付き表示
- 場所・タグ付け表示

(「アクションリンク付き表示」と「場所・タグ付け表示」を共存させることが出来ませんでした(※理由は要調査))

## \#1-a. アクションリンク付き表示

「いいね！」「コメント」の横に、「今すぐダウンロード！ (AppStoreへのリンク)」のようなリンク(アクションリンク)を追加できます。

APIリクエスト例:

![アクションリンク付き表示-APIリクエスト例](/img/2015/03/posting-facebook-feed-patterns-4.png)

表示:

![アクションリンク付き表示-表示](/img/2015/03/posting-facebook-feed-patterns-5.png)

## \#1-b. 場所・タグ付け表示

投稿に関連するFacebookページ(場所)やユーザー(フレンド)をタグ付け出来ます。

APIリクエスト例:

![場所・タグ付け表示-APIリクエスト例](/img/2015/03/posting-facebook-feed-patterns-6.png)

表示:

![場所・タグ付け表示-表示](/img/2015/03/posting-facebook-feed-patterns-7.png)

投稿にはpublish_actionsの権限が必要です※[後述](#publish_actions)

# \#-2. Custom Open Graph パターン

Facebookでは[オープングラフ](https://developers.facebook.com/docs/sharing/opengraph)の機能により、ユーザーのアプリ上での行動を表現する事が出来ます。

> ◯◯ さんが △△ で □□ を ×× しました。

このセンテンスの集まりとつながりをFacebookではOpen Graphと読んでいます。

- アクター: 行動の主体となるユーザー
- アプリ: あなたの開発したアプリ
- オブジェクト: アクターの行動の対象
- アクション: アクターの行動
- ストーリー: オブジェクトとアクションを組み合わせたセンテンス

Open Graphにおいて開発者は_オブジェクト_・_アクション_・_ストーリー_を独自に定義出来ます。

![Custom Open Graph パターン-例](/img/2015/03/posting-facebook-feed-patterns-8.png)

では、実際に[やきそば弁当](https://www.maruchan.co.jp/products/search/42.html)を食べるストーリーを作成してみます。

※あらかじめ[デベロッパーページ](https://developers.facebook.com/apps/)よりアプリケーションの作成を行って下さい

## a. 名前空間の設定

**MyApp > Settings**

namespaceに`satoshun-test`を設定します。

## b. オブジェクトの定義(やきそば弁当)

**MyApp > Open Graph > Object Types > Add Object Type**

`yakisoba_bento`オブジェクトを作成します。

## c. アクションの定義(食べる)

**MyApp > Open Graph > Action Types > Add Action Type**

`eat`アクションを作成します。

詳細設定より、Capabilitiesを変更します。

- Tags: フレンドのタグ付けを許可します
- UserMessages: アクションの投稿時にユーザー入力のメッセージを許可します
- UserGeneratedPhoto: アクションの投稿時にユーザーの画像アップロードを許可します
- Place: 場所のタグ付けを許可します
- Explicitly Shared: 明示的なシェアを有効にします(※[後述](#explicit_share))

![Custom Open Graph パターン-アクションの定義-Capabilities](/img/2015/03/posting-facebook-feed-patterns-9.png)

## d. ストーリーの定義(やきそば弁当を食べる)

**MyApp > Open Graph > Stories > Add Custom Story**

アクションに`Eat`、オブジェクトに`Yakisoba Bento`を選択します。

**MyApp > Open Graph > Stories > Edit Attachments**

レイアウトと説明(キャプション)を設定します。

キャプションにはオブジェクトのプロパティを挿入できます。

![Custom Open Graph パターン-ストーリーの定義-Edit Attachments](/img/2015/03/posting-facebook-feed-patterns-10.png)

## e. センテンスの翻訳

今のストーリーのままでは全て英語の表記になってしまうため、翻訳を行います。

**MyApp > Localize > Translate Open Graph Story**

必要に応じてセンテンスを翻訳します。

![Custom Open Graph パターン-センテンスの翻訳](/img/2015/03/posting-facebook-feed-patterns-11.png)

※１つのストーリーに対し、翻訳すべきセンテンスが１００以上あり、管理が大変そうです…

## g. 投稿する

APIリクエスト例:

![Custom Open Graph パターン-投稿する-APIリクエスト例](/img/2015/03/posting-facebook-feed-patterns-12.png)

ユーザーのフィードにこのアクションを流すには、`fb:explicitly_shared`(明示的なシェア※[後述](#explicit_share))を`true`にする必要があります。

`yakisoba_bento`フィールドにはOpen GraphオブジェクトへのURLを指定します。

Open GraphオブジェクトはOGP準拠のHTMLコンテンツである必要があります。

(参考: [BasicなOGPタグ](https://developers.facebook.com/docs/sharing/webmasters#basic)、[OGPのデバッガー(Facebook公式)](https://developers.facebook.com/tools/debug/og/object/))

```yakisobabento.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <meta property="fb:app_id" content="450643005091501" /> 
    <!-- og:typeは {namespace}:{Object}の形式 -->
    <meta property="og:type" content="satoshun-test:yakisoba_bento" /> 
    <meta property="og:url" content="https://s3-ap-northeast-1.amazonaws.com/yakisobabentos/yakisobabento.html" /> 
    <meta property="og:title" content="タイトル" /> 
    <meta property="og:image" content="https://s3-ap-northeast-1.amazonaws.com/yakisobabentos/yakisoba.jpeg" /> 
    <meta property="og:description" content="説明文" />
    <meta property="og:site_name" content="サイト名" />
</head>
<body>
    おいしいお
</body>
</html>
```

表示:

![Custom Open Graph パターン-投稿する-表示](/img/2015/03/posting-facebook-feed-patterns-13.png)

## ※審査に関して

Custom Open Graph パターンで作成したオリジナルのアクションは、プロダクション環境で運用する場合、Facebookによる審査が必要になります。

審査開始前に、作成したアイテムを審査対象に追加するのを忘れないようにして下さい。

(Facebookフィードへの投稿にはpublish_actionsの権限が必要なため、どのみち審査を受ける必要はあります)

# \#-3. Common Open Graph パターン

FacebookGraphAPIにはあらかじめいくつかの定義済みオブジェクトとアクションが用意されています。

Custom Open Graph パターンに比べ、オブジェクトの管理と翻訳にかかるコストをおさえられたり、審査の項目が少なくなったりするなどのメリットがあります。

(参考: [定義済みセットの一覧が見られます](https://developers.facebook.com/docs/reference/opengraph))

「動画を観る」アクションを投稿してみます。

`video.watches`APIリクエスト例:

![Open Graph パターン-APIリクエスト例](/img/2015/03/posting-facebook-feed-patterns-14.png)

ユーザーのフィードにこのアクションを流すには、`fb:explicitly_shared`(明示的なシェア※[後述](#explicit_share))を`true`にする必要があります。

`video`フィールドにはOpen GraphオブジェクトへのURLを指定します。

Open GraphオブジェクトはOGP準拠のHTMLコンテンツである必要があります。

(参考: [BasicなOGPタグ](https://developers.facebook.com/docs/sharing/webmasters#basic)、[OGPのデバッガー(Facebook公式)](https://developers.facebook.com/tools/debug/og/object/))

```video.html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <meta property="fb:app_id" content="450643005091501" /> 
    <meta property="og:type" content="video.movie" /> 
    <meta property="og:url" content="https://www.youtube.com/watch?v=CeNTHATp5eU&list=FLw1_5CLspYMVjEjQ2GSjWaw" /> 
</head>
<body>
    
</body>
</html>
```

`video.watches`表示:

![Common Open Graph パターン-表示](/img/2015/03/posting-facebook-feed-patterns-15.png)

# publish_actionsパーミッション <a id="publish_actions"></a>

今回のようにAPIコールでユーザーのタイムラインへ投稿を行う場合、"publish_actions"のパーミッションが必要です。

アプリが"publish_actions"のパーミッションをユーザーに要求するにはFacebookの審査が必要です。(参考: https://snowadays.jp/2014/05/2762)

※ただし、Share Dialogを使えば"publish_actions"のパーミッションなしで、アクションの投稿が可能です

# Explicit share(明示的なシェア) <a id="explicit_share"></a>

Explicit share(明示的なシェア)とは、ユーザーがダイアログ等を使って明示的に投稿することを指します。

反対にImplicit share(暗黙的なシェア)とは音楽プレイヤーでいま聴いている曲をシェアし続けるような、アプリのフロー上でシェアを行うことを指します。

昨年からFacebookはスパム対策の一環としてImplicit shareを非推奨とし、ユーザーのタイムライン上に出ないようにしています。(参考: https://developers.facebook.com/blog/post/2014/05/27/more-control-with-sharing/)

ユーザータイムラインへの投稿を行いたい場合はExplicit shareを使いましょうという話です。

# Pre-fill問題について

[Facebookプラットフォームポリシー](https://developers.facebook.com/docs/apps/review/prefill)によれば:

> Pre-fill the user message parameter with any content the user didn't enter themselves, even if they can edit or delete that content before sharing.

ユーザーがシェアする文章に、あらかじめ何かしらの文章が入力されていてはならないとのことです。

![Pre-fill問題について](/img/2015/03/posting-facebook-feed-patterns-16.png)
赤枠で囲った部分にユーザーが入力した内容の文章以外のものを挿入しないよう注意して下さい。
