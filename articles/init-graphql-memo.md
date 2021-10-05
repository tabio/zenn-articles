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

## 3章：GraphQLの問い合わせ言語

|項目|memo|
|---|---|
|SQL|DBに対してデータを操作するための言語|
|REST|SQLの考えを受け継いで、クエリではなくURLのエンドポイントに対してデータ操作をする考え方|
|GraphQL|SQLの考えを受け継いだインターネットのための問い合わせ言語(DB、ファイル、REST API、WebSocketなどあらゆるモノに対する問い合わせ)|

|機能|SQL|GraphQL|memo|
|---|---|---|---|
|検索|Select|Query|dataフィールドにリクエスト結果が入っている。エラーはerrorフィールド|
|登録|Create|Mutation||
|更新|Update|Mutation||
|削除|Delete|Mutation||
|監視| - |Subscription|ソケット通信でデータの変更を検知するもの(SQLにはない)|


#### GraphQL実行ツール

|ツール名|memo|
|---|---|
|[GraphiQL](https://github.com/graphql/graphiql)|Facebook社製|
|[GraphQL Playground](https://github.com/graphql/graphql-playground)|Prismaチーム製。Web版は[こちら](https://www.graphqlbin.com/v2/new)|
|[Altair GraphQL Client](https://github.com/altair-graphql/altair)|最近使っているやつ。すべてのプラットフォームに対応している|

#### 公開 GraphQL API

GraphQLのスキーマの参考になる[一覧](https://github.com/APIs-guru/graphql-apis)


## 2章：グラフ理論

GraphQLのコンセプトであるグラフ理論についての章
注釈にもあるようにこの章を読まなくても使うことへの支障はない

- [グラフ理論](https://ja.wikipedia.org/wiki/%E3%82%B0%E3%83%A9%E3%83%95%E7%90%86%E8%AB%96)
  - 人や物、アイディアなどのオブジェクト同士の関係性を形式的に表現することで理解を深めることができる
  - 四角や丸のオブジェクトをそれぞれ線で結びつけるだけだが、登場人物が多くなってくると視認性抜群のグラフになる
    - 組織図や家系図などが分かりやすい（文章だと即時に理解はできない）
  - Facebookのユーザーは複数のユーザーと繋がっていて、そのユーザーも別の友だちと繋がっている(無方向のグラフ)
    - オブジェクトと関連が複雑に組み合う中で、必要な情報を必要に応じて取得したいというニーズはGraphQLのクエリからも読み取れる


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
