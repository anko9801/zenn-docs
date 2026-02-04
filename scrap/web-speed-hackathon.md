# Web Speed Hackathon 2024
レギュとしては任意の記事を参照してもよいので満点を目指すって感じです！

## 公式ルール
- 0.1vCPU / 512MB のサーバーを 1 つ使用
- Chrome最新版での E2E テストと VRT をクリアすること
- 漫画ビューアーの画像に難読化を施すこと

## なにをどうやって計測する？

Web Speed Hackathon のバックエンド版 ISUCON で「推測するな、計測せよ」の精神をこちらにも適用しちゃいましょう！

今回チューニングで改善すべき数値とは [Lighthouse v10 のスコア](https://developer.chrome.com/docs/lighthouse/performance/performance-scoring?hl=ja#lighthouse-10)です。

| 指標 | 説明 | 推奨される数値 |
|---|---|---|
| [First Contentful Paint (FCP)](https://web.dev/articles/fcp?hl=ja) | 最初の DOM コンテンツを描画するまでの時間  | 1.8s 以内 |
| [Largest Contentful Paint (LCP)](https://web.dev/articles/lcp?hl=ja) | 最も大きな要素を描画するまでの時間 | 2.5s 以内 |
| [Speed Index (SI)](https://developer.chrome.com/docs/lighthouse/performance/speed-index?hl=ja) | 時間ごとの視覚的な変化量 | 3.4s 以内 |
| [Total Blocking Time (TBT)](https://web.dev/articles/tbt?hl=ja) | ユーザー操作がブロックされてる合計の時間 | 200ms 未満 |
| [Cumulative Layout Shift (CLS)](https://web.dev/articles/cls?hl=ja) | 視覚的な安定性 | 0.1 未満 |
| [Interaction to Next Paint (INP)](https://web.dev/articles/inp?hl=ja) | インタラクティブ性 | 200ms 未満 |

これらを改善していけばいいのですが Web Speed Hackathon、いつも凶悪なサイトを作り込んでいるため Lighthouse でも重すぎて正しく評価できません。あと単純に測るまで時間がかかります。それなので次のメトリクスを測ってそれを改善させていきます。

- バンドルサイズ
- bundle analyzer
- devtools (Performance タブ・Network タブ・Lighthouse タブ)

フロントエンドにおいてバンドルサイズはネックになりやすいらしいのでそこをきちんと測って改善すると良いそう。そこでバンドルサイズを測るスクリプトをささっと書きました。bundle analyzer については [ESBuild の bundle analyzer のサイト](https://esbuild.github.io/analyze/) にメタファイルを投げ込むことで見れます。

```bash:profile.sh
#!/bin/bash

function convert_MB() {
  FILE=$(echo $1 | sed -e 's/.*\///')
  BYTE=$(wc -c $1 | awk '{print $1}')
  echo "scale=2; $BYTE / 1024 / 1024" | bc | xargs printf "$FILE: %.2fMB\n"
}

pnpm run build
convert_MB ./workspaces/client/dist/client.global.js
convert_MB ./workspaces/client/dist/serviceworker.global.js
convert_MB ./workspaces/server/dist/server.js
pnpm run start
```

バンドルサイズ
- client: 119.89MB
- serviceworker: 6.14MB
- server: 36.89MB

bundle analyzer
![](https://storage.googleapis.com/zenn-user-upload/dcc1b16c8869-20250103.png)

スコア: 27.75 / 700.00

# バンドルサイズを削る

バンドルとは JavaScript のコードを 1 ファイルにまとめることです。バンドルサイズを削減すると、通信量が減り、モバイル環境や低スペックサーバーでも快適に動作します。

削減方法
- ソースマップはでかいので削ろう
- ESM 版ライブラリを使おう
- 不要なファイルをバンドルから追い出そう
- minify や code splitting も適用してこう

## ビルドコンフィグの最適化
今回は tsup でビルドしており、
モジュール形式は CommonJS (CJS) より treeshaking が効く ES Module (ESM) を選ぼう。
クライアントを ESM 形式に置き換えるとプロファイルと minify ができなくなってしまう為いったん iife のまま動かします。また tsup では ESM 形式以外については code splitting が未実装とのこと。

| やること | 効果 | メモ |
|---|---|---|
| ソースマップを削除 | 119.89MB → 46.16MB | ビルドファイルと元のソースコードの対応を示してデバッグ時のスタックトレースに役立ちますがとてもでかい |
| minify | 46.16MB → 36.62MB |
| production モードでビルド | 36.62MB → 36.38MB |
| ビルドターゲットを最新版の Chrome に絞る | 36.38MB → 36.29MB |
| treeshaking | 36.29MB → 36.30MB | 設定するだけ大きくなっていそうですがあとで少し小さくなるので設定しておきます。 |


計 70% の削減に成功しました！

https://github.com/anko9801/web-speed-hackathon-2024/commit/24e38a433a02c9bc7a43739f9e80af46c4950694

## ダイアログの内容とヒーロー画像を配信する

次は Bundle Analyzer でサイズが大きく不要なパッケージをバンドルから追い出します。

| やること | 効果 |
|---|---|
| [Dialog の内容をサーバー配信](https://github.com/anko9801/web-speed-hackathon-2024/commit/381fb982774637d93f7744f44467be6b2ac6dc8a) | -12.96MB |
| [heroImage を画像化](https://github.com/anko9801/web-speed-hackathon-2024/commit/52aaf895aad63451111725a8bb6b0cb40d70aeba) | -12.27MB |

これで 36.30MB → 11.07MB と 25MB 削減できました！

## 不要なパッケージの削除

| やること | 効果 | メモ |
|---|---|---|
| [mui の treeshaking](https://github.com/anko9801/web-speed-hackathon-2024/commit/a0d27359fbce99284cc2977f6c3ea81b3b3ab8d7) | -4.03MB |
| [magika](https://github.com/anko9801/web-speed-hackathon-2024/commit/5308da2dd37bdbb1393c20e99603055334f9c79a) | -2.23MB | ファイルからファイル拡張子を DeepLearning で特定するライブラリ、既に確定しているのでそれを活用します |
| [unicode-collation-algorithm2](https://github.com/anko9801/web-speed-hackathon-2024/commit/8d6e9be6717a3ce047ab90ad46c91749539c76cb) | -1.77MB | 文字の比較に使用していたのを Intl.Collator で代替 |
| [moment-timezone](https://github.com/anko9801/web-speed-hackathon-2024/commit/965466471d32bf070ef044f5c31dc070b8416241) | -0.81MB | [YOU MIGHT NOT NEED *](https://youmightnotneed.com/)|
| [polyfill](https://github.com/anko9801/web-speed-hackathon-2024/commit/1d0064e89e272740bde05b348970877802e59997) | -0.65MB | 最新版の Chrome のみに対応すればいいので polyfill は必要ありません。 |
| [three.js](https://github.com/anko9801/web-speed-hackathon-2024/commit/3a309068b7621c567c23c568378bd14fd04de442) | -0.47MB | |
| [jQuery](https://github.com/anko9801/web-speed-hackathon-2024/commit/852be2cbc2d615c4b721f93158ea3fa4590f4c8d) | -0.09MB | 単に置換するとロードされなくて戸惑うけど DOMContentLoaded のイベントリスナーをセットするのがイベント発火に間に合ってないことが原因なので後ろに await を持ってくることで解決しました。[YOU MIGHT NOT NEED *](https://youmightnotneed.com/) |
| [lodash](https://github.com/anko9801/web-speed-hackathon-2024/commit/23a426d74cca57a3cc1f5b4d51e8d3f5270db9d5) | -0.07MB | [YOU MIGHT NOT NEED *](https://youmightnotneed.com/) |
| [underscore](https://github.com/anko9801/web-speed-hackathon-2024/commit/761f88d71b31303b82e18b2972f69eafd48fd1b8) | -0.02MB |
| [zustand]() | -1.5kB |
| [usehooks]() | -0.1kB |

これで 11.07MB → 0.92MB (1MB 未満) の削減に成功しました！今回ここまでやりましたがある程度小さくなると他の箇所にボトルネックに移るので臨機応変に対応していった方が良さそうです。

## バンドルを admin と client に分ける
どのページでも client と admin どちらも配信していたので別々にバンドルして出し分けて上げます。
- client 0.39MB
- admin 0.70MB

https://github.com/anko9801/web-speed-hackathon-2024/commit/490b51f74c5dc70f95de591be49564740b5e55bd

これで bundle analyzer のお仕事は見納めです。120MB → 0.39MB と 99.7% も削減してくれました！ありがとう！

# リクエストの最適化

次は devtools の Network タブを見てみます。トップページで測ったのをよく見ると 651 リクエスト 100 MB の通信 187MB のリソース 1.1min の通信時間とめっちゃでかい通信してます。ここも最適化していきましょう！

- 既に届いたレスポンスを活用する (651 → 154 reqs)
  - [トップページ](https://github.com/anko9801/web-speed-hackathon-2024/commit/e2fc09e9ed53cb0fbb126fce563bfa0eae43dc5a)や[作品詳細ページ](https://github.com/anko9801/web-speed-hackathon-2024/commit/e2fc09e9ed53cb0fbb126fce563bfa0eae43dc5a)でデータを捨ててもう一度リクエストを飛ばしていたので再利用しました
- [Cache-Control を入れる](https://github.com/anko9801/web-speed-hackathon-2024/commit/d51eba4d2024fe9f548a76ddc93fa8abe6ae759a)
  - 静的ファイルはキャッシュすることで再度サイトに訪れたときリクエストを飛ばさずにメモリからロードされます
  - スコアには影響なさそうですがユーザー体験に効果絶大なので入れておきます
- ServiceWorker の並列化
  - [並列数を 5 から上限なし](https://github.com/anko9801/web-speed-hackathon-2024/commit/6250217bcec5b9c846bc3dc75413a469fc99ebd4)
  - [jitter をなくしました](https://github.com/anko9801/web-speed-hackathon-2024/commit/fbd7e58164a09066ed7fcfc73a6b2778fb89d9a8)
- [解凍をブラウザで行う](https://github.com/anko9801/web-speed-hackathon-2024/commit/5afcac423eb93b74ac8892afa223c0bff4588919)
  - サーバーで圧縮して ServiceWorker で解凍してクライアントに送るみたいなことをしていたのでクライアントまで圧縮を掛けておきます。
- いらないデータを送らない

# フォント・画像を最適化

画像形式 tier は BMP < PNG < JPEG < WebP < AVIF・JPEG XL で JPEG XL は Chrome で対応していないので AVIF で殴れば良さそう。JavaScript が実行される前にリクエストする `rel='preload'` とスクロールした際に読み込みする `loading='lazy'` を組み合わせてそれぞれの画像のロードタイミングを操り、高速な描画が実現できます。

- [画像の preload を止める](https://github.com/anko9801/web-speed-hackathon-2024/commit/b335bcb1ddfbdc59a4b2685cda0f383067b8f539)
  - 画面に入っていない画像や要らない画像までむやみに preload するとそれが終わるまで他の描画がブロックされるのでやめます。
- [loading='lazy'](https://github.com/anko9801/web-speed-hackathon-2024/commit/27368b01caa218cafaabdebb5353212468260c75)
  - 画面に入っていない画像達を lazy loading します。画面内に収まっている画像に付けるのは逆効果になります。
- [フォントを削除](https://github.com/anko9801/web-speed-hackathon-2024/commit/65e2bdc2660e18306523c9fcfa362a87e2788882)
  - 一見 NotoSansJP が使われているように見えますがデフォルトの NotoSansJP を使っているので要りません。紛らわしい～
- [cyber-toon.svg の不要な部分を削る](https://github.com/anko9801/web-speed-hackathon-2024/commit/fa63abc82230a5c52462b92221bd322d86964038)
- [AVIF に変換！](https://github.com/anko9801/web-speed-hackathon-2024/commit/aa937abfa9c930616a80b6c140a65010e1d29747)
- ヒーロー画像を解像度によって srcset sizes picture で出し分け
- fetchpriority="high"
- シンプルな

計測してないのですが AVIF に変換したらとても速くなりました。これで 1.1s で読み込み完了するようになりました！

# INP を上げる
- [ReDoS](https://github.com/anko9801/web-speed-hackathon-2024/commit/fe114795c5068228c8233db65854775f5bb9782c)
- [検索 debounce](https://github.com/anko9801/web-speed-hackathon-2024/commit/2c9efa1adde778c203e8e21ce78016a7c50f8504)
- passive: true

# CLS を上げる


https://blog.jxck.io/entries/2016-04-16/stale-while-revalidate.html
