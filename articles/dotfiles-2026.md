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

...と日々新しいアプリがでているけれど、すっと手が伸びにくくなってるんじゃないでしょうか。

そこでこれまで管理においてどんな問題を解決してきたかをたどり、Nix と LLM で新しくて便利なものをすぐに使える環境を目指していきます。

## dotfiles はどう進化してきたか

エンジニアにとって PC は毎日朝から晩まで触るもので、そうすると日々負担に感じているものも至る所にあります。これを便利な設定や便利なツールを入れることで解消し、作業効率を上げる営みがあります。Linux において `.zshrc` `.vimrc` など `.` から始まるファイルたち dotfiles というのがその設定に当たります。

dotfiles という言葉は UNIX を開発した Ken Thompson が ls コマンドで `.` (現在のディレクトリ) `..` (親のディレクトリ) が毎回表示されていて煩わしかったので先頭が `.` なら表示しないとしたことで、隠しファイルに使えるバグが生まれ、それをいろんなアプリが設定ファイルとして使ったことで、それらをまとめて dotfiles と言っています。

さてそんな dotfiles を管理する方法はだんだんと進化して移り変わってきました。

### 設定を同期したい

設定を育てていて最初に困るのはその育てた設定が他のマシンには入ってなくて不便なままなことです。私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からになってしまいます。

そこで dotfiles というリポジトリにコピペして同期させるようにしました。これで他のマシンにも設定を入れやすいし、ついでに他の人の工夫も参考にできてうれしいです。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。これを自分でスクリプトを書くと「既存の設定とコンフリクトしたとき大変」「設定が増えたらスクリプトも書き足さないといけない」「Windows のリンクの仕様がトリッキー」みたいに考えることが多いのでツールが生まれました。

- シンボリックリンク方式 ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
- コピー方式 ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
- 直接 Git 管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデメありますが、怠惰な僕にとっては yadm が好みです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすい。テンプレート機能、シークレット管理もしてくれます。

### 環境そのものを再現したい

設定が適用できたからといって、まっさらな状態に設定だけいれても動きません。インストールしないと動きません。でも新しいマシンで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いです。

それじゃインストールも自動化しちゃおう！ってことでよくこんなツールを使います。

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

しかしこれだと他のマシンに適用したときに動的ライブラリやランタイムのバージョンやパスの違いなどの暗黙的な依存関係よって動かないときがよくあります。動く環境そのものが欲しいのだからこういった問題は起きてほしくないです。そこで暗黙的な依存関係を明示し、それをそのまま持ってくるようにすることでどこでも環境が再現するでしょと言ったのが Nix です。

https://github.com/NixOS/nix

Nix はビルドシステムかつパッケージマネージャーで多機能すぎて紹介しきれません。ただすべての基盤となる考え方はシンプルで、Nix 言語で環境を宣言的に記述し、Flakes という仕組みで依存するパッケージのバージョンを `flake.lock` に固定して再現性を保ちます。

たとえば Nix 言語はこんな感じで複数の flake から flake を作るように作られています。

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

この仕組みの上に、さまざまなエコシステムが構築されています。

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

といった恩恵があります。ただデメリットもあり、

1. 学習コストが高い
  読みにくいエラーメッセージな上に日本語情報が少ない。
2. ストレージ消費が多い
  依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数あったり、ロールバック用の古い世代が残ってディスクが圧迫します。そのため定期的なガベージコレクションが必要となります。
3. Windows に対応していない
  Unix のシステムに強く依存してる為に Linux macOS は対応できますが Windows とは相性が悪いです。

それに加えて育て続けるコストも侮れません。

### LLM で管理コストを下げたい

dotfiles を育てるには地道な作業が続きます。

1. 記事を読んだり他の dotfiles を読む
2. よさそうな設定を見つける
3. 自分の環境にどう入れるか、自分の哲学と合っているかを考える
4. 設定ファイルを編集する
5. 適用してみる
6. 壊れたら直す
7. 他のマシンにも反映する

特に試行錯誤の段階が重くて、環境の切り替えに 10 分かかったり壊すリスクを考えるとさまざまなツールの中からベストを選び続けるには相当な時間と労力が必要です。

