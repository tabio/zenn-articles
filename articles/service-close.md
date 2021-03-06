---
title: "サービス終了決定をふりかえって"
emoji: "👻"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["サービス終了"]
published: true
---

# きっかけ

新規サービス開発に携わっている仕事柄、残念ながらサービス終了に立ち会うことが度々あります。  
終了するに至った経緯は保守義務のため話せませんが、作業自体は汎用性があるので備忘録としてまとめておくことにしました。

# チームの目線合わせ

チームメンバーにサービス終了の経験者がいない場合、まずは必要なタスクを洗い出し共有する

### ユーザーへの告知方法の確認

- 利用規約に明記されている場合があるので法務部へ確認
- 掲載箇所の確認
  - 利用ユーザーが目に付きやすい場所候補の洗い出し
    - アプリ起動後
    - 運営側のお知らせ画面
    - PUSH通知
  - 取得している個人情報(メールアドレスなど)の利用可否
- 新規ユーザーへの対応
  - 新規登録できないよう動線の削除

### ステークホルダーへの確認

- チームメンバー
  - 終了に伴いチーム解散や退職が加速する可能性が高まる
  - 業務が個人に強く依存しているものがないかを確認し、コアメンバーとはて退職後もコンタクトを取れるようにしておく
- 社内
  - CEOやCTOなどの役員メンバー
  - 広報に会社としての告知の有無
- 社外
  - toBサービスの場合は契約内容の確認

### 個人情報の取り扱いやデータの保存期間の確認

- 基本的には法務部へ確認
- ユーザーからの情報開示要求に対応する必要がある場合は作業フローを洗い出しドキュメント化しておく
  - サービス終了後はチーム解体し、場合によっては当事者が存在せずという可能性は十分ある

### 事業譲渡の有無

譲渡先と秘密保持の覚書を交わし、現状のシステム設計図や仕様書、DB情報、ソースなどを提出し、質問に適宜打ち返していく作業が発生

### エンジニア向け作業の洗い出し

- ドメイン維持期間の確認
  - ドメインを開放直後は、SEOパワーが残っているので検索エンジンに引っかかる。そのため悪意あるユーザーに悪用される可能性がある。
- ユーザー告知用の開発
  - アプリの場合は強制アップデート機能を入れておかないとこれに対応できない可能性が高いのでローンチ時には確認しておきたい
- リソースの削除
  - 本番環境以外は不要であればコストの面から事前に削除しておく
    - 経験的に開発環境は残しておくと便利
- サードパーティの契約終了調整

### クローズに向けたスケジュール調整

上記を踏まえてプロダクトオーナーと詳細を詰めていく


# ふりかえりミーティング

クローズに向けてスケジュールがある程度決まったところでふりかえりを実施する。  
サービスクローズという性質上、チームメンバー全員が腹落ちしている状態ではないことが多いです。  
そのため、感情的になり議論が紛糾・発散しないように注意して進める必要があります。  

### ふりかえりのゴール

- 「終了」＝「失敗」ではないことを共有
  - 失敗から得られた何かを次に活かせば失敗ではなくなる
- 結果として不変のものは事実ベースで共有しておく
  - サービス指標
    - アクティブユーザー数
    - コスト
    - コンバージョンなど
- 得られたことはメンバー間で違っていても良い
  - ふりかえりで出てきたアイディアは、統一されている必要はなく、個々に感じたことを次回別プロジェクトで語れることが大事
    - 説得力あるレベルまで消化できている必要はある

### 紛糾しないために

- 事実に着目する
- 主語を「チーム」とする
  - 私は〇〇と言ったや誰々が▲▲と言ったはNG
- 意見が食い違った場合は反論者の大切にしている価値観を深堀りする


### ふりかえりテーマ

:::message
サービスのふりかえり
:::

:::message
開発プロセスのふりかえり
:::

ぐらいの粒度で出てきたアイディアを深堀りしていくと良いと思います。


# さいごに

サービス終了という経験は誰しももつものではないので型化し難い部分があると思っています。  
また、扱っているサービスの種類によって不足しているものがあると思っているため、適宜記事をメンテナンスできればなと思っています。(そんなにサービス終了に立ち会いたくはないですが。。。)
