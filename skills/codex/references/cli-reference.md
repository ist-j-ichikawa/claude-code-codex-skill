# Codex CLI Reference (v0.115.0)

> このファイルはスキル実行時に不明な点がある場合のみ参照する。
> 公式ドキュメント: https://developers.openai.com/codex
> ソースコード: https://github.com/openai/codex/tree/main/codex-rs
> 最終更新: 2026-03-17（v0.115.0 で確認済み）

## 目次

1. [コマンド一覧](#コマンド一覧)
2. [codex exec の全フラグ](#codex-exec-の全フラグ)
3. [グローバルフラグ](#グローバルフラグ)
4. [Web検索の設定](#web検索の設定)
5. [Sandbox モード](#sandbox-モード)
6. [config.toml の主要フィールド](#configtoml-の主要フィールド)
7. [推論レベル](#推論レベル)

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
| `codex cloud` | - | [実験的] Codex Cloud タスク |
| `codex completion` | - | シェル補完スクリプト生成 |

## 対話モード（codex）のフラグ

`codex` の対話モードでのみ使用可能なフラグ:

```
--search                         ライブWeb検索を有効化（exec では使えない）
--no-alt-screen                  代替スクリーン無効（TUI用）
```

## 共通フラグ

対話モード・exec 両方で使用可能:

```
-m, --model <MODEL>              モデル指定
-c, --config <key=value>         config 値をオーバーライド（TOMLとして解析）
-s, --sandbox <MODE>             read-only | workspace-write | danger-full-access
-a, --ask-for-approval <MODE>    untrusted | on-request | never
-C, --cd <DIR>                   作業ディレクトリ指定
-i, --image <FILE>               画像添付（カンマ区切り / 複数指定可）
-p, --profile <PROFILE>          名前付きプロファイル使用
--full-auto                      on-request承認 + workspace-writeサンドボックス
--enable <FEATURE>               フィーチャーフラグを強制有効化（複数指定可）
--disable <FEATURE>              フィーチャーフラグを強制無効化（複数指定可）
--add-dir <DIR>                  追加の書き込み可能ディレクトリ
--oss                            ローカルOSSプロバイダー使用（Ollama等）
--dangerously-bypass-approvals-and-sandbox  全承認・サンドボックスをバイパス（危険）
PROMPT                           位置引数（省略するとTUI起動）
```

## codex exec の全フラグ

グローバルフラグに加えて:

```
--ephemeral                      セッションを保存しない
--json, --experimental-json      JSONL イベント出力
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
model = "gpt-5-codex"
model_reasoning_effort = "medium"
model_reasoning_summary = "auto"        # auto | concise | detailed | none
model_verbosity = "low"                 # low | medium | high (GPT-5用)
approval_policy = "on-request"          # untrusted | on-request | never
sandbox_mode = "workspace-write"
web_search = "live"                     # disabled | cached | live
personality = "default"

[model_providers.custom]
base_url = "https://..."
api_key_env = "MY_KEY"
name = "My Provider"

[mcp_servers.example]
command = "node"
args = ["/path/to/server.js"]
enabled = true
```

## 推論レベル

`model_reasoning_effort` に指定可能な値:
`minimal` / `low` / `medium`(デフォルト) / `high` / `xhigh`
（利用可能なレベルはモデルにより異なる。例: gpt-5.4-mini は `medium`/`high` のみ）

設定の優先順位: CLI フラグ > `-c key=value` > `~/.codex/config.toml`
