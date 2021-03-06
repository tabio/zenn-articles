---
title: "手を動かして学ぶTypeScriptの備忘録"
emoji: "🍍"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "Nodejs"]
published: true
---

## はじめに

手を動かして学ぶTypeScriptの備忘録

https://www.amazon.co.jp/dp/4863543557

## 1章

### サブタイプ

以下のリテラル型はどれも通常のデータ型のサブタイプとなっている
サブタイプとは継承のようなもの

```ts
const num: 123 = 123
const name: 'Jack' = 'Jack'
console.log(num) // numは123型でnumber型のサブタイプ
console.log(name) // nameは'Jack'型でstring型のサブタイプ
name.toLowerCase() // string型のサブタイプなのでtoLowerCaseを呼び出せる
```

### リテラル型とString型の型推論

letで定義したname1はstringとして推測させる
一方で、sayHelloの引数はリテラル型として推測されるので、コンパイルエラーになる

```ts
let name1 = 'Hoge'

const sayHello = (personName: 'Fuga') {
  console.log(personName)
}

sayHello(name1) // 型 'string' の引数を型 '"Fuga"' のパラメーターに割り当てることはできません。
```


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


## 4章

### グローバルの名前空間で変数宣言すると型情報は参照できてしまう罠

そのため別の変数と名前が競合する場合衝突してしまう

getHoby.ts上ではexportしてないただの定数だが

```ts:getHoby.ts
const getHoby = () => 'game';
```

同じ階層のindex.tsから参照できる

```ts:index.ts
const hoby = getHoby();
```

しかし、型の参照ができているだけで実際の関数を参照している訳でない
そのためビルド後に実行するとロジックの中身がないとエラーが発生する

> ReferenceError: getHoby is not defined

ではどうしたら良いかでいうとexport, importを使う

```ts:getHoby.ts
export const getHoby = () => 'game';
```

```ts:index.ts
import { getHoby } from './getHoby';
const hoby = getHoby();
```

exportが何をしているかというと`ローカルスコープ内`に閉じ込める役割を担っている
（先程はexportを付けないことでグローバル空間に変数宣言していたた）

index.ts側でgetHobyを参照したい場合はimportを使う
これによりコンパイル後にちゃんとgetHobyをファイルから読み込んで参照できる形にしてくれる

このexportとimportの仕組みは`ファイルモジュール`と呼ばれている

### export defaultの罠

基本使わない
1. importで呼び出す時に名前を勝手に変えることができる
  そのため別ファイルで名前が衝突することがある
2. エディタとの相性良くない
  vscodeで変数名を記述することでimportを補完してくれる機能が働かない

### モジュールシステムの歴史

- サーバーサイド(Node.js)のJSモジュール
  - CommonJS
    - サーバーサイドJSの仕様のこと
    - CommonJSの仕様に沿ったJSのサンプルコードは
      ```js:sum.js
      module.export = function(a,b) {
        return a + b
      }
      ```
      ```js:index.js
      const sum = require('./sum.js');
      console.log(sum(1,2));
      ```
- ブラウザ環境でのJSモジュール
  - AMD(Asynchronous module definition)
    - ブラウザ上でもモジュールを非同期でロードする仕様も広まっていった
      ```js
      define(function() {
        return function sum(a, b) {
          return a + b
        }
      })
      ```
      ```js
      define(['sum'], function(sum) {
        console.log(sum(1, 2))
      })
      ```
- ブラウザ環境での実行できるようにビルドするためのツール
  - Browserify
    - CommonJS形式のJSの依存関係を解決して1つのファイルにまとめて出力してくれる
  - RequireJS
    - Browserifyと違い事前にビルドして依存関係を解決する訳でない
    - ランタイム上で非同期に依存関係を解決する

Node.jsではCommonJS形式で、ブラウザではBrowserifyやRequireJSでと開発者が大変な時代があった

その後2015にECMAScript Modules(ESModules)に統一する動きがでた
ESModulesのサンプルはTypeScriptの書き方のそれ
```js
export function sum(a,b) {
  return a + b
}
```
```js
import { sum } from './sum.js';
console.log(sum(1,2));
```

しかし、正式なモジュール仕様が定められたけど、ブラウザの種類によっては対応してないものもあるのが現状
具体的にはIEはESModules未対応(Chrome, Firefox, Edgeなどは対応済み)

ブラウザによってはまちまちな状況に対してWebpackというモジュールバンドラーが出てきた
CommonJSだろうとESModulesだろうと、すべての形式をブラウザで動く形に変換してくれるという優れもの

