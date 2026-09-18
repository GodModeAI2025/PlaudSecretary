---
name: plaud-abfrage
metadata:
  version: "2.3.0"
  author: Mark
description: >-
  Wertet Plaud-Aufnahmen VOLLSTÄNDIG aus – inklusive aller unbenannten Kurz-Memos, deren Inhalt nie im Titel steht. Klassifiziert jede Aufnahme, liest Notes UND Transkripte, trennt Aufgaben von Terminen, belegt jede Aussage mit Aufnahme und Zeitmarke, ordnet anhand einer mitwachsenden Wissensbasis (Personen, Projekte, ASR-Korrekturen, Inhaltsindex) korrekt zu und lernt nach jedem Lauf dazu. IMMER verwenden bei: Plaud, Plaud-Aufnahmen, Sprachmemos, Diktate, Aufnahmen auswerten, Themenübersicht aus Aufnahmen, Aufgaben aus Plaud, Termine aus Aufnahmen, Wochenrückblick, Tagesrückblick aus Aufnahmen, "was habe ich diktiert/aufgenommen", "gibt es eine Aufnahme zu X", Suche nach Begriffen in Aufnahmen (z. B. TÜV, Versicherung, Termin, Name einer Person), sowie bei Rückfragen über die gesamte Aufnahme-Bibliothek ("was haben wir zu X entschieden", "wer war dagegen", "seit wann reden wir über Y"). Auch bei einzelnen Nachfragen zu Themen aus früheren Plaud-Auswertungen. Titelsuche allein ist NIEMALS ausreichend – dieser Skill ist Pflicht, sobald Plaud-Daten berührt werden.
---

# Plaud-Abfrage: Vollständige, lernende Auswertung

## Warum es diesen Skill gibt

Am 09.08.2026 wurde eine Themenübersicht erstellt, die nur betitelte Aufnahmen auswertete. Aufgaben wie **TÜV-Termin** und **Autoversicherung klären** wurden übersehen, weil sie in unbenannten 3–17-Sekunden-Memos steckten (Titel = Zeitstempel). Das darf nie wieder passieren.

**Eiserne Regel Nr. 1:** Der Titel einer Aufnahme sagt nichts über ihren Inhalt. Eine Aufnahme mit Zeitstempel-Namen (`2026-08-07 16:03:26`) ist UNGELESEN, bis ihr Transkript geholt wurde.

**Eiserne Regel Nr. 2:** "Nichts gefunden" darf erst gesagt werden, wenn ALLE Aufnahmen im Zeitraum gelesen wurden. Bleibt etwas ungelesen (z. B. sehr lange Aufnahmen), wird das explizit ausgewiesen: "X von Y Aufnahmen ausgewertet, ungelesen: …".

**Eiserne Regel Nr. 3:** Vor jeder Abfrage die **externe Wissensbasis aus dem Arbeitsordner** laden (Auflösungsreihenfolge siehe unten). Nach jeder Abfrage dort ergänzen. Die Live-Wissensbasis ist NIEMALS Teil des Skills.

## Werkzeugregeln

- **`list_files` immer mit `date_from` und `date_to`.** Ohne Datumsfilter
  entscheidet die Seitengröße über den Umfang. In Lauf 1 verschluckte
  `page_size: 40` acht Aufnahmen eines einzigen Tages.
- **`matched` gegen die Zahl tatsächlich gelesener Aufnahmen prüfen**, nicht
  gegen die Länge der zurückgegebenen Liste. Differenz ≠ 0 heißt: weiterlesen.
- **`get_transcript` bei Fehler genau einmal wiederholen.** Der Endpunkt liefert
  sporadisch HTTP 500; der zweite Versuch war in Lauf 1 erfolgreich. Ein
  unwiederholter Fehlschlag verliert eine Aufnahme stillschweigend.
- **Leeres Transkript ≠ leere Aufnahme.** `[]` bedeutet „noch nicht
  transkribiert" → Warteliste, nicht „nichts gesagt".
