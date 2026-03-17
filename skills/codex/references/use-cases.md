# Codex スキル — ユースケース集

> このファイルはスキルの具体的な活用パターンをまとめたもの。
> 新しいユースケースが確認されたら随時追記する。
> 最終更新: 2026-03-17

## 目次

1. [ファクトチェック・最新情報の確認](#1-ファクトチェック最新情報の確認)
2. [Bedrock Claude Code での WebSearch 代替](#2-bedrock-claude-code-での-websearch-代替)
3. [セカンドオピニオン](#3-セカンドオピニオン)
4. [コードレビュー](#4-コードレビュー)
5. [ドキュメント・レポートのファクトチェック](#5-ドキュメントレポートのファクトチェック)
6. [SDK / ライブラリのバージョン調査](#6-sdk--ライブラリのバージョン調査)

---

## 1. ファクトチェック・最新情報の確認

**いつ使うか:** Claude の知識カットオフ以降の情報が必要な場合、公式ドキュメントの
最新状態を確認したい場合。

**実績:** とあるプロジェクトで OpenAI API 移行レポートのファクトチェックに使用。
Chat Completions API のステータス、Responses API のリリース日、SDK バージョンなど
10項目を Web 検索で検証し、3件の誤りを発見・修正した。

```bash
codex exec --ephemeral -s read-only -c 'web_search="live"' \
  "2026年3月時点の最新情報に基づいて、以下の事実関係をファクトチェックしてください: ..."
```

**ポイント:**
- `web_search="live"` は必須（ファクトチェックには最新情報が不可欠）
- 検証項目を番号付きリストで渡すと、項目ごとに「正確/不正確/要修正」で判定してくれる
- Bash の timeout を 300000（5分）に設定する（多数の Web 検索で時間がかかる）

---

## 2. Bedrock Claude Code での WebSearch 代替

**いつ使うか:** Amazon Bedrock 経由で Claude Code を利用しており、WebSearch /
WebFetch ツールが使えない環境。

**背景（2026年3月時点）:**
- WebSearch: Bedrock は**非対応**、Vertex AI は対応、Anthropic API は対応
- WebFetch: Bedrock / Vertex AI ともに**非対応**、Anthropic API のみ対応
- Anthropic の公式ロードマップに Bedrock 対応の ETA はない
- AWS 側ドキュメントに `web_search_20250305` の記載が出始めているが、
  Anthropic の feature matrix はまだ非対応扱い

**代替手段の比較:**

| 手段 | 普及度 | セットアップ | 特徴 |
|---|---|---|---|
| **MCP Search Server**（Brave/Tavily/Perplexity） | 主流 | `.mcp.json` に追加 | Claude Code にネイティブ統合。API キーが必要 |
| **Codex CLI** | ニッチ（上級者向け） | `npm i -g @openai/codex` | 別 AI からの回答。OpenAI API キーが必要 |
| **Playwright / browser-use MCP** | ニッチ | MCP設定 + ブラウザ | フルブラウザ操作。セットアップが重い |
| **curl / wget スクリプト** | レガシー | なし | HTML を直接取得。パース処理が必要 |
| **Vertex AI / Anthropic API に切替** | 根本解決 | 環境変数変更 | WebSearch が公式対応 |

**推奨:**
- まず MCP Search Server（Brave Search か Tavily）を導入する
- Codex CLI は MCP では得られない「別 AI の視点」が欲しい場合に併用する
- WebFetch まで必要なら Anthropic API 直接利用を検討する

```bash
# Codex CLI での代替パターン
codex exec --ephemeral -s read-only -c 'web_search="live"' \
  "以下について最新の公式情報を調べてください: <質問>"
```

---

## 3. セカンドオピニオン

**いつ使うか:** アーキテクチャ設計、技術選定、実装方針について別の AI の意見が
欲しい場合。

```bash
codex exec --ephemeral -s read-only \
  "以下のアーキテクチャ設計について評価してください。問題点やリスクがあれば指摘してください: ..."
```

**ポイント:**
- Web 検索は通常不要（`-c web_search` は省略）
- 設計のコンテキスト（制約条件、技術スタック等）を十分に含める
- 「問題点を指摘して」と明示すると批判的な視点が得られやすい

---

## 4. コードレビュー

**いつ使うか:** Claude（自分自身）が書いたコードに対して、別の AI の視点で
レビューしたい場合。

```bash
# 未コミットの変更をレビュー
codex exec review --uncommitted

# 特定ブランチとの差分をレビュー
codex exec review --base main
```

**ポイント:**
- `codex exec review` は専用サブコマンドで、git diff を自動で読み取る
- `--uncommitted` はステージング済み + 未ステージングの両方を含む
- `--base` で比較ブランチを指定できる

---

## 5. ドキュメント・レポートのファクトチェック

**いつ使うか:** 作成したドキュメントやレポートの内容が事実に基づいているか、
公式情報と照合して検証したい場合。

**実績:** とあるプロジェクトで OpenAI API 移行レポートを作成した直後に Codex で
全項目をファクトチェック。以下の誤りを発見:
- 「Chat Completions API は非推奨」→ 実際は非推奨ではない（要修正）
- 「2025年にリリース」→ 正確には「2025年3月11日」（要修正）
- 組み込みツールが3種→実際は6種（不完全）

```bash
codex exec --ephemeral -s read-only -c 'web_search="live"' \
  "以下のドキュメントの事実関係をファクトチェックしてください。
   各項目について正確/不正確/要修正で判定し、不正確な場合は正しい情報を
   提示してください。

   レビュー観点:
   1. ...
   2. ...

   ドキュメント全文:
   <ドキュメント内容>"
```

---

## 6. SDK / ライブラリのバージョン調査

**いつ使うか:** 特定のライブラリの最新バージョン、リリース日、特定機能が
どのバージョンから利用可能かを調べたい場合。

**実績:** OpenAI Node.js SDK で `responses.create` が使えるようになった
バージョン（v4.87.0）を特定。GitHub のリリースタグを辿って検証。

```bash
codex exec --ephemeral -s read-only -c 'web_search="live"' \
  "OpenAI Node.js SDK (openai) の最新安定版は何か？
   また、responses.create メソッドはどのバージョンから利用可能か？
   GitHubのリリースノートを確認して回答してください。"
```

**ポイント:**
- GitHub のリリースページや npm/PyPI のバージョン履歴を辿らせるのが有効
- 「GitHubのリリースノートを確認して」と明示するとソースの信頼性が上がる
