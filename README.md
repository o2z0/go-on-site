# go-on-site

iOS アプリ **Go on** の紹介ページ / プライバシーポリシー / サポートページ。

- https://go-on.o2z0.com/
- https://go-on.o2z0.com/privacy
- https://go-on.o2z0.com/support

App Store Connect に登録する URL なので、**パスを変えない**こと。
アプリ本体のリポジトリは別 (m-k0yama/go-on, private)。

## 制約

JavaScript と外部リクエストを持たない。ページを追加・変更したら、配信される
HTML / CSS に対して次が 0 件であることを確認する:

    grep -rniE "analytics|gtag|googletagmanager|cookie|<script|<iframe|href=\"http|src=\"http" . --include=*.html --include=*.css

Web フォントの CDN 読み込みも禁止。フォントは端末内蔵のものだけを使う。
言語切替は JavaScript ではなく CSS の `:has()` で行う。
