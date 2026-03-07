---
title: "Nix × LLM で作る、使うほど賢くなる作業環境"
emoji: "🦔"
type: "tech"
topics: ["nix", "dotfiles", "llm"]
published: false
---

LLM によってコードを書いたり、設計を考えたり、調べものをしたりと、あらゆる作業が速くなりました。なのになぜ、作業環境の整備は相変わらず自分でやっているのでしょうか。

先日、複数あるメールクライアントの一本化に LLM を使って GUI 操作を任せてみました。すると、スクリーンショットを撮って判断してクリックするという並列化しにくいフローになっており、自分でやれば 5 分程度で終わるのに 20 分程度掛かりました。このボトルネックは LLM モデルの問題ではなく OS を含めた設計の問題で、そう簡単には解消されそうにありません。

こうした手間は、PC を買い替えるたびに、転職するたびに、サーバーを立てるたびに、何度もやり直すことになります。その上、開発効率が上がり世の中にアプリが増えてきて管理しきれず、cmux や codex アプリなど日々新しいアプリが出ているのに、すっと手が伸びにくくなってるんじゃないでしょうか。

そこで今回は、エンジニアが作業環境を管理するために使ってきた dotfiles に環境を宣言的に記述し、LLM のコンテキスト理解とのシナジーで賢く管理する方法を紹介します。

## dotfiles はどう進化してきたか

私自身、dotfiles を長いこと管理してきて、シェルスクリプトで管理し始め、chezmoi を試し、Ansible に冪等性を教えてもらい、yadm に移行し、 Nix に落ち着きました。dotfiles の管理方法はこれだけ選択肢があって、どれも一長一短あります。

これらを体系的に解説します。

### dotfiles 

dotfiles という言葉の由来は Ken Thompson が書いたバグです。ls コマンドで `.` (現在のディレクトリ) と `..` (親のディレクトリ) が毎回表示されて煩わしいので、先頭が `.` なら非表示にしました。その副産物として隠しファイルが使えるようになり、いろんなアプリが設定ファイルに使い始めました。それらをまとめて dotfiles と呼んでいます。

### 設定を同期したい

設定を育てていて最初に困るのはその育てた設定が他のマシンには入ってなくて不便なままなことです。私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からになってしまいます。

そこで dotfiles というリポジトリにコピペして同期させるようにしました。これで他のマシンにも設定を入れやすいし、ついでに他の人の工夫も参考にできてうれしいです。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。それに自分でスクリプトを書くと、既存の設定とのコンフリクト、設定が増えるたびの書き足し、Windows のリンクの仕様のトリッキーさ...と考えることが多いです。そこでたくさんのツールが生まれました。

- シンボリックリンク方式 ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
- コピー方式 ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
- 直接 Git 管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデメありますが、怠惰な僕にとっては yadm が好みです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすい。テンプレート機能、シークレット管理もしてくれます。


### 環境そのものを再現したい

設定が適用できたからといって、まっさらな状態に設定だけいれても動きません。インストールしないと動きません。でも新しいマシンで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いです。

それじゃインストールも自動化しちゃおう！ってことでよくこんなツールを使います。

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

しかしこれだと、動的ライブラリやランタイムのバージョン・パスの違いという暗黙的な依存関係で動かないことがよくあります。暗黙的な依存関係を明示し、そのまま持ち運ぶことでどこでも環境が再現できると言ったのが Nix です。

https://github.com/NixOS/nix

Nix はビルドシステムかつパッケージマネージャーです。Nix は全部紹介しようとすると多機能すぎて紹介しきれません。ただすべての基盤は Nix 言語で環境を宣言的に記述し、Flakes という仕組みで依存するパッケージのバージョンを `flake.lock` に固定して再現性を保つ仕組みです。

たとえば Flakes のエントリーポイントとなる `flake.nix` はこんな感じです。これをもとにバージョンが固定された `flake.lock` が生成されます。

