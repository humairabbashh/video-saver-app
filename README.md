# HD Video Saver

No-watermark video downloader — TikTok, YouTube, Facebook, Instagram, X.
HD video (720p / 1080p / 2K / 4K / Max) + MP3 audio. One rewarded ad per
download. Downloads land in the phone's Downloads folder.

**NO Google Play Store** — sideload APK from GitHub Actions (same pipeline
as Royal Games: Node 20, Java 17, `npx cap add android` + `cap sync`,
`./gradlew assembleDebug`).

## Download engine (verified live 2026-10-28)

- **Cobalt** (primary, all platforms): `POST https://api.cobalt.tools/`
  `{"url","videoQuality","filenameStyle":"basic","downloadMode","audioFormat"}`.
  Response `status` = `tunnel` / `redirect` / `picker` / `local-processing`
  / `error` — see `docs/api.md` in imputnet/cobalt.
- **TikWM** (TikTok): `GET https://www.tikwm.com/api/?url=...&hd=1` —
  verified live: `code 0`, `play` + `hdplay` + `wmplay` + `music` URLs.
- **LIVE TEST RESULTS (2026-10-28, from this VM):**
  - `api.cobalt.tools` YouTube → HTTP 400
    `{"status":"error","error":{"code":"error.api.auth.jwt.missing"}}` —
    the main instance now needs JWT auth (Turnstile). YouTube via cobalt
    will usually fail in-app; the app shows the friendly
    "YouTube ne block kiya, thodi der baad dobara try karo" error.
  - `co.wuk.sh` (alt cobalt instance) → unreachable from this VM (HTTP 000).
    Kept as fallback in-app; may work from the phone's network.
  - TikWM TikTok → HTTP 200, real video resolved (title, cover, 110s,
    hdplay + play + music URLs). **TikTok is the reliable platform.**
  - `tdownv4` TikTok fallback → HTTP 500 "All extraction methods failed"
    — NOT used.
  - CORS: TikWM and cobalt both send `Access-Control-Allow-Origin: *`,
    so `fetch()` works from the Capacitor WebView.
- Only **public** videos work. Private / friends-only never work.
- **No Snapchat** — it has no public video URLs; no tool can download it.

## Ads (Google test IDs — swap for real IDs before release)

- App ID `ca-app-pub-3940256099942544~3347511713` (injected into the
  generated AndroidManifest + strings.xml by the workflow).
- Banner `ca-app-pub-3940256099942544/6300978111` (always-on, bottom).
- Interstitial `ca-app-pub-3940256099942544/1033173712` (every 120s).
- Rewarded `ca-app-pub-3940256099942544/5224354917` (one ad per download,
  any quality; download starts only after the reward).

## Download to Downloads folder

`Filesystem.downloadFile({path, url, directory: 'DOWNLOADS'})`
(@capacitor/filesystem v6, native streaming — no base64 in memory).
Fallbacks in code: `EXTERNAL_STORAGE` + `Download/` path, then
fetch → blob → base64 → `writeFile`. Browser preview uses a plain
`<a download>` link. **The on-device write is untested until the user
installs the APK** — verify on the phone that files appear in Downloads.

## Push (Royal Games Codespace)

```
git pull && curl -L -o vds.zip "<hosted-zip-url>" && unzip -o vds.zip && cp -r video-downloader/. . && rm -rf video-downloader vds.zip && git add -A && git commit -m "HD Video Saver v1" && git push
```

Then download the `video-saver-apk` artifact from Actions → uninstall
old → install new.
