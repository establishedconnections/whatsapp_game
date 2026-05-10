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
- Meerdere gebruikers met eigen naam, eigen score en eigen herhalingsschema.

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

Laat je zoon eerst `/start` sturen naar de bot. De bot toont dan een kort commando-overzicht en vraagt om zijn naam om een profiel aan te maken.

Bij de eerste keer vraagt de bot:

```text
Hoi! Ik ben je Griekse woordjesbot.

Zo werkt het:
/toets 10 - start een toetsronde van 10 woorden
/toets struikel - toets woorden die eerder fout gingen
/uitleg - oefen foute woorden met meerkeuze en uitleg
/hint - krijg hulp bij een open vraag
status - bekijk je weekscore en beloning

Hoe heet je? Stuur je naam, dan maak ik je profiel aan.
```

Daarna worden score, open quizvragen en herhalingsschema per gebruiker apart bijgehouden.

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

Optioneel kan de bot OpenAI gebruiken om flexibeler te beoordelen. Dat helpt bij synoniemen die niet letterlijk in de database staan en bij kleine spelfouten. De AI krijgt alleen het huidige quizwoord plus de verwachte vertaling als context en mag alleen een gestructureerd oordeel teruggeven; scores en database-updates blijven in de backend-code.

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-5.5
AI_GRADING_ENABLED=true
AI_HINTS_ENABLED=true
AI_MIN_CONFIDENCE=0.72
OPENAI_TIMEOUT_SECONDS=8
```

Als OpenAI uit staat of faalt, valt de bot automatisch terug op de gewone database-check.

## Spelregels

Na elk beantwoord woord krijgt de leerling feedback plus:

```text
Wil je er nog een? Antwoord met ja of nee.
```

Er zijn twee modi:

- `/toets`: echte toetsmodus. Antwoorden tellen mee voor score, beloningen en herhalingsschema.
- `/uitleg`: oefenmodus op recente fouten uit de toets. Antwoorden tellen niet mee; de bot begint speels met multiple choice en legt daarna het woord extra uit met hint en vormen.

Je kunt een toetsronde starten met een vast aantal woorden:

```text
/toets 10
/toets struikel
/toets struikel 8
```

`/toets struikel` gebruikt woorden die eerder fout gingen. Zonder getal gebruikt hij `TOETS_DEFAULT_COUNT`, standaard 10.

`quiz` blijft als oude alias voor `/toets` werken. Als hij `ja`, `meer`, `volgende` of `nog een` stuurt, krijgt hij meteen een nieuw woord in dezelfde modus als net gebruikt.

Als hij `nee`, `stop`, `klaar` of `later` stuurt, stopt de speelsessie rustig.

Met `status`, `score` of `beloning` krijgt hij de weekscore te zien.

Met `/hint`, `hint`, `tip` of `hulp` krijgt hij een hulpje richting het antwoord, zonder dat het antwoord letterlijk verklapt wordt. In `/toets` telt een goed antwoord na een hint standaard voor een half punt in de weekscore:

```env
HINT_SCORE=0.5
```

Hints worden per Grieks woord opgeslagen in de database. De eerste hint kan dus via OpenAI gemaakt worden; daarna wordt dezelfde hint goedkoop hergebruikt voor dat woord.

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
De weekscore rekent met punten: goed zonder hint is 1 punt, goed na een hint is `HINT_SCORE`, fout is 0.
