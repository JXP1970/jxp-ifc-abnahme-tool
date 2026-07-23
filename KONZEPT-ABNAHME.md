# Konzept: Neuorganisation der Abnahme (Dalux-Workflow)

## Ausgangslage

Heute erzeugt der Abnahme-Tab automatisch eine Liste **aller** Gruppen des
Modells (`abGrps`, Zeile 1215) und der Prüfer arbeitet sie ab. Der Prüfumfang
ist damit vom Gruppierfeld diktiert, nicht vom Prüfer.

Gewünscht ist das Gegenteil: **im Büro am Modell die Prüfpunkte festlegen**,
daraus ein Protokoll erzeugen und dieses als PDF bzw. Screenshot in eine
Dalux-Checkliste hängen.

---

## 1. Neues Kernobjekt: der Prüfpunkt

Statt „Position = Abnahmezeile" wird der **Prüfpunkt** das zentrale Objekt.
Er entsteht durch Markieren im Viewer und trägt:

| Feld | Inhalt |
|---|---|
| `nr` | fortlaufend P1, P2, P3 … (nummeriert das Protokoll und die Bildmarker) |
| `titel` | frei, z. B. „Achse C, untere Lage" |
| `guids` | die GUIDs der markierten Stäbe — **stabiler Bezug**, unabhängig vom Gruppierfeld |
| `view` | Kamerazustand (`target`, `radius`, `theta`, `phi`) → Ansicht reproduzierbar |
| `bild` | gerendertes PNG der Ansicht |
| `attrGewaehlt` | welche Attribute in diesem Prüfpunkt im Protokoll erscheinen |
| `kriterien` | Prüfkriterien mit Status i.O. / n.i.O. / n.z. |
| `soll` | aus dem IFC abgeleitete Werte (Anzahl, Ø, Länge, Masse) |
| `ist`, `kommentar`, `foto` | vor Ort erfasst |

Der GUID-Bezug behebt nebenbei Befund F aus der [Analyse](ANALYSE.md): heute
zeigen erfasste Einträge ins Leere, sobald man das Gruppierfeld wechselt.

---

## 2. Ablauf in der Oberfläche

**Schritt 1 — Prüfumfang festlegen (Viewer, Büro)**

Im Markierungs-Panel kommt neben „📋 Stichprobe" ein Knopf
**„+ Als Prüfpunkt"**. Er nimmt die aktuell markierten Stäbe, friert die
Kameraansicht ein, rendert das Bild und legt den Prüfpunkt an. Die Markierung
wird geleert, damit direkt der nächste Prüfpunkt gesetzt werden kann.

Das bestehende Werkzeug bleibt dabei vollständig nutzbar: Rechteck-Selektion,
Gruppenfilter, Isolieren, Messwerkzeug. Für die Bildqualität wird beim Rendern
automatisch isoliert und der Rest halbtransparent gestellt (`isolateMode` +
`ghostMode` sind vorhanden).

**Schritt 2 — Prüfumfang ordnen (Abnahme-Tab, neu)**

Der Abnahme-Tab zeigt nicht mehr alle Modellgruppen, sondern die angelegten
Prüfpunkte:

- links die Prüfpunktliste mit Miniaturbild, umsortierbar, umbenennbar
- Mitte der gewählte Prüfpunkt: Bild, Soll-Werte, Attributtabelle, Kriterien
- rechts der Protokollkopf: Bauvorhaben, Bauteil, Achse, Prüfer, Firma, Datum

Ein Klick auf einen Prüfpunkt stellt im Viewer die gespeicherte Ansicht wieder
her — man kann jederzeit zurück ans Modell.

**Schritt 3 — Attribute wählen**

Welche IFC-Attribute im Protokoll erscheinen, wird über die vorhandene
Checkbox-Liste gewählt (`buildAttrList`, Zeile 925 — bereits für Info-Panel
und Summen im Einsatz). Zwei Ebenen:

- **global**: gilt als Vorgabe für alle Prüfpunkte
- **pro Prüfpunkt**: überschreibt die Vorgabe, wenn an einer Stelle etwas
  Besonderes zu dokumentieren ist

Zusätzlich pro Attribut die Wahl *Einzelwert* oder *Summe über die Auswahl*
(Masse und Stabanzahl will man summiert, den Durchmesser nicht).

---

## 3. Prüfkriterien mit Ankreuzkästchen

Vorschlag für die Standardvorlage einer Bewehrungsabnahme — vollständig
editierbar, Reihenfolge änderbar, eigene Vorlagen speicherbar:

| # | Kriterium |
|---|---|
| 1 | Stabdurchmesser |
| 2 | Stabanzahl / Verlegeabstand |
| 3 | Betondeckung, Abstandhalter |
| 4 | Übergreifungsstöße (Länge und Versatz) |
| 5 | Verankerungslängen |
| 6 | Lage und Höhenlage der Bewehrung |
| 7 | Bügel / Verbügelung |
| 8 | Sauberkeit, Rost, Verunreinigung |
| 9 | Aussparungen und Einbauteile |
| 10 | Schalung geschlossen, Maße |

Dreiwertiger Status je Kriterium: **i.O. ☑ / n.i.O. ☒ / n.z. ☐** — in
Deutschland üblich, weil „nicht zutreffend" sonst als offener Punkt gelesen wird.

Zwei Ausgabevarianten, beide sinnvoll:

