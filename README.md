# TRUST GOLD Rechner

Eine einzelne, selbstständige HTML-Seite (`index.html`, kein Build-Prozess, kein Backend) zur monatlichen Durchrechnung eines rabattierten physischen Goldkauf-Plans mit drei Rabattmodellen.

Live-Demo (nach Aktivierung von GitHub Pages, siehe unten):
`https://<dein-github-benutzername>.github.io/trust-gold-rechner/`

## Verwendung

Einfach `index.html` im Browser öffnen – es werden keine externen Bibliotheken oder Server benötigt, alles (HTML, CSS, JavaScript) steckt in der einen Datei. Jede Eingabeänderung berechnet den gesamten Plan sofort neu.

Alle Eingaben (Plan-Parameter, Strategie, alle Barrenpreise) werden automatisch im `localStorage` des Browsers gespeichert und beim nächsten Aufruf der Seite wiederhergestellt – auch nach Schließen des Tabs oder Browsers. Das ist rein clientseitig (kein Server, keine Cookies im eigentlichen Sinn) und pro Browser/Gerät getrennt. Über den Link „Auf Standardwerte zurücksetzen“ am Ende von Abschnitt 1 wird der gespeicherte Stand gelöscht und die Seite neu geladen.

## GitHub Pages aktivieren

Damit die Seite unter einer öffentlichen URL erreichbar ist:

1. Im Repository auf **Settings → Pages** gehen.
2. Unter **Build and deployment** → **Source** die Option **Deploy from a branch** wählen.
3. Als **Branch** `main` und als Ordner `/ (root)` auswählen, dann **Save**.
4. Nach ein bis zwei Minuten ist die Seite unter `https://<dein-github-benutzername>.github.io/trust-gold-rechner/` erreichbar (GitHub zeigt den Link auch direkt auf der Pages-Einstellungsseite an).

Jeder weitere Push auf `main` aktualisiert die Live-Seite automatisch.

## Was der Rechner tut

Der Rechner simuliert Monat für Monat, wie ein Sparplan in physisches Gold (in Standardbarrengrößen) investiert wird, wobei je nach gewählter **Strategie** unterschiedliche **Rabattmodelle** zum Einsatz kommen. Ziel ist es, am Ende der Planungsdauer zu zeigen, wie viel Gold angesammelt wurde und wie viel Barguthaben übrig bleibt.

### Eingaben (Abschnitt 1)

| Feld | Bedeutung |
|---|---|
| Startkapital | Einmalzahlung im ersten Monat (Monat 0) |
| Monatliches Nachschießen | Zusätzlicher Betrag, der ab Monat 1 jeden Monat eingezahlt wird |
| Planungsdauer | Anzahl der simulierten Monate, 1–1200 (100 Jahre). Die Obergrenze ist rein eine technische Sicherheitsbremse gegen Tippfehler (z. B. eine versehentliche zusätzliche Null), keine inhaltliche Einschränkung – für jedes reale Planungsszenario ist das quasi unbegrenzt |
| Goldpreissteigerung p.a. | Jährliche Preisänderung, wird **wie Zinseszins verzinst** (`multiplier = (1 + growth)^(Monat/12)`, bei `1 + growth <= 0` auf 0 begrenzt). Beispiel bei −10 %: aus 120 € werden nach 12 Monaten 108 €, nach 24 Monaten 97,20 € – jedes Jahr −10 % vom *aktuellen*, nicht vom ursprünglichen Preis. Auch **negative Werte** (z. B. `-20` für ein Szenario mit −20 % p.a.) sind erlaubt; der Preis nähert sich dabei asymptotisch der Null an, statt sie (wie bei einer linearen Rechnung) bei einer bestimmten Laufzeit exakt zu erreichen |
| Startmonat | Kalendermonat von Monat 0 |
| Strategie | Siehe unten |

### Barrenpreise (Abschnitt 2)

Für jede der zehn Standardgrößen (1 g, 2 g, 5 g, 10 g, 20 g, 50 g, 100 g, 250 g, 500 g, 1 kg) gibt es zwei editierbare Preise:

