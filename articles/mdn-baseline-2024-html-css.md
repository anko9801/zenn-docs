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

このロゴは Baseline という主要なブラウザにおいて、ある機能がサポートされているかを表す指標です。これにより各ブラウザの挙動の違いに悩まされることなく、安心して開発に活用できます。ここで扱われる主要なブラウザとは以下のようなものです。

- Chrome (PC、Android)
- Edge
- Firefox (PC、Android)
- Safari (macOS、iOS)

そして Baseline は段階に応じて 3 種類のサポート状況に分けられます。

- Widely available: 2 年半以上すべての主要ブラウザで利用可能
- Newly avaliable: 最新バージョンのブラウザで利用可能
- Limited availability: 一部のブラウザでのみ利用可能

これを見れば、今までのように CSS や JavaScript などの最新技術を採用する際に [Can I Use](https://caniuse.com/) のサポート率を逐一確認する必要がなく、ブラウザの互換性を素早く把握できます。ちなみに 2 年半という基準は企業や組織がシステムを更新する際の一般的なライフサイクルに基づいて決められています。

そして今年 Newly Available となったものは [53 個](https://webstatus.dev/?q=baseline_date%3A2024-01-01..2024-12-31&sort=baseline_status_desc) でした！この記事ではそれら全てを 1 つ 1 つ簡単に紹介していきます。こんな最新技術が開発で使えるようになったんだ～って思いながら流し読みしてってください。

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

- トップレイヤーに昇格z-index を設定する必要がない
- ポップオーバー以外の部分をクリックすると自動的に閉じるライトディスミス機能
- キーボードやスクリーンリーダーのユーザーにも優しいアクセシビリティ

実装は簡単でポップオーバーの UI に popover 属性を付けてトグルボタンに popovertarget 属性を付けるだけ。

```html
<button popovertarget="my-popover">Open Popover</button>

<div id="my-popover" popover>
  <p>I am a popover with more information.<p>
</div>
```

@[codepen](https://codepen.io/anko9801/pen/VYZMjeQ)

さらにまだ実験的な機能ですが CSS Anchor Positioning と組み合わせることで

## backdrop-filter: 背景にぼかしや色変化を与える

Popover API の codepen で
https://coliss.com/articles/build-websites/operation/css/css-property-backdrop-filter.html


## AVIF: 次世代の高効率画像フォーマット

AVIF (AV1 Image File Format) は AV1 ビデオコーデックを基盤に開発された、静止画や画像シーケンス用のフォーマットです。その大きな特徴は JPEG PNG WebP といった既存フォーマットを超える圧縮効率と高画質です。

Google が開発した WebP も高圧縮率で知られていますが、MDN では「AVIF は WebP より圧縮率が高い」と評価されています。
(参考: [MDNドキュメント](https://developer.mozilla.org/ja/docs/Web/Media/Formats/Image_types#avif_%E7%94%BB%E5%83%8F))

![](/images/webp_vs_avif.webp)

かつて AVIF は一部のブラウザでサポートが不十分でしたが今回 Newly available となり「とりあえず AVIF で保存しておけば問題ない」ようになりました。


## content-visibility: レンダリングパフォーマンスの向上

通常ブラウザはすべてのコンテンツをレンダリングしているのですが、ページ読み込み時には不必要な画面外の要素までレンダリングしてしまいます。このレンダリングをスキップし、スクロール時に逐一レンダリングする `content-visibility: auto` がブラウザ互換となりました。

これにより巨大または複雑なサイトにおいてレンダリングパフォーマンスが大幅に向上することが見込めます。加えてレンダリングがスキップされているかを確認できる `checkVisibility()` 関数も Newly available となりました。

https://web.dev/articles/content-visibility?hl=ja

## スクロールのガタツキを防ぐ

スクロールバーの出現によってコンテンツが揺れる現象にイライラしたことはありませんか？
ユーザー体験
`scrollbar-gutter: stable`
`scrollbar-width` を使えば、スクロールバーの太さを制御できます。

## ブロックレイアウト上の align-content

これまで `align-content` プロパティはフレックスレイアウトやグリッドレイアウトで使用されていましたが、新たにブロックレイアウトでも使用可能になりました。

`align-content` はフレックスやグリッドレイアウトで複数行あるときに各行の間隔を調整するプロパティです。このプロパティは `justify-content` などの関連するプロパティと組み合わせて使用されますがそれぞれ役割が違います。

| プロパティ | 説明 | 対象レイアウト |
|---|---|---|
| justify-content | 主軸方向 (通常は水平方向) に各列の子要素の集まりを<br>コンテナ内で左寄せ・中央寄せなどに配置します | block flex grid |
| justify-items | 主軸方向に個々のアイテムをセル内で配置します | block grid |
| justify-self | 主軸方向に特定のアイテムをセル内で個別に配置します | block grid absolute |
| align-content | 交差軸方向 (通常は垂直方向) に各行の子要素の集まりを<br>コンテナ内で上寄せ・中央寄せなどに配置します | block flex grid |
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

## フォーム要素が縦書きにできる
## scroll-to-text フラグメント
https://labs.jxck.io/scroll-to-text-fragment/#:~:text=しかし,ない
scroll-to-text
::target-text

## text-wrap white-space-collapse: テキストの折り返しをより便利に

- `text-wrap`
- `white-space-collapse`
- `white-space`
https://developer.mozilla.org/ja/docs/Web/CSS/text-wrap
https://coliss.com/articles/build-websites/operation/css/about-text-wrap-balance.html

## ルビ
ruby-align
ruby-position

親に向かってなんだその

## font-size-adjust: 異なるフォントが混ざっても綺麗なタイポグラフィ 

英語圏では異なるフォントを同じ文章で扱うとき、フォントサイズで揃えても文字の大きさがズレてしまってピッタリ合いません。そこでフォントサイズではなくフォントの小文字の高さ `x-height` で揃えることで合わせようというのが `font-size-adjust` です。

`x-height = font-size * font-size-adjust` が小文字の高さとなるようにフォントサイズを調整してくれます。

@[codepen](https://codepen.io/anko9801/pen/MYgJQoo)

## @property: CSS カスタムプロパティの新たな表現
カスタムプロパティに型のチェックやデフォルト値の設定、プロパティの値の継承するかを設定することができるようになります。

これは CSS では `@property` JavaScript では `CSS.registerProperty()` によって設定でき、特に JavaScript で動的にカスタムプロパティを追加する場合により安全に変数を初期化できるようになります。例えば次のコードは構文は `<color>` 継承せず初期値は `#c0ffee` となります。
```css
@property --logo-color {
  syntax: "<color>";
  inherits: false;
  initial-value: #c0ffee;
}
```

## さまざまな色空間での相対色とグラデーション

OKLAB OKLCH など比較的最近実装された色空間での相対色とグラデーションの構文が実装されました。

```css
html {
  /* 背景は基準色の明度を下げて、文字は明度を上げた */
  --base: green;
  --base-bg: oklch(from var(--base) calc(l * 0.75) c h / 0.5);
  --base-text: oklch(from var(--base) calc(l * 1.5) c h);
}

body {
  background: var(--base-bg);
  color: var(--base-text);
}

.gradation {
  /* oklch 色空間において blue から red までのグラデーション */
  background: linear-gradient(in oklch to right, blue, red);
}
```
例えば OKLCH と相対色表現を組み合わせることで 1 つの基準色からライトテーマやダークテーマの色などを生成する保守性の高いカラーパレットが作成できます。
@[codepen](https://codepen.io/anko9801/pen/azoygqV)

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

## :state()
https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet
https://developer.mozilla.org/en-US/docs/Web/CSS/:state

## CSS Animation 関連の充実
- offset-position と offsetpath の値
  - offset
- zoom: レイアウトが変わる transform-scale
重なるようにその要素が大きくなる
`transform: scale(3);`
レイアウトの再計算をする
`zoom: 3;`
- transform-box: 
- transition-behavior: 
https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior

## @page 文書を印刷するときに一部の CSS プロパティを変更する



## CSS ステップ関数 `round()` `mod()` `rem()`
四捨五入などを計算できる `round()` と剰余を計算する `mod()` `rem()` が追加されました。

`mod()` `rem()` の違いはマイナスになったときの挙動で、それぞれ割る数と割られる数の符号に依存して返す符号が変わります。
```js
mod(?, ±) = ±
rem(±, ?) = ±
```

## まとめ

## 参考記事
- https://webstatus.dev/?q=baseline_date%3A2024-01-01..2024-12-31&sort=baseline_status_desc&num=100
- https://web.dev/series/baseline-newly-available?hl=ja
- https://web-platform-dx.github.io/web-features/
