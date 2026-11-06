# 余白 / yohaku

写真の周りへ白い余白を付け、Instagram で切られずに出すための小さな Web アプリ。

https://higa4yama3.github.io/yohaku/

A tiny web app that pads a photo with a white margin so Instagram shows the whole frame. Runs entirely in the browser; nothing is uploaded.

## 何ができるか

- 写真を選ぶと、白い背景の中央に置いた絵が出る
- 型は正方形 1:1、フィード 4:5、ストーリー 9:16
- 余白は背景の長辺に対する比率。既定は 6.5%
- 背景の色を変えられる
- 保存すると JPEG（長辺 1080 px、または 2160 px）が共有シートに出る。iPhone なら「画像を保存」で写真アプリに入る

写真はブラウザの中で処理し、サーバーには送らない。Canvas で描き直すので、撮影場所などの EXIF は書き出しに残らない。

## iPhone で使う

1. Safari で上の URL を開く
2. 共有ボタンを押し、「ホーム画面に追加」
3. 以後はホーム画面の「余白」から開く。一度開けばオフラインでも動く

## 余白の決め方

手本にした Instagram の白いカードを測った値から決めた。

| 測ったもの | 値 |
|---|---|
| 背景 | 正方形 |
| 写真 | 3:2 |
| 左右の余白 | 背景の 6.5% |
| 上下 | 中央 |

一般化すると、余白 m を背景の長辺 × 6.5% とし、写真は幅 W−2m、高さ H−2m に収まる最大の大きさで中央に置く。横写真なら左右が m、縦写真なら上下が m になる。1080×1080 に 3:2 の写真を置くと、写真は 940×627、余白は左右 70 px。

```js
function layout(W, H, pw, ph, marginPct) {
  const m = Math.round(Math.max(W, H) * marginPct / 100);
  const s = Math.min((W - 2 * m) / pw, (H - 2 * m) / ph);
  const w = Math.round(pw * s), h = Math.round(ph * s);
  return { x: Math.round((W - w) / 2), y: Math.round((H - h) / 2), w, h, m };
}
```

## ファイル

| ファイル | 役割 |
|---|---|
| `index.html` | 本体。画面、`layout()`、書き出し。iPhone 用のアイコンは base64 の PNG で埋めてある |
| `manifest.webmanifest` | ホーム画面に置いたときにブラウザの枠なしで開く設定。アイコンは data URL の SVG |
| `sw.js` | 一度開いた後はオフラインでも開けるように 3 ファイルを端末に置く Service Worker |

依存はない。ビルドもない。3 ファイルを静的に配信すれば動く。

## 自分で置く

3 ファイルを任意の https のサーバーに置く。パスは相対なので、`https://example.com/yohaku/` のような下層でも動く。Service Worker は https か localhost でだけ登録される。

手元で試すときは、次のコマンドで配信して開く。

```bash
python3 -m http.server 8000
```

`sw.js` を含めて何かを直したら、`sw.js` の `VERSION` を上げる。古い版を置いた端末は、次に開いたときに新版を取り、もう一度開き直すと切り替わる。
