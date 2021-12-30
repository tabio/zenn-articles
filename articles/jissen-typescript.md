---
title: "実践TypeScriptの備忘録"
emoji: "🍇"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript"]
published: true
---

## 概要

[実践TypeScript](https://www.amazon.co.jp/-/jp/dp/483996937X/)を読んでの写経

## 6章 TypeScriptの高度な型

TypeScriptでは、型で型を定義することが可能(型プログラミング)

### 変数に対するGenerics

型における変数のようなもの
型を可変にすることで柔軟な型定義を書ける

Genericsを利用する型を宣言するときの慣習として、**T**、**U**、**K**の名称を利用する
**<T>** はT型エイリアスと呼ぶ

```typescript
interface Box<T = string> { // 関数のデフォルト引数と同じように初期型を定義できる
  value: T
}
const box0: Box = {value: 'test'}
const box1: Box<string> = {value: 'test'}
const box2: Box<number> = {value: 1}
const box3: Box<number> = {value: 'test'} // Error
```

一方で、型を絞りたい場合もある
その場合は**extends**による制約を追加すればよい

```typescript
interface Box<T extends string | number> {
  value: T
}
const box1: Box<string> = {value: 'test'}
const box2: Box<number> = {value: 1}
const box3: Box<number> = {value: false} // Error
```

### 関数の引数に対してGenericsを利用する場合

```typescript
function boxed<T>(props: T) {
  return {value: props}
}
boxed(1)
```

引数をnullableにしたい場合は宣言時にasを付与

```typescript
const box = boxed(false as boolean | null)
const box2 = boxed<string | null>(null)
```

変数の時と同じようにextendsで制約を追加できる

```typescript
function boxed<T extends string>(props: T) {
  return {value: props}
}
const box = boxed(0) // Error
const box2 = boxed('test')
```

引数の型が明示されていることで関数内部の安全性も上がる

```typescript
interface Props {
  amount: number
}
function boxed<T extends Props>(props: T) {
  return {value: props.amount.toFixed} // amountがnumberなのでtoFixedが呼べる
}
const box = boxed({amount: 19})
```

### 複数のGenericsが出てくる場合

```typescript
function pick<T, K extends keyof T>(props: T, key: K) {
  return props[key] // propsはオブジェクトを想定していて、必ず存在するプロパティ名が保証される
}
const obj = {
  name: 'Taro',
  amout: 0
}
pick(obj, 'name')
pick(obj, 'name1') // プロパティがないのでError
```

### ClassのGenerics

クラスにGenericsを利用することでコンストラクタの引数に制約を付与できる

```typescript
class Person<T extends string>{
  name: T
  constructor(name: T) {
    this.name = name
  }
}
const person = new Person('test')

// クラスメンバーにIndexed　Access Typesを利用した例
class Person<T extends PersonProps>{
  name: T['name']
  age: T['age']
  constructor(props: T) {
    this.name = props.name
    this.age = props.age
  }
}
const person = new Person({name: 'test', age: 1})
```

### 型を抽出したい場合について

対象の型から特定の型を抽出したい場合について理解するためには、型の定義にもIF文(Conditional Types)が適用できることを知っておく必要がある
部分型として型抽出するinferというシグネチャはCondtional Types内で利用するため

以下は**if T は X と互換性のある型であれば ? Y : Z** と読み替えるとわかりやすい

```ts
T extents X ? Y : Z
```

条件に合致した型を抽出する例
Pick型という組み込みUtility Typesを使うのが一番手っ取り早い

```ts
interface User {
  name: string
  kana: string
  gender: 'male' | 'female' | 'other'
}

type UserGender = Pick<User, 'gender'>
// UserGender = { gender: 'male' | 'female' | 'other' }
```

キーの名称だと抽出するのは簡単だけどもう少し抽象化して、stringの型のものを抽出したいとなったら
その場合は以下のように独自で定義したFilter型のようなものを作ってやる

```ts
type Filter<T, U> = {
  [K in keyof T]: T[K] extends U ? K : never // T[K]と互換性のある型Uと一致する場合はK型を返すのがポイント
}[keyof T] // 内部でK型を返しているので末尾は[keyof T]となる

type stringKeys = Filter<User, string> // 'name' | 'kana' というUnion Typeが抽出できる
```

これをPick型と組み合わせて使うとstringに一致するものを抽出できる

```ts
type strings = Pick<User, stringKeys>
```

これまでは直下の階層の型の抽出だったが、ネストした階層の型を抽出したい場合の例

```ts
interface DeepNest {
  deep: { nest: { value: string } }
}

interface Properties {
  deep: DeepNest
}

type Salvage<T extends DeepNest> = T['deep']['nest']['value'] // extends DeepNestがあるのでIndexed Access表記が使える
type DeepDive<T> = {
  [K in keyof T]: T[K] extends DeepNest ? Salvage<T[K]> : never
}[keyof T]
type X = DeepDive<Properties> // X = string
```

inferも型を抽出できる機能を持つ
Conditional Types（IF文）の中で利用する
なにはともあれ例を見ながら読み解く
greet関数に一致したらそのgreet関数が返す型を返す

```ts
function greet() {
  return 'String型が返るよ'
}

type Return<T> = T extends (...args: any[]) => infer U ? U : never
type R = Return<typeof greet> // type R = string
```

`(...args: any[]) => infer U` の部分がConditional Typesに該当する
(..args: any[])は引数を取る関数で、アロー演算子でU型という戻り型を返していると解釈
つまり、Tが関数型と互換があれば、その関数の戻り型を返すというIF文
結果、type Rにはstringが入ってくる

Conditional Typesの中であればinferはどこでも利用できる
以下のサンプルでは戻り型ではなく、引数として利用している

```ts
function greet(name: string, age: number) {
  return 'Hello ${name} ${kana}'
}

type A1<T> = T extends (..args: [infer U, ...any[]]) => any ? U : never // 第一引数の型を抽出
type X = A1<typeof greet> // type X = string

type A2<T> = T extends (..args: [any, infer U, ...any[]]) => any ? U : never // 第一引数の型を抽出
type Y = A2<typeof greet> // type Y = number
```

### Utility Typesについて

車輪の発明をしないようによくあるパターンは組み込み型として定義されている
（いちいち上述したConditional Typesで独自の定義をしなくて済むようにしている）

#### 従来の組み込みUtility Types

```ts
interface User = {
  name: string
  age?: number | undefined
}
```

- Readonly型：Readonlyを強制
  ```ts
  type Hoge = Readonly<User>
  {
    readonly name: string
    readonly age?: number | undefined
  }
  ```
- Partial型：Optionalを強制
  ```ts
  type Hoge = Partial<User>
  {
    name?: string | undefined
    age?: number | undefined
  }
  ```
- Required型：Optionalを排除
  ```ts
  type Hoge = Partial<User>
  {
    name: string
    age: number
  }
  ```
- Record型：新しいObjectを作る。ただし第一引数がキーになって、第2引数が型
  ```ts
  type Hoge = Record<'user', User>
  {
    user: User
  }
  ```
- Pick型：型の抽出。第一引数が対象の型、第2引数がそのプロパティ
  ```ts
  type Hoge = Pick<User, 'name'>
  {
    name: string
  }
  ```
- Omit型：型の除外。第一引数が対象の型、第2引数がそのプロパティ
  ```ts
  type Hoge = Pick<User, 'name'>
  {
    age?: number | undefined
  }
  ```

#### 新しい組み込みUtility Types

- Exclude型：T型の中からUと互換性がある型を`除き`、新しい型を生成
  ```ts
  type X = Exclude<"a" | "b"> // "a"が抽出（"b"はstringで、"a"と互換性があるので"a"を除外して新しい型として生成）
  ```
- Extract型：T型の中からUと互換性がある型を`残し`、新しい型を生成
  ```ts
  type X = Extract<"a" | "b"> // "b"が抽出（"b"はstringで、"a"と互換性があるので"a"を残して、"b"を抽出して新しい型として生成）
  ```
- NonNullable型：T型の中からnullとundefinedを除外した新しい型を生成
  ```ts
  type X = Nonnullable<string | null | undefined> // string
  ```
- ReturnType型：T型は関数であること。関数ではない場合はコンパイルエラー。そしてその関数の戻り値の型を返す
  ```ts
  type X = ReturnType<() => string> // string
  type Y = ReturnType<string> // Error
  ```
- InstanceType型：コンストラクター関数型のインスタンス型を取得
  ```ts
  class C {
    a = 0
    b = 0
  }
  type X = InstanceType<typeof C>
  const n = {} as X // { a: number, b: number }
  ```

## 5章 TypeScriptの型システム

この章は型の互換性について説明されている
型には互換性がある
例えば、値側で推測された型と変数側に定義されている型に互換性がないので代入ができないといったケースがありえる

### string型とString Literal Types

詳細な型(String Literal Types)に抽象な型(String型)を代入できない
number型とNumber Literal Typesの関係も同じ

```typescript
// Errorがでない場合
let s1: 'test' = 'test'
let s2: string = s1

// Errorが出る場合
let s1: string = 'test'
let s2: 'test' = s1
```

### any型は何にでもなれるので不適切な型を代入できてしまう危険なもの

```typescript
let a1: any = false
let a2: string = a1 // booleanなのstringに代入しようとしているがエラーにならない
```
### unknownは as で型を決めるまで別の方を代入できない

```typescript
let u1: unknown = 'test'
let u2: string = unknown // Error
let u3: number = u1 as number // u1の型が決まるので代入できるようになる
```
### asにも互換性がある

値と互換性のないアサーションは定義できない

```typescript
let s1 = 0 as string // Error
```

### **{}型(オブジェクトリテラル)** は特殊
object型として定義した場合は理解しやすいが、{}型にすると理解が難しくなる


```typescript
let o1: {} = 0 // OK
let o2: {} = '0' // OK
let o3: {} = false // OK
let o4: {} = {} // OK

let o1: object = 0 // Error
let o2: object = '0' // Error
let o3: object = false // Error
let o4: object = {} // OK
```

{}型の中身の理解をするのにkeyofでオブジェクトのプロパティ一覧を取得してみれば良い
K2, K3, K4はプロパティを持っていることがわかる。代入できるということは、プリミティブ型は{}のサブタイプと言える。

```typescript
type K0 = keyof {} // never
type K1 = keyof { K: 'K'} // 'K'
type K2 = keyof 0 // "toString" | "toFixed" ...
type K3 = keyof '1' // "charAt" | "toString" ...
type K4 = keyof false // "valueOf"
```

{}型の代入は特定のプロパティが異なるとエラー、また、互いに一致するプロパティがないとエラー

```typescript
let p1 = {p1: 'test'}
let p2 = {p1: 1}
p1 = p2 // Error

let p3 = {p3: 'test'}
let p4 = {p4: 'test'}
p3 = p4 // Error
```

部分的にプロパティが一致している場合は代入する方向によってはOK

```typescript
let p3 = {p3: 'test'}
let p4 = {p3: 'test', p4: 'test'}
p3 = p4
```

### 関数にも互換性がある

引数に互換性がない場合はエラー

```typescript
let fn1 = (a: string) => {}
let fn2 = (a: number) => {}
fn1 = fn2 // Error
```

引数に部分的に一致している場合は引数が多い方へ代入が可能

```typescript
let fn1 = (a: string) => {}
let fn2 = (b: string, c: number) => {}
fn2 = fn1
```

### クラスにも互換性

クラスメンバーが比較対象になる
コンストラクターの引数型は関係ない

```typescript
class Animal {
  feat: number
  constructor(name: string) {}
}

class Human {
  feat: number
  hands: number
  constructor(name: string, gender: number) {}
}

let animal: Animal = new Animal(`dog`)
let human: Human = new Human(`taro`, 2)
animal = human
```

### 宣言空間

TypeScriptにはValue、型、名前空間の3つの宣言空間が存在している
それぞれの空間内で宣言名は重複できない

#### Value宣言空間

値に対して割り当てられる

```typescript
const greet = 'test'
function greet() = {} // 同じ認識子であるgreetを定義できないとError
```

#### Type宣言空間

Typeを宣言する方法は2つ（interface or type alias）で違いはOpen endedへの準拠（後付できるか）
この違いを抑えておく
type aliasの場合は宣言の重複エラーが発生する
interfaceよりもtype aliasが推奨される理由はOpen endedによって予期せぬ不具合を防ぐためか

```typescript
// interfaceの場合
interface User {
  name: string
}
interface User {
  age: number
}
↓定義が上書きされて結合
interface User {
  name: string
  age: number
}

// type aliasの場合
type User = {
  name: string
}
type User = {
  age: number
}
```

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
