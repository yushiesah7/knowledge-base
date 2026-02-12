# tmux 実践レシピ集

> ユースケース別のスクリプト・設定例。コピペして使えるが、カスタマイズ前提。
> コマンド単体のリファレンスは `commands.md`、仕組みの理解は `architecture.md` を参照。

---

## 1. 開発環境を一発構築する

### 基本テンプレート

```bash
#!/bin/bash
SESSION="myapp"

# 既存セッションがあれば接続
tmux has-session -t $SESSION 2>/dev/null
if [ $? -eq 0 ]; then
    tmux attach -t $SESSION
    exit 0
fi

# セッション作成（バックグラウンド）
tmux new-session -s $SESSION -c ~/Projects/myapp -n editor -d

# エディタ起動
tmux send-keys -t $SESSION:editor 'nvim .' C-m

# サーバー用ウィンドウ（上下分割）
tmux new-window -t $SESSION -c ~/Projects/myapp -n server
tmux send-keys -t $SESSION:server 'npm run dev' C-m
tmux split-window -v -t $SESSION:server -c ~/Projects/myapp
tmux send-keys 'npm run watch:css' C-m

# ターミナル用ウィンドウ
tmux new-window -t $SESSION -c ~/Projects/myapp -n terminal

# Git用ウィンドウ
tmux new-window -t $SESSION -c ~/Projects/myapp -n git
tmux send-keys -t $SESSION:git 'git status' C-m

# 最初のウィンドウを選択してアタッチ
tmux select-window -t $SESSION:editor
tmux attach -t $SESSION
```

### 使い方

```bash
chmod +x ~/scripts/tmux-myapp.sh
~/scripts/tmux-myapp.sh
```

---

## 2. 複数プロジェクトを同時に立ち上げる

### ワンライナー

```bash
tmux new-session -s work -c ~/Projects/frontend -n fe -d && \
tmux new-window -t work -c ~/Projects/backend -n be && \
tmux new-window -t work -c ~/Projects/infra -n infra && \
tmux attach -t work
```

### 汎用関数（.zshrc / .bashrc に追加）

```bash
# 使い方: devstart frontend:~/Projects/fe backend:~/Projects/be
devstart() {
    local session="dev-$(date +%H%M)"
    local first=true

    for arg in "$@"; do
        local name="${arg%%:*}"
        local dir="${arg##*:}"
        dir="${dir/#\~/$HOME}"

        if $first; then
            tmux new-session -s "$session" -c "$dir" -n "$name" -d
            first=false
        else
            tmux new-window -t "$session" -c "$dir" -n "$name"
        fi
    done

    tmux select-window -t "$session:0"
    tmux attach -t "$session"
}
```

```bash
devstart frontend:~/Projects/frontend backend:~/Projects/backend db:~/Projects/db
```

---

## 3. モニタリングダッシュボード

サーバー監視やログ閲覧用のレイアウト:

```bash
#!/bin/bash
SESSION="monitor"

tmux new-session -s $SESSION -d -n dashboard

# 4分割レイアウト
tmux send-keys 'htop' C-m
tmux split-window -h
tmux send-keys 'tail -f /var/log/syslog' C-m
tmux split-window -v
tmux send-keys 'docker stats' C-m
tmux select-pane -t 0
tmux split-window -v
tmux send-keys 'watch -n 5 df -h' C-m

tmux attach -t $SESSION
```

---

## 4. SSH先で使い捨てセッション

リモートで作業中、切断されても安全:

```bash
# SSH先で即セッション作成
ssh myserver -t 'tmux new-session -A -s remote'
```

`-A` オプション: 既存セッションがあればアタッチ、なければ新規作成。

---

## 5. ペアプログラミング（セッション共有）

```bash
# ホスト側: セッション作成
tmux new-session -s pair

# ゲスト側: 同じセッションにアタッチ（同じマシンにSSHした前提）
tmux attach -t pair

# 独立したビューが欲しい場合（ウィンドウを別々に操作可能）
tmux new-session -t pair -s pair-guest
```

---

## 6. レイアウトのプリセット

tmux には組み込みのレイアウトがある:

```bash
# ペインを均等に水平分割
tmux select-layout even-horizontal

# ペインを均等に垂直分割
tmux select-layout even-vertical

# メイン画面 + 右に小さいペイン群
tmux select-layout main-vertical

# メイン画面 + 下に小さいペイン群
tmux select-layout main-horizontal

# タイル状に均等配置
tmux select-layout tiled
```

キーバインドでの切替: `Ctrl+b Space` で順番にレイアウトを切り替え。

---

## 7. tmuxinator を使う（YAML宣言的管理）

### インストール

```bash
gem install tmuxinator
```

### プロジェクト設定

```bash
tmuxinator new myapp
```

`~/.tmuxinator/myapp.yml`:

```yaml
name: myapp
root: ~/Projects/myapp

windows:
  - editor:
      panes:
        - nvim .
  - server:
      layout: even-vertical
      panes:
        - npm run dev
        - npm run watch
  - console:
      root: ~/Projects/myapp
      panes:
        - # 空のシェル
  - logs:
      panes:
        - tail -f logs/app.log
```

```bash
# 起動
tmuxinator start myapp

# 停止
tmuxinator stop myapp

# プロジェクト一覧
tmuxinator list

# 設定を編集
tmuxinator edit myapp
```

---

## 8. 便利な .tmux.conf スニペット集

```bash
# --- ペイン分割をわかりやすいキーに ---
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# --- 新しいウィンドウを現在のディレクトリで開く ---
bind c new-window -c "#{pane_current_path}"

# --- vim風ペイン移動 ---
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# --- ペインリサイズ ---
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# --- ペインの同期（全ペインに同じ入力）---
bind S setw synchronize-panes

# --- 設定リロード ---
bind r source-file ~/.tmux.conf \; display "Reloaded!"
```

---

## 9. 自動起動（ターミナル起動時にtmux）

`.zshrc` / `.bashrc` の末尾に追加:

```bash
# ターミナル起動時に自動でtmuxセッションに入る
if command -v tmux &> /dev/null && [ -z "$TMUX" ]; then
    tmux new-session -A -s main
fi
```

---

## 10. セッションの保存と復元（tmux-resurrect）

tmux のセッションは再起動で消える。プラグインで永続化:

### インストール（tpm経由）

`~/.tmux.conf`:

```bash
# tpm（プラグインマネージャ）
set -g @plugin 'tmux-plugins/tpm'

# セッション保存・復元
set -g @plugin 'tmux-plugins/tmux-resurrect'

# 自動保存（オプション）
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @continuum-restore 'on'

# tpm初期化（.tmux.confの最後に書く）
run '~/.tmux/plugins/tpm/tpm'
```

```bash
# tpmインストール
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# プラグインインストール（tmux内で）
Ctrl+b I
```

### 使い方

| 操作 | キー |
|---|---|
| セッション保存 | `Ctrl+b Ctrl+s` |
| セッション復元 | `Ctrl+b Ctrl+r` |
