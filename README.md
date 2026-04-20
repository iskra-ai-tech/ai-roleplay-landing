# AI Anime Roleplay — Landing

Single-file, ultra-light landing page. Dark purple gradient, horizontal screenshot carousel, animated pink INSTALAR button, Spanish (Mexico) copy. No build step. No frameworks.

## File layout
```
ai-roleplay-landing/
├─ index.html
└─ assets/
   ├─ screen1.jpg
   └─ screen2.jpg
```

## Configure (2 edits in `index.html`)

1. **Facebook Pixel ID** — find this line and replace the string:
   ```js
   var FB_PIXEL_ID = 'YOUR_PIXEL_ID_HERE';
   ```
   Also replace `YOUR_PIXEL_ID_HERE` inside the `<noscript>` `<img>` tag further down.

2. **Install destination URL** (App Store / Play Store / deep link):
   ```js
   var INSTALL_URL = '#';
   ```

That's it. On tap the button fires `fbq('track', 'Lead')` + a custom `InstallClick` event, then navigates to `INSTALL_URL`.

## Publish to GitHub Pages

```bash
cd ai-roleplay-landing
git init
git add .
git commit -m "Initial landing"
git branch -M main
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin main
```

Then in the repo on GitHub: **Settings → Pages → Source: Deploy from a branch → Branch: main / root → Save**. The page will be live at `https://<user>.github.io/<repo>/` in ~30 seconds.

## Notes
- Uses `100dvh` so iOS Safari address bar doesn't push content off-screen.
- `overflow:hidden` on `html,body` + flex layout keeps everything above the fold without any scroll.
- Carousel auto-advances every 3.2s and pauses briefly after user swipes.
- Images were compressed to ~170KB each (from 12MB originals) — full page < 200KB.
- Works on iOS Safari, Android Chrome, desktop Chrome/Firefox/Safari/Edge.
