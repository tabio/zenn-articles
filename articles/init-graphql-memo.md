---
title: "はじめてのGraphQLの備忘録"
emoji: "📕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GraphQL"]
published: true
---

## はじめに

はじめてのGraphQLを読んで、章ごとに学んだことをメモする
途中で挫折することなく読み切るための備忘録

https://www.amazon.co.jp/dp/487311893X


## 2章：グラフ理論

TBD


## 1章： GraphQLへようこそ

- Facebookが作成したクエリ言語
  - 最初はRESTで作っていたが、作り直し時に性能の課題とデータ構造の要件を満たす解決策として誕生
  - 今では広く普及しGithubのAPIもv4からGraphQLになるほど浸透
- RESTのデメリット
  - 欲しい情報を取得するためにAPIを複数呼び出すパターン
    - 一覧→詳細の順にAPIをコールする例
  - 追加開発する際にクライアントとサーバーのコミュニケーションコストがかかる
    - 既存のAPIだとレスポンスの中身を最適化できないから、新しくエンドポイントを作る場合
- 既存のサービスをGraphQLに一気に置き換えるのではなく、併用するパターンで移行作業している
  - GraphQLサーバーでリクエストを受けてRESTに流すBFFの構成

- GraphQLのクライアントで有名なもの
  - [Relay](https://relay.dev/)
  - [Apollo](https://www.apollographql.com/)
