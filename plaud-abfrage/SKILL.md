---
name: plaud-abfrage
metadata:
  version: "2.1.0"
  author: Mark
description: >-
  Wertet Plaud-Aufnahmen VOLLSTÄNDIG aus – inklusive aller unbenannten Kurz-Memos, deren Inhalt nie im Titel steht. Klassifiziert jede Aufnahme, liest Notes UND Transkripte, ordnet Aufgaben/Themen anhand einer mitwachsenden Wissensbasis (Personen, Projekte, ASR-Korrekturen) korrekt zu und lernt nach jedem Lauf dazu. IMMER verwenden bei: Plaud, Plaud-Aufnahmen, Sprachmemos, Diktate, Aufnahmen auswerten, Themenübersicht aus Aufnahmen, Aufgaben aus Plaud, Wochenrückblick, Tagesrückblick aus Aufnahmen, "was habe ich diktiert/aufgenommen", "gibt es eine Aufnahme zu X", Suche nach Begriffen in Aufnahmen (z. B. TÜV, Versicherung, Termin, Name einer Person). Auch bei einzelnen Nachfragen zu Themen aus früheren Plaud-Auswertungen. Titelsuche allein ist NIEMALS ausreichend – dieser Skill ist Pflicht, sobald Plaud-Daten berührt werden.
---

# Plaud-Abfrage: Vollständige, lernende Auswertung

## Warum es diesen Skill gibt

Am 09.08.2026 wurde eine Themenübersicht erstellt, die nur betitelte Aufnahmen auswertete. Aufgaben wie **TÜV-Termin** und **Autoversicherung klären** wurden übersehen, weil sie in unbenannten 3–17-Sekunden-Memos steckten (Titel = Zeitstempel). Das darf nie wieder passieren.

**Eiserne Regel Nr. 1:** Der Titel einer Aufnahme sagt nichts über ihren Inhalt. Eine Aufnahme mit Zeitstempel-Namen (`2026-08-07 16:03:26`) ist UNGELESEN, bis ihr Transkript geholt wurde.

**Eiserne Regel Nr. 2:** "Nichts gefunden" darf erst gesagt werden, wenn ALLE Aufnahmen im Zeitraum gelesen wurden. Bleibt etwas ungelesen (z. B. sehr lange Aufnahmen), wird das explizit ausgewiesen: "X von Y Aufnahmen ausgewertet, ungelesen: …".

**Eiserne Regel Nr. 3:** Vor jeder Abfrage die **externe Wissensbasis aus dem Arbeitsordner** laden (Auflösungsreihenfolge siehe unten). Nach jeder Abfrage dort ergänzen. Die Live-Wissensbasis ist NIEMALS Teil des Skills.

## Aufnahme-Klassen und Lesestrategie

| Klasse | Erkennung | Strategie |
|---|---|---|
| **A: Betitelt** | Name beginnt mit `MM-DD` + KI-Titel | `get_note` → Summary + Action Items. Note leer? → wie Klasse B/C behandeln. |
| **B: Unbenanntes Kurz-Memo** | Name = Zeitstempel, `duration` < 120 000 ms | `get_transcript` PFLICHT. Meist 1 Satz = 1–3 Aufgaben. Billig, immer alle lesen. |
| **C: Unbenannt, lang** | Name = Zeitstempel, `duration` ≥ 120 000 ms | Erst `get_note` versuchen. Leer (`[]`)? → `get_transcript` paginiert (`next_cursor`), bei sehr langen Aufnahmen mindestens Anfang + gezielte Suche, Rest als "ungelesen" ausweisen. |

**Technik-Fallen:**
- `duration` ist in **Millisekunden** (38000 = 38 s).
- `get_note` liefert bei unbenannten Aufnahmen oft `[]` – das heißt NICHT "kein Inhalt", sondern "keine KI-Notiz erzeugt". Transkript holen!
- `list_files` mit `query` durchsucht nur **Namen**, nie Inhalte. Als Vorfilter ok, als Beleg für "gibt es nicht" wertlos.
- `serial_number` unterscheidet Quellen (Hardware-Gerät vs. App), siehe Wissensbasis.

## Workflow 1: Übersicht / Rückblick ("Themen und Aufgaben der letzten N Tage")

1. `list_files` mit `date_from`/`date_to`.
2. Alle Aufnahmen in Klassen A/B/C einteilen. Anzahl je Klasse nennen.
3. Klasse A: alle Notes holen. Klasse B: **alle** Transkripte holen. Klasse C: Notes, sonst Transkripte.
4. Aufgaben und Themen extrahieren. Jede Aufgabe bekommt: Datum + Uhrzeit der Aufnahme als Quelle.
5. Zuordnung nach Themen-Taxonomie der Wissensbasis. Kurz-Memos, die zeitlich clustern (≤ 15 min Abstand), gehören meist thematisch zusammen (z. B. 07.08. 14:03–14:15 = private Aufgabenserie Auto/Firma).
6. Ausgabe nach Ausgabeformat (unten). Danach: Lernpflicht.

