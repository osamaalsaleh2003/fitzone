# SPEC.md · FitZone

Quelle: individuelles Interview mit Lisa Sommer (K2), Meeting 1 mit zwölf Nachrichten und Meeting 2 mit acht Nachrichten, 30.10.2026. N1–N20 bezeichnen die Antworten auf die fortlaufend nummerierten Nachrichten.

**KI-Unterstützung:** ChatGPT wurde zur Formulierung der Interviewfragen sowie zur Strukturierung, Ausarbeitung und Prüfung dieser SPEC.md eingesetzt.

## Entitäten

IDs, Statusbezeichnungen und die Zusammenfassung ähnlicher Vorgänge sind eigene Modellierungsentscheidungen. Nicht ausdrücklich bestätigte Annahmen sind gekennzeichnet. Die drei Tarife werden als feste Werte mit den Regeln aus BR1 modelliert, nicht als eigene Entität. Auch später umzusetzende Angebote sind berücksichtigt.

| Entität | Attribute |
|---|---|
| Mitglied | mitglied_id, name, email, telefon, geburtstag (optional), beitrittsdatum, tarif (Basic / Plus / Premium), status (aktiv / pausiert), tarifhistorie |
| Kurstyp | kurstyp_id, bezeichnung, level, interner_kurswert |
| Termin | termin_id, art (Gruppenkurs / Personal Training), beginn, dauer_minuten, format (Studio / Online), teilnehmerlimit, status (geplant / stattgefunden / abgesagt), zugangslink |
| Trainer | trainer_id, name, bietet_personal_training_an |
| Raum | raum_id, bezeichnung, kapazitaet |
| Buchung | buchung_id, status, eingetragen_am, angebot_am, bestaetigungsfrist, fest_gebucht_am, aktiv_bestaetigt_am, storniert_am, automatisch_nachgerueckt, kontingent_wirksam, gebuehrenbetrag, terminnotiz |
| Mitgliedschaftspause | pause_id, startdatum, geplantes_enddatum, grund |
| Video | video_id, titel, video_verweis, dauer_minuten |

**Belege und Ableitungen:** Mitgliedsdaten und Historie aus N18; Kurstyp/Raum/Trainer/Termin aus N6; Kurswert aus N16; PT und Notizen aus N17; Pausen aus N12/N18. Video als aufgezeichneter Kurs aus N11: Titel, Verweis und Dauer sind vorgeschlagene beschreibende Attribute, keine bereits erhobenen Kundenvorgaben. Das PT-Angebot ist für Marie belegt; bei anderen Trainern noch offen.

Die Warteliste ist ein Buchungsstatus: „wartend“ → „angeboten“ → „fest gebucht“. Kurzfristiges automatisches Nachrücken führt direkt zu „fest gebucht“. Weitere Status sind „storniert“, „teilgenommen“, „nicht erschienen“, „Angebot abgelaufen“ und „wegen Kontingent entfernt“. Feste Buchung und aktive Bestätigung werden getrennt, weil die Gebühren davon abhängen. Die Kontingentwirkung bleibt auch bei später Stornierung oder No-Show erhalten. (N8–N9, N15–N16, N19–N20)

## Beziehungen

| Beziehung | Kardinalität und Bedeutung |
|---|---|
| Kurstyp – Termin | 1:n. Ein Kurstyp hat mehrere Gruppenkurstermine. Ein Gruppenkurstermin gehört zu einem Kurstyp; Personal Training benötigt keinen Kurstyp. (N6/N17; gemeinsame Termin-Entität als Modellentscheidung) |
| Trainer – Termin | 1:n. Ein Trainer hat mehrere Termine; pro Termin wird ein Trainer angenommen. Teamteaching wurde nicht geklärt. (N6/N17) |
| Raum – Termin | 1:n. Ein Raum kann über die Zeit mehrere Termine haben. Studio-Termine haben einen Raum, Online-Termine keinen physischen Raum. (N6–N7/N17) |
| Trainer – Kurstyp | n:m. Trainer können mehrere Kursarten unterrichten. Mehrere qualifizierte Trainer je Kurstyp sind eine Modellannahme; diese Richtung wurde nicht ausdrücklich bestätigt. (N6/N12) |
| Mitglied – Buchung | 1:n. Jede Buchung gehört zu genau einem Mitglied; ein Mitglied kann mehrere Buchungen haben. (N2/N19) |
| Termin – Buchung | 1:n. Jede Buchung betrifft einen Termin. Gruppenkurse erlauben mehrere feste Teilnehmer; PT höchstens einen. (N7/N17) |
| Mitglied – Mitgliedschaftspause | 1:n. Jede Pause gehört zu einem Mitglied; mehrere historische Pausen sind möglich. (N18, aus der Historie abgeleitet) |
| Trainer – Video | n:m als Modellannahme: Die Aufzeichnungen stammen von Trainern; mehrere Aufzeichnungen je Trainer und gemeinsame Aufzeichnungen sind im Entwurf möglich. Genaue Zuordnung noch offen. (N11) |

