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

Bedrock 版 Claude Code など WebSearch が使えない環境では Codex の Web検索で代替する。
WebSearch が利用可能な場合はそちらを優先する。

## 前提

- `npm i -g @openai/codex` または `brew install --cask codex` でインストール済み
- `OPENAI_API_KEY`（または `CODEX_API_KEY`）で認証済み、もしくは `codex login` 済み
- 公式ドキュメント: https://developers.openai.com/codex

## 基本パターン

```bash
# 質問・調査（最も基本的な形）
codex exec --ephemeral -s read-only "プロンプト"

# Web検索（config.toml に [profiles.search] があれば推奨）
codex exec --ephemeral -s read-only -p search "プロンプト"
# プロファイル未設定時: -m gpt-5.4-mini -c 'model_reasoning_effort="low"' -c 'web_search="live"'

# コードレビュー
codex exec review --uncommitted
codex exec review --base main
codex exec review --commit <SHA>
```

## フロー

### Step 0: プロジェクト設定の確認

以下の2ファイルを Read ツールで同時に読む（Glob/Search は不要。存在しなければエラーを無視）:
- `.codex/config.toml`（カレントディレクトリ相対パス）
- `~/.codex/config.toml`（グローバル設定）

設定済みの値（model, web_search, profiles 等）を把握し、`-c` で重複指定しない。

### Step 1: プロンプトの構築

ユーザーの意図をそのまま抽出するのが基本。曖昧な場合はプロジェクトのコンテキスト
（言語、フレームワーク等）を補足する。日本語で問題ないが、技術的な質問で英語の方が
精度が出そうなら翻訳してもよい。

### Step 2: Web検索の判定

デフォルトは `cached`（OpenAI管理インデックス）。以下に該当する場合は `live` に切り替える:
- 「最新の」「現在の」「調べて」「ニュース」「公式ドキュメント」「料金」「リリース」への言及
- バージョン番号・日付・URL・時事的な話題
- ファクトチェック・公式情報の確認
- WebSearch ツールが利用不可な環境で Web情報が必要な場合

`live` が必要と判定したら、Step 0 で確認した `[profiles.search]` の有無に応じて
`-p search` またはフォールバック（基本パターン参照）で切り替える。
config.toml に `web_search = "live"` があれば追加指定は不要。
技術質問でも正確な情報が求められる場合は live 検索を使う。
不要なのは概念的な説明（例: 「再帰とは何か」）など、最新性が不要なケースのみ。

### Step 3: sandbox モードの選択

| モード | 用途 |
|---|---|
| `read-only`（デフォルト） | 質問・調査。ファイルを変更しない |
| `workspace-write` | コード変更タスク。`--full-auto` で自動承認 |
| `danger-full-access` | 外部サンドボックス内でのみ使用 |

### Step 4: 実行

基本パターンのコマンドを実行する。Bash ツールのタイムアウトはデフォルト120秒。
Codex の応答が長くなりそうな場合は `timeout` パラメータを 300000（5分）に設定する。

### Step 5: 出力の処理

- stdout の内容をそのまま表示する
- 出力が長大（200行超）な場合は要約してから表示する
- exit code が非0の場合は stderr を確認してエラーを報告する

## モデル選択

ユーザーがモデルを明示指定した場合はそのまま `-m` に渡す（テーブルにない名前でもOK）。
指定がない場合、config.toml のデフォルトをベースにタスクの複雑さで切り替える:

| タスク | モデル | 推論 | 理由 |
|--------|--------|------|------|
| Web検索・バージョン確認 | `gpt-5.4-mini`* | `low` | 最速。検索精度は mini で十分 |
| 簡単な質問・コード説明 | `gpt-5.4-mini`* | デフォルト | 2倍高速、コスト効率が良い |
| ファクトチェック（複数項目）・複雑な分析 | `gpt-5.4`（デフォルト） | デフォルト | 最高の推論力 |
| コードレビュー | `gpt-5.4-mini`* or `gpt-5.4` | デフォルト | 規模と複雑さで判断 |

*`gpt-5.4-mini` は CLI ピッカー未登録。`-m` / プロファイルで直接指定すれば動作する。

推論レベル: `minimal` / `low` / `medium`(デフォルト) / `high` / `xhigh`。
`-c model_reasoning_effort=low` で指定。全モデル・プロファイル設定は
`references/cli-reference.md` を参照

## よく使うオプション

| フラグ | 説明 |
|---|---|
| `-p <profile>` | 名前付きプロファイルで設定を一括切り替え |
| `-c 'web_search="live"'` | Web検索有効化（exec では `--search` 不可） |
| `-o <path>` | 最終メッセージをファイルに保存 |
| `-i <file>` | 画像を添付（スクリーンショット等） |
| `--output-schema <path>` | JSON Schema で出力形状を制約 |
| `--full-auto` | `on-request` 承認 + `workspace-write` |
| `-C <dir>` | 作業ディレクトリを指定 |
| `--json` | JSONL イベントストリーム出力 |

全フラグ・エラー対処・公式ドキュメントは `references/cli-reference.md` を参照。
