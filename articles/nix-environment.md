---
title: "Nix × LLM で作る、使うほど賢くなる作業環境"
emoji: "🦔"
type: "tech"
topics: ["nix", "dotfiles", "llm"]
published: false
---

LLM に調べさせて設計させてコードを書かせたりといろんな作業が速くなりましたが、今一番ボトルネックとなるのは手作業の部分です。そんな中で環境整備はほとんどの人が手作業なんじゃないでしょうか？

環境整備って最初に環境構築して、新しいツールが出たら入れたり盆栽して、PC が変わればまたそれの繰り返しのことで、人によっていろんな工夫をして楽にしていると思うのですが、近年 LLM によってツールがたくさん開発されて良いツールがあるのに管理コストが高くて手が伸びにくいみたいな状況がよくある or これから来ると思います。

そこで今回は、エンジニアが作業環境を管理するために使ってきた dotfiles に Nix で環境を宣言的に記述し、LLM と組み合わせて効率よく管理する方法を紹介します。

## dotfiles はどう進化してきたか

私自身、dotfiles で長いこと設定を管理してきたのですが、シェルスクリプトで管理し始め、chezmoi を試し、Ansible に冪等性の良さを教えてもらい、yadm に移行し、 Nix に落ち着きました。dotfiles の管理方法はこれだけ選択肢があって、どれも一長一短あります。

dotfiles という言葉は Unix 開発者が意図せず生み出した副産物によるものです。

ls コマンドの開発中、カレントディレクトリ `.` と親ディレクトリ `..` が表示されるのが煩わしいため、先頭がドットのファイルをすべて非表示にする処理が加えられました[^1]。それが意図せずして隠しファイルという仕組みが生まれ、あらゆるアプリが設定ファイルをドットから始めるようになり、それらをまとめて dotfiles と呼んでいます。

エンジニアにとってそのようなアプリは毎日朝から晩まで使う道具であり、少しでも快適にしようと、便利なツールを入れ、設定を調整します。そうやって積み重ねた設定ファイルが溜まっていきます。そうなると他のマシンでも使いたくなってきます。

### 設定を同期したい

そうなると問題は複数のマシン間で設定を同期するかになります。これがないと私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からで不便なままです。

そこでリポジトリで管理して、同期しつつ他人の知恵や工夫も参考にできるようにしました。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。それに自分でスクリプトを書くと、既存の設定とのコンフリクト、設定が増えるたびの書き足し、Windows のリンクの仕様のトリッキーさ...と考えることが多いです。そこでいくつかのツールが生まれました。

- シンボリックリンク方式 ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
- コピー方式 ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
- 直接 Git 管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデメありますが、yadm がもっとも認知負荷が低くて好きです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすいです。

このように設定を管理できましたが、設定が揃っても、まっさらなマシンでは動きません。

### 環境そのものを再現したい

インストールしないと動きません。でも新しいマシンで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いです。

それじゃインストールも自動化しよう！ってことでよくこんなツールを使います。

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

しかしこれだと、動的ライブラリやランタイムのバージョン・パスの違いという暗黙的な依存関係で動かないことがよくあります。暗黙的な依存関係を明示し、そのまま持ち運ぶことでどこでも環境が再現できると言ったのが Nix です。

## Nix とは

Nix はパッケージマネージャーであり、かつビルドシステムです。

