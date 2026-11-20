# CHIRIMEN コントリビュート自動化キット

ClaudeCodeを使って、[CHIRIMEN](https://chirimen.org/) プロジェクトへのOSSコントリビュート作業
（新しいドライバの追加、サンプルコード・部品情報の登録）を半自動で支援するためのファイル一式です。

このファイル一式はGitで管理せず、ファイルそのものをコピーして配布する前提で作られています。
以下の「配置方法」に従って、ご自身の作業用プロジェクトに配置してください。

---

## 対応している作業

| 依頼内容 | 参照するガイド |
|---|---|
| `chirimen-drivers` リポジトリへの新しいドライバ追加 | `CHIRIMEN_DRIVER_CONTRIBUTION.md` |
| `chirimen.org` リポジトリへのサンプルコード・部品リスト・部品写真の登録 | `CHIRIMEN_SAMPLE_CONTRIBUTION.md` |

いずれも、ClaudeCodeは**ファイルの生成まで**を行い、`git commit` / `git push` / GitHub上でのPR作成など
読み取り以外のGit/GitHub操作は一切行いません。それらは必ず人間が行います（詳細は後述）。

---

## ファイル一覧

配布・コピーする際は、以下のファイル・ディレクトリ構成を**そのまま維持**してください
（ガイド同士が相対パスで参照し合っているため、配置がずれると正しく動作しません）。

```
（プロジェクトルート）
├── README.md                          ← このファイル
├── CLAUDE.md                          ← ClaudeCodeへの短い道しるべ（要・取り込み手順あり）
├── CHIRIMEN_DRIVER_CONTRIBUTION.md    ← ドライバ追加の詳細手順書
├── CHIRIMEN_SAMPLE_CONTRIBUTION.md    ← サンプル・部品情報登録の詳細手順書
├── devices/
│   ├── _TEMPLATE.driver.md            ← ドライバ追加の入力シート雛形
│   └── _TEMPLATE.sample.md            ← サンプル・部品情報登録の入力シート雛形
└── .claude/
    └── skills/
        ├── chirimen-driver-contribution/SKILL.md
        └── chirimen-sample-contribution/SKILL.md
```

| ファイル | 役割 |
|---|---|
| `CLAUDE.md` | ClaudeCodeが2つのSkill・ガイドを自動認識するための短い道しるべ |
| `CHIRIMEN_DRIVER_CONTRIBUTION.md` | ドライバ追加作業の全手順（制約・入力定義・チェック・生成方法・Git/GitHubワークフロー） |
| `CHIRIMEN_SAMPLE_CONTRIBUTION.md` | サンプル・部品情報登録作業の全手順（同上） |
| `.claude/skills/chirimen-driver-contribution/SKILL.md` | 「ドライバを追加したい」という依頼を自動認識し、上記ガイドへ誘導するSkill |
| `.claude/skills/chirimen-sample-contribution/SKILL.md` | 「サンプル・部品情報を登録したい」という依頼を自動認識し、上記ガイドへ誘導するSkill |
| `devices/_TEMPLATE.driver.md` | ドライバ追加時に人間が事前に埋める入力シートの雛形 |
| `devices/_TEMPLATE.sample.md` | サンプル・部品情報登録時に人間が事前に埋める入力シートの雛形 |

---

## 配置方法

1. 上記「ファイル一覧」のディレクトリ構成を保ったまま、作業用プロジェクトのルートに配置する
2. **`CLAUDE.md`だけは特別対応が必要**です。作業用プロジェクトに既存の`CLAUDE.md`が無ければ、
   そのまま保存してください。既に`CLAUDE.md`がある場合は、上書きせず、ファイル冒頭に記載されている
   HTMLコメントの指示に従って、既存ファイルの末尾に本セクションを追記してください
   （複数人の成果物を1つのCLAUDE.mdにまとめても安全に追記できる形式になっています）
3. `devices/`ディレクトリは、各自の作業デバイスごとの入力ファイル（`<device>.driver.md`など）を
   置いていく場所になります。雛形2つ以外は空の状態で配布して問題ありません

---

## 事前に必要なもの

作業を始める前に、以下を各自でご用意ください（ClaudeCodeが代わりに用意することはできません）。

- GitHubアカウント、および `chirimen-drivers` / `chirimen.org` の**個人Fork**
- 個人Forkの**ローカルクローン**（ドライバ追加なら`chirimen-drivers`、サンプル登録なら`chirimen.org`）
- （ドライバ追加の場合のみ）実機で動作確認済みのデバイスクラス実装がある、**プロトタイピング用の
  個人リポジトリ**（`chirimen-drivers`本体とは別の、自分の検証用リポジトリ）
- 社内ステージングFork（`KDDI-Technology`組織）へのアクセス権（PRを作成する段階で必要）

---

## 使い方

### 1. 新しいドライバを追加したい場合

1. `devices/_TEMPLATE.driver.md` をコピーして `devices/<device>.driver.md` として保存し、
   `[ ]` で囲まれたプレースホルダを埋める
2. ClaudeCodeに自然な言葉で依頼する。例:
   > 「CHIRIMENに新しいI2Cドライバをコントリビュートしたいです。デバイス名は`<device>`です。」
3. ClaudeCodeが入力ファイルを読み取り・チェックし、不明点があれば確認しながら
   `packages/<device>/`配下のファイル一式を生成する
4. 完了報告（コミット対象ファイル一覧・警告事項・次に人間が行う手順）に従って、
   実機での動作確認 → コミット・プッシュ → PR作成を行う

### 2. サンプルコード・部品情報を登録したい場合

1. `devices/_TEMPLATE.sample.md` をコピーして `devices/<device>.sample.md` として保存する
   （すでに追加済みの「ドライバ追加済みブランチのパス」を指定すると、多くの情報がそこから自動転記されます）
2. ClaudeCodeに自然な言葉で依頼する。例:
   > 「CHIRIMENに、追加済みの`<device>`ドライバのサンプルコードと部品情報を登録したいです。」
3. 以降は1と同様に、チェック→生成→完了報告の流れになる

どちらの作業も、ファイルパスを明示しなくてもClaudeCodeが自動的に該当ガイドを見つけて実行します
（依頼内容に応じて自動的にSkillが発動するため）。

---

## ClaudeCodeが行うこと／行わないこと

| 作業 | 担当 |
|---|---|
| 入力ファイルの読み取り・不備チェック | ClaudeCode |
| ソースコード・`package.json`・`README.md`等の生成、Prettier整形 | ClaudeCode |
| 設計・配線図ファイルや部品写真の**作成そのもの** | 人間（ClaudeCodeは存在確認のみ） |
| 生成内容のレビュー・実機での動作確認 | 人間 |
| `git add` / `git commit` / `git push` | 人間 |
| ブランチ作成、社内ステージングFork側の受け入れブランチ作成 | 人間 |
| Pull Request作成（個人Fork→社内ステージングFork→本家という2段階） | 人間 |

ClaudeCodeが行う読み取り専用のGit操作（`git status` / `git diff` / `git log` / `git ls-remote`など）を
除き、リポジトリの状態を変える操作は一切行いません。完了時には必ず「次に人間が行うべき手順」を
提示して作業を終えます。

---

## 補足

- 実際のCHIRIMENコントリビュート手順は、[公式CONTRIBUTINGサイト](https://www.chirimen.org/chirimen-drivers/CONTRIBUTING)
  をベースにしていますが、公式サイトには明記されていない実務上の手順（プロトタイピング工程、
  社内ステージングForkを経由する2段階PR、import文の変換ルールなど）も含まれています。詳細は各ガイド冒頭の
  「⚠️ 公式CONTRIBUTINGサイトとの相違点」を参照してください
- 別チーム・別組織でこのキットを流用する場合は、社内ステージングFork（`KDDI-Technology`）のURLなど、
  組織固有の値をガイド内の該当箇所で置き換えてください
