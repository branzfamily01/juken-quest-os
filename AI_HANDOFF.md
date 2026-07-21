# AI_HANDOFF.md — 受験クエストOS v2 引き継ぎ書

このファイルは、**次にこのアプリを修正するAI(ChatGPT / Claude / Gemini など)に渡すための引き継ぎ書**です。
修正を頼むときは、必ず「本体HTML」と「このファイル」をセットで渡してください。

---

## 1. プロジェクトの正体

- **受験クエストOS v2**:中学受験用の教材を「教材JSONを貼るだけ」でゲーム化する家庭用学習アプリ
- 利用者:小5の姉と小3の弟(+親)。将来は外部販売も検討(play / custom / platform の3エディション)
- **本体は2ファイル**:`index.html`(自動版・GitHub連携あり)と `index-manual-only.html`(手動版・GitHub連携なし)。
  中身はGitHub連携部分(コード内【22】セクション)以外まったく同一。**片方だけ直して差分がずれないよう、
  機能追加・バグ修正は必ず両ファイルに同じ変更を入れる**(あるいは手動版を先に直し、自動版へコピーしてから【22】を足す)
- 教材JSONは①データ画面への手貼り付け(両版共通)、②GitHub(publicリポジトリ)からの自動取得(`index.html`のみ)、の2経路
- 正となる仕様書:`docs/SPEC.md`(仕様の食い違いはこちらが優先)

## 2. 絶対に守る技術的前提(壊すと動かなくなる/データが消える)

| # | 不変条件 | 理由 |
|---|---|---|
| 1 | **各HTMLはそれぞれ単体のHTML/CSS/JS 1ファイルで完結**(外部ライブラリ・ビルド禁止)。`index.html`と`index-manual-only.html`の2本立てだが、どちらも単独でダブルクリック起動できる状態を保つ | ダブルクリックで動く・オフラインで動くことが製品価値 |
| 2 | localStorage のキー名は **`"jukenOS1"`** から変えない | 変えると子どもの成績が全部消える |
| 3 | schema名 **`juken-pack@1` / `juken-theme@1` / `juken-profile@1` / `juken-config@1`** を変えない | 過去に作った教材JSONが読めなくなる |
| 4 | データ項目を増やすときは **`ensureProfile()` / `load()` に初期値の補修を追加**する | 古いデータで開いた瞬間にエラーになるのを防ぐ(マイグレーション) |
| 5 | XP式(レベルnに必要な累計XP = n×(n+1)×50)・復習間隔 INTERVALS を勝手に変えない | ゲームバランスは相談の上で変更する項目 |
| 6 | **サンプル教材・市販教材の本文を本体HTMLに埋め込まない** | 教材はJSON貼り付けで追加する方針+著作権(市販教材の複製はNG) |
| 7 | 子ども向け画面の文言は **ひらがな多め・小3が読める表現** | 利用者は小学生 |
| 8 | 修正後は必ず**構文チェック**(後述 §6)をしてから納品 | 1文字のミスで全画面が白くなるため |

## 3. コード地図(本体HTML内の【セクション番号】と対応)

コード内に `【セクションN】` の日本語コメントが入っています。目次は `<script>` 冒頭にもあります。

