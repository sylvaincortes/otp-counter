# OTP Counter

A single-file, offline, client-side **TOTP (Time-based One-Time Password)** generator. Enter or generate a Base32 master key, and the page displays a live 6-digit code that automatically refreshes every 30 seconds — the same mechanism used by apps like Google Authenticator or Microsoft Authenticator.

Everything runs **entirely in your browser**. There is no backend, no build step, no external dependency, and no network call of any kind — the master key never leaves the device it was entered on.

---

## Table of contents

- [Features](#features)
- [How it works](#how-it-works)
- [Getting started](#getting-started)
- [Usage guide](#usage-guide)
- [Advanced options](#advanced-options)
- [Security notes](#security-notes)
- [Browser compatibility](#browser-compatibility)
- [Project structure](#project-structure)
- [FAQ](#faq)
- [References](#references)
- [License](#license)

---

## Features

- **Generate or enter a master key** — create a fresh, cryptographically random Base32 secret with one click, or paste in an existing one.
- **Live OTP display** — a large, monospaced code that updates automatically, with a circular countdown ring showing time remaining before the next refresh (turning amber, then red, as the window closes).
- **One-click copy** — copies the current code to the clipboard.
- **Configurable algorithm** — SHA-1 (default, compatible with standard authenticator apps), SHA-256, or SHA-512.
- **Configurable digit count** — 6 (default) or 8 digits.
- **Configurable period** — the validity window in seconds (default 30, matching the RFC 6238 standard).
- **Optional local persistence** — an explicit "remember this key on this device" checkbox stores the key in the browser's `localStorage` so it survives a page reload; it is never saved unless this box is checked.
- **Fully offline** — no CDN, no web font, no analytics, no external script or stylesheet of any kind. The page works with no internet connection at all, including opened directly from disk (`file://`).
- **Light/dark theme aware** — automatically matches the operating system's color scheme.
- **Responsive** — usable on both desktop and mobile screen widths.

## How it works

This tool implements **TOTP** as defined in [RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238), which itself builds on **HOTP** ([RFC 4226](https://datatracker.ietf.org/doc/html/rfc4226)):

1. The master key you provide (or that is generated for you) is a shared secret, encoded in **Base32** ([RFC 4648](https://datatracker.ietf.org/doc/html/rfc4648)) — the same encoding used by every major authenticator app.
2. The current Unix time is divided by the period (30 seconds by default) and rounded down, producing a **counter** value that only changes once per period.
3. That counter is signed with **HMAC** (SHA-1, SHA-256, or SHA-512) using the secret key, via the browser's native [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API).
4. The resulting signature is "dynamically truncated" down to a short numeric code (6 or 8 digits) following the algorithm specified in the RFC.
5. The page recomputes this every second, refreshing the displayed code whenever the counter rolls over, and animates a countdown ring showing how much of the current period remains.

Because this is a deterministic function of *(secret, time, algorithm, digits, period)*, any other TOTP-compliant application configured with the exact same master key and the same settings (SHA-1, 6 digits, 30-second period being the near-universal defaults) will always compute the **same code at the same moment** — that's what makes it interoperable with standard authenticator apps.

## Getting started

No installation, no build tools, no dependencies. Choose whichever is most convenient:

- **Open directly**: double-click `index.html` (or drag it into a browser window). Everything works immediately, offline.
- **Serve locally** (optional, useful for testing on a phone on the same network):
  ```bash
  python3 -m http.server 8000
  # then open http://localhost:8000
  ```
- **Publish with GitHub Pages**: push this repository to GitHub, enable Pages in the repository settings (branch: `main`, folder: `/root`), and the page will be served at `https://<your-username>.github.io/<repo-name>/`.

## Usage guide

1. **Get a master key.**
   - Click **"Generate random key"** to create a new, cryptographically random 160-bit Base32 secret, or
   - paste an existing Base32 secret into the **Master key** field (spaces are ignored, so keys copied in groups of 4 characters work fine).
2. **Read the live code.** As soon as a valid key is present, the OTP panel appears: a large 6-digit code, a countdown ring, and a label showing the active algorithm/digits/period combination.
3. **Copy the code** with the **Copy** button next to it.
4. **Reveal or hide the key** with the eye icon next to the input field (the field is masked, like a password field, by default).
5. **Remember the key on this device** (optional) by checking the box underneath the key actions. This stores the key in `localStorage`, scoped to this page's origin, so it's pre-filled the next time you open the page **on the same browser and device**. Leave it unchecked to keep the key in memory only, cleared as soon as the page is closed or reloaded.
6. **Clear everything** with the **Clear** button — it empties the field, unchecks "remember", and removes the key from `localStorage`.

## Advanced options

Expand **"Advanced options"** to change:

| Option | Default | Choices | Notes |
|---|---|---|---|
| Algorithm | `SHA-1` | SHA-1 / SHA-256 / SHA-512 | Most authenticator apps (Google Authenticator, Microsoft Authenticator, etc.) only support SHA-1, even though the RFC allows the others. Change this only if the other system you're matching against also supports it. |
| Digits | `6` | 6 / 8 | 6 digits is the near-universal default. |
| Period | `30` | any integer ≥ 10 (seconds) | 30 seconds is the near-universal default. |

Changing any of these settings changes the generated codes — the same master key with different settings produces a **different** sequence of codes. Keep the settings identical on every system that needs to compute matching codes.

## Security notes

This project is a **learning and utility tool**, not a hardened credential vault. Please read this section before relying on it for anything sensitive:

- **No network activity.** The page makes zero network requests (no fonts, no scripts, no analytics, no telemetry). You can verify this yourself by opening the browser's Network tab while using it, or by disconnecting from the internet entirely — it will still work.
- **Nothing is sent anywhere.** The master key and every generated code stay in the browser's memory (and, optionally, in `localStorage`) and are never transmitted.
- **`localStorage` is not encrypted.** If you check "remember this key on this device", the Base32 secret is stored in plain text in the browser's local storage for this page's origin. Anyone with access to that browser profile (or a malicious browser extension with broad permissions) could read it. Only use this option on a personal, trusted device, and use **Clear** to remove it when you're done.
- **Clipboard exposure.** The "Copy" button places the current code on the system clipboard, where other applications can potentially read it until it's overwritten.
- **This is not a replacement for a dedicated authenticator app** for protecting real-world accounts. It's best suited for development, testing, demos, or educational purposes — for example, generating TOTP codes for a service you're building or testing yourself.
- **Secure context.** The Web Crypto API used for the cryptographic computation requires a "secure context." Modern browsers treat both `https://` pages and locally opened `file://` pages as secure contexts, so opening `index.html` directly should work; if a browser ever refuses (some corporate/hardened configurations restrict `file://`), serve the file over `http://localhost` or `https://` instead (see [Getting started](#getting-started)).

## Browser compatibility

Requires a browser with support for the [Web Crypto API](https://caniuse.com/cryptography) (`crypto.subtle`) and `crypto.getRandomValues` — this covers all current versions of Chrome, Edge, Firefox, and Safari, on both desktop and mobile. No polyfills are included, so very old browsers (e.g., Internet Explorer) are not supported.

## Project structure

```
.
├── index.html          # The entire application: markup, styles, and logic in one file
├── README.md           # This file
└── CODE_STRUCTURE.md   # Detailed technical breakdown of index.html
```

See [`CODE_STRUCTURE.md`](./CODE_STRUCTURE.md) for a full walkthrough of how the code is organized.

## FAQ

**My code doesn't match my authenticator app for the same key.**
Double-check that the algorithm, digit count, and period match on both sides (SHA-1 / 6 digits / 30 seconds is the standard combination most apps assume by default). Also verify your system clock — TOTP is time-based, so if the device's clock is significantly out of sync, the codes will not line up.

**Can I use a secret I already generated elsewhere (e.g., from a QR code)?**
Yes — paste the Base32 string associated with that secret into the Master key field. This tool does not scan QR codes; you'll need the raw Base32 value.

**Does this store or transmit my key anywhere outside my browser?**
No. See [Security notes](#security-notes).

**Why isn't there a QR code scanner or generator?**
To keep the tool small, dependency-free, and fully offline. It may be added in a future version.

## References

- [RFC 6238 — TOTP: Time-Based One-Time Password Algorithm](https://datatracker.ietf.org/doc/html/rfc6238)
- [RFC 4226 — HOTP: An HMAC-Based One-Time Password Algorithm](https://datatracker.ietf.org/doc/html/rfc4226)
- [RFC 4648 — The Base16, Base32, and Base64 Data Encodings](https://datatracker.ietf.org/doc/html/rfc4648)
- [MDN — Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)

## License

MIT License
