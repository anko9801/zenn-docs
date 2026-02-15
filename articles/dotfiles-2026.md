---
title: "フィードバックループを回す dotfiles"
emoji: "🦔"
type: "tech"
topics: ["nix"]
published: false
---

:::message
注意: AI と
:::


みなさん！dotfiles 作ってますか？

ｳｵｰ!! みなさんのたくさんのこだわりの dotfiles が見えました！

生産性向上をあげたい。こだわりのキーボード、チェア、ガジェットやツール、Vim で LLM で楽に自動化できたりして嬉しい。

しかし、私用 PC、社用 PC、コンテナ、クラウドなど年々増え続けるマシン、LLM によって開発効率・リリース頻度が上がり加速的に増え続けるアプリ、それに比例して増える設定やキーバインド、そんなの覚えてらんないし、それぞれのマシンにインストール方法しらべて前の設定みつけて入れるなんて苦行だよ～とそんなこんなで日々新しいアプリがでているけれどもすっと手が伸びにくくなってるんじゃないでしょうか。

それを解決するのが今回紹介したい Nix + LLM による管理です。エンジニアだけではなく、PCのツールを管理したい人全般の人に対しておすすめできます。新しくて便利なものを触り続けたい人必見です。

生産性向上 = 人の負担を徹底的に減らす環境を作るためにはどうすべきか？

## dotfiles がどう進化してきたか

エンジニアにとって PC は毎日朝から晩まで触るもので、そうすると日々負担に感じているものも至る所にあります。これを便利な設定や便利なツールを入れることで解消し、作業効率を上げたいという営みがあります。

Linux において `.zshrc` `.vimrc` など `.` から始まるファイルたち dotfiles というのがその設定に当たります。これは UNIX を開発した Ken Thompson が ls において `.` (現在のディレクトリ) `..` (親のディレクトリ) が毎回表示されていて煩わしかったので先頭が `.` なら表示しないとしたことで、本来見せなくていい設定についてはドットをつけて隠しファイルにできるバグが生まれ、便利でさまざまなアプリケーションで使われたため設定のことを dotfiles というようになってます。

さてそんな dotfiles を管理する方法はだんだんと進化して移り変わってきました。

### 設定を同期したい

設定を育てていて最初に困るのはその育てた設定が他のマシンには入ってなくて不便なままなことです。私用 PC で使っていた `.zshrc` が社用 PC で使えなかったり WSL を入れ直したら最初からになってしまいます。

そこで dotfiles というリポジトリにコピペして同期させるようにしました。これで他のマシンにも設定を入れやすいし、ついでに他の人の工夫も参考にできてうれしいです。

ただリポジトリに入れただけじゃ適用されなくて、ファイルをホームディレクトリにコピペしたり、リンクを貼らないといけません。自分でスクリプトを書くと「既存の設定とコンフリクトしたときに正しくマージできてるかな」「設定が増えたらスクリプトも書き足さないと」「Windows のリンクの仕様がトリッキー」みたいに考えることが多いのでツールが生まれました。

- シンボリックリンク方式 ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
- コピー方式 ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
- 直接管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデリありますが、怠惰な僕にとっては yadm が好みです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすい。テンプレート機能、シークレット管理もしてくれます。

これで設定を管理するのは楽になりました。

### 環境そのものを再現したい

設定ができても新しいマシンで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いです。

