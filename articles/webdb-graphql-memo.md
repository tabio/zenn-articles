---
title: "WEB+DB PRESS Vol.125のGraphQL備忘録"
emoji: "📗"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["GraphQL"]
published: true
---

## 概要

[WEB+DB PRESS Vol.125](https://www.amazon.co.jp/gp/product/4297124351)のGraphQL章の備忘録


## GraphQLの概要

- Facebook社が作ったWeb APIの仕様
  - RESTでのツラミを解決したかった
    - 1度に複数のリソースを取得する
    - オーバーフェッチングやアンダーフェッチングしない(必要以上なデータをフェッチしない。その反対も)
- GitHubやTwitter、Netflix、Shopify、はてななど多くのテック企業での利用実績あり
- GraphQL自体はFB社から[GraphQL Funcation](https://graphql.org/foundation/)に移管された(2018年)
- RelayはFB社が作成したReactやReact Native向けのクライアントライブラリ
  - クライアントが使いやすいようにGraphQLのスキーマに一部制約を導入しているのが特徴
    - interface Node導入するなどのアレ
- GraphQL over HTTP
  - 本来HTTP以外のプロトコルでも利用できるが普及している大半がHTTP
  - 標準化に向けて準備中
  - レスポンスにdataとerrorを含めるルール
- リクエストの単位は「オペレーション」と呼ぶ

### オペレーション

- 種類は3つ
  - Query
  - Mutation
  - Subscription

- スキーマの書き方
  - オペレーション毎に以下のように書くのが一般的
    QueryやMuatation、Subscriptionはデフォルト型名なので省略可能とのこと
    ```graphql
    # QeuryやMuationの型をルートオペレーション型という
    schema {
      query: Query
      mutation: Mutation
    }
  
    # Query型を定義して対応する中身を書く
    type Query {
      posts: [Post!]!
      post(id: ID!): Post!
    }
    ```
- クライアント側からのQueryオペレーションの例(オペレーション構文)
  基本の構文は
  > 種別 名前(引数) セレクションセット
  ```graphql
  # 詳細
  query getPost($id: ID!) {
    post(id: $id) {
      title
      body
    }
  }

  # 一覧
  query {
    posts {
      title
    }
  }
  ```

### ディレクティブ

@include、@skip、@deprecatedがある動的に振る舞いを変えたいときに利用する
例えば、フラグによって記事のタイトルのみ or タイトル＋本文を切り替えたいとき
```graphql
query listPosts($isBody: boolean!) {
  posts {
    title
    body @include(if: $isBody)
  }
}
```