- **Klasse C paginieren.** Bei gesetztem `next_cursor` weiterlesen, bis er
  `null` ist.
- **Grenzen des Zielsystems nie schätzen.** Anlegen versuchen und die
  `failures`-Liste auswerten. Eine aus dem Gedächtnis vermutete Plangrenze war
  in Lauf 1 schlicht falsch.

## Klassifikation

Entscheidend ist **nicht das Namensmuster, sondern ob ein KI-Titel vorliegt.**
Ein KI-Titel ist ein vom Dienst erzeugter, inhaltlich sprechender Name.
Alles andere — Zeitstempel, Dateiname, leerer Name — gilt als unbenannt.

| Klasse | Erkennung | Aktion |
|---|---|---|
| **A** | KI-Titel vorhanden (i. d. R. `MM-DD ` + Titel) | `get_note`. Bei leerer Note Rückfall auf `get_transcript`. |
| **B** | Kein KI-Titel, Dauer < 120.000 ms | `get_transcript` — **Pflicht, keine Ausnahme** |
| **C** | Kein KI-Titel, Dauer ≥ 120.000 ms | `get_transcript`, bei Bedarf paginiert über `next_cursor` |

**Bekannte Namensmuster ohne KI-Titel** (alle → B oder C):
- `YYYY-MM-DD HH:MM:SS` — Hardware-Gerät
- `AUDIO-YYYY-MM-DD-HH-MM-SS` — Plaud-App
- Leerer oder rein numerischer Name

Diese Liste ist **nicht abschließend**. Im Zweifel gilt: unbenannt.
Das `serial_number`-Feld unterscheidet die Quelle und erklärt abweichende
Namensmuster — es ist kein Klassifikationskriterium.

**Technik-Fallen:**
- `duration` ist in **Millisekunden** (38000 = 38 s).
- `get_note` liefert bei unbenannten Aufnahmen oft `[]` – das heißt NICHT "kein Inhalt", sondern "keine KI-Notiz erzeugt". Transkript holen!
- `list_files` mit `query` durchsucht nur **Namen**, nie Inhalte. Als Vorfilter ok, als Beleg für "gibt es nicht" wertlos.

## Referenz-Aufrufe

### Zeitraum vollständig erfassen
    list_files(date_from: "2026-08-07", date_to: "2026-08-14")
    → matched notieren. Diese Zahl ist der Nenner der Abdeckungszeile.

### Klasse A
    get_note(file_id: "<id>")
    → leeres Ergebnis? get_transcript(file_id: "<id>")

### Klasse B
    get_transcript(file_id: "<id>")
    → []? Auf die Warteliste, nicht als "nichts gefunden" werten.
    → Fehler? Genau einmal wiederholen.

### Klasse C
    get_transcript(file_id: "<id>", limit: 50)
    → next_cursor != null? get_transcript(file_id: "<id>", cursor: "<token>")

### Alternative Transkript-Blöcke
    block: "transaction"          Standard, Sprecher + Zeitstempel
    block: "transaction_polish"   KI-bereinigt, gleiche Struktur
    block: "outline"              Gliederung

## Auswertungsprofile & Prompt-Variablen (v2.3)

Nicht jede Aufnahme verdient dieselbe Auswertung. Ein Meeting braucht
Entscheidungen und Zusagen, ein Ideen-Memo braucht gerade **keine** Todo-Liste,
ein Diktat für ein Buchkapitel braucht wörtliche Treue statt Zusammenfassung.

Profile liegen in der Wissensbasis (§ 9), **niemals im Skill** — sie sind
Nutzerkonfiguration, keine Skill-Logik. Ein Profil legt fest: Ausgabeabschnitte,
zusätzlich zu extrahierende Felder, Tonfall, und ob überhaupt Todos entstehen.

**Auflösung (stapelnd, spezifischer gewinnt bei Konflikt):**

