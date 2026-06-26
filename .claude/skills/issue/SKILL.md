---
name: issue
description: gh issue create で勘定くんリポジトリに日本語テンプレ（背景・目的/要件/受け入れ条件/マイルストーン/補足）のIssueを起票する。docs/requirement.md やマイルストーンから機能・タスクをIssue化したいとき、バグや改善を登録したいときに使う。
---

## 役割
勘定くんリポジトリ（katsudon08/kanjo-kun）に `gh issue create` でIssueを起票する。日本語の統一テンプレートで本文を組み立て、ラベルを付与する。docs/requirement.md やマイルストーンから機能・タスクをIssue化する起票補助も行う。本 skill は新規 Issue の起票を担う（既存 Issue の更新・クローズは扱わない）。

## 前提
- gh CLI が認証済みであること（このリポジトリでは katsudon08 で認証済み）。
- 記述はすべて日本語。ドメイン用語（会計/店舗ブランチ/はしご/代表者/メンバー/支払い源/依存関係（債務グラフ）/フローチャート/割り勘比重/定数金額/精算/レシート自動入力）を統一して使う。
- マイルストーン区分: 【〜リリース(MVP)】【〜第1回 機能追加】【〜第1回 リファクタリング】。
- ラベル体系の唯一の正は `docs/workflow.md` の「issue 運用 / ラベル体系」節に定める（type プレフィックス方式）。本 skill と Issue テンプレートはともにこれを参照し、語彙を一致させる。

## 手順
1. 起票内容を決める。
   - ユーザー指定の内容がある場合はそれを使う。
   - 「要件から起票」「マイルストーンから起票」と指示された場合は、`docs/requirement.md` を読み、機能要件・マイルストーンの各項目を1Issue=1関心事の粒度に分解して下書きする（下記「起票補助」参照）。
2. ラベルを準備する。
   - ラベル体系は `docs/workflow.md` の「ラベル体系」を唯一の正とする（type プレフィックス方式）。
   - `gh label list --repo katsudon08/kanjo-kun` で既存ラベルを確認する。
   - 採用するラベル運用（type プレフィックス）:
     - `type:feat`（新機能） / `type:bug`（不具合） / `type:fix`（修正） / `type:docs`（ドキュメント） / `type:refactor`（リファクタ） / `type:test`（テスト） / `type:chore`（雑務）
     - 任意でドメイン scope ラベル: `receipt` / `member` / `settlement` / `flowchart` / `branch` / `payment-source` / `auth`
   - GitHub のデフォルトラベル（bug, documentation, enhancement 等）は使わない。`docs/workflow.md` の方針に従い、type プレフィックス方式に統一する（デフォルトラベルは読み替える/使用しない）。Issue テンプレート（feature.md / bug.md）も同じ type プレフィックスを付与する前提である。
   - 付与したいラベルが未作成なら、先に作成する。例:
     - `gh label create "type:feat" --repo katsudon08/kanjo-kun --color a2eeef --description "新機能"`
     - `gh label create "type:bug" --repo katsudon08/kanjo-kun --color d73a4a --description "不具合"`
     - `gh label create "type:docs" --repo katsudon08/kanjo-kun --color 0075ca --description "ドキュメント"`
   - 補足: Issue テンプレート経由の Web UI 起票では、frontmatter に指定したラベルが未作成だと無視され付与されない。テンプレートでも type プレフィックスラベルを使うため、`type:feat` / `type:bug` 等は事前に上記コマンドで作成しておく。
3. 本文を組み立てる。
   - 下記テンプレートを日本語で埋める。テンプレートファイルではなく、ヒアドキュメント等で組み立てて `--body` に直接渡す。
   - 「## マイルストーン」には該当区分（MVP / 第1回 機能追加 / 第1回 リファクタリング）を記載。
4. Issueを作成する。
   - `gh issue create --repo katsudon08/kanjo-kun --title "<日本語タイトル>" --body "<組み立てた本文>" --label "type:feat"` の形で実行する。
   - タイトルは簡潔に（例 `レシートAI自動入力の会計JSON変換`、`割り勘比重・定数金額の管理`）。種別はラベルで表すため、タイトルにtypeプレフィックスは付けない。
   - 任意オプション:
     - 複数ラベル: `--label` を繰り返す（例 `--label "type:feat" --label "receipt"`）。
     - 担当者アサイン: `--assignee <github-username>`（自分なら `--assignee @me`）。開発体制は自分＋先輩の2名、役割は流動的なので必要に応じて確認する。
     - GitHub Milestone を運用している場合は `--milestone "<名称>"`。
5. 作成後の案内。
   - 出力されたIssueのURLを提示する。
   - 実装に着手する際は、このIssue番号を先頭に付けた機能ブランチ `type/<このIssue番号>-<kebab要約>`（例 `feat/12-receipt-ai-input`。詳細は docs/workflow.md「ブランチ命名規則」）を切るよう案内する。PR作成は /pr を案内し、PR本文で `Closes #<このIssue番号>` を書くと自動クローズできる旨も伝える。

## Issue本文テンプレート（--body に渡す全文）
```
## 背景・目的
<なぜこのIssueが必要か。解決したい課題やユーザー価値を日本語で。
例: はしごで積み重なる「誰が誰にいくら」の依存関係をなぁなぁにせず可視化するため 等>

## 要件
- <満たすべき機能・仕様を箇条書き>
- <要件2>

## 受け入れ条件
- [ ] <完了と判断できる客観的な条件>
- [ ] <条件2>

## マイルストーン
<【〜リリース(MVP)】/【〜第1回 機能追加】/【〜第1回 リファクタリング】 のいずれか>

## 補足
- <参考リンク・関連Issue・設計メモ・スタック依存で未確定な点 等>
- （技術スタック依存の詳細は技術スタック決定後に記載 / docs/architecture.md 参照）
```

## 起票補助（docs/requirement.md・マイルストーンから）
- `docs/requirement.md` の「機能要件」と「マイルストーン」を読み、1Issue=1関心事に分解する。MVPの代表的な起票候補:
  - 店舗ブランチとフローチャート（はしごの店舗ノードと依存関係の可視化）
  - フローチャートに基づく金額の依存関係表示（権限別: 本人のみ / 全体一覧）
  - メンバー管理
  - 依存関係（債務グラフ）の構築と、最小回数の送金へ簡約する精算ロジック（債務簡約）
- 第1回 機能追加の候補: 割り勘比重・定数金額の管理 / 単一・複数の支払い源管理 / レシートAI自動入力（会計JSONへの変換, サブスク）。
- 第1回 リファクタリングの候補: モデル整理 / ディレクトリ構成見直し / テスト整備 / Storybook整備。
- 各Issueには適切なマイルストーン区分と type ラベル（type:feat / type:docs 等）、必要ならドメイン scope ラベルを付ける。

## 規約・注意
- 出力・本文・コメントはすべて日本語。
- スタック依存の記述（具体コマンド・ライブラリ・ディレクトリ構成）は決め打ち・捏造しない。未確定箇所は「（技術スタック決定後に記載 / docs/architecture.md 参照）」と書く。
- ラベルは存在しないものを `--label` 指定するとエラーになるため、付与前に存在確認し、無ければ作成する。
- Issue起票はGitHub Flowの起点。実装はIssue→機能ブランチ→PR（/pr）→マージの流れに乗せる。
