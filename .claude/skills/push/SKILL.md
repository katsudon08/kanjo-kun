---
name: push
description: 現在のブランチを origin へ push する。コミット済みの変更をリモートへ反映したいとき、「push して」「リモートに上げて」「プッシュ」等と言われたときに使う。
---

## 役割
現在の機能ブランチを origin へ push する。upstream 未設定なら追跡設定込みで push し、push 後に PR 未作成であれば /pr を案内する。main への直 push はガードする。

## 前提
- Git 運用は GitHub Flow。main への直 push は禁止。push は機能ブランチに対してのみ行う。
- リモートは GitHub の `katsudon08/kanjo-kun`（デフォルトブランチ main）。gh CLI は認証済み（katsudon08）。
- マージはローカルではなく PR 経由で行う。push はあくまでブランチをリモートへ反映する操作。

## 手順
1. `git status` で未コミットの変更が無いか確認する。未コミットの変更が残っている場合は、push 前に /commit でコミットするようユーザーに促す（意図しない取りこぼしを防ぐ）。
2. `git branch --show-current` で現在のブランチを確認する。`main` だった場合は push を中止し、main への直 push は禁止である旨を伝え、機能ブランチへ切り替える（または機能ブランチを作る）よう促す。
3. upstream（追跡ブランチ）の設定有無を確認する。`git rev-parse --abbrev-ref --symbolic-full-name @{upstream}` が失敗する＝未設定。
   - 未設定の場合: `git push -u origin <現在のブランチ名>` で追跡設定込みで push する。
   - 設定済みの場合: `git push` で push する。
4. push 結果（成功可否、反映先ブランチ、push したコミット範囲）をユーザーに報告する。
5. 当該ブランチの PR がまだ無い場合は、PR 作成のため /pr を案内する。`gh pr view --json url,state` 等で既存 PR の有無を確認し、未作成なら /pr、作成済みなら既存 PR の URL を伝える。

## 規約・注意
- main への直 push は禁止。手順2のガードを必ず通す。
- `--force` / `--force-with-lease` は安易に使わない。履歴の強制上書きが必要な場合は、必ずユーザーに理由を確認・合意のうえで実行する。
- push 前に秘密情報を含むコミットが混入していないか留意する（混入の疑いがあれば push せず /commit 側の確認に戻す）。
- ビルトイン skill と重複しない。コミットは /commit、PR 作成・更新は /pr、PR レビューは /review を案内する。