1. **Standardprofil** — gilt immer, definiert die Grundausgabe.
2. **Cluster-/Schlagwort-Profil** — greift, wenn die Aufnahme einem Cluster oder
   Signalwort der Taxonomie zugeordnet wurde. Mehrere Treffer **stapeln**:
   ergänzende Anweisungen addieren sich, bei echtem Widerspruch entscheidet das
   spezifischere Profil.
3. **Auftrag für diesen Lauf** — was der Nutzer jetzt sagt, schlägt jedes Profil.

Welche Profile gegriffen haben, wird in der Ausgabe genannt (eine Zeile). Ohne
diese Zeile ist für den Nutzer nicht erkennbar, warum zwei Aufnahmen
unterschiedlich behandelt wurden.

**Prompt-Variablen.** Profile bleiben kurz, indem sie auf die Wissensbasis
zeigen statt deren Inhalt zu kopieren. Erlaubt sind:

| Variable | Wird ersetzt durch |
|---|---|
| `{{personen}}` | Personen-Register (§ 2) |
| `{{cluster}}` | Themen-Taxonomie (§ 1) |
| `{{asr}}` | ASR-Korrekturliste (§ 4) |
| `{{wiedervorlage}}` | offene Punkte mit „offen seit" (§ 0b) |
| `{{zeitraum}}` | der Zeitraum dieses Laufs |
| `{{aufnahmedatum}}` | Datum/Uhrzeit **der gerade gelesenen Aufnahme** |
| `{{heute}}` | heutiges Datum |

Kopierte Personen- oder Cluster-Listen in einem Profil sind ein Fehler: sie
veralten still, während die Wissensbasis weiterwächst. Unbekannte Variablen
werden **nicht geraten**, sondern gemeldet.

## Workflow 1: Übersicht / Rückblick ("Themen und Aufgaben der letzten N Tage")

1. `list_files` mit `date_from`/`date_to`.
2. Alle Aufnahmen in Klassen A/B/C einteilen. Anzahl je Klasse nennen.
3. Klasse A: alle Notes holen. Klasse B: **alle** Transkripte holen. Klasse C: Notes, sonst Transkripte.
4. Aufgaben, **Termine** und Themen extrahieren. Jeder Punkt bekommt seine
   Belegstelle: Aufnahmedatum, Uhrzeit und Zeitmarke im Transkript.
5. Zuordnung nach Themen-Taxonomie der Wissensbasis. Kurz-Memos, die zeitlich clustern (≤ 15 min Abstand), gehören meist thematisch zusammen (z. B. 07.08. 14:03–14:15 = private Aufgabenserie Auto/Firma).
6. Auswertungsprofil je Aufnahme bestimmen (siehe oben) und dessen
   Zusatzabschnitte bedienen.
7. Ausgabe nach Ausgabeformat (unten). Danach: Inhaltsindex fortschreiben und Lernpflicht.

## Workflow 2: Begriffs-/Themensuche ("Gibt es was zu X?")

1. **Inhaltsindex prüfen** – bereits gelesene Aufnahmen liefern Treffer, ohne dass ein Transkript erneut geholt wird.
2. `list_files` mit `query` als Vorfilter über Titel – Treffer notieren.
3. ASR-Varianten des Begriffs mitdenken (Wissensbasis → ASR-Korrekturen): "Perplexity" kann als "Peplexity" transkribiert sein etc.
4. **Unabhängig vom Titel-Ergebnis:** Transkripte aller Klasse-B-Memos im Zeitraum scannen, Klasse-C-Aufnahmen ohne Treffer in der Note ebenfalls prüfen.
5. Erst dann antworten. Fundstellen wörtlich zitieren (kurz), mit Belegstelle (Datum, Uhrzeit, Zeitmarke), unsichere ASR-Stellen mit `(?)` markieren.
6. Kein Treffer → Abdeckung ausweisen: welche Aufnahmen wurden gelesen, welche nicht.