そこで LLM を使うわけですが、ここで Nix が 2 度おいしいんです。マシン全体の環境が一箇所に宣言的に書かれているからこそ、LLM が全体を見通して最適な提案ができます。

1. LLM で大量の記事をリサーチさせて改善点を洗い出す
2. 自分の判断基準をもとに採否を決める
3. 壊れてもロールバックで即座に戻せる
4. 全ホストにリモート適用

さらに「どんな環境にしたいか」という自分の哲学を文章にして渡せば、判断の部分まで自動化できます。バグの修正、日々のアップデート、改善点の発見といったフィードバックループを LLM に回してもらい、修正からデプロイまでを一気通貫で行うのが理想の姿です。

## じゃあどう設計するか

今までをまとめると Nix + LLM でやるとメリットとしては

1. 設定を他のマシンや他の人と共有できる
2. 誰がいつどのマシンで実行しても同じ環境が立ち上がる再現できる
3. 依存が明示的
4. 管理コストが低い

デメリットとしては

1. ストレージ消費が多い
2. Windows に対応していない

これらを解決しつつ、どの OS のどのユーザーでもバグなく同じ環境が再現され、マシン間で同期し、他人の良い設定は流し込むだけで取り込め、すべての設定に理由が残っていて見返しても迷わず、既存の環境を壊さず段階的に採用でき、機密情報はディスクに一切残さず、情報を与えるほど生産性が上がり続ける dotfiles を設計していきます。そしてセットアップは Nix をインストールして `nix run github:anko9801/dotfiles#switch` を叩くだけでいいようにします。


### 管理がらくちん

設定が増えてくると、ツール間の依存関係が複雑になり LLM でも全体を把握しきれなくなります。これを防ぐために、このように責務をレイヤーで切り分けて管理しています。

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

**ツール** (`shell/`、`editor/`、`terminal/` など) は、カテゴリごとにディレクトリを分け、各 `.nix` ファイルが1つのアプリの責務だけを持ちます。OS 差分はそれぞれのファイル内で条件分岐して吸収するので、たとえば Fish の設定を変えたければ `shell/fish.nix` だけを見ればよいです。

**システム** (`system/`) は、Home Manager では管理できない管理者権限が必要な設定や、特定のツールに依存しない OS 差分を扱います。

**テーマ** (`theme/`) は、カラースキーム・フォント・カーソルなどを一元管理します。Stylix がここの定義を読んで、ターミナルやエディターなど全ツールの色合いを自動で揃えてくれます。

`config.nix` はこれら 3 つを束ねる司令塔で、ホストやユーザーごとにどのモジュールを有効にするかをここで切り替えます。

この構造は LLM との相性も良く、「fish の設定を変えて」と言えば `shell/fish.nix` だけを、「テーマを変えて」と言えば `theme/` だけを触ればよいと伝えられます。影響範囲が明確なので、LLM が余計なファイルに触れるリスクが減ります。


### 既存の環境を壊さず段階的に採用できる

移行において安心して実行できる環境が重要で、勝手にユーザー名が書き換わったり、バックアップがなされずに環境が壊されてしまうみたいな悲劇が起こらない仕組みづくりが重要です。

例えば 1 コマンドでセットアップがすべて済む setup スクリプトは便利ですが他の人のスクリプトを実行するのはセキュリティ的な面や環境が破壊されるリスクによる抵抗がとても大きいので書かず、自分で Nix をインストールし、実行してもらうという形をとっています。

また Nix には `home-manager.backupFileExtension = "backup"` という既存ファイルは自動でバックアップされる機能があり、`.zshrc` が既にあれば `.zshrc.backup` にリネームされてから新しい設定が適用されます。なので脳死で適用して大丈夫で、何か問題があれば `.backup` ファイルから戻せます。

まずユーザー情報、全ホストの構成、モジュールの組み合わせがすべて `config.nix` に集約されていて、これを数行編集することで新たなホストの設定が完了します。

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

