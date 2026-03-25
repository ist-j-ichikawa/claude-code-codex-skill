# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.2.0] - 2026-03-25

### Added

- 高速Web検索: `-p search` プロファイルで mini + low reasoning + live web search を一括設定
- GPT-5.4-nano ドキュメント: API キー認証環境向けの最速・最安モデル（cli-reference.md）
- 推論レベル `none`（GPT-5.2+）のドキュメント
- モデル一覧テーブル（GPT-5.4 世代: 5.4 / mini / nano）
- プロファイル機能のガイド（config.toml の `[profiles.<name>]` セクション）

### Changed

- モデル選択テーブルに「推論」列を追加。Web検索タスクに `low` を推奨
- config.toml 例を GPT-5.4 世代に更新（model_verbosity のデフォルトを medium に）
- SKILL.md を 147行 → 126行に圧縮（Progressive Disclosure 強化）

## [1.1.0] - 2026-03-18

### Added

- モデル選択ガイダンス: タスクの複雑さに応じて `gpt-5.4-mini` / `gpt-5.4` を自動選択
- ユーザー指定モデルのパススルー: テーブルにないモデル名でもそのまま `-m` に渡す

### Fixed

- Step 0: config.toml が存在しない場合のフォールバック（ローカル → グローバル → デフォルト）を追加。config 不在時のハルシネーションを防止

## [1.0.0] - 2026-03-17

### Added

- Codex CLI ヘッドレス呼び出しスキル（SKILL.md）
  - Web 検索（`-c 'web_search="live"'`）
  - ファクトチェック・セカンドオピニオン
  - コードレビュー（`codex exec review`）
- Bedrock Claude Code での WebSearch 代替機能
- CLI リファレンス（v0.115.0 対応）
- ユースケース集（6パターン）
- テストケース（6件）
- Claude Code プラグイン形式（`.claude-plugin/plugin.json`）
