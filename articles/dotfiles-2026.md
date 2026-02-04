---
title: "今一度 dotfiles を考え直そう 2026"
emoji: "🦔"
type: "tech"
topics: []
published: false
---

みなさん！dotfiles 作ってますか？

ｳｵｰ!! こだわりの dotfiles がたくさん見えた気がします。

今回は純粋関数型パッケージマネージャ Nix をベースに管理しやすい構造を作った記録になります。dotfiles 作ったことない人にとっても使いやすい構成にしてみたので始めてみたかったよって人は是非参考に盆栽を始めてみてください。

## マシンをどう管理すべきか

まずは dotfiles がこれまでの形でいいのか？よりよい管理は存在しないのか？

こういうことを考える為により広くマシンを管理する方法について考えましょう。年々管理するマシンは増えてきました。

- 私用 PC、社用 PC、ラズパイ
- コンテナ、Kubernetes
- クラウドリソース、外部サービス

これらすべてを網羅的に管理する為にはどうすべきかを私なりに考えてみると、複数のマシンを総合的に記述するレイヤーと各マシンの詳細を記述するレイヤーだと上手く切り分けられて便利そう。これはクラスター構成 manifest とマシン設定 dotfiles という 2 つのリポジトリと対応していて、じゃもうすでに完成されてるじゃん！...のように見えますがこの視点から見ると私の dotfiles は設定を網羅できてない上にインストールしてくれなくて私用 PC に密結合していて汎用性・可読性・再現性の低いものになっていました。

これをミニマルに設定してマシン間で同期して他の人の良い設定をすぐに取り込めて後で見返してもなぜこの設定があるかに悩まなくて良いようなものがほしい。そして普段使っていてバグらなくてパフォーマンスが上がり生産性が向上する。これを目指して作っていきます。


## これまでの管理方法

まず .vimrc や .zshrc とかの設定ファイルをコピペして git 管理するとこから考えると

- git 管理してるから昔の設定に戻しやすい
- 設定を共有して参考にできる
- 設定を適用するにはコピペやリンクしないとでめんどくさい
- インストールもやらないといけない

それじゃ適用を自動化しよう！ってなって自分でスクリプトを書くと既存の設定とコンフリクトしたときに正しく適用できるかな、設定が増えたらスクリプトも書き足さないと、 Windows のリンクの仕様がトリッキーだったりして考えることが多いのでツールに任せた方が楽です。代表的なツールは管理手法で主に 3 種類に分けられます。