workstation と server で共通モジュールをまとめ、ホストごとに差分だけ追加します。新しいマシンが増えたら数行書くだけでよくて、fork するユーザーはこのファイルのユーザー情報とホスト構成を書き換えるだけで、自分の環境が立ち上がります。

こんな感じでユーザー情報と基本ツールだけ書き換えれば段階的に有効化できます。これなら既存の `.zshrc` や `.vimrc` はそのまま残ります。慣れてきたら `./shell/zsh` を追加して移行していけばいいです。モジュール単位で段階的に移行できるのは Nix の強みです。

### どの OS のどのユーザーでもバグなく同じ環境が再現される

Nix に乗っかっているのでほとんどのアプリはちゃんと動きます。それに加えて GitHub Actions でビルド・適用・ベンチマークを取ってちゃんと動くことを保証しています。

Nix は Windows をネイティブサポートしていません。それではすべての OS を宣言的に管理できなくて困ります。

そこでシステムは Windows のパッケージマネージャ WinGet を用いて宣言的に管理し、ユーザーレベルは WSL 上で Home Manager でビルドして、ビルド結果をスクリプトを用いて適用します。

### LLM に改善案を出してもらえる

Nix ですべてを宣言的に書いているので自分の環境にとって最善の選択を選ぶことができます。

具体的には `AGENTS.md` に「URL を渡されたらどう分析するか」を書いています。

```markdown:AGENTS.md
## Reference Analysis

When user shares a URL (article, repository, dotfiles):
1. Fetch: Clone repo or fetch article
2. Analyze: Read all Nix files, README, structure
3. Compare: Missing inputs, better patterns, new tools
4. Prioritize: High / Medium / Low
5. Plan: Enter plan mode with specific file changes
6. Apply: After approval, implement → fmt → build
```

生産性向上に関する記事や他の dotfiles について良いなと思ったものを探してきて貼ると、LLM がそれをよく調べて改善案を提示してくれます。

例えばこんな感じ

```text
TODO
```

さらにツールの選定基準を `docs/` で言語化して LLM に読ませることで意図を汲み取ってリサーチの最適化をしてくれます。私であれば徹底的に認知負荷が低くなるように入れていて、不採用にした理由も残して似たツールは再提案されないようにしてます。

こういった言語化をしていくことで精度が上がり全部任せて大丈夫な環境にしていきます。

### 機密情報を外に逃す

AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...みたいな想定しますよね。

雑にやると攻撃されかねないので外部サービスにすべて逃がしましょう。私はすべての機密情報を 1Password に置いていて、SSH や Git で秘密鍵が必要になったら Touch ID や Windows Hello などの生体認証で呼び出しています。API キーの環境変数は op コマンドで起動時に展開するようにしてます。漏洩防止として pre-commit に gitleaks を入れています。

パスワードマネージャーは E2EE でサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全です。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので SOC 2 認証を受けたサービスを選びましょう。

## 移行方法

僕のおすすめとしては理解が深まるので自分で Nix を書くことなのですが、考えるべきことが多いので以下のリポジトリを用いて設定する方法もお伝えします。

https://github.com/anko9801/dotfiles

3 ステップです。

1. リポジトリを fork clone する
2. `config.nix` にユーザー情報とホストに入れる環境を最小限に選択する
3. Nix を入れて `nix run .#switch -- <ホスト名>` と適用する

これで既存の環境にツールが揃った状態で立ち上がります。

そこから自分に合わない部分を削ったり、足りないものを追加したりすればいいです。`config.nix` の `hosts` でマシンごとのモジュール構成を変えられるし、GUI の有無も切り替えられます。また既存の環境を取り込みたい場合は nix-migrate というスキルを叩いて LLM に Nix へ移行してもらうこともできます。

## まとめ

バグから生まれた dotfiles、最初はただの設定ファイルの寄せ集めだったのが、宣言的に管理することでマシンの環境を記述し、今では LLM によって自動でフィードバックループが回せるようになっています。

あなたの dotfiles にも、まだ言語化されていない「なぜこのツールなのか」が眠っているはず。それを Nix で宣言し、`docs/` に書き出して LLM に読ませたとき、自分でも忘れていたこだわりが返ってくるかもしれません。
