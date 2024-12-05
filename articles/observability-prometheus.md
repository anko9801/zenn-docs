---
title: "Web アプリケーション全体の監視環境の構築と運用"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

インシデントマネジメント

## インシデントマネジメント

オブザーバビリティによってデータを可視化し、インシデント管理・振り返り
インシデントに対応









https://speakerdeck.com/grimoh/jun-hajian-teirugaguan-cha-siteinai-dekao-eruinsidentomanezimento













迅速にインシデントに対応する

- 属人化してるインシデント対応
- 経験と勘所
- 深いドメイン知識
- システムの複雑性、マイクロサービス
- インシデント対応に時間を避けない
- インシデント発生時の情報共有やプロセスの効率化

- 頻発する問題について同じ解決策を何度も調べている

根拠となるデータを元に観察し原因を推定する、オブザーバビリティ
頻発する問題について同じ解決策を何度も調べている状況のときランブックが活用できる。

## オブザーバビリティとは

- インシデントマネジメント
  - 予期せぬサービスの中断や品質低下といったインシデントが発生した場合に、迅速かつ効果的に対応するためのプロセスと体制
  - インシデントの速やかな解決、システムやサービスを運用する担当者の負担軽減、今後のインシデント抑制
  - 信頼性の向上
    - システムの安全性の強化
  - ユーザー体験の向上

システムがどのような状態になったとしても、それがどんなに斬新で奇妙なものであっても、**どれだけ理解し説明できるかを示す尺度**です。
もし、新しいコードをデプロイする必要がなく、どんな斬新で奇妙な状態でも理解できるなら、オブザーバビリティがあると言えます。
検知・対応が遅れればユーザーが満足に利用できない

各サービスからテレメトリーを計装してオブザーバビリティバックエンドに流し込む

原義的にはオブザーバビリティ (可観測性) とはシステムの出力を調べることで内部の状態を測定する能力を指す。そして監視は何が起きたかを見続けること、オブザーバビリティはなぜそれが起きたかを探り出すこと。監視を包含する。

現代におけるオブザーバビリティがどう効果を発揮するか

- デバッグが必要なコードがシステムのどこにあるかを迷いなく見つけ出す
- システムのパフォーマンス改善
- マイクロサービスを立てまくってもスケールする
- アラートによって異変にすぐ気付ける
- IaC みたいにインフラアーキテクチャの生きるドキュメントにもなる

ユーザー視点に立って改善すべき

そして構築と組織が互いに出来上がっていかないといけない

リード役とファシリテーション役

巻き込み

## どうしてなにを観測するのか

継続的プロファイル

### 監視の例

CPU メモリ ネットワーク監視

- トラフィック量
- ダウンタイム → サーバーの死活管理
- リソース不足・過多 → AWS のリソースとインフラコストのバランスを取る

データベース監視

- クエリ回数、合計検索時間、平均検索行数 → パフォーマンス改善
- write / read 優劣

トレース

- 分散トレース (クライアントの体験を起点に DB アクセスまでのスタックトレース) → エラーの追跡
    - ソースマップと連携してわかりやすくなる

メトリクス

- サービスのレスポンスタイム・スループット → パフォーマンス改善
- デプロイ前後のレスポンス・エラーの変化 → 問題の発見
- Web Vitals → パフォーマンス改善

プロファイル

- フレームグラフによるボトルネックの発見 → パフォーマンス改善

リアルユーザーモニタリング (RUM)

- ユーザーからの報告なしで異常を発見できる
- Content Security Policy のエラーの監視 → セキュリティの向上

セッションリプレイ

- ユーザーの行動を録画してダッシュボードで再生 → エラーの再現・UX 改善

## サービスの検討

OpenTelemetry

- ベンダー非依存を目指してるのにライブラリはベンダーに依存してませんか
- ほとんど飾りかな

eBPF でなんかできそう

一部の APM サービスではログのフィルタリングとかグルーピングとか検索の機能も多機能になってうれしい。ドラッグアンドドロップとかコピペとかで操作しやすい。


| サービス | | | | |
|---|---|---|---|---|
| New Relic | Metrics, Logs, Traces | |  Standard: 99$/user/month  Pro: 349$/user/month ??? ||
| DataDog | Metrics, Logs, Traces | | 19$/user/month ~ ||
| Sentry | Logs, Traces | フロント session replay | Team: $26/month 90日保持する, Business: $80/month |エラー監視に超特化 情報が多い バックエンドはメトリクス・プロファイルが取れない |
| Grafana | Metrics, Logs, Traces | | free / 19$/user/month | 大変, フロントエンドはまだまだ未成熟 |
| hyperDX | | $20/monthで50GB、30日保持する |
| AWS Azure マネージドな監視サービス | |

NextJS に OpenTelemetry のサポートがあるのでそれを使う