## Workflow 3: Recherche über die gesamte Bibliothek (v2.3)

Workflow 2 beantwortet „gibt es etwas zu X?" in einem Zeitraum. Workflow 3
beantwortet Fragen **ohne Zeitraum und mit Argumentationsbedarf**: „Was haben
wir zur Preisänderung entschieden, und hat jemand widersprochen?" Solche Fragen
brauchen mehrere Aufnahmen und eine belegte Gegenüberstellung.

1. **Frage zerlegen.** Was ist die Behauptung, was der Beleg, was die
   Gegenposition? Suchbegriffe je Teilfrage bilden, ASR-Varianten und Synonyme
   aus der Wissensbasis dazunehmen.
2. **Inhaltsindex zuerst** (§ 10 der Wissensbasis). Er benennt Kandidaten, ohne
   dass die Bibliothek erneut gelesen wird.
3. **Kandidaten lesen** — Note bei Klasse A, Transkript bei B/C. Nur die
   Kandidaten, nicht alles.
4. **Nachschärfen.** Neue Namen, Projektnamen und Begriffe aus dem Gelesenen
   werden zu neuen Suchbegriffen. Zurück zu Schritt 2.
5. **Aktiv nach Widerspruch suchen.** Bei Entscheidungs- und Bewertungsfragen
   ist das Fehlen von Gegenstimmen ein Befund, der belegt sein will. Erst
   gezielt danach suchen ("aber", "Bedenken", "sehe ich anders", Namen der
   Beteiligten), dann behaupten, es habe keine gegeben.
6. **Abbruch** bei einem der drei Kriterien: zwei Runden ohne neue Fundstelle ·
   die Frage ist belegt beantwortbar · das mit dem Nutzer vereinbarte Lesebudget
   ist erschöpft. Im dritten Fall wird der Abbruch ausgewiesen, nicht kaschiert.

**Suchprotokoll ist Pflicht.** Vor der Antwort steht in drei bis fünf Zeilen,
welche Begriffe gesucht, welche Aufnahmen gelesen und welche verworfen wurden.
Eine Recherche ohne sichtbaren Weg ist für den Nutzer nicht prüfbar — und damit
genauso wertlos wie „nichts gefunden" ohne Abdeckungszeile.

Der Index ist ein **Zeiger auf Fundstellen, kein Ersatz für das Transkript**.
Zitiert wird ausschließlich aus Note oder Transkript.

## Ereignisse: Termine sind keine Aufgaben (v2.3)

Aus einem Satz entstehen oft beide: „Ich muss noch beim TÜV anrufen, der Termin
ist am Dienstag um zehn" enthält eine Aufgabe **und** ein Ereignis. Wer nur
Todos extrahiert, verliert das Datum; wer nur Termine extrahiert, verliert die
Handlung.

| | Aufgabe | Ereignis |
|---|---|---|
| Kennzeichen | etwas ist zu **tun** | etwas **findet statt** |
| Beispiel | „Termin bei Reifen Reber vereinbaren" | „Dienstag 10 Uhr Reifen Reber" |
| Ziel | Todo-Objekt | Termin-Objekt |

**Relative Zeitangaben werden gegen das Aufnahmedatum aufgelöst, nie gegen
heute.** „Nächsten Dienstag", diktiert am 07.08., ist der 12.08. — auch wenn der
Lauf erst am 20.08. stattfindet. Diese Verwechslung ist der häufigste stille
Fehler beim Import aus Sprachmemos; sie erzeugt Termine, die plausibel aussehen
und falsch sind. Aufgelöste Datumsangaben immer mit der Originalformulierung
ausweisen: `12.08. („nächsten Dienstag", aufgenommen 07.08.)`.

**Verbindlichkeit** je Ereignis unterscheiden: `fix` (steht fest) ·
`vorgeschlagen` (im Raum, nicht bestätigt) · `zu vereinbaren` (nur Absicht).
Nur `fix` und `vorgeschlagen` sind Ereignisse; „zu vereinbaren" ist eine
Aufgabe. Fehlt die Uhrzeit, bleibt sie leer — **niemals raten**, lieber ❓.

