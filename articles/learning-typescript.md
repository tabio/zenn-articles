---
title: "手を動かして学ぶTypeScriptの備忘録"
emoji: "😎"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "Nodejs"]
published: false
---

## はじめに

手を動かして学ぶTypeScriptの備忘録

https://www.amazon.co.jp/dp/4863543557

## 3章

```ts:package.json
{
  "name": "app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc",
    "dev": "tsc -w"
  },
}
```

- tsconfig.jsonはtscによるコンパイルを実行のために必要
- `tsc -w` wオプションはファイルを監視して変更があれば自動でビルド実行してくれる
- `--save-dev`はアプリケーションの実行時には使用しないパッケージのインストールに使う
  - 例えば型定義ファイルなど
  - `-D`が省略形
- 無名async関数の書き方
  ```ts
  (async () => {
    const result = await Hoge();
  })();
  ```

### TypeScriptではクラスも型情報として扱える

JavaScriptにおけるクラスのインスタンスはただのオブジェクト
オブジェクトの型はインターフェースを使って表せる
インターフェースは型
つまりクラスの型情報＝インターフェース

```ts
class HitAndBlow {
  answerSource: string[]
  answer: string[]
  tryCount: number

  constructor() {
    this.answerSource = ['0', '1', '2', '3', '4', '5', '6', '7','8','9']
    this.answer = []
    this.tryCount = 0
  }
}

interface HitAndBlowInterface {
  answerSource: string[]
  answer: string[]
  tryCount: number
}

const game: HitAndBlowInterface = new HitAndBlow(); // これはコンパイルエラーにならない
```

### constructorなくても初期値を定義できる

上のクラスをリファクタリング

```ts
class HitAndBlow {
  answerSource: string[] = ['0', '1', '2', '3', '4', '5', '6', '7','8','9']
  answer: string[] = []
  tryCount: number = 0
}

更に型推論が効くので以下のように省略できる
answerプロパティは文字の配列であることを期待したいので型を残している

class HitAndBlow {
  answerSource = ['0', '1', '2', '3', '4', '5', '6', '7','8','9']
  answer: string[] = []
  tryCount= 0
}
```