Node.jsもESModules対応されているけど、CommonJS形式のモジュールはまだ多く残っているのが現状

ESModulesも良いところばかりではない、importが深くネストしていると、ラウンドドリップタイムが長くなってしまう

### 型定義ファイル(DefinitelyTyped)が必要かどうかの確認方法

npmのサイトでパッケージを検索し`DT`のマークがあれば、`@types/ライブラリ名`

node_modules/@types/ライブラリ名

でインストールされる
型情報は`index.d.ts`に記載されている

![](/images/learning-typescript/definitely-typed.png =300x)

アプリケーションコード上でimportすると、そのライブラリのindex.d.tsを参照してくれるという

一方で、@types/ライブラリ名は別の人が作成しているので、`最新の型情報に追従している保証がない`ことに注意


### DefinitelyTypedがない場合は自分で作るしかない

ts/@types/uuid/index.d.ts

```ts
declare module 'uuid' {
  namespace uuid {
    function v4(): string
  }
  export = uuid
}
```

細かい型定義は省略。declareでライブラリの型定義ができることだけ覚えておく

### enumを使うべきでない理由

- TypeScriptのコンセプトに合ってない
  TypeScriptはコンパイルするとJavaScriptになる
  コンパイル後は型定義は全て削除される
  つまり、TypeScriptの構文はJavaScriptのランタイムに影響与えない
  しかし、enumはランタイムに残る構文でありランタイムに影響を与えている
  TypeScriptのコンセプトにenumはそぐわないのが理由の１つ
- 数値列挙型のEnumは方安全でない
  ```ts
  enum Color {
    Red,
    Yellow
  }
  const num = 10
  const color: Color = num // エラーが発生しない
  ```

### enumの代替

オブジェクト型を利用することでenumの機能を代替できる
keyofによる型の抽出と、型アサーションのas const、typeofを使用する

### tsconfig

- noUnusedParameters
関数の途中の引数を使わない場合、引数名の先頭に `_` を付けることを強制するオプション
_付けないとコンパイルエラーになる
```ts
dragula([this.doingList, this.todoList, this.doneList]).on("drop", (el, target, _source, sibling) => {
  let newStatus: Status = statusMap.todo;
  if (target.id === "doingList") newStatus = statusMap.doing;
  if (target.id === "doneList") newStatus = statusMap.done;

  onDrop(el, sibling, newStatus)
})
```

### Assertion Functions

文字列の場合のみ大文字にしたい関数があったとする
一方で引数が何くるか分からないのでanyにしている場合はこんな感じでロジック書ける
```ts
function toUpper(value: any) {
  if (typeof value !== 'string') {
    throw new Error('avlue is not a string')
  }
  return value.toUpperCase()
}
```

Assertion Functionsを使って型判定ロジックを外だしすることができる

```ts
function toUpper(value: any) {
  assertIsString(value)  // ポイント：以降のコードではvalueはstring以外は入ってこないと扱ってくれる
  return value.toUpperCase()
}

function assertIsString(value: any): asserts value is string { // ポイント：asserts T is U という構文
  if (typeof value !== 'string') {
    throw new Error('avlue is not a string')
  }
}
```

より複雑なanyの判定ロジックにて大きな力を発揮する
例えばJSONはいろんな型の値を内包しているので便利
なにより、`as`を使うこと無く`型安全`にコードを記述できる強みがある
一方で、例外を必ず投げることになるのでエラーハンドリングが必要になる

### Conditional Types

代入されるT型が { message: unknown } に代入可能であれば、T["message"] を返す。そうでなければ never を返す。

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never

interface Email {
  message: string
}

interface Dog {
  bark(): void
}

type EmailMessage = MessageOf<Email> // string
type DogMessage = MessageOf<Dog> // never
```

### unkown型 と any型 の使い所

unknownはanyと同じ様に何でも入れることができる。一方でその使い所がピンとこない。
ポイントは、`使うかどうか` で区別する。

- anyは何でも入れられて、参照も可能
- unknownはなんでも入れられるが、参照できない(unknown型を参照しようとするコンパイルエラーになる)

Conditional Typesでunkown型を使っているが、これはmesssageを参照する訳ではなく、ただその型にハマるかどうか(messageをキーに持つオブジェクトであるかどうか)だけの用途だからunkownを使っている。

実際の利用シーンとしては、型を保証できないケースで利用されることが多い（外部APIのレスポンスや何かをパースした結果など）