## Zuordnungs-Regeln

- **Themen-Cluster** aus der externen Wissensbasis verwenden; neue Themen dort ergänzen statt Sammelbecken "Sonstiges" aufzublähen.
- **Personen:** Namen über das Personen-Register auflösen (Milli = Millie = dieselbe Person). Unbekannte Namen → in "Neu gelernt" aufnehmen.
- **ASR-Fehler:** Offensichtliche Transkriptionsfehler über die Korrekturliste auflösen. Neue Verdachtsfälle mit `(?)` markieren, nie stillschweigend "korrigieren" und als Fakt ausgeben.
- **Keine Erfindungen:** Nur zuordnen, was belegbar in Note/Transkript steht. Vermutungen als solche kennzeichnen.

## Belegstellen (v2.3)

Eine Aussage ohne Fundstelle ist eine Behauptung. Die Aufnahme allein genügt
nicht — bei einer 40-Minuten-Aufnahme ist „steht da irgendwo" keine Quelle.

**Belegstelle = Datum · Uhrzeit der Aufnahme · Zeitmarke im Transkript.**
Der Block `transaction` liefert Sprecher und Zeitstempel je Segment; die
Startzeit des zitierten Segments ist die Zeitmarke. Format: `(07.08., 14:03 · 01:12)`.

Liegt nur eine KI-Note ohne Segmentzeiten vor, wird die Zeitmarke weggelassen
statt geschätzt — dann steht dort `(07.08., 14:03 · Note)`.

In Fließtext-Antworten (Workflow 3) stattdessen **nummerierte Quellen**: `[1]`
im Satz, am Ende ein Verzeichnis mit Aufnahme, Belegstelle und `file_id`. Jede
Position wird einzeln belegt — gerade bei Gegenpositionen muss nachvollziehbar
sein, wer wann was gesagt hat.

## Ausgabeformat

```
[Themen-Cluster 1]
- [ ] Aufgabe … (07.08., 14:03 · 01:12)
…
📅 Termine
- Di 12.08., 10:00 · Reifen Reber, HU — fix (07.08., 14:03 · 00:24)
- ❓ „Anfang September" Verlagstermin — Datum unklar (08.08., 09:11 · 02:02)

[Nicht sicher zuordenbar]
- "…wörtliches Zitat…" (Datum) – Deutung unklar

Abdeckung: X/Y Aufnahmen ausgewertet (A: n Notes, B: n Transkripte, C: n). Ungelesen: …
Profile: Standard · Meeting (3 Aufnahmen) · Ideen (1)
Ausgeklammert: n Aufnahmen ohne Lesefreigabe

🧠 Neu gelernt: [nur wenn vorhanden]
- Person/Projekt/ASR-Korrektur …
```

Der Abschnitt „📅 Termine" entfällt, wenn es keine gibt. Die Zeilen „Profile"
und „Ausgeklammert" ebenso, solange nur das Standardprofil greift und nichts
gesperrt ist.

Kompakt bleiben; auf Mobilgeräten zählt Scanbarkeit. Bei Nachfragen zu Einzelthemen nicht die ganze Übersicht wiederholen.

## Abgleich mit dem Zielsystem

Vor jeder Ausgabe von Wiedervorlagen wird der tatsächliche Zustand geprüft.

1. Offene und erledigte Einträge aus dem Zielsystem holen.
2. Über `quelle_id` zuordnen, ersatzweise über `quelle_datum` und Inhalt.
3. Erledigte Punkte aus der Wiedervorlage entfernen und in der Wissensbasis
   als abgeschlossen markieren — mit Erledigungsdatum, weil daraus die
   realistische Bearbeitungsdauer je Cluster ablesbar wird.
