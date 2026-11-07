# yohaku (余白)

A tiny web app that pads a photo with a white margin so Instagram shows the whole frame instead of cropping it.

https://higa4yama3.github.io/yohaku/

Everything runs in the browser. Nothing is uploaded.

## What it does

- Pick a photo and it is placed in the centre of a white canvas
- Canvas shapes: square 1:1, feed 4:5, story 9:16
- Margin is a percentage of the canvas width. Default 6.5%. Changing the shape keeps the photo the same size and only adds white above and below
- Background colour is adjustable
- Save exports a JPEG (1080 px wide, or 2160 px) through the share sheet. On iPhone, "Save Image" puts it in Photos

The photo is redrawn on a canvas, so EXIF data such as location is not carried into the export.

## On iPhone

1. Open the URL above in Safari
2. Share button, then "Add to Home Screen"
3. Launch it from the home screen. After the first visit it works offline

## Why 6.5%

The number was measured from an Instagram post I liked: a 3:2 photo on a square white background.

| Measured | Value |
|---|---|
| Background | square |
| Photo | 3:2 |
| Left and right margin | 6.5% of the background |
| Top and bottom | centred |

With that geometry the photo covers 50.5% of the square, so photo and white weigh the same, while the thin sides make it read as a page rather than a mat.

Generalised: margin `m` is 6.5% of the canvas width, and the photo is scaled to the largest size that fits inside `W − 2m` by `H − 2m`, then centred. On a 1080 × 1080 canvas a 3:2 photo becomes 940 × 627 with 70 px on each side. The same photo on 4:5 or 9:16 stays 940 × 627; only the white above and below grows.

```js
function layout(W, H, pw, ph, marginPct) {
  const m = Math.round(Math.min(W, H) * marginPct / 100);
  const s = Math.min((W - 2 * m) / pw, (H - 2 * m) / ph);
  const w = Math.round(pw * s), h = Math.round(ph * s);
  return { x: Math.round((W - w) / 2), y: Math.round((H - h) / 2), w, h, m };
}
```

For stories, exporting 1:1 and placing the square on a coloured background in Instagram's story editor gives a clearer frame than exporting 9:16.

## Files

| File | Role |
|---|---|
| `index.html` | The app: UI, `layout()`, export. The iOS home-screen icon is embedded as a base64 PNG |
| `manifest.webmanifest` | Standalone display when added to the home screen. Icon is an SVG data URL |
| `sw.js` | Service worker that caches the three files so the app opens offline |

No dependencies, no build step. Serve the three files statically and it works.

## Self-hosting

Put the three files on any HTTPS server. Paths are relative, so a subdirectory such as `https://example.com/yohaku/` works. The service worker only registers over HTTPS or localhost.

To try it locally:

```bash
python3 -m http.server 8000
```

After changing anything, bump `VERSION` in `sw.js`. Devices with the old version fetch the new one on the next open and switch on the open after that.
