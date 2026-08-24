# VORTEX 84

Birthday invitation site for Juho and Mave's combined 42nd (42 + 42 = 84).

**Saturday 26.9.2026 · 17:00–04:00 · Lapinlahdenkatu 16, Helsinki, Building 5**

Single-file static site. No build step, no dependencies. Open `index.html` in a
browser and it works.

## The two things to fill in

Both are constants at the top of the `<script>` block in `index.html`:

```js
const RSVP_URL  = 'PASTE_RSVP_LINK_HERE';        // the RSVP form link
const AUDIO_SOURCES = [ ...m4a, ...mp3 ];   // the party track, best format first
```

**RSVP link.** Until a real URL is in there, the RSVP button stays on the page
but changes to "Link coming soon" when clicked instead of going nowhere. Paste
the form link and it becomes a normal button that opens in a new tab.

**Music.** Currently the Turtlebeats "Dark Synthwave Spectral" track, 2:00 long,
in `assets/audio/`. It ships twice: AAC at 128 kbps (1.9 MB) and the original
MP3 at 256 kbps (3.8 MB). The page checks `canPlayType` and takes the AAC, which
halves the download so the music starts sooner on mobile data; the MP3 is the
fallback for anything that cannot decode AAC. To swap the track, drop new files
in that folder and update the `AUDIO_SOURCES` list. A single entry is fine.

macOS has no MP3 encoder, so the smaller file was made with `afconvert`:

```bash
afconvert -f WAVE -d LEI16 track.mp3 track.wav
afconvert -f m4af -d aac -b 128000 -s 2 track.wav track.m4a
```

The page probes for that file on load and adapts:

- **File present:** the entry screen offers "♫ ENGAGE AUDIO" alongside "Enter in
  silence". Engaging starts the track on loop, shows a floating audio toggle
  bottom-right, and drives the neon glows and grid from the track's actual bass
  and mids through a Web Audio analyser. Volume swells as you scroll deeper.
- **File missing:** the audio button and toggle are hidden, the entry screen just
  says "Enter the vortex", and the visuals fall back to a slow synthetic pulse.
  The page is complete either way.

No browser will start audio on scroll alone, so that first click on the entry
screen is what unlocks it. That is a browser rule, not a choice. A visitor's
mute preference is remembered in `localStorage`.

On iPhones the audio is routed through Web Audio so the visuals can react to the
beat, and iOS silences Web Audio when the physical ringer switch is off. Nothing
in the page can detect that, so a one-off "check the ringer switch" note appears
next to the toggle on iOS. If that turns out to be a real problem for guests, the
fix is to play the audio element directly on iOS and fall back to the synthetic
pulse for the visuals there.

Any format the browser can decode works. To use something other than an MP3,
point `AUDIO_SRC` at it.

## Deploying

Same setup as the HRV Breathing App: push to a GitHub repo and
`.github/workflows/deploy.yml` publishes it to GitHub Pages on every push to
`main`.

```bash
gh repo create vortex-84 --public --source=. --remote=origin --push
```

Then in the repo: Settings → Pages → Source → **GitHub Actions**.

`robots.txt` and a `noindex` meta tag keep the poster, the names and the address
out of search results. The repo itself is public, so anyone with the URL can see
the page. Make the repo private and deploy elsewhere if that matters.

## Files

```
index.html                  everything: markup, CSS, JS
robots.txt                  keeps the page out of search engines
Vortex 84.png               master poster, 1024x1535, not served to browsers
assets/img/                 poster in WebP + JPEG at 1024 and 640, plus a
                            1200x630 crop used for link previews
assets/audio/               the track goes here
.github/workflows/          GitHub Pages deploy
```

To regenerate the image variants after changing the poster:

```bash
python3 - <<'PY'
from PIL import Image
src = Image.open("Vortex 84.png").convert("RGB"); W,H = src.size
for w in (1024, 640):
    im = src.resize((w, round(H*w/W)), Image.LANCZOS)
    im.save(f"assets/img/poster-{w}.jpg", "JPEG", quality=84, optimize=True, progressive=True)
    im.save(f"assets/img/poster-{w}.webp", "WEBP", quality=80, method=6)
crop_h = round(W / (1200/630))
src.crop((0, 105, W, 105+crop_h)).resize((1200, 630), Image.LANCZOS) \
   .save("assets/img/og-1200x630.jpg", "JPEG", quality=88, optimize=True, progressive=True)
PY
```

## Notes

- Countdowns target `2026-09-26T17:00+03:00` (party) and `2026-09-11T23:59+03:00`
  (RSVP deadline). Helsinki is UTC+3 on both dates.
- Honours `prefers-reduced-motion`: animation stops, everything stays readable.
- Local preview: `python3 -m http.server 8084`, then open
  `http://127.0.0.1:8084`. Use a server rather than opening the file directly,
  since the audio probe needs HTTP.
