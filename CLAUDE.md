# CLAUDE.md — ai-roleplay-landing

You are working on a **single-page landing site** that drives iOS App Store installs for the app **Iskra · AI Roleplay Characters** from paid social traffic (primarily Mexico, Spanish-language). This document is your operating manual for every change in this repo. **Read it fully before editing anything.**

---

## 1. Project purpose

One goal only: **convert a paid-ad click into an App Store install**. Every design and code decision must be evaluated against that goal. Nothing else belongs in this codebase — no blog, no analytics dashboard, no multi-page navigation, no framework. The page is a conversion surface.

- **App**: https://apps.apple.com/ua/app/iskra-ai-roleplay-characters/id6761598951
- **App Store ID**: `6761598951`
- **Product**: AI character roleplay — users chat with anime/fictional characters powered by an LLM.
- **Primary market**: **México** (Spanish, Mexican dialect copy)
- **Platform targeting**: **iOS only** (the only install destination is the App Store; no Android version exists yet)

## 2. Live site & repo

- **Production URL**: https://iskra-ai-tech.github.io/ai-roleplay-landing/
- **GitHub repo**: https://github.com/iskra-ai-tech/ai-roleplay-landing (public)
- **Hosting**: GitHub Pages, `main` branch, root path. Pushes to `main` auto-deploy in ~30s.
- **GitHub account**: `iskra-ai-tech`
- **Local path**: `/Users/nikitakalaganov/StudioProjects/ai-roleplay-landing/`
- **Git author email** (local repo config): `277723447+iskra-ai-tech@users.noreply.github.com` — **NEVER** commit with any other email. The user scrubbed personal-email traces from history and does not want them to reappear.

## 3. File layout

```
ai-roleplay-landing/
├─ index.html        ← the entire site (HTML + CSS + JS all inline)
├─ CLAUDE.md         ← this file
├─ README.md         ← setup/deploy summary for humans
├─ .nojekyll         ← disables GitHub Pages' Jekyll pipeline (required)
└─ assets/
   ├─ screen1.avif   ← primary (smallest, served to ~96% of users)
   ├─ screen1.webp   ← fallback
   ├─ screen1.jpg    ← last-resort fallback
   ├─ screen2.avif
   ├─ screen2.webp
   └─ screen2.jpg
```

The images are **720 × 1560 px** iOS app screenshots. Do not upscale. If new screenshots are added, encode all three formats via `avifenc -q 55`, `cwebp -q 78`, and `sips -s format jpeg -s formatOptions 82`. The encoders are installed via `brew install libavif webp`.

## 4. Hard design constraints (do not violate)

| Constraint | Why |
|---|---|
| **No scroll, ever.** Everything must fit in the viewport on iPhone SE and up. | Install CTA must be visible without interaction — every pixel of scroll loses conversions. |
| **Dark purple gradient** (`#3a0d5c → #07001a`) — never white, never bright. | Users run ads at night; white flashes annoy them. Also matches the app's visual identity. |
| **Page must NEVER flash white during load.** | Handled by an inline `<style>html,body{background-color:#0a0122}</style>` placed *before* any other tag inside `<head>`. Keep this first. |
| **Pink animated `INSTALAR` button** — gradient, pulsing shadow, sweeping shine, blurred glow. | This is the one interactive element. It must look pressable. |
| **Spanish, Mexican dialect** — warm, slightly slangy (`a todo dar`, `platica`, `chido`-adjacent). Never Castilian Spanish. | Audience is 18–34 Mexicans. |
| **iOS Safari first.** Then Android Chrome. Then desktop. | ~80% of paid traffic lands on iOS. |
| **Total page weight ≤ 250 KB gzipped.** Currently ~170 KB. | Fast LCP on 4G = higher conversion. Don't add heavyweight libraries. |
| **No frameworks, no build step, no external fonts.** Single HTML file, inline CSS/JS, system font stack. | Zero build complexity, zero supply-chain risk, instant deploys. |

### Known iOS Safari gotchas that are already fixed — don't reintroduce them:
- `background-attachment: fixed` — broken on iOS. Use static background images instead.
- `100vh` alone — gives wrong height when the address bar toggles. Use `100dvh` with `100%` fallback on `html` and `body`.
- Background on `body` only — paint on `html` too, otherwise iOS shows white during overscroll / address-bar reveal.
- `position: fixed` on both `html` and `body` simultaneously — causes rendering glitches on iOS. Put it only on `body`.

## 5. Install-click conversion hook (IMPORTANT)

