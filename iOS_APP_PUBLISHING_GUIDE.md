# iOS試験アプリ 公開プロセスガイド
> 賃貸不動産経営管理士アプリの公開経験をもとに作成。今後の横展開用。

---

## 全体フロー（7ステップ）

```
① テンプレートコピー＆データ差し替え
② アイコン作成・配置
③ EAS Build（ビルド）
④ App Store Connect で新規アプリ登録
⑤ Transporter で .ipa アップロード
⑥ App Store Connect で審査情報入力＆提出
⑦ 審査通過 → 販売開始
```

---

## ① テンプレートコピー＆データ差し替え

### やること
```bash
cd ~/dev/
cp -r chintai-kanrishi {新アプリ名}
cd {新アプリ名}
```

### 変更するファイル
| ファイル | 変更内容 |
|---------|---------|
| app.json | name, slug, bundleIdentifier, version |
| data/categories.json | 試験の分野名（日本語） |
| data/kakomon/questions.json | 問題データ |
| appConfig.json | タイトル・サブタイトル |
| assets/icon.png | アプリアイコン（1024×1024 PNG） |

### ⚠️ つまづきポイント
- **answerは0始まり**：PDFの正解「3」→ JSONでは `"answer": 2`
- **カテゴリ名はローマ字禁止**：`hourei` ではなく `関係法令（有害業務に係るもの以外）`
- **categories.json と questions.json のカテゴリ名は完全一致**させる
- **解説は第三者サイトからコピーしない**：CCにオリジナル生成させる
- **questionImage フィールド**：テキストonly試験なら不要。コードが残っていたら削除
- **問題数が少なすぎないか確認**：24問のサンプルのまま提出しない。最低200問以上
- **5択の試験に注意**：衛生管理者・危険物乙4・登録販売者は5択。テンプレートが4択なら修正必要

---

## ② アイコン作成・配置

### 仕様
- **サイズ**：1024×1024px
- **形式**：PNG（透過なし）
- **角丸**：不要（iOSが自動で角丸にする）
- **配置場所**：`assets/icon.png`

### ⚠️ つまづきポイント
- **プレースホルダーアイコンのまま提出すると却下される**（Guideline 2.3.8）
- app.json の `"icon"` が正しいパスを指しているか確認
- ファイルサイズが極端に小さい（2KB以下）なら中身が空の可能性あり
- Illustratorで作成 → 1024×1024 PNG書き出し → `assets/icon.png` に上書き

---

## ③ EAS Build（ビルド）

### コマンド
```bash
cd ~/dev/{アプリ名}
eas build --platform ios --profile production
```

### 初回のみ
- 証明書セットアップの対話が出る → 基本全部 Yes
- 「Do you want to log in to your Apple account?」→ `yes` とタイプしてEnter
- ⚠️ `yes` コマンドをパイプしない（無限ループする）

### ⚠️ つまづきポイント
- **EAS無料枠の上限**：月のiOSビルド回数に制限あり。上限到達→リセットは翌月1日
- **Starter プラン $19/月**で即ビルド可能。4本ビルドしたら翌月解約でOK
  - https://expo.dev/accounts/{ユーザー名}/settings/billing
- **ローカルビルド**（`--local`）はXcode必要＆トラブル多い。EAS課金の方が確実
- **Bundle ID末尾の数字**でエラーになることがある（「Could not find target '4'」）
  - app.json の name/slug を確認
- **`--non-interactive` は初回NG**：証明書セットアップが必要なため

---

## ④ App Store Connect で新規アプリ登録

### URL
https://appstoreconnect.apple.com/apps

### 手順
1. 「＋」→「新規アプリ」
2. プラットフォーム：iOS
3. 名前：「{試験名} 完全攻略」
4. プライマリ言語：日本語
5. バンドルID：`com.paul19820113.{英字}`
6. SKU：`{英字}`（Bundle IDの末尾と同じでOK）

### ⚠️ つまづきポイント
- **バンドルIDがドロップダウンに出ない場合**：まだEAS Buildが完了していない。ビルド完了後に表示される
- 登録後、**Apple ID（数字）をメモ**→ `eas.json` の `ascAppId` に設定

---

## ⑤ .ipa アップロード

### 方法1：eas submit（不安定）
```bash
eas submit --platform ios --latest
```
- `ascAppId` が正しく設定されていること
- **⚠️ 「Something went wrong」エラーが頻発**するが、実際にはアップロード成功していることがある→ TestFlightタブで確認

### 方法2：Transporter（確実・推奨）
1. Mac App Store から Transporter をインストール（無料）
   - https://apps.apple.com/jp/app/transporter/id1450874784
2. .ipa をブラウザからダウンロード：
   ```bash
   eas build:list --platform ios --limit 1
   ```
   → 表示される Application Archive URL をブラウザで開く → 自動ダウンロード
