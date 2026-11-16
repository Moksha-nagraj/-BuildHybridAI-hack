# PlainReport

**A private medical-report explainer that runs on your device first and uses the cloud only when it must, after removing your identity and asking your permission.**

Built for the [#BuildHybridAI 2026](https://devpost.com) hackathon with **Firebase AI Logic hybrid inference**.

- **Live demo:** `TODO: https://your-project.web.app`
- **Demo video (3 min):** `TODO: https://youtube.com/...`
- **Platform:** Web (Chrome), installable as a PWA

> PlainReport explains documents in plain language. It is **not medical advice** and never gives a diagnosis. Always talk to your doctor.

---

## The problem

Lab reports, prescriptions, discharge summaries and insurance letters are full of jargon. Patients and caregivers often leave the clinic unsure what the document says or what to ask next.

Cloud AI can explain them, but it forces a bad choice:

1. **Privacy:** you must upload some of your most sensitive documents to a remote server.
2. **Connectivity:** it fails without a good connection, which is a real barrier on budget phones and limited or expensive data.

## The solution

PlainReport uses hybrid inference so the document stays on your device whenever possible.

1. **On-device first.** If the browser has a local model (Gemini Nano in Chrome), the summary, glossary and doctor questions are generated locally. It is private, free and works offline.
2. **Redact, preview, consent, then cloud.** If the document is too long or the device has no local model, PlainReport removes names, phone numbers, IDs and similar details on your device, shows you exactly what will be sent, and asks for your consent. Only then does the redacted text go to the cloud.
3. **Always transparent.** Every result carries a badge: **"Ran on your device"** or **"Used cloud (redacted)"**.

### What you get

- A plain-language summary of the document
- A glossary of medical terms found in it
- A "what to ask your doctor" list
- A private history of past results, with delete-all

---

## How it uses Firebase AI Logic

All AI calls go through **Firebase AI Logic** using its hybrid inference support: prefer the on-device model when available, fall back to the cloud when it isn't.

| Product | Role in the app |
| --- | --- |
| Firebase AI Logic (hybrid inference) | Runs prompts on-device when a local model exists, otherwise in the cloud |
| App Check | Protects the cloud fallback from abuse (required by AI Logic) |
| Authentication | Anonymous sign-in so each user has a private history |
| Firestore | Stores results only, never the original document |
| Remote Config | Tunes the document length limit and daily cloud-call cap without redeploying |
| Hosting | Serves the app over HTTPS |

**Where to find it in the code:** every AI call lives in one module, `src/ai/hybrid.ts` `TODO: confirm path`, which imports Firebase AI Logic and calls it at runtime. The routing rules live in `src/router/`.

### Routing rules

| Situation | What happens |
| --- | --- |
| Local model available, document within the length limit | Runs fully on-device. No consent screen. |
| Document above the length limit | Offers cloud, with redaction and consent |
| No local model on this device | Same cloud path, with a note that this device has no local model |
| User turned cloud off, or the daily cap is reached | Stays on-device and says what was skipped |

---

## Privacy and redaction

Nothing leaves your device until you have seen exactly what will be sent and agreed.

1. A pattern pass finds phone numbers, emails, ID-like numbers, dates of birth, postal codes and name or address headers.
2. When a local model exists, an on-device pass looks for remaining personal details.
3. Each match is replaced with a typed placeholder such as `[NAME]` or `[PHONE]`.
4. A preview highlights every removal. You can redact more or restore items before confirming.
5. Only the redacted text is sent.

**Limits, stated honestly:** automatic redaction can miss unusual formats, and clinical details can still identify a person. The redacted text should not be treated as anonymous. The app says this in the interface.

Firestore stores results only. The original document and the placeholder map are never saved.

---

## Tech stack

TypeScript, Vite, React, Firebase JS SDK, pdf.js (text extraction from PDFs), a service worker for offline use.

## Getting started

### Prerequisites

- Node.js 20 or newer `TODO: confirm`
- A Firebase project with **Firebase AI Logic** enabled
- Desktop Chrome with the on-device model available, to test the on-device path. See Chrome's built-in AI documentation for current requirements. Check `chrome://on-device-internals` for model status.

### 1. Clone and install

```bash
git clone https://github.com/TODO-YOUR-USERNAME/plainreport.git
cd plainreport
npm install
```

### 2. Configure Firebase

1. Create a Firebase project and add a **web app**.
2. Enable **Firebase AI Logic**, **Authentication** (Anonymous), **Firestore** and **Remote Config**.
3. Copy `.env.example` to `.env.local` and fill in your web app config:

```bash
cp .env.example .env.local
```

```env
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
VITE_APPCHECK_SITE_KEY=
```

### 3. Set up App Check

Firebase AI Logic requires App Check.

1. In the Firebase console, register your web app with a reCAPTCHA provider and put the site key in `VITE_APPCHECK_SITE_KEY`.
2. For local development, use a **debug token**. Enable debug mode in your dev build, run the app, copy the token printed in the browser console, and register it under App Check in the Firebase console.
3. Register your production domain before deploying.

### 4. Remote Config defaults

Create these parameters in Remote Config `TODO: match the names used in code`:

| Parameter | Purpose | Example |
| --- | --- | --- |
| `max_local_chars` | Longest document handled on-device before offering cloud | `TODO` |
| `cloud_enabled` | Master switch for the cloud fallback | `true` |
| `daily_cloud_cap` | Cloud calls allowed per user per day | `TODO` |

### 5. Run

```bash
npm run dev
```

Open the printed local URL in Chrome.

### 6. Build and deploy

```bash
npm run build
npx firebase-tools deploy --only hosting
```

---

## Testing both paths

To see hybrid inference working, test all of these:

| Scenario | Expected result |
| --- | --- |
| Chrome with the local model, short report | Badge: "Ran on your device" |
| Same, with airplane mode on | Still works, badge: "Ran on your device" |
| Very long report | Redaction preview, consent dialog, badge: "Used cloud (redacted)" |
| Device or browser without a local model, such as many budget Android phones | Same cloud path with a no-local-model note |
| User declines consent | Nothing is sent, and the app explains what was skipped |
| Empty or scanned PDF | Clear error message, no crash |

## Project structure

```text
src/
  ai/          # Firebase AI Logic hybrid inference wrapper (TODO: confirm)
  router/      # Routing rules and consent flow
  redaction/   # Pattern and on-device redaction
  components/  # UI: input, preview, results, badge, history
  firebase/    # App Check, Auth, Firestore, Remote Config setup
```

`TODO: adjust to match your real folders.`

---

## Built during the hackathon

`TODO: Describe what you built during the submission period. If this builds on an existing app, state the core value of the earlier app and what is new.`

## Roadmap

- Android and iOS versions using Firebase AI Logic on supported devices
- More languages for explanations
- Photo input for printed reports

## Disclaimer

PlainReport provides general, plain-language explanations of documents for education. It is not a medical device and does not provide medical advice, diagnosis or treatment. Do not rely on it for medical decisions. Contact a qualified healthcare professional with any questions.

## License

Licensed under the **Apache License 2.0**. See [LICENSE](LICENSE).

## Acknowledgements

Built with [Firebase AI Logic](https://firebase.google.com/docs/ai-logic) for the #BuildHybridAI 2026 hackathon.
