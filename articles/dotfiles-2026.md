---
title: "Nix × LLM で加速的に成長し続ける dotfiles"
emoji: "🦔"
type: "tech"
topics: ["nix", "dotfiles", "llm"]
published: false
---

みなさん dotfiles 作ってますか！

エンジニアはキーボード、チェア、ガジェットに加え、便利なツールを日々こだわり抜いて生産性を上げてると思います。

しかし手持ちの PC に加え、大量のサーバーを持っていると扱うマシンは多く、LLM で開発効率が上がった分だけアプリも増えて、それに比例して設定やキーバインドも増えていき、そんなの覚えてらんないし、それぞれのマシンにインストール方法しらべて前の設定みつけて入れるなんて苦行だよ～

...と日々新しいアプリがでているけれど、すっと手が伸びにくくなってるんじゃないでしょうか。私も 500 個ほどのアプリを管理していて辛くなってきました。

そこでこの記事では管理の歴史をたどりながら、LLM × Nix で新しくて便利なものをすぐに使える環境を目指します。

## dotfiles はどう進化してきたか

エンジニアにとって PC は毎日朝から晩まで触るもので、そうすると日々負担に感じているものも至る所にあります。これに便利な設定や便利なツールを入れて作業効率を上げる営みがあります。Linux において `.zshrc` `.vimrc` など `.` から始まるファイルたち dotfiles というのがその設定に当たります。



dotfiles という言葉の由来は Ken Thompson のバグです。ls コマンドで `.` (現在のディレクトリ) と `..` (親のディレクトリ) が毎回表示されて煩わしいので、先頭が `.` なら非表示にしました。その副産物として隠しファイルが使えるようになり、いろんなアプリが設定ファイルに使い始めました。それらをまとめて dotfiles と呼んでいます。

さてそんな dotfiles を管理する方法はだんだんと進化して移り変わってきました。

### 設定を同期したい

設定を育てていて最初に困るのはその育てた設定が他のマシンには入ってなくて不便なままなことです。私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からになってしまいます。

そこで dotfiles というリポジトリにコピペして同期させるようにしました。これで他のマシンにも設定を入れやすいし、ついでに他の人の工夫も参考にできてうれしいです。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。自分でスクリプトを書くと、既存の設定とのコンフリクト、設定が増えるたびの書き足し、Windows のリンクの仕様のトリッキーさ...と考えることが多いです。そこでツールが生まれました。

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

Nix はビルドシステムかつパッケージマネージャーで多機能すぎて紹介しきれません。ただすべての基盤は Nix 言語で環境を宣言的に記述し、Flakes という仕組みで依存するパッケージのバージョンを `flake.lock` に固定して再現性を保つ仕組みです。

たとえば Flakes のエントリーポイントとなる `flake.nix` はこんな感じです。

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

といった恩恵があります。ただしデメリットもあり、

1. 学習コストが高い
  独自の言語で読みにくいエラーメッセージな上に日本語情報が少ない。
2. ストレージ消費が多い
  依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数あったり、ロールバック用の古い世代が残ってディスクが圧迫します。そのため定期的なガベージコレクションが必要となります。
3. Windows に対応していない
  Unix のシステムに強く依存してる為に Linux macOS は対応できますが Windows とは相性が悪い。

このようなデメリットが強いため、Nix はあんまり使われていない気がしています。逆に言えばこれらを解決できればたくさん使ってもらえそうです。


### LLM で管理コストを下げたい

1. 記事を読んだり他の dotfiles を読んで、よさそうな設定を見つける
2. 自分の環境にどう入れるか、自分の哲学と合っているかを考える
3. 設定ファイルを編集する
4. 適用して起動してみる
5. 壊れたら直す
6. 他のツールも 2~5 を行い、比較検討する
7. 他のマシンにも反映する

特に試行錯誤の段階が重くて、環境の切り替えに 10 分かかったり壊すリスクを考えるとさまざまなツールの中からベストを選び続けるには相当な時間と労力が必要です。

そこで LLM を使うわけですが、ここで Nix が 2 度おいしいんです。マシン全体の環境が一箇所に宣言的に書かれているからこそ、LLM が全体を見通して最適な提案ができます。そうすると

1. LLM で大量の記事をリサーチさせて改善点を洗い出す
2. 自分の判断基準をもとに採否を決めて適用する
3. 壊れてもロールバックで即座に戻せる
4. 全マシンに適用

さらに「どんな環境にしたいか」という自分の哲学を文章にして渡せば、判断の部分まで自動化できます。バグの修正、日々のアップデート、改善点の発見といったフィードバックループを LLM に回してもらい、修正からデプロイまでを一気通貫で行うのが理想の姿です。


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

`config.nix` はこれら 3 つを束ねる司令塔で、ホストやユーザーごとにどのモジュールを有効にするかをここで切り替えます。

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

この構造は LLM との相性も良く、「fish の設定を変えて」と言えば `shell/fish.nix` だけを、「テーマを変えて」と言えば `theme/` だけを触ればよいと伝えられます。影響範囲が明確なので、LLM が余計なファイルに触れるリスクが減ります。

### LLM に改善案を出してもらう

Nix ですべてを宣言的に書いているので、環境全体を LLM に渡して「自分の環境にとって最善の選択は何か」を問えます。具体的には `AGENTS.md` に「URL を渡されたらどう分析するか」を書いています。生産性向上に関する記事や他の dotfiles で良いと思ったものを貼ると、LLM がそれをよく調べて改善案を提示してくれます。