3. Transporter を開く → Apple IDでログイン → .ipa をドラッグ&ドロップ → 「配信」

### ⚠️ つまづきポイント
- **curl でのダウンロードは失敗しやすい**。ブラウザから直接ダウンロードが確実
- 「バージョンは既にアップロード済み」エラー → 実は成功している。TestFlightタブで確認
- **eas.json の ascAppId が古いアプリのIDのまま**だとエラーになる。新アプリのIDに変更

---

## ⑥ App Store Connect で審査情報入力＆提出

### 必要な情報

#### スクリーンショット（3枚以上）
- iPhone 6.5インチ用（1284×2778 または 1242×2688）
- iPhoneミラーリングでスクショ撮影
- ホーム画面・問題画面・結果画面の3枚が基本

#### 説明文
```
{試験名}試験対策の一問一答アプリです。
{問題数}問＋解説収録。分野別学習・模擬試験モード搭載。
オフライン対応でいつでもどこでも学習できます。
```

#### カテゴリ
教育

#### 価格
App Store Connect → 価格および配信状況 → 価格表で設定

#### App Review Information（Notes欄）← 重要！
```
1. Screen Recording: [添付済み] This is a simple quiz app with no login, no account system, no in-app purchases, no user-generated content, and no access to sensitive data.

2. Purpose: This app helps users prepare for the {試験名} national exam. It provides {問題数} practice questions with explanations across {分野数} categories.

3. Instructions: No login required. Launch the app → tap any category → answer multiple-choice questions → view explanations. All content is available immediately.

4. External Services: None. The app is fully offline with no external APIs, no analytics, no authentication services.

5. Regional Differences: The app functions consistently across all regions. Content is in Japanese as the exam is Japan-specific.

6. Regulatory: This app is an independent study tool and does not claim official affiliation with any examination body.
```

#### 画面録画
- iPhoneで実機を操作しながら画面収録（設定→コントロールセンター→画面収録）
- 起動→分野選択→問題回答→結果画面まで30秒程度
- App Review の返信画面に添付

### ビルドの選択
- 「配信」タブ → バージョン 1.0 → 「ビルド」セクション
- 古いビルドがある場合：クリックして「−」で削除 → 「＋」で新しいビルドを選択

### ⚠️ つまづきポイント
- **Guideline 2.1 - Information Needed**：初回提出は高確率で返される。上記Notesと画面録画を最初から入れておけば防げる
- **Guideline 2.3.8 - Accurate Metadata**：アイコンがプレースホルダーだと却下。本物のアイコンでビルドしてから提出
- **スクリーンショットは実機の画面**を使う。デザインモックではNG（Guideline 2.3.3）

---

## ⑦ 審査通過 → 販売開始

### 自動リリースの場合
- 「このバージョンを自動でリリースする」を選択していれば、審査通過後に自動公開

### 手動リリースの場合
- 「このバージョンを手動でリリースする」→ 保存 → 「このバージョンをリリース」ボタン

### ⚠️ つまづきポイント
- **有料アプリ契約が必須**：ビジネスタブで「有料アプリ契約」が「有効」になっていないと販売できない
  - 契約署名 → 銀行口座登録 → 納税フォーム → 反映まで最大24時間
  - 口座情報は事前に設定しておくとスムーズ
- **「保存」がグレーアウト**：変更がない状態。正常。ページリロードで解消することもある
- **App Storeの言語が「EN（英語）」表示**になることがある。App Store Connect の「アプリ情報」→ ローカリゼーション → 日本語が設定されているか確認

---

## 横展開チェックリスト

新しいアプリを出す前に確認：

```
□ app.json の name / slug / bundleIdentifier を変更した
□ categories.json を日本語で更新した
□ questions.json に200問以上入っている
□ answer が 0始まりインデックスになっている
□ 選択肢の数（4択/5択）に合わせてUIが対応している
□ アイコン（1024×1024 PNG）が assets/icon.png に配置済み
□ OGP不要（iOSアプリ）
□ App Store Connect で新規アプリ登録済み
□ ascAppId を eas.json に設定済み
□ App Review Notes を事前に記入済み
□ 画面録画を撮影済み
□ スクリーンショット3枚以上
□ 有料アプリ契約が「有効」
□ 価格を設定済み（¥600 or ¥400）
```

---

## 価格設定

| アプリ | 問題数 | 価格 |
|--------|:---:|:---:|
| 賃貸不動産経営管理士 | 250問 | ¥600 |
| 第一種衛生管理者 | 300問+ | ¥600 |
| 第二種衛生管理者 | 200問+ | ¥400 |
| 登録販売者 | 600問 | ¥600 |
| 危険物乙4 | 200問+ | ¥600 |
