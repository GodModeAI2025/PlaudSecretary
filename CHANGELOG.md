# Changelog

Alle nennenswerten Änderungen an `plaud-abfrage` werden hier festgehalten.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

## [Unveröffentlicht]

- Repository-Struktur, README und Landingpage angelegt.

## [2.3.0]

Anlass: Funktionsvergleich mit [Speakr](https://github.com/murtaza-nasir/speakr),
einer selbstgehosteten Transkriptions-App. Übernommen ist, was ohne Oberfläche
und ohne eigene Datenbank funktioniert — die Wissensbasis übernimmt dabei die
Rolle, die dort Konfiguration und Index spielen.

### Hinzugefügt
- **Auswertungsprofile mit Prompt-Variablen** – Standard, Cluster-Profile und der Auftrag des laufenden Gesprächs stapeln sich; Profile zeigen über `{{personen}}`, `{{cluster}}`, `{{asr}}` auf die Wissensbasis, statt deren Inhalt zu kopieren und veralten zu lassen.
- **Termine als eigenes Objekt** – „Termin vereinbaren" ist eine Aufgabe, „Dienstag 10 Uhr" ein Ereignis. Relative Angaben werden gegen das **Aufnahmedatum** aufgelöst, nicht gegen heute, und tragen die Originalformulierung mit. Verbindlichkeit `fix` / `vorgeschlagen`; ohne auflösbares Datum keine Übergabe, sondern Rückfrage.
- **Belegstellen mit Zeitmarke** – `(07.08., 14:03 · 01:12)` statt nur Aufnahmedatum; in Fließtext-Antworten nummerierte Quellen mit `file_id`. Bei reinen KI-Notes entfällt die Zeitmarke, statt sie zu schätzen.
- **Workflow 3: Recherche über die gesamte Bibliothek** – iterativ suchen, lesen, nachschärfen, mit drei Abbruchkriterien und sichtbarem Suchprotokoll. Bei Entscheidungsfragen wird aktiv nach Gegenpositionen gesucht, bevor „niemand hat widersprochen" behauptet wird.
- **Inhaltsindex** in der Wissensbasis – eine Zeile je gelesener Aufnahme macht Recherchen bezahlbar. Er erspart das Wiederholen, ersetzt aber nie das Lesen und nie den Beleg.
- **Lesefreigaben** – gesperrte Cluster, Personen und Aufnahmen werden gelesen, aber nicht zitiert, zusammengefasst oder übergeben. Die Ausgabe weist die Zahl der ausgeklammerten Aufnahmen aus.

## [2.2.0]

Anlass: Lauf 1 vom 14.08.2026 über 48 Aufnahmen (07.–14.08.). Jeder Punkt behebt
einen Fehler, der in diesem Lauf real aufgetreten ist.

### Geändert
- **Klassenerkennung an „kein KI-Titel“ statt an Namensmuster gebunden** – App-Aufnahmen (`AUDIO-*`) fielen zuvor durch das Raster; genau darin steckte die einzige Aufgabe mit harter Frist.
- **Zweistufige Übergabe** mit Vorschau vor dem Schreiben und Abgleich des Ergebnisses gegen die Vorschau.
- **Ein einziges Zielsystem: Todoist.** Der `cluster` bestimmt das Zielprojekt, die Abbildung liegt in der Wissensbasis.
- **Rückkanal** – die Wiedervorlage wird vor der Ausgabe gegen den Zustand im Zielsystem abgeglichen, statt gegen das Gedächtnis der Wissensbasis zu altern.

### Hinzugefügt
- **Werkzeugregeln** – Datumsfilter verpflichtend, `matched`-Gegenprüfung, genau ein Retry bei Transkript-Fehlern, Paginierung für Klasse C, Grenzen des Zielsystems nie schätzen.
- **Referenz-Aufrufe** für alle Klassen inklusive alternativer Transkript-Blöcke.
- **Idempotenzschlüssel** aus Quelldatum und `file_id` im Beschreibungsfeld – ohne ihn legt jeder Folgelauf alles doppelt an.
- Wissensbasis-Template um **„Zielsysteme“** und **„Übergabe-Log“** erweitert.

### Entfernt
- Verzweigung nach privat/dienstlich/eigen und das Feld `sphaere` – ersatzlos, inklusive der Zuordnungs-Regel „Dienstlich vs. privat“ und der Spalte „Sphäre“ in der Themen-Taxonomie.

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
