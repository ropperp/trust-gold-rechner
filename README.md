# TRUST GOLD Rechner

Eine einzelne, selbstständige HTML-Seite (`index.html`, kein Build-Prozess, kein Backend) zur monatlichen Durchrechnung eines rabattierten physischen Goldkauf-Plans mit drei Rabattmodellen.

Live-Demo (nach Aktivierung von GitHub Pages, siehe unten):
`https://<dein-github-benutzername>.github.io/trust-gold-rechner/`

## Verwendung

Einfach `index.html` im Browser öffnen – es werden keine externen Bibliotheken oder Server benötigt, alles (HTML, CSS, JavaScript) steckt in der einen Datei. Jede Eingabeänderung berechnet den gesamten Plan sofort neu.

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
| Planungsdauer | Anzahl der simulierten Monate |
| Goldpreissteigerung p.a. | Jährliche Preissteigerung, wird **linear** auf Monatsbasis umgerechnet (`multiplier = 1 + growth × (Monat/12)`) |
| Startmonat | Kalendermonat von Monat 0 |
| Strategie | Siehe unten |

### Barrenpreise (Abschnitt 2)

Für jede der zehn Standardgrößen (1 g, 2 g, 5 g, 10 g, 20 g, 50 g, 100 g, 250 g, 500 g, 1 kg) gibt es zwei editierbare Preise:

- **Verkaufspreis** (`buy` im Code) – der Preis, zu dem der Anbieter an den Kunden verkauft (Listenpreis vor Rabatt).
- **Ankaufspreis** (`sell` im Code) – der Preis, zu dem der Anbieter bei Fälligkeit zurückkauft.

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

- **Guthaben nach Ablauf**: Bargeld-Saldo am Ende der Planungsdauer.
- **Gold in laufenden Verträgen** / **Geschätzter Goldwert**: Gold, das am Planungsende noch in nicht fälligen Verträgen gebunden ist, in Gramm bzw. Euro.
- **Kapital gesamt nach Ablauf**: Guthaben + geschätzter Goldwert der noch laufenden Verträge.
- **Eigenkapital eingezahlt**: Startkapital + (monatliches Nachschießen × Planungsdauer) – die Summe aller eigenen Einzahlungen.
- **Gesamtwert / eingesetztes Kapital** und **Guthaben / eingesetztes Kapital**: die beiden vorherigen Werte jeweils im Verhältnis zum eingezahlten Eigenkapital, in Prozent.
- **Wiederverwendete Rabattgutschriften**: Summe aller monatlichen Rabattgutschriften aus Modell 2/3 über die gesamte Laufzeit.

### Monatliche Tabelle (Abschnitt 4)

Zeigt für jeden Monat: Datum, aktives Modell, verfügbaren Betrag vor dem Kauf, Nachschuss, Rabattgutschrift, Rückkaufserlös bei Fälligkeit, tatsächlichen Kaufbetrag, Fälligkeitsmonat des neuen Vertrags, Saldo des Verrechnungskontos, Stückzahl je Barrengröße, insgesamt in diesem Monat gekaufte Gramm sowie das zu diesem Zeitpunkt in laufenden Verträgen gebundene Gold samt Schätzwert.

## Hinweise zu späteren Anpassungen

Der gesamte Code liegt in `index.html` in einem einzigen `<script>`-Block, zentrale Funktion ist `plan()`. Ansatzpunkte für typische Änderungen:

- **Rabattsätze/Laufzeiten der Modelle** → die Objekte `terms` (Laufzeit in Monaten) und `rates` (monatliche Rabattrate) sowie der Sofortrabatt-Faktor `0.08` in der Zeile mit `effective=buy.map(...)`.
- **Kaskaden-Reihenfolge/Schwellenwerte** → die verschachtelten Bedingungen in der Zeile, die `model` anhand von `strategy` und `monthsRemaining` bestimmt.
- **Kaufreihenfolge der Barrengrößen** (aktuell: größte zuerst) → die Schleife `for(let i=9;i>=0;i--)`.
- **Neue Barrengrößen/Standardpreise** → die Arrays `grams` und `defaults` am Anfang des Scripts (Reihenfolge muss zu den Spaltenüberschriften der Tabelle passen).
- **Anzeige/Kennzahlen** → die Objekte in `summary.innerHTML=[...]` bzw. die Template-Zeile in `rows.innerHTML=out.map(...)`.

Der Rechner dient ausschließlich der unverbindlichen Veranschaulichung und stellt keine Finanz- oder Vertragsberatung dar.
