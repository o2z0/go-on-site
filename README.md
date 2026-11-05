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
    js/ga.js           GA4 の共通部分（o2z0 の正本のコピー。書き換えない）
    privacy/ support/  ドキュメント 2 ページ（<body class="doc">）
    assets/            画像（マーク・実画面・カード）+ OGP + apple-touch-icon
    fonts/             self-host した Newsreader のサブセット
    favicon.ico

## 制約

**第三者へのリクエストは GA4（`www.googletagmanager.com` と `google-analytics.com`）だけ許可。**
CDN・GA4 以外の解析・埋め込み・Web フォントの外部読み込みは禁止。2026-11-05 に「解析も禁止」から緩めた
（アプリの「データを送らない」はアプリの話で、LP は計測する。方針の正本は
`~/Project-local/2026/lab/docs/superpowers/specs/2026-11-04-o2z0-family-ga4-design.md` の §5・§7b）。
`google-analytics.com` への送信は gtag.js が実行時に出すもので、HTML には現れない。

ページを足す / 変えたら、配信される HTML / CSS / JS に対して次の 2 本を走らせる。
**zsh では `--include` のグロブをクォートしないと展開されて grep が黙って落ちる**ので、
下のとおりシングルクォート込みで打つこと:

    grep -rhoE 'https?://[^"'"'"' )]+' . --include='*.html' --include='*.css' --include='*.js' | sort | uniq -c

    grep -rniE 'analytics|gtag|googletagmanager|cookie|<iframe' . --include='*.html' --include='*.css'

1 本目でヒットしてよいのは次だけ:

- `https://www.googletagmanager.com/gtag/js?id=G-71RZL2MG8W` — **3 件（3 ページの `<script src>`）。
  ページが自分で出す第三者リクエストはこれだけ**
- `https://go-on.o2z0.com/...` — canonical / og:url / og:image の 3 件（OGP は相対パスを許さない）
- `https://apps.apple.com/app/id6797468303` — CTA の飛び先 2 件
- privacy の GA4 の節のリンク 4 件 — `business.safety.google`（JA）/ `policies.google.com`（EN）/
  `tools.google.com/dlpage/gaoptout`（JA は `?hl=ja` 付き、EN は無し）

GA4 の `src` 以外はすべて `href`（利用者がクリックして離脱する先）で、ページが自分で出すリクエストではない。
`src` / `link` / `@font-face` に googletagmanager 以外のホストが出てきたらそれは違反。

2 本目でヒットしてよいのは次の 6 行だけ（`<iframe` は 0 件）:

- 3 ページの `<script async src="https://www.googletagmanager.com/gtag/js?...">` の行 × 3
- `privacy/index.html` の GA4 の節 × 3（JA 本文 1 行・EN 見出し 1 行・EN 本文 1 行。Cookie の語は本文と同じ行）

**GA4 の設定は `js/ga.js`（o2z0 の `public/js/ga.js` を無改変でコピー。冒頭コメントに正本の日付）。**
ここでは書き換えず、正本を直してコピーし直す。各ページには次の 2 本だけを置く:

    <script async src="https://www.googletagmanager.com/gtag/js?id=G-71RZL2MG8W"></script>
    <script defer src="/js/ga.js" data-ga-id="G-71RZL2MG8W" data-ga-host="go-on.o2z0.com"></script>

`data-ga-host` があるので `go-on.o2z0.com` 以外（localhost・手元プレビュー）では何も送らない。
自分の端末は一度 `https://go-on.o2z0.com/?internal=on` を開くと送らなくなる（印は localStorage、オリジン単位）。
App Store へのクリックは GA4 の拡張計測（離脱クリック = `click`）に任せていて、独自イベント（`ga-events.js`）は無い。
ページを足すときは 2 本のタグも足し、privacy の GA4 の節と食い違わないようにする。

**ページ自身の JavaScript は index.html 末尾の FINALE 用インラインスクリプト（約 20 行）と `js/ga.js` だけ。**
FINALE のスクリプトは「最下部でブランドが一度消える」という演出そのもので、代替手段がないので例外扱いにしている。
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
