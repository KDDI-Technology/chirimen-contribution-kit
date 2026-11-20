---
name: chirimen-sample-contribution
description: Use when the user wants to register a device's sample code, parts list entry, or parts photo to the chirimen.org repository (esm-examples, partslist.csv, partsImgs). Triggers include "CHIRIMENのサンプルコードを追加して", "部品リストに登録して", "partsImgsに写真を登録して", "chirimen.orgにサンプルを追加".
---

# CHIRIMEN サンプルコード・部品情報登録

デバイスのサンプルコード・部品リスト・部品写真を `chirimen.org` リポジトリへ登録する作業の詳細手順は、
プロジェクトルートの `CHIRIMEN_SAMPLE_CONTRIBUTION.md` に定義されている。

## 実施手順

1. まず `CHIRIMEN_SAMPLE_CONTRIBUTION.md` を読み込む（冒頭の「公式CONTRIBUTINGサイトとの相違点・補足」も含めて）
2. 特に「0. 絶対に守る制約」（個人Fork/社内ステージングFork/本家、いずれに対してもgit commit/push/PR作成
   などの書き込み操作を一切行わない）を最優先で遵守する
3. 入力ファイル `devices/<device>.sample.md` の有無を確認する
   - 存在しない場合は、雛形 `devices/_TEMPLATE.sample.md` をコピーして `devices/<device>.sample.md` を
     作成するようユーザーに案内する（ゼロから書式を説明する必要はない。雛形のプレースホルダを埋めてもらう）
   - 「ドライバ追加済みブランチのパス」を必ず確認し、そこから型番・説明文・
     インターフェース種別・メソッド一覧を読み取る。入力ファイルで別途聞くのは、ドライバ側には無い情報
     （カテゴリー・商品URL・配線図の日本語補足・設計/配線図ファイルの場所・部品写真）のみに絞る
   - 設計・配線図ファイル（`.ai`/`.svg`/`.fzpz`/`.fzz`/`schematic.png`）や部品写真はClaudeCodeでは
     作成できないため、存在確認のみ行い、無ければユーザーに用意を依頼する
4. `CHIRIMEN_SAMPLE_CONTRIBUTION.md` の「3. 入力ファイルのチェック」「4. 生成対象ファイル一覧」
   「5. Git/GitHubワークフロー」に厳密に従って作業する

ドライバ本体（npmパッケージ）を `chirimen-drivers` リポジトリへ追加する作業は本スキルの対象外。
その依頼が来た場合は `chirimen-driver-contribution` skill（`CHIRIMEN_DRIVER_CONTRIBUTION.md`）を案内する。
