---
title: "MDN Baseline Newly Available 2024 を振り返ってみよう"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## Baseline

みなさん Baseline をご存知でしょうか。

こんな機能が開発で使えるんだ～という気持ちで

順番は主観的に注目度が高かった順です。

## 宣言型 Shadow DOM

Web Components

[Shadow DOM](https://developer.mozilla.org/ja/docs/Web/API/Web_components/Using_shadow_DOM) は再利用可能なコンポーネント
これまでは JavaScript

https://azukiazusa.dev/blog/declarative-shadow-dom/

## Popover API
## コンテンツの公開設定

https://web.dev/articles/content-visibility?hl=ja

## offset-position と offsetpath の値

## ブロック レイアウト上の Align-content


## text-wrap `white-space-collapse`

**テキストをバランスして折り返します。**

`white-space`
https://developer.mozilla.org/ja/docs/Web/CSS/text-wrap
https://coliss.com/articles/build-websites/operation/css/about-text-wrap-balance.html

## カラースキーマを相対的に表現 CSS: Relative colors

## ダークテーマが組み込みに！ light-dark()

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
```
https://developer.mozilla.org/ja/docs/Web/CSS/color_value/light-dark


## :state()
https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet
https://developer.mozilla.org/en-US/docs/Web/CSS/:state

## レイアウトが変わる zoom
重なるようにその要素が大きくなる
`transform: scale(3);`
レイアウトの再計算をする
`zoom: 3;`

## 背景にぼかしや色変化を与える backdrop-filter


## ステップ関数 `round()` `mod()` `rem()`

**CSS でステップ関数が使えます。**

値の丸め込みとして `round()` は `up` `down` `nearest` `to-zero`
さらに割った余りを返す CSS 値関数として `mod()` `rem()` が追加されました。
```
mod()
```
違いはマイナスになったときの挙動で、それぞれ割る数と割られる数の符号に依存して返す符号が変わります。
```
mod(?, ±) = ±
rem(±, ?) = ±
```
例えば
ユースケースがあまり思い付かない



## CSS: transition-behavior
https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior
### CSS: font-size-adjust

## Array.fromAsync() 静的メソッド
Promise.all() 遅延評価版

## Promise.withResolvers
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers

## ArrayBuffer transfer() と transferToFixedLength()
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/transfer

## Resizable buffers
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/resize
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



## intl.Segmenter
デフォルトで単語ごとに区切る
https://polypane.app/blog/using-the-intl-segmenter-api/

## WebGL API
Color management for WebGL
Color management for WebGL2

## JS - HTMLVideoElement: requestVideoFrameCallback()

## willReadFrequently
## cookieEnabled
### Mutually exclusive <details> elements
 Multiple <details> elements which use the same name attribute are mutually exclusive. When one member of the group is opened, all other members are closed.
https://developer.mozilla.org/ja/docs/Web/API/HTMLDetailsElement/open




## Alt text for generated content

## Unsanitized HTML parsing methods

    Document: parseHTMLUnsafe() static method
    Element: setHTMLUnsafe() method
    ShadowRoot: setHTMLUnsafe() method


## Registered custom properties
https://developer.mozilla.org/en-US/docs/Web/API/CSS/registerProperty_static

## API: Async clipboard
## Gradient interpolation
## Set methods
## Vertical form controls
## transform-box
## AbortSignal.any()
## checkVisibility()
## Extended constant expressions (WebAssembly)
## AVIF