```nix:flake.nix
{
  description = "dotfiles";
  # 依存する外部 flake。バージョンは flake.lock に自動で固定される
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  # inputs を用いて外部に公開するもの
  outputs = { home-manager, nixpkgs, ... }:
  let
    system = "aarch64-darwin";
    pkgs = import nixpkgs { inherit system; };
  in
  {
    # `home-manager switch --flake .#default` を実行すると設定が適用される
    homeConfigurations.default =
      home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home/home.nix ];
      };
  };
}
```

このように複数の Flakes に依存して新たな Flake を作るように作っていきます。この仕組みの上に、さまざまなエコシステムが構築されています。

| 機能 | 説明 |
|---|---|
| nixpkgs | パッケージのビルドレシピ集。12万以上のパッケージが揃っている |
| Home Manager | PATH へのツール追加や `~/.zshrc` などの設定ファイル配置を管理する |
| NixOS | カーネル・サービス・ユーザーまで OS 全体を宣言的に管理する |
| devShells | プロジェクトごとの開発環境を Nix で定義する |
| nixos-anywhere | リモートマシンに SSH で NixOS を自動でセットアップする |
| agenix / sops-nix | API キーやパスワードなどを暗号化して Git で安全に管理する |

他にも便利なエコシステムはたくさんありますが、今回は nixpkgs と Home Manager を使って dotfiles を管理していきます。

これらを駆使してすべてのツールを Nix で宣言的に管理すると dotfiles は単なる設定集を超えて、マシン環境そのものの記述になります。これにより、

- 環境のスナップショットがとれて、以前の状態にいつでも戻せる
- 具体的な操作から離れてどんな環境にしたいかだけを考えればよくなる
- 壊れたり紛失したり水没しても、新しいマシンで同じ環境をすぐに再現できる
- バックアップがほとんど不要になる

といった恩恵があります。ただしデメリットもあり、

1. 学習コストが高い
  独自の言語で読みにくいエラーメッセージな上に日本語情報が少ない。
2. ストレージ消費が多い
  依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数あったり、ロールバック用の古い世代が残ってディスクが圧迫します。そのため定期的なガベージコレクションが必要となります。
3. Windows に対応していない
  Unix のシステムに強く依存してる為に Linux macOS は対応できますが Windows とは相性が悪い。

このようなデメリットが強いため、Nix はあんまり使われていない気がしています。逆に言えばこれらを解決できればたくさん使ってもらえそうです。


### LLM で管理コストを下げたい

Nix は便利ですが dotfiles を育て続けるのは管理がめんどくさいです。

1. 記事を読んだり他の dotfiles を読んで、よさそうな設定を見つける
2. 自分の環境にどう入れるか、自分の哲学と合っているかを考える
3. 設定ファイルを編集する
4. Nix に翻訳して書く
5. 適用して起動してみる
6. 壊れたら直す
7. 他のツールも 2~6 を行い、比較検討する
8. 他のマシンにも反映する

特に試行錯誤の段階が重くて、環境の切り替えに 10 分かかったり壊すリスクを考えるとさまざまなツールの中からベストを選び続けるには相当な時間と労力が必要です。

そこで LLM を使うわけですが、ここで Nix が 2 度おいしいんです。マシン全体の環境が一箇所に宣言的に書かれているからこそ、LLM が全体を見通して最適な提案ができます。そうすると

1. LLM で大量の記事をリサーチさせて改善点を洗い出す
2. 自分の判断基準をもとに採否を決めて適用する
3. 壊れてもロールバックで即座に戻せる
4. 全マシンに適用

さらに「どんな環境にしたいか」という自分の哲学を文章にして渡せば、LLM の提案が意図に近づき、採否の判断コストも下がっていきます。週次で改善提案を通知する仕組みを組んでおけば、あとは流れてくる提案に「採用 / 却下」を返すだけで環境が育ち続けます。そのようなバグの修正、日々のアップデート、改善点の発見してデプロイまでを一気通貫で回す。


## じゃあどう設計するか

Nix × LLM には、設定をマシンや人をまたいで共有でき、誰がいつどこで実行しても同じ環境が立ち上がり、依存が明示的で管理コストが低いというメリットがあります。一方でストレージ消費の多さと Windows 非対応という課題も残ります。

目指すのは、これらの課題を解消した上でどの OS のどのユーザーでもバグなく同じ環境が再現され、マシン間で同期し、他人の良い設定は流し込むだけで取り込め、すべての設定に理由が残っていて見返しても迷わず、既存の環境を壊さず段階的に採用でき、機密情報はディスクに一切残さず、情報を与えるほど生産性が上がり続ける dotfiles です。


### 管理しやすい構造にする

設定が増えてくると、ツール間の依存関係が複雑になり LLM でも全体を把握しきれなくなります。これを防ぐために、責務をレイヤーで切り分けて管理しています。

```text
dotfiles/
├── flake.nix               # エントリーポイント
├── config.nix              # ユーザー・ホスト・モジュールの全定義
├── AGENTS.md               # LLM への作業手順
├── docs/                   # 判断基準と意思決定の記録
│   └── tool-selection.md
├── system/                 # OS ごとのシステム設定・ビルダー関数群
│   ├── darwin/
│   ├── nixos/
│   └── windows/
├── theme/                  # Stylix (全ツールの色・フォント・カーソル)
├── shell/                  # Zsh, Fish, Starship, ...
│   ├── zsh/
│   ├── fish.nix
│   ├── starship.nix
│   └── ...
├── editor/                 # Neovim, Vim, VSCode, Helix, Zed
├── terminal/               # Ghostty, WezTerm, Zellij, tmux
├── tools/                  # Git, Yazi, Bat, ...
├── dev/                    # Rust, Go, Python, Node, mise
├── security/               # 1Password, SSH, GPG, gitleaks
├── ai/                     # Claude Code, Aider
└── desktop/                # Wayland, IME
```

構造は 3 つの関心事に分かれていて、`config.nix` がそれらを束ねています。

1. **ツール** (`shell/`、`editor/`、`terminal/` など)
   カテゴリごとにディレクトリを分け、各 `.nix` ファイルが 1 つのアプリの責務だけを持ちます。OS 差分やツール間の整合性はそれぞれのファイル内で条件分岐して吸収するので、たとえば Fish の設定を変えたければ `shell/fish.nix` だけを見ればよいです。
2. **システム** (`system/`)
   Home Manager では管理できない管理者権限が必要な設定や、特定のツールに依存しない OS 差分を扱います。
3. **テーマ** (`theme/`)
   カラースキーム・フォント・カーソルなどを一元管理します。Stylix がここの定義を読んで、ターミナルやエディターなど全ツールの色合いを自動で揃えてくれます。

`config.nix` は 3 つを束ねる司令塔として、ホストやユーザーごとにどのモジュールを有効にするかをここで切り替えます。

```nix:config.nix
users = {
  your-name = {
    editor = "nvim";
    git = {
      name = "your-name";
      email = "your-email@example.com";
      sshKey = "ssh-ed25519 AAAA...";
    };
  };
};

