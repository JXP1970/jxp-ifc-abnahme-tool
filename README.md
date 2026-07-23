# IFC Bewehrungsabnahme

Webbasiertes Werkzeug für die Bewehrungsabnahme auf Grundlage von IFC-Modellen.
Läuft vollständig im Browser, ohne Server und ohne Installation: IFC-Datei per
Drag & Drop laden, im 3D-Viewer den Prüfumfang festlegen, Prüfprotokoll als PDF
und Screenshot ausgeben.

Die Ausgaben sind für die Weiterverwendung in **Dalux-Checklisten** gedacht:
je Prüfpunkt ein PDF und ein beschriftetes Bild, gebündelt als ZIP.

## Funktionen

**Editor** — Attribute der Bewehrungsobjekte anzeigen, filtern, ändern und
ergänzen; Export als CSV oder als bearbeitete IFC-Datei.

**3D-Viewer** — Modell drehen, zoomen, nach Positionen einfärben und filtern,
Stäbe markieren (Einzelauswahl oder Rechteck), isolieren, halbtransparent
schalten. Messwerkzeug mit Objektfang auf Stabenden und -mitte.

**Abnahme** — Prüfpunkte werden im Viewer am Modell festgelegt: markieren,
„＋ Prüfpunkt". Jeder Prüfpunkt speichert

- die GUIDs der geprüften Stäbe,
- die Kameraansicht (jederzeit wiederherstellbar),
- ein beschriftetes Modellbild,
- die gewählten IFC-Attribute, wahlweise als Einzelwert oder als Summe,
- frei definierbare Prüfkriterien mit Ankreuzkästchen i.O. / n.i.O. / n.z.,
- Soll/Ist-Stückzahl, Kommentar und Baustellenfoto.

**Export** — je Prüfpunkt ein PDF und ein Screenshot, dazu ein ZIP-Paket mit
allen Prüfpunkten und einer CSV-Übersicht. Die Ankreuzkästchen lassen sich leer
ausgeben (zum Abhaken vor Ort) oder mit dem in der App gesetzten Status.

## Bedienung auf dem Tablet

Ein Finger dreht, zwei Finger zoomen und verschieben, Tippen markiert. Die
Schriftgrößen sind für den Außendienst ausgelegt. Info- und Markierungsfenster
lassen sich verschieben und in der Größe ändern.

## Unterstützte Dateien

IFC2x3 und IFC4. Bewehrung wird als `IFCREINFORCINGBAR` oder als
`IFCBUILDINGELEMENTPROXY` gelesen. Die Geometrie muss tesseliert vorliegen
(`IFCTRIANGULATEDFACESET`); Dateien mit `IFCSWEPTDISKSOLID` laden ihre Attribute,
zeigen aber noch keine 3D-Geometrie.

## Aufbau

Eine einzige `index.html` ohne Build-Schritt. `vendor/` enthält jsPDF lokal.
Three.js und JSZip werden derzeit noch von einem CDN geladen, die App braucht
also beim Start eine Internetverbindung.

Weitere Unterlagen: [ANALYSE.md](ANALYSE.md) zum Ausgangsstand,
[KONZEPT-ABNAHME.md](KONZEPT-ABNAHME.md) zum Aufbau der Abnahme.

## Herkunft

Weiterentwicklung von [jxp-ifc-tool](https://github.com/jxp1970/jxp-ifc-tool)
ab dessen Version 1.8. Das ursprüngliche Werkzeug bleibt dort unverändert
bestehen.
