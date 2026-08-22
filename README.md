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
- **Saved settings** — role, connection code, audio source, streaming quality, password, and an *auto-connect on open* option persist between sessions.
- **Installable PWA** — add it to the home screen; launches full-screen like a native app and works offline after the first load (the pairing handshake still needs internet).
- **Keep screen awake** — an optional toggle that stops the screen from sleeping while the app is in the foreground.
- **Background audio** — a Media Session integration keeps audio playing when the app is minimized on Android, with a play/pause/stop control in the notification shade.
- **Resilient auto-reconnect** — if an established stream drops (a network blip, or the display waking from sleep), the sender detects it and re-establishes the connection automatically, so brief interruptions recover on their own.
- **Optional password** — the receiver can require a password, so only senders who enter it can connect. Useful in shared settings such as a classroom, to stop an unauthorized device from connecting to the display.
- **Compact, responsive layout** — controls arrange into two columns on a wider window (and landscape displays) to save vertical space, and collapse to a single column on a narrow phone.
- **Streaming quality control** — the sender can choose the audio bitrate (Auto, or 32 / 64 / 128 / 256 kbps). Higher settings enable stereo and are better for music; lower settings save bandwidth for voice. This is a sender-side setting and is applied to the live connection.

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

> When you update the app, re-upload the changed files and bump the `CACHE` version string in `sw.js` (e.g. `...-v8` → `...-v9`) so already-installed copies refresh automatically.

---

## Usage

**On the receiving device (e.g. the display):**

1. Open the app and choose **Receive audio**.
2. Enter a short connection code (e.g. `living-room`).
3. Tap **Start listening** once — this is required so the browser allows audio to play.
4. Leave it running. It waits for a sender and reconnects automatically.

**On the sending device (e.g. the PC):**

1. Open the app and choose **Send audio**.
2. Pick the audio source: **Microphone** or **System / tab audio**, and (optionally) the **streaming quality**.
3. Enter the **same code** the receiver is using.
4. Click **Connect & stream**.

> For **system / tab audio** in Chrome/Edge, remember to tick **“Share tab audio”** / **“Share system audio”** in the browser's share dialog, or no sound will be captured. For music, choose **High (128 kbps)** or **Max (256 kbps)** — these also enable stereo.

Turn on **Auto-connect** on the sender to have it reconnect to the saved code automatically the next time the page opens.

### Password protection

To stop an unauthorized device (for example, a student who discovers the app) from connecting to the display:

1. On the **receiver**, set a value in **Require a password**. It is saved on that device, so you only set it once.
2. On the **sender**, enter the **same password**. Senders that supply the wrong password — or none — are refused, and no audio ever reaches the display.

The password is combined with the connection code and hashed (SHA-256) before being sent, so it is not transmitted in plain text during pairing. This is meant as practical prank-prevention for a shared environment, not high-security access control. The password is stored locally on each device (in the browser's local storage) so it persists between uses.

---

## Install as an app (PWA)

Open the live URL in Chrome on each device and use the **⬇ Install this app** button on the page (or the browser menu's **Install app** / **Add to Home screen**). Once installed it runs full-screen and launches from its own icon — ideal for a kiosk-style display.

> **Window size:** there is no web standard for setting an installed PWA's initial window size — the browser decides it. On desktop, Chrome opens at a default size and then reopens at whatever size you last left the window, so resize it once and it sticks. On Android the app runs full-screen, so this isn't a factor there.

---

## Keeping the display running

For a shared Android interactive display, two features work together:

- **Keep screen awake** (in-app toggle) holds the screen on while the app is the visible, foreground app. By design, browsers release this lock the moment the app is minimized, so it can't keep the screen on in the background.
- **Media Session** keeps the *audio* alive when the app is minimized, and is far less likely to be suspended by Android because the OS treats it as an active media player.

For a truly always-on display, also consider these device-level settings, which the web app cannot control:

- Increase the device's **screen timeout**, or enable **Stay awake while charging** (Android Developer Options) for a plugged-in display.
- If background audio ever drops, exempt the app (or the browser) from **battery optimization** in Android settings.

---

## Troubleshooting

**Audio cuts out after a minute or two.** The most common cause is the display's screen turning off — many Android devices default to a 1–2 minute screen timeout, and when the screen sleeps the audio can be suspended. Fixes, in order:

1. Turn on **Keep screen awake** in the app (works while the app is in the foreground).
2. Increase the device's **screen timeout** (or enable **Stay awake while charging** in Developer Options for a plugged-in display).
3. Exempt the browser/PWA from **battery optimization** in Android settings, so the system doesn't suspend it in the background.

The app also **auto-reconnects**: if an established stream drops for any reason, the sender keeps trying and re-establishes the connection automatically once the receiver is reachable again, so brief interruptions recover on their own.

**A sender says "Refused by receiver."** The password doesn't match the one set on the receiver (or the receiver requires one and the sender left it blank). Enter the exact same password on both devices.

**A sender says "No connection / receiver not listening."** The two devices are using different codes, the receiver isn't running, or a device has no internet for the handshake. Confirm the code matches and the receiver shows it's waiting.

---

## Privacy & data

- The **audio stream is peer-to-peer and encrypted** (WebRTC uses DTLS/SRTP) and never touches a third-party server.
- During the brief pairing handshake, the connection codes and the devices' network addresses pass through the public PeerJS rendezvous service. After pairing, nothing does.
- Nothing is stored remotely. Saved settings (including any password) live only in the browser's local storage on each device.

---

## Limitations

- **The public rendezvous service (peerjs.com) requires internet for pairing** and is a free, best-effort service with no uptime guarantee. To remove this dependency, run your own [`peerjs-server`](https://github.com/peers/peerjs-server) on your LAN and point the app's **Custom signaling host** field (under *Advanced*) at it. This keeps the easy code-based flow while staying fully local.
- **Background audio is not guaranteed indefinitely.** A page playing audio is exempt from normal background-freezing and Media Session strengthens this, but under memory pressure or aggressive vendor battery-optimization, Android can still reclaim a backgrounded app.
- **The password is prank-prevention, not strong security.** It stops casual/unauthorized connections on a shared network; it is not a substitute for real access control.
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

Designed and programmed by **Claude Opus 4.8** (High reasoning effort), built at the direction of **Mike Young** — who steered the project, setting the requirements, making the design calls, and guiding and testing each iteration.

## Credits & license

Peer-to-peer connectivity is provided by [**PeerJS**](https://peerjs.com/), which is open source under the MIT license.

You're free to license your copy of this project as you wish; the [MIT License](https://opensource.org/licenses/MIT) is a good default for a small project like this.
