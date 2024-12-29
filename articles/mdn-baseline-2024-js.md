---
title: "MDN Baseline Newly Available 2024 を振り返ってみよう"
emoji: "✅️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

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