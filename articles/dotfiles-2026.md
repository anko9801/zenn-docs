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

dotfiles がこれまでの形でいいのか？よりよい管理は存在しないのか？

こういうことを考える為により広くマシンを管理する方法について考えましょう。年々管理するマシンは増えてきました。

- 私用 PC、社用 PC、ラズパイ
- コンテナ、Kubernetes
- クラウドリソース、外部サービス

これらすべてを網羅的に管理する為にはどうすべきかを私なりに考えてみると、複数のマシンを総合的に記述するレイヤーと各マシンの詳細を記述するレイヤーだと上手く切り分けられて便利そう。これはクラスター構成 manifest とマシン設定 dotfiles という 2 つのリポジトリと対応していて、じゃもうすでに完成されてるじゃん！

...のように見えますが、この視点から見ると私の dotfiles は設定を網羅できてない上にインストールしてくれなくて私用 PC に密結合していて汎用性・可読性・再現性の低く、各マシンを記述したものとは到底言えないものになっていました。

これをミニマルに設定してマシン間で同期して他の人の良い設定をすぐに取り込めて後で見返してもなぜこの設定があるかに悩まなくて良いようなものがほしい。そして普段使っていてバグらなくてパフォーマンスが上がり生産性が向上する。これを目指して作っていきます。


## これまでの管理方法

まず .vimrc や .zshrc とかの設定ファイルをコピペして git 管理するとこから考えると

- git 管理してるから昔の設定に戻しやすい
- 設定を共有して参考にできる
- 設定を適用するにはコピペやリンクしないとでめんどくさい
- インストールもやらないといけない

それじゃ適用を自動化しよう！自分でスクリプトを書くと既存の設定とコンフリクトしたときに正しく適用できるかな、設定が増えたらスクリプトも書き足さないと、 Windows のリンクの仕様がトリッキーだったりして考えることが多いのでツールに任せた方が楽です。代表的なツールは管理手法で主に 3 種類に分けられます。

1. シンボリックリンク ([Dotbot](https://github.com/anishathalye/dotbot), [GNU Stow](https://www.gnu.org/software/stow/), [Mackup](https://github.com/lra/mackup), [rcm](https://github.com/thoughtbot/rcm))
2. コピー ([chezmoi](https://github.com/twpayne/chezmoi), [dotdrop](https://github.com/deadc0de6/dotdrop), [dotter](https://github.com/SuperCuber/dotter))
3. 直接管理 (Bare Git, [yadm](https://yadm.io/))

ここらへんの管理手法はメリデリ色々ありますが、怠惰な僕にとっては yadm が一番好きです。git と全く同じインターフェースで覚えるのが少ないし、直接管理するから Single Source of Truth でわかりやすいし楽ちん、さらにはテンプレート機能、シークレット管理までしてくれます。

毎回ちまちまインストール方法調べてコマンド叩いて間違えたら修正を繰り返すのは辛い、さらにインストールするとこまで自動化したい！それなら自前でスクリプトを書くと冪等性やクロスプラットフォームにするのに骨が折れるので主に次のツールを使います。

- 構成管理ツール ([Ansible](https://github.com/ansible/ansible), [mitamae](https://github.com/itamae-kitchen/mitamae))
- パッケージマネージャー ([Nix](https://github.com/NixOS/nix), [Homebrew](https://brew.sh/ja/), [WinGet](https://github.com/microsoft/winget-cli))

これについては構成管理ツールで手続き的に管理するよりパッケージマネージャーで宣言的にやった方が認知負荷が低いので後者がおすすめです。すべてを宣言的に管理しておいてコマンド 1 発で確実に起動する安心感はすごいです。

そして最近は LLM を用いた dotfiles の管理が主流になってきました。LLM に指示して機能を追加するという形です。それをさらに押し進めて設定をよりよくするところさえ自動化したい！大量に生産性向上記事を読み込ませて適用！をしたい。

それが Nix をベースにしたスキルと構造で実現したというのがこの記事です。

じゃあデメリットないの？というとストレージ周りがちょっと気になります。Nix はすべてのパッケージをハッシュ付きでストアに保存していて、環境の切り替えはシンボリックリンクの付け替えでやっています。これで古い世代がストアに残り続けるからこそ即座にロールバックできるんですが、裏を返せばガベージコレクションしない限りストレージを食い続けてしまいます。以前はこれを手動でやるのが面倒で敬遠していたんですが、`nix.gc.automatic` で定期的に走らせれば一切気にならなくなりました。ただし世代の蓄積と GC による大量削除を繰り返す性質上、ストレージへの書き込みは多くなるので SSD の寿命や容量が気になる環境にはあまり向かないかもしれないです。

## 結論構成

1 コマンドでどんな OS でもどんなユーザーでもセットアップが完了し、情報を流し込むだけで生産性が向上する仕組みを実現する為に必要な

- ユーザー、テーマ、システム、ツール で責務を分離
- 機密情報は外部に逃がす


### 責務の分離

ベースは Nix を用いて管理していて、ディレクトリ構造はこんな感じです。

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

flake.nix が全体のエントリポイントです。外部依存（nixpkgs, home-manager, nixvim 等）を宣言し、wsl, mac, nixos-desktop といったプラットフォームごとの構成を定義します。各構成は 共通モジュール + プラットフォーム固有モジュール の組み合わせで作られます。

theme/ は Stylix でカラー・フォント・カーソルを一括管理し、ターミナルからエディタまで統一されたデザインを適用します。users/ は setup スクリプトが生成するユーザー固有の設定（git の名前・email 等）です。

home/shell/, home/dev/, home/security/, theme/ はディレクトリにファイルを置くだけで自動読み込みされます（haumea）。一方 home/tools/ と home/editor/ は os-specific/*.nix から明示的に import されます。これにより、サーバー構成から GUI エディタを外す、といった制御ができます。

system/ は OS レベルの設定を担当します。NixOS と macOS は Nix で管理しますが、Windows は Nix が対応していないため WinGet の JSON で宣言的に管理しています。

| | Linux | NixOS | macOS | Windows |
|---|---|---|---|---|
| システム | なし | NixOS | nix-darwin | WinGet + PowerShell |
| ツール | Home Manager | Home Manager | Home Manager | なし |

最後に home マシン共通のツールを読み込みます。


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

認知負荷を下げる為にもツール全体で統一したいものっていくつかあって Stylix と温かみのある手作業でやってます。私の設定だとテーマは [Catppuccin Mocha](https://github.com/catppuccin)、フォントは [Moralerspace](https://github.com/yuru7/moralerspace)、キーリマッパーは [Kanata](https://github.com/jtroo/kanata)、キーバインディングは「目的語 + 動詞」で統一してます。

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

私は ghq より zoxide が好きでディレクトリで自由にカテゴリ分けできるからリポジトリ名を忘れたときに探しやすいし、新しくコマンドを覚えなくていいので助かります。差分は構文解析した上で表示してくれる difftastic、コンフリクトは差分に加えて共通祖先も表示する zdiff3、コミットは絵文字や Conventional Commit がインタラクティブにできる czg を使っています。

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

dotfiles 管理は、Git の素朴さで押し切る yadm と、テンプレートやパスワードマネージャ連携まで面倒を見てくれる chezmoi に分かれていて、マシン差分や秘密情報の扱いをどう吸収するかが評価軸になっています。Ansible はプレイブックとインベントリで構成を整理する王道ですが、最近は「セットアップの入口は軽く、定常運用は宣言的に」の組み合わせが強く、ブートストラップを軽いスクリプトに寄せて、日常は Nix/chezmoi/yadm に寄せる構成が多い印象です。