hosts = {
  linux-wsl = {
    system = "x86_64-linux";
    manager = "home-manager";
    modules = moduleSets.workstation ++ [ ./terminal/zellij ];
  };
  mac-arm = {
    system = "aarch64-darwin";
    manager = "nix-darwin";
    modules = moduleSets.workstation;
    systemModules = [ ./system/darwin/desktop.nix ];
  };
  nixos-desktop = {
    system = "x86_64-linux";
    manager = "nixos";
    modules = moduleSets.workstation ++ [ ./desktop ];
    systemModules = [ ./system/nixos/desktop.nix ];
  };
  # ...
};
```

この構造は LLM との相性も良く、`AGENTS.md` にどのディレクトリが何に対応するのか書けばどのファイルを変更すればよいのかわかってくれますし、影響範囲が明確なので、LLM が余計なファイルに触れるリスクが減ります。


### LLM に改善案を出してもらう

Nix ですべてを宣言的に書いているので、環境全体を LLM に渡して「自分の環境にとって最善の選択は何か」を問えます。具体的には `AGENTS.md` に「URL を渡されたらどう分析するか」を書いています。生産性向上に関する記事や他の dotfiles で良いと思ったものを貼ると、LLM がそれをよく調べて改善案を提示してくれます。

```text
高インパクト改善案

1. nh — nixos-rebuild/home-manager switchの上位互換