- **Verkaufspreis** (`buy` im Code) – der Preis, zu dem der Anbieter an den Kunden verkauft (Listenpreis vor Rabatt).
- **Ankaufspreis** (`sell` im Code) – der Preis, zu dem der Anbieter bei Fälligkeit zurückkauft.

Die Standardwerte sind die tatsächlichen Live-Preise von Trust Gold (Stand: siehe Commit-Datum dieser Änderung) und enthalten bereits den echten Spread zwischen Verkaufs- und Ankaufspreis (rund 11,8 % je Barrengröße). Vorherige Versionen hatten hier versehentlich identische Verkaufs-/Ankaufspreise als Platzhalter, was Modell 2/3 spürbar zu optimistisch rechnete – siehe Punkt 7 unten.

### Die drei Rabattmodelle

| Modell | Code-Kürzel | Laufzeit | Mechanik |
|---|---|---|---|
| Sofortrabatt | `m1` | 2 Monate | Beim Kauf werden **8 % Rabatt auf den Verkaufspreis** gewährt – mit dem verfügbaren Geld können also sofort mehr Barren gekauft werden. Bei Fälligkeit erfolgt der Rückkauf zum (aktuellen, hochgerechneten) **Verkaufspreis**, nicht zum Ankaufspreis. |
| 1 Jahr | `m2` | 12 Monate | Kein Sofortrabatt beim Kauf. Stattdessen erhält der Kunde **3 % des investierten Betrags pro Monat** als Rabattgutschrift, insgesamt 12 Monate lang. Bei Fälligkeit Rückkauf zum **Ankaufspreis**. |
| 2 Jahre | `m3` | 24 Monate | Wie `m2`, aber **4 % pro Monat**, 24 Monate lang, Rückkauf ebenfalls zum Ankaufspreis. |

Die monatliche Rabattgutschrift fließt sofort wieder in den verfügbaren Betrag des jeweiligen Monats ein und kann direkt für neue Barrenkäufe verwendet werden.

### Strategien (Kaskaden vs. Einzelmodell)

Die Strategie bestimmt, welches Modell in jedem Monat für einen **neuen** Vertrag verwendet wird. Die Entscheidung hängt von den **verbleibenden Monaten** (`monthsRemaining = duration − aktueller Monat`) ab, weil ein Vertrag nur begonnen wird, wenn er innerhalb der Planungsdauer auch fällig werden kann:

- **Kaskade 2 J. → 1 J. → Sofort (`c321`)**: solange ≥ 24 Monate übrig sind → Modell 3; danach solange ≥ 12 Monate übrig sind → Modell 2; danach solange ≥ 2 Monate übrig sind → Modell 1; sonst kein neuer Vertrag.
- **Kaskade 2 J. → Sofort (`c31`)**: ≥ 24 Monate → Modell 3; sonst ≥ 2 Monate → Modell 1.
- **Kaskade 1 J. → Sofort (`c21`)**: ≥ 12 Monate → Modell 2; sonst ≥ 2 Monate → Modell 1.
- **Nur Modell 1/2/3 (`m1`/`m2`/`m3`)**: es wird ausschließlich das gewählte Modell verwendet (und nur, solange die Restlaufzeit reicht).

Reicht die Restlaufzeit für kein Modell mehr, wird in diesem Monat **kein neuer Vertrag** abgeschlossen; das verfügbare Geld bleibt als Guthaben auf dem Verrechnungskonto stehen.

### Der monatliche Ablauf (Kernschleife im Code)

Für jeden Monat `m` von `0` bis `duration`:

