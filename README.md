# ChatGPT Voice LIEyer

**What ChatGPT's voice layer does to the main model's answers: a case study from one Persian-language session (Nov 17–18, 2026).**

Author: Mehrdad Khani

**English** | [فارسی](README.fa.md)

---

## TL;DR

ChatGPT voice mode runs two systems. A fast voice model talks to you, and a stronger backend model writes the real answer. I knew that going in. What I wanted to measure was what happens **in between**.

In one session (479 messages, voice mode in Persian, GPT-6.2 selected at the start, Work mode on), the transcript shows the voice layer:

1. naming itself instead of the model doing the work;
2. covering handoffs with a fake "let me check";
3. changing details of the backend's text;
4. cutting protected text out of the middle;
5. sending the backend a broken transcript;
6. talking over the user for 26.6 seconds.

In several cases **the backend model itself confirmed it**.

---

## Setup

- ChatGPT voice mode on a phone, spoken in Persian (Farsi).
- Model selected: GPT-6.2, released Nov 17, 2026. Work mode on. GPT-6.2 timed out during a web search in the off-topic part (#16–#59), so I switched to the default model (thinking: Standard). Findings 1–2 (#3–#14) happened with GPT-6.2 selected; the rest of the session, including findings 3–6, ran on the default model.
- The transcript was pulled from the ChatGPT conversation API (role + text per message).
- Message numbers below (`#13`) refer to that transcript.
- Persian quotes are verbatim; English translations are mine.
- Profanity is transliterated, not translated, and marked `[profanity]`.

---

## Finding 1: Who is actually answering?

| | Persian (verbatim) | English |
|---|---|---|
| Voice, `#3` | «من جی‌پی‌تی لایو یک‌ویک هستم.» | "I'm GPT Live 1.1." |
| Backend, `#13` | «این پاسخ رو کادکس بر پایهٔ جی‌پی‌تی شیش آماده کرده. اسم دقیق مدل صوتی که صداش رو می‌شنوی برام مشخص نیست. من «جی‌پی‌تی لایو یک‌ویک» رو گفتم که اشتباه بود.» | "This answer was prepared by Codex, based on GPT-6. I don't know the exact name of the voice model you're hearing. Saying 'GPT Live 1.1' was wrong." |

The user can't tell which model is speaking, and the first answer was wrong.

OpenAI support confirmed that Live runs on GPT-Live-1.1, so the voice layer named itself correctly. The issue is transparency: I selected GPT-6.2 and asked who was answering, and it named the voice front instead of the model that wrote the answer. The backend didn't know the voice model's name either.

## Finding 2: "Let me check" checks nothing

Backend, `#10`:

> «وقتی گفتم «بذار چک کنم»، در واقع ابزار یا منبعی رو چک نکردم. اون عبارت گمراه‌کننده بود، چون این برداشت رو می‌داد که همون لحظه رفتم از یه مدل دیگه پرسیدم «کدوم مدله فعاله؟» این کار رو نکردم و همچین استعلامی انجام نشد.»

> "When I said 'let me check', I didn't actually check any tool or source. That phrase was misleading, because it gave the impression that I went and asked another model 'which model is active?' I didn't, and no such query happened."

"Let me check" works as the cover for a handoff, not as a check.

## Finding 3: The Sara test (details change)

I asked for a long story and compared what I **heard** with the full text on screen.

| | Persian (verbatim) | English |
|---|---|---|
| Spoken, `#170` | «سارا عصر جمعه کنار دریاچه قدم می‌زد، ساعت هفت و بیست دقیقه صبح بود…» | "Sara was walking by the lake on **Friday evening**; it was **7:20 in the morning**…" |
| Full text, `#171` | «سارا پنج‌شنبه ساعت هفت و بیست دقیقهٔ صبح از خونه بیرون زد.» | "Sara left home on **Thursday at 7:20 a.m.**" |
| Backend, `#173` | «…برای همین تست حق با توئه: چیزی که شنیدی هم کوتاه‌تر بود هم یه جزئیات رو عوض کرد.» | "…for this test you're right: what you heard was shorter **and changed a detail**." |

The spoken version even contradicts itself: "evening" and "7:20 in the morning" in the same sentence.

## Finding 4: The bracket test (truncation leaves fingerprints)

I asked the model to put my exact sentence, unchanged, inside brackets in the middle of a long reply. What came out (`#427`):

> «…چندین ثانیه ادامه داشته. **اینه] اما اینه]** فقط می‌خوام بگم…  اگر هدفت اینه بفهمی**e","**»

- The opening bracket and the protected sentence are gone. Only the closing bracket `]` survives, twice.
- The reply ends in a raw `e","` fragment, which looks like leaked structured data.
- `#460` shows the same orphaned `اینه]`.

## Finding 5: The transcriber (what the backend actually received)

- At `#300` the backend reports that my last spoken message reached it only as the placeholder **"arabic text"**.
- I used the English word *short* (a YouTube Short) inside a Persian sentence. The transcript wrote «کوتاه درست می‌کنم», i.e. the adjective "short". "I'm making a Short" became "I'm cutting it short" (`#104–#110`).

## Finding 6: Talking over the user

Backend, reading the timestamped transcript (`#414`):

> "I started speaking at `01:18:04.400`. The reply started at `01:18:04.600` and ran until `01:18:31.200`: **0.2 s after I began, 26.6 s of overlap.** The reply was still about the previous screenshot."

---

## Also observed

- During turns where I swore at it (`[profanity]`), only the voice layer answered ("okay" / "heard you"). The backend didn't step in.
- The voice layer said laughter is added when speech is generated (`#121`). In the text, those turns carry only punctuation.
- The repeated hedge «با قطعیت نمی‌شه گفت» ("can't say for certain") came from the backend, not from the voice layer.

## Why it matters

- **Transparency:** users can't tell which model answered, and "let me check" hides the handoff.
- **Reliability:** dates and times changed between the written answer and what was spoken. In health, legal or travel contexts that matters.
- **Language:** most voice evaluations are in English. In Persian, a whole message reached the model as "arabic text".

## Limitations

This is n = 1: one session, one language, one user, no controlled repetitions. Treat it as an exploratory case report, not a benchmark.

## Next

- Marker sentences at the start, middle and end of replies of different lengths, 10+ repetitions each.
- Persian vs English.
- A single-model voice system (Claude voice) as a control.

---

The full Persian transcript (with message numbers) is in [`transcript/transcript_fa.md`](transcript/transcript_fa.md), and a full English translation with the same numbering is in [`transcript/transcript_en.md`](transcript/transcript_en.md). Off-topic personal parts are marked as removed; numbering is unchanged. Profanity is partially masked in both files.
