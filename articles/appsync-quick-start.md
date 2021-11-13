---
title: "AppSyncでHelloWorld"
emoji: "🌰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AppSync", "AWS"]
published: true
---

## 目的

[AppSyncのクイックスタートガイド](https://docs.aws.amazon.com/ja_jp/appsync/latest/devguide/quickstart.html)でサンプルスキーマを起動しGraphQL APIを作成してみる

### AppSyncのAPI作成

AppSyncのConsoleからAPI作成ボタンを押すと以下の画面が表示されるので下段のサンプルプロジェクトから開始するでイベントアプリを選択し作成ボタンを押す
![](/images/appsync-quick-start/appsync_1.png)

### リソースを作成

デフォルトで設定してあったもので作成ボタンを続行する
![](/images/appsync-quick-start/appsync_2.png)
### クエリ

しばらくするとAppSyncのリソースがデプロイされた
左のメニューからクエリを選択
Mutation(CreateEvent)とQuery(ListEvents)がそれぞれ定義済みの状態になっていた
実行ボタンを押すとどっちを実行するか選択できる
Mutationを実行してからQuery側を実行した後のキャプチャが下図

![](/images/appsync-quick-start/appsync_3.png)

### データソース

このサンプルプロダクトはDynamoにデータが書き込まれる仕組みになっていた
データソースの画面を開くと2つのDynamoテーブルがあり
![](/images/appsync-quick-start/appsync_4.png)
さきほどのMutationでCreateEventしたのでAppSyncEventTableにデータが登録されていた
![](/images/appsync-quick-start/appsync_5.png)
### クライアントツールからクエリの発行できるか確認

[Altair GraphQL Client](https://altair.sirmuel.design/)を使って動作確認してみる
このサンプルプロダクトのデフォルトの認証がAPI Keyタイプになっていた
AppSyncのConsoleからAPIキーをコピーし
Altair GraphQL Clientのヘッダー設定で **x-api-key** を設定して確認した
![](/images/appsync-quick-start/appsync_client_1.png)
![](/images/appsync-quick-start/appsync_client_2.png)