1. **Rabattgutschrift ermitteln**: Summe aus `Betrag × Rate` über alle bereits laufenden `m2`/`m3`-Verträge (jeder Vertrag zahlt vom Monat nach Abschluss bis einschließlich Fälligkeitsmonat).
2. **Preis-Multiplikator**: `1 + growth × (m/12)` für die lineare Goldpreissteigerung.
3. **Rückkaufserlös**: Für alle Verträge, die genau in diesem Monat fällig werden, wird die eingelagerte Barrenmenge zum hochgerechneten Rückkaufpreis verkauft (Modell 1 → Verkaufspreis, Modell 2/3 → Ankaufspreis).
4. **Verfügbarer Betrag** = Saldo vom Vormonat + Einzahlung dieses Monats (Startkapital in Monat 0, sonst monatliches Nachschießen) + Rabattgutschrift + Rückkaufserlös.
5. **Modell für einen eventuellen neuen Vertrag** wird anhand von Strategie und Restlaufzeit bestimmt (siehe oben). Passt kein Modell mehr in die verbleibende Zeit, wird kein neuer Vertrag eröffnet.
6. **Barren kaufen**: Ist ein Modell aktiv, wird der komplette verfügbare Betrag (beim Sofortrabatt zum bereits 8 % reduzierten Preis) **gierig von der größten zur kleinsten Barrengröße** verteilt (zuerst so viele 1-kg-Barren wie möglich, danach vom Rest so viele 500-g-Barren wie möglich, usw. bis 1 g) – das minimiert den nicht investierbaren Restbetrag.
7. **Neuer Saldo** = verfügbarer Betrag − tatsächlich ausgegebener Kaufbetrag. Dieser Rest wandert als Guthaben in den nächsten Monat.
8. Wurde ein neuer Vertrag abgeschlossen, wird er der Vertragsliste hinzugefügt (mit Fälligkeitsmonat, Kaufbetrag, Rate, Modell und gekauften Stückzahlen je Größe).
9. Zusätzlich werden zur Anzeige berechnet: das insgesamt in **noch laufenden** Verträgen gebundene Gold (Gramm) und dessen **aktueller Schätzwert** (mit Preissteigerung hochgerechnet).

### Ergebnis-Kennzahlen (Abschnitt 3)

- **Guthaben nach Ablauf**: Bargeld-Saldo am Ende der Planungsdauer. Da nie ein Vertrag eröffnet wird, der nicht mehr innerhalb der Planungsdauer fällig werden kann (siehe Strategien oben), ist am Ende planmäßig kein Gold mehr in offenen Verträgen gebunden – das Guthaben nach Ablauf ist daher zugleich das gesamte Endkapital.
- **Höchster Goldbestand**: Der größte Betrag an physischem Gold (in Gramm), der zu irgendeinem Zeitpunkt der Planung **gleichzeitig** in noch nicht fälligen Verträgen lag, bevor er wieder verkauft wurde. Das ist **keine** Summe aller jemals gekauften Barren – bei kurzlaufenden Modellen (z. B. dem 2-Monats-Sofortrabatt) wird dasselbe Kapital über die Planungsdauer viele Male hintereinander investiert, verkauft und neu investiert; eine reine Summe aller Käufe würde diese Umschichtungen fälschlich als zusätzliches Gold zählen, obwohl real nie mehr als der hier ausgewiesene Spitzenwert gleichzeitig vorhanden war. Direkt darunter wird dieser Spitzenwert zusätzlich je Modell aufgeschlüsselt (Modelle können sich in Kaskaden zeitlich überlappen, siehe unten).
- **Gesamter Rückkaufserlös**: Summe aller Ankaufserlöse aus fällig gewordenen Verträgen über die gesamte Laufzeit (das ist ein reiner Geldfluss über die Zeit, keine Bestandsgröße – hier ist eine Summe sinnvoll).
- **Wiederverwendete Rabattgutschriften**: Summe aller monatlichen Rabattgutschriften aus Modell 2/3 über die gesamte Laufzeit (ebenfalls ein Geldfluss, keine Bestandsgröße).
- **Eigenkapital eingezahlt**: Startkapital + (monatliches Nachschießen × Planungsdauer) – die Summe aller eigenen Einzahlungen.
- **Endkapital / eingesetztes Kapital**: Guthaben nach Ablauf im Verhältnis zum eingezahlten Eigenkapital, in Prozent, zusätzlich als „-fache“ (z. B. „336,56 % ≈ 3,37-fache“) darunter – dieselbe Kennzahl, nur als Vervielfachung statt Prozent, für alle, die das lieber so lesen.

