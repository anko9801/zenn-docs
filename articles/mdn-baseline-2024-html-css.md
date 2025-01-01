---
title: "MDN Baseline Newly Available 2024 を振り返ってみよう (HTML CSS 編)"
emoji: "✅️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend", "Tech"]
published: true
---

## みなさん Baseline をご存知でしょうか

MDN Web Docs を読んでいると、次のようなロゴを目にしませんか。

![](/images/newly_available.png)

このロゴは Baseline という主要なブラウザにおいて、ある機能がサポートされているかを表す指標です。これにより各ブラウザの挙動の違いに悩まされることなく、安心して開発に活用できます。Baseline がサポートを保証している主要なブラウザとは次のブラウザです。

- Chrome (PC、Android)
- Edge
- Firefox (PC、Android)
- Safari (macOS、iOS)

そして Baseline は段階に応じて 3 種類のサポート状況に分けられます。

- Widely available: 2 年半以上すべての主要ブラウザで利用可能
- Newly avaliable: 最新バージョンのブラウザで利用可能
- Limited availability: 一部のブラウザでのみ利用可能

これを見れば、今までのように CSS や JavaScript などの最新技術を採用する際に [Can I Use](https://caniuse.com/) のサポート率を逐一確認する必要がなく、ブラウザの互換性を素早く把握できます。ちなみに 2 年半という基準は企業や組織がシステムを更新する際の一般的なライフサイクルに基づいて決められています。

そして今年 Newly Available となったものは [53 個](https://webstatus.dev/?q=baseline_date%3A2024-01-01..2024-12-31&sort=baseline_status_desc) でした！この記事ではそれら全てを 1 つ 1 つ簡単に紹介していきます。前半は HTML CSS に絞ります。こんな最新技術が開発で使えるようになったんだ～って思いながら流し読みしてってください。

## Declarative Shadow DOM

Shadow DOM は Web Components を構成する重要な要素で、コンポーネント内の DOM を外部から分離し、スタイルやスクリプトをカプセル化する技術です。これまでは JavaScript を用いた構築が必要で SSR との組み合わせが難しい課題がありました。今回サポートされた Declarative Shadow DOM によって HTML 上で宣言的に書けるようになり、SSR に対応しました。

実装としては `<template>` タグに `shadowrootmode` 属性を与えることで Shadow DOM が作れます。
```html
<hello-shadow-dom>
  <template shadowrootmode="open">
    <span>I'm in the shadow DOM</span>
    <style>~~</style>
    <script>~~</script>
  </template>
</hello-shadow-dom>
```

また Declarative Shadow DOM を動的に挿入する為に `innerHTML` を使うと XSS 攻撃などによる Shadow DOM のインジェクションなどセキュリティ上の問題が発生する為エラーが生じます。それでも使いたい場合には `document.parseHTMLUnsafe()` `element.setHTMLUnsafe()` が使えます。ただし上のような攻撃を防ぐことはできないので注意が必要です。

https://azukiazusa.dev/blog/declarative-shadow-dom/
https://zenn.dev/cybozu_frontend/articles/web-standardized-component-in-server-and-client

## CustomStateSet: カスタム要素も状態を扱えるように

Web Components の 1 要素であるカスタム要素において `:hover` `:visited` などの疑似クラスのような状態を設定できるカスタム状態を扱うことが出来るようになりました。

JavaScript で CustomStateSet という API を通すことでカスタム状態を定義できます。 CustomStateSet はカスタム要素の中で `attachInternals()` メソッドから得られる ElementInternals オブジェクトの states プロパティで操作できます。そしてそこで追加されたカスタム状態は CSS で `:state()` 擬似クラスを用いてアクセスできます。

## Popover API

従来は JavaScript を用いて実装しなければならなかったのを
ポップオーバーはよく使われるのに対し、実装が大変でした。

- トップレイヤーに昇格z-index を設定する必要がない
- ポップオーバー以外の部分をクリックすると自動的に閉じるライトディスミス機能
- キーボードやスクリーンリーダーのユーザーにも優しいアクセシビリティ

実装は簡単でポップオーバーの UI に popover 属性を付けてトグルボタンに popovertarget 属性を付けるだけでできます。

```html
<button popovertarget="my-popover">Open Popover</button>

<div id="my-popover" popover>
  <p>I am a popover with more information.<p>
</div>
```

@[codepen](https://codepen.io/anko9801/pen/VYZMjeQ)

さらにまだ実験的な機能ですが特定の要素からの相対位置で指定できる Anchor Positioning があります。これにより JavaScript を使わなくても toolchip を実装することができます。

## backdrop-filter: 背景にぼかしや色変化を与える

背景にぼかしや色変化を入れることができ、重要な情報をわかりやすく伝えるようになりました。上のポップオーバーを開くと背景にブラーを掛けているのも `backdrop-filter: blur(4px)` と指定しています。詳しくは次の記事を参考にしてください。

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

## scrollbar-gutter: スクロールのガタツキを防ぐ

スクロールバーの出現によってコンテンツが揺れてしまうという現象が時折発生します。これには `scrollbar-gutter: stable` とすることでスクロールできなくてもスクロールバーの幅をレイアウトに組み込むことでこのガタツキを防ぐことができます。また `scrollbar-width` によってスクロールバーの太さを調節することも出来るようになりました。

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

## フォーム要素を縦書きにできる

`<input>` `<textarea>` などのフォーム要素で縦書きがサポートされました。

@[codepen](https://codepen.io/anko9801/pen/vEBeWEd)

## テキストフラグメント

ページ内の特定へジャンプするにはこれまで id 属性を持つ要素にしか対応していませんでしたが、テキストフラグメントを活用することでどのテキストも指定できるように拡張されました。

テキストフラグメントの構文は `#:~:text=[prefix-,]textStart[,textEnd][,-suffix]` で開始位置・終了位置・prefix・suffix を指定できます。これに一致した一番最初のテキストにスクロールし、ハイライトされます。

| フラグメント | 説明 |
|---|---|
| [`#テキストフラグメント`](#テキストフラグメント) | 従来通り id 属性に基づくリンク |
| [`#:~:text=テキストフラグメント`](#:~:text=フラグメントディレクティブ) | ページ内の「テキストフラグメント」という文字列が最初に現れる箇所にスクロール |
| [`#:~:text=これまでページ内の,ように拡張されました`](#:~:text=フラグメントディレクティブ) | 特定の開始位置と終了位置を指定する |

指定したテキストフラグメントに飛び、ハイライトされます。ハイライトされるテキストについては `::target-text` によってスタイルを書き換えられます。
```css
::target-text {
  background-color: rebeccapurple;
  color: white;
}
```

## text-wrap: テキストの折り返しをより綺麗に

テキストの 1 行の長さが丁度同じように折り返すなどといったアルゴリズムがサポートされました。

`text-wrap` はよりよいアルゴリズムでテキストの折り返しを制御する組版の為のプロパティ、`white-space-collapse` は連続した空白やタブ、改行を 1 つの空白に置き換えたり、削除したりするプロパティです。

これまで使われていた `white-space` はこれから `text-wrap` `white-space-collapse` を用いることで表現できるショートハンドプロパティとなります。

@[codepen](https://codepen.io/anko9801/pen/JoPrOdW)

## ルビの位置の調整

ルビは文字の上に注釈する文字のことで日本語の振り仮名が発祥の概念です。もともとはフォントサイズが Perl Diamond などの宝石の名前として表されており、そのうち Ruby が日本活版の業界で振り仮名として使うようになったのが由来です。そのルビの位置を調節するプロパティ `ruby-align` `ruby-position` がサポートされました。

@[codepen](https://codepen.io/anko9801/pen/KwPvLbj)

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

OKLAB OKLCH など比較的最近実装された色空間での相対色とグラデーションの構文が実装されました。例えば OKLCH と相対色表現を組み合わせることで 1 つの基準色からライトテーマやダークテーマの色などを生成する保守性の高いカラーパレットが作成できます。

```css
.badge {
  background: oklch(from green calc(l * 0.75) c h / 0.5);
  color: oklch(from green calc(l * 1.5) c h);
}

.gradation {
  background: linear-gradient(in oklch to right, blue, red);
}
```
@[codepen](https://codepen.io/anko9801/pen/azoygqV)

## light-dark(): ダークテーマのスタイルを簡単に当てられる

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

## CSS Animation の機能の充実

CSS Animation に関していくつか新しくサポートされました。

| プロパティ名 | 説明 |
|---|---|
| offset | 要素を動かすときの軌跡を表現するプロパティ |
| zoom | 拡大するプロパティ。transform-scale は拡大しても要素の大きさが変わらず、zoom はレイアウトを再計算しながら拡大する。 |
| transform-box | 変換の基準となる要素の枠組みを設定するプロパティ |
| [transition-behavior](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior) | `background-image` `visibility` などの離散プロパティなども動かせるようにできる |

## @page Web ページの印刷時のスタイルを設定

Web ページを印刷したいときに `@page` ルールでページの余白や用紙サイズ、縦向き横向きなどを指定することができます。

これは `@media print` と合わせて CSS 組版である Vivliostyle でよく使われています。`@media print` は印刷時にスタイルを変更する際に使えて、例えばヘッダーやナビゲーションを消したり、段組みで縦に 2 列に文章を分けたりできます。

```css
@page {
  size: A4 portrait;
  margin: 8mm 2.5mm 2.5mm 2.5mm;

  @top-left {
    content: "Newly available 2024";
  }

  @top-right {
    content: "Page " counter(pageNumber);
  }
}

@media print {
  body {
    column-count: 2;
    column-rule: 1px solid;
    column-gap: 6mm;
  }
}
```

## CSS ステップ関数 `round()` `mod()` `rem()`
四捨五入などを計算できる `round()` と剰余を計算する `mod()` `rem()` が追加されました。

`mod()` `rem()` の違いはマイナスになったときの挙動で、それぞれ割る数と割られる数の符号に依存して返す符号が変わります。
```js
mod(?, ±) = ±
rem(±, ?) = ±
```

## まとめ

Web フロントエンドにおいて既に知っていても使いにくかった技術が存分に使えるようになった Newly available の紹介でした。今年の Newly available は Web Components, i18n, アクセシビリティ周りがよく進んでいる感じがありました。次の記事で JavaScript、API 編も紹介するのでぜひそちらも読んでみてください。

## 参考記事
https://web.dev/series/baseline-newly-available?hl=ja
https://web-platform-dx.github.io/web-features/
