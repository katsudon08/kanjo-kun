# ドキュメント案内 — 勘定くん

勘定くんのドキュメント一覧と、それぞれの役割。

| ファイル | 内容 |
| --- | --- |
| [requirements.md](requirements.md) | 要件定義（目的・背景・対象ユーザー・ユーザーストーリー・機能要件・マイルストーン） |
| [domain.md](domain.md) | ドメイン用語とドメインモデル（店舗グラフ／債務グラフ、店舗＝ノード・ブランチ＝分岐 など） |
| [workflow.md](workflow.md) | 開発フロー（GitHub Flow・issue/PR・レビュー運用・体制・マイルストーン） |
| [adr/](adr/) | アーキテクチャ決定記録（ADR）。重要な技術的意思決定を残す。書式は [adr/README.md](adr/README.md) を参照 |
| [ai/README.md](ai/README.md) | AI 作業メモ（research.md / plan.md）の運用ルールとテンプレート。メモ本体は git 管理外で、この README のみチーム共有 |

技術スタックは未定。決定したら ADR に記録し、`architecture.md`（技術スタック・ディレクトリ構成）を追加する予定。機能単位の設計説明は今後 `design/` に追加する予定。