*(Frühere Versionen zeigten zusätzlich „Gold in laufenden Verträgen“, „Geschätzter Goldwert“, „Kapital gesamt nach Ablauf“ und „Gesamtwert / eingesetztes Kapital“ – diese vier Werte waren durch die oben beschriebene Fälligkeits-Regel strukturell immer 0 bzw. exakte Duplikate von „Guthaben nach Ablauf“ und wurden deshalb entfernt. Eine weitere Zwischenversion zeigte „Insgesamt gekauftes Gold“ als Summe aller Käufe über die Laufzeit – das zählte bei kurzlaufenden Modellen jede Wiederanlage desselben Kapitals erneut mit und wurde durch den nicht-kumulativen „Höchster Goldbestand“ ersetzt.)*

### Monatliche Tabelle (Abschnitt 4)

Zeigt für jeden Monat: Datum, aktives Modell, verfügbaren Betrag vor dem Kauf, Nachschuss, Rabattgutschrift, Rückkaufserlös bei Fälligkeit, tatsächlichen Kaufbetrag, Fälligkeitsmonat des neuen Vertrags, Saldo des Verrechnungskontos, Stückzahl je Barrengröße, insgesamt in diesem Monat gekaufte Gramm sowie das zu diesem Zeitpunkt in laufenden Verträgen gebundene Gold samt Schätzwert (diese beiden letzten Spalten sind für die Zwischenmonate sinnvoll, in der letzten Zeile aber immer 0/€0, siehe oben).

Die Tabelle hat immer `Planungsdauer + 1` Zeilen (Monat 0 bis Monat `Planungsdauer`): Der letzte Monat eröffnet keinen neuen Vertrag mehr, sondern verbucht nur noch die zu diesem Zeitpunkt fälligen Rückkäufe/Gutschriften, damit das Endergebnis vollständig abgerechnet ist.

## Bei der Prüfung gefundene und behobene Punkte

Bei einer genauen Durchsicht der Rechenlogik wurden folgende Punkte korrigiert:

1. **Goldpreissteigerung galt nur beim Rückkauf, nicht beim Kauf.** Die eingegebene jährliche Steigerung wurde bisher ausschließlich auf Rückkaufpreise und die Bewertung laufender Verträge angewendet, nicht aber auf den Preis, zu dem in späteren Monaten neue Barren gekauft werden – neue Barren wurden also immer zum Preis von Monat 0 „gekauft“, obwohl sie laut Modell später und zu höheren Preisen verkauft wurden. Das führte bei jeder Steigerung > 0 % zu einem künstlich zu guten Ergebnis. Der Kaufpreis-Multiplikator wird jetzt konsequent auf beide Seiten angewendet. Bei 0 % Steigerung (Standardeinstellung) ändert sich dadurch nichts an den Ergebnissen.
2. **„Kapital gesamt nach Ablauf“, „Gold in laufenden Verträgen“ und „Geschätzter Goldwert“ waren am Ende der Planung strukturell immer 0 bzw. Duplikate.** Weil nie ein Vertrag eröffnet wird, der nicht mehr fristgerecht fällig werden kann, gibt es am letzten Monat nie noch offene Verträge – diese Kennzahlen waren also immer 0 (bzw. „Kapital gesamt“ und „Gesamtwert / eingesetztes Kapital“ exakt identisch mit „Guthaben nach Ablauf“ bzw. „Guthaben / eingesetztes Kapital“). Siehe Screenshot-Beispiel: beide Prozentwerte waren immer exakt gleich. Ersetzt durch „Insgesamt gekauftes Gold“ und „Gesamter Rückkaufserlös“, die tatsächlich aussagekräftig sind.
3. **Verträge mit 0 € Kaufbetrag.** Reichte das verfügbare Geld in einem Monat nicht einmal für den kleinsten 1-g-Barren, wurde trotzdem ein „leerer“ Vertrag mit 0 € Kaufbetrag angelegt und in der Tabelle als aktives Modell angezeigt, obwohl nichts gekauft wurde. Das ist jetzt abgefangen; in einem solchen Monat steht korrekt „Kein neuer Vertrag“.
4. **Erklärender Hinweis zur zusätzlichen Zeile.** Die Tabelle hatte schon immer eine Zeile mehr als die eingegebene Planungsdauer (Monat 0 bis Monat *Planungsdauer*), weil der letzte Monat nur noch Fälligkeiten abrechnet. Das ist rechnerisch beabsichtigt, wurde aber nirgends erklärt – jetzt gibt es dazu einen Hinweistext direkt über der Tabelle.

