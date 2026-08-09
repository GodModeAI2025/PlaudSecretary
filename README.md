# PlaudSecretary

> **Der Sekretär für deine Plaud-Aufnahmen.** Ein Claude Skill, der *jede* Aufnahme liest – auch die unbenannten Drei-Sekunden-Memos, in denen der TÜV-Termin steckt.

[![Skill Version](https://img.shields.io/badge/skill-v2.1.0-0b7285)](plaud-abfrage/SKILL.md)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-d97706)](https://code.claude.com/docs)
[![Landingpage](https://img.shields.io/badge/Landingpage-live-2b8a3e)](https://godmodeai2025.github.io/PlaudSecretary/)
[![Podcast](https://img.shields.io/badge/think%3AAI-Folge%2052%20Exokortex-c22d1e)](https://think-ai.podigee.io/52-exokortex)

---

## Das Problem

Plaud-Aufnahmen bekommen automatisch KI-Titel – **aber nur die längeren**. Kurze Memos heißen einfach `2026-08-07 16:03:26`. Wer nach Titeln sucht oder nur die betitelten Aufnahmen auswertet, übersieht genau die Zettel-am-Kühlschrank-Notizen, für die man das Gerät eigentlich benutzt.

Genau das ist am 09.08.2026 passiert: Eine Themenübersicht ließ **TÜV-Termin** und **Autoversicherung klären** unter den Tisch fallen, weil beide in unbenannten 3–17-Sekunden-Memos steckten. Dieser Skill ist die Antwort darauf.

## Die Lösung

`plaud-abfrage` erzwingt eine vollständige Auswertung und führt eine **mitwachsende Wissensbasis**, damit jeder Lauf besser wird als der davor.

| | |
|---|---|
| 🔍 **Vollständigkeit erzwungen** | Jede Aufnahme wird klassifiziert (A/B/C) und gelesen. Unbenannte Memos → Transkript ist Pflicht, nicht optional. |
| 📊 **Abdeckung ausgewiesen** | „Nichts gefunden“ gibt es nur mit Beleg: *X von Y Aufnahmen ausgewertet, ungelesen: …* |
| 🧠 **Lernende Wissensbasis** | Personen, Projekte, Themen-Cluster und ASR-Korrekturen wachsen extern mit – niemals im Skill selbst. |
| 🔁 **Delta-Abfragen** | Ein Auswertungs-Log merkt sich den letzten Lauf. „Was ist neu seit gestern?“ liest nur das Delta. |
| 🎯 **Prioritäts-Triage** | 🔴 dringend · 🟡 diese Woche · ⚪ Backlog – plus ABC-Delegation (selbst / delegieren / automatisieren 🤖). |
| ⛔ **Wiedervorlage mit Alterung** | Wartende Punkte tragen ein „offen seit“-Datum. Ab ca. einer Woche schlägt der Skill aktives Nachfassen vor. |
| 📤 **Übergabe-Protokoll** | Todos gelten erst als verarbeitet, wenn sie übergeben sind: privat → Apple Reminders, dienstlich → Cowork. |

## Wie es funktioniert

Der Kern sind drei eiserne Regeln und eine Lesestrategie pro Aufnahme-Klasse.

**Die drei eisernen Regeln**

1. Der Titel sagt nichts über den Inhalt. Eine Aufnahme mit Zeitstempel-Namen ist **ungelesen**, bis ihr Transkript geholt wurde.
2. „Nichts gefunden“ erst, wenn alle Aufnahmen im Zeitraum gelesen wurden – sonst wird die Lücke explizit ausgewiesen.
3. Vor jeder Abfrage die externe Wissensbasis laden, nach jeder Abfrage dort ergänzen.

**Die Aufnahme-Klassen**

| Klasse | Erkennung | Strategie |
|---|---|---|
| **A: Betitelt** | Name beginnt mit `MM-DD` + KI-Titel | `get_note` → Summary + Action Items. Note leer? → wie B/C behandeln. |
| **B: Unbenanntes Kurz-Memo** | Name = Zeitstempel, `duration` < 120 000 ms | `get_transcript` **Pflicht**. Billig, immer alle lesen. |
| **C: Unbenannt, lang** | Name = Zeitstempel, `duration` ≥ 120 000 ms | Erst `get_note`. Leer (`[]`)? → `get_transcript` paginiert; Rest als „ungelesen“ ausweisen. |

**Technik-Fallen**, die der Skill kennt: `duration` ist in Millisekunden · `get_note` liefert bei unbenannten Aufnahmen oft `[]` (heißt *keine KI-Notiz*, nicht *kein Inhalt*) · `list_files` mit `query` durchsucht nur Namen, nie Inhalte.

## Installation

**Voraussetzung:** ein Plaud-MCP-Server ist verbunden (Tools `list_files`, `get_note`, `get_transcript`). Wie diese Anbindung aufgesetzt wird, erklären wir in Folge 52 „Exokortex“ des Podcasts [think:AI](https://think-ai.podigee.io/52-exokortex).

### Claude Code

```bash
git clone https://github.com/GodModeAI2025/PlaudSecretary.git
cp -r PlaudSecretary/plaud-abfrage ~/.claude/skills/
```

Projektweit statt global: nach `<projekt>/.claude/skills/` kopieren.

### Claude Desktop / Web

Ordner `plaud-abfrage/` als ZIP packen und in den Skill-Einstellungen hochladen:

```bash
cd PlaudSecretary && zip -r plaud-abfrage.zip plaud-abfrage
```

## Benutzung

Der Skill zieht bei jeder Plaud-Berührung automatisch – es reicht, normal zu fragen:

```
Was habe ich diese Woche aufgenommen?
Gibt es eine Aufnahme zum Thema TÜV?
Was ist neu seit gestern?
Wochenrückblick aus meinen Plaud-Aufnahmen
```

**Beispiel-Ausgabe**

```
Auto
- [ ] 🔴 A · HU-Termin bei Reifen Reber vereinbaren (07.08., 14:03)
- [ ] 🟡 B · Autoversicherung: Wechseloption bis 30.11. prüfen → Milli? (07.08., 14:07)

Firma
- [ ] ⚪ C 🤖 · Belege monatlich exportieren – Automatisierungskandidat (07.08., 14:15)

[Nicht sicher zuordenbar]
- "…Termin mit Peplexity(?)…" (08.08.) – Deutung unklar

Abdeckung: 14/14 Aufnahmen ausgewertet (A: 3 Notes, B: 9 Transkripte, C: 2). Ungelesen: –

🧠 Neu gelernt:
- Person: Milli (= Millie) – Assistenz, privat
- ASR: „Peplexity“ → „Perplexity“ (sicher)
```

## Die Wissensbasis

Der Skill enthält **nur** das leere Template ([`references/wissensbasis-template.md`](plaud-abfrage/references/wissensbasis-template.md)). Die Live-Daten – Personen, Projekte, Auswertungs-Log – bleiben ausschließlich bei dir.

**Auflösungsreihenfolge beim Laden** (erste Fundstelle gewinnt):

1. **Arbeitsordner** — `<Arbeitsordner>/plaud/wissensbasis.md` (oder jede `*wissensbasis*.md`). Wird gelesen **und** nach dem Lauf direkt zurückgeschrieben.
2. **Claude App/Web** — angehängte Datei mit „wissensbasis“ im Namen oder Projektwissen. Rückschreiben ist hier nicht möglich → der Skill gibt am Laufende die aktualisierte Datei zur Ablösung aus.
3. **Nichts gefunden** — neue Basis aus dem Template, mit kurzem Hinweis an dich.

Geschrieben wird nur ergänzend: niemals ohne Auftrag kürzen oder löschen, Stand-Datum und Änderungslog bei jedem Lauf fortschreiben, unsichere Einträge mit `(?)`.

> **Datenschutz:** Die `.gitignore` dieses Repos hält jede `wissensbasis.md` und den Ordner `plaud/` bewusst außerhalb der Versionskontrolle. Persönliche Aufnahme-Inhalte gehören nicht in ein Git-Repo.

## Grenzen

- Nicht transkribierte Aufnahmen sind unsichtbar. Der Skill kann Transkription nicht auslösen – er führt eine Warteliste und erinnert.
- ASR-Qualität bei Kurz-Memos ist begrenzt. Unsichere Deutungen bleiben mit `(?)` markiert und werden nie als Fakt behandelt.
- Ohne bereitgestellte Wissensbasis startet ein Lauf mit dem leeren Template – die Zuordnungen sind dann schwächer.

## Projektstruktur

```
PlaudSecretary/
├── plaud-abfrage/
│   ├── SKILL.md                          # der Skill selbst
│   └── references/
│       └── wissensbasis-template.md      # leere Struktur, keine Live-Daten
├── .github/
│   └── repo-metadata.md                  # Description, Topics, Pages-Hinweise
├── index.html                            # Landingpage – Pages liefert aus / (root)
├── .nojekyll                             # kein Jekyll-Preprocessing
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Landingpage veröffentlichen

Die Seite liegt unter **https://godmodeai2025.github.io/PlaudSecretary/** und aktualisiert sich bei jedem Push auf `main`.

Konfiguriert ist das über **Settings → Pages → Source: `Deploy from a branch` → Branch `main`, Ordner `/ (root)`**. Deshalb liegt [`index.html`](index.html) im Wurzelverzeichnis und nicht in einem Unterordner – bei dieser Einstellung ist die Repo-Wurzel zugleich die Wurzel der Website. `.nojekyll` reicht das HTML ohne Jekyll-Preprocessing durch.

Ein Actions-Workflow ist bewusst nicht vorgesehen: Der `GITHUB_TOKEN` eines Workflows darf keine Pages-Site anlegen (`Resource not accessible by integration`), die erste Aktivierung bleibt also ohnehin manuell. Branch-Deployment kommt danach ganz ohne Build-Minuten aus.

## Mitwirken

Issues und Pull Requests sind willkommen. Wer den Skill erweitert: Änderungen an `SKILL.md` bitte mit einem Eintrag in [`CHANGELOG.md`](CHANGELOG.md) und einer angehobenen `metadata.version` im Frontmatter.

## Lizenz

[MIT](LICENSE) · © 2026 Mark Zimmermann
