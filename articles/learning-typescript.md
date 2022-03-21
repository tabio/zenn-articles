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

### never型の使いみち

never型のどんな型も入れられないという特性を持つ
何が嬉しいかというとコンパイル前にエラーを検知できる
型を使ったコードの安全性を向上させれる

```ts
// neverにはどんな値も入れられない
let neverVal: never;
neverVal = 'aaaa'

//  Type 'string' is not assignable to type 'never'.
```

Modeはnormalとhardしか理論的に来ないからswitch文のdefault値はなくても動く
将来的にModeに`ultra`が追加された場合、これだとdefaultに入ってくることになる
`問題は、コンパイル後に初めてErrorが発生してcaseの分岐が足りなかったと気づくこと`
コンパイル前にcase分岐追加を気づくことはできないか？

```ts
class Hoge {
  Mode: 'normal' | 'hard';

  fuga(mode: Mode) {
    switch(mode) {
      case "normal":
        return 3;
      case "hard":
        return 4;
      default:
        throw new Error(`${this.mode}は無効なモードです`); // ここのthis.modeの型はstring literalの`ultra`になっているのがポイント
    }
  }
}
```

never型の変数に値を代入させるロジックにしておくことで気づける
コンパイルが通らずエラー箇所を見るとcase文に追加するの忘れてたねということになる
つまり、never型を上手く利用することによって開発時の安全性が増しますよ

```ts
class Hoge {
  Mode: 'normal' | 'hard';

  fuga(mode: Mode) {
    switch(mode) {
      case "normal":
        return 3;
      case "hard":
        return 4;
      default:
        const neverValue: never = this.mode; // string literalのultraをnever型に代入しようとする。never型はnever型以外のどんな値も入れることができない
        throw new Error(`${neverValue}は無効なモードです`);
    }
  }
}

### TypeとInterface

- チーム内でルールを作ってどちらを使うかを決めておく
- Typeは必ず先頭で宣言しておく
- 合併の記述が異なる
  ```ts
  // Type
  type Hoge = { name: string }
  type Fuga = { age: number }
  type Human = Hoge & Fuga

  // Interface
  interface Hoge = { name: string }
  interface Fuga = { age: number }
  interface Human extends  Hoge, Fuga {}
  ```
- Interfaceは同じ名前の定義があったら自動でマージする(declaration merging)ので意図しない型にならないように注意
  ```ts
  interface Person {
    name: string
    age: number
  }

  interface Person {
    height: number
  }

  const person: Person = {
    name: 'hoge',
    age: 20
  }
  // heightが無いというエラーが表示される
  ```

### 型アサーション(as)

- 型アサーションはTypeScriptの型推論を開発者側で上書きできる
- TypeScriptの型チェックの恩恵から外れることになるので多用はさけるべき
- 型キャストと別物であることに注意
  - 型キャストは値そのものを変えてしまうのでランタイムに影響する
  - 型アサーションはあくまでコンパイル時の型解決に影響を与えるものでランタイムには影響しない
    - 故にランタイム後の入力値によってエラーが発生する隙きを生んでしまう可能がある

### ジェネリクス

引数にstringかnumber型を使うことができてその値を返す関数を書く場合

```ts
const returnVal = (value: string | number) => {
  return value;
}
const personName = returnVal('hoge');
const age = returnVal(20);
```

という風に書けるが、引数の型のバリエーションが増えいくと結構辛い
そこでジェネリクスという動的に型を付与する書き方がある

```ts
const returnValue = <T>(value: T) => {
  return value;
}
const personName2 = returnValue<string>('hoge');
const age2 = returnValue<number>(20);
const personName3 = returnValue('hoge'); // stringの部分は型推論が効くので省略可
```

Tの部分は何でも良いHogeHogeでも問題ない
慣習的に`T`,`S`,`U`が使われることが多い

### Stringの配列から型を抽出

```ts
const modes = ['normal' , 'hard']
```
から
```ts
type Mode = 'normal' | 'hard'
```
を作ることができる
方法としては
- `typeof`で型を抽出する
- `as const`でstring literalのまま抽出
- タプル型の要素を抽出できるようにnumberを使う
で結果としては以下のようになる
```ts
const modes = ['normal' , 'hard'] as const
type Mode = typeof modes[number]
```

```ts
typeof modes // typeofだけだとstring[]として抽出されてしまう
```

そこで`as const`を付けることでstring literalとして抽出できる
```ts
const modes = ['normal' , 'hard'] as const
typeof modes // タプル型として抽出できる ['normal' , 'hard']
```

最終的にはユニオン型を表現したい
配列やタプルの型を取り出す方法はインデックス番号を指定してあげれば良い
しかし、今回は全要素が対象なので、そんなときのための便利なnumberキーワードを使う

```ts
const modes = ['normal' , 'hard'] as const
typeof modes[number] // modes[0]だと'normal'というstring literal型が抽出される
```

### constructorの省略形

```ts
class Person {
  readonly name: string;
  constructor(name: string) {
    this.name = name;
  }
}
const person = new Person('hoge');
```

と　

```ts
class Person {
  constructor(private name: string) {} // 引数にはprivate, public, readonlyなどの修飾子がないと自動でプロパティとしてセットしてくれないのがポイント
}
const person = new Person('hoge');
```

は同じ

### インデックスシグネチャ

```ts
type GameStore = {
  'hit and blow': HitAndBlow,
  'janken': Janken
}
```
新しい要素追加の際にキーと値を追加するのではなく、値の追加だけで対応したいという要望に
キーの汎化であるインデックスシグネチャが使える
```ts
type GameStore = {
  [key: string]: HitAndBlow | Janken
}
```
インデックスシグネチャを使うことでスッキリした表現になる
一方で、キーが何でも入ってしまうので型による安全性は下がる
そこでMapped Typesによるキーの制限を入れると良い

### Mapped Typesによるキーの制限

JavaScriptで配列をmapするとコールバックの第一引数でそれぞれの値を取れるのと同じような仕組み

```ts
type Member = 'John' | 'Anna'
type Band = {
  [key in Member]: { part: string }
}
```

一般化するとこんな感じ

> { [K in T]: U }

Tはユニオンが入ってくる(文字列 or 数字の)
KはTの中の1つの型
Uはオブジェクトにした際の型

