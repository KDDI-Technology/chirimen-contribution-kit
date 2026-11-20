# CHIRIMEN サンプルコード・部品情報登録 — ClaudeCode作業手順書

このファイルは、[chirimen.org](https://github.com/chirimen-oh/chirimen.org) リポジトリへ
デバイスの「サンプルコード・部品リスト・部品写真・配線図」を登録するコントリビュートを、ClaudeCodeが
半自動で支援するための作業手順を定義する。

参照元: https://www.chirimen.org/chirimen-drivers/CONTRIBUTING の
「新しいドライバの追加」内オプション手順（`docs/contributing/add-driver.html`）

> ドライバ本体（npmパッケージ）を `chirimen-drivers` リポジトリへ追加する作業は対象外。
> それは [`CHIRIMEN_DRIVER_CONTRIBUTION.md`](./CHIRIMEN_DRIVER_CONTRIBUTION.md) を参照する。
> 本作業はドライバ追加とは別リポジトリ・別作業であり、ドライバ追加後に独立して実施してよい
> （元記事でも「オプション手順」として位置づけられている）。ただし本ガイドの入力は、
> ドライバ追加作業で作成済みのブランチ（`packages/<device>/`）を前提にしている。

---

## ⚠️ 公式CONTRIBUTINGサイトとの相違点・補足

公式サイトの説明では `pizero/src/esm-examples/<device>/` に `main.js` と `readme.md` を置く、
としか書かれていないが、実際の既存サンプル（例:
[`grove-water-level-sensor`](https://github.com/chirimen-oh/chirimen.org/tree/master/pizero/src/esm-examples/grove-water-level-sensor)）
を確認したところ、以下が判明したため本ガイドに反映している。

1. **`package.json` も必要**: `main.js` / `readme.md` に加え、そのサンプル用の`package.json`
   （`@chirimen/<device>` 等への依存を記載）も同ディレクトリに存在する
2. **設計・配線図ファイル一式が必要**: 配線図の画像（`schematic.png`）に加え、`.svg` / `.fzpz`
   （Fritzingパーツ）/ `.fzz`（Fritzing配線図）や、そのSVGの元になった編集用デザインファイル
   （Illustratorの`.ai`とは限らず、使用ツールに応じて`.fig` / `.pdf` / `.psd`等さまざまな形式があり得る）
   が含まれる。これらは**ClaudeCodeでは作成できない**（画像生成・Fritzing/デザインファイル作成は
   スコープ外）ため、人間が用意したファイルを配置するのみとする。配置先は一律ディレクトリ直下ではなく、
   `schematic.png`のみディレクトリ直下、それ以外（拡張子を問わず）は`files/`サブディレクトリの中に置く
   （`grove-water-level-sensor`の実際の構成に合わせている。詳細は「4. 生成対象ファイル一覧」参照）
3. **import文の変換が必要な場合がある**: ドライバ追加時のプロトタイピングリポジトリ
   （`CHIRIMEN_DRIVER_CONTRIBUTION.md`参照）にある動作確認用アプリ（`app-<device>.mjs`など）を
   `main.js`の元にする場合、`import { requestI2CAccess } from "chirimen";` のような書き方になっていることがあるが、
   chirimen.org側の`main.js`では `import { requestI2CAccess } from "node-web-i2c";` を使う。
   この変換ルールは公式ガイドに明記されていないため、本ガイドで明示する
4. **readme.mdの定型フォーマット**: 既存サンプルのreadme.mdは「仕様 / 配線図 / インストール方法 /
   ファイル説明 / メソッドごとのサンプルコード説明 / 参考URL」という構成になっている

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
- `chirimen.org`（個人Fork）リポジトリ配下のローカルファイルの新規作成・編集（今回の自動構成そのもの）
- `chirimen-drivers`（ドライバ追加作業で作成済みのブランチ）配下のファイルの**読み取り**
- `git status` / `git diff` / `git log` / `git ls-remote` などの**読み取り専用**Gitコマンド

ClaudeCodeの作業はあくまで「作業ツリー上にファイルを用意するところまで」。
個人Fork・社内ステージングFork・本家の3リポジトリにまたがるコミット・プッシュ・PR作成は、
すべて必ず人間が行う。ClaudeCodeは完了時に、次に人間が行うべき手順を提示して終了する。

### 作業範囲まとめ

| 作業 | 担当 |
|---|---|
| 入力ファイルの読み取り・チェック（3章） | ClaudeCode |
| `pizero/src/esm-examples/<device>/` 配下のファイル生成、CSV追記（4章） | ClaudeCode |
| 設計・配線図ファイル / 部品写真の**用意**（作成そのもの） | 人間（ClaudeCodeは存在確認のみ） |
| 生成内容のレビュー・実機での動作確認 | 人間 |
| `git add` / `git commit` / `git push` | 人間 |
| ブランチ作成 | 人間 |
| 社内ステージングForkの受け入れブランチ作成 | 人間 |
| Pull Request作成（個人Fork→社内ステージングFork、社内ステージングFork→本家） | 人間 |
| レビュー対応・再PR | 人間 |

迷ったら「GitHub上のリポジトリの状態を変える操作、または画像・図面などClaudeCodeが生成できない
成果物の作成は人間、ローカルの作業ツリーにテキストファイルを用意するだけの操作はClaudeCode」
という基準で判断する。

---

## 1. 対象リポジトリと前提

| リポジトリ | 役割 | ローカルクローン | 例 |
|---|---|---|---|
| ドライバ追加済みブランチ（`chirimen-drivers`） | 本ガイドの主入力。`packages/<device>/`から情報を読み取る | 必要（ClaudeCodeが読み取る） | `CHIRIMEN_DRIVER_CONTRIBUTION.md`で作成したもの |
| 個人Fork（`chirimen.org`） | 自分のGitHubアカウントにForkした`chirimen.org`。ここにサンプル・部品情報を追加する | 必要（ClaudeCodeがファイルを生成する） | `https://github.com/<自分のアカウント>/chirimen.org` |
| 社内ステージングFork | 本家へPRする前に経由する組織のFork。事前に受け入れ用ブランチが用意されている前提。URLは組織固定値（下記参照） | **不要**。ClaudeCodeは`git ls-remote`でリモートの存在確認のみ行う。PR作成はGitHub上で人間が行う | `https://github.com/KDDI-Technology/chirimen.org` |
| 本家（upstream） | 最終的にPRする公式リポジトリ | **不要**。ローカルでの編集対象ではなく、PR作成もGitHub上で人間が行う | `https://github.com/chirimen-oh/chirimen.org` |

ClaudeCodeがローカルファイルの読み取り・生成を行うのは「ドライバ追加済みブランチ」と「個人Fork」の2つのみ。
社内ステージングFork・本家はGitHub上のリモートリポジトリとしてのみ登場し、ローカルにクローンしておく必要はない
（`git ls-remote <URL>` はクローンなしで実行できる）。

パスは作業時に人間から指示を受けるか、入力ファイル（後述）の「基本情報」セクションで指定する。
パスが不明・存在しない場合は推測で進めず、人間に確認する。

> **実行ディレクトリについて**: ClaudeCodeは特定のディレクトリで実行される前提はなく、
> どこで起動されても構わない。入力ファイル中のパスは**絶対パス**、またはその時点の
> ClaudeCodeのカレントディレクトリからの相対パスとして解釈する。相対パスを使う場合は、
> どこを基準にした相対パスかを人間が把握できる書き方にする（例: 実行時のカレントディレクトリからの
> 相対パス）。パスの意味が曖昧な場合は推測せず人間に確認する。

> 社内ステージングFork（`https://github.com/KDDI-Technology/chirimen.org`）のURLは、組織として
> 固定の値である。デバイスごとに変わるものではないため、入力ファイルにはこの値を記入させず、
> 本ガイド内の固定値として扱う。別チームがこのガイドを流用する場合は、この値を実際の組織のFork URLに
> 置き換えること。

---

## 2. 入力ファイル定義

**1デバイス = 1 Markdownファイル**。雛形 [`devices/_TEMPLATE.sample.md`](./devices/_TEMPLATE.sample.md)
をコピーし、`devices/<device>.sample.md` として保存の上、プレースホルダ（`[ ]`で囲まれた部分）を
書き換えて人間が事前に用意する。ゼロから書式を覚えて作成する必要はない。

デバイス名・インターフェース種別・説明文・メソッド仕様などは、可能な限り「ドライバ追加済みブランチのパス」
から自動転記する。この入力ファイルで新たに記入が必要なのは、ドライバ側には存在しない情報
（カテゴリー・商品URL・配線図補足・設計/配線図ファイル・部品写真等）のみ。

### 雛形の構成

雛形は以下の見出し・項目で構成されている（詳細な記入例は雛形ファイル本体を参照）。

| 見出し | 項目 | 必須/任意 |
|---|---|---|
| 基本情報 | device / ドライバ追加済みブランチのパス / 個人Forkした chirimen.org のローカルパス | 必須 |
| 部品リスト用の情報 | カテゴリー | 必須 |
| 部品リスト用の情報 | 商品URL | 任意 |
| サンプルコードの元ネタ | 動作確認用アプリのパス | 任意（未記入ならドライバのREADMEから叩き台を作成） |
| 配線・仕様の補足説明 | （自由記述） | 必須 |
| 設計・配線図ファイル / 部品写真 | 設計・配線図ファイルを置いたローカルディレクトリ / 部品写真のパス | 任意（無ければ完了報告で未登録として案内） |
| Fork先URL | 個人Fork | 必須（社内ステージングForkは組織で固定のURLのため入力不要。「1. 対象リポジトリと前提」参照） |

ClaudeCodeがこのファイルを読み込む際は、Markdownとして厳密にパースするのではなく、
見出し・箇条書きの内容を読解して各項目の値を取り出す。プレースホルダの`[`が
書き換えずに残っている行は「未入力」として扱う。

---

## 3. 入力ファイルのチェック（生成前に必ず実施）

以下を満たさない場合は **ファイル生成を行わず、不備を人間に報告する**（推測で埋めない）。

- [ ] 「device」が小文字英数字・ハイフンのみで構成されている
- [ ] 「ドライバ追加済みブランチのパス」が存在し、`package.json` / `index.js` / `README.md` を含んでいる
- [ ] 「ドライバ追加済みブランチのパス」のディレクトリ名と、「device」が一致する
- [ ] 「個人Forkした chirimen.org のローカルパス」`/pizero/src/esm-examples/<device>/` がまだ存在しない（重複追加の防止）
- [ ] 同パス`/_data/partslist.csv` に同じ型番の行がまだ存在しない。`partslist.csv`の「型番」列は
      `BME280`のような表示用の型番表記であり、`package.json`の`name`（`@chirimen/<device>`という
      npmスコープ付き名）とは書式が異なるため、比較には後述「表示用型番（ドライバ側の`display_name`相当）
      の取得方法」で復元した表示用型番を使う
- [ ] 「カテゴリー」「配線・仕様の補足説明」が空でない
- [ ] 「動作確認用アプリのパス」が記入されている場合、実際にそのパスにファイルが存在する
- [ ] 「設計・配線図ファイルを置いたローカルディレクトリ」が記入されている場合、そのディレクトリの中身を確認する。
      拡張子を限定せず、ディレクトリ内に存在する**すべてのファイル**が配置対象になる
      （`.ai`はあくまで一例であり、SVGの元になった編集用ファイルがどの拡張子かは限定しない）。
      `schematic.png`が無ければ配線図が未登録として、`schematic.png`以外に1つもファイルが無ければ
      設計ファイル一式が未登録として、それぞれ完了報告で警告し、生成自体は続行する
- [ ] 「部品写真のパス」が記入されている場合、実際にそのパスにファイルが存在する
- [ ] （読み取り専用）「個人Forkした chirimen.org のローカルパス」で
      `git remote get-url origin` を実行し、「Fork先URL」の個人Forkと一致するか確認する。
      一致しない場合は生成自体は続行してよいが、誤ったリモートへpushしないよう完了報告で
      明確に警告する（入力ファイルの記載ミス、または別のリポジトリを指定してしまっている可能性がある）
- [ ] プレースホルダ（`[`で始まる未書き換えの値）が必須項目に残っていないか確認し、
      残っていれば生成せず人間に確認する
- [ ] 「商品URL」が無い場合は警告（生成は続行してよいが、空欄のまま登録する）
- [ ] （読み取り専用）`git ls-remote https://github.com/KDDI-Technology/chirimen.org`
      （固定URL。「1. 対象リポジトリと前提」参照）で、想定するブランチ命名
      （後述「6. 完了報告フォーマット」のブランチ名案）が既に社内ステージングFork側に
      用意されているか確認し、無ければ完了報告時に警告として明記する
      （社内ステージングForkへのブランチ作成はGitHub操作であり、ClaudeCodeでは行えないため）

---

## 4. 生成対象ファイル一覧

| ファイル | 操作 |
|---|---|
| `pizero/src/esm-examples/<device>/main.js` | 新規作成。動作するサンプルコード |
| `pizero/src/esm-examples/<device>/package.json` | 新規作成。下記テンプレート参照 |
| `pizero/src/esm-examples/<device>/readme.md` | 新規作成。下記フォーマット参照 |
| `pizero/src/esm-examples/<device>/schematic.png` | 「設計・配線図ファイルを置いたローカルディレクトリ」内に存在すればコピー |
| `pizero/src/esm-examples/<device>/files/`（`schematic.png`以外の全ファイル） | 「設計・配線図ファイルを置いたローカルディレクトリ」内の`schematic.png`以外のファイルを、拡張子を問わずそのまま`files/`サブディレクトリを新規作成してコピー（存在するもののみ） |
| `pizero/src/esm-examples/index_examples.csv` | 末尾に1行追記。列は `ID,機能説明,詳細説明` |
| `_data/partslist.csv` | 末尾に1行追記（下記フォーマット厳守） |
| `partsImgs/<device>.jpg` | 「部品写真のパス」で指定された画像をコピー（存在する場合のみ。ファイル名は下記参照） |

> `partsImgs/`配下の既存ファイル名は表記が統一されておらず（省略形・スペース無しのCamelCase等）、
> 命名規則を機械的に再現することはできない。そのため本ガイドでは決定的に扱える`<device>`
> （デバイススラッグ）をファイル名に使う。既存の見た目に合わせたい場合は、人間が後から
> リネームして`_data/partslist.csv`の画像URL列も合わせて修正すること。

> 「設計・配線図ファイルを置いたローカルディレクトリ」自体は、人間が用意しやすいようフラットな
> 1つのフォルダのままでよい（`files/`のようなサブディレクトリに事前に分けてもらう必要はない）。
> フォルダ内のどれが`schematic.png`でどれがそれ以外かを拡張子で判別し、コピー先を
> 振り分けるのはClaudeCode側の仕事とする。

### 表示用型番（ドライバ側の`display_name`相当）の取得方法

readme.mdのタイトルや`_data/partslist.csv`の重複チェックのために、人間が読める表示用の型番
（`CHIRIMEN_DRIVER_CONTRIBUTION.md`の入力ファイルにあった`display_name`相当の文字列）が必要になる。
ただしこの値自体はどのファイルにも保存されないため、以下の順で復元する:

1. ドライバ側`package.json`の`description`フィールドは `"CHIRIMEN driver for <display_name> sensor"`
   という定型文になっている（`CHIRIMEN_DRIVER_CONTRIBUTION.md`のpackage.jsonテンプレート参照）ため、
   ここから`<display_name>`部分を抽出する
2. 抽出できない場合（descriptionが定型文でない等）は、ドライバ側`README.md`のタイトル行（先頭の`#`見出し）
   から判断する
3. それでも判断できない場合は、推測で埋めず人間に確認する

### 設計・配線図ファイル・部品写真のコピー方法

これらは画像・Fritzing・各種デザインツール形式等の**バイナリファイル**である。テキスト編集用のツール
（Read/Write/Edit）で読み書きすると、テキストとして解釈される過程でバイナリ内容が破損する
おそれがあるため、**必ずファイルコピー用のコマンド（`cp`）で複製する**こと。

コピー先はファイル名で振り分ける（`grove-water-level-sensor`の実際の構成に合わせている）:
- `schematic.png` → `pizero/src/esm-examples/<device>/schematic.png`（ディレクトリ直下）
- それ以外のファイル（`.svg` / `.fzpz` / `.fzz` や、SVGの元になった編集用デザインファイルなど。
  拡張子は問わず、元のファイル名のまま） → `pizero/src/esm-examples/<device>/files/`
  （`files/`ディレクトリが無ければ新規作成してから配置する）

環境の制約（サンドボックス等）により`cp`でのコピー自体が失敗することもある。その場合は
`npx prettier --write`等と同様に、黙ってスキップしたり成功したように装ったりせず、
**完了報告で警告として明記し、コピー元・コピー先の絶対パスを含む手動コピーコマンドを
人間向けに提示する**。あわせて、コピーが完了していないにもかかわらずreadme.md側で
「配置済み」であるかのような記載になっていないか確認し、コピー失敗時は該当箇所に
「（コピー未完了。手動でのコピー後に有効になります）」等の注記を添える
（詳細は「readme.md のフォーマット」の3・5参照）。

### main.js の生成方法

1. 「動作確認用アプリのパス」が記入されている場合、その内容をベースにする。**必ず**
   `import ... from "chirimen"`（または類似のNode向けchirimenパッケージimport）を、
   ドライバの`interface`に応じたパッケージ（下記参照）からのimportに置き換える
2. 「動作確認用アプリのパス」が未記入の場合、ドライバ側`README.md` の使用方法セクションを元に、
   `init()` → 各メソッド呼び出し → `console.dir`等での結果表示、という流れのサンプルを作成する
3. ドライバのインポートは `import <ClassName> from "@chirimen/<device>";` とする
4. アクセスAPIのimport元は、ドライバ側`package.json`から読み取った`interface`により決まる:
   - `i2c`の場合: `import { requestI2CAccess } from "node-web-i2c";`
   - `spi`/`gpio`の場合: `pizero/src/esm-examples/`内にある同じインターフェースを使う既存サンプルを
     1つ探し、そのimport文・パッケージ名に合わせる（推測で新しいパッケージ名を作らない。
     見つからない場合は人間に確認する）

### package.json テンプレート

```json
{
  "name": "<device>",
  "type": "module",
  "version": "1.0.0",
  "description": "<device> Example",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "license": "MIT",
  "dependencies": {
    "@chirimen/<device>": "^1.0.0",
    "node-web-i2c": "^1.1.51"
  }
}
```
- バージョン番号は既存の類似サンプルの記載に合わせる（不明な場合は`^1.0.0`のままでよい）
- `node-web-i2c`への依存は`interface`が`i2c`の場合のみ付ける。`spi`/`gpio`の場合は
  「main.js の生成方法」で特定した既存サンプルと同じ依存パッケージに合わせる
- `node-web-i2c`のバージョンは、ドライバ側`package.json`の`peerDependencies`に記載があれば
  そのバージョンを踏襲する（ドライバとサンプルでバージョン範囲が食い違わないようにするため）。
  記載が無ければテンプレートのデフォルト値のままでよい

### readme.md のフォーマット

既存サンプル（`grove-water-level-sensor`）を参考にした定型フォーマット。以下の順で記載する。

1. タイトル（「表示用型番（ドライバ側の`display_name`相当）の取得方法」で復元した表示用型番）
2. **センサー仕様**: ドライバ側`README.md`の仕様欄・「カテゴリー」・「配線・仕様の補足説明」をもとに箇条書き
3. **配線図**: 「設計・配線図ファイルを置いたローカルディレクトリ」に`schematic.png`が存在すれば
   `![配線図](./schematic.png "schematic")` を掲載。存在しなければ「配線図は別途追加予定」等、未登録である旨を明記する。
   コピー元には存在するがコピー自体が失敗した場合（前述「設計・配線図ファイル・部品写真のコピー方法」参照）は、
   画像記法自体は残しつつ「（コピー未完了。手動でのコピー後に有効になります）」と明記する
4. **ドライバのインストール**: `npm i @chirimen/<device>`
5. **ファイル説明**: `files/`ディレクトリにコピーしたファイル（`schematic.png`は対象外。「配線図」で
   別途掲載済みのため）を、リンクや`files/`というパス表記は付けず**ファイル名のみ**を箇条書きの見出しにして、
   それぞれの用途・対応アプリを記載する（`grove-water-level-sensor`のreadme.md実例に合わせた形式。
   コピー元に存在しないファイルは書かない）。コピー自体に失敗したファイルは、一覧には含めつつ
   「（コピー未完了）」と明記し、実際にはまだ配置されていないことが分かるようにする
6. **サンプルコード説明**: `index.js`のメソッドごとに、シグネチャと役割を説明する
   （ドライバ側`README.md`のAPIリファレンスを流用可）
7. **参考URL**: npm（`https://www.jsdelivr.com/package/npm/@chirimen/<device>`）、
   「商品URL」、ドライバ側の「参考にした実装元URL」

### `pizero/src/esm-examples/index_examples.csv` の列
```
ID,機能説明,詳細説明
```
追記例:
```
grove-water-level-sensor,水位センサー,静電容量式の水位センサーで水位(%)を取得する
```

### `_data/partslist.csv` の列（ヘッダー行の順序を厳守）
```
インターフェイス,カテゴリー,型番,商品URL,説明,画像URL,サンプルコードURL,回路図URL,データシートURL,製造元資料URL,アプリケーションノートURL,仕様書URL,説明書URL,ガイドURL,microbitサンプルコードURL,piZeroサンプルコードURL
```
- 空欄が許容される列（商品URL・回路図URL・各種資料URLなど）は無理に埋めない
- ダブルクォートで囲む必要がある値（カンマを含む場合など）はCSVとして正しくエスケープする
- `画像URL` 列は「部品写真のパス」が指定されコピーできていれば `partsImgs/<device>.jpg`、無ければ空欄にする
- `回路図URL` 列は、サイト上に`schematic.png`等が公開された後のURLを指すため、
  ローカルでの生成時点では基本的に空欄でよい（人間が別途更新する）

---

## 5. Git/GitHubワークフロー（人間が実施）

ドライバ追加と同様、個人Fork→社内ステージングFork→本家、という2段階のPRチェーンになる。

1. 個人Fork（`chirimen.org`）のローカルクローンでブランチを作成する（例: `docs/<device>-sample`）
2. 生成された`pizero/src/esm-examples/<device>/`・`index_examples.csv`・`_data/partslist.csv`・
   `partsImgs/<device>.jpg`をコミットし、個人Forkへプッシュする
3. 社内ステージングFork（`https://github.com/KDDI-Technology/chirimen.org`。「1. 対象リポジトリと前提」参照）に
   受け入れ用ブランチが**事前に**用意されていることを確認する（この確認自体は「3. 入力ファイルのチェック」で
   ClaudeCodeが`git ls-remote`により実施済みのはずである。完了報告で警告が出ていた場合は、ここで人間が
   GitHub上でブランチを作成する）
4. 個人Forkのブランチ → 社内ステージングForkの受け入れブランチへPull Requestを作成する
5. 社内ステージングFork側でマージされた後、社内ステージングFork → 本家（`chirimen-oh/chirimen.org`）へ
   Pull Requestを作成する
6. 本家でのレビューにより修正依頼があった場合は、**手順4（社内ステージングForkへのPR作成）からやり直す**
7. 本家レビューで問題なければマージされる

---

## 6. 完了報告フォーマット

- **コミット対象ファイル一覧**: 個人Forkのローカルクローンで `git status --porcelain`（読み取り専用）を実行し、
  新規作成・変更されたファイルを**相対パスで漏れなく**列挙する。そのまま使える形で
  `git add pizero/src/esm-examples/<device>/main.js pizero/src/esm-examples/<device>/package.json ...`
  のようなコマンド例も添える。この一覧を過不足なくコミットすれば、人間はそれ以上ファイルを
  作成せずコミット・プッシュに進める状態になっていることを保証する（4章の一覧はあくまで種類の
  説明であり、本項目が実際にその回で変更された正確なファイルパスの一覧になる）
  - **完全性の自己チェック**: `git status --porcelain` の結果を「4. 生成対象ファイル一覧」と突き合わせ、
    一覧にある種類のファイルが実際に含まれているか確認する。設計・配線図ファイルや部品写真が
    未登録のため意図的に含まれていない場合を除き、想定より不足しているものがあればこの完了報告で
    明示的に警告する。これはユーザー確認用の一覧であると同時に、ClaudeCode自身が生成漏れを
    検出するための工程でもある
- チェックで警告になった項目（配線図・設計ファイル未登録、商品URL未設定、
  社内ステージングFork側の受け入れブランチ未作成など）
- **現状ドライバ側ブランチだけでは用意できない情報**（毎回のリマインドとして明記する）:
  - 「カテゴリー」（partslist.csv用のカテゴリー分類）
  - 「商品URL」（商品購入ページ）
  - 「配線・仕様の補足説明」（配線・仕様の日本語補足文）
  - 設計・配線図ファイル一式（配線図画像`schematic.png`、`.svg` / `.fzpz` / `.fzz`や
    SVGの元になった編集用デザインファイルなど。拡張子は問わない）と「部品写真のパス」
    （いずれも画像・図面であり、ClaudeCodeでは生成できないため人間が用意する。これらが未登録の間は
    「コミット対象ファイル一覧」には含まれない旨も明記する）
- 推奨コミットメッセージ案（Conventional Commits形式。例: `docs: add grove-water-level-sensor example and parts info`）
- 推奨ブランチ名案（例: `docs/grove-water-level-sensor-sample`）
- 人間が次に行うべき手順:
  1. 生成内容のレビュー・実機での動作確認
  2. 上記「現状ドライバ側ブランチだけでは用意できない情報」を用意し、不足していれば追加する
  3. 上記「コミット対象ファイル一覧」の内容で「5. Git/GitHubワークフロー」に従って
     個人Fork→社内ステージングFork→本家の順にコミット・プッシュ・PR作成を行う

---

## 7. 参考ページ一覧

- [新しいドライバの追加（オプション手順部分）](https://www.chirimen.org/chirimen-drivers/docs/contributing/add-driver.html)
- [はじめに・基本ルール](https://www.chirimen.org/chirimen-drivers/docs/contributing/getting-started.html)
- 参考にした実例: [pizero/src/esm-examples/grove-water-level-sensor](https://github.com/chirimen-oh/chirimen.org/tree/master/pizero/src/esm-examples/grove-water-level-sensor)
