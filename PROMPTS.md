# How TomarMedis was built: the prompts

TomarMedis (TakeMeds in English) was built in one afternoon by a parent and an AI coding agent: **Gemini in [Google Antigravity](https://antigravity.google/)**, deployed on **Google Cloud**. This file lists the prompts, in order, that reproduce the same process from scratch. You can also start from the code in this repo and use the prompts to adapt it to your family.

Replace the `<PLACEHOLDERS>`. A few habits that made it work:

- Work in a normal local folder, let the agent show a plan first, and review the diffs it proposes.
- Ask it to **verify** (typecheck, build, headless browser screenshots, `curl` against the live URL) instead of trusting a summary.
- Never paste API keys or real health data into prompts or commits. Personal settings (child name, medicines) live only on the device.

---

## 0. Quick prototype (optional, Google AI Studio "Build")

```text
Create a fun web app so my 4-year-old can take his medicines every morning and night.
Big touch-friendly cards for each medicine (name, dose, a drawing of a measuring cup or an
oral syringe), a sun/moon switch, a weekly calendar with stars, a companion character that
cheers him on with a voice, and a reward video when all the doses are taken.
Spanish UI, designed for an iPad.
```

## 1. Turn the prototype into a real project

Attach the prototype (zip) and ask:

```text
This is a prototype of a kids' medicine routine app made in AI Studio. Turn it into a
professional, open-source-ready project.

Themes: PAW Patrol, SuperThings, Frozen, Toy Story, The Lion King and Wish. Each theme has its
own palette, display font, animated background, sound effects and 4-8 characters with their own
lines in Spanish and English.

Flow: the child picks a companion, taps each medicine as it is taken (the character cheers),
and when the dose is complete earns ONE official video of 2-5 minutes (mostly ~3) from that
show, with a "watch another" button that prefers videos not watched yet.

Parents zone behind a grown-up check: child name, language, theme, 1-10 medicines (name,
dose/size, how it is taken, colour, which doses), schedule presets (morning & night, once a
day, every 12/8/6 hours, custom times), reward limits, sound, backup/restore and a setup link
to configure another device.

Natural voices: a server endpoint that uses Google Cloud Text-to-Speech (Gemini-TTS) with a
voice and a style prompt per character, cached; fall back to the device voice.

Stack: React + Vite + TypeScript + Tailwind, a small Node/Express server that serves the app
and the APIs, and a Dockerfile for Cloud Run. Everything configurable through environment
variables. No secrets and no personal data in the repo.

Work in a new local folder. Show me the plan first, then build it, run a typecheck and a
headless-browser smoke test with screenshots for iPad landscape and portrait.
```

## 2. Official videos only

```text
Research the official YouTube channels of each franchise (Disney / Disney Junior, Pixar, PAW
Patrol, Magic Box / SuperThings) for Spanish (Spain) and English. Verify every channel id from
its handle page. Build a curated list of embeddable official videos between 120 and 300
seconds, verified with oEmbed and the watch page, tagged by character. Never invent video ids.

Then add a server endpoint that searches ONLY inside those channels with the YouTube Data API
(safeSearch strict, embeddable, 90-300 s, region aware, titles that clearly belong to the theme,
no trailers, no audio-only uploads, no making-of), cached for 12 hours, with the curated list
as fallback. In the player, enforce the maximum length and skip any video that fails to load.
```

## 3. Character pictures without shipping artwork

```text
Do not commit any official artwork. Characters show a friendly emoji avatar by default.
Add (1) per-device picture upload or URL in the parents zone and (2) a THEME_OVERRIDES_JSON
server setting with an image URL per character, served only to devices that entered the
family code.
```

## 4. Voices that feel like characters

```text
Give each character a Gemini-TTS voice and a short style prompt (for example "a goofy, warm
little snowman who loves warm hugs"). Prefetch the lines the selected companion will say and
the greetings of the others, cache the audio on the server and on the device (Cache Storage),
and play it through Web Audio so iPad Safari does not block it.
```

## 5. Deploy to Google Cloud Run

```text
Write deploy/deploy-cloud-run.sh that reads PROJECT_ID and REGION from the environment,
enables the required APIs, creates a runtime service account with roles/aiplatform.user
(needed for Gemini-TTS), stores a family ACCESS_CODE, an optional YouTube API key and the theme
overrides in Secret Manager, and deploys with `gcloud run deploy --source` (scale to zero,
max 2 instances, public URL).

Then deploy it to project <PROJECT_ID> in <REGION> and verify on the live URL: health check,
login with the family code, a TTS request and a video search.
```

## 6. Family code that does not get in the way

```text
When the server has a family code, show a screen with two buttons of the same size:
"Enter with code" and "Continue without code". Remember the choice on the device and let
parents enter the code later from the parents zone. Change the code to <NEW_CODE>.
```

## 7. Let the child change companion at any time

```text
On the main screen, add an always-visible row with the companions of the current theme
(one tap to switch, the new one greets the child) and a clear "change companion" button in the
header.
```

## 8. Custom domain

```text
I bought <DOMAIN> at <REGISTRAR>. Help me verify it for the Google account that owns the Cloud
Run project, map <DOMAIN> and www.<DOMAIN> to the service, tell me the exact DNS records (DNS
only, no proxy), redirect www to the root domain and monitor until the certificates are issued.
```

## 9. Name, polish and open source

```text
Rename the app to <NAME> (<ENGLISH_NAME> in English) in the UI, the PWA manifest and the
READMEs, without changing internal storage keys. Then prepare the repo for GitHub:
Apache-2.0 license, README in English and Spanish with screenshots, local setup and deploy
docs, a trademark disclaimer, .gitignore/.gcloudignore that keep secrets out, a scan for keys
and personal data, and a zip I can upload.
```

---

## Gotchas we hit (so you don't)

- **Gemini-TTS through Cloud Text-to-Speech** needs `aiplatform.endpoints.predict` on the runtime identity (`roles/aiplatform.user`); without it the server silently falls back to Chirp 3 HD.
- **iPad Safari**: unlock Web Audio on the first tap, use `playsinline` for YouTube, and "Add to Home Screen" so the browser does not clear local data.
- **YouTube**: many English Disney music videos refuse to play when embedded even if oEmbed works; the player must catch errors and skip.
- **Cloudflare DNS**: records pointing to Google must be *DNS only* (grey cloud) or the managed certificate never issues.
- **Changing domain**: device data lives per origin, so export a backup on the old URL and restore it on the new one (or use a setup link).
- **Firebase Hosting** in front of Cloud Run only forwards a cookie named `__session`; Cloud Run domain mapping does not have that limitation.
