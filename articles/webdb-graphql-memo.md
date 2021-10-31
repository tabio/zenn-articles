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

## サーバー側

### RESTの以下のツラミをGraphQLでは解決できる

- クライアントに特化させたAPIを作らないといけなかったりする(メンテナンス性低い)
- 1つ画面でAPIを何度もリクエストしないといけなかったりする

### GraphQLフェデレーション

マイクロサービスのAPI Gatewayパターンと相性が良い
要は複数のAPIを束ねる技術
ドメインごとのチームで並列でAPI開発できるので効率化できる
[GraphQL Federation](https://www.apollographql.com/docs/federation/federation-spec/)という仕様書が公開されている


### サーバーとスキーマとリゾルバの関係

多くのサーバーサイド用のライブラリがあるが[GraphQL.js](https://github.com/graphql/graphql-js)を参考にしたものが大半
ライブラリ(GraphQLサーバー)から呼び出される関数のことを「リゾルバ」という
リゾルバはスキーマで定義されている個々のフィールドの値を解決する役割を担う


### スキーマとリゾルバの関連性の管理

コードファースト・スキーマファーストの2つのアプローチがある

|種別名|例|
|---|---|
|コードファースト|GraphQL.jsやNexus|
|スキーマファースト|Apollo Server|

```graphql
# コードファースト: スキーマとリゾルバ一体化
query: new GraphQLObjectType({
  name: 'Query',
  fields: {
    hello: {
      type: GraphQLString,
      resolve: () => 'hello world',
    }
  }
})
```

```graphql
# スキーマファースト：スキーマとリゾルバが分離
const typeDefs = gql`
  type Query {
    hello: String
  }
`;

const resolvers = { # リゾルバがスキーマ型と一致しているか保証されてないのでフォローが必要(GraphQL Code Generatorなど)
  Query: {
    hello: () => 'hello world',
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});
```

### N+1問題対策

データローダーという仕組みを導入する
１つ１つのデータの取得を **遅延評価** して一括で処理できるようにするもの
dataloaderパッケージを追加してリゾルバを跨いで共有できるcontextにdataloaderインスタンを保存しておくのが良い
prismaはDataloader対応が良しなにされている様子([参考](https://qiita.com/joe-re/items/6e09e741ed2e0c6637b0#dataloader))

## 実践パターン

### Relayの追加仕様で参考になるもの

- [Nodeインターフェース](https://zenn.dev/tabio/articles/init-graphql-memo#global-object-identification)
- [ページネーション](https://zenn.dev/tabio/articles/init-graphql-memo#cursor-connections)

### Viewerパターン

TBD

### エラー表現

基本はレスポンスにerorrsフィールドを持ち、中身のフィールドは以下
- message
- locations
- paths

GraphQLはエラーのフィールド拡張を認めている
extensionsフィールドを追加して、httpのステータスコードなどを持たせるのが良い

```graphql
{
  "errors": {
    "message": "登録が失敗しました",
    "locations": [{"line": 2, "column": 3}],
    "path": ["node"],
    "extensions": {
      "code": "NOT_FOUND"
    }
  }
}
```

ミューテーションの入力に対するバリデーション結果などはスキーマ自体にエラーオブジェクトを組み込むのが良い

```graphql
type Mutation {
  createPost(
    blogId: ID!
    input: PostInput!
  ): CreatePostPayload!
}

type CreatePostPayload {
  userErrors: [UserError!]!
  post: Post
}

type UserError {
  message: String!
  field: [String!]
}
```


### APIのバージョニング

RESTのように明示的にバージョニングする必要はない
フィールドを失効させたい場合は@deprecatedディレクティブを使ってクライアントに教えてあげれば良い
