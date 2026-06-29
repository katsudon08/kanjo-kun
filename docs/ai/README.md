# docs/ai

このディレクトリは、AI との議論・調査・実装計画の一時保管場所です。

原則として、このディレクトリ内の作業メモは Git 管理しません（`.gitignore` 済み・個人用に蓄積し参照する）。

チームに共有すべき内容は、以下のいずれかに昇格します。

- `docs/requirements.md` — 要件定義
- `docs/domain.md` — ドメイン用語・ドメインモデル
- `docs/workflow.md` — 開発フロー
- `docs/adr/` — アーキテクチャ決定記録（技術的な意思決定）
- `CLAUDE.md` — プロジェクト全体の概要・規約
- GitHub の issue / Pull Request — 作業チケット・変更提案

## ファイルの命名規則

メモのファイル名は次の形式にする。

- 形式: `<issue番号>-<title>.md`
  - `<issue番号>`: 対応する GitHub issue があればその番号。issue を伴わない作業は省略し `<title>.md` とする。
  - `<title>`: ブランチ名から type プレフィックスを除いた部分、または調査・議論の概要を要約したもの。いずれも**ケバブケース**（小文字・ハイフン区切り）。
- 例:
  - ブランチ `feat/12-receipt-ai-input`（issue #12）→ `12-receipt-ai-input.md`
  - issue を伴わない議論 → 概要を要約して `requirement-rethink.md` など
- `README.md` はこの規則の対象外（このディレクトリの固定の案内であり、共有する唯一のファイル）。