## Workflow 2: Begriffs-/Themensuche ("Gibt es was zu X?")

1. `list_files` mit `query` als Vorfilter über Titel – Treffer notieren.
2. ASR-Varianten des Begriffs mitdenken (Wissensbasis → ASR-Korrekturen): "Perplexity" kann als "Peplexity" transkribiert sein etc.
3. **Unabhängig vom Titel-Ergebnis:** Transkripte aller Klasse-B-Memos im Zeitraum scannen, Klasse-C-Aufnahmen ohne Treffer in der Note ebenfalls prüfen.
4. Erst dann antworten. Fundstellen wörtlich zitieren (kurz), mit Datum/Uhrzeit, unsichere ASR-Stellen mit `(?)` markieren.
5. Kein Treffer → Abdeckung ausweisen: welche Aufnahmen wurden gelesen, welche nicht.

## Zuordnungs-Regeln

- **Themen-Cluster** aus der externen Wissensbasis verwenden; neue Themen dort ergänzen statt Sammelbecken "Sonstiges" aufzublähen.
- **Personen:** Namen über das Personen-Register auflösen (Milli = Millie = dieselbe Person). Unbekannte Namen → in "Neu gelernt" aufnehmen.
- **ASR-Fehler:** Offensichtliche Transkriptionsfehler über die Korrekturliste auflösen. Neue Verdachtsfälle mit `(?)` markieren, nie stillschweigend "korrigieren" und als Fakt ausgeben.
- **Dienstlich vs. privat:** EnBW-/Projekt-Themen = dienstlich; Auto, Firma (eigene GmbH/Auflösung), Familie, Bücher = privat/eigen. Im Zweifel Kontext der Nachbar-Memos nutzen.
- **Keine Erfindungen:** Nur zuordnen, was belegbar in Note/Transkript steht. Vermutungen als solche kennzeichnen.

## Ausgabeformat

```
[Themen-Cluster 1]
- [ ] Aufgabe … (07.08., 14:03)
…
[Nicht sicher zuordenbar]
- "…wörtliches Zitat…" (Datum) – Deutung unklar

Abdeckung: X/Y Aufnahmen ausgewertet (A: n Notes, B: n Transkripte, C: n). Ungelesen: …

🧠 Neu gelernt: [nur wenn vorhanden]
- Person/Projekt/ASR-Korrektur …
```

Kompakt bleiben; auf Mobilgeräten zählt Scanbarkeit. Bei Nachfragen zu Einzelthemen nicht die ganze Übersicht wiederholen.

## Status-Gedächtnis & Delta-Abfragen (v1.1, nach Secretary-Muster)

Die Wissensbasis führt ein **Auswertungs-Log**: letzter Lauf, abgedeckter Zeitraum, Warteliste nicht transkribierter Aufnahmen (IDs + Datum).
- Vor jedem Lauf: Log prüfen. Bei Folge-Abfragen nur das **Delta** seit dem letzten Lauf neu lesen ("Was ist neu seit gestern?") statt alles erneut.
- Warteliste bei jedem Lauf erneut anpingen (`get_transcript`) – sobald der Nutzer in der Plaud-App transkribiert hat, nachziehen und einsortieren.
- **Wiedervorlage mit Alterung (v2.1):** Die Wissensbasis führt wartende Punkte (externe Zuarbeit, ungeklärte Fragen) mit "offen seit"-Datum. Bei jedem Lauf das Alter ausweisen ("⛔ Jens – offen seit 3 Tagen"); ab etwa einer Woche aktiv Nachfassen vorschlagen.
- **Nachkontrolle (v2.1):** Bei jedem Lauf mit dem Vorlauf vergleichen: Was ist weiterhin offen (Überhang), was erledigt, was übertragen? Dauerläufer kennzeichnen statt sie kommentarlos erneut zu listen.
- Nach jedem Lauf: Log und Wiedervorlage fortschreiben.

## Prioritäts-Triage, ABC & Statusmarker (v2.1)

Jedes Todo erhält bei der Ausgabe eine Priorität:
- 🔴 **dringend**: Signalwörter ("dringend", "sofort"), genannte Fristen, wartende externe Personen (Verlage, Kunden)
- 🟡 **diese Woche**: konkrete nächste Schritte ohne Frist
- ⚪ **Backlog/Idee**: Konzepte, "irgendwann", Ideen-Memos

