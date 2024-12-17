---
title: "MDN Baseline Newly Available 2024 を振り返ってみよう"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## Baseline

みなさん Baseline をご存知でしょうか。

すべての主要なブラウザで対応されている機能なので
開発での違いに頭を抱えずに

こんな機能が開発で使えるんだ～という気持ちで

順番は主観的に注目度が高かった順です。

## 宣言型 Shadow DOM

Web Components

動的 Declarative Shadow DOM

    Document: parseHTMLUnsafe() static method
    Element: setHTMLUnsafe() method
    ShadowRoot: setHTMLUnsafe() method

[Shadow DOM](https://developer.mozilla.org/ja/docs/Web/API/Web_components/Using_shadow_DOM) は再利用可能なコンポーネント
これまでは JavaScript

https://azukiazusa.dev/blog/declarative-shadow-dom/

## Popover API
## コンテンツの公開設定
- content-visibility
- checkVisibility()
https://web.dev/articles/content-visibility?hl=ja

## offset-position と offsetpath の値

## ブロック レイアウト上の Align-content


## text-wrap white-space-collapse: テキストの折り返しをより便利に
- `text-wrap`
- `white-space-collapse`
- `white-space`
https://developer.mozilla.org/ja/docs/Web/CSS/text-wrap
https://coliss.com/articles/build-websites/operation/css/about-text-wrap-balance.html

## font-size-adjust: 異なるフォントが混ざっても綺麗なタイポグラフィ 

これは英語圏限定の機能なのですが同じフォントサイズでも異なるフォントだと文字の大きさがズレてしまったり、ピッタリ合わず、辛い思いをします。小文字の高さ `x-height` で揃えることで綺麗に見えるというものが `font-size-adjust` です。

`x-height = font-size * font-size-adjust` となるように揃えてくれます。

@[codepen](https://codepen.io/anko9801/pen/MYgJQoo)

## Relative colors: カラースキーマを相対的に表現

カラーパレットの管理が大変なので色を決めたら後は OKLCH と Relative colors で表現してしまおう。

```css
.lighten-by-25 {
  background: oklch(from blue calc(l * 1.25) c h / 0.8);
}
.success {
  --c: green;
}
aside {
  background: oklch(from var(--c) calc(l * 0.75) c h / 0.5);
  color: oklch(from var(--c) calc(l * 1.5) c h);
}
```

## light-dark(): ダークテーマが組み込みに！

```css
@media (prefers-color-scheme: dark) {
}
```

```css
:root {
  color-scheme: light dark;
}
body {
  color: light-dark(#333b3c, #efefec);
  background-color: light-dark(#efedea, #223a2c);
}

.times {
  font-family: Times, serif;
  font-size: 24px;
}
.verdana {
  font-family: Verdana, sans-serif;
  font-size: 24px;
}
.adjust {
  font-size-adjust: 0.545;
}
```
https://developer.mozilla.org/ja/docs/Web/CSS/color_value/light-dark

## Gradient interpolation
## backdrop-filter: 背景にぼかしや色変化を与える

## :state()
https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet
https://developer.mozilla.org/en-US/docs/Web/CSS/:state

## zoom: レイアウトが変わる transform-scale
重なるようにその要素が大きくなる
`transform: scale(3);`
レイアウトの再計算をする
`zoom: 3;`

## transform-box: 
## transition-behavior: 
https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior

## ステップ関数 `round()` `mod()` `rem()`

値の丸め込みとして `round()` は `up` `down` `nearest` `to-zero`
さらに割った余りを返す CSS 値関数として `mod()` `rem()` が追加されました。
```
mod()
```
違いはマイナスになったときの挙動で、それぞれ割る数と割られる数の符号に依存して返す符号が変わります。
```js
mod(?, ±) = ±
rem(±, ?) = ±
```
例えば
ユースケースがあまり思い付かない

## groupBy() 関数
`Object.groupBy()` `Map.groupBy()`
```javascript
Object.groupBy(
  [
    { type: "bird", name: "Peacock" },
    { type: "fish", name: "Tuna" },
    { type: "animal", name: "Dog" },
    { type: "animal", name: "Horse" },
    { type: "fish", name: "Sardine" },
    { type: "animal", name: "Lion" },
  ],
  ({ type }) => type
);
```

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy

## `Array.fromAsync()`
Promise.all() 遅延評価版

## Promise.withResolvers
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers

## ArrayBuffer や Set のメソッドが充実しました
配列や集合の操作に便利なメソッドが生やされました
Set は値の集合を表すデータ構造です。

- `ArrayBuffer.prototype.resize()`
- `ArrayBuffer.prototype.transfer()`
- `ArrayBuffer.prototype.transferToFixedLength()`
- ArrayBuffer.prototype.detached
- ArrayBuffer.prototype.maxByteLength
- ArrayBuffer.prototype.resizable
- `Set.prototype.intersection()`
- `Set.prototype.union()`
- `Set.prototype.difference()`
- `Set.prototype.symmetricDifference()`
- `Set.prototype.isSubsetOf()`
- `Set.prototype.isSupersetOf()`
- `Set.prototype.isDisjointFrom()`

```js
const a = new Set([1, 2, 3]);
const b = new Set([1, 3, 5]);
```

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/transfer
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/resize

## 単語分割 intl.Segmenter
デフォルトで単語ごとに区切る
https://polypane.app/blog/using-the-intl-segmenter-api/

## WebGL API
Color management for WebGL
Color management for WebGL2

## JS - HTMLVideoElement: requestVideoFrameCallback()

## Canvas willReadFrequently
## cookieEnabled
```
if (!navigator.cookieEnabled) {
  // ブラウザーが対応していないか、クッキーが設定されることをブロックしています。
}
```
## Mutually exclusive <details> elements
 Multiple <details> elements which use the same name attribute are mutually exclusive. When one member of the group is opened, all other members are closed.
https://developer.mozilla.org/ja/docs/Web/API/HTMLDetailsElement/open

## Alt text for generated content
## Registered custom properties
https://developer.mozilla.org/en-US/docs/Web/API/CSS/registerProperty_static

## API: Async clipboard
## Vertical form controls
## AbortSignal.any()
## Extended constant expressions (WebAssembly)
## AVIF