| セクション | 中身 | こんな依頼のときに触る |
|---|---|---|
| 【1】DEFAULT_CONFIG / CFG() | エディション・XP倍率・ガチャ確率・親レンジ初期値 | 販売3版の作り分け、バランス調整 |
| 【2】BUILTIN_THEMES | **テーマ10種(キャラ・配色・ガチャ景品・セリフ)** | ★キャラクター追加・世界観追加 |
| 【3】SAMPLE_PACKS(空) | サンプル教材は空。旧サンプル自動削除リストあり | 触らない(教材はJSONで追加) |
| 【4】BADGES ほか定数 | バッジ・復習間隔・読み上げ速度・**FMT_NAMES(出題形式一覧)** | ★出題形式追加の入口 |
| 【5】load / save / ensureProfile | localStorage 読み書き・データ補修 | ★成績データの構造変更(要マイグレーション) |
| 【6】speak / cycleRate | 読み上げ(TTS) | 音声まわり |
| 【7】levelOf / gainXP / grantBadge | レベル・XP・バッジ付与 | レベル制の調整 |
| 【8】applyTheme / updateHud / show | 色の適用・上部バー・画面切替 | 画面遷移の追加 |
| 【9】ウィザード | 新プロフィール作成(名前→テーマ→機体名) | 初期設定フロー変更 |
| 【10】renderProfiles | 起動時のプロフィール選択 | |
| 【11】renderHome | ホーム・今日のクエスト・バックアップ警告 | |
| 【12】SUBJECTS / renderPacks | **教科タブ**・パック一覧 | ★教科の追加(SUBJECTS配列に1語足すだけ) |
| 【13】openSesSetup / launchTrain | 子セレクト(問題数・難度・形式の絞り込み) | 出題ロジック調整 |
| 【14】startInput / flowStep | 暗記カード・きき流し | |
| 【14b】startListen / lisStepQ〜 / startSubjectListen | **きき流し特訓**(問題→答え→解説の自動読み上げ・名前入り応援) | ★新形式追加時はlistenAnswerText/Speechにも分岐追加 |
| 【15】startSession / renderQ / ans系 / judge | **クイズエンジン(6形式)** | ★出題形式追加の本体・採点/成績記録 |
| 【16】dueReviews / startReview | 復習(ライトナー式) | |
| 【17】startWrite | 手書き課題 | |
| 【18】renderMonsters / weakTags | モンスター図鑑・弱点判定 | |
| 【19】renderGacha / rollGacha | ガチャ | |
| 【20】renderParent / saveRange | 親ページ・出題レンジ設定・PIN・リセット | |
| 【21】doImport / importSchemaObject / downloadBackup | **教材JSON登録(共通処理はimportSchemaObject)・バックアップ** | ★JSON受け入れ形式の変更 |
| 【22】githubBase / fetchFromGithub | **GitHubから教材JSONを自動取得**(publicリポジトリのみ) | ★取得元の仕様変更・対応スキーマ追加 |
| 【23】起動処理 | ページを開いた直後の分岐(githubSync.autoならここで自動取得も呼ぶ) | |

## 4. よくある改修の具体手順

### A. キャラクター(テーマ)を1つ追加する
1. 【2】の配列末尾に、既存テーマ(例:theme_ocean)を丸ごとコピーして貼る
2. `id` を重複しない新名に(例:`theme_soccer`)。name / ship / colors / chars / monster / gacha / words を書き換える
3. colors は8色すべて指定。背景(bg)は**薄い色**にする(カード内の文字が読めなくなるため濃色背景は不可)
4. 以上で完了(ウィザードのタイル一覧は BUILTIN_THEMES を自動で全部表示する)
- 補足:JSON貼り付け(juken-theme@1)で追加したテーマは、**現状ウィザードの一覧には出ない**(既存プロフィールの themeId を書き換えれば使える)。ウィザードにも出したい場合は renderWizard 内の `BUILTIN_THEMES` を `S.themes` に変える改修が必要(既知の制限 §7-1)

### B. 出題形式を1つ追加する(例:fillblank 穴埋め)
1. 【4】`FMT_NAMES` に `fillblank:"あなうめ"` を追加(親ページの許可チェックにも自動で出る)
2. 【15】`renderQ()` の `qt` 分岐に画面の作り方を追加(choice の分岐を手本に)
3. 回答関数 `ansFillblank()` を作り、正誤を **`judge(true/false)`** に渡す(成績・XP・モンスターは judge が全部やるので触らない)
4. 読み上げに対応するなら `readQ()` に分岐を追加
5. 【14b】きき流し特訓の `listenAnswerText()` と `listenAnswerSpeech()` にも答えの組み立て分岐を追加(忘れると新形式がきき流しで無音になる)
6. 統合仕様書の「出題形式スキーマ」章に、JSONの書き方を追記する
7. 教材変換プロンプト(PROMPTS_FOR_NEXT_AI.md ①)にも新形式の説明を追記する

### C. 教科を追加する(例:英語)
1. 【12】`SUBJECTS` 配列に `"英語"` を追加 — これだけ
2. 教材JSON側で `"subject":"英語"` と書けばタブに表示される(文字は完全一致)

### D. 成績データに項目を足す
1. 保存したい場所(profile直下 or itemStats)を決める
2. **必ず `ensureProfile()` に初期値補修を追加**(§2-4)
3. 書き込みは基本 `judge()` か終了処理(endSession)に置き、`save()` を忘れない

