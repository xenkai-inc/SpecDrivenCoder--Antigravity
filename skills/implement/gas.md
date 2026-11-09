# /implement — GAS アプリの場合の追加ルール

> SKILL.md の手順（仕様の確認 → プランの提示 → 実装 → テスト → 引き継ぎ）はそのまま使う。GAS では「テスト実行」のあとに **保存（push）と確認用 URL の案内** が加わる。確認用 URL で受け入れ基準を確かめるのは、`/review` のあと（`skills/review/gas.md`）。

## 実装の書き方

- **判定・計算・整形は、小さな関数に分けて `.gs` に書く**。`doGet` や、画面から呼ばれる関数（`google.script.run` の先）の中に直接書かない。分けておくと `/test` で自動テストできる
- **秘密情報は `PropertiesService.getScriptProperties().getProperty('GEMINI_API_KEY')` のように読む**。キーそのものをコードに書かない。スクリプトプロパティへの値の入力は、利用者が Apps Script の「プロジェクトの設定」で自分の手で行う（エージェントは手順を案内する）
- **画面へ返す値は JSON にできる形だけ**（文字列・数値・真偽値・配列・オブジェクト）。日付は `toISOString()` などで文字列にして返す（`Date` のままだと画面側で受け取れない）
- **画面へ返す値に、秘密情報やシート全体のデータを含めない**。画面に必要な項目だけを返す（画面側に届いた値は、ブラウザから誰でも見られるため）
- 画面側から呼ばれる関数でも、**入力チェックは `.gs` 側でもう一度行う**（画面のチェックだけでは素通りされることがあるため）

## テストのあと：保存（push）と確認用 URL の案内（SKILL.md「4. テスト実行」に続けて行う）

### 1. 保存（push）の前の確認

- 差分に API キーらしき文字列が入っていないかを見る（「安全の約束」2）
- `.clasp.json` があるか確認する。**ない場合**（まだ Google 側にファイルが無い）は、利用者に「Google ドライブに Apps Script のファイルを作ってよいか」を確認してから作る：
  ```bash
  clasp create --type webapp --title "<アプリ名>"
  ```
  > 注意：このコマンドは `appsscript.json` を上書きする（clasp 3.4.1 で確認。`timeZone` が `America/New_York` になり、`webapp` の設定が消える）。実行前に控えを取り、実行後に戻す：
  > ```bash
  > cp appsscript.json appsscript.json.bak
  > clasp create --type webapp --title "<アプリ名>"
  > mv appsscript.json.bak appsscript.json
  > ```

### 2. 保存（push）

```bash
clasp push --force
```

- **`--force` は必ず付ける**。`appsscript.json` が Google 側と違うとき、付けないと「Skipping push.」と出て **何も送られない**（対話なしで実行すると、上書きしてよいかの確認が「いいえ」扱いになるため。clasp 3.4.1 で確認）。PC 側が正本なので、常に付けてよい
- 結果に「Pushed N files」と出て、`appsscript.json`・`.gs`・`.html` が並んでいることを確かめる。並んでいなければ送れていない

利用者には一言添える：

> 「PC のファイルを Google 側に送りました。Google 側（Apps Script のエディタ）で直接書き換えた内容があった場合は、今の送信で上書きされています。直すときは、いつもこちら（PC 側）で直しましょう」

### 3. 確認用 URL を案内する

確認用 URL は、保存した最新の内容がすぐ見られる、**作った本人だけが開ける URL**（末尾が `/dev`）。

```bash
clasp deployments
```

一覧の `@HEAD` の行にある ID を使い、`https://script.google.com/macros/s/<ID>/dev` を案内する。

- 初めて開くときは「権限の確認」画面が出る。何を許可しているか（スプレッドシート・外部への接続など）を先に説明し、**承認は利用者本人が行う**（エージェントは代行できない）
- 「公開」（`clasp deploy`、末尾が `/exec` の URL を人に渡すこと）は `/implement` では行わない。公開の話が出たら `/secure` を通す
- **受け入れ基準の確認は、ここではまだしない**。確認用 URL の AC は `/review` で仕様とのズレを直したあとに、まとめて確かめる（先に確かめても、レビューで直すとやり直しになるため）。利用者には一言予告する：

> 「確認用 URL はこちらです。画面で受け入れ基準を確かめるのは、次のレビューでズレを直したあとに一緒にやりましょう」

## 報告の仕方（SKILL.md の読み替え）

「動作確認の手順」は `npm run dev` ではなく、確認用 URL を伝え、確かめるのは `/review` のあとだと書く：

```
## 確認用 URL
https://script.google.com/macros/s/<ID>/dev
（画面で確かめる受け入れ基準は、次のレビューのあとに一緒に確かめます）
```

## やらないこと（追加）

- 利用者の確認なしに `clasp create`（Google ドライブにファイルを作る）・`clasp deploy`（公開）をしない
- スクリプトプロパティの値を、エージェントが読み出して表示しない
- Google 側のエディタで直接コードを書き換えない（PC 側が正本）
