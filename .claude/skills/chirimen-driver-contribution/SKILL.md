---
name: chirimen-driver-contribution
description: Use when the user wants to add/contribute a new device driver to the chirimen-drivers repository (CHIRIMEN's I2C/SPI/GPIO sensor driver npm packages). Triggers include "CHIRIMENの新しいドライバを追加して", "〜のドライバをコントリビュートしたい", "chirimen-driversにパッケージを追加して", or a request to create a new package under chirimen-drivers/packages/.
---

# CHIRIMEN ドライバ新規追加

新しいCHIRIMENドライバを `chirimen-drivers` リポジトリへ追加する作業の詳細手順は、
プロジェクトルートの `CHIRIMEN_DRIVER_CONTRIBUTION.md` に定義されている。

## 実施手順

1. まず `CHIRIMEN_DRIVER_CONTRIBUTION.md` を読み込む（冒頭の「公式CONTRIBUTINGサイトとの相違点」も含めて）
2. 特に「0. 絶対に守る制約」（個人Fork/社内ステージングFork/本家、いずれに対してもgit commit/push/PR作成
   などの書き込み操作を一切行わない）を最優先で遵守する
3. 入力ファイル `devices/<device>.driver.md` の有無を確認する
   - 存在しない場合は、雛形 `devices/_TEMPLATE.driver.md` をコピーして `devices/<device>.driver.md` を
     作成するようユーザーに案内する（ゼロから書式を説明する必要はない。雛形のプレースホルダを埋めてもらう）
   - 「index.jsの元にするファイル」が未記入、または候補が複数ある場合は、
     必ずユーザーに選んでもらう。ClaudeCodeが推測で決めない
4. `CHIRIMEN_DRIVER_CONTRIBUTION.md` の「3. 入力ファイルのチェック」「4. 生成対象ファイル一覧」
   「5. Git/GitHubワークフロー」に厳密に従って作業する（index.jsは「index.jsの元にするファイル」の
   内容を忠実に移植して生成し、要約や書き換えは行わない）

部品写真・サンプルコード・部品リストの `chirimen.org` リポジトリへの登録は本スキルの対象外。
その依頼が来た場合は `chirimen-sample-contribution` skill（`CHIRIMEN_SAMPLE_CONTRIBUTION.md`）を案内する。
