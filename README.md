# LAN Audio Streamer

A single-page web app (installable as a PWA) that streams audio **peer-to-peer** from one device to another over your local network. It was built for a specific use case — streaming audio from a **PC to an Android-based interactive display** — but it works between any two devices with a modern browser.

The audio travels directly between the two devices and never passes through any server. There's nothing to install on the sender, no native app required, and connection settings are remembered between uses.

---

## How it works

The app uses **WebRTC** for the actual audio, which flows directly device-to-device and stays on your LAN.

WebRTC needs a brief "introduction" step (called signaling) before the two devices can find each other. To keep this effortless, the app uses the free public **PeerJS** rendezvous service: each device is identified by a short, memorable **connection code**. The receiver picks a code and waits; the sender enters the same code to connect. Once connected, the rendezvous service is out of the loop entirely — the audio is pure peer-to-peer.

> **The only thing that needs internet is that initial handshake.** The audio itself never leaves your LAN. See [Limitations](#limitations) for how to remove the internet dependency entirely.

---

## Features

- **One page, two roles** — each device chooses to be a **sender** or a **receiver**.
- **Easy code-based pairing** — no IP addresses to type; connect with a short memorable code.
- **Sender audio source** — stream the **microphone** or **system / tab audio** (what's playing on the device).
- **Saved settings** — role, connection code, audio source, and an *auto-connect on open* option persist between sessions.
- **Installable PWA** — add it to the home screen; launches full-screen like a native app and works offline after the first load (the pairing handshake still needs internet).
- **Keep screen awake** — an optional toggle that stops the screen from sleeping while the app is in the foreground.
- **Background audio** — a Media Session integration keeps audio playing when the app is minimized on Android, with a play/pause/stop control in the notification shade.
- **Optional password** — the receiver can require a password, so only senders who enter it can connect. Useful in shared settings such as a classroom, to stop an unauthorized device from connecting to the display.
- **Compact interface** — a small, single-column layout that stays out of the way.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app (UI + logic + the PeerJS library, all inlined). |
| `manifest.webmanifest` | PWA manifest (name, icons, standalone display). |
| `sw.js` | Service worker — enables install + offline caching. |
| `icon-192.png`, `icon-512.png` | App icons. |
| `icon-maskable-512.png` | Maskable icon for Android adaptive icons. |

Everything is static. There is no build step and no backend.

---

## Deploy to GitHub Pages

1. Create a new **public** repository (e.g. `lan-audio-streamer`).
2. Upload all the files above to the repository (**Add file → Upload files → Commit**).
3. Go to **Settings → Pages**, set **Source** to **Deploy from a branch**, choose **main / (root)**, and **Save**.
4. After a minute, your app is live at:

   ```
   https://<your-username>.github.io/<repo-name>/
   ```

HTTPS is automatic on `github.io`, which is required both for PWA installation and for microphone / system-audio capture.

> When you update the app, re-upload the changed files and bump the `CACHE` version string in `sw.js` (e.g. `...-v3` → `...-v4`) so already-installed copies refresh automatically.

---

## Usage

**On the receiving device (e.g. the display):**

1. Open the app and choose **Receive audio**.
2. Enter a short connection code (e.g. `living-room`).
3. Tap **Start listening** once — this is required so the browser allows audio to play.
4. Leave it running. It waits for a sender and reconnects automatically.

**On the sending device (e.g. the PC):**

1. Open the app and choose **Send audio**.
2. Pick the audio source: **Microphone** or **System / tab audio**.
3. Enter the **same code** the receiver is using.
4. Click **Connect & stream**.

> For **system / tab audio** in Chrome/Edge, remember to tick **“Share tab audio”** / **“Share system audio”** in the browser's share dialog, or no sound will be captured.

Turn on **Auto-connect** on the sender to have it reconnect to the saved code automatically the next time the page opens.

### Password protection

To stop an unauthorized device (for example, a student who discovers the app) from connecting to the display:

1. On the **receiver**, set a value in **Require a password**. It is saved on that device, so you only set it once.
2. On the **sender**, enter the **same password**. Senders that supply the wrong password — or none — are refused, and no audio ever reaches the display.

The password is combined with the connection code and hashed (SHA-256) before being sent, so it is not transmitted in plain text during pairing. This is meant as practical prank-prevention for a shared environment, not high-security access control. The password is stored locally on each device (in the browser's local storage) so it persists between uses.

---

## Install as an app (PWA)

Open the live URL in Chrome on each device and use the **⬇ Install this app** button on the page (or the browser menu's **Install app** / **Add to Home screen**). Once installed it runs full-screen and launches from its own icon — ideal for a kiosk-style display.

---

## Keeping the display running

For a shared Android interactive display, two features work together:

- **Keep screen awake** (in-app toggle) holds the screen on while the app is the visible, foreground app. By design, browsers release this lock the moment the app is minimized, so it can't keep the screen on in the background.
- **Media Session** keeps the *audio* alive when the app is minimized, and is far less likely to be suspended by Android because the OS treats it as an active media player.

For a truly always-on display, also consider these device-level settings, which the web app cannot control:

- Increase the device's **screen timeout**, or enable **Stay awake while charging** (Android Developer Options) for a plugged-in display.
- If background audio ever drops, exempt the app (or the browser) from **battery optimization** in Android settings.

---

## Privacy & data

- The **audio stream is peer-to-peer and encrypted** (WebRTC uses DTLS/SRTP) and never touches a third-party server.
- During the brief pairing handshake, the connection codes and the devices' network addresses pass through the public PeerJS rendezvous service. After pairing, nothing does.
- Nothing is stored remotely. Saved settings live only in the browser's local storage on each device.

---

## Limitations

- **The public rendezvous service (peerjs.com) requires internet for pairing** and is a free, best-effort service with no uptime guarantee. To remove this dependency, run your own [`peerjs-server`](https://github.com/peers/peerjs-server) on your LAN and point the app's **Custom signaling host** field (under *Advanced*) at it. This keeps the easy code-based flow while staying fully local.
- **Background audio is not guaranteed indefinitely.** A page playing audio is exempt from normal background-freezing and Media Session strengthens this, but under memory pressure or aggressive vendor battery-optimization, Android can still reclaim a backgrounded app.
- The app currently streams **one-way** (sender → receiver).
- Best supported in **Chrome / Edge**. iOS/Safari PWA and capture support is more limited.

---

## Roadmap ideas

- Fully **offline pairing** mode (QR / copy-paste) that needs no internet at all.
- **Self-hosted signaling server** setup for a fully local deployment.
- True **fullscreen** display mode for kiosk use.
- Optional **two-way** audio.

---

## Authors

Designed and coded by **Claude Opus 4.8** (High reasoning effort), built to the direction of **Mike Young** — who steered the project as architect and product lead: setting the requirements, making the design calls, and guiding each iteration (in the modern vernacular, the *vibe coder* in chief).

## Credits & license

Peer-to-peer connectivity is provided by [**PeerJS**](https://peerjs.com/), which is open source under the MIT license.

You're free to license your copy of this project as you wish; the [MIT License](https://opensource.org/licenses/MIT) is a good default for a small project like this.
