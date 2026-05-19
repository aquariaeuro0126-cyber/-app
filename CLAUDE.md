# 買い物学習アプリ

## プロジェクト概要
買い物を通じて学習するWebアプリケーション。GitHub Pagesで公開中。

## 現在の状態
- GitHub Pages公開中：リポジトリルートのindex.html
- GitHubリポジトリ：git@github.com:aquariaeuro0126-cyber/-app.git
- git管理：2026年5月より開始

## ファイル構成
```
買い物学習アプリ/
├── index.html            ← メインアプリ（公開中・直接編集する）
├── register-app.html     ← 登録用アプリ
├── html5-qrcode.min.js   ← QRコードライブラリ
└── 品物カード/           ← サンプル画像（GitHubには上げない）
```

## 今後の方針
- index.html・register-app.htmlを直接編集してコミットする
- バージョンフォルダは作らない（gitが履歴を管理するため）
- 品物カード/フォルダはローカルのみ保管（GitHubには上げない）
- GitHub PagesはリポジトリルートのURLで公開中

## 作業時の注意
- pushの前に変更内容をユーザーに報告・承認を得ること
- 品物カード/は.gitignoreで除外済み（誤ってpushしない）

## 作業記録

### 2026-05-19

#### 背景と出発点
買い物学習アプリ（register-app.html）はQRコード読み取り機能を持つシングルHTMLファイルとして完成していた。
前セッションまでにQRコード生成・スキャンが動作する状態になっていたが、今セッションではUIの学習設計改善・iPadでの動作不具合・公開環境の整備という3つの課題に取り組んだ。

#### 合計金額の非表示化（学習設計の改善）

**背景**：ユーザーから「合計金額は会計で入力後に答え合わせで表示してほしい」という要望があった。現状はスキャン画面・会計画面ともに合計金額がリアルタイムで見えており、学習の意味が薄れていた。

**設計の考え方**：
- スキャン画面：合計を完全に隠し「かいけいでこたえあわせ！」というグレーテキストに置き換える
- 会計画面：商品一覧（名前・値段）は表示して「何を買ったか」は確認できるようにする。合計は「いくらになるかな？🤔」と表示して隠す
- 結果オーバーレイ：正解・不正解どちらでも正しい合計金額を表示する
- 不正解時：「もういちどやる」で会計画面に戻り同じカートで再挑戦できるようにする

**実装手順**：
1. CSS：`#totalPrice { display: none; }` と `.totalHidden`（グレーテキスト）クラスを追加
2. スキャン画面HTML：`totalPrice` の隣に「かいけいでこたえあわせ！」テキストを追加
3. CSS：会計画面に `.hiddenPrice`（「いくらになるかな？」表示用）クラスを追加
4. 会計画面HTML：商品一覧表示エリア（`checkoutCartList`）を追加、合計金額を `display:none` 初期値に
5. `showScreen()` の `studentCheckoutScreen` 分岐：カートの商品一覧をHTMLで生成して挿入、合計は非表示のまま
6. `showResult()` の `resultSub` テキスト：正解・不正解ともに合計金額を文字で表示するよう変更
7. 結果ボタンのonclick：正解時は `resetAll()`、不正解時は会計画面に戻る分岐を追加
8. CSS：`resultSub` に `white-space: pre-line` を追加（`\n` による改行を有効化）

#### iPadでQRスキャナーが起動しない問題の修正

**症状**：PCでは動作するが、iPadで開いたところQRスキャナーが起動しなかった。

**原因究明**：
`html5-qrcode.min.js` は375KBの外部ファイルとして同フォルダに配置し `<script src="html5-qrcode.min.js">` で読み込む方式を取っていた。iPadのSafariでは `file://` プロトコルからの相対パス読み込みがセキュリティ制限によりブロックされるため、`Html5Qrcode` が未定義になりスキャナーが起動しなかった。加えて `html5-qrcode` 自体がiOS SafariのWebkit実装と相性が悪い問題もある。

**解決方針の検討**：
- html5-qrcodeをインライン化 → 375KBのため断念（前セッションで423KBになりChromeがクラッシュした経験あり）
- `jsQR`（130KB）+ ブラウザ標準 `getUserMedia` API の組み合わせに切り替える → 採用
  - jsQRはHTMLにインライン埋め込み可能なサイズ（62KB + 130KB = 192KB、問題なし）
  - `getUserMedia` はiOS Safari 11以降で標準サポート
  - 外部ファイルが不要になりHTMLファイル1つで完結する副次的メリットもある

**実装手順**：
1. `curl` でjsQR 1.4.0のminified版（130KB）を取得
2. HTMLの `<script src="html5-qrcode.min.js">` をjsQRのインライン `<script>` に置き換え
3. CSSに `#scanVideo`・`#scanCanvas`・`.scanFrame`（スキャン枠のオーバーレイ）のスタイルを追加
4. HTMLの `#readerContainer` 内に `<video id="scanVideo">`・`<canvas id="scanCanvas">`・`<div class="scanFrame">` を追加
5. JS：`startScanner()` を全面書き換え（`getUserMedia` + 背面カメラ優先・フォールバックあり）
6. JS：`scanLoop()` を新規実装（`requestAnimationFrame` + jsQRでフレーム解析）
7. JS：`stopScanner()` を全面書き換え（カメラ解放・ループ停止）
8. `html5-qrcode.min.js` は不要になり、HTMLファイル1つで完結する構成に変更

**結果**：
- ファイル構成がHTMLのみ（191KB）に簡素化
- iPadのSafariでもカメラが起動するようになった

#### iPadでの開き方の問題とGitHub Pages公開の決定

**背景**：
iPadへのAirDropは成功していたが、ファイルアプリから開くと「クイックルック」で開かれカメラが使えなかった。iOS版Safariは仕様上 `file://` のローカルHTMLファイルを開く機能がなく、アドレスバーに `file:///` を入力しても拒否された。

**解決策**：GitHub Pages（無料・HTTPS）を採用。`https://` URLになるためiPad SafariでカメラAPIが使用可能になる。

**GitHub Pages公開手順**（ユーザーが実施）：
1. GitHubアカウントにログイン（既存アカウントあり）
2. 新規リポジトリを Public で作成
3. `register-app.html` を `index.html` としてアップロード
4. Settings → Pages → Branch: main → 保存
5. 数分後にURLが発行される

#### 決定事項・次のタスク
- アプリの公開方法：GitHub Pages（HTTPS）に決定
- HTMLファイル1本で完結する構成（外部依存ファイルなし）が確立した
- `html5-qrcode.min.js` は不要になったが削除はユーザーの判断に委ねる
- 次のタスク：GitHub PagesのURLが発行された後、iPadのSafariで動作確認を行う
