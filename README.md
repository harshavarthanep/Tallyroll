# Tallyroll

A screen recorder that runs entirely in a browser tab. No watermark, no time limit, no account to record, and no server to pay for.

## Replace these files on GitHub Pages

| File | Changed |
|---|---|
| `index.html` | **rewritten** — layout fix, three capture modes, draggable + auto-dodging camera, sharing, built-in viewer |
| `sw.js` | updated (cache bumped to v2, external requests no longer intercepted) |
| `manifest.json` | updated description |
| `icons/*` | unchanged |

Commit, wait for Pages to redeploy, then **hard-refresh** (Ctrl/Cmd+Shift+R). The old service worker caches aggressively; without a hard refresh you'll keep seeing v1.

---

## What was broken

The viewfinder had `aspect-ratio: 16/9` inside a `flex: 1` fit-to-viewport shell. On a short or wide window the aspect ratio forces a height the flex row can't give it, so the box overflowed its container and painted straight over the control deck — which is exactly what the screenshot showed.

The rebuild drops the fit-to-viewport shell entirely. The page now flows naturally and scrolls, the viewfinder is clamped with `max-height: min(58dvh, 600px)`, and the action bar is `position: sticky` at the bottom so Start is always in reach without the layout having to fight for space. Less clever, but it cannot overflow on any viewport.

## New in this version

**Three capture modes.** Screen · Screen + camera · Camera only. The last one matters: `getDisplayMedia` does not exist on any mobile browser — that's an OS restriction, not something code can work around. Camera-only mode uses `getUserMedia`, which mobile *does* support, so the app now genuinely does something useful on a phone instead of showing an error. On mobile it selects that mode automatically and says why.

**Camera bubble you control.**
- *Place it myself* — drag the bubble anywhere on the viewfinder, including mid-recording. Pointer events, so it works with mouse, trackpad and touch.
- *Auto-dodge* — every 2.2 seconds the app downsamples the frame to 64×36, measures luminance variance in each corner, and glides the bubble to the calmest one. Busy region = high variance = something you're probably showing, so it moves away. Hysteresis (a 0.75 bias toward staying put) stops it ping-ponging between two similar corners.
- Size (small/medium/large) and shape (circle/rounded).

**Sharing with expiry** — see below.

**Rest of it:** device pickers for mic and camera, countdown toggle, keyboard shortcuts (<kbd>R</kbd> record, <kbd>P</kbd> pause, <kbd>S</kbd> download), live file-size readout, a library of your shared links, and toast messages instead of a error bar that could sit under the overflow.

---

## Turning on sharing (5 minutes, free, one time)

Sharing is off until you add a Google client ID. Without it the app still records and downloads perfectly; the share button just explains it isn't configured.

1. [console.cloud.google.com](https://console.cloud.google.com) → create a project
2. **APIs & Services → Library** → enable **Google Drive API**
3. **APIs & Services → OAuth consent screen** → External → fill in the app name and your email → add the scope `.../auth/drive.file` → publish
4. **Credentials → Create credentials → OAuth client ID → Web application**
5. Under **Authorised JavaScript origins** add `https://YOURNAME.github.io`
6. Copy the client ID into the top of `index.html`:

```js
const GOOGLE_CLIENT_ID = '1234....apps.googleusercontent.com';
```

All of this is free with no card. Step 5 is the one people get wrong — the origin must be the bare origin, no path and no trailing slash.

### How sharing works

The video uploads to **the sharer's own Google Drive**, not yours. Every Google account has 15 GB free, so your hosting bill stays zero no matter how popular the app gets — which is the only version of "shareable links" that survives contact with success on a free stack.

- Scope is `drive.file`: the app can only touch files it created itself. It cannot see anything else in their Drive, and the consent screen says so.
- Upload is resumable with a real progress bar, so a large recording survives a network hiccup.
- The file is set to *anyone with the link can view*. **Viewers need no account.**
- The link points at your site (`yoursite/#/v/FILEID~EXPIRY`), not at Drive. The viewer page is inside `index.html` and routes on the hash, so GitHub Pages needs no redirect rules and you still only replace one file.

### Expiry

Pick 1 day, 7 days, 30 days or never. It's enforced twice:

1. **The link refuses to play** once the timestamp in the URL has passed, and shows an expired page instead.
2. **The file is deleted from Drive** on the sharer's next visit to Tallyroll, so their 15 GB doesn't silently fill up with old recordings.

The second half only runs when the owner opens the app — a static site has nothing that can run on a schedule. In practice that's fine for a daily-use tool, and they can force it any time from **Your shared links → Delete now**.

The viewer page shows the exact deletion date, the time remaining, a **Download video** button, and a link back to your site. The copy button puts the link, the expiry and your site URL on the clipboard in one block — that's the word-of-mouth loop, and it costs you nothing.

---

## Device support, honestly

| | Screen | Camera only | Share | View a link |
|---|---|---|---|---|
| Chrome / Edge desktop | ✅ | ✅ | ✅ | ✅ |
| Firefox desktop | ✅ (WebM) | ✅ | ✅ | ✅ |
| Safari desktop | ✅ | ✅ | ✅ | ✅ |
| iOS / Android | ❌ *(no browser API exists)* | ✅ | ✅ | ✅ |

The layout is built for every size — the deck is an auto-fitting grid, the action bar wraps to two columns under 520px, and the viewfinder shrinks with the viewport. Watching a shared link works everywhere, including phones, which is the half of the loop that most needs to.

HTTPS is required for capture. GitHub Pages gives you that. `file://` will not work — test locally with `python3 -m http.server` and open `http://localhost:8000`.

## What I deliberately left out

- **Trimming** — doable client-side only by re-recording in real time, which is slow and lossy. Worth adding once you can pull in `ffmpeg.wasm` (~25 MB), not before.
- **Click highlights and drawing on screen** — a web page cannot observe clicks in other windows. Any tool that does this is a native app. Claiming it in a browser tab would be a lie.
- **Background blur** — needs a segmentation model (~5 MB download). Good candidate for the paid tier.

## Monetising this

The things worth charging for are the ones that are pure client-side compute, so they never add a running cost: custom intro/outro clips, the user's *own* logo as a watermark, background blur, trimming. Sell one-time unlocks through Gumroad or Lemon Squeezy — they host checkout and take a cut of actual sales, so there's no monthly bill.

Not ads. This audience showed up specifically because they were tired of being monetised against.

## License

MIT.