Mitglied und Termin stehen über **Buchung in einer n:m-Beziehung**: Ein Mitglied besucht mehrere Termine, ein Gruppentermin hat mehrere Mitglieder. Diese Zusammenfassung zählt nicht zusätzlich zu den acht oben aufgeführten Beziehungen. Entitäten können zunächst ohne zugehörige Vorgänge bestehen, z. B. ein Mitglied ohne Buchung.

## Business Rules

Die erhobenen Bedingungen sind in fünf fachliche Regeln mit Unterbedingungen und Ausnahmen geordnet.

### BR1 – Tarif und aktive Mitgliedschaft bestimmen die Nutzung

- Basic darf nur Studio-Gruppenkurse buchen: maximal fünf kontingentwirksame Buchungen pro Kalendermonat. Keine Live-Streams oder Videos, keine Tarifausnahmen. Plus und Premium erhalten Studio, Live-Streams und Videos ohne Kursbuchungslimit. (N14)
- Es zählt der Monat des Kurstermins, nicht der Buchungstag. Wartelistenplätze belasten das Kontingent noch nicht; feste Buchungen schon. Stornierungen mindestens zwei Stunden vorher geben den Slot zurück; spätere Absagen und No-Shows verbrauchen ihn. (N19–N20)
- Während einer Pause darf das Mitglied nicht buchen und es erfolgt keine Abbuchung. Maximal drei Monate Pause pro Jahr, mit Start- und geplantem Enddatum. Termine während der Pause werden automatisch gebührenfrei storniert; die Warteliste rückt nach. Automatische Reaktivierung mit Benachrichtigung; Lisa kann das Enddatum ändern. (N12/N18)

### BR2 – Nachrücken erfolgt innerhalb von Kapazität und Berechtigung

- Studio- und Live-Stream-Termine benötigen Buchungen. Den Stream-Link gibt es nach erfolgreicher Buchung. Das Teilnehmerlimit gilt je Termin; bei voller Belegung ist eine Warteliste mit maximal fünf Personen möglich, nach Eintragungsreihenfolge ohne Tarifvorrang. (N7–N8)
- Mindestens zwei Stunden vor Beginn: reserviertes Angebot mit 30 Minuten Bestätigungszeit. Ohne Bestätigung erhält die nächste Person das Angebot. Weniger als zwei Stunden vorher: automatisch fest buchen und informieren. Genau zwei Stunden gehören zur Bestätigungsvariante. (N15/N20)
- Bei ausgeschöpftem Basic-Kontingent wird der Wartelisteneintrag entfernt, das Mitglied informiert und die nächste berechtigte Person berücksichtigt. Späte Absagen und No-Shows bleiben für das Kontingent relevant. (N19–N20)
- Studio: Yoga/HIIT höchstens 15, Spinning 12 entsprechend den Fahrrädern; zusätzlich Raumkapazität beachten. Online nennt Lisa ungefähr 20 oder 25 je nach Kurstyp. Ein pflegbares Teilnehmerlimit pro Termin ist eine Modellableitung, kein bestätigtes universelles Online-Limit. (N6–N7)

### BR3 – Späte Absagen und No-Shows lösen eine Gebühr aus, mit Ausnahmen

- Bei mindestens zwei Stunden Vorlauf ist die Absage kostenlos. Bei weniger Vorlauf oder No-Show grundsätzlich 50 % des internen Kurswerts. Dieser wird je Kurstyp von Lisa konfiguriert. Yoga 8 €, HIIT 8 €, Spinning 10 € sind vorgeschlagene Beispielwerte, keine endgültige Preisliste. (N5/N16/N20)
- Premium ist gebührenfrei. Bei automatischem Nachrücken ohne aktive Bestätigung gibt es für Nichterscheinen weder Strike noch Geldgebühr. Nach aktiver Bestätigung gelten die normalen Regeln einschließlich Premium-Ausnahme. Pausenbedingte Stornierungen sind ebenfalls gebührenfrei. (N4–N5/N15–N16/N18)
- Eine gleiche Gebührenregel für Personal Training wurde nicht bestätigt. Die Gebührenausnahme ist auch nicht automatisch eine Ausnahme vom Basic-Kontingent.

### BR4 – Trainer müssen qualifiziert und verfügbar sein