根幹となるのは環境を宣言的に記述する Nix 言語と依存パッケージのバージョンを固定をする Flakes という仕組みです。たとえば Flakes のエントリーポイントとなる `flake.nix` はこんな感じです。

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
    homeConfigurations.default =
      home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home/home.nix ];
      };
  };
}
```

このように JSON っぽい感じで設定を書けます。この仕組みの上に、さまざまなエコシステムが構築されています。

| 機能              | 説明                                                              |
| ----------------- | ----------------------------------------------------------------- |
| nixpkgs           | パッケージのビルドレシピ集。12万以上のパッケージが揃っている      |
| Home Manager      | PATH へのツール追加や `~/.zshrc` などの設定ファイル配置を管理する |
| NixOS             | カーネル・サービス・ユーザーまで OS 全体を宣言的に管理する        |
| devShells         | プロジェクトごとの開発環境を Nix で定義する                       |
| nixos-anywhere    | リモートマシンに SSH で NixOS を自動でセットアップする            |
| agenix / sops-nix | API キーやパスワードなどを暗号化して Git で安全に管理する         |

これらを使ってすべてのツールを Nix で宣言的に管理すると、dotfiles は単なる設定集を超えてマシン環境そのものの記述になります。

つまり環境のスナップショットがとれて、壊れたり紛失したり水没して新しいマシンを買っても、いつでも以前の状態に戻せるのでバックアップがほとんど不要になります。また具体的な操作から離れてどんな環境にしたいかだけを考えればよくなります。

これに対してデメリットがいくつかあります。

1. 学習コストが高い
   読みにくいエラーメッセージな上に日本語情報が少ない。
   → LLM で下駄を履く
2. ストレージ消費が多い
   依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数あったり、ロールバック用の古い世代が残ってディスクが圧迫します。そのため定期的なガベージコレクションが必要となります。
   → 毎週自動でガベコレすることで軽減
3. Windows に対応していない
   Unix のシステムに強く依存してる為に Linux macOS は対応できますが Windows とは相性が悪い。
   → nix-windows という WSL から Windows のアプリやレジストリなどを設定する Flake を作りました

### Nix × LLM のシナジー

スクショ撮って考えての繰り返しという並列化しにくいフローになっており、このボトルネックは LLM モデルの問題ではなく OS を含めた設計の問題で、そう簡単には解消されそうにありません。根本にあるのは、環境がテキストで書かれていないことです。LLM が得意なのはテキストを読んでテキストを書くこと。渡せるものがなければ、画面を見るしかない。
Nix はその構造を変えます。環境を宣言的にテキストで書く。そのファイルがそのまま LLM のコンテキストになる。哲学を叩き込むほど提案の精度が上がり、ループが自動で回り始める。
LLM が得意とするのは、文章を読み、文章を書くことです。

環境を LLM に改善してもらうのに必要なのは今の環境が、読み書きできる形で書かれていることです。それも手続き的なものではなくて、状態そのものが記述されていることです。

Nix は、ちょうどそれです。

Nix の宣言的な設定ファイルは、LLM にとって理想的な入力になります。このマシンには何が入っていて、なぜ入っているかについて全体を見通せます。だから提案の質が変わります。文脈を読んで、一気通貫で、答えてくれます。

すると LLM が環境の統一インターフェースになります。話しかければアプリが入り、聞けば来歴を答えてくれる。画面ぽちぽちしなくていいし、設定の書き方も覚えなくていい。さらに自分の哲学を叩き込めば、どんどん精度が上がり、自動化されて、使うほど環境が賢くなっていきます。

ただし、精度に課題があります。

書く精度については、Nix の型システムはまだ成熟途上にあり、LLM が生成したコードの意味的な正しさを自動では保証できません。linter による静的検査で緩和します。万が一適用が失敗してもロールバックできます。

読む精度については、設定が増え依存が絡み合うにつれ、LLM はしだいに文脈を失います。これは次のセクションで述べる設計によって解決します。

## 設計と使用感

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

```
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

これだけで bash, starship, git, vim が宣言的に管理される状態になります。既存の `.bashrc` や `.gitconfig` は `.bashrc.backup` のようにリネームされるだけなので、元の環境が壊れることはなく、戻したくなったら backup を元に戻せばいいです。

### どの OS でもバグなく同じ環境が再現される

Nix は Windows をネイティブサポートしていません。パッケージ定義がほぼすべて Linux/macOS 向けに書かれており、Windows のファイルシステムやシステムコールとそもそも相性が悪いためです。

