# Griekse Werkwoorden Quizbot

Een kleine Telegram/WhatsApp-quizbackend voor de Griekse werkwoordenlijst t/m les 17.

## Wat doet dit?

- Stuurt op ingestelde momenten een Grieks werkwoord naar je zoon.
- Hij heeft 5 minuten om te antwoorden met de Nederlandse vertaling.
- De backend geeft direct feedback via Telegram of WhatsApp.
- Resultaten worden opgeslagen in SQLite.
- Woorden die fout gaan komen vaker terug; woorden die goed gaan komen minder vaak.
- Na elk antwoord vraagt de bot: "Wil je er nog een?"
- Weekscore met beloningen, bijvoorbeeld ijsje / bios-bezoek / t-shirt.
- Kleine uitleg na een fout antwoord.
- Grappige micro-beloningstekst na een goed antwoord.

## Aanbevolen: Telegram

Voor dit project is Telegram waarschijnlijk de simpelste optie:

- gratis Bot API
- geen WhatsApp Business-account
- geen approved templates
- geen 24-uurs service-window voor geplande quizvragen
- simpele webhook met HTTPS

Maak een bot via `@BotFather` en zet in `.env`:

```env
BOT_PROVIDER=telegram
TELEGRAM_BOT_TOKEN=123456:...
TELEGRAM_CHAT_ID=
TELEGRAM_WEBHOOK_SECRET=kies-hier-een-lange-random-string
```

Laat je zoon eerst `/start` sturen naar de bot. De backend kan dan zijn `chat_id` uit de events loggen; je kunt die invullen als `TELEGRAM_CHAT_ID` voor geplande vragen.

Telegram webhook URL:

```text
https://whatsapp-game.establishedconnections.com/telegram/webhook
```

Webhook instellen:

```bash
curl -X POST "https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://whatsapp-game.establishedconnections.com/telegram/webhook","secret_token":"<TELEGRAM_WEBHOOK_SECRET>","allowed_updates":["message"]}'
```

## WhatsApp opties

Deze backend kan ook met Twilio of direct met Meta WhatsApp Cloud API werken.

WhatsApp heeft een 24-uurs serviceregel: buiten 24 uur na het laatste bericht van de gebruiker mag je meestal geen vrij tekstbericht sturen, maar moet je een goedgekeurde template gebruiken.

Twilio:

- `BOT_PROVIDER=twilio`
- `TWILIO_CONTENT_SID` leeg: vrije tekstberichten, handig voor sandbox/test en binnen de 24-uurs sessie.
- `TWILIO_CONTENT_SID` gevuld: geplande quizvragen worden als template verstuurd met variabele `{{1}}` voor het Griekse woord.

Feedback na een antwoord kan normaal als vrij bericht, omdat de leerling dan net zelf heeft gereageerd.

## Direct Meta WhatsApp Cloud API

Meta Cloud API is meestal fijner dan de Twilio Sandbox voor langdurig gebruik: geen sandbox join-code, geen Twilio tussenlaag, en je werkt direct met je WhatsApp Business phone number.

De WhatsApp-regel blijft wel hetzelfde: buiten de 24-uurs service window moet je een goedgekeurde template gebruiken. Binnen die window mag vrije tekst.

Zet in `.env`:

```env
BOT_PROVIDER=meta
STUDENT_TO=31612345678
META_GRAPH_VERSION=v25.0
META_PHONE_NUMBER_ID=...
META_ACCESS_TOKEN=...
META_VERIFY_TOKEN=kies-hier-een-lange-random-string
META_APP_SECRET=...
```

In Meta Developers configureer je de webhook callback URL:

```text
https://whatsapp-game.establishedconnections.com/meta/webhook
```

Gebruik dezelfde waarde voor Verify Token als `META_VERIFY_TOKEN`.

Subscribe op WhatsApp webhook events voor incoming messages. Voor geplande quizvragen buiten de 24-uurs window maak je een approved template aan, bijvoorbeeld:

