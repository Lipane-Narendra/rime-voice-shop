# Voice Kirana Assistant — DataForge 2026 (Rime AI Track)

A voice-native ordering assistant for small kirana (grocery) stores. A customer
speaks their order out loud; the assistant understands the item and quantity,
confirms the total price, and **speaks the confirmation back using Rime's
text-to-speech API.**

## Why voice is essential

Kirana store customers (and often store owners) are used to speaking, not
typing — many orders happen over a phone call or a counter conversation, not
a screen. If you removed voice from this product, you'd be left with a plain
form, which defeats the purpose: this is meant for hands-busy, screen-light
interactions, the same way an actual shopkeeper takes an order by ear.

## Hard voice problem chosen: Perceived Response Time

**Claim:** the assistant responds to a spoken order in ~3.5–4.5 seconds,
measured from the moment the customer stops speaking to the moment audio
playback actually starts (not just when the server responds). Measurement
shows ~99% of this time is the Rime TTS synthesis call itself, not
client-side or order-parsing overhead.

See `RIME_EVIDENCE.md` for the acceptance test, method, and results.

## PROTOTYPE DEMO VIDEO LINK

https://youtu.be/OhWjuUR_Nkw

## PROTOTYPE SNAPSHOTS

https://drive.google.com/drive/folders/1XHCQURfFnZfvBE5o79t3CHyJyK9GJX9_?usp=sharing




## Architecture

```
Browser (Web Speech API - free, built-in speech-to-text)
   → transcript sent to Node/Express backend
   → backend runs a small transparent rule-based order parser
   → backend calls Rime TTS API (POST /v1/rime-tts)
   → audio streamed back to browser and played
   → browser measures perceived response time (T1 - T0) and displays it
```

- **Speech-to-text:** Browser's native `SpeechRecognition` API (Chrome). No
  external STT service used — keeps the demo simple and free.
- **Order understanding:** A small, transparent keyword + quantity matcher in
  `server.js` (see `understandOrder()`). This is intentionally rule-based, not
  an LLM, so the logic is fully inspectable — disclosed here rather than
  presented as more sophisticated than it is.
- **Text-to-speech:** Rime API (`https://users.rime.ai/v1/rime-tts`), model
  `coda`, speaker `taru` (native Hindi male voice), `lang: "hi"`, format
  `audio/mpeg`. Hindi is only available on Rime's Coda model, not mistv2 — see
  Rime's Coda voice catalog (`taru` and `nadi` are currently Rime's two Hindi
  voices). Change the speaker in `server.js` if you want the other one, or a
  different language/voice entirely.
- **Timing instrumentation:** the frontend records `T0` (end of user speech)
  and `T1` (moment `audio.onplay` fires), and the backend reports its own
  processing time via the `X-Server-Ms` header, so total vs. server-side time
  is visible separately.

## What's real vs. simplified (honesty disclosure)

- The Rime TTS call is **real** — every reply you hear is generated live by
  Rime's API, not pre-recorded.
- The "understanding" of the order is a **simple rule-based parser**, not an
  LLM. This is a deliberate scope decision for a fast build and is disclosed
  here rather than hidden.
- Speech-to-text uses the browser's built-in engine (not part of Rime); this
  is standard and disclosed.

## Setup

1. `npm install`
2. Copy `.env.example` to `.env` and add your real Rime API key.
3. `npm start`
4. Open `http://localhost:3000` in **Google Chrome** (Web Speech API support
   is best there).
5. Click the mic button, speak an order like *"do kilo chawal"* or *"ek kilo
   cheeni"*, and listen to the confirmation.

## Known limitations

- Only recognizes a small fixed catalog of items (चावल/rice, आटा/atta,
  चीनी/sugar, तेल/oil, नमक/salt, घी/ghee) — not a general grocery vocabulary.
- Rime currently offers only 2 Hindi voices (`taru`, `nadi`), both on the
  Coda model — no Hindi option on mistv2.
- Chrome's `hi-IN` speech recognition returns Devanagari script; the item
  matcher checks both Devanagari and a few Latin-script spellings, but is not
  a general transliteration system — unrecognized spellings will fall through
  to the clarification reply.
- No persistent order storage / cart — single-turn confirmation only.
- Speech recognition accuracy depends on the browser's engine and the user's
  microphone/accent; no custom acoustic tuning was done.
- Not tested over real telephony — browser microphone only.

## AI assistance disclosure

Parts of this starter codebase (server logic, frontend UI, README structure)
were drafted with AI assistance (Claude) and reviewed/modified by the team.
The team is responsible for understanding, defending, and extending every
part of this submission.

## License

Code in this repository: MIT License (add your own LICENSE file if needed).
Rime API usage is subject to Rime's own terms of service.
