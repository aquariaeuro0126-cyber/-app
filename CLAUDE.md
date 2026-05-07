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