[可観測可能な「Next.js」アプリケーションを作る⸺UXからパフォーマンスまでWebアプリケーション全ての改善に役立つ方法を⸺](https://cloudnativedays.jp/cndt2023/talks/2043)
[【OpenTelemetry】オブザーバビリティバックエンド8種食べ比べ](https://zenn.dev/sumiren/articles/dfe5219a272cd3)
![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/25857247-29f1-4e91-8fe7-22498fb55b17/b489848a-d449-4830-8fa6-18d36ffa91cf/image.png)
![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/25857247-29f1-4e91-8fe7-22498fb55b17/7db6319e-4198-49b6-8a88-6d5c67bfb227/image.png)

## 構成

Grafana には次のようなサービスがある。

| サービス | 説明 |
| --- | --- |
| Grafana | あらゆる情報をダッシュボードで管理する |
| Grafana Loki | あらゆるアプリケーションからのログを収集、検索 |
| Grafana Pyroscope | プロファイルを収集 |
| Grafana Alloy | 他のサービスから情報を受け取り、効率化して Grafana にデータを転送する |
| Grafana Alert | Slack とかにアラートしてくれる |

今回 Grafana Cloud を用いるので、Grafana、Loki、Pyroscope を Grafana Cloud で Alloy をローカルで構築する。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/25857247-29f1-4e91-8fe7-22498fb55b17/4ca2fc96-5037-403d-bdd8-84534436346b/image.png)

手順

1. Grafana Cloud のアカウントを作成
2. アプリケーション側でログやプロファイルを出力するようにする
3. Alloy を立ててテレメトリデータを集める
4. Grafana Cloud の OpenTelemetry Collector で集める
5. ダッシュボードで表示する

## 構築

### メトリクスの出力

Grafana に届ける情報列であるメトリクスはアプリケーション側で SDK を用いて実装し、出力する。OpenTelemetry の SDK を入れて実装する。

1. go get でフレームワークに対応したモジュールを取得する
2. 初期設定をする
3. net/http Gin Echo などにミドルウェアを挿入してトレースを開始する

詳細はドキュメントを参照してください。

[Getting Started](https://opentelemetry.io/docs/languages/go/getting-started/)

Pyroscope の SDK は pprof を使う pyroscope-go にした。

- https://github.com/grafana/pyroscope-go
- https://github.com/grafana/otel-profiling-go

Alloy に通すためには Go だと Pull モードで動かすのですがそれを行うライブラリがアーカイブされてたのとそこまで Alloy で出来ることがないのでやめて直接 Grafana Cloud に送っている。

### Grafana Alloy で集める

Docker イメージとして配布されているのでそれを起動

```yaml
services:
  alloy:
    image: grafana/alloy:v1.4.2
    ports:
      - 12345:12345 
    volumes:
      - ./config.alloy:/etc/alloy/config.alloy
    command: [
      "run",
      "--server.http.listen-addr=0.0.0.0:12345",
      "--storage.path=/var/lib/alloy/data",
      "--stability.level=experimental",
      "/etc/alloy/config.alloy",
    ]
```

設定ファイル `alloy.config` を書いていく。

例として alloy の 0.0.0.0:4318 から入ったテレメトリを Grafana Cloud に送るだけの設定

```json
logging {
  level  = "info"
  format = "logfmt"
}

livedebugging {
  enabled = true
}

otelcol.receiver.otlp "basic" {
  http { }

  output {
    metrics = [otelcol.processor.batch.basic.input]
    logs    = [otelcol.processor.batch.basic.input]
    traces  = [otelcol.processor.batch.basic.input]
  }
}

otelcol.processor.batch "basic" {
  output {
    metrics = [otelcol.exporter.otlphttp.default.input]
    logs    = [otelcol.exporter.otlphttp.default.input]
    traces  = [otelcol.exporter.otlphttp.default.input]
  }
}

otelcol.auth.basic "default" {
  username = "<YOUR_GRAFANA_CLOUD_INSTANCE ID>"
  password = "<ACCESS_POLICY_TOKEN>"
}

otelcol.exporter.otlphttp "default" {
  client {
    endpoint = "<OTLP_ENDPOINT_URL>"
    auth     = otelcol.auth.basic.default.handler
  }
}
```

[localhost:12345](http://localhost:12345)/graph からこの構成図が見られる。

今回の最終的な構成は入ってきたテレメトリデータについてメトリクスとトレースは Grafana Cloud の OpenTelmetry Collector へ、ログは loki へ回しています。構成図はこんな感じ。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/25857247-29f1-4e91-8fe7-22498fb55b17/6d09ffa4-6fe8-4ef1-8293-2e4a0ccf907d/image.png)

すべてのコンポーネントの大枠としては 3 段階あり、受け取って処理して次に渡すという感じです。

```json
receiver -> batch -> exporter
```

### ダッシュボードの設定

https://zenn.dev/taisho6339/articles/0654040691aaab
https://prometheus.io/docs/prometheus/latest/querying/basics/


ここら辺は大量の知見が必要なので他の良い記事を参考にするのがいい。ただそんな記事は存在しないのでリファレンスとか色々見ながら頑張る。

Grafana に来た OpenTelemetry Protocol は Prometheus のメトリクスに変換される。Grafana ではそのメトリクスを用いた PromQL というクエリを用いてグラフのクエリを書く。

```jsx
request_duration_seconds_count
```

これをメトリックと言う。

| メトリック | 説明 |
| --- | --- |
| request_duration_seconds_count | 直近 1 秒に測定されたリクエスト総数 |

```
sum(increase(request_duration_seconds_count[1h]))
sum(rate(http_requests_total[1h]))
sum by (method) (increase(request_duration_seconds_count[1h]))
sum by (handler, method) (rate(http_requests_total[1m]))
```

直近 1 時間の

| 関数 | 説明 |
| --- | --- |
| sum(...) | 累計 |
| topk(10, ...) | 大きい値を持つ上位 10 件の要素を返す |
| rate(...) |  |
| increase(...) |  |

status code 毎にメトリクスがあるので `sum by (...) (...)` でグルーピングします。

```
sum(increase(request_duration_seconds_count{code=~"[45].."}[1h])) / sum(increase(request_duration_seconds_count[1h]))
```

条件