1. シンボリックリンク ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
2. コピー ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
3. 直接管理 (Bare Git, [yadm](https://yadm.io/))

ここらへんの管理手法はメリデリ色々ありますが、怠惰な僕にとっては yadm が一番好きです。git と全く同じインターフェースで覚えるのが少ないし、直接管理するから Single Source of Truth でわかりやすいし楽ちん、さらにはシークレット管理までしてくれます。

さらにインストールするとこまで自動化したい！ってなると

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Nix](https://github.com/NixOS/nix), [Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

これについては構成管理ツールで手続き的に管理するよりかはパッケージマネージャーで宣言的な管理をした方が認知負荷が低いので後者がおすすめです。

今回は Nix を採用しました。毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すより、すべてを宣言的に管理しておいてコマンド 1 発で確実に起動する安心感はすごいです。ただ Nix にもデメリットもあってそれについてはあとでお伝えします。

## 結論構成

ベースは Nix を用いて管理しています。例外的に Windows は Nix が対応してないので WinGet を用いて宣言的に管理しています。

| | Linux | NixOS | macOS | Windows |
|---|---|---|---|---|
| システム | なし | NixOS | nix-darwin | WinGet + PowerShell |
| ツール | Home Manager | Home Manager | Home Manager | なし |

ファイル構造はこんな感じです。

```
dotfiles/
├── flake.nix           # Flake entry point
├── setup               # Bootstrap script (Unix/Windows polyglot)
├── system/             # System-level config
│   ├── darwin/         # nix-darwin, Homebrew
│   ├── nixos/          # NixOS modules
│   └── windows/        # winget, Windows Terminal, wsl.conf
├── home/               # Home Manager modules
│   ├── shell/          # zsh, starship, fzf, atuin, zoxide
│   ├── editor/         # neovim (nixvim), vscode, zed
│   ├── tools/          # git, lazygit, ghostty, CLI tools (rg, fd, jq...)
│   ├── dev/            # mise, language toolchains
│   ├── desktop/        # niri, fuzzel, swaync (Linux Wayland)
│   ├── security/       # 1password, ssh, gitleaks
│   └── os-specific/    # wsl, darwin, linux
├── theme/              # Stylix: colors, fonts, cursor
└── users/              # Per-user settings (setup generates)
```

これまで Nix は便利だけどガベージコレクションがあまりにひどい機能でそれだけで避けていたのですが、ガベコレを自動的にしてくれるデーモンが付けられるのに気が付いてからはもう虜です。宣言的に書かれてあるから LLM とも相性がよくて、大量に生産性向上記事を読み込ませて適用！みたいなことが楽にできます。

ガベコレを回すとよくストレージの書き換えが頻繁に起こるので SSD を長持ちさせたい方はあまり向いてないかもしれません。

mise devenv


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


## おまけ: ツールへのこだわり

生産性向上とは認知負荷の削減。認知負荷というのはタスクに対して短期記憶から払い出されるメモリのことで、これが少ないほど本質をより深く考えられるようになります。例えば関数型だったり宣言的とかが認知負荷の削減になります。移植性のある少数精鋭。ツールの選定は次の順でやってます。

1. シンプル (Simple is not Easy)
2. 不具合が少ない・ユーザーが多い
3. クロスプラットフォーム
4. 高パフォーマンス
5. セキュリティ

他の方と大体同じなのでサクサクいきます。

### 日用系ツール

ターミナルは Ghostty と Windows Terminal を使っています。Wezterm を作り込みたいな～と思って放置してます。ターミナルマルチプレクサはターミナルのデフォルト使うより選択コピーや prefix key のキーバインドができて便利なので Zellij を使ってます。

シェルは zsh を使っています。fish や Nushell などの非 POSIX 準拠シェルだと書き換えが必要だったり LLM で間違うことが多くて使いづらいのと、今や zsh でも補完などの体験が良くて困ってないです。起動時間は 200ms 以下だと体感が変わらなくなるので 200ms になるように遅延読み込みを調整しています。プラグインについては次を使っています。

| zsh プラグイン | 説明 |
|---|---|
| fzf-tab | いろんなコマンドで fzf を設定する |
| zsh-abbr | スペースを押すと展開されるエイリアス |
| zsh-autosuggestions | 補完 |
| zsh-defer | 遅延読み込み |
| zsh-fast-syntax-highlighting | シンタックスハイライト |

プロンプトは Starship、IME は SKK、ランチャーは Raycast

エディターはデスクトップなら VSCode、ノパソなら Zed、気分で Neovim です。やっぱりノパソで VSCode fork app 動かしてると重い！動かん！って思うことが多いです。

### カラースキーム & フォント & キーバインディング

認知負荷を下げる為にもツール全体で統一したいものっていくつかあって Stylix と温かみのある手作業でやってます。私の設定だとテーマは [Catppuccin Mocha](https://github.com/catppuccin)、フォントは [Moralerspace](https://github.com/yuru7/moralerspace)、キーリマッパーは [Kanata](https://github.com/jtroo/kanata)、キーバインディングは「目的語 + 動詞」で統一してます。

現代において Ctrl, Alt, Win/Cmd の 3 種類のキーがあり、目的語であるアプリケーションについてウィンドウマネージャーは Win/Cmd、ペインは Ctrl-q、タブは Ctrl-t、シェルは Ctrl-g、Vim は Space みたいに対応しています。何をするかは上下左右に動かすときは hjkl、リサイズは HJKL、縦横分割は |-、閉じるは x、ズームは z、新規は n みたいな感じでやってます。

こうすると覚えることが少なくてめっちゃ楽です。ただしマウスでやるよりキーボードでやった方が基本速いですが、キーボードに偏りすぎると認知負荷が高くなって疲れてしまうので人それぞれの良いバランスでやるといいかなと思ってます。本当は英語の文法順にあわせて動詞 + 目的語にして認知負荷減らしたいですがむずい。あと目的語の方の名前空間が枯渇しがちなのでリマップをもっと凝ったり自作キーボードもこだわりたいなとも思ってます。

Vim/Emacs は Esc が Tab の位置だったり、Meta Super Hyper などのキーがあったりした 1970 年代のキーボードを元に設計されてるので、RSI 予防に CapsLock をタップで Esc、長押しで Ctrl にリマップしたり、できるだけ Ctrl に近い Q キーなどで組んだりしてます。ゲームと作業を切り替えがちなので Home Row Mod とかは相性が悪い感じです。NeoVim だと which-key.nvim があるとキーが覚えやすくて便利です。zsh-abbr もモダンツールを覚える為に使うのもいいです。


### Git

私は ghq より zoxide が好きでディレクトリで自由にカテゴリ分けできる分リポジトリ名を忘れたときに探しやすいし、新しく覚えなくていいので助かります。差分は構文解析した上で表示してくれる difftastic、コンフリクトは差分に加えて共通祖先も表示する zdiff3、コミットは絵文字や Conventional Commit がインタラクティブにできる czg か Claude Code、TUI は機能が豊富な lazygit を使ってます。

ジジイが気に入ってるエイリアス 5 選

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

pre-commit に gitleaks を設定してシークレット漏洩を防ぎ、GitHub の noreply アドレスを使ってコミットにアドレスを残さないようにしています。

jujutsu 使えたらいいな。


### セキュリティ

例えば AWS の API キーとか SSH の秘密鍵とかを管理するとき、いつも gitignore でやってるけど万が一 push しちゃったら...とか社用 PC がウイルスに感染したら...みたいな想定しますよね。

こういうのは雑に管理したり気軽にセルフホストすると脆弱性を埋め込むので外部サービスに全部置きましょう。私は 1Password が好きですべての機密情報をここで管理しています。このサービスは E2EE というユーザー以外は暗号化されておりサーバー管理者にも復号できないようになっていて、サーバーを攻撃されても解読できず事実上安全という訳です。注意ですが E2EE だからといって信用はできなくて、バックドアを仕込んで機密情報を奪ってる可能性もあるっちゃあるので法律が機能する法人に頼みましょう。

1Password で管理しているのは Git コミット署名、SSH エージェント、API キーです。Git コミット署名は SSH 鍵で署名していて秘密鍵がディスクに存在せず 1Password がロック中は署名できないようになっています。SSH エージェントも `~/.1password/agent.sock` 経由で GitHub/GitLab へ接続していて秘密鍵はディスクに書かず生体認証で呼び出します。例えば API キーは env ファイルに `OPENAI_API_KEY=op://Personal/OpenAI/credential` みたいに書くと op コマンドを使って起動時に展開してくれます。生体認証は Touch ID や Windows Hello とかを使ってます。


### LLM

Claude Code のみを使ってます。インターフェースはちょっと悩んでいてキーボードの他に音声操作を使っていて割と適当に指示しても動いてくれます。しかしなにか別の作業をしているときにお口がフリーなのでお菓子で忙しくないときは自分の作業に干渉しない音声操作がすごく欲しいです。

### その他

[XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/latest/)
OSC 52

## まとめ