```text
改善点まとめ（48記事から抽出）

High Priority

1. nixd に home-manager options 補完を追加
現状 nixpkgs.expr しか設定してない。options を追加すると .nix ファイル編集時に home-manager
のオプション補完・定義ジャンプが効くようになる。
# lsp/servers.nix の nixd.settings.nixd に追加
options.home-manager.expr = ''
  (builtins.getFlake "/home/anko/dotfiles").homeConfigurations."wsl".options
'';
出典: momeemt/dotfiles2025

2. checks flake output の追加
今は pre-commit.check.enable = false で、CI も nix build を直接叩いてる。checks を定義すれば nix flake check
一発で全構成を検証できる。--option abort-on-warn true で deprecation warning も検出可能。
出典: soracat/nix-flake-check

3. nix-output-monitor を devShell に追加
ビルド出力が見やすくなる。switch スクリプトにも | nom をパイプできる。
出典: nazozokc/dotfiles

4. gh extension の宣言的管理
programs.gh.extensions で gh-dash 等を mkDerivation + fetchurl で管理できる。今は extensions が空。
出典: atrae/gh-aw

Medium Priority
...
```

さらにツールの選定基準を `docs/` で言語化して LLM に読ませることで、意図を汲み取った提案をしてくれるようになります。こうした言語化を積み重ねることで LLM の提案精度が上がり、すべて任せて放置しているだけで環境が改善していくようにしていきます。


### 既存の環境を壊さず段階的に採用できる

他人の dotfiles を試すときの一番の心理的ハードルは「今の環境が壊れるかもしれない」という不安です。勝手にユーザー名が書き換わったり、バックアップがなされずに環境が壊されてしまうみたいな悲劇が起こらない仕組みづくりが重要です。

まず setup スクリプトは用意していません。他人のスクリプトをそのまま実行するのはセキュリティ面でも環境破壊のリスク面でも抵抗が大きいので、自分で Nix をインストールしてこれを叩いてセットアップできるようになっています。

```sh
nix run github:anko9801/dotfiles#switch
```

適用時の安全網として `home-manager.backupFileExtension = "backup"` を設定しています。既存の `.zshrc` があれば `.zshrc.backup` に自動でリネームしてから新しい設定が適用されるので、脳死で実行して大丈夫です。何か問題があれば `.backup` ファイルから即座に戻せます。

移行も段階的に進められます。fork するユーザーは `config.nix` のユーザー情報とホスト構成を書き換えるだけで最低限の環境が立ち上がり、既存の `.zshrc` や `.vimrc` はそのまま残ります。慣れてきたところで `./shell/zsh` を追加するように、モジュール単位で少しずつ乗り換えられるように設計しています。


### どの OS でもバグなく同じ環境が再現される

Nix に乗っているのでほとんどのアプリはそのまま動きます。さらに GitHub Actions でビルド・適用・ベンチマークを回して、常に動くことを保証しています。

問題は Windows で、Nix はネイティブサポートしていません。パッケージ定義がほぼすべて Linux/macOS 向けに書かれており、Windows のファイルシステムやシステムコールとそもそも相性が悪いためです。そこでパッケージマネージャ WinGet で宣言的に管理し、WSL 上でビルドしてスクリプトで配置するという構成で適用します。


### 機密情報を外に逃す

AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...と不安になることありませんか。根本的な解決として外部サービスにすべて逃がすのが得策です。

私はすべての機密情報を 1Password に置いていて、SSH や Git で秘密鍵が必要になったら Touch ID や Windows Hello などの生体認証で呼び出しています。API キーの環境変数は op コマンドで起動時に展開するようにしてます。漏洩防止として pre-commit に gitleaks を入れています。

パスワードマネージャーは E2EE でサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全です。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので SOC 2 認証を受けたサービスを選びましょう。他には Bitwarden Vaultwarden などがあります。

## 移行方法

僕のおすすめとしては理解が深まるので自分で Nix を書くことなのですが、考えるべきことが多いので以下のリポジトリを用いて設定する方法もお伝えします。

私の徹底的な認知負荷の削減により

- 重複するコマンドやプラグインの削減
- キーバインドは「目的語 + 動詞」で統一する
- 不採用にした理由も残しておくことで、似たツールを再提案されなくなります。

https://github.com/anko9801/dotfiles

3 ステップです。

1. リポジトリを fork して clone する
2. `config.nix` にユーザー情報とホストに入れる環境を最小限に選択する
3. Nix を入れて `nix run .#switch -- <ホスト名>` と適用する

これで既存の環境にツールが揃った状態で立ち上がります。

そこから自分に合わない部分を削ったり、足りないものを追加したりすればいいです。`config.nix` の `hosts` でマシンごとのモジュール構成を変えられるし、GUI の有無も切り替えられます。また既存の環境を取り込みたい場合は nix-migrate というスキルを叩いて LLM に Nix へ移行してもらうこともできます。

## まとめ

バグから生まれた dotfiles は、最初はただの設定ファイルの寄せ集めでしたが、宣言的に管理することでマシン環境の記述になり、今では LLM がフィードバックループを自動で回しています。

dotfiles を作ったことない人もぜひ作ってみてください。触っているうちに、なぜこのツールを使ってるのかがだんだん見えてきます。それを言語化して、また削って、また足す。そのうち自分の哲学が輪郭を持ち始めます。LLM が設定の管理を引き受けてくれるようになった今、本質的な「どんな環境にすべきか」を考えることが楽しくなってきます。その哲学をみんなで共有できたらいいなと思います。
