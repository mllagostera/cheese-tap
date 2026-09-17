# 🧀 cheeseTap — Ruta del Queso

A mobile-first, single-file web app for tracking your way through the 101-cheese
board at **Les Grands Buffets** (Narbonne, France).

Mark what you have tasted, rate it from 1 to 5 cheese wedges, filter by intensity,
and send your personal tasting report to Telegram when you are done. Two separate
profiles (one per diner) keep their notes independently, each locked behind its own
password and stored encrypted in the browser.

No backend, no build step, no dependencies to install: the whole app is one
`index.html` file.

---

## Features

- **101 cheeses**, grouped into five intensity levels (20 / 20 / 20 / 20 / 21).
- **Intensity filter** — show everything, or only one level at a time.
- **Tap to taste, tap to rate** — tapping a card marks the cheese as tasted;
  tapping a wedge gives it a 1–5 rating.
- **Two profiles** — "Sra. Queso" and "Sr. Queso", switchable from the bottom bar,
  each with its own password and its own notes.
- **Encrypted local storage** — notes are serialised to JSON and encrypted with
  AES (CryptoJS) using the profile password before being written to
  `localStorage`. Nothing ever leaves the device unless you send it to Telegram.
- **Telegram export** — builds a Markdown report of everything you tasted, with
  intensity and rating, and posts it through the Telegram Bot API.
- **Offline-friendly UI** — the only network call at runtime is the CryptoJS CDN
  script (and Telegram, on demand).

## Tech stack

| | |
|---|---|
| Markup / styling | Plain HTML + CSS (CSS custom properties, no framework) |
| Logic | Vanilla JavaScript, no modules, no bundler |
| Crypto | [CryptoJS 4.1.1](https://cdnjs.com/libraries/crypto-js) via cdnjs |
| Storage | `localStorage` |
| Integration | Telegram Bot API (`sendMessage`) |

## Getting started

Clone the repository and open the file:

```bash
git clone https://github.com/mllagostera/cheese-tap.git
cd cheese-tap
open index.html      # or just double-click it
```

Because everything is static, you can also serve it locally:

```bash
python3 -m http.server 8000
# then browse to http://localhost:8000
```

Or publish it straight to GitHub Pages — no configuration required.

### First run

1. Pick a profile at the bottom (`👩‍🦰 Sra. Queso` / `👨 Sr. Queso`).
2. Enter a password. On first use you will be asked to confirm creation of the
   profile; that password becomes the encryption key for that profile's notes.
3. Optionally fill in your Telegram bot token and chat ID (see below).
4. Start tapping cheeses.

Use **🔒 Cerrar Perfil** to lock the profile again; the notes stay on the device,
encrypted, until the password is entered once more.

## Telegram setup (optional)

1. Talk to [@BotFather](https://t.me/BotFather) and create a bot — it will give
   you a token like `123456:ABC-def...`.
2. Get your chat ID (for example by messaging your bot and reading
   `https://api.telegram.org/bot<TOKEN>/getUpdates`).
3. Paste both into the login screen. They are remembered under the
   `buffet_tg_token` / `buffet_tg_id` keys.
4. Tap **✈️ Enviar mis notas a Telegram** to send the report.

## Data model

Notes are stored per profile under `buffet_vault_v4_senora` and
`buffet_vault_v4_senor`. Decrypted, the payload is a flat map of cheese id to
rating:

```json
{ "1": 4, "47": 5, "88": 2 }
```

A cheese absent from the map has not been tasted. The catalogue itself lives in
the `cheeses` array inside `index.html` — add or edit entries there to adapt the
app to a different cheese board.

## Security notes

Please read this before assuming the app protects anything valuable.

- **The AES encryption is obfuscation-grade, not attack-grade.** CryptoJS derives
  the key from the password with `EVP_BytesToKey` (MD5, a single iteration and no
  configurable work factor). That is enough to keep a curious tablemate out of
  your notes; it is *not* enough to resist an offline brute-force attack by
  anyone with a copy of the `localStorage` blob.
- **The Telegram bot token is stored in plain text** in `localStorage`, by design
  — the code comments say as much. Anyone with access to the browser profile, or
  to the device, can read it and control the bot. Revoke and regenerate the token
  via BotFather if that ever happens.
- **The token is also exposed client-side** on every request to
  `api.telegram.org`. This is unavoidable without a backend proxy, and is fine for
  a throwaway personal bot — do not reuse a bot that has access to anything else.
- There is no password recovery. Forget the profile password and the notes are
  gone.

## Project layout

```
.
├── index.html   # the entire application: markup, styles, catalogue and logic
└── README.md
```

## License

No license has been declared yet. Until one is added, all rights are reserved.