じゃあインストールも自動化しちゃおう！

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Nix](https://github.com/NixOS/nix), [Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

しかしこれだと他のマシンに適用したときに動的ライブラリやランタイムのバージョンやパスの違いなどの暗黙的な依存関係よって動かないときがよくあります。動く環境そのものが欲しいのだからこういった問題は起きてほしくないです。そこで暗黙的な依存関係を明示し、それをそのまま持ってくるようにすることでどこでも環境が再現するでしょと言ったのが Nix です。

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

さて Nix + LLM による管理がよさそうですがデメリットもあって

1. ストレージ消費が多い
  依存関係をバージョンごとに隔離しているので同じライブラリの異なるバージョンが複数存在します。またガベージコレクションしないとロールバック用の古い世代が残ってディスクが圧迫します。SSD の寿命や容量が気になる環境にはあまり向かないかもしれないです。
2. Nix が Windows に対応していない
  これについては Windows のパッケージマネージャ WinGet を WSL から使うことでインストールと Nix でビルドした設定を与えられます。

これらを解決しつつ、どの OS のどのユーザーでもバグなく同じ環境が再現され、マシン間で同期し、他人の良い設定は流し込むだけで取り込め、すべての設定に理由が残っていて見返しても迷わず、既存の環境を壊さず段階的に採用でき、機密情報はディスクに一切残さず、情報を与えるほど生産性が上がり続ける dotfiles を設計していきます。そしてセットアップは Nix をインストールして `nix run github:anko9801/dotfiles#switch` を叩くだけでいいようにします。

### 全体構造

Nix は割と自由なディレクトリ構成をすることができてエントリーポイントの flake.nix のみあれば他はほとんど制約はありません。逆に言えばどんなディレクトリ構成にすべきかで使いやすさが変わります。

```
dotfiles/
├── flake.nix          # エントリーポイント
├── config.nix         # ユーザー・ホスト・モジュールの全定義
├── AGENTS.md          # LLM への作業手順
├── docs/              # 判断基準と意思決定の記録
│   └── tool-selection.md
├── system/            # OS ごとのシステム設定・ビルダー関数群
│   ├── darwin/
│   ├── nixos/
│   └── windows/
├── theme/             # Stylix (全ツールの色・フォント・カーソル)
├── shell/             # Zsh, Fish, Bash, Starship, ...
├── editor/            # Neovim, Vim, VSCode, Helix, Zed
├── terminal/          # Ghostty, Zellij, tmux
├── tools/             # Git, Yazi, Bat, ...
├── dev/               # Rust, Go, Python, Node, mise
├── security/          # 1Password, SSH, GPG, gitleaks
├── ai/                # Claude Code, Aider
└── desktop/           # Wayland, IME
```

基本的にカテゴリごとにディレクトリを分け、各 `.nix` ファイルが 1 つのアプリの責務を持っています。たとえば fish の設定を変えたかったら `shell/fish.nix` を編集します。

Nix はシステムレベルでは NixOS は `/etc/nix/configuration.nix`、macOS では nix-darwin を用いて設定し、ユーザーレベルは Home Manager を用いて管理しています。そのようなシステムレベルやユーザーレベルの全体的な設定を `system/` が管理しています。


### `config.nix` で全ホストを宣言する

まずユーザー情報、全ホストの構成、モジュールの組み合わせがすべて `config.nix` に集約されていいて、これを数行編集することで新たなホストの設定が完了します。

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

`moduleSets.workstation` と `moduleSets.server` で共通モジュールをまとめ、ホストごとに差分だけ追加する。新しいマシンが増えたら数行書くだけでいい。fork するユーザーはこのファイルのユーザー情報とホスト構成を書き換えるだけで、自分の環境が立ち上がる。

### Stylix でテーマを一括管理する

カラースキーム、フォント、カーソルを Stylix で一括管理する。1 箇所変えるとターミナルからエディタまで全ツールに反映される。

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

色のテーマは `theme/catppuccin-mocha.nix` で定義し、フォントやカーソルは `theme/default.nix` で設定する。ツールごとにカラーコードをコピーして回る必要がない。

### 1Password で機密情報をディスクに残さない

すべての機密情報は 1Password に置く。秘密鍵はディスクに書かず、Touch ID や Windows Hello の生体認証で呼び出す。

<!-- 1Password の op:// 参照、SSH エージェント、Git 署名の具体例を入れる -->

### GitHub Actions で全ホストのビルドを検証する

`config.nix` に CI 用の `runner` ユーザーが定義してある。

```nix
runner = {
  editor = "vim";
  git = {
    name = "CI";
    email = "ci@localhost";
  };
};
```

GitHub Actions で全ホスト構成の `nix flake check` と `home-manager build` を実行すれば、壊れた設定が main に入ることを防げる。CI 環境では Nerd Fonts のインストールや systemd のサービス起動を自動でスキップするように `platform.environment` で分岐している。

<!-- GitHub Actions の workflow YAML を入れる -->

### Windows 対応

Nix は Windows をネイティブサポートしない。しかし「すべての OS を宣言的に管理する」という原則は崩さない。WSL 上で Home Manager を使いつつ、Windows ネイティブ側は WinGet の JSON で宣言的に管理する。


### LLM に調査を任せる

自分のこだわりや方針は自分で決める。ただ、新しいツールを調べたり他人の dotfiles を比較したりする**調査の手間**は LLM に任せられる。すべてが宣言的に書いてあるから、LLM が構造を理解して「自分の環境にとって意味のある情報」を拾ってきてくれる。

具体的には `AGENTS.md` に「URL を渡されたらどう分析するか」を書いてある。

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

気になる記事やリポジトリの URL を投げると、自分の dotfiles と比較して改善点を優先度つきでリストアップしてくれる。採用するかどうかは自分で決める。

さらに `docs/tool-selection.md` に自分の選定基準を文書化しておくと、LLM が提案する際にその基準を踏まえてくれる。「シンプルさ最優先」と書いてあれば、高機能だが複雑なツールは推薦しなくなる。

```markdown
### Rejected
| Tool | Reason |
|------|--------|
| Fish, Nushell | Not POSIX — LLM assistance fails, requires rewrites |
| gitui | No worktree; no customCommands |
```

不採用の理由を具体的に残しておくと、同じようなツールを再提案されなくなる。判断の積み重ねがそのまま LLM の精度向上に繋がる。

こだわりは自分のもの。LLM はそのこだわりを実現するための調査係だ。
### セキュリティ

例えば AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...みたいな想定しますよね。

こういうのは雑に管理したり気軽にセルフホストすると脆弱性を埋め込むので外部サービスに全部置きましょう。僕は 1Password が好きですべての機密情報をここで管理してます。E2EE でユーザー以外は暗号化されておりサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので法律が機能する法人に頼みましょう。

1Password のおすすめ機能はこちら。

- SSH エージェント: `~/.1password/agent.sock` 経由で GitHub/GitLab に接続。秘密鍵はディスクに書かず Touch ID や Windows Hello の生体認証で呼び出す
- API キー: env ファイルに `OPENAI_API_KEY=op://Personal/OpenAI/credential` みたいに書くと op コマンドで起動時に展開してくれる
- Git コミット署名: SSH 鍵で署名していて秘密鍵がディスクに存在しない。1Password がロック中は署名できない

1Password CLI は WSL では Windows 側の `op.exe` を使っていて、シェル側では op プラグインを遅延ロードして gh / aws / gcloud / az を初回実行時に初期化しています。漏洩防止として pre-commit に gitleaks を入れていて、`op://` の参照文字列は誤検知しないよう許可リスト化してます。

## 移行
setupスクリプトは書かない

ここまで解説してきた設計はすべて、以下のリポジトリに実装されている。`config.nix` のユーザー情報を書き換えるだけで動く。

```bash
# 1. fork して clone
git clone https://github.com/your-username/dotfiles.git ~/dotfiles
cd ~/dotfiles
```

```nix
# 2. config.nix のユーザー情報を自分のものに書き換える
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
```

#### 既存の設定との衝突を避ける


**既存ファイルは自動でバックアップされる。** このリポジトリでは `home-manager.backupFileExtension = "backup"` が設定済みなので、`.zshrc` が既にあれば `.zshrc.backup` にリネームされてから新しい設定が適用される。何も考えずに `nix run .#switch` して大丈夫だ。何か問題があれば `.backup` ファイルから戻せる。

それでも不安なら、`config.nix` の `modules` を絞って段階的に有効化することもできる。

```nix
# 最初は git と基本ツールだけ — シェルやエディタの設定には触らない
linux-wsl = {
  system = "x86_64-linux";
  manager = "home-manager";
  modules = [
    ./tools/git
    ./tools/cli.nix
    ./tools/bat.nix
  ];
};
```

これなら既存の `.zshrc` や `.vimrc` はそのまま残る。慣れてきたら `./shell/zsh` を追加して移行していけばいい。モジュール単位で段階的に移行できるのは Nix の強みだ。

```bash
# 3. 適用する
nix run .#switch
```

これだけでシェル、エディタ、Git、ターミナルまで全部揃った環境が立ち上がる。

そこから自分に合わない部分を削ったり、足りないものを追加したりすればいい。`config.nix` の `hosts` でマシンごとのモジュール構成を変えられるし、`moduleSets.workstation` と `moduleSets.server` で GUI の有無も切り替えられる。

```nix
# 例：WSL ではワークステーション構成 + zellij
linux-wsl = {
  system = "x86_64-linux";
  manager = "home-manager";
  modules = moduleSets.workstation ++ [ ./terminal/zellij ];
};

# 例：サーバーはミニマル構成
linux-server-arm = {
  system = "aarch64-linux";
  manager = "home-manager";
  modules = moduleSets.server;
};
```

`AGENTS.md` と `docs/tool-selection.md` も入っているので、前述の LLM 調査もそのまま使える。選定基準は自分の価値観に合わせて書き換えていけばいい。


## 締め

dotfiles は 50 年前のバグから生まれた。最初はただの設定ファイルの寄せ集めだったが、宣言的に管理することで「マシンのあるべき姿の記述」になる。どの OS でも 1 コマンドで同じ環境が立ち上がり、こだわりの設定が確実に再現される。認知負荷を削って、自分が本当に集中したいことに向かうための土台だ。

fork して `config.nix` のユーザー情報を書き換えて `nix run .#switch` する。そこから自分のこだわりを載せていってほしい。

https://github.com/anko9801/dotfiles



## おまけ: ツールへのこだわり

生産性向上とは認知負荷の削減。認知負荷というのはタスクに対して短期記憶から払い出されるメモリのことで、これが少ないほど本質をより深く考えられるようになります。例えば関数型だったり宣言的とかが認知負荷の削減になります。移植性のある少数精鋭。ツールの選定は次の順でやってます。

1. シンプル (Simple is not Easy)
2. 不具合が少ない・ユーザーが多い
3. クロスプラットフォーム
4. 高パフォーマンス
5. セキュリティ

他の方と大体同じなのでサクサクいきます。

### 日用系ツール

ターミナルは Ghostty と Windows Terminal を使っています。Wezterm を作り込みたいな～と思って放置してます。ターミナルマルチプレクサは Ghostty でコピーオンセレクトやキーバインドなど再現できるので入れてないです。プロンプトは Starship、IME は SKK、ランチャーは Raycast、エディターはデスクトップなら Cursor ノパソは Zed 気分で Neovim にしています。

シェルは zsh を使っています。fish shell や Nushell などの非 POSIX 準拠シェルだと書き換えが必要だったり LLM で間違うことが多くて使いづらいのと、今やプラグインが充実して zsh でも補完などの体験が良くて困ってないです。

| zsh プラグイン | 設定 |
|---|---|
| fzf-tab | git / ssh / kubectl / terraform などで fzf を使えるように |
| zsh-abbr | スペースを押すと展開されるエイリアス |
| zsh-autosuggestions | 補完 |
| zsh-defer | 遅延読み込み |
| zsh-fast-syntax-highlighting | シンタックスハイライト |

シェルの起動時間は 200ms 以下だと体感変わらなくなるので 200ms に収まるように zsh-defer で遅延読み込みを調整しています。

その他ツールについてはコマンド履歴は atuin、移動は zoxide を cd に割り当て、ランタイムと linter / formatter は mise で管理してます。Nix あるなら mise いらんやろ。Neovim や pbcopy コマンドで OSC 52 に対応して SSH 越しでもクリップボードが飛ぶようにしてます。地味に便利なのは GNU 系コマンドの置き換えで find → fd, grep → rg, htop → btm みたいに zsh-abbr でエイリアスを貼ると、スペース押した瞬間に展開されるので移行コストなく自然に覚えられます。


### カラースキーム & フォント & キーバインディング

ツール全体で統一したいものもたくさんあると思います。

認知負荷を下げる為にもツール全体で統一したいものっていくつかあって Stylix と温かみのある手作業でやってます。僕の設定だとテーマは [Catppuccin Mocha](https://github.com/catppuccin)、フォントは [Moralerspace](https://github.com/yuru7/moralerspace)、キーリマッパーは [Kanata](https://github.com/jtroo/kanata)、キーバインディングは「目的語 + 動詞」で統一してます。

Ctrl, Alt, Win/Cmd の 3 種類のキーがあって、目的語であるアプリケーションについてこう対応させています。

- ウィンドウマネージャー → Win/Cmd
- ペイン → Ctrl-q
- タブ → Ctrl-t
- シェル → Ctrl-g
- Vim → Space

動詞は上下左右に動かすときは hjkl、リサイズは HJKL、縦横分割は |-、閉じるは x、ズームは z、新規は n みたいな感じでやってます。

こうすると覚えることが少なくてめっちゃ楽です。ただしマウスでやるよりキーボードでやった方が基本速いですが、キーボードに偏りすぎると認知負荷が高くなって疲れてしまうので人それぞれの良いバランスでやるといいかなと思ってます。本当は英語の文法順にあわせて動詞 + 目的語にして認知負荷減らしたいですがむずい。あと目的語の方の名前空間が枯渇しがちなのでリマップをもっと凝ったり自作キーボードもこだわりたいなとも思ってます。

Vim/Emacs は Esc が Tab の位置だったり、Meta Super Hyper などのキーがあったりした 1970 年代のキーボードを元に設計されてるので、RSI 予防に CapsLock をタップで Esc、長押しで Ctrl にリマップしたり、できるだけ Ctrl に近い Q キーなどで組んだりしてます。ゲームと作業を切り替えがちなので Home Row Mod とかは相性が悪い感じです。NeoVim だと which-key.nvim があるとキーが覚えやすくて便利です。zsh-abbr もモダンツールを覚える為に使うのもいいです。


### Git

僕は ghq より zoxide が好きでディレクトリで自由にカテゴリ分けできるからリポジトリ名を忘れたときに探しやすいし、新しくコマンドを覚えなくていいので助かります。差分は構文解析した上で表示してくれる difftastic、コンフリクトは差分に加えて共通祖先も表示する zdiff3、コミットは絵文字や Conventional Commit がインタラクティブにできる czg を使っています。

気に入ってるエイリアス 5 選

```bash
# pull してマージされたブランチを削除
git pl # pull --ff-only --rebase --prune && git branch -vv | grep ': gone]' | awk '{print $1}' | xargs -r git branch -d
# force push おねがい
git please # push --force-with-lease --force-if-includes
# worktree 操作が楽ちん (git-wt)
git wt <branch>
# 現在の変更を過去のコミットに自動で吸い取ってもらう (git-absorb)
git absorb
# 直前のコミットを取り消す
git undo
```

設定としては pre-commit に gitleaks を設定してシークレット漏洩を防ぎ、GitHub の noreply アドレスを使ってコミットにアドレスを残さないようにしています。

lazygit には czg を直接叩くカスタムコマンドを入れていて、TUI からそのまま Conventional Commit + emoji で書けるようにしています。

jj は一応入れてる状態で使えてないので今後活用していきたいです。


### LLM

Claude Code のみを使ってます。CLAUDE.md やスラッシュコマンドを管理しています。インターフェースはちょっと悩んでいてキーボードの他に音声操作を使っていて割と適当に指示しても動いてくれます。しかしなにか別の作業をしているときにお口がフリーなのでお菓子で忙しくないときは自分の作業に干渉しない音声操作がすごく欲しいです。


### まとめ

最近の環境構築系の記事を眺めると、流れはだいぶ「宣言的・再現性・安全性」に寄っていて、手続き的に手で直すより、差分が追える形で設計するのが主流になっています。Nix はストアの分離とロールバック、バイナリキャッシュで「壊れにくい更新」と「再現性」を優先する設計が明確で、Home Manager でユーザー環境まで統一するのが定番っぽいです。

https://github.com/anko9801/dotfiles










## なぜ Nix なのか

ここまでの進化をまとめると、dotfiles に求められる性質は 3 つある。

- **共有できる**: 設定を他のマシンや他の人と共有できる
- **再現できる**: 誰がいつどのマシンで実行しても同じ環境が立ち上がる
- **依存が明示的**: ライブラリ、ツール、環境変数まですべて記述されている

Nix はこの 3 つをすべて満たす。すべてのパッケージをその依存関係のハッシュで一意に識別し、`/nix/store` 以下に隔離して配置する。同じ入力からは同じ出力が得られ、異なるバージョンが互いに干渉しない。[Home Manager](https://github.com/nix-community/home-manager) を使えば、dotfiles をモジュールとして宣言的に管理し、`home-manager switch` 1 コマンドで設定もパッケージもまとめて適用できる。前述のロールバックもこの仕組みの上に成り立っている。

### トレードオフは正直に

**学習コストが高い。** Nix 言語には独特の癖があり、エラーメッセージも親切とは言いがたい。慣れるまでの立ち上がりは他のツールより明らかに重い。

**ストレージ消費が多い。** 依存関係をバージョンごとに隔離するので、同じライブラリの異なるバージョンが複数存在する。ガベージコレクションで古い世代を定期的に削除しないとディスクが圧迫される。SSD の寿命や容量が気になる環境にはやや不向き。

**Windows は Nix 単体ではカバーできない。** ただし WSL 上で Home Manager を使いつつ、Windows ネイティブ側は WinGet の JSON で宣言的に管理すれば、実質すべての OS を統一的に扱える。この方法は設計パターンの章で詳しく説明する。

これらを踏まえた上で「複数のマシンを管理する」「環境の再現性が重要」「壊れたときにすぐ戻したい」のどれかに当てはまるなら、学習コストを払う価値はある。


## 移行ステップ
