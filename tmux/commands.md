# tmux コマンドリファレンス（コピペ用）

> コピペですぐ使えることを目的としたリファレンス。
> 仕組みの理解は `architecture.md`、実践的なスクリプトは `recipes.md` を参照。

---

## セッション操作

```bash
# 新規セッション作成
tmux new-session -s mysession

# 指定ディレクトリでセッション作成
tmux new-session -s mysession -c ~/Projects/myapp

# バックグラウンドでセッション作成（アタッチしない）
tmux new-session -s mysession -d

# セッション一覧
tmux ls

# セッションにアタッチ
tmux attach -t mysession

# セッションからデタッチ（セッション内で）
Ctrl+b d

# セッション名を変更
tmux rename-session -t old_name new_name

# セッション削除
tmux kill-session -t mysession

# 全セッション削除
tmux kill-server
```

---

## ウィンドウ操作

```bash
# 同じセッションにウィンドウ追加
tmux new-window -t mysession -n window_name

# 指定ディレクトリでウィンドウ追加
tmux new-window -t mysession -c ~/Projects/backend -n backend

# ウィンドウ名を変更
tmux rename-window -t mysession:0 new_name
```

### ウィンドウ操作（キーバインド / セッション内で使用）

| 操作 | キー |
|---|---|
| 新規ウィンドウ | `Ctrl+b c` |
| 次のウィンドウ | `Ctrl+b n` |
| 前のウィンドウ | `Ctrl+b p` |
| 番号指定で切替 | `Ctrl+b 0`〜`9` |
| ウィンドウ一覧 | `Ctrl+b w` |
| ウィンドウ名変更 | `Ctrl+b ,` |
| ウィンドウ閉じる | `Ctrl+b &` |

---

## ペイン操作

```bash
# 水平分割（左右）
tmux split-window -h -t mysession

# 垂直分割（上下）
tmux split-window -v -t mysession

# 指定ディレクトリで分割
tmux split-window -h -t mysession -c ~/Projects/api

# 特定ペインにコマンドを送信
tmux send-keys -t mysession:0.1 'npm run dev' C-m
```

### ペイン操作（キーバインド / セッション内で使用）

| 操作 | キー |
|---|---|
| 水平分割（左右） | `Ctrl+b %` |
| 垂直分割（上下） | `Ctrl+b "` |
| ペイン移動 | `Ctrl+b 矢印キー` |
| ペインサイズ変更 | `Ctrl+b Ctrl+矢印キー` |
| ペインの配置切替 | `Ctrl+b Space` |
| ペイン閉じる | `Ctrl+b x` |
| ペインを別ウィンドウに分離 | `Ctrl+b !` |
| ペイン番号表示 | `Ctrl+b q` |
| ペインをズーム（全画面⇔元に戻す） | `Ctrl+b z` |

---

## セッション切り替え・検索

```bash
# セッション切り替え（キーバインド）
Ctrl+b s          # セッション一覧から選択
Ctrl+b (          # 前のセッション
Ctrl+b )          # 次のセッション
```

---

## コマンドモード

```bash
# tmux内でコマンドモードを開く
Ctrl+b :

# コマンドモードの例
:new-window -n test
:split-window -h
:resize-pane -R 10
```

---

## コピーモード（スクロール・テキスト選択）

| 操作 | キー |
|---|---|
| コピーモード開始 | `Ctrl+b [` |
| スクロール上 | `↑` / `PgUp` |
| スクロール下 | `↓` / `PgDn` |
| 選択開始 | `Space` |
| コピー（選択確定） | `Enter` |
| ペースト | `Ctrl+b ]` |
| コピーモード終了 | `q` |

---

## ターゲット指定の書式

コマンドで特定の場所を指す際のフォーマット:

```
セッション名:ウィンドウ番号.ペイン番号

例:
mysession:0      → mysessionの最初のウィンドウ
mysession:2.1    → mysessionの3番目ウィンドウの2番目ペイン
mysession:editor → mysessionの "editor" という名前のウィンドウ
```

---

## オプション設定（即時反映）

```bash
# マウス操作を有効にする
tmux set -g mouse on

# ステータスバーを上に
tmux set -g status-position top

# ベースインデックスを1にする（0始まりをやめる）
tmux set -g base-index 1
tmux set -g pane-base-index 1

# ヒストリ上限を増やす
tmux set -g history-limit 50000
```

---

## その他便利コマンド

```bash
# 現在のキーバインド一覧
tmux list-keys

# 環境変数一覧
tmux show-environment

# 設定の再読み込み
tmux source-file ~/.tmux.conf

# セッションの情報表示
tmux display-message -p '#S:#I.#P'
# → セッション名:ウィンドウ番号.ペイン番号
```
