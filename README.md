# go-on-site

iOS アプリ **Go on** の紹介ページ / プライバシーポリシー / サポートページ。

- https://go-on.o2z0.com/
- https://go-on.o2z0.com/privacy/
- https://go-on.o2z0.com/support/

App Store Connect に登録する URL なので、**パスを変えない**こと（末尾スラッシュ付きが正準形）。
アプリ本体のリポジトリは別 (m-k0yama/go-on, private)。

## 構成

    index.html         LP（演出 CSS はこの中にインライン）
    style.css          3 ページ共通の地（トークン・書体・言語トグル・CTA・footer）
    privacy/ support/  ドキュメント 2 ページ（<body class="doc">）
    assets/            画像（マーク・実画面・カード）+ OGP + apple-touch-icon
    fonts/             self-host した Newsreader のサブセット
    favicon.ico

## 制約

**第三者へのリクエストを 1 つも持たない。** CDN・解析・埋め込み・Web フォントの外部読み込みは全部禁止。
ページを足す / 変えたら、配信される HTML / CSS に対して次の 2 本を走らせる。
**zsh では `--include` のグロブをクォートしないと展開されて grep が黙って落ちる**ので、
下のとおりシングルクォート込みで打つこと:

    grep -rhoE 'https?://[^"'"'"' )]+' . --include='*.html' --include='*.css' | sort | uniq -c

    grep -rniE 'analytics|gtag|googletagmanager|cookie|<iframe' . --include='*.html' --include='*.css'

1 本目でヒットしてよいのは次の 2 つだけ:

- `https://go-on.o2z0.com/...` — canonical / og:url / og:image の 3 件（OGP は相対パスを許さない）
- `https://apps.apple.com/app/id6797468303` — CTA の飛び先 2 件。**これは `href`（利用者がクリックして
  離脱する先）であって、ページが自分で出すリクエストではない**ので「第三者へのリクエスト 0 件」は保たれる。
  `src` / `link` / `@font-face` に apple.com が出てきたらそれは違反。

2 本目は 0 件。

**JavaScript は index.html 末尾の FINALE 用インラインスクリプト（約 20 行）だけ。**
これは「最下部でブランドが一度消える」という演出そのもので、代替手段がないので例外扱いにしている。
言語切替は JS ではなく CSS の `:has()` で行う（3 ページとも **EN 既定** + 右上トグルで JA）。

### フォント

見出しのセリフは Newsreader（SIL Open Font License 1.1）を `fonts/` に同梱している。
Google Fonts の latin スライスを `fontTools` で絞ったもの:

- `newsreader-latin.woff2` — wght 400..500 可変 / **opsz 36 に固定**
- `newsreader-latin-italic.woff2` — wght 400 固定 / opsz 36 固定
- 収録は印刷可能 ASCII + 約物（`’ “ ” — – · …`）。**和文は入っていない**（Hiragino Mincho へフォールバック）

278KB → 33KB。opsz を固定したのは LP のセリフ実寸が 26–52px に収まっているため。
**EN コピーに ASCII 外の文字（アクセント付きラテン等）を足すとその字だけ別書体に化ける。**
足すときは `style.css` の `unicode-range` と一緒にサブセットを作り直すこと。

### スクロール演出

LP の演出は CSS scroll-driven animations で、
`@media (prefers-reduced-motion: no-preference)` と
`@supports ((animation-timeline: view()) and (animation-range: entry))` の二重ガードの中にある。

**ピン留め（`position: sticky`）とシーンの尺（`420vh` 等）も同じガードの中に置く。**
ここを外に出すと、非対応ブラウザと `prefers-reduced-motion: reduce` のユーザーが
「アニメは動かないのに尺だけ残る」= 約 1,005vh の空スクロールを通らされる。
ガード外（＝静止版）では泡が普通に並び、シーンは 1 画面ずつになる。