- Gruppenkurse dürfen nur passenden und zeitlich verfügbaren Trainern zugeordnet werden. Marie: Yoga/Pilates; Tom: Spinning/HIIT. Eine unzulässige Zuweisung wie Tom zu Yoga wird verhindert. (N6/N12)
- Bei PT werden verfügbare Trainer angeboten; ihre anderen Termine sind zu berücksichtigen. (N17)

### BR5 – Premium-Coaching umfasst einen nicht übertragbaren monatlichen Slot

- Premium enthält einen 1:1-Coaching-Slot pro Monat; ungenutzte Slots verfallen. Der Slot wird bei Bestätigung verbraucht, nicht erst bei Durchführung. (N4/N17)
- Buchung mit Mitglied, Trainer, Datum/Uhrzeit, Dauer, Ort und Trainingsnotiz. Standard 60 Minuten, manchmal 90; Studio-Raum oder online. (N13/N17)

## Widersprüche

- **W1 – Basic-Online-Zugang:** Online zunächst erst ab Plus (N3), später angeblich Live-Streams im Basic-Kontingent (N10). **In N14 ausdrücklich korrigiert:** Basic ausschließlich Studio; kein Live-Stream-/Video-Zugang. Diese Klarstellung gilt.
- **W2 – Sofort gebucht oder Bestätigung:** N8 nennt sowohl automatisches „du bist drin“ als auch Bestätigung. **N15/N20 legen die Fallunterscheidung fest:** ab einschließlich zwei Stunden Vorlauf Bestätigung binnen 30 Minuten; darunter automatisch fest buchen. Die No-Show-Gebührenausnahme für automatische Buchungen ist in N16 bestätigt.

Zusätzliche Klarstellung: Bisher offener Online-Gruppenlink ist der Ist-Zustand; kontrollierte Buchung das gewünschte Soll (N2/N6–N7). Plus ist ausdrücklich unbegrenzt (N10/N14).

## Änderung aus Meeting 2

In N13 bittet Lisa auf die Zukunftsfrage darum, Personal Training von Anfang an einzuplanen. In N17 konkretisiert sie den Buchungsablauf, Slot-Verbrauch bei Bestätigung, Dauer, Ort und Notizen. Personal Training war bereits in N4/N11 erwähnt; die Änderung ist seine konkrete Integration in das Modell. Die Umsetzung bleibt nach ihrer Klarstellung in N17 zweite Priorität.

## Prioritäten

1. **v1:** Gruppen-Kursbuchung für Studio und Online, Warteliste/Nachrücken, Stornierungen, Mitgliederverwaltung mit Tarif/Kontingent/Pausen und Trainerzuordnung. (N12)
2. **Zweite Priorität:** Personal-Training-Buchung mit Premium-Slot; bereits im Modell vorgesehen. (N17)
3. **Später:** On-Demand-Videothek mit den bestätigten Tarifrechten. (N11–N12/N14)
4. **Nicht aktuell vorgesehen:** zweite Location, bislang nur vage Idee. (N11)

## Offene Fragen und Annahmen

- Fortgeltung der 30-Minuten-Frist, wenn ein laufendes Angebot die Zwei-Stunden-Grenze überschreitet; zählen angebotene Plätze zu den maximal fünf Wartenden?
- Kontingent kann zwischen Angebot und Bestätigung ausgeschöpft werden. Eine erneute Prüfung folgt aus dem Fünferlimit; Behandlung des dann unbestätigten Angebots ist offen.
- Gebührenfreiheit einer aktiv vorgenommenen späten Absage nach automatischem Nachrücken: ausdrücklich bestätigt ist nur die No-Show-Ausnahme.
- Pausen: Kalender- oder Vertragsjahr, angebrochene Monate, Kontingentrückgabe bei kurzfristiger Pause und Behandlung von PT-Slots.
- Coaching: Monatsbezug bei Vorausbuchungen, Slot-Rückgabe bei Absage sowie Preise/Zugang für zusätzliche kostenpflichtige Termine.
- Gruppenkursdauern, genaue Trainer-Verfügbarkeitsdaten und Teamteaching. Ein Trainer pro Termin ist die vorläufige Modellannahme.
- Videoattribute und Trainerzuordnung sind Modellvorschläge. Die n:m-Beziehung Mitglied–Termin ist unabhängig davon belegt.
- Tarifhistorie muss nachvollziehbar sein; konkrete Historienfelder und Speicherung noch festzulegen. Berechtigungen sind nur teilweise erhoben: Lisa pflegt Kurswerte/Pausen, Trainer benötigen PT-Notizen.
- Historische Gebühren bei Preisänderungen und Zahlungsabwicklung sind nicht spezifiziert. Die App soll zunächst keine unbelegte Zahlungsintegration voraussetzen.