**Unverändert, aber zur Kenntnisnahme:** Modell 1 (Sofortrabatt) kauft beim Rückkauf weiterhin zum Verkaufspreis (nicht zum Ankaufspreis) – das wurde bewusst so belassen, da dies laut Rückmeldung den tatsächlichen Konditionen entspricht. Die Kauf-Reihenfolge „größte Barrengröße zuerst“ ist ein bewusster, einfacher Greedy-Ansatz; er ist in der Praxis meist optimal (größere Barren sind pro Gramm günstiger), garantiert aber rechnerisch nicht in jedem Fall die maximale Grammzahl für den eingesetzten Betrag.

5. **Planungsdauer auf 1200 Monate (100 Jahre) erweitert** (vorher 120, dann 360). Dabei zeigte sich: Bei sehr langen Laufzeiten verbringen die Kaskaden-Strategien (und erst recht die Einzelstrategie „Nur Modell: Sofortrabatt“) einen Großteil bzw. die gesamte Zeit im Sofortrabatt-Modell (Modell 1), das alle 2 Monate erneut 8 % Rabatt gewährt und zum vollen Verkaufspreis zurückkauft (Punkt „Modell 1 Rückkauf“ oben). Wiederholt sich dieser Zyklus über Jahrzehnte, wirkt er wie Zinseszins und kann rechnerisch außergewöhnlich hohe Endsummen ergeben (Beispielrechnung: 200 €/Monat, reines Sofortrabatt-Modell, über 240 Monate/20 Jahre ergab rechnerisch über 100 Mio. € Endkapital aus 48.000 € Einzahlung). Das ist eine bewusst bestätigte Annahme des Nutzers (der Sofortrabatt ist beliebig oft hintereinander nutzbar, solange der Anbieter besteht – vergleichbar mit dem Totalausfallrisiko bei anderen Finanzprodukten wie Aktien/ETFs) und keine neue Änderung an der Berechnung, sondern eine Konsequenz der bereits bestätigten Modell-1-Regel, die bei kurzen Laufzeiten kaum auffällt und erst bei sehr langen Laufzeiten sichtbar wird. Ein entsprechender Hinweis dazu steht im Info-Kasten der Seite.
6. **Performance-Optimierung für lange Laufzeiten.** Damit auch 1200 Monate flüssig berechnet werden, werden bereits abgelaufene (fällige) Verträge am Ende jedes Simulationsmonats aus der internen Vertragsliste entfernt, statt sie bis zum Planungsende mitzuschleppen. Das ändert nichts am Ergebnis (fällige Verträge werden ohnehin nicht mehr für künftige Monate gebraucht), macht die Berechnung aber deutlich schneller: 1200 Monate rendern in unter 150 ms statt mit wachsender, spürbarer Verzögerung.
7. **Standardpreise hatten keinen Spread (Verkaufspreis = Ankaufspreis).** Bis zu dieser Änderung waren die editierbaren Standardwerte für Verkaufs- und Ankaufspreis identisch – ein Platzhalter-Artefakt aus der ursprünglichen Vorlage, kein bewusster Wert. In der Realität liegt der Ankaufspreis (Rückkaufpreis) spürbar unter dem Verkaufspreis (bei den aktuellen Live-Preisen rund 11,8 % je Barrengröße). Die Standardwerte wurden durch die echten Live-Preise ersetzt (Verkaufs- und Ankaufspreis getrennt, jeweils in Euro). Das betrifft ausschließlich Modell 2/3, die beim Rückkauf den Ankaufspreis verwenden – Modell 1 bleibt unverändert, da es laut bestätigter Regel immer zum Verkaufspreis zurückkauft. Effekt an einer Beispielrechnung (200 €/Monat, 240 Monate, kein Startkapital-Bonus): Modell 2 fiel von ≈ 6,9 Mio. € auf ≈ 2,45 Mio. € Endkapital, Modell 3 von ≈ 38,6 Mio. € auf ≈ 26,9 Mio. € – der Spread ist also ein sehr wesentlicher Faktor, den die alten Standardwerte komplett verschwiegen haben. Modell 2 trifft es relativ stärker als Modell 3, weil sein Vertrag nur 12 statt 24 Monate läuft und der Spread-Verlust dadurch doppelt so oft pro Zeiteinheit anfällt.
8. **Startkapital wurde in der Tabelle fälschlich als „Nachschuss“ in Monat 0 angezeigt.** Die Spalte „Nachschuss“ soll nur den *monatlichen* Betrag zeigen; Monat 0 hat aber keinen Nachschuss, sondern das einmalige Startkapital (das bereits korrekt in „Verfügbar vor Kauf“ eingerechnet ist). Die Tabelle zeigte in Monat 0 trotzdem das Startkapital in der Nachschuss-Spalte an – reiner Anzeigefehler, die zugrunde liegende Berechnung war davon nicht betroffen. Monat 0 zeigt in der Nachschuss-Spalte jetzt korrekt 0,00 €.
9. **Goldpreissteigerung von linear auf Zinseszins (geometrisch) umgestellt.** Bei der linearen Rechnung (`1 + growth × Monat/12`) erreicht ein hinreichend starker/lang anhaltender Preisrückgang bei einer bestimmten Laufzeit **exakt** 0 und bleibt danach dort stehen (z. B. bei −10 % p.a. exakt bei Monat 120 = 10 Jahre). Das ist kein sanftes „Gold wird billiger“, sondern ein harter Sprung auf „Gold ist komplett wertlos und unverkäuflich“ – jeder zu diesem Zeitpunkt gerade laufende Vertrag verliert dadurch schlagartig seinen kompletten Wert, und ab da kann für den Rest der Planung gar nichts mehr gekauft werden (Beispiel 200 €/Monat, 240 Monate, Modell 1, −10 % p.a.: zwei Verträge im Gesamtwert von ca. 12.189 € wurden bei Monat 120/121 auf 0 € zurückgekauft, danach 120 Monate lang nur noch unverzinste Bargeld-Ansammlung). Auf ausdrücklichen Wunsch des Nutzers rechnet der Multiplikator jetzt geometrisch/wie Zinseszins: `multiplier = (1 + growth)^(Monat/12)` (bei `1 + growth <= 0` weiterhin auf 0 begrenzt, das betrifft aber nur noch den entarteten Fall von −100 % p.a. oder mehr). Bei 0 % Steigerung ändert sich nichts (`1^x = 1`); ein negativer Wert nähert sich jetzt asymptotisch der Null an, statt sie exakt zu erreichen, sodass es diesen Klippeneffekt nicht mehr gibt.

