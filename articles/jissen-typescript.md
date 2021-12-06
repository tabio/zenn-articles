---
title: "実践TypeScriptの備忘録"
emoji: "🍇"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript"]
published: true
---

## 概要

[実践TypeScript](https://www.amazon.co.jp/-/jp/dp/483996937X/)を読んでの備忘録

## 4章 TypeScriptの型安全

どの言語でもそうだけど如何にバグを生まないようにするかは重要
TypeScriptは型がるためその恩恵を受けやすい(TypeScriptを選ぶ理由の1つになる)
型推論だけでなく、意図的に型を**絞り込む**ことで更にバグを生みにくくすることが期待できる

### 制約による型安全

TypeScriptでは早期リターンをすることで、型が絞込まれた推論が適用されている
このような型の絞り込みの処理を**Type Guard**や**ガード節**と呼ぶ
ガード節の良くあるパターンは後述
```typescript
function getFormattedValue(value: null | number) {
  if (value === null) return value // value: nullと推論される
  return `${value.toFixed(1)} pt` // value: numberと推論される
}
```

引数の型にundefinedは含まれていないが、「?(オプショナル型)」を付与することで
TypeScriptが自動で引数のundefinedを考慮しれくれる
```typescript
function greet(name?: string) { // TypeScriptは function greet(name: string | undefined) と解釈している
  return `Hello ${name.toLowerCase()}`
}
```
これの何が嬉しいかというと実行エラーが事前に気づける(nameがundefinedの場合、toLowerCaseが呼べないのでコケる)
実際にこのコードをVSCodeで記述すると、nameがundefinedである可能性を指摘してくれる
つまり、型のみ論理的にチェックをすることで事前にバグを検知できる
Guard節により型の絞り込みをおこなうことで対処できる
```typescript
function greet(name?: string) {
  if (name === undefined) return 'Hello'
  return `Hello ${name.toLowerCase()}`
}
```

デフォルト引数を利用した場合は少し興味深い挙動をする
```typescript
function getFormattedValue(value: number, unit = 'pt') {
  return `${value.toFixed(1)} ${unit.toLowerCase()}`
}
```

VScodeで関数の推論された型をみると

> function getFormattedValue(value: string, unit?: string): string

とunitはundefinedの可能性を推論している
undefinedと推論した場合、関数内部のtoLowerCaseが実行できない可能性を考慮してエラーを検知してくれるかと思いきやエラーにならない(greet関数の例)
理由は単純でデフォルト値があるから(はundefinedにならないことを保証している)
もちろん、

> getFormattedValue(100, 0)

だとunitの型が違うためエラーとして検知される


オブジェクトの型安全についてはWeak Typeというものがある
定義としてはすべてのプロパティがオプショナルな型なもの
一つでも一致していれば意図したものだと判断してくれる
```typescript
type User = {
  name?: string
  age?: number
}
function register(user: User) {}

const maybeUser = {
  age: 1,
  gender: 'male'
}
register(maybeUser) // 1つでも一致していればエラーでない

const notUser = {
  gender: 'male',
  graduate: 'Tokyo'
}
register(notUser) // Error
```

一方で、オブジェクトリテラルを直接引数に入れるとエラーになる
**Excess Property Checks(過剰なプロパティチェック)** と呼ばれるもの
オブジェクトリテラルを直接利用することは実際の開発現場で良くあり(設定値を渡すシーンなど)、存在しないプロパティに対して過剰に検査をするようになっている
```typescript
register({
  age: 1,
  name: 'Hoge',
  gender: 'male'
})
```

readonlyの使い方は2つ
1つはreadonlyシグネチャを利用する
```typescript
type State = {
  readonly id: number
  name: string
}
const state: State = {
  id: 1,
  name: `hoge`
}
state.id = 2 // Error
```
もう1つはReadonly型を適用する
```typescript
type State = {
  id: number
  name: string
}
const state: Readonly<State> = {
  id: 1,
  name: `hoge`
}
state.id = 2 // Error
```
Readonly型はすべてのプロパティに一括でreadonlyシグネチャを付けているのと同じ
しかし、悲しいかなJavaScriptの振る舞いとしては実際には値が書き換わってしまう
あくまでTypeScript上で安全を得ることができるものである

JavaScript側で書き換わるのを制御したいという場合はObject.freezeを使えば良い
```typescript
type State = {
  id: number
  name: string
}
const state: State = {
  id: 1,
  name: `hoge`
}
const frozenState = Object.freeze(state) // VScode上ではReadonly<State>として推論されている
frozenState.id = 2
```

### 抽象度による型安全

ダウンキャストは抽象的な型から詳細な型を付与すること
TypeScriptよりもプログラマーの方が型に詳しい時に用いる
よってプログラマーは定義した型に責任を持つ必要がある

```typescript
const defaultTheme = {
  backgroundColor: "orange" as "orange", // literal typesのorangeとして定義
  borderColor: "red" // stringと推論
}

defaultTheme.backgroundColor = "blue" // Error
defaultTheme.borderColor = "blue" // No Error
```

ちなみにダウンキャストは互換性のある型にしか適用できない
"orange"のliteralをbooleanにキャストすることはできない

```typescript
const defaultTheme = {
  backgroundColor: "orange" as false
}
```

アップキャストは抽象度を上げる(詳細から抽象へ)
抽象度を上げれば安全に聞こえるかもしれないが注意は必要
例えば以下のanyにアップキャストしたことで実行時にしかエラーに気づけ無い

```typescript
const fiction: number = toNumber('1,000') // anyにアップキャストしたので実際にnumberを期待している変数に入れることができてしまう
fiction.toFixed()  // Runtime Error発生する
```

インデックスシグネチャ **[k: string]** はオブジェクトに動的なプロパティを定義したいときに使う

```typescript
type User = {
  name: string
  [k: string]: any
}
const userA: User = {
  name: 'Taro',
  age: 1
}
const x = userA.age // anyとして推論
```

ただし、トップレベルのプロパティに互換性がない場合はコンパイルエラーを引き起こすの注意
nameがstringでnumberと互換性ないと怒っている

```typescript
type User = {
  name: string
  [k: string]: number
}

↓Union Typesにすることで回避できる
type User = {
  name: string
  [k: string]: number | string
}
```

インデックスシグネチャを利用時に**値**に対して制限加えたい場合
そんな場合は以下のような定義をすると良い

```typescript
type Nickname = 'hoge' | 'foo' | 'fuga'
type User = {
  [k: string]: Nickname
}
const userA: User = {
  name: 'hoge'
}
const x = userA.name // Nickname型と推論される
const y = userA.aaa // Nickname型として推論されて、aaaプロパティがないのにエラーにならない
```

↑の変数yはコードとしては危険なのでそれを防ぐためundefinedを追加しておくこと

```typescript
type User = {
  [k: string]: Nickname | undefined
}
const y = userA.aaa // Nickname | undefinedとして推論されるので後続処理でガード入れることを教えることができる
```

インデックスシグネチャを利用時に**キー**に対して制限加えたい場合
**inキーワード** を利用する

```typescript
type Properties = 'age' | 'hight'
type User = {
  [K in Properties]: number
}
const userA: User = {
  age: 1,
  hight: 170
}

const x = userA.age
```

constでも別の変数に入れたり、関数の戻りとして返した時は抽象化されてしまう
そこで、const assertionというものがある
readonlyを全体に付与してくれる
別の変数にコピーされても型定義はそのまま

```typescript
const tuple1 = [1, '2', false] as const // readonly [1, '2', false]
let tuple2 = tuple1

const a = 'a'
let b = a  // 通常は変数にコピーされると型が抽象化されてしまう(Widening Literary Types)
```

関数の場合は以下のようになる

```typescript
function increment() {
  const res = { type: 'INCREMENT' }
  return res
}
const x = increment() // { type: string }と抽象化されてしまう

function increment2() {
  const res = { type: 'INCREMENT' } as const
  return res
}
const y = increment2() // { readonly type: 'INCREMENT' }
```

anyを乱発しないことの例
この例からも分かるように型の緩い不要な戻り型はバグの温床になってしまう

```typescript
function greet(): any {
  console.log('Hello')
}
const message = greet()
console.log(message.toUpperCase()) // anyによって型エラーで検知できない
```

Non-null assersionはプログラマー都合で欺かれた定義であり、その場しのぎでしかない
よって使うべきではない

```typescript
function greet(name?: string) {
  console.log(`Hello ${name!.toUpperCase()}`) // nameがundefinedだとエラーだが型から検知できなくなる
}
```

ほぼお目にかかる機会がないがアサーションを重複して付けることができる
よほどのことがない限り利用はしないこと

```typescript
const myName = 0 as any as string
console.log(myName.toUpperCase())  // 明らかにエラーでるけど型からエラーを検知できない
```
### 良くあるガードパターン

typeof演算子

```typescript
function reset(value: number | string) {
  const v0 = value // const v0: number | string
  if (typeof value === 'number') {
    // ここの段階でvalueはnumberのみと推論される
    const v1 = value // v1: number
    return 0
  }
  const v2 = value // v2: string
  return ''
}
```

プロパティをin演算子で比較すると型が絞り込まれる

```typescript
function judgeUser(user: UserA | UserB) {
  if ('gender' in user) {
    const u0 = user // UserA | UserB
  }
  if ('name' in user) {
    const u1 = user // UserA
  }
  if ('age' in user) {
    const u2 = user // UserB
  }
}
```

instanceof演算子はtypeofと同じで違いはクラスに対しての話

```typescript
class Creature {
  breathe() {}
}
class Animal extends Creature {
  shakeTail() {}
}
class Human extends Creature {
  greet() {}
}

function action(creature: Creature | Animal | Human) {
  if (creature instanceof Creature) {
    const c1 = creature // Creature | Animal | Human
  }
  if (creature instanceof Animal) {
    const c2 = creature // Animal
  }
  if (creature instanceof Human) {
    const c3 = creature // Human
  }
}
```

タグ付きUnion Types
switch文によって型の絞り込みができる
しかし、条件があり、比較されるオブジェクトは共通するプロパティを持っていること
また、その型はリテラルタイプであること

```typescript
function judgeUserType(user: UserA | UserB) {
  switch(user.gender) {
    case 'male':
      const u0 = user // UserA
      return u0
    case 'female':
      const u1 = user // UserB
      return u1
    default:
      const u2 = user // never defaultブロックに到達するこてゃないため
      return u2
  }
}
```

### TypeScriptがプログラマーを信用して型を推論しているパターン

getUserTypeでは引数anyに対してユーザーが定義した型を推論されている
その中核にあるのは関数の戻り値に **引数 is Type** と記述しbooleanを返している関数
これを **ユーザー定義 guard types** という
ここのポイントはTypeScriptがプログラマーを信用して型を推論していること

```typescript
type User = { gender: string; [k: string]: any }
type UserA = User & { name: 'hoge' }
type UserB = User & { age: 1 }

function isUserA(user: UserA | UserB): user is UserA {
  return user.name !== undefined
}

function isUserB(user: UserA | UserB): user is UserB {
  return user.age !== undefined
}

function getUserType(user: any) {
  const u0 = user // any
  if (isUserA(user)) {
    const u1 = user // UserA
    return 'A'
  }
  if (isUserB(user)) {
    const u2 = user // UserB
    return 'B'
  }
  return 'unkown'
}

const x = getUserType({name: 'aaa'}) // 'A' | 'B' | 'unkown'
```

### よく利用するfilterは型を絞り込むことができないけどユーザー定義ガード節を利用することで可能になる

```typescript
type User = { name: string }
type UserA = User & { gender: 'male' | 'female' | 'other'}
type UserB = User & { graduate: string }

const users: (UserA | UserB)[] = [
  {name: 'Taro', gender: 'male'},
  {name: 'Hanako', graduate: 'Tokyo'},
]

const filteredUsers = users.filter(user => 'generate' in user) // (UserA | UserB)[] と推論されてる
```

ここにユーザー定義ガード節を利用した関数を併用すると

```typescript
function filterUser(user: UserA | UserB): user is UserB {
  return 'graduate' in user
}

const filteredUsers = users.filter(filterUser) // UserB[] と推論される

// 匿名関数を使ってこうも書ける
const filteredUsers = users.filter(
  (user: UserA | UserB): user is UserB => 'graduate' in user
)
```

## 3章 TypeScriptの型推論

- TypeScriptは変数に型を必ずしも付与する必要はない
  - 代入される値から良しなに型を推論してくれるため
  - letとconstの変数は推論の挙動が異なるので注意
    - constはLiteral Typeになる(ゆえにに再代入ができない)
    ```typescript
    const hoge = 'Taro'; // hogeの型は'Taro'という文字列リテラルとして推測される
    ```
    - しかしconstで定義した変数をletの変数に代入するとLiteral Typesがなくなる
    ```typescript
    const hoge = 0 // 0型
    const hoge2 = 'Taro' as 'Taro' // 明示的にアノテーションを付けると再代入でもLiteral Typesを維持してくれる
    let fuga = hoge // number型になる
    let fuga2 = hoge2 // Taro型
    ```
  - Array型とTuple型の推論はアノテーションを付けなければ良しなに推論してくれる
    ```typescript
    // Array
    const arr = [0, "1"] // (number | string)[]]
    const arr2 = [0 as 0, "1" as "1"] // (0 | "1")[]

    // Tuple
    const tuple1 = [false, 1, "2"] as [boolean, number, string]
    tuple1.push(false) // OK
    tuple1.push(true) // NG: 2番目はnumber型なので
    ```
  - Object型のプロパティは再代入できる。プロパティをLiteral Typesとして認識させるためにはアサーションを使う
    ```typescript
    const obj = {
      foo: boolean,
      bar: false as false
    }
    obj["foo"] = true // ok
    obj["bar"] = true // error
    ```
  - 関数の戻り値も推論が効く。returnする型がわかりきっている場合は定義してあげた方がバグを生まない安全設計になる。
    一方で、if文で分岐が複数あり複数の型を返す可能性がある場合は良しなに推論してUnion Typesで型推論してくれる
    ```typescript
    function hoge(score: number) {
      if (score > 10) return null
      if (score < 5) return "hoge"
      return score
    }
    // 推論結果： function hoge(score: number): number | "hoge" | null
    ```
  - Promiseの型推論はresolveに型を付けることで安全設計になる
    - resolveの型の付け方は2つある(どちらもresolveにstring以外が入るとコンパイルエラー)
      ```typescript
      // 関数の戻り値型にアノテーション付けるパターン
      function hoge: Promise<string> {
        return new Promise(resolve => {
          resolve("hoge")
        })
      }
      // Promiseのインスタンス作成時に型を付けるパターン
      function hoge {
        return new Promise<string>(resolve => {
          resolve("hoge")
        })
      }
      ```
    - async関数はreturnされる型を良しなに推論
      ```typescript
      // async自体がPromiseを返すのでPromise<string>
      async function hoge() {
        const message = await fuga() // messageにはstringが入ってくる想定
        return message
      }
      ```
    - importも定義元の型を推論してくれる（ただしrequireはしない）
      ```typescript
      // test.ts
      export const value = 10
      -----
      import { value } from "./test"
      const v1 = value // v1: 10
      ```
    - jsonは型を推論してくれるので便利
      ただし、jsonファイルを外部モジュールとしてインポートするには**tsconfig.json**の**resovleJsonModule**と**esModuleInterop**をtureにする

## 2章 TypeScriptの基礎

- 関数の **引数** には型を付けましょう
  - jsは文字列を暗黙的に数値に変換してしまう。意図しない不具合を避けるために型を定義する
- 関数の **戻り値** は型推論が効くのでbetter
- null型 / undefined型という名前の型がそれぞれ存在する(単体では役に立たない)
- object型はブレースを使って定義するとエラーを吐かないので注意
  ```typescript
  let objectBrace: {}
  let objectType: object
  objectBrace = true // エラーにならない
  objectType = true
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
- クラス
  - メンバー修飾子はprivateやpublic以外でいうとprotectedがある
- enumは数値も文字列も両方定義できる
  - open endedに準拠しているので同じenumのTypeがあったら自動でマージしてくれる(ただし文字列列挙型に限る)
    ```typescript
    type Hoge {
      Fuga = "fuga"
    }
    type Hoge {
      Foo = "foo"
    }
    // HogeはFugaとFooを持つ
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