viperML作、nix-community公式

現状の nix run .#switch を置き換える。差分をカラー表示し、ビルド進捗を nix-output-monitor で可視化する。

nh os switch .        # nixos-rebuild switch --flake .
nh home switch .      # home-manager switch --flake .
nh search ripgrep     # nix search の高速版
nh clean all --keep 5 # 世代管理

あなたの哲学との合致: 速さ（Rust製）、認知負荷削減（1コマンドで全操作）、安全（差分プレビュー付き）

2. Neovimをflake outputとしてエクスポート

Mic92パターン

任意のマシンで nix run github:anko/dotfiles#nvim すれば自分のNeovim環境が即座に使える。サーバーにsshした時もホストを汚さず自分の環境を持ち込める。

# flake.nix outputs に追加
packages.${system}.nvim = config.programs.nixvim.build;

あなたの哲学との合致: 環境統一（どこでも同じエディタ）、復元可能（nix store内で完結）

3. Self-gating modules

wimpysworld パターン (19ホスト管理者)

現状: config.nix で各ホストのモジュールリストを手動管理
# 今
modules = moduleSets.workstation ++ [ ./system/darwin/desktop.nix ];

改善: 全モジュールを全ホストにimportし、モジュール自身が条件判定
# モジュール内部
{ config, lib, ... }:
lib.mkIf config.myHost.isWorkstation {
  programs.wezterm.enable = true;
}

あなたの哲学との合致: 認知負荷削減（「どのホストにどのモジュール」を覚えなくていい）。ただし現状の moduleSets パターンも十分シンプルなので、ホスト数が増えた時に検討。
...
```

さらにツールの選定基準を `docs/` で言語化して LLM に読ませることで、意図を汲み取った提案をしてくれるようになります。こうした言語化を積み重ねることで LLM の提案精度が上がり、すべて任せて放置しているだけで環境が改善していくようにしていきます。


### 既存の環境を壊さず段階的に採用できる

他人の dotfiles を試すときの一番の心理的ハードルは「今の環境が壊れるかもしれない」という不安です。勝手にユーザー名が書き換わったり、バックアップがなされずに環境が壊されてしまうみたいな悲劇が起こらない仕組みづくりが重要です。

他人のスクリプトをそのまま実行するのはセキュリティ面でも環境破壊のリスク面でも抵抗が大きいので、テンプレートから始められるようにしました。

```shell
# 1. Nix をインストール
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install

# 2. テンプレートから初期化
mkdir ~/dotfiles && cd ~/dotfiles
nix flake init -t github:anko9801/dotfiles

# 3. config.nix を編集（$USER と一致させる）
# users.your-username の部分を書き換え

# 4. 適用
nix run .#switch
```

これだけで bash, starship, git, vim が宣言的に管理される状態になります。既存の `.bashrc` や `.gitconfig` は `.bashrc.backup` のようにリネームされるだけなので、元の環境が壊れることはなく、戻したくなったら backup を元に戻せばいいです。


### どの OS でもバグなく同じ環境が再現される

Nix は Windows をネイティブサポートしていません。パッケージ定義がほぼすべて Linux/macOS 向けに書かれており、Windows のファイルシステムやシステムコールとそもそも相性が悪いためです。

そこで Windows のパッケージマネージャ WinGet で宣言的にアプリを管理し、設定の方は WSL 上でビルドしたものを配置することで、Nix の機能を活用しつつ 1 アプリ = 1 ファイルを実現しています。

これですべての OS に対応できて、その上で GitHub Actions でビルド・適用・ベンチマークを回して、動くことを保証しています。


### 機密情報を外に逃す

AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...と不安になることありませんか。根本的な解決として外部サービスにすべて逃がすのが得策です。

私はすべての機密情報を 1Password というパスワードマネージャーに置いています。他には Bitwarden Vaultwarden などがありますね。SSH や Git で秘密鍵が必要になったら Touch ID や Windows Hello などの生体認証で呼び出しており、API キーの環境変数は op コマンドで起動時に展開するようにして、漏洩防止として pre-commit に gitleaks を入れています。

パスワードマネージャーは E2EE でサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全です。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので SOC 2 認証を受けたサービスを選びましょう。


## 始め方

このような設計を一瞬で始められるようにテンプレートを用意させていただきました。

### ステップ 1: テンプレートを取得

Nix をインストールしてテンプレートを入れます。

```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install

