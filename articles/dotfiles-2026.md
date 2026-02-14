---
title: "Nix によるこれからの時代の dotfiles"
emoji: "🦔"
type: "tech"
topics: ["nix"]
published: false
---

みなさん！dotfiles 作ってますか？

ｳｵｰ!! みなさんのたくさんのこだわりの dotfiles が見えました！

生産性向上って気になるよね。こだわりのキーボード、チェア、ガジェットやツール、LLM で楽に自動化できたりして嬉しい。

しかし、私用 PC、社用 PC、コンテナ、クラウドなど年々増え続けるマシン、LLM によって開発効率・リリース頻度が上がり加速的に増え続けるアプリ、それに比例して増える設定やキーバインド、そんなの覚えてらんないし、それぞれのマシンにインストール方法しらべて前の設定みつけて入れるなんて苦行だよ～とそんなこんなで日々新しいアプリがでているけれどもすっと手が伸びにくくなってるんじゃないでしょうか。

それを解決するのが今回紹介したい Nix + LLM による管理です。エンジニアだけではなく、PCのツールを管理したい人全般の人に対しておすすめできます。新しくて便利なものを触り続けたい人必見です。

## dotfiles がどう進化してきたか

エンジニアをやってるといろんなアプリをカスタマイズしたくなるので .zshrc や .vimrc などの設定ファイルを作って、自分好みに育てます。そしてこの設定が育ってくると他のマシンをセットアップするときに不便なので dotfiles リポジトリに手動でコピペして git で管理します。

- 設定を共有して互いの設定を参考にできる
- 設定を適用するにはコピペやリンクしないとでめんどくさい
- インストールもやらないといけない

じゃあ適用を自動化しよう！となるんですが、自分でスクリプトを書くと「既存の設定とコンフリクトしたときに正しく」「設定が増えたらスクリプトも書き足さないと」「Windows のリンクの仕様がトリッキー」だったりして考えることが多いのでツールに任せた方が楽です。代表的なツールは管理手法で主に 3 種類に分けられます。

