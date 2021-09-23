---
title: "Lambdaのライフサイクルついて学ぶ"
emoji: "🍭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Lambda"]
published: true
---

# 概要

AWS Lambdaの起動から終了までのライフサイクル、およびその周辺について学ぶ

# Lambdaとは

LambdaはNodeやPythonなどのコードを実行できる環境を提供してくれるサービス
サーバーを用意しなくて良いのでユーザーはビジネスロジックの開発に集中できるのが売り
図のLambda Serviceが何をするかを管理していて、Lambda APIを使用してコードの実行や[ログ出力](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-logs-api.html)などの操作を行っている

[AWS公式の出典より](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-context.html)
![lambfda service](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images/logs-api-concept-diagram.png)

### 実行環境

LambdaからAPIを介して指示が来るのでそれを受けて処理を行う場所
関数の実行に必要なリソース管理を担っている

##### リソース管理

- 関数を実行する[ランタイム環境](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/lambda-runtimes.html)を用意
  - ランタイム環境はAmazon Linuxベースのコンテナ
  - NodeやPython、Javaなどの主要な言語毎に用意されている
  - 各言語の最新のバージョンには追従していない(結構遅れて対応してくるイメージ)
  - LambdaからランタイムAPIを使用して関数実行処理される
- 関数が使用できるメモリや最大実行時間などの設定
- 実行環境自体のライフサイクルをサポート

##### 実行環境のライフサイクル

[AWS公式の出典より](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-context.html)
![lambfda lifecycle](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images/Overview-Successful-Invokes.png)
![lambfda lifecycle](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images/Overview-Full-Sequence.png)

|フェーズ||
|--|--|
|[INIT](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-extensions-api.html#runtimes-extensions-api-reg)| 設定されたリソースをベースに実行環境を作成 or フリーズ解除|
||関数コードやLayerをS3からDLしてランタイムを初期化<br/>関数のハンドラーの外部にあるコードの実行|
||INITも詳細には3つのサブフェーズ(拡張機能・ランタイム・関数のそれぞれの初期化)からなる|
|[INVOKE](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-extensions-api.html#runtimes-lifecycle-invoke)|Lmabda関数のハンドラーの呼び出し。関数が実行され完了したら次の呼び出し準備に備える|
|[SHUTDOWN](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-extensions-api.html#runtimes-lifecycle-shutdown)|Lambda関数が一定時間呼び出しされないとシャットダウン|ランタイムのシャットダウン|
|| 実行環境自体の削除|

X-RayでもINIT、INVOKEの順番で動作することが確認できる

[AWS公式の出典より](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/foundation-progmodel.html)
![x-ray](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images/features-initialization-trace.png)

##### 再利用 

[Black Belt P.87](https://d1.awsstatic.com/webinars/jp/pdf/services/20150701_AWS-BlackBelt-runcodeinthecloud.pdf)から抜粋
INVOKE後にコードの変更がない＆時間がそれほど経過してない場合はLambda側でランタイムを再利用する
これによりランタイムの初期化をスキップできるのでパフォーマンス向上する(ウォームスタート)

:::message
再利用されるかどうかの判断はLambda側でブラックボックス化している
:::

コールドスタートとウォームスタートのイメージに関しては[Black Belt P.65-69](https://d1.awsstatic.com/webinars/jp/pdf/services/20190402_AWSBlackbelt_AWSLambda%20Part1%262.pdf)を参照

##### [プロビジョニング済み同時実行](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/configuration-concurrency.html)

負荷が急激上がったりすると、再利用だけではリクエストを捌けなくなるため実行環境を増やす
すると、コールドスタート処理が多発してレイテンシー増加となる
これを防ぐ方法としてプロビジョニング済みを事前に用意しておくオプションが存在する

プロビジョニング済みの中身としては、Lambda側で意図的に関数実行インスタンスを再利用できるように調整してくれる
具体的にはINITの初期化を数時間毎に定期実行してくれている
デメリットとしては、リクエストが来ない時でも初期化処理を行うためコストに跳ね返ってくる

### VPCアクセス

LambdaからVPC内のリソースにアクセスすることは多い(例えばVPC内のRDSに接続するなど)
その場合はIPアドレスをアタッチしないとRDSと通信できない
これを実現するために、INIT時にENI(Elastic Network Interface)を付与してくれている
以前はENIの紐付けに時間がかかり処理が遅い原因でしたが、今は事前に用意されたENIを利用してVPCへアクセスする形式へと改善されました

[AWS公式の出典より](https://aws.amazon.com/jp/blogs/news/announcing-improved-vpc-networking-for-aws-lambda-functions/)
![VPC Lambda](https://d2908q01vomqb2.cloudfront.net/b3f0c7f6bb763af1be91d9e74eabfeb199dc1f1f/2019/09/04/v2n-architecture-1024x613.png)
