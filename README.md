# claude-code-codex-skill

Claude Code から OpenAI Codex CLI をヘッドレスで呼び出すプラグイン。

## できること

- **ファクトチェック**: Claude が生成したドキュメントの事実関係を Web 検索で検証
- **Web 検索代替**: Bedrock Claude Code など WebSearch ツールが使えない環境で最新情報を取得
- **セカンドオピニオン**: 技術設計・実装方針について別の AI の視点を取得
- **コードレビュー**: `codex exec review` で git diff をレビュー

## 前提条件

Codex CLI がインストール済みで、認証が完了していること。

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

### GitHub からインストール

```bash
claude plugin install ist-j-ichikawa/claude-code-codex-skill
```

### ローカルからインストール

```bash
git clone https://github.com/ist-j-ichikawa/claude-code-codex-skill.git
claude plugin install ./claude-code-codex-skill
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
│   └── plugin.json              # プラグインマニフェスト
├── skills/
│   └── codex/
│       ├── SKILL.md             # スキル本体
│       ├── references/
│       │   ├── cli-reference.md # CLI 全フラグ・config 詳細（v0.115.0）
│       │   └── use-cases.md     # ユースケース集（実績ベース）
│       └── evals/
│           └── evals.json       # テストケース（6件）
└── README.md                    # このファイル
```

## ライセンス

MIT
