# 開発フロー — 勘定くん

本書は勘定くんの開発フロー（Git 運用・コミット規約・PR/issue/レビュー運用・体制・マイルストーン運用）を定める。記述は日本語で統一する。

## 開発体制

- 自分＋先輩エンジニアの2名体制。役割は流動的。
- `.claude/`（エージェント設定・skill）はリポジトリにコミットして2名で共有する。

## ブランチ運用: GitHub Flow

- `main` を常にデプロイ可能な安定ブランチとして維持する。
- 作業は必ず `main` から機能ブランチを切って行う。
- 機能ブランチ → Pull Request → レビュー → `main` へマージ、というフローで進める。
- **`main` への直コミット/直 push は禁止**する（ガードする）。

### ブランチ命名規則

形式: `type/<issue番号>-<kebab要約>`（対応する issue がある場合）／ `type/<kebab要約>`（対応する issue がない場合）

- `type` はコミット規約の type と同じ語彙（feat, fix, docs, ...）を用いる。
- **対応する GitHub issue がある場合は、その issue 番号をハイフン区切りで要約の先頭に付ける**（例: issue #12 に対応するなら `feat/12-...`）。issue から作業を起こす運用（issue → ブランチ → PR）と対応づけて追跡しやすくするため。
- 要約はケバブケース（小文字 + ハイフン）。
- issue を伴わない軽微な作業のみ、issue 番号を省略してよい。

例:

- `feat/12-receipt-ai-input`（issue #12 に対応）
- `fix/34-settlement-rounding`（issue #34 に対応）
- `chore/setup-claude-env`（対応 issue なし）

## コミット規約: Conventional Commits

形式: `type(scope): 日本語の要約`

- `type` は英語、要約・本文・説明は日本語。
- `scope` は任意。小文字のドメイン名（例: `receipt`, `member`, `settlement`, `flowchart`, `branch`, `payment-source`, `auth`）。
- 本文には「なぜ」を日本語で書く。
- footer に `Closes #N`。破壊的変更は本文/footer に `BREAKING CHANGE:` を記載する。
- **Co-Authored-By 等のトレーラ・署名（共著者表記・ツール署名）は付与しない。** これはコミット・PR・issue いずれにも適用する全面禁止ルールとする。

### type 一覧

| type | 用途 |
| --- | --- |
| feat | 新機能の追加 |
| fix | バグ修正 |
| docs | ドキュメントのみの変更 |
| style | 動作に影響しない変更（フォーマット・空白等） |
| refactor | 機能追加でもバグ修正でもないコード改善 |
| perf | パフォーマンス改善 |
| test | テストの追加・修正 |
| build | ビルドシステム・依存関係の変更 |
| ci | CI 設定・スクリプトの変更 |
| chore | その他の雑務（上記に当てはまらない変更） |
| revert | 以前のコミットの取り消し |

### コミット例

```
feat(receipt): レシート画像から会計JSONを生成するAI入力を追加

はしごで会計が増えると手入力の負担が大きいため、
レシートからの自動入力で代表者の入力コストを下げる。

Closes #12
```

```
fix(settlement): 債務簡約時の端数で合計がずれる不具合を修正

按分の丸め後に合計が元金額と一致しないケースがあったため、
端数を調整して総額の一致を保証する。

Closes #34
```

## Pull Request 運用

- base は必ず `main`。
- 1つの PR は1つのまとまった変更に絞る（レビューしやすい粒度）。
- 関連 issue を `Closes #N` で紐付ける。
- マージ前にレビューを受ける（2名体制のため、可能な範囲で相互レビュー）。
- gh CLI（認証済み: katsudon08）を用いて PR を作成・操作できる。

### PR テンプレート（唯一の正）

PR テンプレートの唯一の正は `.github/pull_request_template.md`（GitHub が PR 作成時に自動適用する）。節構成は **概要 / 変更点 / 関連 issue / 動作確認 / レビュー観点 / スクリーンショット（任意）**。`/pr` skill もこの節構成に沿って本文を組み立てる。二重管理を避けるため、本書ではテンプレ本文を再掲せず `.github/pull_request_template.md` を参照する。

## issue 運用

- 作業の起点として issue を作成し、目的・背景・完了条件を日本語で記述する。
- マイルストーン（MVP / 第1回機能追加 / 第1回リファクタリング）と紐付けて整理する。
- PR では `Closes #N` で対応 issue をクローズする。
- gh CLI で issue を作成・操作できる。Issue の起票補助は `/issue` skill が担う。
- Issue の種別はタイトルの type プレフィックスではなく、ラベルで表す（後述「ラベル体系」）。

### ラベル体系（唯一の正）

ラベル運用は **type プレフィックス方式** に統一する。`/issue` skill と `.github/ISSUE_TEMPLATE/`（feature.md / bug.md）は、いずれも本節を唯一の正として同じ語彙を用いる。

- 種別ラベル（type プレフィックス）:
  - `type:feat`（新機能） / `type:bug`（不具合） / `type:fix`（修正） / `type:docs`（ドキュメント） / `type:refactor`（リファクタ） / `type:test`（テスト） / `type:chore`（雑務）
- ドメイン scope ラベル（任意）: `receipt` / `member` / `settlement` / `flowchart` / `branch` / `payment-source` / `auth`
- GitHub のデフォルトラベル（`enhancement` / `bug` / `documentation` 等）は使用しない（type プレフィックスへ読み替える）。混在を避けるため、必要に応じてデフォルトラベルは削除または不使用とする。
- 種別はラベルで表すため、Issue タイトルに `feat:` / `fix:` 等の type プレフィックスは付けない。
- 上記ラベルは事前に作成しておく（未作成のラベルは、テンプレート経由の Web UI 起票では無視され付与されない）。作成コマンドは `/issue` skill 参照（例: `gh label create "type:feat" --repo katsudon08/kanjo-kun --color a2eeef --description "新機能"`）。

## レビュー運用

既存のビルトイン skill を活用する（再実装しない）。

- ローカルの変更差分のコードレビュー: `/code-review`
- GitHub 上の Pull Request のレビュー: `/review`
- アプリ起動確認: `/run`
- 変更の動作検証: `/verify`

2名体制では、PR 提出前に作成者自身が `/code-review` を回し、必要に応じて `/run`・`/verify` で動作を確認する。レビュアーは GitHub 上で `/review` を活用する。

## 2名体制での運用方針

- 役割は流動的。実装担当・レビュー担当を案件ごとに柔軟に分担する。
- 小さく早く PR を出し、相互レビューのリードタイムを短く保つ。
- レビュアーが不在/即時対応できない場合も、`main` 直 push は行わず PR 経由を厳守する。
- 設定・運用ルール（`.claude/` 含む）の変更も PR で共有し、2名で認識を合わせる。

## マイルストーン運用

マイルストーンは要件定義（docs/requirement.md）に準拠する。

- **〜リリース（MVP）**: 店舗ブランチとフローチャート / メンバー管理 / FE・BE 作成 / 依存関係表示（権限別）。
- **〜第1回 機能追加**: 比重・定数金額管理 / 単一・複数の支払い源管理。
- **〜第1回 リファクタリング**: モデル整理 / ディレクトリ構成見直し / テスト整備 / Storybook 整備。

issue・PR を各マイルストーンに割り当て、進捗を管理する。スタック依存の具体的な開発コマンド等は（技術スタック決定後に記載 / docs/architecture.md 参照）。

## 関連ドキュメント

- 要件定義: docs/requirement.md
- ドメインモデル: docs/domain.md
- アーキテクチャ決定記録: docs/adr/