The FB/Meta pixel has been removed. A generic hook is now exposed for any pixel provider (Reddit, TikTok, Twitter, Google, etc.):

```js
// Somewhere in <head> AFTER you've loaded the pixel's base library:
window.__onInstallClick = function(){
  // Fire conversion here. Examples:
  // Reddit:   rdt('track', 'Lead');
  // TikTok:   ttq.track('ClickButton', {...});
  // Google:   gtag('event', 'conversion', {...});
};
```

The button handler in `index.html` calls `window.__onInstallClick()` synchronously, then waits **150 ms** before navigating to the App Store URL. That gives the pixel beacon time to leave before `window.location` kills the request. **Do not reduce this delay below ~100 ms** or mobile Safari will drop the beacon.

The App Store destination is defined once near the top of `<head>`:
```js
var INSTALL_URL = 'https://apps.apple.com/ua/app/iskra-ai-roleplay-characters/id6761598951';
```
Change this one variable if the URL ever changes. Do not hardcode it elsewhere.

## 6. When adding a new pixel (the user's stated next step — Reddit)

1. Paste the pixel's base script into the `<head>` where the comment placeholder is (near top of `index.html`, right after `<title>`).
2. Define `window.__onInstallClick` in the same `<script>` block to fire the pixel's **conversion event** (for Reddit that's `rdt('track', 'Lead')` or a custom event — confirm with the pixel's own docs).
3. **Do not add a PageView track on load** unless the pixel library requires it — most modern pixels auto-fire PageView on init.
4. Test the fire path: open the live site in Incognito Chrome (no extensions), open DevTools → Network → filter by the pixel's domain, reload, confirm PageView beacon. Click INSTALAR, confirm conversion beacon fires *before* navigation.
5. **Ad blockers will block the pixel** — `ERR_BLOCKED_BY_CLIENT` in the console is an **extension**, not a code bug. Expect ~25–40% event loss from real users. The long-term fix is server-side CAPI-equivalent, not more client code.

## 7. Deploy workflow

The user has `gh` CLI authenticated (`iskra-ai-tech` account). To ship changes:

```bash
cd /Users/nikitakalaganov/StudioProjects/ai-roleplay-landing
git commit -am "<message>"
git push
# Pages rebuilds in ~30s; verify with:
curl -sI https://iskra-ai-tech.github.io/ai-roleplay-landing/ | head -1
```

If a force-push ever becomes necessary (e.g. scrubbing a mistake), **confirm with the user first** — force-pushes on a public repo leave orphaned SHAs accessible for ~90 days, which caused concern before.

## 8. Testing checklist before any push

- [ ] Open `index.html` in Safari (macOS) — verify dark purple fills the entire viewport, no white anywhere.
- [ ] Resize window tiny (< 600 px tall) — INSTALAR button still fully visible, no scroll.
- [ ] Click INSTALAR — confirms redirect to the App Store URL works.
- [ ] Open in Chrome DevTools iPhone emulator — carousel auto-advances, dots update, swipe snaps.
- [ ] Run `curl -sI` on the live URL after push — confirm HTTP 200.
- [ ] If a pixel is installed: open in Incognito, confirm the pixel's events appear in its dashboard's test tool.

## 9. What NOT to do

- ❌ Add a framework (React, Vue, Svelte, anything).
- ❌ Add external fonts (Google Fonts, Typekit). System font stack only.
- ❌ Add multiple pages or routes.
- ❌ Add cookie banners / GDPR modals unless explicitly requested (user's jurisdiction is Mexico-focused ads; EU traffic is minimal here).
- ❌ Commit with any email other than `277723447+iskra-ai-tech@users.noreply.github.com`.
- ❌ Upscale images or add new screenshots without re-encoding all three formats (AVIF/WebP/JPEG).
- ❌ Add `background-attachment: fixed` back.
- ❌ Remove the early-paint inline style block in `<head>` — it prevents the white flash.

## 10. User's working style (observed across this project)

- Prefers **decisive action over questions**. When the path is clear, execute and report.
- Wants **short, direct answers**. No preamble, no hedging. "Do X because Y."
- Frustrated by ceremony and boilerplate. Cuts straight to the core of the task.
- Explicitly values: fast loading, minimalism, correctness on iOS Safari, clean git history.
- Gets frustrated when tools fail silently (e.g. ad-blocker blocking pixel) — lead with the diagnosis, not a lecture.

---

**When in doubt: does this change make the INSTALAR button tap → App Store install flow faster, more reliable, or more likely? If not, don't do it.**
