# Griekse Werkwoorden WhatsApp Game

Een kleine WhatsApp-quizbackend voor de Griekse werkwoordenlijst t/m les 17.

## Wat doet dit?

- Stuurt op ingestelde momenten een Grieks werkwoord naar je zoon.
- Hij heeft 5 minuten om te antwoorden met de Nederlandse vertaling.
- De backend geeft direct feedback via WhatsApp.
- Resultaten worden opgeslagen in SQLite.
- Woorden die fout gaan komen vaker terug; woorden die goed gaan komen minder vaak.
- Na elk antwoord vraagt de bot: "Wil je er nog een?"
- Weekscore met beloningen, bijvoorbeeld ijsje / bios-bezoek / t-shirt.
- Kleine uitleg na een fout antwoord.
- Grappige micro-beloningstekst na een goed antwoord.

## Belangrijk voor WhatsApp

Deze backend is gemaakt voor Twilio WhatsApp webhooks.

WhatsApp/Twilio heeft een 24-uurs sessieregel: buiten 24 uur na het laatste bericht van de gebruiker mag je meestal geen vrij tekstbericht sturen, maar moet je een goedgekeurde WhatsApp-template gebruiken. Daarom ondersteunt deze backend twee manieren:

- `TWILIO_CONTENT_SID` leeg: vrije tekstberichten, handig voor sandbox/test en binnen de 24-uurs sessie.
- `TWILIO_CONTENT_SID` gevuld: geplande quizvragen worden als template verstuurd met variabele `{{1}}` voor het Griekse woord.

Feedback na een antwoord kan normaal als vrij bericht, omdat de leerling dan net zelf heeft gereageerd.

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

## WhatsApp spelregels

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
