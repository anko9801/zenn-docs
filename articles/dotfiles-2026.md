---
title: "フィードバックループを回す dotfiles"
emoji: "🦔"
type: "tech"
topics: ["nix"]
published: false
---

みなさん！dotfiles 作ってますか？

ｳｵｰ!! みなさんのたくさんのこだわりの dotfiles が見えました！

生産性を上げたい。こだわりのキーボード、チェア、ガジェットやツール、Vim で LLM で楽に自動化できたりして嬉しい。

しかし、私用 PC、社用 PC、コンテナ、クラウドなど年々増え続けるマシン、LLM によって開発効率・リリース頻度が上がり加速的に増え続けるアプリ、それに比例して増える設定やキーバインド、そんなの覚えてらんないし、それぞれのマシンにインストール方法しらべて前の設定みつけて入れるなんて苦行だよ～とそんなこんなで日々新しいアプリがでているけれどもすっと手が伸びにくくなってるんじゃないでしょうか。

それを解決するのが今回紹介したい Nix + LLM による管理です。エンジニアだけではなく、PCのツールを管理したい人全般の人に対しておすすめできます。新しくて便利なものを触り続けたい人必見です。

## dotfiles がどう進化してきたか

エンジニアにとって PC は毎日朝から晩まで触るもので、そうすると日々負担に感じているものも至る所にあります。これを便利な設定や便利なツールを入れることで解消し、作業効率を上げたいという営みがあります。

Linux において `.zshrc` `.vimrc` など `.` から始まるファイルたち dotfiles というのがその設定に当たります。これは UNIX を開発した Ken Thompson が ls において `.` (現在のディレクトリ) `..` (親のディレクトリ) が毎回表示されていて煩わしかったので先頭が `.` なら表示しないとしたことで、本来見せなくていい設定についてはドットをつけて隠しファイルにできるバグが生まれ、さまざまなアプリケーションで設定ファイルに使うのが便利で設定を dotfiles というようになってます。

さてそんな dotfiles を管理する方法はだんだんと進化して移り変わってきました。

### 設定を同期したい

設定を育てていて最初に困るのはその育てた設定が他のマシンには入ってなくて不便なままなことです。私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からになってしまいます。

そこで dotfiles というリポジトリにコピペして同期させるようにしました。これで他のマシンにも設定を入れやすいし、ついでに他の人の工夫も参考にできてうれしいです。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。自分でスクリプトを書くと「既存の設定とコンフリクトしたときに正しくマージできてるかな」「設定が増えたらスクリプトも書き足さないと」「Windows のリンクの仕様がトリッキー」みたいに考えることが多いのでツールが生まれました。

- シンボリックリンク方式 ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
- コピー方式 ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
- 直接管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデメありますが、怠惰な僕にとっては yadm が好みです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすい。テンプレート機能、シークレット管理もしてくれます。

これで設定を管理するのは楽になりました。

### 環境そのものを再現したい

設定ができても新しいマシンで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いです。

じゃあインストールも自動化しちゃおう！

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

