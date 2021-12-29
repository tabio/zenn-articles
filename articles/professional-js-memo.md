---
title: "プロフェッショナルWebプログラミングJavaScript"
emoji: "🍲"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: true
---

## はじめに

書籍「[プロフェッショナルWebプログラミングJavaScript](https://www.amazon.co.jp/dp/4295201049/)」の備忘録

## Chapter 3

### Google Chrome DevTools Tips

- 履歴のクリア
  ![](/images/professional-js-memo/clear-history.png)
- リロードしても履歴を保持
  ![](/images/professional-js-memo/preserve-log.png)
- ログのレベルのfilter
  ![](/images/professional-js-memo/filter-log-level.png)
- console.tableで配列を見やすく
  ```js
  const arr = [111,1111,11111,22,33333,444]
  console.table(arr)
  ```
  ![](/images/professional-js-memo/console-table.png)
- Live ExpressionをONにして評価したい式を入力するとその式がLiveで実行される
  ![](/images/professional-js-memo/live-expression.png)

## Chapter 2

- 絵文字入りの文字列の長さを測るのは難しい
  絵文字の種類によってうまくいったり行かなかったりする
  正確な文字数を知りたい場合はライブラリを見つけて使うのが良い
  ```js
  const hoge = "絵文字🌵❗🍲"
  console.log(hoge.length) // 8
  console.log(hoge.split('').length) // 8
  console.log(Array.from(hoge).length) // 6
  console.log([...hoge].length) // 6
  ```
- reduceは繰り返しの中で処理を加えることができるので便利
  ```js
  const arr = [111,222,333,444]
  const sum = arr.reduce((res, val) => {
    return res + val
  }, 1000000)
  console.log(sum) // 1001110
  ```
- async functionと書くことでその関数は非同期関数となり、暗黙的にPromiseオブジェクトを返す
  - async functionの内部ではawaitを使うことでPromiseの処理を待ちつつプログラムを進めることができる

## Chapter 1

- NaNはNot-A-Numberの略で `計算結果` が数字として表現できないもの
  - 0/0 や "aaa" * "bbb" などで発生する
- 変数展開する時に使っている **``** はテンプレートリテラルと呼ばれている
- nullは `存在しない or 無効を表す`
- undefinedは `変数として宣言したけど値を設定しない` 時の状態
- 関数の引数として定義された関数を`コールバック関数`という
  - ES6で導入されたアロー関数は良くコールバック関数として利用される

### thisについて

JavaScriptはthisの挙動が特殊
use strictでないときthisは`window`を指す
一方で、use strictを有効化していることがほとんどだと思うのでその場合の挙動について把握しておくことが大事
以下use strictが有効化されている前提の話
関数の中で呼ばれるthisは`undefined`になる

```js
function hoge() {
  console.log(this);
}
hoge() // -> undefined
```

一方でオブジェクトの中の関数の中のthisはそのオブジェクト自信を指す

```js
const hoge = {
  fuga: 'fuga',
  test: function() {
    console.log(this);
  }
}
hoge.test()

↓
{
  fuga:"fuga",
  test:f test {...}
}

hoge.test.apply('ここがthisに反映されます !!') // ここがthisに反映されます !!
hoge.test.call('ここがthisに反映されます !!') // ここがthisに反映されます !!

const bindHoge = hoge.test.bind('bindを使ったよ')
bindHoge() // bindを使ったよ
```

ここで少しthisから離れるが、JavaScriptにおいて`関数もオブジェクトの1つ`で関数オブジェクトと呼ばれている
関数オブジェクトにはapply, callというメソッドが用意されていて、thisを外側から変更することができる
オブジェクトであることを考慮すると理解はしやすい
callとapplyの違いは引数の定義が異なるだけで機能としては同じ
また、bindというメソッドがありcallとapplyとの違いはその場で実行されずに、thisを置き換えた関数を返す

### 分割代入

変数に値を入れる時に便利な場合があるので覚えておく
...etcのように残りをすべて代入することができ`残余構文`と呼ばれている

```js
// arrayの場合
const arr = ['aaa', 'bbb', 'ccc']
const [a, ...etc] = arr
console.log(etc)

['bbb', 'ccc']

// objectの場合
const objs = {
  aaa: 'aaaa',
  bbb: 'bbbb',
  ccc: 'cccc'
}
const {aaa, ...etc} = objs
console.log(etc)

{
bbb:"bbbb",
ccc:"cccc"
}
```

## Chapter 0

### JavaScriptとWebページの関係性

- JavaScriptはJavaScriptエンジン上で動くプログラミング言語
- JavaScriptからWebページを操作するためのルールがDOM（Document Object Model）
- JavaScriptエンジンがJSで記載されたテキストを解読しDOMを動かしている
- JavaScriptエンジンは複数種類あるがすべてECMAScriptという仕様に沿っている
- ブラウザ上でHTMLを表示するための機能がレンダリングエンジン
  - JavaScriptエンジンと同様ブラウザ間で利用しているレンダリングエンジンは異なる(表示がブラウザによって若干異なる理由)

これらは以下の図でみると理解しやすい

![](https://typescriptbook.jp/assets/files/browser-rendering-engine-javascript-engine-ecmascript-relations-977f2898337ec4fefb4fa76955d63913.svg)

> 出典：サバイバルTypeScript

- JavaScriptエンジン上のメモリ内では、文字列はUTF-16として処理されている
  - UTF-8はサイズは小さいが1文字毎のサイズがいびつで処理しにくいため