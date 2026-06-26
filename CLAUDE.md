# 勘定くん

はしご飲みで積み重なる「代表者の立替 → メンバーの精算」の債務関係を可視化し、ことら送金/PayPay 等へ直接遷移して精算を完結させる割り勘アプリ。

## サービス概要

代表者が立て替えた会計をレシート等から取り込み、「誰が誰にいくら」の債務グラフ（依存関係）として可視化する。はしご（複数店舗）で複雑化した債務を最小回数の送金へ簡約し、送金サービスへ遷移して精算を完結させる。

## 主要ドメイン用語

- **会計**: 1店舗・1回分の支払い（店名・会計時刻・合計金額・メニュー明細）
- **店舗ブランチ**: はしご内の各店舗訪問ノード（1会計を内包）。Git のブランチに着想を得た命名で、グループの分岐・合流を表す。会計はその店の参加メンバー内で割り勘
- **はしご(イベント)**: 1回の催し。店舗ブランチが分岐・合流しうるグラフ（Git のコミットグラフに相当）
- **代表者**: その会計を立て替えて支払った人
- **メンバー**: 割り勘の参加者
- **支払い源**: 代表者が立替に用いた原資（単一/複数）
- **依存関係(債務グラフ)**: 「誰が誰にいくら」を表す有向グラフ
- **フローチャート**: 依存関係の可視化（権限により本人のみ/全体一覧）
- **割り勘比重 / 定数金額**: 参加者ごとの負担割合 / 比重に依らない固定負担額
- **精算**: 送金により債務を解消すること（ことら送金/PayPay へ遷移）
- **レシート自動入力**: AIでレシートを会計データ(JSON)へ変換するサブスク機能

用語の詳細は [docs/domain.md](docs/domain.md) を参照。

## 技術スタック

未定（FE/BE/言語/フレームワーク/DB/インフラは一切決まっていない）。ADR-0002 で決定予定。決定後に `docs/architecture.md` を作成して記載する。

## ディレクトリ構成

（技術スタック決定後に記載 / docs/architecture.md 参照）

## 開発フロー

- **Git運用**: GitHub Flow。`main` + 機能ブランチ → PR でマージ。`main` への直コミット/直pushは禁止。
- **ブランチ命名**: `type/<issue番号>-<kebab要約>`（対応 issue があれば。例 `feat/12-receipt-ai-input`）／無ければ `type/<kebab要約>`（例 `chore/setup-claude-env`）。
- **コミット規約**: Conventional Commits。形式は `type(scope): 日本語の要約`。type は feat,fix,docs,style,refactor,perf,test,build,ci,chore,revert。scope は任意の小文字ドメイン名（例 receipt, member, settlement, flowchart, branch, payment-source, auth）。本文は「なぜ」を日本語で。footer に `Closes #N`、破壊的変更は `BREAKING CHANGE:`。
- **トレーラ/署名の禁止**: Co-Authored-By 等のトレーラ・署名（共著者表記・ツール署名）は付与しない。コミット・PR・issue いずれにも付けない。

詳細は [docs/workflow.md](docs/workflow.md) を参照。

## よく使うコマンド

（技術スタック決定後に記載 / docs/architecture.md 参照）

## ドキュメント

- [docs/requirement.md](docs/requirement.md) — 要件定義（目的・背景・ユーザーストーリー・機能要件・マイルストーン）
- [docs/domain.md](docs/domain.md) — ドメイン用語の定義
- docs/architecture.md — 技術スタック・ディレクトリ構成（ADR-0002 で技術スタック決定後に作成予定。現時点では未作成）
- [docs/workflow.md](docs/workflow.md) — 開発フロー（GitHub Flow・ブランチ命名・Conventional Commits・ラベル体系）

## Claude Code 運用

### プロジェクト skill（`.claude/skills/`）

- `/commit` — Conventional Commits 規約に沿ってコミットを作成する
- `/push` — `main` への直push をガードしつつリモートへ push する
- `/pr` — gh CLI で PR を作成する
- `/issue` — gh CLI で issue を起票する
- `/plan` — 作業計画を立てる
- `/docs` — docs/ 配下のドキュメントを整備する

### ビルトイン skill の活用

- `/code-review` — 現在の差分のコードレビュー
- `/review` — GitHub PR のレビュー
- `/run` — アプリの起動確認
- `/verify` — 変更の動作検証

これらは再実装せず、プロジェクト skill から呼び出す/案内する形で活用する。

### agents

Phase2（技術スタック決定後）に `settlement-modeler`（債務簡約ロジック向け）等の追加を予定。
