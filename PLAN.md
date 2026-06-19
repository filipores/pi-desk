# pi-todo MVP Plan

## Ziel

Ein generisches Pi-Paket, das pro Projekt eine persönliche, priorisierte To-do-Rangliste führt.

## Paketform

Wie `sheepdog`/`slop`:

```text
pi-todo/
├── package.json
├── pi-extension/
│   ├── index.js
│   ├── package.json
│   └── test/
├── CONTEXT.md
└── PLAN.md
```

Root-`package.json`:

- `name`: `pi-todo`
- `keywords`: `pi-package`, `pi`, `todo`, `prioritization`
- `pi.extensions`: `./pi-extension/index.js`

Kein Web, keine App, kein Sync, keine DB.

## Speicher

Global privat unter `~/.pi/agent/pi-todo/`.

- Projekt-ID: Git-Root; außerhalb von Git fallback auf `cwd`
- Pro Projekt getrennte JSON-Datei: `projects/<hash>.json`
- Kontextauswahl pro Projekt in derselben Datei oder daneben

Offene Einträge sind die Quelle der Wahrheit. Erledigte Einträge werden hart gelöscht.

## Commands

- `/todo` — gespeicherte Rangliste anzeigen, kein Agent
- `/todo <text>` — Inbox-Eintrag erfassen, bei erstem Projektlauf Setup starten, dann automatisch priorisieren
- `/todo sort` — manuell neu priorisieren
- `/todo done <id>` — Eintrag hart löschen
- `/todo move <id> <rank>` — manuelle Prioritätsvorgabe setzen; künftige Sortierungen respektieren sie
- `/todo clear` — nur mit Bestätigung
- `/todo setup` — Kontextdateien neu suchen/vorschlagen/bestätigen

## Kontext-Setup

Beim ersten `/todo <text>` in einem Projekt:

1. Setup-Agent sucht nach Kontextdateien.
2. Tools: lesen + bash, keine mutierenden Tools.
3. Ziel: Docs/Root-Dateien, max. 5 Vorschläge.
4. User bestätigt oder ändert die Auswahl.
5. Auswahl wird pro Projekt gespeichert.

Setup-Agent darf vorschlagen, keinen Kontext zu verwenden.

## Priorisierung

Nach jedem neuen Eintrag und bei `/todo sort`:

1. Extension liest To-dos und bestätigte Kontextdateien.
2. Priorisierungs-Agent bekommt keine Tools.
3. Agent fragt das Priorisierungskriterium jedes Mal ab.
4. Bei Unklarheit stellt er Klärungsfragen einzeln, bis die Rangliste belastbar ist.
5. Ergebnis ist eine lineare Rangliste, keine Buckets, keine Scores.
6. Anzeige: Rang + Text + kurzer Grund.
7. Gründe und Fragen folgen der Sprache der Eingabe.

## Sicherheitsgrenzen

- Nie `.env`, Credentials, `node_modules`, `.git`, Build-Artefakte oder große Binärdateien als Kontext lesen/vorschlagen.
- Priorisierungs-Agent hat keine Tools.
- Nur Setup-Agent darf suchen; mutierende Tools bleiben aus.
- `clear` braucht Bestätigung.

## Tests

Minimal:

- Command-Parser: `/todo`, `/todo text`, `/todo done 3`, `/todo move 2 1`, `/todo clear`, `/todo setup`, `/todo sort`
- Projekt-ID-Ermittlung: Git-Root und cwd-Fallback
- JSON-Store: add/list/done/move/clear
- Kontextfilter: Secrets/ignored dirs ausgeschlossen

## Offene Implementierungsentscheidung

Wie der Agent technisch gespawnt wird:

- bevorzugt: Pi-Subprozess im JSON/print-Modus mit strengem Prompt und strukturiertem Output
- fallback: einfache interne Promptfunktion, falls Subprozess-API zu viel wird
