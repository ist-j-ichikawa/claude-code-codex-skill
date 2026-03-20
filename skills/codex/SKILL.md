---
name: codex
effort: low
description: |
  Invoke the OpenAI Codex CLI headlessly via `codex exec` to get answers from
  another AI, perform live web searches, run code reviews, or get a second opinion.
  Use this skill whenever the user mentions Codex, GPT, OpenAI, or wants another
  AI's perspective — even if they don't explicitly say "codex". Also trigger when
  the user wants to fact-check something against live web data, get a second opinion
  on a technical decision, or delegate a research task to another AI.
  Also useful as a WebSearch alternative in Bedrock Claude Code or other
  environments where WebSearch/WebFetch tools are unavailable.
  Typical triggers: /codex, "Codexに聞いて", "Codexで調べて", "Codexでweb検索",
  "GPTで調べて", "GPTに聞いて", "OpenAIに聞いて", "Codexを使って", "別のAIに聞いて",
  "セカンドオピニオン", "Codexにタスクを投げて", "ask Codex", "use Codex to search",
  "what does Codex think", "run this through Codex", "Codexでレビューして",
  "ファクトチェックして", "最新情報を確認して", "web検索して", "ネットで調べて".
---

# /codex — Codex CLI ヘッドレス呼び出し

OpenAI Codex CLI を `codex exec` でヘッドレス実行し、別の AI からの回答を取得する。
Codex は独自のコンテキストと（オプションで）ライブWeb検索を持つため、
ファクトチェック・最新情報の確認・セカンドオピニオンとして特に有効。

## Bedrock Claude Code での Web検索代替

Bedrock 版 Claude Code では WebSearch / WebFetch ツールが利用できない。
WebSearch が使えない環境で Web情報が必要な場合、Codex の Web検索で代替する。
WebSearch が利用可能な場合はそちらを優先する。

```bash
# WebSearch 代替パターン
codex exec --ephemeral -s read-only -c 'web_search="live"' \
  "以下について最新の公式情報を調べてください: <ユーザーの質問>"
```

## 前提

- `npm i -g @openai/codex` または `brew install --cask codex` でインストール済み
- `OPENAI_API_KEY`（または `CODEX_API_KEY`）で認証済み、もしくは `codex login` 済み
- 公式ドキュメント: https://developers.openai.com/codex

## 基本パターン

```bash
# 質問・調査（最も基本的な形）
codex exec --ephemeral -s read-only "プロンプト"

# Web検索が必要な場合（最新情報・ファクトチェック）
# 重要: --search は対話モード専用。exec では -c で設定する
codex exec --ephemeral -s read-only -c 'web_search="live"' "プロンプト"

# コードレビュー
codex exec review --uncommitted
codex exec review --base main
```

## フロー

### Step 0: プロジェクト設定の確認

カレントディレクトリの `.codex/config.toml` を確認する。
- **ファイルが存在する場合**: 中身を読み、設定済みの値（model, web_search 等）を把握する。config.toml の設定は自動適用されるため `-c` で重複指定しない
- **ファイルが存在しない場合**: グローバル設定（`~/.codex/config.toml`）を確認する
- **どちらも存在しない場合**: Codex のデフォルト設定で動作する。必要なオプションはすべて `-c` や `-m` で明示指定する

### Step 1: プロンプトの構築

ユーザーの意図をそのまま抽出するのが基本。曖昧な場合はプロジェクトのコンテキスト
（言語、フレームワーク等）を補足する。日本語で問題ないが、技術的な質問で英語の方が
精度が出そうなら翻訳してもよい。

### Step 2: Web検索の判定

以下のいずれかに該当する場合、ライブWeb検索を有効にする:
- 「最新の」「現在の」「調べて」「ニュース」「公式ドキュメント」「料金」「リリース」への言及
- バージョン番号・日付・URL・時事的な話題
- ファクトチェック・公式情報の確認
- WebSearch ツールが利用不可な環境で Web情報が必要な場合

config.toml に `web_search = "live"` があれば追加指定は不要。なければ:
```bash
codex exec --ephemeral -s read-only -c 'web_search="live"' "プロンプト"
```

時事性のない純粋な技術質問には不要（デフォルトの cached 検索で十分）。

### Step 3: sandbox モードの選択

| モード | 用途 |
|---|---|
| `read-only`（デフォルト） | 質問・調査。ファイルを変更しない |
| `workspace-write` | コード変更タスク。`--full-auto` で自動承認 |
| `danger-full-access` | 外部サンドボックス内でのみ使用 |

### Step 4: 実行

```bash
codex exec --ephemeral -s read-only "プロンプト"
```

Bash ツールのタイムアウトはデフォルト120秒。Codex の応答が長くなりそうな場合は
`timeout` パラメータを 300000（5分）に設定する。

### Step 5: 出力の処理

- stdout の内容をそのまま表示する
- 出力が長大（200行超）な場合は要約してから表示する
- exit code が非0の場合は stderr を確認してエラーを報告する

## モデル選択

ユーザーがモデルを明示指定した場合はそのまま `-m` に渡す（テーブルにない名前でもOK）。
指定がない場合、config.toml のデフォルトをベースにタスクの複雑さで切り替える:

| タスク | モデル | 理由 |
|--------|--------|------|
| 簡単な質問・バージョン確認・コード説明 | `gpt-5.4-mini` | 2倍高速、レート制限の30%で済む |
| ファクトチェック（複数項目）・複雑な分析 | `gpt-5.4`（デフォルト） | 高い推論力が必要 |
| コードレビュー | `gpt-5.4-mini` or `gpt-5.4` | 規模と複雑さで判断 |

```bash
# 軽量タスク → mini で高速・低コスト
codex exec --ephemeral -s read-only -m gpt-5.4-mini "プロンプト"

# 複雑なタスク → デフォルトモデル（config.toml に従う）
codex exec --ephemeral -s read-only "プロンプト"
```

推論レベル: `low` / `medium`(デフォルト) / `high`。`-c model_reasoning_effort=high` で指定。
全レベル（`minimal`〜`xhigh`）は `references/cli-reference.md` を参照

## よく使うオプション

| フラグ | 説明 |
|---|---|
| `-c 'web_search="live"'` | Web検索有効化（exec では `--search` 不可） |
| `-o <path>` | 最終メッセージをファイルに保存 |
| `-i <file>` | 画像を添付（スクリーンショット等） |
| `--output-schema <path>` | JSON Schema で出力形状を制約 |
| `--full-auto` | `on-request` 承認 + `workspace-write` |
| `-C <dir>` | 作業ディレクトリを指定 |
| `--json` | JSONL イベントストリーム出力 |

全フラグ・エラー対処・公式ドキュメントは `references/cli-reference.md` を参照。
