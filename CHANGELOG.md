# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

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
