# 👀 FocusBuddy

A quiet, **private** webcam "body double" that helps you stay at your desk and on task.
It works like a study-with-me stream or a Focusmate partner: the gentle sense that
*someone's there* keeps you in your seat — without anyone actually watching.

**Everything runs in your browser.** Your camera is analyzed on-device with Google's
MediaPipe face model. No video, image, or frame is ever uploaded — there is no server,
no account, and no database.

![status: gentle by default](https://img.shields.io/badge/default-gentle-34d399) ![privacy: on--device](https://img.shields.io/badge/privacy-on--device-5b8cff)

---

## What it actually measures (and what it can't)

A camera is honest about a few things and clueless about the rest. FocusBuddy sticks to
what it can actually see:

- **Presence** — are you at the desk? (face detected / not)
- **Attention** — is your head roughly pointed at the screen, or turned away? (head pose vs. a calibrated baseline)

That's it. A webcam **cannot** tell hard-thinking from zoning out, or reading from
daydreaming. So FocusBuddy treats the camera as a *presence signal*, not a lie detector —
its real value is the body-double effect, not surveillance. If you want a true
productivity metric, pair it with app/website tracking (see [Roadmap](#roadmap)).

## Features

- 🟢 Live **Focused / Looking-away / Away** indicator with a colored ring
- ⏱️ Running **focused-minutes** timer + session breakdown (focused / distracted / away)
- 🎚️ **Intensity dial** — *Gentle* (default), *Standard*, *Strict*. Gentle just checks in; Strict reminds persistently. All tones stay respectful (nagging and shame backfire).
- 🔔 Optional soft chime on nudges (off by default)
- 📈 **All-time stats** (total focus time, sessions, best focus %, longest streak) stored locally
- 🧭 **Auto-calibration** — measures *your* "looking at screen" pose for ~2s so it adapts to your camera placement
- 🔒 No network calls except loading the model files; **no telemetry**

## Run it

Because browsers only grant camera access in a [secure context](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts),
serve it over `http://localhost` (or deploy to HTTPS). Opening the file directly with
`file://` can block the camera or the model import in some browsers.

```bash
# from the project folder
python3 -m http.server 8000
# then open http://localhost:8000
```

Or with Node:

```bash
npx serve .
```

Then click **Start session** and allow the camera when prompted.

## Deploy free on GitHub Pages

1. Push this repo to GitHub (see below).
2. Repo → **Settings → Pages** → *Source: Deploy from a branch* → branch `main`, folder `/ (root)`.
3. Open the `https://<you>.github.io/focusbuddy/` URL it gives you. HTTPS means the camera just works.

## How it works

```
webcam ──getUserMedia──▶ <video> ──▶ MediaPipe FaceLandmarker (WASM, on-device)
                                          │
                     face present? ◀──────┤
                     head yaw/pitch ◀─────┘
                                          ▼
        calibrate baseline ─▶ classify (focused/distracted/away)
                                          ▼
              debounce 0.7s ─▶ accumulate timers ─▶ nudge if past threshold
```

- Head pose comes from the model's facial transformation matrix (yaw/pitch in degrees).
- A short **debounce** stops the state flickering on a quick glance.
- Deviation is measured from *your* calibrated baseline, so it works whether your camera
  is above, below, or beside your screen.

## Privacy

- Frames are processed in-page and immediately discarded; nothing is stored or sent.
- The only outbound requests are to a CDN for the model + WASM runtime on first load.
  Want it fully offline? Download `@mediapipe/tasks-vision` and the `.task` model, then
  point the paths in `index.html` at your local copies.
- Stats live in `localStorage` and never leave the browser. Reset them from the UI anytime.

## A note on the strict mode

Surveillance-and-shame motivation reliably backfires — people feel watched, get anxious,
and quit within days. That's why FocusBuddy defaults to **Gentle** and keeps even *Strict*
mode encouraging rather than punishing. Make it a tool you don't resent by Wednesday.

## Roadmap

- [ ] Streaks & history charts (currently single best/last)
- [ ] Optional desktop notifications when the tab is in the background
- [ ] Pomodoro mode (work/break cycles)
- [ ] **Real** productivity signal via active-app/website tracking (needs a desktop wrapper like Electron — a webcam alone can't do this)
- [ ] Optional end-of-session AI summary that sends only *text* stats (never video) to an LLM

## Tech

Plain HTML/CSS/JS, no build step. [MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe/solutions/vision/face_landmarker) for on-device face landmarks + head pose.

## License

MIT — see [LICENSE](LICENSE).