**ABC-Delegations-Dimension** (klassische Sekretariats-ABC-Analyse): Zusätzlich einordnen, WER es tun muss:
- **A** = wichtig, nur der Nutzer selbst kann es (Entscheidungen, Gespräche, Verhandlungen)
- **B** = wichtig, aber delegierbar → Person aus dem Register vorschlagen ("→ Milli?")
- **C** = Routine → mit 🤖 als **Automatisierungskandidat** markieren (Mac Mini / Agent Harness / Skill). Wiederkehrende C-Aufgaben aktiv als Automatisierungs- oder Checklisten-Vorschlag melden.

**Aufgabenraster vor der Ausgabe** (Filterkaskade): Ist die Aufgabe noch nötig? Nein → als "vermutlich obsolet" kennzeichnen statt blind listen. Delegierbar? → B. Automatisierbar? → C/🤖. Sonst → A ausführen.

**Ergebnis- statt Tätigkeits-Formulierung:** Todos immer als Ergebnis formulieren. Nicht "Gespräch mit Vincent", sondern "Mit Vincent klären, ob >40 Windows-Installationen existieren".

Statusmarker im Fließtext: ⛔ blockiert durch X (Abhängigkeit benennen), ❓ Klärung nötig, ✅ als erledigt erwähnt. Abhängige Aufgaben nie kommentarlos als frei verfügbar listen.

## Übergabe-Protokoll (Handoff, v1.1)

Todos sind erst "verarbeitet", wenn sie übergeben wurden. Ziel-Systeme nach Sphäre: **privat → Apple Reminders** (Liste je Cluster), **dienstlich → EnBW Cowork** (bzw. dortige Aufgabenpflege). Auf Wunsch des Nutzers Reminders direkt per Tool anlegen (mit Priorität + Quelle im Notizfeld). Übergebene Todos im Auswertungs-Log als "übertragen" führen, damit sie bei der nächsten Abfrage nicht erneut als offen erscheinen.

## Grenzen (Limitations)

- Nicht transkribierte Aufnahmen sind unsichtbar; der Skill kann Transkription nicht auslösen, nur die Warteliste führen und den Nutzer erinnern.
- ASR-Qualität bei Kurz-Memos ist begrenzt; unsichere Deutungen bleiben mit `(?)` markiert und werden nie als Fakt behandelt.
- Ohne bereitgestellte externe Wissensbasis startet ein Lauf mit dem leeren Template – Zuordnungen sind dann schwächer; Nutzer darauf hinweisen.

## Externe Wissensbasis: Ablage & Auflösung (v2.0)

Der Skill enthält **nur** `references/wissensbasis-template.md` (leere Struktur). Die Live-Daten (Personen, Projekte, ASR-Korrekturen, Auswertungs-Log) liegen ausschließlich beim Nutzer.

**Auflösungsreihenfolge beim Laden (erste Fundstelle gewinnt):**
1. **Arbeitsordner** (Cowork / Claude Code): `<Arbeitsordner>/plaud/wissensbasis.md`, ersatzweise jede Datei `*wissensbasis*.md` im Arbeitsordner → lesen UND nach dem Lauf **direkt dort zurückschreiben**.
2. **Claude-App/Web:** vom Nutzer angehängte Datei in `/mnt/user-data/uploads/` (Name enthält "wissensbasis") oder Projektwissen → lesen; Rückschreiben an den Ursprungsort ist hier nicht möglich → am Laufende die **aktualisierte `wissensbasis.md` als Datei ausgeben** mit dem Hinweis, die alte im Arbeitsordner/Projekt zu ersetzen.
3. **Nichts gefunden:** Aus dem Template eine neue Wissensbasis im Arbeitsordner anlegen (bzw. als Datei ausgeben), Nutzer kurz informieren, dass eine frische Basis gestartet wurde.

**Schreibregeln:** Nur ergänzen und aktualisieren, niemals ohne Auftrag kürzen oder löschen. Pro Lauf: Auswertungs-Log fortschreiben, Änderungslog-Eintrag mit Datum, Stand-Datum im Kopf aktualisieren. Unsichere Einträge mit `(?)` übernehmen.

## Lernpflicht ("der Skill lernt alles")

Nach JEDER Abfrage prüfen: neue Personen? Neue Projekte/Begriffe? Neue ASR-Fehlmuster? Neue Themen-Cluster? Neue Routinen des Nutzers? Erledigte/übertragene Todos?

Alles davon in die **externe Wissensbasis** schreiben (Auflösung siehe oben). Im Chat zusätzlich den Block "🧠 Neu gelernt" zeigen, damit der Nutzer die Änderungen sieht. Niemals stillschweigend Wissen verwerfen: Was einmal gelernt wurde (z. B. "Reifen Reber macht evtl. TÜV"), muss im nächsten Lauf wieder verfügbar sein.
