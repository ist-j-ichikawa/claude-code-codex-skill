# claude-code-codex-skill

![Claude Code](https://img.shields.io/badge/Claude_Code-%3E%3D2.1.80-blue) ![Codex CLI](https://img.shields.io/badge/Codex_CLI-required-orange)

Claude Code から OpenAI Codex CLI をヘッドレスで呼び出すプラグイン。

## できること

- ファクトチェック: Claude が生成したドキュメントの事実関係を Web 検索で検証
- Web 検索代替: Bedrock Claude Code など WebSearch ツールが使えない環境で最新情報を取得
- セカンドオピニオン: 技術設計・実装方針について別の AI の視点を取得
- コードレビュー: `codex exec review` で git diff をレビュー
- モデル自動選択: 軽量タスクは `gpt-5.4-mini`（2倍高速・コスト70%削減）、複雑なタスクは `gpt-5.4` を自動で使い分け
- 高速Web検索: `-p search` プロファイルで mini + low reasoning + live web search を一括設定

## 前提条件

- Claude Code 2.1.80 以上（`effort` フロントマター対応が必要）
- Codex CLI がインストール済みで、認証が完了していること

```bash
# インストール
npm i -g @openai/codex

# 認証（いずれか）
codex login                    # ブラウザ認証
codex login --device-auth      # デバイス認証
codex login --with-api-key     # API キーを stdin から渡す
export OPENAI_API_KEY=sk-...   # 環境変数で設定
```

## インストール

```bash
# 1. マーケットプレイスとして登録
claude plugin marketplace add ist-j-ichikawa/claude-code-codex-skill

# 2. インストール（user scope: 全プロジェクトで使用）
claude plugin install -s user claude-code-codex-skill
```

特定プロジェクトのみで使う場合は `claude plugin install -s project claude-code-codex-skill` に変更する。

### 更新

```bash
claude plugin marketplace update
claude plugin update claude-code-codex-skill
```

## 使い方

Claude Code 内で以下のように呼び出す:

```
/codex 最新のNode.js LTSバージョンを教えて
```

```
Codexでファクトチェックして: このドキュメントの内容は正しい？
```

```
Codexでレビューして
```

「web検索して」「ネットで調べて」「セカンドオピニオン」「別のAIに聞いて」などでもトリガーされる。
全トリガーフレーズは `skills/codex/SKILL.md` の description を参照。

### 高速Web検索の設定（推奨）

`~/.codex/config.toml` にプロファイルを追加すると、Web検索が高速化される:

```toml
[profiles.search]
model = "gpt-5.4-mini"
model_reasoning_effort = "low"
web_search = "live"
```

スキルが自動的にこのプロファイルを検出し、`-p search` で使用する。

## Bedrock Claude Code での活用

Bedrock 版 Claude Code では WebSearch / WebFetch ツールが利用できない（2026年3月時点）。
このプラグインの Web 検索機能でその制約を補える。

コミュニティの主流は MCP Search Server（Brave Search / Tavily）だが、
このプラグインは「別の AI の視点も欲しい」場合に特に有効。

詳細は `skills/codex/references/use-cases.md` のユースケース #2 を参照。

## ファイル構成

```
claude-code-codex-skill/
├── .claude-plugin/
│   ├── plugin.json              # プラグインマニフェスト
│   └── marketplace.json         # マーケットプレイス定義
├── skills/
│   └── codex/
│       ├── SKILL.md             # スキル本体
│       └── references/
│           ├── cli-reference.md # CLI 全フラグ・config 詳細（v0.116.0）
│           └── use-cases.md     # ユースケース集（実績ベース）
├── evals/
│   └── evals.json               # テストケース
└── README.md                    # このファイル
```

## ライセンス

MIT