しかしこれだと他のマシンに適用したときに動的ライブラリやランタイムのバージョンやパスの違いなどの暗黙的な依存関係よって動かないときがよくあります。動く環境そのものが欲しいのだからこういった問題は起きてほしくないです。そこで暗黙的な依存関係を明示し、それをそのまま持ってくるようにすることでどこでも環境が再現するでしょと言ったのが [Nix](https://github.com/NixOS/nix) です。

さらに Nix によりすべてのツールを宣言的に管理することで、単なる設定集ではなく、マシンの環境を記述したものと言えるようになります。

### 管理コストを下げたい

dotfiles を育てるのも少し大変です。

1. 記事を読んだり、他の dotfiles を見てよさそうなのを見つける
2. 自分の環境にどう入れるか、自分の哲学と合っているか考える
3. 設定ファイルを編集する
4. 適用してみる
5. 壊れたら直す
6. 他のマシンにも反映する

実は Nix で宣言的にマシン全体の環境が書かれてあると人間も読みやすいけど LLM も読みやすくて 2 度おいしいんです。Nix + LLM を使ってやると

1. LLM に大量の記事をリサーチして改善点を洗い出してもらう
2. ユーザーの判断基準をもとに決定し、書いてもらう
3. 壊れたら Nix のロールバックで戻せる
4. 全ホストにリモート適用

とステップ数が削れて判断基準も自動化したければ自分の哲学を文章にして与えればできます。これにより LLM によるフィードバックループを回せて、バグが起きたり日々のアップデートや改善すべき点が見つかれば修正し、デプロイされるというのが全自動で行われます。

## 結論設計

今までをまとめると Nix + LLM でやるとメリットとしては

1. 共有できる
  設定を他のマシンや他の人と共有できる
2. 再現できる
  誰がいつどのマシンで実行しても同じ環境が立ち上がる
3. 依存が明示的
  ライブラリ、ツール、環境変数まですべて記述されている

Nix + LLM による管理がよさそうですがデメリットもあって

1. ストレージ消費が多い
  依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数存在します。またガベージコレクションしないとロールバック用の古い世代が残ってディスクが圧迫します。SSD の寿命や容量が気になる環境にはあまり向かないかもしれないです。
1. Nix が Windows に対応していない
  これについては Windows のパッケージマネージャ WinGet を WSL から使うことでインストールと Nix でビルドした設定を与えられます。

これらを解決しつつ、どの OS のどのユーザーでもバグなく同じ環境が再現され、マシン間で同期し、他人の良い設定は流し込むだけで取り込め、すべての設定に理由が残っていて見返しても迷わず、既存の環境を壊さず段階的に採用でき、機密情報はディスクに一切残さず、情報を与えるほど生産性が上がり続ける dotfiles を設計していきます。そしてセットアップは Nix をインストールして `nix run github:anko9801/dotfiles#switch` を叩くだけでいいようにします。

### 全体構造

Nix は割と自由なディレクトリ構成をすることができてエントリーポイントの flake.nix のみあれば他はほとんど制約はありません。逆に言えばどんなディレクトリ構成にすべきかで使いやすさが変わります。

```
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
├── shell/                  # Zsh, Fish, Bash, Starship, ...
├── editor/                 # Neovim, Vim, VSCode, Helix, Zed
├── terminal/               # Ghostty, Zellij, tmux
├── tools/                  # Git, Yazi, Bat, ...
├── dev/                    # Rust, Go, Python, Node, mise
├── security/               # 1Password, SSH, GPG, gitleaks
├── ai/                     # Claude Code, Aider
└── desktop/                # Wayland, IME
```

基本的にカテゴリごとにディレクトリを分け、各 `.nix` ファイルが 1 つのアプリの責務を持っています。たとえば fish の設定を変えたかったら `shell/fish.nix` を編集します。

Nix はシステムレベルでは NixOS は `/etc/nixos/configuration.nix`、macOS では nix-darwin を用いて設定し、ユーザーレベルは Home Manager を用いて管理しています。そのようなシステムレベルやユーザーレベルの全体的な設定を `system/` が管理しています。

https://github.com/nix-community/home-manager


### `config.nix` で全ホストを宣言する

まずユーザー情報、全ホストの構成、モジュールの組み合わせがすべて `config.nix` に集約されていて、これを数行編集することで新たなホストの設定が完了します。

```nix:config.nix
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

### Stylix でテーマを一括管理する

カラースキーム、フォント、カーソルを Stylix で一括管理して、ターミナルやエディターなどそれぞれのツールにその情報を渡して設定することで全体的に一貫性をもった色合いになってくれます。

```nix
# theme/default.nix
stylix = {
  enable = true;
  fonts.monospace = {
    package = pkgs.moralerspace;
    name = "Moralerspace Neon";
  };
  cursor = {
    package = pkgs.bibata-cursors;
    name = "Bibata-Modern-Classic";
    size = 24;
  };
  opacity.terminal = 0.95;
};
```

カラーテーマはファイルごとに分けてホストごとにテーマを変えられるようにしています。

### 機密情報をディスクに残さない

AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...みたいな想定しますよね。

すべての機密情報は 1Password に置いています。秘密鍵はディスクに書かず、Touch ID や Windows Hello の生体認証で呼び出しています。API キーの乗った環境変数は op コマンドで起動時に展開するようにしてます。

雑に管理したりセルフホストすると脆弱性を埋め込むので外部サービスに全部置きましょう。E2EE でユーザー以外は暗号化されておりサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので法律が機能する法人に頼みましょう。

1Password CLI は WSL では Windows 側の `op.exe` を使っていて、シェル側では op プラグインを遅延ロードして gh / aws / gcloud / az を初回実行時に初期化しています。漏洩防止として pre-commit に gitleaks を入れていて、`op://` の参照文字列は誤検知しないよう許可リスト化してます。


### GitHub Actions で全ホストのビルドを検証する

`config.nix` に CI 用の `runner` ユーザーが定義してあります。

GitHub Actions で全ホスト構成の `nix flake check` と `home-manager build` を実行すれば、壊れた設定が main に入ることを防げます。CI 環境では Nerd Fonts のインストールや systemd のサービス起動を自動でスキップするように `platform.environment` で分岐しています。

### Windows 対応

Nix は Windows をネイティブサポートしていません。それではすべての OS を宣言的に管理できなくて困ります。

そこでシステムは Windows のパッケージマネージャ WinGet を用いて宣言的に管理し、ユーザーレベルは WSL 上で Home Manager でビルドして、ビルド結果をスクリプトを用いて適用します。

### LLM に調査を任せる

自分のこだわりや方針は自分で決めます。ただ、新しいツールを調べたり他人の dotfiles を比較したりする**調査の手間**は LLM に任せられます。すべてが宣言的に書いてあるから、LLM が構造を理解して「自分の環境にとって意味のある情報」を拾ってきてくれます。

具体的には `AGENTS.md` に「URL を渡されたらどう分析するか」を書いています。

```markdown
## Reference Analysis

When user shares a URL (article, repository, dotfiles):
1. Fetch: Clone repo or fetch article
2. Analyze: Read all Nix files, README, structure
3. Compare: Missing inputs, better patterns, new tools
4. Prioritize: High / Medium / Low
5. Plan: Enter plan mode with specific file changes
6. Apply: After approval, implement → fmt → build
```

気になる記事やリポジトリの URL を投げると、自分の dotfiles と比較して改善点を優先度つきでリストアップしてくれます。採用するかどうかは自分で決めます。

さらに `docs/tool-selection.md` に自分の選定基準を文書化しておくと、LLM が提案する際にその基準を踏まえてくれます。「シンプルさ最優先」と書いてあれば、高機能だが複雑なツールは推薦しなくなります。

```markdown
### Rejected
| Tool | Reason |
|------|--------|
| Fish, Nushell | Not POSIX — LLM assistance fails, requires rewrites |
| gitui | No worktree; no customCommands |
```

不採用の理由を具体的に残しておくと、同じようなツールを再提案されなくなります。判断の積み重ねがそのまま LLM の精度向上に繋がります。

## 移行方法

Nix + LLM に段階的に移行する方法について解説します。

僕のおすすめとしては理解が深まるので自分で Nix を書くことなのですが、考えるべきことが多いので以下のリポジトリを用いて設定する方法もお伝えします。

https://github.com/anko9801/dotfiles

### 安全に移行するために

移行において安心して実行できる環境が重要で、勝手にユーザー名が書き換わったり、バックアップがなされずに環境が壊されてしまうみたいな悲劇が起こらない仕組みづくりが重要です。

例えば 1 コマンドでセットアップがすべて済む setup スクリプトは便利ですが他の人のスクリプトを実行するのはセキュリティ的な面や環境が破壊されるリスクによる抵抗がとても大きいので書かず、自分で Nix をインストールし、実行してもらうという形をとっています。

また Nix には `home-manager.backupFileExtension = "backup"` という既存ファイルは自動でバックアップされる機能があり、`.zshrc` が既にあれば `.zshrc.backup` にリネームされてから新しい設定が適用されます。なので脳死で適用して大丈夫で、何か問題があれば `.backup` ファイルから戻せます。


### やり方

まずリポジトリを fork して clone します。そこにある `config.nix` を書き換えて適用するだけで終わりです。

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
    modules = [
      ./tools/git
      ./tools/cli.nix
      ./tools/bat.nix
    ];
  };
};
```

こんな感じでユーザー情報と基本ツールだけ書き換えれば段階的に有効化できます。これなら既存の `.zshrc` や `.vimrc` はそのまま残ります。慣れてきたら `./shell/zsh` を追加して移行していけばいいです。モジュール単位で段階的に移行できるのは Nix の強みです。

それができたら適用します。

```bash
nix run .#switch -- linux-wsl
```

これで既存の環境にツールが揃った状態で立ち上がります。

そこから自分に合わない部分を削ったり、足りないものを追加したりすればいいです。`config.nix` の `hosts` でマシンごとのモジュール構成を変えられるし、GUI の有無も切り替えられます。

また既存の環境を取り込みたい場合は nix-migrate というスキルを叩いて LLM に Nix へ移行してもらうこともできます。

## まとめ

バグから生まれた dotfiles、最初はただの設定ファイルの寄せ集めだったのが、宣言的に管理することでマシンの環境を記述し、今では LLM によって自動でフィードバックループが回せるようになっています。

これで日々でてくる最新のツールを入れまくっても破綻せず、こだわりの作業環境を追い求めることができるんじゃないでしょうか。