4. Nur noch tatsächlich offene Punkte altern lassen.
5. Bereits übergebene Todos **nicht erneut anlegen**.

Ist das Zielsystem nicht erreichbar, wird die Wiedervorlage trotzdem ausgegeben,
aber mit dem Hinweis „Stand aus Wissensbasis, nicht abgeglichen".

## Status-Gedächtnis & Delta-Abfragen (v1.1, nach Secretary-Muster)

Die Wissensbasis führt ein **Auswertungs-Log**: letzter Lauf, abgedeckter Zeitraum, Warteliste nicht transkribierter Aufnahmen (IDs + Datum).
- Vor jedem Lauf: Log prüfen. Bei Folge-Abfragen nur das **Delta** seit dem letzten Lauf neu lesen ("Was ist neu seit gestern?") statt alles erneut.
- Warteliste bei jedem Lauf erneut anpingen (`get_transcript`) – sobald der Nutzer in der Plaud-App transkribiert hat, nachziehen und einsortieren.
- **Wiedervorlage mit Alterung (v2.1):** Die Wissensbasis führt wartende Punkte (externe Zuarbeit, ungeklärte Fragen) mit "offen seit"-Datum. Bei jedem Lauf das Alter ausweisen ("⛔ Jens – offen seit 3 Tagen"); ab etwa einer Woche aktiv Nachfassen vorschlagen.
- **Nachkontrolle (v2.1):** Bei jedem Lauf mit dem Vorlauf vergleichen: Was ist weiterhin offen (Überhang), was erledigt, was übertragen? Dauerläufer kennzeichnen statt sie kommentarlos erneut zu listen.
- Nach jedem Lauf: Log und Wiedervorlage fortschreiben.

## Inhaltsindex (v2.3)

`list_files` kennt nur Namen, und Namen sagen nichts. Eine Frage über die ganze
Bibliothek würde damit jedes Mal jede Aufnahme erneut lesen — unbezahlbar,
sobald die Bibliothek wächst. Der Inhaltsindex in der Wissensbasis (§ 10) löst
das: **Was einmal gelesen wurde, bleibt auffindbar.**

Jede gelesene Aufnahme bekommt eine Zeile: `file_id` · Datum/Uhrzeit · Dauer ·
Klasse · **ein Satz Inhalt** · Schlagworte (auch ASR-Varianten) · Cluster ·
gelesen am.

- Nach jedem Lauf für alle neu gelesenen Aufnahmen fortschreiben.
- Workflow 3 startet im Index, nicht in der Bibliothek.
- Nicht indizierte Aufnahmen im Zeitraum bleiben **ungelesen** im Sinne der
  eisernen Regel Nr. 2 — der Index ersetzt das Lesen nicht, er erspart nur das
  **Wiederholen**.
- Zitiert wird nie aus dem Index, immer aus Note oder Transkript.

## Lesefreigaben (v2.3)

Nicht alles, was aufgenommen wurde, darf in eine Zusammenfassung oder ins
Zielsystem. Die Wissensbasis führt unter § 11 gesperrte Cluster, Personen und
Aufnahmen.

- Gesperrtes wird **gelesen** (sonst wäre die Abdeckung falsch), aber nicht
  zitiert, nicht zusammengefasst und nie nach Todoist übergeben.
- In der Ausgabe erscheint nur die Zahl: „Ausgeklammert: 2 Aufnahmen ohne
  Lesefreigabe" — Schweigen wäre eine Lücke ohne Ausweis.
- Aufhebbar nur durch ausdrückliche Ansage für diesen Lauf, nie durch
  Schlussfolgerung des Skills.
- Im Zweifel gilt eine Aufnahme als frei; Sperren entstehen durch Eintrag, nicht
  durch Vermutung.

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

## Vorschau vor Übergabe

Vor jedem Schreibvorgang ins Zielsystem gilt eine zweistufige Abfolge.

