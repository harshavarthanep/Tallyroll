# Tallyroll

A screen recorder that runs entirely in a browser tab. No watermark, no time limit, no account, no upload.

Capture, camera compositing, audio mixing and encoding all happen on the user's machine using `getDisplayMedia`, `MediaRecorder` and a canvas. There is no backend. That is not a shortcut — it's the product claim, and it's what makes hosting free forever.

---

## What's in the MVP

- Screen, window or tab capture
- Round camera bubble burned into the recording, four corner positions
- Microphone + system audio, mixed live via Web Audio
- 3-2-1 countdown, pause/resume, running timecode and file size
- MP4 where the browser supports it, WebM otherwise (detected at runtime)
- Playback and save without ever leaving the page
- Installable PWA, works offline

## Deploy it (free, ~3 minutes)

1. Create a GitHub repo and push these files to the root of `main`.
2. Settings → Pages → Source: *Deploy from a branch* → `main` / `/ (root)`.
3. Open `https://<user>.github.io/<repo>/`.

HTTPS is required for screen and camera capture — GitHub Pages gives you that automatically. `file://` will not work; if you're testing locally run `python3 -m http.server` and use `http://localhost`.

Cloudflare Pages and Netlify's free tiers work identically if you'd rather have a custom domain.

## Browser reality

| | Screen capture | MP4 out |
|---|---|---|
| Chrome / Edge desktop | yes | yes |
| Firefox desktop | yes | no (WebM) |
| Safari desktop | yes | yes |
| Mobile browsers | **no** | — |

`getDisplayMedia` does not exist on mobile browsers — that's an OS restriction, not something code can route around. The app detects this and says so. Mobile is a viewing and install target, not a recording one.

---

## The one thing this cannot do, and what to do about it

Loom's actual product isn't recording. It's **the link you paste into Slack**. That link needs video hosting, and video hosting is the single most expensive thing on the internet. There is no free tier that survives a viral hit.

So the honest MVP is download-first. Three ways to close the gap without taking on a bill:

1. **Bring-your-own storage.** After recording, offer "Upload to your Google Drive / Dropbox" using their JS SDKs. The user's own quota pays for it. Costs you nothing, and the resulting share link still carries your product.
2. **Peer-to-peer.** WebRTC direct transfer while both tabs are open. Free, genuinely private, but the sender has to stay online.
3. **Charge for hosting.** Free tier downloads; paid tier gets hosted links. This is the only version where hosting costs are covered by the people creating them.

Start with 1. It's a weekend of work and it restores most of the share loop.

## Monetization paths that fit a zero-cost stack

- **Sponsor / donate link** in the footer — lowest friction, lowest ceiling.
- **One-time unlock** via Gumroad or Lemon Squeezy (they host checkout; you pay only a cut of actual sales). Sell the things that are pure client-side compute: custom intro/outro clips, brand watermark of the user's *own* logo, background blur, click-zoom, trimming.
- **Paid hosted sharing**, per above.
- **Open-core**: repo stays MIT, a paid desktop build adds system-wide hotkeys.

Deliberately not on this list: ads. This audience arrived because they were tired of being monetized against, and an ad in the corner of a privacy tool undoes the pitch.

## Where the first users come from

The distribution is built in: every recording gets sent to someone else. Make sure the saved filename and the app itself are memorable, then seed it where the complaint already lives — r/loom, r/software, r/edtech, r/freelance, Show HN, and the "free Loom alternative" question threads that get asked weekly.

Post it as *"I got tired of the 5-minute cap, so I built this — no signup, nothing uploaded, here's the source"*. That framing does better on these forums than a product pitch, because it's true.

## License

MIT. Do what you like with it.
