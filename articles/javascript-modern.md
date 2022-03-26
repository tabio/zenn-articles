---
title: "JavaScriptモダンプログラミング完全ガイド"
emoji: "🍔"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript"]
published: true
---

## はじめに

JavaScriptモダンプログラミング完全ガイドの備忘録

https://www.amazon.co.jp/dp/4295010561

## 1章

### オブジェクトリテラルでプロパティ名を算出するときは角括弧を使う

```js
let hoge = { [field.toLowerCase()]: 1 }
```

### 配列に対するtypeof演算子は'object'を返すので、配列の判定はArray.isArrayを使う

```js
let hoge = [1,2]
typeof hoge // 'object'
Array.isArray(hoge) // true
```

### JSONはすべての文字列をダブルクォートで囲む（シングルクォートではない）

```js
{ "name": "hoge", "age": 10 }
```

### rest宣言

配列を分割するときに使う

```js
numbers = [1,7,3,5]
let [first, second, ...etc] = numbers
first // 1
second // 7
etc // [3,5]
```

### 変数にデフォルト値入れられる

```js
let [first, second = 0] = [10]
first // 10
second // 0
```

## 3章

クロージャは自由変数を持つ関数のこと
自由変数とはコード内で使われる変数のうち、パラメータとしてもローカル変数としても宣言されていない変数のこと
以下のtextが自由変数である
```js
const sayLayer = (text, when) => {
  let task = () => console.log(text)
  setTimeout(task, when)
}

sayLayer("GoodBye", 1000)
sayLayer("Hello", 1000)
```

よくある関数の中で関数を呼ぶ例におけるクロージャの説明
要はクロージャを使うことで、関数が再実行されるまで値を保持することができる
その値は、プライベートな変数として扱うことができ、外側にある外部要因によって影響することはない

https://note.affi-sapo-sv.com/js-closure.php


## 4章

プロトタイプオブジェクトの概念はこちらを参照すると分かりやすい

https://jsprimer.net/basic/prototype-object/#object-is-origin

いわゆるメソッドやプロパティの集まりであって、メソッドもオブジェクトなのでメソッド自体もこのObject.prototypeに定義されているメソッドを継承していることになる


サブクラスのコンストラクタは必ずスーパークラスのコンストラクタを呼び出す必要がある
this参照はsuperを呼び出した後でなければ使えない

```js
class Manager extends Employee {
  constructor(name, bonus) {
    super(name) // まずスーパークラスのコンストラクタ呼び出し必須
    this.bonus = bonus // その後thisが有効になる
  }
}
```

## 7章

### Map

- キーと値のペアの連想配列として利用できる
- JavaScriptのオブジェクトはMapの一種
- あえてオブジェクトではなくMapクラスを使う利点としては以下
  - Mapはキーの型を問わない
    - オブジェクトはシンボルか文字列なので
  - Mapは要素が挿入された順番を覚えている

```js
const weekdays = new Map([["mon", 1], ["tue",2]]) // Map(2) {'mon' => 1, 'tue' => 2}

or

const weekdays = new Map()
// setで追加
weekdays.set("mon",1) // Map(1) {'mon' => 1}

// deleteで削除
weekdays.delete("wed")

// キーの存在確認
weekdays.has("mon") // true

// 値を取り出すにはget
weekdays.get("mon")

// forEachも使える
weekdays.forEach((val, key) => {
    console.log(key, val)
})
// mon 1
// tue 2
```

### Set

pythonのSetと同じで重複のない要素を保存できる

```js
const hoge = new Set()
hoge.add(1) // Set(1) {1}
hoge.add(2) // Set(2) {1, 2}
hoge.add(1) // Set(2) {1, 2}

hoge.delete(1) // 要素の削除
hoge.has(1) // 要素の存在確認
hoge.clear() // すべての要素の削除
```