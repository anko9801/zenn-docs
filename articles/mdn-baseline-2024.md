---
title: "MDN Baseline Newly Available 2024 を振り返ってみよう"
emoji: "✅️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## みなさん Baseline をご存知でしょうか

MDN Web Docs を読んでいると、次のようなロゴを目にしませんか。

![](/images/newly_available.png)

このロゴは Baseline というすべての主要なブラウザ (Chrome、Edge、Firefox、Safari など) である機能がサポートされているかを表す指標です。各ブラウザの挙動の違いに悩まされることなく、安心して開発に活用できます。

Baseline は段階に応じて 3 種類のサポート状況に分けられます。

- Widely available: 2 年半以上すべての主要ブラウザで利用可能
- Newly avaliable: 最新バージョンのブラウザで利用可能
- Limited availability: 一部のブラウザでのみ利用可能

これを見れば、今までのように CSS や JavaScript などの最新技術を採用する際に [Can I Use](https://caniuse.com/) を逐一確認する必要がなく、ブラウザの互換性を素早く把握できます。ちなみに 2 年半という基準は企業や組織がシステムを更新する際の一般的なライフサイクルに基づいて決められています。

2024 年に Newly Available
こんな最新技術が開発で使えるんだ～って思いながら読んでってください。

## 宣言型 Shadow DOM

Web Comopnents の内の Shadow DOM が HTML 上で宣言的に書けるようになりました。

Web Components とは

再利用できるようにしたのが Web Components です。Web Components

- Shadow DOM
- Custom Elements
- HTML テンプレート

動的 Declarative Shadow DOM

Document: parseHTMLUnsafe() static method
Element: setHTMLUnsafe() method
ShadowRoot: setHTMLUnsafe() method

[Shadow DOM](https://developer.mozilla.org/ja/docs/Web/API/Web_components/Using_shadow_DOM) は再利用可能なコンポーネント
これまでは JavaScript

https://azukiazusa.dev/blog/declarative-shadow-dom/

- declarative shadow dom と HTML Templates の違い

```html
<div id="host">
  <template shadowrootmode="open">
    <span>I'm in the shadow DOM</span>
  </template>
</div>
```

## Popover API

従来は JavaScript を用いて実装しなければならなかったのを
ポップオーバーはよく使われるのに対し、実装が大変でした。

- 最上位レイヤーに昇格
  - z-index を指定せずに
- ライトディスミス機能
- デフォルトのフォーカス管理
- アクセシブルなキーボードバインディング
- アクセシブルなコンポーネントバインディング

<button popovertarget="my-popover"> Open Popover </button>

<div id="my-popover" popover>
  <p>I am a popover with more information.<p>
</div>
anchor position を popover api に

## AVIF

## content-visibility: コンテンツ最適化

`content-visibility`
要素がビューポート外にあるとき描画や計算を笑楽できるため、ブラウザの負担が軽減され、ページの読み込み速度が向上します。
- レンダリングの最適化、レイアウト再計算の削減によりパフォーマンスが向上します。

- content-visibility
- checkVisibility()
  - visibility プロパティによる不可視状態や opacity プロパティによる不透明度によるチェックも行えます。

https://web.dev/articles/content-visibility?hl=ja

## offset-position と offsetpath の値

## ブロックレイアウト上の align-content

これまで `align-content` プロパティはフレックスレイアウトやグリッドレイアウトで使用されていましたが、新たにブロックレイアウトでも使用可能になりました。

`align-content` はフレックスやグリッドレイアウトで複数行あるときに各行の間隔を調整するプロパティです。このプロパティは `justify-content` などの関連するプロパティと組み合わせて使用されますがそれぞれ役割が違います。

| プロパティ | 説明 | 対象レイアウト |
|---|---|---|
| justify-content | 主軸方向 (通常は水平方向) に各列の子要素の集まりを<br>コンテナ内で左寄せ・中央寄せなどに配置します | block flex grid |
| justify-items | 主軸方向に個々のアイテムをセル内で配置します | block grid |
| justify-self | 主軸方向に特定のアイテムをセル内で個別に配置します | block grid absolute |
| align-content | 交差軸方向 (通常は垂直方向) に各行の子要素の集まりを<br>コンテナ内で配置します。 | block flex grid |
| align-items | 交差軸方向に個々のアイテムをセル内で配置します | flex grid |
| align-self | 交差軸方向に特定のアイテムをセル内で個別に設定します | flex grid absolute |

使用例として要素を垂直方向 (交差軸) で中央揃えする場合において、これまで以下のようにフレックスレイアウトを設定していました。
```css
display: flex;
align-items: center;
```
ブロックレイアウトでの対応により単に次のように設定するだけで垂直方向の中央揃えができます。
```css
align-content: center;
```

## text-wrap white-space-collapse: テキストの折り返しをより便利に

- `text-wrap`
- `white-space-collapse`
- `white-space`
https://developer.mozilla.org/ja/docs/Web/CSS/text-wrap
https://coliss.com/articles/build-websites/operation/css/about-text-wrap-balance.html

## font-size-adjust: 異なるフォントが混ざっても綺麗なタイポグラフィ 

英語圏では異なるフォントを同じ文章で扱うとき、フォントサイズで揃えても文字の大きさがズレてしまってピッタリ合いません。そこでフォントサイズではなくフォントの小文字の高さ `x-height` で揃えることで合わせようというのが `font-size-adjust` です。

`x-height = font-size * font-size-adjust` が小文字の高さとなるようにフォントサイズを調整してくれます。

@[codepen](https://codepen.io/anko9801/pen/MYgJQoo)

## from <color>: 相対的な色表現
例えば OKLCH と Relative colors を組み合わせることで 1 つの基準色からライトテーマやダークテーマの色などを生成する保守性の高いカラーパレットが作成できます。
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
変数として柔軟に設定できるので

## light-dark(): ダークテーマのスタイルを簡単に当てられる！


`light-dark()` は [CSS Color Module Level 5](https://drafts.csswg.org/css-color-5/#light-dark) で追加されたユーティリティ関数で従来の `prefers-color-scheme` メディアクエリよりもライトテーマとダークテーマで異なるスタイルを簡潔に当てられます。

`light-dark()` はユーザーがライトモード (または不明) のときに第一引数を出力し、ダークモードのときに第二引数を出力します。具体的には `prefers-color-scheme` メディアクエリでこのように実装していたところを
```css
:root {
  color-scheme: light dark;
}

@media (prefers-color-scheme: light) {
  body {
    color: #333b3c;
    background-color: #efedea;
  }
}

@media (prefers-color-scheme: dark) {
  body {
    color: #efefec;
    background-color: #223a2c;
  }
}
```
`light-dark()` 関数でこのように書けます。
```css
:root {
  color-scheme: light dark;
}

body {
  color: light-dark(#333b3c, #efefec);
  background-color: light-dark(#efedea, #223a2c);
}
```
画像を切り替えたりしてもよさそう

## Gradient interpolation
## backdrop-filter: 背景にぼかしや色変化を与える
## Vertical form controls
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

## @property: CSS カスタムプロパティの新たな表現
カスタムプロパティに型やデフォルト値なども含めて表現できるようになりました。

JavaScript API である CSS.registerProperty() と、同等の仕組みを CSS から直接利用するための @property
以下は @property の例です。--my-color という名前でカラーのみを受け付けるカスタムプロパティを定義しています。デフォルトカラーが宣言されており、inherits: false により、親要素で定義した値は子孫に継承されていないことがわかります。
https://developer.mozilla.org/en-US/docs/Web/API/CSS/registerProperty_static


```css
--logo-color: #c0ffee
```
```css
@property --logo-color {
  syntax: "<color>";
  inherits: false;
  initial-value: #c0ffee;
}
```

## CSS ステップ関数 `round()` `mod()` `rem()`

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

## AbortSignal.any()
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

## intl.Segmenter: 文章を単語ごとに分割する
## WebGL API
Color management for WebGL
Color management for WebGL2

## requestVideoFrameCallback(): HTMLVideoElement
## willReadFrequently: Canvas
## cookie の有効性
```
if (!navigator.cookieEnabled) {
  // ブラウザーが対応していないか、クッキーが設定されることをブロックしています。
}
```
## Mutually exclusive <details> elements
 Multiple <details> elements which use the same name attribute are mutually exclusive. When one member of the group is opened, all other members are closed.
https://developer.mozilla.org/ja/docs/Web/API/HTMLDetailsElement/open

## Alt text for generated content
## API: Async clipboard
## Extended constant expressions (WebAssembly)
## 参考記事
- https://webstatus.dev/?q=baseline_date%3A2024-01-01..2024-12-31&sort=baseline_status_desc&num=100
- https://web.dev/series/baseline-newly-available?hl=ja
- https://web-platform-dx.github.io/web-features/