---
title: "実践TypeScriptの備忘録"
emoji: "🍇"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: false
---

## 概要

[実践TypeScript](https://www.amazon.co.jp/-/jp/dp/483996937X/)を読んでの備忘録

## 1章 開発環境と設定

TypeScriptのセットアップ操作を通じ的な設定とその意味について解説している
tsconfig.jsonの項目は多く初見では理解が大変なので遭遇したタイミングで立ち戻ってくるのが良い

- tscコマンド
  tsファイルからjsファイルをビルドするコマンド
  コンパイルの設定はtsconfig.jsonにて定義
- tsconfig.json
  - target: どのバージョンのjsとして出力するか
    - 簡単に言うと古いバージョンにするとオブジェクトや関数が使えないということ
  - module: どのモジュールパターンで出力するか
    - commonjs: サーバー側(Node.js)利用想定
    - amd: ブラウザ側利用
    - umd: commonjsとamdの両方に対応
  - strict: true
    - noImplicityanyやnoimplicitthisなどの基本的な型チェックを **一括** で有効化
  - outdir
    - コンパイルしたjsファイルの出力先ディレクトリ
    - 明記しない場合はtsファイルと同じ階層にjsファイルがビルドされる
  - declaration: true
    - 型宣言ファイル(xxx.d.ts)を出力
    - 通常の開発で有効にするケースは少ない
    - 利用シーンとしてはライブラリを自作やProject Referenceを利用する場合
      - [Project Referenceについて参考](https://zenn.dev/katsumanarisawa/articles/58103deb4f12b4)
  - exclude: ビルドから除外するファイル・ディレクトリを指定
  - include: excludeの反対。excludeより弱い
  - files: ビルドに含むファイル。excludeより強い
  - extends
    - tsconfig.jsonを継承できる
    - チームで共通する設定がある時に使うイメージ
- Build Mode
  - tscコマンドのbオプション
  - どのtsconfig.jsonを使うかを明示できる
    ```sh
    # src/tsconfig.jsonを利用する
    tsc -b src
    ```