# CHIRIMEN ドライバ新規追加 — ClaudeCode作業手順書

このファイルは、[chirimen-drivers](https://github.com/chirimen-oh/chirimen-drivers) リポジトリへ
「新しいドライバを追加する」コントリビュートを、ClaudeCodeが半自動で支援するための作業手順を定義する。

参照元: https://www.chirimen.org/chirimen-drivers/CONTRIBUTING
（特に `docs/contributing/add-driver.html`, `coding-standards.html`, `repository.html`, `getting-started.html`）

> 部品写真・サンプルコード・部品リストの `chirimen.org` リポジトリへの登録は対象外。
> それらは [`CHIRIMEN_SAMPLE_CONTRIBUTION.md`](./CHIRIMEN_SAMPLE_CONTRIBUTION.md) を参照する
> （ドライバ本体とは別リポジトリ・別作業のため、必要な場合のみ実施する独立した作業）。

---

## ⚠️ 公式CONTRIBUTINGサイトとの相違点

以下は、公式サイト（`add-driver.html` 等）には明記されていないが、実際のコントリビュート運用として
このドキュメントに反映している内容。他のCHIRIMENコントリビューターの一般的な手順とは異なる可能性があるため
明示しておく。

1. **事前プロトタイピングの工程がある**: 公式ガイドは「`packages/hello-world` をコピーして直接編集する」
   という手順のみだが、実際には別の個人リポジトリ（実機I2C動作確認用のスケルトンリポジトリ）で
   先にデバイス単体のクラス実装と動作確認を行い、それを`chirimen-drivers`の`index.js`へ移植する
2. **Pull Requestが2段階になる**: 公式ガイドは「個人Fork → 本家(chirimen-oh)へ直接PR」のみだが、
   実際には「個人Fork → 社内ステージング用Fork（例: `KDDI-Technology/chirimen-drivers`）→ 本家」という
   2段階のPRチェーンを経由する。社内ステージングFork側の受け入れブランチは事前に用意されている必要がある
3. **ルート`README.md`は編集してはいけない**: リポジトリルートの`README.md`には、公開済みパッケージへの
   jsDelivrリンクを列挙した「Download」セクションがあり、新規パッケージ追加のたびに1行ずつ増えている。
   一見コントリビューターが更新すべきファイルに見えるが、実際には`.github/workflows/release.yml`内の
   自動化スクリプト（`update-readme-download.sh`）が`master`へのpush時に検知し、`github-actions[bot]`が
   自動でコミットする仕組みになっている（実際のコミット履歴で`github-actions[bot]`によるコミットを複数確認）。
   公式ガイドに記載が無いのは「記載漏れ」ではなく「コントリビューターの作業範囲外」であるため、
   **ClaudeCodeもこのファイルには一切手を加えない**

これらは公式ドキュメントに存在しない、組織固有・実務上の追加ワークフロー。実施前提が異なる別チームで
このガイドを流用する場合は、本セクションと「5. Git/GitHubワークフロー（人間が実施）」を実態に合わせて調整すること。

---

## 0. 絶対に守る制約（最優先）

ClaudeCodeは **読み取り以外のGit/GitHub操作を一切実行しない**。

**禁止（人間が実施する）:**
- `git add` / `git commit` / `git push`
- `git checkout -b` などのブランチ作成・切り替え
- GitHub上でのPull Request作成・Issueコメント等、GitHub APIを介した書き込み操作全般
- Fork作成、GitHub上でのブランチ新規作成などGitHub APIを介した操作全般
  （個人Fork・社内ステージングFork・本家、いずれのリポジトリに対しても同様）

**許可:**
- `chirimen-drivers`（個人Fork）リポジトリ配下のローカルファイルの新規作成・編集（今回の自動構成そのもの）
- プロトタイピングリポジトリ（`chirimen-i2c-tutorial-base`など）配下のファイルの**読み取り**
- `git status` / `git diff` / `git log` / `git ls-remote` などの**読み取り専用**Gitコマンド
  （変更内容の確認、リモートブランチの存在確認などの目的）
- `npm install`、`npx prettier --write` など、ローカルファイルの生成・整形を行うだけのビルド系コマンド

ClaudeCodeの作業はあくまで「作業ツリー上にファイルを用意するところまで」。
個人Fork・社内ステージングFork・本家の3リポジトリにまたがるコミット・プッシュ・PR作成は、
すべて必ず人間が行う。ClaudeCodeは完了時に、次に人間が行うべき手順を提示して終了する。

### 作業範囲まとめ

| 作業 | 担当 |
|---|---|
| 入力ファイルの読み取り・チェック（3章） | ClaudeCode |
| `packages/<device>/` 配下のファイル生成（4章） | ClaudeCode |
| ルート`README.md`の編集 | **誰も行わない**。`github-actions[bot]`が自動更新するため人間もClaudeCodeも触らない |
| 生成物へのPrettier整形（`npx prettier --write`） | ClaudeCode |
| 生成内容のレビュー・実機での動作確認 | 人間 |
| `git add` / `git commit` / `git push` | 人間 |
| ブランチ作成 | 人間 |
| 社内ステージングForkの受け入れブランチ作成 | 人間 |
| Pull Request作成（個人Fork→社内ステージングFork、社内ステージングFork→本家） | 人間 |
| レビュー対応・再PR | 人間 |

迷ったら「GitHub上のリポジトリの状態を変える操作は人間、ローカルの作業ツリーにファイルを
用意するだけの操作はClaudeCode」という基準で判断する。

---

## 1. 対象リポジトリと前提

今回のコントリビュートには、性質の異なる複数のリポジトリ（ローカルクローン）が登場する。

| リポジトリ | 役割 | ローカルクローン | 例 |
|---|---|---|---|
| プロトタイピングリポジトリ | 実機でのI2C動作確認用に、デバイス単体のクラス実装を書く個人リポジトリ。`chirimen-drivers`とは無関係の別リポジトリ | 必要（ClaudeCodeが読み取る） | `chirimen-i2c-tutorial-base`（フォーク） |
| 個人Fork（`chirimen-drivers`） | 自分のGitHubアカウントにForkした`chirimen-drivers`。ここに新しいパッケージを追加する | 必要（ClaudeCodeがファイルを生成する） | `https://github.com/<自分のアカウント>/chirimen-drivers` |
| 社内ステージングFork | 本家へPRする前に経由する組織のFork。事前に受け入れ用ブランチが用意されている前提。URLは組織固定値（下記参照） | **不要**。ClaudeCodeは`git ls-remote`でリモートの存在確認のみ行う。PR作成はGitHub上で人間が行う | `https://github.com/KDDI-Technology/chirimen-drivers` |
| 本家（upstream） | 最終的にPRする公式リポジトリ | **不要**。ローカルでの編集対象ではなく、PR作成もGitHub上で人間が行う | `https://github.com/chirimen-oh/chirimen-drivers` |

ClaudeCodeがローカルファイルの読み取り・生成を行うのは「プロトタイピングリポジトリ」と「個人Fork」の2つのみ。
社内ステージングFork・本家はGitHub上のリモートリポジトリとしてのみ登場し、ローカルにクローンしておく必要はない
（`git ls-remote <URL>` はクローンなしで実行できる）。

パスは作業時に人間から指示を受けるか、入力ファイル（後述）の「リポジトリのパス」セクションで指定する。
パスが不明・存在しない場合は推測で進めず、人間に確認する。

> **実行ディレクトリについて**: ClaudeCodeは特定のディレクトリで実行される前提はなく、
> どこで起動されても構わない。入力ファイル中のパスは**絶対パス**、またはその時点の
> ClaudeCodeのカレントディレクトリからの相対パスとして解釈する。相対パスを使う場合は、
> どこを基準にした相対パスかを人間が把握できる書き方にする（例: 実行時のカレントディレクトリからの
> 相対パス）。パスの意味が曖昧な場合は推測せず人間に確認する。

> 社内ステージングFork（`https://github.com/KDDI-Technology/chirimen-drivers`）のURLは、組織として
> 固定の値である。デバイスごとに変わるものではないため、入力ファイルにはこの値を記入させず、
> 本ガイド内の固定値として扱う。別チームがこのガイドを流用する場合は、この値を実際の組織のFork URLに
> 置き換えること（「⚠️ 公式CONTRIBUTINGサイトとの相違点」も参照）。

---

## 2. 入力ファイル定義

**1デバイス = 1 Markdownファイル**。雛形 [`devices/_TEMPLATE.driver.md`](./devices/_TEMPLATE.driver.md)
をコピーし、`devices/<device>.driver.md` として保存の上、プレースホルダ（`[ ]`で囲まれた部分）を
書き換えて人間が事前に用意する。ゼロから書式を覚えて作成する必要はない。

デバイス実装の主入力は「プロトタイピングリポジトリの既存コード」であり、この入力ファイルは
それを補うメタ情報（コード上に現れない情報）を提供する位置づけ。

### 雛形の構成

雛形は以下の見出し・項目で構成されている（詳細な記入例は雛形ファイル本体を参照）。

| 見出し | 項目 | 必須/任意 |
|---|---|---|
| 基本情報 | device / display_name / interface | 必須 |
| リポジトリのパス | プロトタイピングリポジトリのパス / 個人Forkした chirimen-drivers のローカルパス | 必須 |
| リポジトリのパス | index.jsの元にするファイル | 任意（未記入なら候補提示の上ユーザーに確認する） |
| I2Cアドレス | アドレス | interfaceがi2cの場合必須 |
| デバイスの説明 | （自由記述） | 必須 |
| 参考情報 | データシートURL / 参考にした実装元URL | 任意 |
| Fork先URL | 個人Fork | 必須（社内ステージングForkは組織で固定のURLのため入力不要。「1. 対象リポジトリと前提」参照） |

> ライセンスの入力欄は無い。`chirimen-drivers`リポジトリ自体がMITライセンスであり、実際に
> 既存パッケージのpackage.jsonも（licenseフィールドを記載しているものは）すべて`"MIT"`だったため
> （`packages/ads1015`, `packages/hello-world`, `packages/scd40`, `packages/grove-water-level-sensor`等で確認）、
> package.jsonの`license`は常に`"MIT"`固定として生成する。既存パッケージが本当に別ライセンスを
> 使いたい特殊なケースに気づいた場合のみ、推測で進めず人間に確認する。

ClaudeCodeがこのファイルを読み込む際は、Markdownとして厳密にパースするのではなく、
見出し・箇条書きの内容を読解して各項目の値を取り出す。プレースホルダの`[`が
書き換えずに残っている行は「未入力」として扱う。

---

## 3. 入力ファイルのチェック（生成前に必ず実施）

以下を満たさない場合は **ファイル生成を行わず、不備を人間に報告する**（推測で埋めない）。

- [ ] 「device」が小文字英数字・ハイフンのみで構成されている
- [ ] 「個人Forkした chirimen-drivers のローカルパス」`/packages/<device>/` がまだ存在しない（重複追加の防止）
- [ ] 「プロトタイピングリポジトリのパス」が存在し、読み取り可能である
- [ ] 「index.jsの元にするファイル」が記入されている場合、そのファイルが上記パス内に存在する
- [ ] 「index.jsの元にするファイル」が **未記入の場合**、プロトタイピングリポジトリ直下の `.js` / `.mjs` ファイル
      （`app.mjs` / `app-*.mjs` のような「サンプル・アプリ側」を除く）を一覧化し、
      **どれを`index.js`のベースにするかユーザーに確認する**。1つしか候補が無くても確認は省略しない
- [ ] 「interface」が `i2c` / `spi` / `gpio` のいずれかである
- [ ] interfaceが`i2c`の場合、「I2Cアドレス」セクションのアドレスが記入されている
- [ ] 「デバイスの説明」が空でない
- [ ] 「Fork先URL」の個人Forkが記入されている
- [ ] （読み取り専用）「個人Forkした chirimen-drivers のローカルパス」で
      `git remote get-url origin` を実行し、「Fork先URL」の個人Forkと一致するか確認する。
      一致しない場合は生成自体は続行してよいが、誤ったリモートへpushしないよう完了報告で
      明確に警告する（入力ファイルの記載ミス、または別のリポジトリを指定してしまっている可能性がある）
- [ ] プレースホルダ（`[`で始まる未書き換えの値）が必須項目に残っていないか確認し、
      残っていれば生成せず人間に確認する
- [ ] （読み取り専用）`git ls-remote https://github.com/KDDI-Technology/chirimen-drivers`
      （固定URL。「1. 対象リポジトリと前提」参照）で、想定するブランチ命名
      （後述「6. 完了報告フォーマット」のブランチ名案）が既に社内ステージングFork側に
      用意されているか確認し、無ければ完了報告時に警告として明記する
      （社内ステージングForkへのブランチ作成はGitHub操作であり、ClaudeCodeでは行えないため）
- [ ] 「データシートURL」が無い場合は警告（生成は続行してよいが、READMEに「データシート要確認」と明記する）

---

## 4. 生成対象ファイル一覧

| ファイル | 内容 |
|---|---|
| `packages/<device>/package.json` | 下記テンプレート参照 |
| `packages/<device>/index.js` | 「index.jsの元にするファイル」の内容を移植して生成 |
| `packages/<device>/README.md` | `index.js`の内容と入力ファイルのメタ情報から生成 |
| `package-lock.json`（ルート） | `npm install` 実行により自動更新（手動編集しない。実行が環境制限で失敗した場合の扱いは「生成後のフォーマット」参照） |

> ⚠️ 上記はコントリビューターが更新すべきファイルの一覧である。これ以外に更新が必要な
> 一覧・索引ファイルが存在しないか気づいた場合は、この一覧に追記して常に最新の状態を保つこと。
> 逆に、公式ガイド・実リポジトリのいずれにもコントリビューターが更新する根拠が無いファイル
> （例: ルート`README.md`。「⚠️ 公式CONTRIBUTINGサイトとの相違点」参照）は、たとえ関連しそうに
> 見えても生成対象に含めないこと。

### index.js の生成方法

「index.jsの元にするファイル」の内容を**そのまま忠実に**`packages/<device>/index.js`へ移植することを
原則とする。プロトタイピングリポジトリのコードは実機で動作確認済みであることが前提のため、ロジックを
作り変えたり要約したりしない。

移植の際、以下のような補完・修正が必要と思われる箇所に気づいた場合は、**ClaudeCode側で無断で追加・変更せず、
必ず人間に「これを追加/修正してよいか」を確認してから**反映する（該当箇所が無ければ確認は不要）:

- 先頭に `// @ts-check` が無い
- 未初期化アクセス時のエラーハンドリング（`if (this.xxxSlave == null) { throw new Error(...) }`）が
  無いメソッドがある（[コーディング規約](https://www.chirimen.org/chirimen-drivers/docs/contributing/coding-standards.html)参照）
- レジスタの意味・キャリブレーション式など、移植だけでは判断できない不明点がある

以下は確認不要でそのまま維持してよい:
- クラス外に定義された定数（例: `THRESHOLD`）
- `export default` されているクラス名（無理に`device`から再生成しない。移植元のコードが実装として正であるため）

「参考にした実装元URL」が入力ファイルに記入されている場合のみ、ファイル先頭にコメントで参照元を明記する
（例: `// Based from <参考にした実装元URL>`）。これは入力ファイルの記載内容をそのまま反映するだけなので確認不要。

### package.json テンプレート

```json
{
  "name": "@chirimen/<device>",
  "description": "CHIRIMEN driver for <display_name> sensor",
  "version": "1.0.0",
  "license": "MIT",
  "type": "module",
  "exports": "./index.js",
  "repository": {
    "type": "git",
    "url": "https://github.com/chirimen-oh/chirimen-drivers.git",
    "directory": "packages/<device>"
  },
  "publishConfig": {
    "access": "public"
  },
  "peerDependencies": {
    "node-web-i2c": "^1.1.51"
  }
}
```
- `peerDependencies`の`node-web-i2c`バージョンは、プロトタイピングリポジトリの`package.json`に
  `node-web-i2c`への依存記載があればそのバージョンを踏襲する。無ければテンプレートのデフォルト値を使う
- `interface` が `i2c` 以外の場合、`peerDependencies` は付けず、`chirimen-drivers/packages/` 内にある
  同じインターフェースを使う既存ドライバを1つ探して同様の依存関係に合わせる（推測で新しい依存名を作らない）。
  **該当する既存ドライバが見つからない場合は、推測で進めず人間に確認する**（`chirimen-drivers`には
  本ガイド作成時点でSPI/GPIOベースの実装済みドライバが存在しないため、この状況は十分起こり得る）

### README.md に含める項目
- デバイス概要（「デバイスの説明」ベース）
- 主な仕様（インターフェース種別・アドレス・型番）
- インストール方法（`npm install @chirimen/<device>`）
- 使用方法のコード例（`init()` → 各メソッド呼び出しの流れ。`index.js`から実際のメソッド名・
  引数を読み取って作成する。手が空いていればプロトタイピングリポジトリ内の対応する`app-*.mjs`も参考にする）
- APIリファレンス（`index.js`のメソッド一覧と、コード中のコメント・JSDocに基づく説明）
- データシートへのリンク（「データシートURL」）
- 参考実装へのリンク（「参考にした実装元URL」があれば記載）

### 生成後のフォーマット

[コーディング規約](https://www.chirimen.org/chirimen-drivers/docs/contributing/coding-standards.html)により
Prettier（スペース2個）でのフォーマットが規定されている。これはpre-commit hookやCIによる自動強制ではなく
コントリビューターが手動実行する想定のため、ファイル生成後にClaudeCode自身が以下を実行して整形する
（「0. 絶対に守る制約」の許可コマンドに該当する、ローカルファイルの整形のみの操作）:

```
npx prettier --write packages/<device>/
```

> 実行環境によっては（サンドボックス制限等により）この`npx prettier --write`や、後述の
> `npm install`が権限エラー（`EPERM`等）で失敗することがある。その場合は黙ってスキップしたり
> 成功したように装ったりせず、**完了報告で警告として明記し、コントリビューター自身のローカル環境で
> 改めて実行するよう案内する**。整形については、失敗時は既存パッケージのインデント幅（スペース2個）に
> 手動で合わせておいてよいが、あくまで代替であり実際のPrettier実行結果ではない旨も明記する。

---

## 5. Git/GitHubワークフロー（人間が実施）

ClaudeCodeはファイル生成までを行い、以下はすべて人間が実施する。個人Fork→社内ステージングFork→本家、
という2段階のPRチェーンになっている点に注意。

1. 個人Fork（入力ファイルの「個人Fork」URL）のローカルクローンでブランチを作成する（例: `feat/<device>`）
2. 生成された`packages/<device>/`と`package-lock.json`をコミットし、個人Forkへプッシュする
3. 社内ステージングFork（`https://github.com/KDDI-Technology/chirimen-drivers`。「1. 対象リポジトリと前提」参照）側に
   受け入れ用ブランチが**事前に**用意されていることを確認する（この確認自体は「3. 入力ファイルのチェック」で
   ClaudeCodeが`git ls-remote`により実施済みのはずである。完了報告で警告が出ていた場合は、ここで人間が
   GitHub上でブランチを作成する）
4. 個人Forkのブランチ → 社内ステージングForkの受け入れブランチへPull Requestを作成する
5. 社内ステージングFork側でマージされた後、社内ステージングFork → 本家（`chirimen-oh/chirimen-drivers`）へ
   Pull Requestを作成する
6. 本家でのレビューにより修正依頼があった場合は、**手順4（社内ステージングForkへのPR作成）からやり直す**
7. 本家レビューで問題なければマージされ、GitHub Actionsにより自動でnpmへ公開される

---

## 6. 完了報告フォーマット

- **コミット対象ファイル一覧**: 個人Forkのローカルクローンで `git status --porcelain`（読み取り専用）を実行し、
  新規作成・変更されたファイルを**相対パスで漏れなく**列挙する。そのまま使える形で
  `git add packages/<device>/package.json packages/<device>/index.js ...` のようなコマンド例も添える。
  この一覧を過不足なくコミットすれば、人間はそれ以上ファイルを作成せずコミット・プッシュに進める状態に
  なっていることを保証する（4章の一覧はあくまで種類の説明であり、本項目が実際にその回で
  変更された正確なファイルパスの一覧になる）
  - **完全性の自己チェック**: `git status --porcelain` の結果を「4. 生成対象ファイル一覧」と突き合わせ、
    一覧にある種類のファイルが実際に含まれているか確認する。含まれていないものがあれば、
    その旨をこの完了報告で明示的に警告する（例: `package-lock.json`の更新が`npm install`未実行のため
    含まれていない、など）。これはユーザー確認用の一覧であると同時に、ClaudeCode自身が生成漏れを
    検出するための工程でもある
- チェックで警告になった項目（データシート未確認、社内ステージングFork側の受け入れブランチ未作成など）
- 推奨コミットメッセージ案（Conventional Commits形式。例: `feat: add grove-water-level-sensor driver`）
- 推奨ブランチ名案（例: `feat/grove-water-level-sensor`）
- 人間が次に行うべき手順:
  1. 生成内容のレビュー・実機での動作確認（[テストについて](https://www.chirimen.org/chirimen-drivers/docs/contributing/testing.html)に基づき自動テストは無いため実機確認が必須）
  2. 上記「コミット対象ファイル一覧」の内容で「5. Git/GitHubワークフロー」に従って
     個人Fork→社内ステージングFork→本家の順にコミット・プッシュ・PR作成を行う
  3. （任意）部品写真・サンプルコード・部品リストへの登録も行う場合は
     [`CHIRIMEN_SAMPLE_CONTRIBUTION.md`](./CHIRIMEN_SAMPLE_CONTRIBUTION.md) を別途参照する

---

## 7. 参考ページ一覧

- [はじめに・基本ルール](https://www.chirimen.org/chirimen-drivers/docs/contributing/getting-started.html)
- [開発環境のセットアップ](https://www.chirimen.org/chirimen-drivers/docs/contributing/setup.html)
- [新しいドライバの追加](https://www.chirimen.org/chirimen-drivers/docs/contributing/add-driver.html)
- [このリポジトリのしくみ](https://www.chirimen.org/chirimen-drivers/docs/contributing/repository.html)
- [コーディング規約](https://www.chirimen.org/chirimen-drivers/docs/contributing/coding-standards.html)
- [テストについて](https://www.chirimen.org/chirimen-drivers/docs/contributing/testing.html)
- [付録（package.json / index.js テンプレート）](https://www.chirimen.org/chirimen-drivers/docs/contributing/appendix.html)
- 参考にした実例: [chirimen-oh/chirimen-drivers の grove-water-level-sensor パッケージ](https://github.com/chirimen-oh/chirimen-drivers/tree/master/packages/grove-water-level-sensor)