**Stufe 1 — Vorschau.** Zieltabelle ausgeben: Was entsteht wo, mit welcher
Priorität, welchem Fälligkeitsdatum. Nichts schreiben. Bei mehr als zehn
Einträgen zusätzlich die Summe pro Zielprojekt nennen.

**Stufe 2 — Ausführung.** Erst nach ausdrücklicher Bestätigung. Danach das
Ergebnis gegen die Vorschau abgleichen und **jede Abweichung benennen** —
Teilerfolge einer Batch-Operation sind der gefährlichste Fall, weil die
erfolgreichen Einträge den Fehlschlag optisch überdecken.

Ausnahme: Bei einem einzelnen Todo entfällt die Vorschau.

## Übergabe

**Ein Zielsystem: Todoist.** Keine Verzweigung nach privat, dienstlich oder
eigen. Todos werden in einem neutralen Zwischenformat erzeugt und vollständig
dorthin übergeben.

### Todo-Objekt

| Feld | Pflicht | Inhalt |
|---|---|---|
| `inhalt` | ja | Als Ergebnis formuliert, nicht als Thema |
| `quelle_datum` | ja | `YYYY-MM-DD HH:MM` der Aufnahme |
| `quelle_id` | ja | `file_id` |
| `quelle_ts` | nein | Zeitmarke im Transkript (`mm:ss`), wenn vorhanden |
| `cluster` | ja | Themen-Cluster aus der Wissensbasis |
| `prioritaet` | ja | `rot` · `gelb` · `weiss` |
| `delegation` | ja | `A` selbst · `B` delegierbar · `C` automatisierbar |
| `faellig` | nein | Nur bei `rot` oder externem Termindruck |
| `frist` | nein | Harte Deadline, falls genannt |
| `unsicher` | nein | ASR-Unsicherheiten, mit `(?)` im Text markiert |

### Termin-Objekt (v2.3)