- **Leer** — Kästchen unausgefüllt, wird gedruckt und vor Ort mit dem Stift
  abgehakt. Das Blatt wird dann fotografiert und in Dalux gehängt.
- **Ausgefüllt** — in der App angekreuzt, Kästchen im PDF bereits gesetzt.

---

## 4. Ausgaben

### 4a — Screenshot je Prüfpunkt

Das Modellbild wird beim Anlegen erzeugt und mit Beschriftung versehen:
Prüfpunktnummer, Titel, betroffene Positionen, Legende, Datum. Die Beschriftung
kommt über ein 2D-Canvas, das über das WebGL-Bild gelegt wird — dadurch bleibt
das Bild ohne Dalux-Kontext lesbar.

*Technischer Hinweis:* Der Renderer wird heute ohne `preserveDrawingBuffer`
angelegt (Zeile 724) und läuft in einer Endlosschleife. `toDataURL()` aus einem
Klick-Handler liefert deshalb ein **leeres Bild**. Lösung ohne Performanceverlust:
unmittelbar vor dem Auslesen einmal synchron `renderer.render()` aufrufen.

### 4b — PDF-Protokoll

Aufbau:

1. Kopf: Bauvorhaben, Bauteil, Achse, Prüfer, Firma, Datum, IFC-Dateiname
2. Übersichtstabelle aller Prüfpunkte mit Status
3. je Prüfpunkt eine Seite: Modellbild, Attributtabelle, Kriterien-Kästchen,
   Soll/Ist, Kommentar, Baustellenfoto
4. Fuß: Unterschriftenfelder Prüfer und Bauleitung

### 4c — Dalux-Paket

Da eine Dalux-Checkliste je Punkt einen Anhang aufnimmt, ist ein ZIP am
praktischsten:

```
Abnahme_2026-07-23.zip
├── Pruefprotokoll.pdf
├── P01_Achse-C-untere-Lage.png
├── P02_Stuetze-S12.png
└── ...
```

Die Bilddateien sind so benannt, dass sie sich in Dalux direkt dem passenden
Checklistenpunkt zuordnen lassen. JSZip ist bereits eingebunden.

---

## 5. Was mit dem Bestehenden passiert

- **Stichprobe** (Zeile 1457) bleibt erhalten und wird zu einem Prüfpunkt-Typ:
  ein Prüfpunkt kann statt Kriterienliste eine Soll/Ist-Zählung tragen.
- **`abGrps`/`getSoll`** werden nicht mehr als Listengenerator gebraucht,
  sondern nur noch zur Soll-Ermittlung innerhalb eines Prüfpunkts. Dabei sollte
  die Doppelzählung aus Befund E der Analyse geprüft werden.
- **CSV-Export** bleibt, bezieht sich künftig auf Prüfpunkte statt Gruppen.
- **Editor und Viewer** bleiben unberührt.

---

## 6. Getroffene Entscheidungen (23.07.2026)

1. **Ankreuzen:** beides. Umschalter „Kästchen im Export" – *leer* zum Abhaken
   vor Ort, *ausgefüllt* mit dem Status aus der App.
2. **PDF-Zuschnitt:** je Prüfpunkt ein eigenes PDF, passend zu je einem
   Dalux-Checklistenpunkt.
3. **PDF-Technik:** echte Datei per jsPDF, lokal unter `vendor/` abgelegt
   (kein CDN, damit es offline bleibt).
4. **Kriterien:** keine feste Vorlage. Die Vorlage in der rechten Spalte startet
   leer, wird frei befüllt und im Browser gespeichert.

## 7. Umgesetzt in V2.0

Alles aus den Abschnitten 1–4 ist gebaut. Beim Testen aufgefallen und
mitkorrigiert:

- **Soll-Zählung.** Die alte `getSoll` summierte `BarCount` über alle Objekte.
  Im Test ergaben 6 Stäbe mit `BarCount` 6 ein „Soll" von 36 – der in der
  Analyse als Befund E vermutete Fehler, hier belegt. Soll ist jetzt die Anzahl
  der ausgewählten Stab-Objekte; `BarCount` steht unverändert daneben.
- **PDF-Größe.** Ohne `compress:true` legt jsPDF das Modellbild als rohes RGB ab:
  3,7 MB je Prüfpunkt. Mit der Option ~145 kB.
- **Zeichenkodierung.** Die jsPDF-Standardschriften können nur Latin-1. Der
  Gedankenstrich wurde verschluckt („A–F / 1–12" → „AF / 112"), „Σ" zerstörte
  die Kodierung der Folgezeichen. Jetzt zentral gefiltert.
- **Bildbeschriftung.** Sie wird erst beim Export erzeugt, nicht beim Anlegen –
  sonst stünde im Bild nie die Bezeichnung, die man erst danach vergibt.
- **Bildausschnitt.** Beim Anlegen wird auf die Auswahl eingepasst;
  „⟳ Neu aufnehmen" übernimmt dagegen die aktuelle Ansicht unverändert.

## 8. Weiterhin offen

- Die Prüfpunkte liegen weiterhin nur im Arbeitsspeicher (Befund A der
  [Analyse](ANALYSE.md)). Gespeichert werden bisher nur Protokollkopf und
  Kriterien-Vorlage. Persistenz der Prüfpunkte selbst ist der nächste Schritt.
- Three.js und JSZip kommen noch vom CDN; nur jsPDF liegt lokal. Für echten
  Offlinebetrieb müssen die beiden ebenfalls nach `vendor/`.
