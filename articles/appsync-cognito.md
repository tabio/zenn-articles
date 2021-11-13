---
title: "AppSyncでCognito認証"
emoji: "🍠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AppSync", "Cognito", "AWS"]
published: true
---


## 目的
AppSyncでCognito認証をしてみる
### AppSyncのGraphQL APIの用意
[前回記事](/articles/appsync-quick-start)にて作成済みのプロダクトを利用する
### Cognito: ユーザー作成
Cognitoのコンソール画面から適当なUserpoolを作って、テストで利用すためだけのユーザーを作成しておく
![](/images/appsync-cognito/cognito_user.png)

### Cognito: クライアントアプリ作成
テストなのでシークレットの作成はOFF
![](/images/appsync-cognito/cognito_client_app.png)

### AppSync: 認証方法をCognitoに変更
![](/images/appsync-cognito/setting_auth_mode.png)

### AppSync: クエリ画面
認証方法をCognitoに変更したためGraphQL APIが叩けなくなっている
![](/images/appsync-cognito/query_1.png)

### AppSync: ユーザープールにログイン
実行ボタン横のユーザープールにログインを押してログイン
![](/images/appsync-cognito/cognito_login.png)

### AppSync: ユーザープールにログイン
無事認証が通り、クエリが実行されていることを確認
![](/images/appsync-cognito/cognito_login_after.png)