| Feld | Pflicht | Inhalt |
|---|---|---|
| `titel` | ja | Worum es geht, nicht was zu tun ist |
| `start` | ja | `YYYY-MM-DD`, Uhrzeit nur wenn genannt |
| `original` | ja | Die Originalformulierung, falls relativ („nächsten Dienstag") |
| `dauer` | nein | Nur wenn genannt |
| `ort` | nein | Nur wenn genannt |
| `beteiligte` | nein | Über das Personen-Register aufgelöst |
| `verbindlichkeit` | ja | `fix` · `vorgeschlagen` |
| `quelle_datum` · `quelle_id` · `quelle_ts` | ja | Aufnahme, `file_id`, Zeitmarke |

Solange kein Kalender angebunden ist, gehen Termine als Todoist-Aufgabe mit
Fälligkeitsdatum und dem Präfix `[Termin]` in dasselbe Zielprojekt wie der
Cluster. `vorgeschlagen` wird im Titel kenntlich gemacht — ein unbestätigter
Termin, der als fixer im Kalender landet, ist schlimmer als gar keiner.

Termine ohne auflösbares Datum werden **nicht übergeben**, sondern mit ❓ in der
Ausgabe zur Klärung gestellt.

### Idempotenz

`quelle_datum` und `quelle_id` gehören **immer** in das Beschreibungsfeld. Das
Paar ist der Schlüssel, an dem der nächste Lauf erkennt, ob ein Todo bereits
übergeben wurde. Ohne diesen Schlüssel legt jeder Folgelauf alles doppelt an.

### Zuordnung

Der `cluster` bestimmt das Zielprojekt, die Abbildung steht in der Wissensbasis
unter „Zielsysteme". Ist ein Cluster dort nicht hinterlegt, geht das Todo in die
Inbox und wird in der Ausgabe als nicht zugeordnet ausgewiesen — niemals raten.

Neue Cluster werden als Section im passenden Projekt angelegt, nicht als neues
Projekt. Die Projektliste bleibt damit stabil und überschaubar.

### Grenzen des Zielsystems

Vor dem ersten Schreiben prüfen, welche Felder unterstützt werden. Nicht
unterstützte Felder werden **umgeleitet, nicht verworfen** — eine nicht
setzbare harte Frist gehört in den Titel. Jede Umleitung wird in der
Wissensbasis unter „Zielsysteme" vermerkt, damit der nächste Lauf sie nicht
erneut entdecken muss.

Übergebene Todos im Auswertungs-Log als "übertragen" führen, damit sie bei der
nächsten Abfrage nicht erneut als offen erscheinen.

## Grenzen (Limitations)

- Nicht transkribierte Aufnahmen sind unsichtbar; der Skill kann Transkription nicht auslösen, nur die Warteliste führen und den Nutzer erinnern.
- ASR-Qualität bei Kurz-Memos ist begrenzt; unsichere Deutungen bleiben mit `(?)` markiert und werden nie als Fakt behandelt.
- Ohne bereitgestellte externe Wissensbasis startet ein Lauf mit dem leeren Template – Zuordnungen sind dann schwächer; Nutzer darauf hinweisen.
- Zeitmarken benennen die Fundstelle, starten aber keine Wiedergabe: der Skill hat keine Oberfläche und keinen Player.
- Die Recherche ist keine semantische Suche über Vektoren, sondern Begriffssuche über Index und Transkripte. Ohne Inhaltsindex bleibt sie auf den gelesenen Zeitraum beschränkt.

## Externe Wissensbasis: Ablage & Auflösung (v2.0)

Der Skill enthält **nur** `references/wissensbasis-template.md` (leere Struktur). Die Live-Daten (Personen, Projekte, ASR-Korrekturen, Auswertungs-Log) liegen ausschließlich beim Nutzer.

**Auflösungsreihenfolge beim Laden (erste Fundstelle gewinnt):**
1. **Arbeitsordner** (Cowork / Claude Code): `<Arbeitsordner>/plaud/wissensbasis.md`, ersatzweise jede Datei `*wissensbasis*.md` im Arbeitsordner → lesen UND nach dem Lauf **direkt dort zurückschreiben**.
2. **Claude-App/Web:** vom Nutzer angehängte Datei in `/mnt/user-data/uploads/` (Name enthält "wissensbasis") oder Projektwissen → lesen; Rückschreiben an den Ursprungsort ist hier nicht möglich → am Laufende die **aktualisierte `wissensbasis.md` als Datei ausgeben** mit dem Hinweis, die alte im Arbeitsordner/Projekt zu ersetzen.
3. **Nichts gefunden:** Aus dem Template eine neue Wissensbasis im Arbeitsordner anlegen (bzw. als Datei ausgeben), Nutzer kurz informieren, dass eine frische Basis gestartet wurde.

**Schreibregeln:** Nur ergänzen und aktualisieren, niemals ohne Auftrag kürzen oder löschen. Pro Lauf: Auswertungs-Log fortschreiben, Änderungslog-Eintrag mit Datum, Stand-Datum im Kopf aktualisieren. Unsichere Einträge mit `(?)` übernehmen.

## Lernpflicht ("der Skill lernt alles")

Nach JEDER Abfrage prüfen: neue Personen? Neue Projekte/Begriffe? Neue ASR-Fehlmuster? Neue Themen-Cluster? Neue Routinen des Nutzers? Erledigte/übertragene Todos? **Neu gelesene Aufnahmen für den Inhaltsindex?** Wünsche, die als Auswertungsprofil dauerhaft gelten sollen („bei Meetings interessieren mich nur Entscheidungen")?

Alles davon in die **externe Wissensbasis** schreiben (Auflösung siehe oben). Im Chat zusätzlich den Block "🧠 Neu gelernt" zeigen, damit der Nutzer die Änderungen sieht. Niemals stillschweigend Wissen verwerfen: Was einmal gelernt wurde (z. B. "Reifen Reber macht evtl. TÜV"), muss im nächsten Lauf wieder verfügbar sein.
