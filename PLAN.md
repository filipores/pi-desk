# Pi Desk MVP Plan

## Ziel

Ein generisches Pi-Paket, das pro Projekt einen kleinen priorisierten Workspace für Inbox-Einträge führt.

## Paketform

Wie `sheepdog`/`slop`. Der Baum zeigt die Repository-/Quellstruktur, nicht den npm-Tarball:

```text
pi-desk/
├── package.json
├── pi-extension/
│   ├── index.js
│   ├── package.json
│   └── test/
├── CONTEXT.md
└── PLAN.md
```

Root-`package.json`:

- `name`: `pi-desk`
- `keywords`: `pi-package`, `pi`, `workspace`, `inbox`, `prioritization`
- `pi.extensions`: `./pi-extension/index.js`

Kein Web, keine App, kein Sync, keine DB.

## Speicher

Global privat unter `~/.pi/agent/pi-desk/`.

- Projekt-ID: Git-Root; außerhalb von Git fallback auf `cwd`
- Pro Projekt getrennte JSON-Datei: `projects/<hash>.json`
- Kontextauswahl pro Projekt in derselben Datei oder daneben

Offene Einträge sind die Quelle der Wahrheit. Erledigte Einträge werden hart gelöscht.

## Commands

- `/todo` — gespeicherte Rangliste anzeigen, kein Agent
- `/todo all` — offene Inbox-Einträge aus allen gespeicherten Projekten anzeigen, kein Agent
- `/todo <text>` — Inbox-Eintrag erfassen, bei erstem Projektlauf Setup starten, dann automatisch priorisieren
- `/todo sort` — manuell neu priorisieren
- `/todo done <id>` — Eintrag hart löschen
- `/todo move <id> <rank>` — manuelle Prioritätsvorgabe setzen; künftige Sortierungen respektieren sie
- `/todo clear` — nur mit Bestätigung
- `/todo setup` — Kontextdateien neu suchen/vorschlagen/bestätigen

## Kontext-Setup

Beim ersten `/todo <text>` in einem Projekt:

1. Extension sucht sichere Kontextdateien.
2. Ziel: Docs/Root-Dateien, max. 5 Vorschläge.
3. User bestätigt oder ändert die Auswahl.
4. Auswahl wird pro Projekt gespeichert.

Die Auswahl darf leer bleiben.

## Priorisierung

Nach jedem neuen Eintrag und bei `/todo sort`:

1. Extension liest Inbox-Einträge, bestätigte Kontextdateien und aktuellen geladenen Pi-Kontext.
2. Priorisierungs-Agent bekommt keine Tools.
3. Agent priorisiert automatisch aus Projektkontext, Nutzerzielen und Inbox-Texten; keine Rückfragen.
4. Ergebnis ist eine lineare Rangliste, keine Buckets, keine Scores.
5. Anzeige: Rang + Text + kurzer Grund.
6. Gründe folgen der Sprache der Eingabe.
7. Bei Agent-Ausfall bleibt die bestehende Reihenfolge ohne Fake-Gründe erhalten.

## Sicherheitsgrenzen

- Nie `.env`, Credentials, `node_modules`, `.git`, Build-Artefakte oder große Binärdateien als Kontext lesen/vorschlagen.
- Priorisierungs-Agent hat keine Tools.
- Kontextsuche läuft lokal in der Extension; keine mutierenden Tools.
- `clear` braucht Bestätigung.

## Tests

Minimal:

- Command-Parser: `/todo`, `/todo text`, `/todo done 3`, `/todo move 2 1`, `/todo clear`, `/todo setup`, `/todo sort`
- Projekt-ID-Ermittlung: Git-Root und cwd-Fallback
- JSON-Store: add/list/done/move/clear
- Kontextfilter: Secrets/ignored dirs ausgeschlossen

## Implementierungsentscheidung

Der Priorisierungs-Agent läuft als Pi-Subprozess ohne Tools und antwortet mit JSON.
