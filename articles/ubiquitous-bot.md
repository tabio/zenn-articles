---
title: "LambdaとAurora Serverlessでユビキタス言語管理用のSlack Botを作ってみた"
emoji: "🍤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Bolt", "AuroraServerless", "ServerlessStack", "Slack", "Lambda"]
published: true
---

# はじめに

チームに新しいメンバーが増えると、チームで利用されている言語の理解が難しいという声が毎回上がってきていました
`共有フォルダにスプレッドシートで管理する` や `Slackにチャンネルを作りそこに溜めていく` という案が出されたものの、いつの間にかメンテナンスされなくなり、また新しいメンバーが入って来た時に同じ問題が掘り起こされるという状態でした
そんな中で、[こちらの記事](https://zenn.dev/ryo_kawamata/articles/tell-me-bot-slack-app)を見つけ、自分のいるチームでもやってみようと思ったのがきっかけです
また、BoltやAurora Serverlessを勉強する良い機会だなと思ったのもモチベーションに繋がりました

# できたもの

### Slackのコマンドとして呼び出す

![](/images/ubiquitous-bot/slack_1.png)
### 存在しないキーワードの場合は登録ボタンが表示されます

![](/images/ubiquitous-bot/slack_2.png)

### キーワードの登録用のモーダルの表示

![](/images/ubiquitous-bot/slack_3.png)

### 登録完了

![](/images/ubiquitous-bot/slack_4.png)

### 登録したキーワードの場合は、編集と削除ができます

![](/images/ubiquitous-bot/slack_5.png)

### あいまい検索にも対応しています

![](/images/ubiquitous-bot/slack_6.png)

# 設計図

![全体設計](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/architecture.png?raw=true)

# 利用した主な技術

- [Bolt](https://slack.dev/bolt-js/ja-jp/concepts)
  - Slack API を使いやすくするための Node.js フレームワーク
- [serverless-express](https://github.com/vendia/serverless-express)
  - Boltからリクエストを良しなに吸収してくれるミドルウェア(Lambda上で動かしています)
- [Serverless Stack](https://serverless-stack.com/)
  - IaCツール。[CDK](https://github.com/aws/aws-cdk)のラッパーで、ローカル環境での開発に便利。
- [Aurora Serverless](https://aws.amazon.com/jp/rds/aurora/serverless/)
  - リクエストが来ない時は自動でインスタンスを一時停止してくれるので、コスト面でGood

# Serverless Stackを利用したデプロイ

```sh
git clone https://github.com/tabio/slack-bot-ubiquitous.git
cd slack-bot-ubiquitous
npm install
npx sst deploy # AWSの設定を事前に行っておく必要あり
```

# Slackアプリを使うための準備

Boltを利用するためにトークンやコマンド利用設定が必要になります

### Signing Secretの取得

Basic InformationのApp Credentialsの中にあります

![](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/signing-secret.png?raw=true)

### Bot User OAuth Token

OAuth & Permissionsの中にあります

![](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/slack-bot-token.png?raw=true)

# SlackからAPI Gatewayにリクエストできるようにする

API Gatewayのエンドポイントを入力してSlackとBoltがやり取りできるようにします
### Interactivity & Shortcuts

![](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/interactivity-shortcuts.png?raw=true)

### Event Subscriptions

![](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/event-subscription.png?raw=true)

# DBの準備

Aurora ServerlessへはAWS Console -> RDS -> クエリエディタ を使って接続します

![](https://github.com/tabio/slack-bot-ubiquitous/blob/main/images/query-editor.png?raw=true)

ユビキタス言語登録用のテーブルを作成します
　
![](/images/ubiquitous-bot/aurora.png)

# おわりに

AWS環境上で動くSlackアプリを作成することができました
Slack上での設定が色々あり大変だったこともあり、今回備忘録がてら記事化してみました
気になるところとしては、Aurora Serverlessを利用しているため、リクエストが無いとDBが止まってしまいます
そのため、使いたい時に直ぐ使うことができない場合があります
そこは、実際に運用してみてRDSのインスタンスに切り替えるか検討してみたいと思います


# その他

[こちら](https://github.com/tabio/slack-bot-ubiquitous)にソースコードをおいておきましたので良かったら見てやってください🙏

