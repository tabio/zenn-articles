---
title: "実践TypeScriptの備忘録"
emoji: "🍇"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: false
---

## 概要

[実践TypeScript](https://www.amazon.co.jp/-/jp/dp/483996937X/)を読んでの備忘録

## 2章 TypeScriptの基礎

- 関数の **引数** には型を付けましょう
  - jsは文字列を暗黙的に数値に変換してしまう。意図しない不具合を避けるために型を定義する
- 関数の **戻り値** は型推論が効くのでbetter
- null型 / undefined型という名前の型がそれぞれ存在する(単体では役に立たない)
- object型はブレースを使って定義するとエラーを吐かないので注意
  ```typescript
  let objectBrace: {};
  let objectType: object;
  objectBrace = true; // エラーにならない
  objectType = true;
  ```
- 複数の型を1つに結合(Intersection Types: &)
  ```typescript
  type Dog = {
    bark: () => void
  }
  type Bird = {
    fly: () => void
  }
  type Kimera = Dog & Bird // barkとflyの2つを持つ型
  ```
- 複数の型のうちどれか1つの型に適合する(Union Types: |)
  ```typescript
  let value: boolean | string
  value = false
  value = "1"
  value = 1 // Error
  ```
- 文字列リテラルを型にできる(String Literal Types)
  これと同じ要領で数字リテラルやBooleanリテラルもある
  ```typescript
  let taro: 'Taro'
  taro = 'Taro'
  taro = 'Taro1' // Error

  let zero: 0
  zero = 0
  zero = 1 // Error
  ```
- typeofで宣言済みの変数の型を取得
  ```typescript
  let myObject = { foo: "foo" }
  let anotherObject: typeof myObject = { foo: "" } // myObjectの型を取得して定義に利用している
  ```
- keyofでオブジェクトのプロパティ名を取得
  ```typescript
  type SomeType = {
    foo: string
    bar: string
  }
  let somekey: keyof SomeType // let someKey: "foo" | "bar"
  ```
- keyofとtypeofの併用でオブジェクトのキーを型として定義できる
  ```typescript
  const myObject = {
    foo: 'FOO'
    bar: 'BAR'
  }
  let myObjectKey: keyof typeof myObject
  myObjectKey = "foo"
  myObjectKey = "far" // Error
  ```
- キャストは2種類の記述方法(<>とas)
  ```typescript
  let someValue: any = "this is a string"
  let length: number = (<string>someValue).length
  let length: number = (someValue as string).length
  ```


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