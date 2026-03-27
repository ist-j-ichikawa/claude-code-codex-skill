# Codex CLI Reference (v0.117.0)

> このファイルはスキル実行時に不明な点がある場合のみ参照する。
> 公式ドキュメント: https://developers.openai.com/codex
> CLI リファレンス: https://developers.openai.com/codex/cli/reference
> 非対話モード: https://developers.openai.com/codex/noninteractive
> 設定リファレンス: https://developers.openai.com/codex/config-reference
> ソースコード: https://github.com/openai/codex/tree/main/codex-rs
> 最終更新: 2026-03-27（v0.117.0 で確認済み）

## 目次

1. [コマンド一覧](#コマンド一覧)
2. [codex exec の全フラグ](#codex-exec-の全フラグ)
3. [共通フラグ](#共通フラグ)
4. [Web検索の設定](#web検索の設定)
5. [Sandbox モード](#sandbox-モード)
6. [config.toml の主要フィールド](#configtoml-の主要フィールド)
7. [プロファイル](#プロファイル)
8. [モデル一覧（v0.117.0 時点の公開モデル）](#モデル一覧v01170-時点の公開モデル)
9. [推論レベル](#推論レベル)

---

## コマンド一覧

| コマンド | エイリアス | 説明 |
|---|---|---|
| `codex` (引数なし) | - | 対話モード (TUI) |
| `codex exec` | `codex e` | ヘッドレス（非対話）実行 |
| `codex exec review` | - | ヘッドレスでコードレビュー |
| `codex exec resume` | - | ヘッドレスでセッション再開 |
| `codex review` | - | 対話モードでコードレビュー |
| `codex resume` | - | 過去セッションを再開 |
| `codex fork` | - | 過去セッションをフォーク |
| `codex login` | - | 認証管理 |
| `codex logout` | - | 認証削除 |
| `codex mcp` | - | MCP サーバー管理 |
| `codex mcp-server` | - | Codex 自体を MCP サーバーとして起動 |
| `codex sandbox` | - | サンドボックスのテスト |
| `codex features` | - | フィーチャーフラグの確認 |
| `codex apply` | `codex a` | 最新の diff を git apply |
| `codex app` | - | デスクトップアプリ起動（macOS: インストーラーも自動DL） |
| `codex app-server` | - | [実験的] アプリサーバー起動 |
| `codex debug` | - | デバッグツール |
| `codex cloud` | - | [実験的] Codex Cloud タスク |
| `codex completion` | - | シェル補完スクリプト生成 |

## 対話モード（codex）のフラグ

`codex` の対話モードでのみ使用可能なフラグ:

```
--search                         ライブWeb検索を有効化（exec では使えない）
--no-alt-screen                  代替スクリーン無効（Zellij 等でスクロールバック保持）
-a, --ask-for-approval <MODE>    untrusted | on-request | never（on-failure は非推奨）
--remote <ADDR>                  リモートアプリサーバーに接続（ws:// or wss://）
--remote-auth-token-env <VAR>    リモート接続のBearerトークン環境変数名
```

## 共通フラグ

対話モード・exec 両方で使用可能:

```
-m, --model <MODEL>              モデル指定
-c, --config <key=value>         config 値をオーバーライド（TOMLとして解析）
-s, --sandbox <MODE>             read-only | workspace-write | danger-full-access
-C, --cd <DIR>                   作業ディレクトリ指定
-i, --image <FILE>               画像添付（複数指定可）
-p, --profile <PROFILE>          名前付きプロファイル使用
--full-auto                      on-request承認 + workspace-writeサンドボックス
--enable <FEATURE>               フィーチャーフラグを強制有効化（複数指定可）
--disable <FEATURE>              フィーチャーフラグを強制無効化（複数指定可）
--add-dir <DIR>                  追加の書き込み可能ディレクトリ
--oss                            ローカルOSSプロバイダー使用（LM Studio / Ollama）
--local-provider <PROVIDER>      OSSプロバイダー指定: lmstudio | ollama
--dangerously-bypass-approvals-and-sandbox  全承認・サンドボックスをバイパス（危険）
PROMPT                           位置引数（省略するとTUI起動）
```

## codex exec の全フラグ

グローバルフラグに加えて:

```
--ephemeral                      セッションを保存しない
--json                           JSONL イベント出力
-o, --output-last-message <FILE> 最終メッセージをファイルに保存
--output-schema <FILE>           出力の JSON Schema 制約
--color <MODE>                   always | never | auto
--skip-git-repo-check            Git リポ外でも実行許可
PROMPT                           位置引数。"-" で stdin から読む
```

### codex exec resume

```
--all                            全ディレクトリのセッションを検索
--last                           直近のセッションを自動再開
SESSION_ID                       特定セッションを再開
```

### codex exec review / codex review

```
--uncommitted                    ステージング済み + 未ステージング + 未追跡の変更をレビュー
--base <BRANCH>                  指定ブランチとの差分をレビュー
--commit <SHA>                   特定コミットの変更をレビュー
--title <TITLE>                  レビューサマリーに表示するコミットタイトル
PROMPT                           カスタムレビュー指示。"-" で stdin から読む
```

## Web検索の設定

3つのモード:
- `disabled` — Web検索無効
- `cached`（デフォルト）— OpenAI管理のインデックスからキャッシュ結果を返す
- `live` — ライブWebフェッチで最新データを取得

設定方法:
```bash
# 対話モードのみ: CLI フラグ
codex --search "検索クエリ"

# exec モード: -c でオーバーライド（--search は exec にはない）
codex exec -c 'web_search="live"' "検索クエリ"

# config.toml（全モード共通）
web_search = "live"    # disabled | cached | live
```

## Sandbox モード

| モード | ファイル | ネットワーク | 用途 |
|---|---|---|---|
| `read-only` (デフォルト) | 読み取りのみ | 無効 | 質問・調査 |
| `workspace-write` | ワークスペース内R/W | 設定可能 | コード変更タスク |
| `danger-full-access` | 全アクセス | 有効 | Docker等の外部サンドボックス内 |

`workspace-write` の追加設定（config.toml）:
```toml
[sandbox_workspace_write]
network_access = true              # アウトバウンドネットワークを許可
writable_roots = ["/tmp/mydir"]    # 追加の書き込み可能ディレクトリ
```

## config.toml の主要フィールド

```toml
model = "gpt-5.4"
model_reasoning_effort = "medium"       # minimal | low | medium | high | xhigh
model_reasoning_summary = "auto"        # auto | concise | detailed | none
model_verbosity = "medium"              # low | medium | high
approval_policy = "on-request"          # untrusted | on-request | never | { granular = {...} }
sandbox_mode = "workspace-write"
web_search = "live"                     # disabled | cached(デフォルト) | live
personality = "friendly"                # none | friendly | pragmatic
service_tier = "fast"                   # flex | fast

[model_providers.custom]
base_url = "https://..."
env_key = "MY_KEY"
name = "My Provider"

[mcp_servers.example]
command = "node"
args = ["/path/to/server.js"]
enabled = true
```

## プロファイル

`[profiles.<name>]` でプリセット設定を定義し、`-p <name>` で切り替える。
プロファイルではトップレベルの設定キー（model, web_search 等）をオーバーライドできる。

```toml
# ~/.codex/config.toml に追加

# 高速Web検索用プロファイル: codex exec -p search "クエリ"
[profiles.search]
model = "gpt-5.4-mini"
model_reasoning_effort = "low"
web_search = "live"
```

使用例:
```bash
# プロファイルで高速Web検索
codex exec --ephemeral -s read-only -p search "最新のNode.js LTSバージョンは？"

# プロファイルなしの場合（同等の設定を手動指定）
codex exec --ephemeral -s read-only -m gpt-5.4-mini \
  -c 'model_reasoning_effort="low"' -c 'web_search="live"' "クエリ"
```

設定の優先順位: 専用フラグ (`-m` 等) > `-c key=value` > プロファイル (`-p`) > `~/.codex/config.toml`

## モデル一覧（v0.117.0 時点の公開モデル）

| モデル | 推論レベル | 説明 |
|--------|-----------|------|
| `gpt-5.4` | low/medium/high/xhigh | デフォルト。最新フロンティアモデル |
| `gpt-5.3-codex` | low/medium/high/xhigh | 前世代のコーディング特化モデル |
| `gpt-5.2-codex` | low/medium/high/xhigh | フロンティアコーディングモデル |
| `gpt-5.2` | low/medium/high/xhigh | 汎用フロンティアモデル |
| `gpt-5.1-codex-max` | low/medium/high/xhigh | 深い推論向けフラッグシップ |
| `gpt-5.1-codex-mini` | medium/high | 高速・低コスト。Web検索・軽量タスク向け |

注意:
- `gpt-5.4-mini` / `gpt-5.4-nano` は v0.117.0 の models.json（CLI ピッカー）に未登録だが `-m` やプロファイルで直接指定すれば API 経由で使用可能
- CLI ピッカーの mini 枠は `gpt-5.1-codex-mini`（medium/high のみ対応）
- `-m` に任意のモデル名を渡せる。models.json にないモデルでも API 側で有効なら動作する

## 推論レベル

`model_reasoning_effort` に指定可能な値:
`minimal` / `low` / `medium`(デフォルト) / `high` / `xhigh`

- `minimal`: gpt-5 のみ対応。最も低レイテンシ
- `low`: 軽い推論。Web検索のクエリ生成・結果要約には十分（mini は非対応）
- `medium`: デフォルト。ほとんどのタスクに適切
- `high`: 複雑な推論・数学・コード生成向け
- `xhigh`: gpt-5.4/5.3-codex/5.2-codex/5.2/5.1-codex-max 対応。最も深い推論

利用可能なレベルはモデルにより異なる（models.json で定義）。
設定の優先順位: 専用フラグ (`-m` 等) > `-c key=value` > プロファイル (`-p`) > `~/.codex/config.toml`

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| `codex: command not found` | `npm i -g @openai/codex` か `brew install --cask codex` |
| 認証エラー (401/403) | `OPENAI_API_KEY` を確認、または `codex login` を再実行 |
| タイムアウト | Bash の `timeout` を 300000 に設定して再実行 |
| 長時間応答なし | `-c model_reasoning_effort=low` で再試行 |
| `--search` エラー | exec では使えない。`-c 'web_search="live"'` に置き換える |
| モデル名エラー | TUI で `/model` を実行して利用可能なモデル名を確認 |
| その他 exit code 非0 | stderr（`2>&1` で取得）の内容を確認して報告する |