1. シンボリックリンク ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
2. コピー ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
3. 直接管理 (Bare Git, [yadm](https://yadm.io/))

それぞれメリデリありますが、怠惰な僕にとっては yadm が好みです。git と全く同じインターフェースで覚えることが少ないし、直接管理するから Single Source of Truth でわかりやすい。テンプレート機能、シークレット管理もしてくれます。

これだけだと設定だけで毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛いのでインストールするとこまで自動化したい！自前でスクリプトを書くと冪等性やクロスプラットフォームにするのに骨が折れるので、ツールを使います。

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Nix](https://github.com/NixOS/nix), [Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

個人的には構成管理ツールで手続き的に管理するよりパッケージマネージャーで宣言的にやった方が認知負荷が低いので後者がおすすめです。すべてを宣言的に管理しておいてコマンド 1 発で確実に起動する安心感はすごいです。

単なる設定ではなく構成主義


動かない原因の多くは暗黙的な依存関係です。たしかに、動的ライブラリ (OpenSSL, zlib など) やヘッダファイルがなかったり、Python や cmake などのバージョンが違ったり、環境変数が足りないことでビルドができなかったり実行できなかったりします。

そうではなく動く最小の環境を作って、すべての依存関係やビルドフラグを明示的に書いておき、それをそのまま持ってくれば動くことが保証されるよねというのが Nix の基本アイデアです。

Nix は依存解決をしないので dependency hell が起きなかったり、バージョン更新時は古い世代をストアに残して即座にロールバックできるようになっています。しかしこれらは裏を返せばバージョンを擦り合わせてストレージ最適化がされなかったり、定期的に古い世代を削除するガベージコレクションがなされるということです。そのためストレージ書き込み量は多いので SSD の寿命や容量が気になる環境にはあまり向かないかもしれないです。

https://syu-m-5151.hatenablog.com/entry/2025/12/18/111500

管理する対象によって名前がいろいろ変わるんですが OS 自体なら NixOS

そしてこれからは LLM による構成のフィードバックループです。すべてのマシンの挙動は宣言的に書かれ、それを LLM が管理し、バグが起きたり日々のアップデートや改善すべき点が見つかれば修正し、デプロイされるというのが全自動で行われ、人はもっと本質的な部分に携われると考えています。

今回はその前段階としてさまざまな記事やリポジトリをリサーチして、それを参考に自分の環境において改善すべき点を洗い出し、提案してくれる、そんなものを作ります。

しかし素の Nix では LLM を活かせません。最大限活かす為の構造を作ったので

宣言的
再現可能
段階的に採用できる

相性が良いという意味で「おいしい」をいれたい

AI がたくさんのツールを日々作っていて管理しきれなくなってる

## やり方

これをミニマルに設定してマシン間で同期して他の人の良い設定をすぐに取り込めて後で見返してもなぜこの設定があるかに悩まなくて良いようなものがほしい。そして普段使っていてバグらなくてパフォーマンスが上がり生産性が向上する。これを目指して作っていきます。

生産性向上 = 人の負担を徹底的に減らす環境を作るためにはどうすべきか？

これらを統一的に管理する方法を考えてみると、「複数のマシンを束ねるレイヤー」と「各マシンの詳細を記述するレイヤー」に分けるときれいに整理できそうです。これはクラスター構成の manifest と dotfiles という 2 つのリポジトリと対応していて、じゃもうすでに完成されてるじゃん！

...のように思いますが、この視点で僕の dotfiles は設定を見返すと、網羅できてない上にインストールしてくれなくて私用 PC に密結合していて汎用性も可読性も再現性も低く、到底「マシンを記述したもの」とは言えない状態でした。

## 結論構成

1 コマンドでどんな OS でもどんなユーザーでもセットアップが完了し、情報を流し込むだけで生産性が向上する仕組みを実現する。さらに機密情報は外部に逃がしたい。

### 膨大な設定を管理する

膨大な設定を管理するためには責務を分離しないといけません。ユーザー、テーマ、システム、ツールで責務を分離します。

```
dotfiles/
├── flake.nix           # Flake entry point
├── setup               # Bootstrap script (Unix/Windows polyglot)
├── system/
│   ├── shared.nix      # User, versions, nix settings, helpers
│   ├── home-manager/
│   │   ├── builder.nix # mkStandaloneHome, mkSystemHomeConfig
│   │   └── core.nix    # Base settings, platform detection
│   ├── darwin/         # nix-darwin, Homebrew
│   ├── nixos/          # NixOS modules
│   └── windows/        # winget, Windows Terminal, wsl.conf
├── ai/                 # Claude Code, Aider, Ollama
├── shell/              # zsh, fish, bash, starship, atuin, zoxide
├── editor/             # neovim (nixvim), vscode, zed
├── tools/              # git, lazygit, yazi, CLI tools (rg, fd, bat...)
├── terminal/           # ghostty, zellij, tmux
├── dev/                # mise, rust, go, python
├── desktop/            # niri, fuzzel, swaync, IME
├── security/           # 1password, ssh, gpg, gitleaks
├── theme/              # Stylix: Catppuccin Mocha
└── users/              # Per-user settings (setup generates)
```

flake.nix が全体のエントリポイントです。外部依存（nixpkgs, home-manager, nixvim 等）を宣言し、wsl, mac, nixos-desktop といったプラットフォームごとの構成を定義します。各構成は 共通モジュール + プラットフォーム固有モジュール の組み合わせで作られます。

以前は `home/` ディレクトリの中にモジュールを入れていましたが、階層が深くなりすぎるのでルートにフラット化しました。`system/home-manager/` には Home Manager のビルダー関数とコアモジュールを置いて、darwin/nixos の builder.nix と対称的な構造にしています。

`system/shared.nix` には共通ヘルパーがあって、`mkNixConfig` で darwin/nixos 両方の nix 設定を生成、`basePackages` と `desktopFonts` で共通パッケージ・フォントを定義しています。重複を減らして変更箇所を一箇所に集約するためです。

theme/ は Stylix でカラー・フォント・カーソルを一括管理し、ターミナルからエディタまで統一されたデザインを適用します。users/ は setup スクリプトが生成するユーザー固有の設定（git の名前・email 等）です。

workstation フラグで GUI ツールの有無を切り替えられます。`workstation = true`（デフォルト）だと ai/ + tools/ + editor/ + terminal/ が含まれ、`workstation = false` だとサーバー向けのミニマル構成になります。

system/ は OS レベルの設定を担当します。NixOS と macOS は Nix で管理しますが、Windows は Nix が対応していないため WinGet の JSON で宣言的に管理しています。

| | Linux | NixOS | macOS | Windows |
|---|---|---|---|---|
| インストール | なし | NixOS | nix-darwin | WinGet |
| 設定 | Home Manager | Home Manager | Home Manager | Home Manager (from WSL) |

### セットアップ

どんな OS でも 1 コマンドで初期状態からセットアップ完了まで動かせるようにしたいので polyglot という黒魔術を用いて実現しました。これは複数の言語で実行できるようなプログラムのことで Bash と PowerShell どっちで実行しても動くプログラムです。具体的にはこんな感じです。

```bash
echo --% > /dev/null ; : ' | out-null
<#'
# Bash
exit 0
#>
# PowerShell
```

PowerShell は `<# ... #>` がコメントとして読み取るので前半は Bash、後半が PowerShell として読まれます。ただし `<#` を Bash がパースできないので 1 行目は `'...'` を用いてこれをコメントにしつつ、それ自体を `--%` で PowerShell がそれを解析しないようにするという形で実現されています。

これで Linux, macOS, Windows 全部で動くプログラムになります。

```bash
# macOS / Linux / WSL
curl -fsSL https://raw.githubusercontent.com/anko9801/dotfiles/master/setup | sh

# Windows (PowerShell as Admin)
iwr https://raw.githubusercontent.com/anko9801/dotfiles/master/setup | iex
```

このセットアップでは公式ではなく UX のいい Determinate Systems の Nix インストーラーを使って Nix を入れ、`~/dotfiles` をクローンして `users/$USER.nix` にユーザー情報を書き込んで適用します。バイナリキャッシュが効くようになっています。

### セキュリティ

例えば AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...みたいな想定しますよね。

こういうのは雑に管理したり気軽にセルフホストすると脆弱性を埋め込むので外部サービスに全部置きましょう。僕は 1Password が好きですべての機密情報をここで管理してます。E2EE でユーザー以外は暗号化されておりサーバー管理者にも復号できないようになっているので、サーバーが攻撃されても解読できず事実上安全。ただし E2EE だからといって完全に信用はできなくて、バックドアを仕込まれてる可能性もあるっちゃあるので法律が機能する法人に頼みましょう。

1Password のおすすめ機能はこちら。

- SSH エージェント: `~/.1password/agent.sock` 経由で GitHub/GitLab に接続。秘密鍵はディスクに書かず Touch ID や Windows Hello の生体認証で呼び出す
- API キー: env ファイルに `OPENAI_API_KEY=op://Personal/OpenAI/credential` みたいに書くと op コマンドで起動時に展開してくれる
- Git コミット署名: SSH 鍵で署名していて秘密鍵がディスクに存在しない。1Password がロック中は署名できない

1Password CLI は WSL では Windows 側の `op.exe` を使っていて、シェル側では op プラグインを遅延ロードして gh / aws / gcloud / az を初回実行時に初期化しています。漏洩防止として pre-commit に gitleaks を入れていて、`op://` の参照文字列は誤検知しないよう許可リスト化してます。


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
