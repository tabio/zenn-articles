---
title: "はじめてのGraphQLの備忘録"
emoji: "📕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["GraphQL"]
published: true
---

## はじめに

はじめてのGraphQLを読んで、章ごとに学んだことをメモする
途中で挫折することなく読み切るための備忘録

https://www.amazon.co.jp/dp/487311893X

## 4章：スキーマの設計

GraphQLスキーマは、どんなオペレーションがあってどんなフィールドが取得できるかを定義したもの
スキーマファイルは慣習で **.graphql** とされている

ID型は固有の識別子を格納するもの。JSONとしては文字列で返されるが、String型とは異なり値が重複しないことをバリデーションしてくれる
**!** はnullにならないの意(null成約)
```graphql
type Photo {
  id: ID!
  name: String!
  description: String
  numbers: [Int!]! # リストにもnull成約を付けることができる。べき論として配列の値がnullのみ場合は空配列にする。
}
```

IDやStringなどはスカラー型と呼ばれて、他に「Int、Float、Boolean」がある。
カスタムもできる
```graphql
scalar DateTime # カスタムスカラー型として定義

type Photo {
  id: ID!
  created: Datetime
}
```
よくあるカスタムスカラー型が実装されている[パッケージ](https://github.com/stylesuxx/graphql-custom-types)

Enumもある。よく見るEnumの定義の仕方と同じ
以下の場合、categoryにHOGEなどの定義外の値を入れるクライアントに返すことはできない
```graphql
enum PhotoCategory {
  SELFIE
  ACTION
  GRAPHIC
}

type Photo {
  id: ID!
  name: String!
  category: PhotoCategory!
}
```

### 多:多

Userは友人達の情報を持っていて、友人達はそれぞれどこで知り合ってどのぐらいの期間友人関係かを持っている
これをGraphQLで表現するとこんな感じになる
```graphql
type User {
  friends: [Friends!]!
}

type Friends: {
  friends: [User!]!
  howLong: Int!
  whereWeMet: Location
}
```

### ユニオン型 / インターフェース

3章でユニオン型やインターフェースでのQueryオペレーションについて学んだので、ここではそれをどうやって定義しているかについて記載されている

```graphql:ユニオン型
union AgendaItem = StudyGroup | Workout

type StudyGroup {
  name: String!
  subject: String!
}

type Workout {
  name: String!
  reps: Int!
}

type Query {
  agenda: [AgendaItem!]!
}
```

```graphql:インターフェース
interface AgendaItem {
  name: String!
  start: String!
  end: String!
}

# interfaceで定義したものは必ずフィールドとして含めないといけない
type StudyGroup impliments AgendaItem {
  name: String!
  start: String!
  end: String!
  topic: String!
}
```

### 引数

引数を使うことで絞り込みができる。書き方はシンプル
```graphql
# スキーマ
type Query {
  Photo(id: ID!): Photo! # IDが必須なのでない場合はGraphQLサーバー側でエラー
  allPhotos(category: PhotoCategory): [Photo!]! # categoryにnullが入ってきた場合はすべてのPhotoを返す
}

# Queryオペレーション
query {
  Photo(id: "hoge") {
    name
    description
  }
}
```

### データページングとソート

実際に運用していくと一度に取得できる件数の絞り込みが必要になってくる
ページングに必要な引数(mysqlでいうlimitやoffset)を定義してあげる
ソートも同じ要領で定義する
```graphql
enum SortDirection {
  DESC
  ASC
}

enum SortField {
  name
  createdAt
}

type Query {
  allUsers(first: Int=50 start: Int=0 sort: SortDirection = DESC sortBy: SortField = name): [User!]!
}
```

### 入力型

queryでもmutationでも運用していくと引数が多くなり非常に見づらくなる
そうした場合は入力型を使って入力値をまとめてあげると良い

```graphql
input PostPhotoInput {
  name: String!
  description: String
  category: PhotoCategory=PORTRAIT
}

type Mutation {
  postPhoto(input: PostPhotoInput!): Photo!
}

# Mutationの実行
mutation newPhoto($input: PostPhotoInput!) {
  postPhoto(input: $input) {
    id
    name
    createdAt
  }
}
```

### 返却型

レスポンスの型もカスタムで作れるという話

```graphql
type AuthPayload {
  user: User!
  token: String!
}

type Mutation {
  githubAuth(code: String!): AuthPayload!
}
```

### スキーマのドキュメント化

3章のイントロスペクションでGraphQLクライアントツールから定義が閲覧できたと思うが、その際に定義書をより分かりやすくできるようにコメントを追加できる。
コメント部分が定義書にも反映されるので便利

```graphql
type User {
  """
  ユーザーのユニークなGitHubログインID
  """
  githubLogin: ID!
}

type Mutation  {
  githubAuth(
    "code(引数)に対してもコメント付けれる"
    code: String!
  ): AuthPayload!
}
```


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


### GraphQL実行ツール

|ツール名|memo|
|---|---|
|[GraphiQL](https://github.com/graphql/graphiql)|Facebook社製|
|[GraphQL Playground](https://github.com/graphql/graphql-playground)|Prismaチーム製。Web版は[こちら](https://www.graphqlbin.com/v2/new)|
|[Altair GraphQL Client](https://github.com/altair-graphql/altair)|最近使っているやつ。すべてのプラットフォームに対応している|

上記のツールはAPIスキーマの定義書を表示してくれる
これは裏側でGraphQLのイントロスペクションという機能を使っている
具体的にはAPIのスキーマを返すQueryオペレーションを実行している

```graphql: イントロスペクションの例
query {
  __schema {
    types {
      name
      description
    }
  }
}
```

### 公開 GraphQL API

GraphQLのスキーマの参考になる[一覧](https://github.com/APIs-guru/graphql-apis)


### Queryオペレーション

```graphql: 選択セット、クエリ引数、スカラー型、オブジェクト型
query lifts {
  allLifts(status: ClOSED) { # status: CLOSED部分がクエリ引数(クエリ結果をフィルタリングしたいとき)
    name
    status
  }
  trailAccess { # 選択セット(波括弧部分のこと) start
    name
    difficulty
  }             # 選択セット end
  Lift(id: "jazz-cat") {
    capacity      # スカラー型：Int, Float, String, Boolean, ID型からなる
    trailAccess { # オブジェクト型： 1つ以上のフィールドの集合
      name
      difficulty
    }
  }
}
```

フラグメントはフィールドが冗長化している時に有効
1つの修正で一気に変更が適応できる

```graphql: フラグメント
fragment liftInfo on Lift {
  name
  status
}

query {
  Lift(id: "jazz-cat") {
    ...liftInfo
  }
  Trail(id: "river-run") {
    name
    difficulty
    accessedByLifts {
      ...liftInfo
    }
  }
}
```

こういうリストの中身に異なる型を持つ場合への対応策としてユニオン型
```
[
  {
    "name": "Cardio",
    "repos": 100
  },
  {
    "name": "Comp Sci",
    "subject": "Computer Science",
    "students": 12
  }
]
```

```graphql:ユニオン型
query schedule {
  agenda {
    ...on Workout { # フラグメントを選択セットの中に書き込む方法(インラインフラグメント)
      name
      reps
    }
    ...on StudyGroup {
      name
      subject
      students
    }
  }
}
```

一方で必須のフィールド以外に任意のフィールドが入ってくる場合がある時はインターフェース
```
[
  {
    "name": "hoge",
    "start": 10,
    "end": 30,
  },
  {
    "name": "hoge",
    "start": 10,
    "end": 30,
    "memo": "fuga"
  },
]
```

```graphql
query schedule {
  agenda {
    name
    start
    end
    ...on Workout {
      memo
    }
  }
```

### Mutationオペレーション

変数は$で始まる
動的にパラメーターを埋め込める

```graphql
mutation createSong($title: String! $numberOne: Int) {
  addSong(title:$title, numberOne:$numberOne) {
    id
    title
    numberOne
  }
}
```

### Subscriptionオペレーション

Facebookのいいね数をリアルタイムで反映する仕組みとして利用
表示時にSubscriptionでいいね数を監視
いいね数が更新されるとSubscriptionを実行しているクライアントに対してデータがpushされる
クライアントは通知を受けて表示を更新するという仕組み

Subscriptionで購読する
![](/images/init-graphql-memo/subscription-1.png)

別タブで値を更新する
![](/images/init-graphql-memo/subscription-2.png)

Subscriptionしたタブで新しい値が流れてきているのがわかる
![](/images/init-graphql-memo/subscription-3.png)


### 抽象構造木(AST: Abstract Syntax Tree)

> curl -X POST -d "{ \"query\": \"query { viewer { login }}\"}" https://hoge.com/graphql

クエリ自体は上記の様に文字列で送られてくる
それをどうやって解釈しているかの話

抽象構造木は構文構造をデータ構造に起こしたもの
DOMツリーの様に要素毎に意味を持ち階層化されているというのがイメージしやすい

一覧のフローを図示
```mermaid
graph TB
    A[クライアント側からはクエリは文字列として渡される] -->|引数や波括弧やコロンをパース| B(断片化)
    B -->|字句解析| C(抽象構造木)
    C -->|ASTを走査してスキーマに対してバリデーション| D(型がスキーマと一致していたらオペレーション実行)
```



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