そこで nix-windows という Flake を作りました。WSL からデプロイすることで Windows のアプリやレジストリなどを管理できます。

これで Linux NixOS macOS Windows すべてに対応できて、その上で GitHub Actions でビルド・適用・ベンチマークを回して、動くことを保証しています。

https://github.com/anko9801/nix-windows

### 機密情報を外に逃す

AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...と不安になることありませんか。根本的な解決として外部サービスにすべて逃がすのが得策です。

私はすべての機密情報を 1Password というパスワードマネージャーに置いています。他には Bitwarden Vaultwarden などがありますね。SSH や Git で秘密鍵が必要になったら Touch ID や Windows Hello などの生体認証で呼び出しており、API キーの環境変数は op コマンドで起動時に展開するようにして、漏洩防止として pre-commit に gitleaks を入れています。

パスワードマネージャーは E2EE でサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全です。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので SOC 2 認証を受けたサービスを選びましょう。

## Nix は難しくない

しかしながら Nix を一から始めるのはハードルが高いです。そこで Nix × LLM を 15 分程度で始められるテンプレートを用意しました。

これ叩くだけでできます。うお～～～

```shell
# Nix をインストール
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install

# テンプレートから初期化
mkdir ~/dotfiles && cd ~/dotfiles
nix flake init -t github:anko9801/dotfiles

# config.nix を編集（$USER と一致させる）
# users.your-username の部分を書き換え

# 適用
nix run .#switch
```

テンプレートを入れてくるとこのような構造が生成されます。

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

ここで生成された `config.nix` を開いて `your-username` を自分のユーザー名 (`$USER`) に合わせてもろもろを設定します。

```nix:config.nix
users = {
  your-username = {
    userName = "Your Name";
    userEmail = "you@example.com";
  };
};
```

ホスト部分、デフォルトで bash, starship, git, vim をセットアップするようにしています。適宜

例えば `tmux` を追加したければ適切な場所に Nix ファイルを書いて

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

`config.nix` の `baseModules` にパスを追加して適用すれば完了です。

```nix:config.nix
baseModules = [
  ./shell/starship.nix
  ./tools/git.nix
  ./tools/tmux.nix     # 追加
  ./editor/vim.nix
];
```

このようにファイルを作って登録するだけで環境が改善していきます。これを LLM に任せることで高速に環境が改善していきます。

## 参考実装

参考用に私の dotfiles もおいておきます。コンセプトは認知負荷の削減です。考えることを減らすほど、仕事やアイデアや環境改善に割けるリソースが増え、改善のループが速く回るようになります。

- 1 ツール 1 役割を徹底してコマンドやプラグインを可能な限り削る
- キーバインドは「目的語 + 動詞」の文法で統一し、人間工学に則った位置に置く
- フロー状態が途切れないために速いツールを選ぶが、体感の差がでないほどの過度な最適化はしない
- どのマシンも同じように使えて、なにを触っているかを考えなくていい
- 安全に操作できるようにして、何かを壊すのを恐れなくていい
- 毎日自分のワークフローに省略・自動化できる部分がないか見直して削る

こんな考えで作った dotfiles です。こんな感じでどこでも適用して 15 分くらい待つだけでカリカリに最適化された自分の環境が手に入るという訳です。

https://github.com/anko9801/dotfiles

## まとめ

バグから生まれた dotfiles は、最初はただの設定ファイルの寄せ集めでしたが、宣言的に管理することでマシン環境の記述になり、今では LLM がフィードバックループを自動で回しています。

dotfiles を作ったことない人もぜひ作ってみてください。触っているうちに、なぜこのツールを使ってるのかがだんだん見えてきます。それを言語化して管理を任せていく。そうすると環境の本質的な哲学を考えられるようになって楽しくなってきます。その哲学をみんなで共有できたらいいなと思います。

[^1]: [Origin of Unix Dot File Names (Rob Pike. 2012)](http://xahlee.info/UnixResource_dir/writ/unix_origin_of_dot_filename.html)
