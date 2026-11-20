## CHIRIMEN OSSコントリビュート作業

<!--
  このセクションは他プロジェクトのCLAUDE.mdへそのまま追記可能な独立ブロックとして書いている。
  Gitで管理せずファイル単体で配布する場合の取り込み方:
  - 配布先にCLAUDE.mdが無ければ、このファイルをそのまま `CLAUDE.md` として保存する
  - 配布先に既存のCLAUDE.mdがあれば、既存ファイルの末尾にこのセクション（このコメントを含む
    "## CHIRIMEN OSSコントリビュート作業" 以下）をそのまま貼り付ける。上書きはしないこと
-->

このリポジトリには、[CHIRIMEN](https://chirimen.org/) プロジェクトへのOSSコントリビュート作業を
ClaudeCodeが半自動で支援するための手順書がある。関連する依頼を受けたら、まず該当するSkill／
手順書ファイルを読み込んでから作業すること。

| 依頼内容 | 参照するSkill | 手順書 |
|---|---|---|
| chirimen-drivers リポジトリへの新しいドライバ追加 | `chirimen-driver-contribution` | `CHIRIMEN_DRIVER_CONTRIBUTION.md` |
| chirimen.org リポジトリへのサンプルコード・部品リスト・部品写真の登録 | `chirimen-sample-contribution` | `CHIRIMEN_SAMPLE_CONTRIBUTION.md` |

**共通の制約**: どちらの作業でも、`git commit` / `git push` / GitHub上でのPR作成など、
読み取り以外のGit/GitHub操作は一切行わない。ローカルファイルの生成・編集はそこまでを行い、
コミット以降は必ず人間が実施する（詳細は各手順書の「0. 絶対に守る制約」を参照）。
