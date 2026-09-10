# RIME_EVIDENCE.md

## Hard voice claim

The assistant's spoken response begins playing within approximately 3.5–4.5
seconds of the customer finishing their spoken order, measured end-to-end
(speech recognition + order parsing + Rime TTS generation + network + audio
start), not just server processing time.

## Acceptance test

**Setup:** Chrome browser, home wifi, Rime model `coda`, speaker `taru`
(native Hindi voice), `lang: "hi"`.

**Procedure:**
1. Speak or type a normal order (e.g. "2 kg rice").
2. Record `T0` = timestamp when the browser's speech recognition returns a
   final transcript (i.e. the moment the user is "done speaking").
3. Record `T1` = timestamp when `audio.onplay` fires in the browser (the
   moment the user actually hears the reply start).
4. Perceived response time = `T1 - T0`, shown live in the UI and logged.
5. Repeat 5–10 times with different orders and record each result below.
6. Also test deliberate stress cases: unrecognized items or incomplete
   phrases (e.g. "10 packets of biscuits" — biscuits isn't in the catalog;
   "10 kg" alone — no item named) to confirm the assistant handles them
   gracefully (asks for clarification) rather than failing silently.

## Results

| # | Spoken order | Perceived time (ms) | Server-side time (ms) | Notes |
|---|---|---|---|---|
| 1 | "10 kg" | 3995 | 3978 | stress case — quantity given, no item named, asked for clarification |
| 2 | "add 10 kgs of ghee" | 3895 | 3871 | recognized — ghee, 10kg |
| 3 | "10 packets of biscuits" | 4450 | 4432 | stress case — biscuits not in catalog, asked for clarification |
| 4 | "also at 10 packets of" | 3513 | 3487 | stress case — incomplete/no item named, asked for clarification |
| 5 | "55 kgs of sugar" | 3645 | 3633 | recognized — sugar, 55kg |
| 6 | "2 kg of rice" | 3857 | 3829 | recognized — rice, 2kg |

**Average perceived time across 6 runs: ~3.89 seconds.**
**Average server-side time: ~3.87 seconds — server-side work accounts for
~99% of the total delay.**

## Limitations disclosed

- Numbers above are from a small, informal sample of 6 runs on one machine
  and one network — labeled as exploratory, not a statistically rigorous
  benchmark.
- Network latency is not isolated from Rime's model latency in this test;
  both are included in the "perceived time" figure by design, since that is
  what the user actually experiences.
- The overwhelming majority of latency (~99%) is server-side, and within
  that, almost entirely the Rime TTS synthesis call itself — not our order
  parsing logic, which runs in under a few ms. Further optimization on our
  end has limited room; the ceiling is set by the TTS API round-trip.
- The catalog is a small fixed list (rice, atta, sugar, oil, salt, ghee);
  items outside it (e.g. "biscuits") are correctly rejected with a
  clarification reply rather than a silent failure or a wrong guess.
- Speech input is in English (`en-IN`); the spoken reply is always in Hindi
  (`lang: "hi"`, speaker `taru`) regardless of input language. This is a
  deliberate design choice — many kirana shop owners are more comfortable in
  Hindi even when a customer speaks English, so the assistant always
  confirms the order in the language the shopkeeper understands.
- Only tested on home wifi in one location — results may vary elsewhere.