### E. 販売用の3エディションを作る
1. 【1】`DEFAULT_CONFIG` の `edition` と `flags` を書き換えた HTML を3つ複製して作る
   - play: canAddMaterial=false, canEditTheme=false, canBrand=false
   - custom: 追加true・ブランドfalse(現在の初期値)
   - platform: すべてtrue
2. play版のみ、同梱するオリジナル自作教材を SAMPLE_PACKS に焼き込む(**市販教材は不可**)
3. platform版は SAMPLE_PACKS を空のまま出荷

## 5. データ構造の最重要ポイント(成績のありか)

- 全データ = 変数 `S` = localStorage `"jukenOS1"` の JSON
- 成績の実体:`S.profiles[子のid].itemStats["教材id:問題id"] = {c:正解数, w:不正解数, box:復習段階0〜5, next:"次に出す日", last:"最終回答日"}`
- 履歴:`profiles[].history`(直近200件)/ 手書きログ:`writeLog` / モンスター:`monsters`(タグ名がキー)
- バックアップ = `S` を丸ごとJSON化したもの。復元 = そのJSONを S に戻して reload

## 6. 修正後の検証手順(納品前に必ず)

1. `<script>〜</script>` の中身を取り出して `node --check` で構文チェック(または新しいブラウザタブで開いてF12コンソールに赤エラーがないこと)
2. 動作確認の最短コース:
   ① ウィザードで新プロフィール作成 → ② データ画面で教材JSONを1つ貼って登録 → ③ TRAINを1周(正解と不正解を両方出す)→ ④ ふくしゅうボタン → ⑤ バックアップDL → ⑥ リロードして続きから開けること
3. 既存データを壊さない確認:修正前のバックアップJSONを「復元」して開けること

## 6.5 GitHub自動取得の運用(【22】githubBase / fetchFromGithub)

- リポジトリ内に、教材を置くフォルダ(既定 `packs/`)を作る
- そのフォルダに **`index.json`**(ファイル名の配列。例:`["wariai_5nen.json","goi_5nen.json"]`)と、各教材の `juken-pack@1` JSONファイルを置く
- アプリの「データ」画面でGitHubユーザー名・リポジトリ名・ブランチ(既定main)・フォルダ名(既定packs)を入力→保存
- 「起動するたびに自動で取得する」にチェックを入れると、次回起動時から自動で取り込む(裏で実行、既存の起動を妨げない)
- **publicリポジトリのみ対応**(privateはCORSの都合で読めない)。教材の中身は勉強内容のみで個人情報を含まない前提のため、公開しても実害は基本ない
- 取得先URLの実体:`https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{path}/index.json`
- 教材を更新したいときは、GitHub側のJSONを書き換えてpushするだけでよい(次回自動取得時、同じidの教材が上書きされる=改訂)

## 7. 既知の制限・注意(次のAIがハマりやすい点)

1. **JSON貼り付けで追加したテーマはウィザード一覧に出ない**(renderWizard が BUILTIN_THEMES 固定のため)。§4-A参照
2. TEST/BOSS は設計として「きょうのセッティング」の対象外(本番再現のため作問どおり出す)。バグではない
3. TTSはブラウザ内蔵音声。iPhoneのマナーモードでは鳴らないことがある/2.5倍以上は機種により不自然。仕様として説明済み
4. order形式は items の並び順そのものが正解。同じ文字列を2回入れると判定が正しく動かないので教材側で避ける
5. localStorage はブラウザ都合で消えることがある前提の設計。だからバックアップ3日リマインドがある(この警告を消す改修はしない)
6. 旧サンプル4教材(pack_m5_wariai ほか)は起動時に自動削除される(【3】SAMPLE_PACK_IDS)。同じidで自作教材を登録しないこと

## 8. 修正を依頼するときのお作法(人間側メモ)

- 渡すもの:①本体HTML ②このAI_HANDOFF.md ③(仕様に関わる変更なら)統合仕様書
- 頼み方の定型文は `PROMPTS_FOR_NEXT_AI.md` にコピペ用があります
- 大きな変更は ROADMAP.md の段階に沿って、1回の依頼で1テーマに絞ると事故が減ります