## Hinweise zu späteren Anpassungen

Der gesamte Code liegt in `index.html` in einem einzigen `<script>`-Block, zentrale Funktion ist `plan()`. Ansatzpunkte für typische Änderungen:

- **Rabattsätze/Laufzeiten der Modelle** → die Objekte `terms` (Laufzeit in Monaten) und `rates` (monatliche Rabattrate) sowie der Sofortrabatt-Faktor `0.08` in der Zeile mit `effective=buy.map(...)`.
- **Kaskaden-Reihenfolge/Schwellenwerte** → die verschachtelten Bedingungen in der Zeile, die `model` anhand von `strategy` und `monthsRemaining` bestimmt.
- **Kaufreihenfolge der Barrengrößen** (aktuell: größte zuerst) → die Schleife `for(let i=9;i>=0;i--)`.
- **Neue Barrengrößen/Standardpreise** → die Arrays `grams` und `defaults` am Anfang des Scripts (Reihenfolge muss zu den Spaltenüberschriften der Tabelle passen).
- **Anzeige/Kennzahlen** → die Objekte in `summary.innerHTML=[...]` bzw. die Template-Zeile in `rows.innerHTML=out.map(...)`.

Der Rechner dient ausschließlich der unverbindlichen Veranschaulichung und stellt keine Finanz- oder Vertragsberatung dar.