mkdir ~/dotfiles && cd ~/dotfiles
nix flake init -t github:anko9801/dotfiles
```

そうするとこのような構造が生成されます。

```text
template/
├── AGENTS.md                      # LLM agent guide
├── README.md                      # Quick start guide
├── flake.nix                      # Inputs, apps (switch/windows), checks
├── config.nix                     # Users, hosts, modules
├── docs/tool-selection.md         # Tool decision template
├── editor/vim.nix                 # Vim/Neovim config
├── shell/
│   ├── bash.nix                   # Bash config (coreModule)
│   └── starship.nix               # Cross-shell prompt
├── system/
│   ├── common.nix                 # Platform detection, defaults, HM bootstrap
│   ├── hosts.nix                  # Host builder, flake module resolution
│   └── windows/
│       ├── setup.sh               # WSL → Windows deployment
│       └── winget-packages.json   # Starter packages
├── theme/default.nix              # Stylix (Catppuccin Mocha)
└── tools/git.nix                  # Git config
```

### ステップ 2: ユーザー情報を記入

先ほど生成された `config.nix` を開いて `your-username` を自分のユーザー名 `$USER` に合わせてもろもろを設定します。

```nix:config.nix
users = {
  your-username = {
    userName = "Your Name";
    userEmail = "you@example.com";
  };
};
```

この情報は `system/common.nix` の `defaults.identity` を経由して `tools/git.nix` など各モジュールに自動で伝わります。

`config.nix` のホスト部分にはどのツールを入れるかも書かれてあるので適宜調整してください。


### ステップ 3: 適用

そして設定したものから自分の環境に適用します。

```bash
nix run .#switch
```

これで bash, starship, git, vim のツールと設定が入ります。

### ステップ 4: モジュールを追加して拡張

ここからより育てる為に例えば `tmux` を追加したい場合は、まず適切な場所にファイルを置きます。

```nix:tools/tmux.nix
_:
{
  programs.tmux = {
    enable = true;
    terminal = "tmux-256color";
    escapeTime = 0;
  };
}
```

そして `config.nix` の `baseModules` にパスを追加して `nix run .#switch` すれば完了です。

```nix:config.nix
baseModules = [
  ./shell/starship.nix
  ./tools/git.nix
  ./tools/tmux.nix     # 追加
  ./editor/vim.nix
];
```

このように「ファイルを作る → `config.nix` に登録 → switch」のサイクルで環境が改善していきます。これを LLM に任せることで高速に環境が改善していきます。


## 参考実装

参考用に私の dotfiles もおいておきます。私のデッキは認知負荷の削減です。考えることを減らすほど、仕事やアイデアや環境改善に割けるリソースが増え、改善のループが速く回るようになります。

- 1 ツール 1 役割を徹底してコマンドやプラグインを可能な限り削る
- キーバインドは「目的語 + 動詞」の文法で統一し、人間工学に則った位置に置く
- フロー状態が途切れないために速いツールを選ぶが、体感の差がでないほどの過度な最適化はしない
- どのマシンも同じように使えて、なにを触っているかを考えなくていい
- 安全に操作できるようにして、何かを壊すのを恐れなくていい
- 毎日自分のワークフローに省略・自動化できる部分がないか見直して削る

この考えをもとにツールの選定や設定を行っています。

https://github.com/anko9801/dotfiles

## まとめ

バグから生まれた dotfiles は、最初はただの設定ファイルの寄せ集めでしたが、宣言的に管理することでマシン環境の記述になり、今では LLM がフィードバックループを自動で回しています。

dotfiles を作ったことない人もぜひ作ってみてください。触っているうちに、なぜこのツールを使ってるのかがだんだん見えてきます。それを言語化して管理を任せていく。そうすると環境の本質的な哲学を考えられるようになって楽しくなってきます。その哲学をみんなで共有できたらいいなと思います。
