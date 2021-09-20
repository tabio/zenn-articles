---
title: "AppSheetで筋トレログアプリを作ってみた"
emoji: "💪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AppSheet", "ノーコード"]
published: true
---

## はじめに

自宅勤務で鈍った体に活を入れるべく週末にジムに通っています。
ジムは自治体が管理しているためかIT化に疎い様で、トレーニングの記録は紙で行っていました。
そこで[AppSheet](https://www.appsheet.com/)というGoogleが提供しているノーコードツールを使ってサクッとログアプリを作ってみました。
作ったアプリはiOS端末から利用できます。
Googleアカウント連携して利用するため、プライベートでの利用を想定しています。
(Googleアカウントに紐づく組織やユーザーが素早く業務アプリ作れるのがAppSheetの特徴)

## 事前準備

- [AppSheet](https://www.appsheet.com/)からサインアップしておく
- [AppSheetアプリ](https://apps.apple.com/jp/app/appsheet/id732548900)を端末にDLしておく
- 今回のアプリ用のスプレッドシートを用意しておく(イメージしやすいようにテストデータを含めています)
  ![](/images/nocode-appsheet/6.png)

## 概要

- 以前1回AppSheetで簡単なアプリを作った経験があるのでなんとなく操作は把握
- 開発時間は **約1時間程度**
- UXは作りながら考えていった


## アプリの作成

![](/images/nocode-appsheet/1.png =300x)

Start with your own dataを選択(事前準備で作ったスプレッドシートを使うため)

![](/images/nocode-appsheet/2.png =300x)

アプリの名前とカテゴリは適当に

![](/images/nocode-appsheet/3.png =300x)

事前準備で作成しておいたスプレッドシートを選択

![](/images/nocode-appsheet/4.png =300x)

AppSheetがスプレッドシートのデータの中身を分析していい感じのUXを作ってくれます

![](/images/nocode-appsheet/5.png =600x)

自動で作ってくれたものを修正していきます

### 一覧のソートとグルーピング

ジムでやったトレーニングを最新日付が上に表示されること
また、日付によってグルーピングしてトレーニングの履歴を把握しやすいようにする

![](/images/nocode-appsheet/7.png =600x)

### 表示用のカラムを作る

「Data」→「Columns」→「add visual column」を選択

![](/images/nocode-appsheet/9.png =600x)

CONCATENATE関数を利用して表示用カラムを作成

![](/images/nocode-appsheet/8.png =600x)

### 一覧の表示デザインを修正

listやphotoなど表示パターンが予め用意されている
画像や文字の部分を選択することで、Dataのどのカラムを表示するかを選択できる

![](/images/nocode-appsheet/10.png =600x)

### アプリのデザインを変える

「UX」→「Brand」→「Primary color」や「App logo」で変更できる

![](/images/nocode-appsheet/11.png =600x)

ヘッダーのON/OFFやスタイル

![](/images/nocode-appsheet/12.png =600x)

フッダーのアイコンも変更

![](/images/nocode-appsheet/15.png =600x)

### 登録画面の確認

デフォルトだと種別や重さを毎回手動で入力しないといけない状態になっている

![](/images/nocode-appsheet/13.png =300x)

カラムのTYPEをEnumにすることでセレクトボックスタイプに自動でなる

![](/images/nocode-appsheet/14.png =600x)

### Deploy

画面上のNot Deployedをクリックするとdeploy用のダイアログが表示されるので画面に沿ってデプロイ作業をする
iOSアプリをDLしてGoogleアカウントとリンクをしておけばメールが来る
メール内のリンクをタップするとAppSheetが起動して開発したトレーニングログアプリが立ち上がり無事利用することができました

## さいごに

簡単な業務アプリであれば数時間で動くものができるという体験は凄いものがありました
非エンジニアがPoCやワイヤーを作るという観点での利用もオススメです
