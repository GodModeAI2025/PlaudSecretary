# Changelog

Alle nennenswerten Änderungen an `plaud-abfrage` werden hier festgehalten.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

## [Unveröffentlicht]

- Repository-Struktur, README und Landingpage angelegt.

## [2.1.0]

### Hinzugefügt
- **Wiedervorlage mit Alterung** – wartende Punkte tragen ein „offen seit“-Datum; ab etwa einer Woche schlägt der Skill aktives Nachfassen vor.
- **Nachkontrolle** – jeder Lauf vergleicht mit dem Vorlauf: Überhang, Erledigtes, Übertragenes. Dauerläufer werden gekennzeichnet.
- **ABC-Delegations-Dimension** – A (nur selbst), B (delegierbar, mit Personenvorschlag), C (Routine, 🤖 Automatisierungskandidat).
- **Aufgabenraster** als Filterkaskade vor der Ausgabe, inklusive Kennzeichnung „vermutlich obsolet“.
- **Ergebnis- statt Tätigkeits-Formulierung** für alle Todos.

## [2.0.0]

### Geändert
- **Externe Wissensbasis** mit definierter Auflösungsreihenfolge (Arbeitsordner → Upload/Projektwissen → Template). Live-Daten liegen ausschließlich beim Nutzer; der Skill enthält nur das leere Template.
- Schreibregeln festgelegt: nur ergänzen, nie ungefragt kürzen, Änderungslog und Stand-Datum je Lauf fortschreiben.

## [1.1.0]

### Hinzugefügt
- **Status-Gedächtnis und Delta-Abfragen** – Auswertungs-Log mit letztem Lauf, abgedecktem Zeitraum und Warteliste nicht transkribierter Aufnahmen.
- **Übergabe-Protokoll** – privat → Apple Reminders, dienstlich → Cowork; übergebene Todos werden im Log als „übertragen“ geführt.
- Prioritäts-Triage 🔴 / 🟡 / ⚪ und Statusmarker ⛔ ❓ ✅.

## [1.0.0]

### Hinzugefügt
- Erstfassung nach dem übersehenen TÜV-Termin vom 09.08.2026.
- Drei eiserne Regeln: Titel sagt nichts über den Inhalt · „Nichts gefunden“ nur mit ausgewiesener Abdeckung · Wissensbasis vor und nach jedem Lauf.
- Aufnahme-Klassen A/B/C mit je eigener Lesestrategie.
- Workflow 1 (Übersicht/Rückblick) und Workflow 2 (Begriffs-/Themensuche) inklusive ASR-Varianten.
