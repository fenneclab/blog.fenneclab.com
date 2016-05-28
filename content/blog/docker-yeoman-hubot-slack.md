---
title: Docker + Yeoman + Hubot + Slack で簡単bot作成
date: '2014-11-28T17:09:49.000Z'
draft: false
type: post
description: >-
  ご存知、Hubotをわりとモダンなやり方で作ってみたのでご紹介します。コピペで作れます。Dockerでいれるので環境が汚れません。
featured: docker-yeoman-hubot-slack-featured.png
featuredalt: ''
featuredpath: date
categories:
  - Development
aliases:
  - /docker-yeoman-hubot-slack
author: satoshun00
---

satoshun00です。
ご存知、[Hubot](https://hubot.github.com/)を、ちょっとモダンなやり方で作ってみたのでご紹介します。

実際にFennecLabチームで使用している`fennecbot`はこの手順で作りました。

Githubにリポジトリを公開しています。

https://github.com/fenneclab/fennecbot

<!--more-->

**pros**

- コピペで作れます
- Dockerでいれるので環境が汚れません

**cons**

- SlackとHubotがURLで連携するため、リモートの置き場所が必要です
  - 今回はさくらVPSで運用

今回はMac OS X(ローカル)とCentOS6(さくらVPS, 実運用)に入れました。

### Installation
##### Docker
[Docker Documentation](https://docs.docker.com/installation/#installation)を参考にインストールします。

**Mac OS X(ローカル)**

```
# インストーラーにてboot2dockerパッケージをインストール後
> boot2docker init
> boot2docker start
> $(boot2docker shellinit)
```

**CentOS6(さくらVPS, 実運用)**
```
> sudo yum install docker-io
```

##### Yeoman
[Yeoman](http://yeoman.io/)はnode.jsアプリケーションの雛形ジェネレータです。

雛形を作るだけなのでローカルのマシンに入れるだけでよいでしょう。

**Mac OS X(ローカル)**
```
> brew install node # nodeがインストールされていなければ…
> npm install -g yo
```

### Hubotアプリケーションを作成
[Hubot](https://hubot.github.com/)アプリケーションを作成します。

ローカルマシンにて以下のコマンドを実行します。

```
> npm install -g generator-hubot # Yeoman用、Hubotジェネレーター
> mkdir test-hubot && cd test-hubot
> yo hubot
```

対話モードでいろいろ訪ねられるので適当に答えます。

`Bot adapter`は`slack`にして下さい。

![yo hubot](/img/2014/11/docker-yeoman-hubot-slack-1.png)

デフォルトでHubotスクリプトがいくつか入っていますね。

![scripts](/img/2014/11/docker-yeoman-hubot-slack-2.png)

##### HubotDockerコンテナーの作成

HubotアプリケーションをDockerのコンテナーとして立ち上げます。

`test-hubot`ディレクトリに以下の`Dockerfile`を作成します。

```Dockerfile
FROM node:latest

# コンテナにHubotアプリケーションを持っていく
RUN mkdir /bot && cd /bot
ADD . /bot

EXPOSE 9999

WORKDIR /bot

# docker runした時の実行コマンド
CMD ["bin/hubot", "--adapter", "slack"]
```

Dockerイメージをビルドします。

```
> docker build -t test-hubot .
```

Dockerコンテナーを起動します。

```
# ローカルマシンではSlackとの連携が出来ないため`CMD`をオーバーライド
> docker run -v "$(pwd)":/bot \
    -it -p 9999:9999 \ # インタラクティブモードでコンテナーを起動
    test-hubot \
    bin/hubot # --adapterを指定せずデフォルト(Shellモード)にしておく
```

`docker run`のオプションに`-it`を指定したので、対話的にHubotが動けば成功です。(`Ctrl + C`でHubotを終了すると、コンテナーもstopします)

```
Hubot> hubot ping
Hubot> PONG
```

作成した`test-hubot`をgithubに`push`しておきましょう。

### Hubot - Slack連携

##### Slackの設定

[Slack](https://slack.com)のIntegration設定にて、`Hubot Integration`を追加し、項目を埋めます。

- `Hubot URL`: リモートのURL(今回だとさくらVPSのIP) + Hubotの待ち受けポート
  - ex) `10.0.0.3:9999`
- `Token`: 後で使うのでどっかにコピーしておきましょう
- `Descriptive Label`: 空欄 
- `Customize Icon`: 愛着のわく何かにしておくといいでしょう

##### Hubotをリモートで起動

さくらVPSで用意した、CentOSにて作業します。

```
# 先ほどpushしたHubotリポジトリをclone
> git clone git@github.com:your-org/test-hubot.git
> cd test-hubot
# 先ほどと同じ要領でDockerイメージをビルド
> docker build -t test-hubot .
```

Hubotの入ったDockerコンテナーを起動します。
(`Dockerfile`の`CMD`で指定したデフォルトの起動コマンドで`docker run`されます)

```
> docker run -e HUBOT_SLACK_TOKEN="[先ほど設定したToken]" \ 
  -e HUBOT_SLACK_TEAM="[Slackのチーム名]" \
  -e HUBOT_SLACK_BOTNAME="[botの名前]" \
  -e PORT=9999 \
  -d -p 9999:9999 \ # -d でdaemon起動
  -v "$(pwd)":/bot \
  test-hubot
```

Slackにて指定したbotに呼びかけてみましょう。

反応が返ってくれば成功です！

![cat me](/img/2014/11/docker-yeoman-hubot-slack-3.png)

### +α(& やり残したこと)

-  `docker run`したHubotへの変更の反映、再起動は？
  - 変更を`git pull` -> [`docker restart CONTAINERID`](https://docs.docker.com/reference/commandline/cli/#restart)
- 運用するリモートサーバーがない
  - [HubotはHerokuへのデプロイも推奨しているようです](https://github.com/github/hubot/blob/master/docs/deploying/heroku.md)
- 「`git pull` -> [`docker restart CONTAINERID`](https://docs.docker.com/reference/commandline/cli/#restart)が面倒くさい」
  - ローカルで作成したHubotのDockerイメージを[Docker Hub](https://registry.hub.docker.com/)に`push`し、運用するためのリモートサーバーにて`docker pull`するのもイケると思います
- 「会社のSlackだから勝手にIntegrationを追加できない」
  - SlackはIRCサーバーとしても使えます(これも[Slackの設定次第ですが](https://slack.zendesk.com/hc/en-us/articles/201727913-Connecting-to-Slack-over-IRC-and-XMPP))
  - Hubotの[IRCAdapter](https://github.com/nandub/hubot-irc)を利用するとよいでしょう
- Hubotの立ち位置
  - 個人が趣味プロではじめたHubotに、いつのまにか業務が依存しだしたら、Jenkinsとかチームが運用できる手段を真剣に考えた方がいいと思います
  - [Githubみたいな使い方](http://www.publickey1.jp/blog/13/githubboxenhubotdevops_day_tokyo_2013.html)を目指すなら運用チームがいないときついと思います
  - 私個人としては、Hubotはちょっとした業務補助ツール兼、おもちゃくらいです

### 参考
- [hubot+slackをIRCで動かす](http://qiita.com/mikesorae/items/b229a8cebe1880ca52b9)
- [slackにHubotを導入(Heroku経由)](http://qiita.com/Katsumata_RYO/items/dc4543aa5827d4c3211c)
- [GitHub社内のDevOpsを支えるツール「Boxen」と「Hubot」（後編）～DevOps Day Tokyo 2013](http://www.publickey1.jp/blog/13/githubboxenhubotdevops_day_tokyo_2013.html)