```text
Grieks quizwoord: {{1}}. Wat is de Nederlandse vertaling?
```

Daarna zet je:

```env
META_TEMPLATE_NAME=naam_van_je_template
META_TEMPLATE_LANGUAGE=nl
```

## Starten

Kopieer `.env.example` naar `.env` en vul je waarden in.

```bash
cd /Users/renewagner/Documents/Codex/2026-05-10/files-mentioned-by-the-user-werkwoordenlijst/whatsapp_game
/Users/renewagner/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3 app.py
```

De server draait standaard op:

```text
http://127.0.0.1:8080
```

## Webhook in Twilio

Zet je Twilio WhatsApp incoming message webhook op:

```text
https://jouw-publieke-url/twilio/inbound
```

Voor lokaal testen kun je bijvoorbeeld ngrok gebruiken:

```bash
ngrok http 8080
```

## Handige URLs

```text
GET  /health
GET  /admin/stats
POST /admin/send-now
POST /twilio/inbound
POST /meta/webhook
POST /telegram/webhook
```

Voor `POST /admin/send-now` kun je leeg posten; de backend kiest dan zelf een woord dat aan de beurt is.

## Speelschema instellen

In `.env` kun je instellen wanneer geplande quizvragen mogen komen.

```env
QUIZ_DAYS=mon,tue,wed,thu,fri,sat,sun
QUIZ_WINDOW_START=07:30
QUIZ_WINDOW_END=20:30
QUIZ_BLOCK_WINDOWS=mon-fri 08:15-15:15
```

Dit voorbeeld betekent:

- wel quizzen tussen 07:30 en 20:30
- op alle dagen van de week
- maar niet maandag t/m vrijdag tussen 08:15 en 15:15, dus niet tijdens schooltijd

Je kunt meerdere blokken gebruiken met puntkomma's:

```env
QUIZ_BLOCK_WINDOWS=mon-fri 08:15-15:15; tue 17:00-18:00
```

Handmatige vragen via `/admin/send-now` of antwoorden met `ja` mogen wel meteen doorgaan. Alleen de automatische scheduler houdt zich aan het speelschema.

## Antwoordregels

De leerling mag een deel van de vertaling geven. Bijvoorbeeld bij `leiden, brengen` zijn `leiden` en `brengen` allebei goed.

## Spelregels

Na elk beantwoord woord krijgt de leerling feedback plus:

```text
Wil je er nog een? Antwoord met ja of nee.
```

Als hij `ja`, `quiz`, `meer`, `volgende` of `nog een` stuurt, krijgt hij meteen een nieuw woord.

Als hij `nee`, `stop`, `klaar` of `later` stuurt, stopt de speelsessie rustig.

Met `status`, `score` of `beloning` krijgt hij de weekscore te zien.

Bij een fout antwoord krijgt hij nu een korte uitleg:

- wat het goede antwoord was
- welke Nederlandse woorden geaccepteerd worden
- praesens / imperfectum / aoristus
- eventueel het ezelsbruggetje uit de flitskaarten

Bij een goed antwoord krijgt hij ook een kleine micro-beloning, bijvoorbeeld:

```text
Meme-modus: professor vibes intensify.
```

Je kunt die teksten aanpassen in `.env`:

```env
GOOD_MICRO_REWARDS=Mini-beloning: Grieks brein unlocked.|Level up. Woord verslagen.
MISS_MICRO_TEXTS=Geen drama, dit is precies hoe herhalen werkt.|Deze gaat op de revanche-lijst.
```

## Weekbeloningen

De beloningen staan in `.env`:

```env
WEEKLY_GOAL_MIN_ANSWERS=10
REWARD_60=ijsje
REWARD_75=bios-bezoek
REWARD_90=t-shirt
```

Het percentage telt zodra er minstens `WEEKLY_GOAL_MIN_ANSWERS` woorden in die week zijn beantwoord. De week begint op maandag.
