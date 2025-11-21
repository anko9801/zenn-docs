
# 今一度 dotfiles を考え直そう 2025

色んなことを調べて記事を書いて読んでもらうより AI に聞いたり、海外の有名な記事を翻訳したもので十分な気がしていて、最近まで記事を書く必要がなくなったように感じていたのですが、抽象的なことはあまり慣れていないようでその部分を重点的に記事にしようと思いました。

https://github.com/anko9801/dotfiles

今のベストプラクティスはAIに聞きましょう。これからどうあるべきかを考えて実装すべき。

## 認知負荷理論


### 

chezmoi はむずかしい
yadm

### 大量の

皆さんのホームディレクトリはこんな感じじゃないですか？

```
workspace/
.aws/
... (60 files)
.vimrc
```

整理しましょう。

```
workspace/
.claude/
.config/
.ssh/
```

XDG Base Directory というものを使うことで .config の中に収納することができます。

```bash:/etc/zsh/zshenv
```

- **シェルも収納** - `/etc/zsh/zshenv` で XDG Base Directory 準拠することで .zshrc も .config に収納

### 1Password

セキュリティ

- **1Passwordで認証管理** - SSH 鍵、GPG 鍵、API トークンを安全に管理 - [op/README.md](../.config/op/README.md)

### yadm

思想を大きく 3 種類に分けると

- シンボリックリンクを貼る (GNU Stow like)
    - 新しく管理すべきものが増えた時貼るスクリプト書かないといけない
- コピーする (chezmoi)
    - Single Source of Truth じゃなくてあんまし直感的じゃない
- ホームディレクトリを git で管理 (Bare Git like)
    - 導入時のマージが面倒
    - 他と比べて管理が楽

これらを総合的に見たとき導入コストを増やして辛さがスケールしない Bare Git の思想が最も良いと考えてます。
- **yadm によるシンプルな管理** - [なぜyadm？](../.config/yadm/README.md)
#### My best tool: yadm
これまで shellscript (GNU Stow like) → Ansible → chezmoi → yadm と使ってきました。

冪等性の担保に惚れて Ansible 使ってたけど、細かいロジックを組めなかったり、欲しい role がなくて結局シェルを書いたり辛さが見えて、技術を扱っている僕らにとって冪等性は常識だから僕らが担保すればいいじゃんね〜と思いました。

chezmoi は上に書いた通り複雑になるのでやめました。あとファイル名を . → dot_ と置換されるのもしっくりこない (消した方がまし) 。

Nix はストレージの使用量がえぐくて定期的にガベコレしないといけないので辛い

yadm は Bare Git の思想の全ての欠点を解決させたツールです。1Password の連携がなかったので自分で書きました。初期導入でマージのシステムがあるともっとうれしい。あと VSCode は git 管理下のワークスペース全体を探索し始めるのでそこには少し注意が必要かも。

ただこれからは AI と対話して環境を整えてもらって chezmoi で管理してもらうみたいな形になるのかな。たとえば「Web 開発用に M2 MacBook をセットアップします。Neovim、fish、Alacritty を使用します。職場用と個人用の SSH キーが必要で、GitHub トークンは 1Password から取得してください。」みたいなプロンプト投げて理想的な環境を整えることができるようになると思う。そうなると dotfiles 自体必要なくなる未来がすぐそこかも〜

### dotfiles に何を求めるか？

新しいマシンでもコマンド一発でいつもの環境を再現したい。理想としてはこんな感じ:

- らくちん - 1コマンドでセットアップ&管理
- ミニマル - .config にすべて収納
- シークレット管理 - SSH 鍵や API キーはパスワードマネージャーで管理、ローカルには置かない
- マシン差分を吸収 - OS (macOS/Linux/WSL/Windows) やコンテキスト (work/personal) に応じた設定
- 再現性 - 必要なツールはすべて宣言的に管理、どこでも同じ環境を構築
- 冪等性 - 何度実行しても既存の設定を壊さない


最も条件を満たすのは Nix かと思いますがガベコレの運用コストが高いので断念。
実装の工夫で冪等性を満たして、宣言的で再現可能にしてみました。


### こだわりポイント

- **冪等性** - [冪等なシェルスクリプトのベストプラクティス](../.config/yadm/README.md)
- **宣言的なパッケージ管理** - [packages.yaml](../.config/packages.yaml) と [mise](../.config/mise/config.toml) で全プラットフォーム統一管理
- **顔文字プロンプト** - いつもは `╰─(*'-') ❯` エラーが出たら `╰─(*;-;) ❯`

環境を素早くセットアップする為によく作るあれです。理想の dotfiles 管理システムは次のようなものです。

- 1 コマンドでセットアップ、管理もシンプル
- マシン差分を吸収
- シークレットは暗号化または 1Password で管理
- 冪等性








Warp は考えられないほどの遅さをしてた


### ターミナルに何を求めるか？

- カスタマイズ可能
- 高いパフォーマンス

Ghostty はそれを満たし
- ゼロコンフィグ

## Git
delta, gitui, gibo, git-lfs, gitleaks
毎日叩くコマンドは
コマンドを
gitui を軽視しがちですが、ターミナルから VSCode に移動してボタンぽちぽちするより、ターミナルから直接操作した方が効率が上がります。
czg でコミット

```bash
# 日常フロー
git f                                          # fetch --prune（最新情報取得）
git pl                                         # pull（ff-onlyで安全）
git sw -c feature/new                          # ブランチ作成
gitui                                          # 作業しながら差分確認
czg                                            # コミット作成
git ps                                         # push（autoSetupRemote有効）
gh pr create                                   # プルリクエスト作成

# プロジェクト管理
gh repo create my-project --private --clone    # 新規作成
ghq get github.com/user/repo                   # クローン
cd $(ghq list --full-path | fzf)               # 移動（zoxideでcd拡張）
gibo dump Python Node > .gitignore             # .gitignore生成
git open                                       # ブラウザで開く（gh browse）

# 状態確認
git st                                         # status -sb
git diff                                       # 差分（delta、side-by-side）
git staged                                     # ステージング済み差分
git lg                                         # ログ（グラフ）
git la                                         # 全ブランチログ

# コミット操作
czg                                            # 規約に沿ったコミット作成
git amend                                      # --amend --no-edit
git undo                                       # reset HEAD~1 --mixed
git unstage <file>                             # reset HEAD --

# ブランチ・リモート
git sw <branch>                                # switch
git sw -                                       # 直前のブランチ
git cleanup                                    # マージ済み削除
git please                                     # --force-with-lease

# 履歴の編集・取り込み
git rebase -i HEAD~3                           # 対話的リベース（autostash有効）
git cherry-pick <commit>                       # 特定コミット取り込み
git stash -u                                   # 未追跡ファイルも含む
git stash pop                                  # 復元（rebase時は自動）
git df                                         # diff --color-words
```

## 2025 年の Git


## Tools

- Shell: zsh + sheldon
- Multiplexer: tmux + TPM
- Editor: Vim + dpp.vim, Neovim + lazy.nvim
- dev env: mise (Node.js, Go, Rust, Ruby, Deno, Bun)
- Python: uv, ruff
- Modern CLI: bat, eza, ripgrep, fd, fzf, gh, starship, zoxide, atuin, mcfly, duf, dust, tokei, sd, bottom, gomi
- AI Assistant: Claude Code, Gemini CLI


macOS Specific:
- Window Manager: yabai + skhd
- Launcher: Raycast