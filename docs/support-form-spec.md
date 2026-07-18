# JDP / 3DGaze — Support Form & Feedback Email Spec

**Goal:** every support message arrives with the device details already attached, so the user is *never*
asked "what phone / which version?" and the reply can go straight to fixing. This modernizes a
production-proven pattern where the app opened its support surface pre-filled with the device's package,
manufacturer, model, SDK level and ABI (carried on every ticket), plus email/order validation and an
auto-reply.

---

## 1. Two channels (and which one carries device info)

| Channel | Device info available? | Role |
|---|---|---|
| **In-app EMAIL SUPPORT** (Settings → MORE) | **Yes** — the app knows everything | **Primary.** This is the rich, auto-filled path. |
| **Website** support@3dgaze.com | No (static site; browser only) | Fallback / desktop. FAQ + mailto. |

GitHub Pages is static — it can't run the old PHP telemetry receiver. So the device fingerprint is built
**in the app** (where the data actually lives) and dropped into the email body. The website's job is the
FAQ and a plain support address; it points users at the in-app button for anything device-specific.

---

## 2. Device fingerprint (auto-attached)

Original fields → modernized, plus JDP-specific state that saves a round-trip:

| Line | Source | Example |
|---|---|---|
| App | `applicationId` + `AppVersion.Version` (`versionCode`) | `com.LiveImage.JDP_Beta_LWP  0.24.3 (286)` |
| Scene | current `WallpaperProduct.displayName` | `JDP_Beta_LWP` |
| Device | `Build.MANUFACTURER` + `Build.MODEL` | `Google Pixel 7` |
| Android | `Build.VERSION.RELEASE` (`API SDK_INT`) + `Build.SUPPORTED_ABIS[0]` | `14 (API 34) · arm64-v8a` |
| Quality | PlayerPrefs `QualitySetting` | `Medium` |
| Remove Ads | `PremiumManager` owned? | `No` |
| Panning | PlayerPrefs `SwipeCamera.panningMode` | `Native` |
| Weather | mode + source (**no coordinates**) | `Simulated` / `Regional` / `Local` |
| Language | device locale | `en-US` |

**Privacy guardrails (must hold):** no precise location, no account/email, no persistent identifiers
beyond the coarse device model. A city name appears only if the user already opted into weather, and only
city-level. This stays inside the app's existing data posture (nothing new is collected).

---

## 3. Email template (in-app)

The app already builds a `mailto` with app version + model/OS/language (`AppLinks.ContactSupport`). Extend
the body builder to emit the block below (all fields are already reachable: `Build.*`, `AppVersion`,
`PremiumManager`, `PlayerPrefs`).

```
To:      support@3dgaze.com
Subject: [JDP] Support — Pixel 7 — v0.24.3

(Write your question above this line — the details below help us help you faster.)

————————————————————————
App:        com.LiveImage.JDP_Beta_LWP  0.24.3 (286)
Scene:      JDP_Beta_LWP
Device:     Google Pixel 7
Android:    14 (API 34) · arm64-v8a
Quality:    Medium
Remove Ads: No
Panning:    Native
Weather:    Simulated
Language:   en-US
————————————————————————
```

- Send via `Intent.ACTION_SENDTO` `mailto:` with URL-encoded `SUBJECT` + `BODY` extras (the existing
  pattern — see `AppLinks`, which already routes through `NEW_TASK`/activity-guarded launch).
- Keep the "write above this line" hint so users don't delete the block.
- **⚠ Address switch:** in-app EMAIL SUPPORT currently targets `jefro357@gmail.com` (Builder row, since
  0.9.2). Flip it to `support@3dgaze.com` once ImprovMX forwarding for that address is tested end-to-end
  (see the website project notes). The website already uses `support@3dgaze.com`.

---

## 4. Website support (static site)

- **Default (recommended):** keep `mailto:support@3dgaze.com` + the FAQ. Simple, no dependencies. The FAQ
  explicitly points users to the in-app button for device-specific issues.
- **Optional web form** (only if we want submissions without an email client): a hosted form service
  (Formspree / Getform free tier) — GitHub Pages can POST to it. Fields: name (optional), email, message.
  A hidden JS-filled field can carry `navigator.userAgent` (rough OS/browser only — **not** app device
  info). Add a honeypot field for spam. This is a convenience, not the device-info path.

---

## 5. Validation (only if a web form is built)

Mirror the proven rules, minus the parts that don't apply to JDP:

- **Email** — standard format check.
- **Message** — require a real description (≥ ~6 words) before enabling Submit.
- **Name** — optional.
- **No "order number" field.** JDP is free with a single IAP (Remove Ads); Google order IDs aren't ours to
  validate, and Remove Ads auto-restores by Google account. Purchase/refund questions go to the Google Play
  help links instead. The **Remove Ads: yes/no** fingerprint line already tells us premium state without
  asking.
- **Honeypot** hidden field to drop bots.

---

## 6. Auto-reply (only if a form / ticketing inbox is used)

Plain-text, no third-party branding:

```
Thanks for contacting 3DGaze — we've got your message and will reply as soon as we can
(usually within a couple of days).

Meanwhile, our FAQ may already have your answer:
  https://3dgaze.com/#faq

Anything about a Google Play download, payment, order or refund is handled by Google directly:
  Can't download / install  —  https://support.google.com/googleplay/answer/1067233
  Payment problems          —  https://support.google.com/googleplay/answer/4646404
  Order history & receipts  —  https://play.google.com/store/account
  Request a refund          —  https://support.google.com/googleplay/answer/2479637
  Contact Google Play       —  https://support.google.com/googleplay/gethelp

Thanks for using 3DGaze wallpapers!
— 3DGaze Support
```

---

## 7. Build order

1. Extend `AppLinks.ContactSupport` body to emit the fingerprint block (§3). *(app)*
2. Switch the in-app support address to `support@3dgaze.com` after the ImprovMX test. *(app + email)*
3. Website: keep mailto + FAQ (done). Google Play help links live on the site (done). *(site)*
4. Only if wanted later: hosted web form (§4) + auto-reply (§6). *(site + service)*

*applicationId `com.LiveImage.JDP_Beta_LWP`; `versionCode` is build-managed (286 at time of writing) and
stamped from `LivingPhoto_Product.asset`.*
