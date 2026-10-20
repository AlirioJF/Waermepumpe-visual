[README.md](https://github.com/user-attachments/files/33703845/README.md)
# Wärmepumpen Visual

**Interaktive Plattform zum Vergleich von Heizsystemen für Ein- und Mehrfamilienhäuser, mit Energieflüssen, 3D-Thermografie und VR-Rundgang (Meta Quest).**

© 2026 Alirio Jiménez Franco. Alle Rechte vorbehalten (siehe `LICENSE`). Entstanden in der Freizeit des Autors.

Aktuelle Version: **1.4** (Oktober 2026). Ältere Versionen bleiben unter `/v0.8/` … `/v1.3/` abrufbar.

---

## Deutsch

### Zweck
Wärmepumpen Visual vergleicht bis zu vier Szenarien technisch und wirtschaftlich. Mögliche Erzeuger:
- Gas-Brennwert;
- Luft/Wasser-WP (Monoblock, Split oder Unitower);
- Sole/Wasser-WP;
- Hybrid;
- Luft/Luft-WP;
- Fernwärme.

Je Szenario lassen sich PV, Solarthermie, Speicher und Kühlen ergänzen.

Die Ergebnisse werden auf verschiedene Weise sichtbar:
- als Energieflüsse;
- in Diagrammen: Jahresdauerlinie, COP-Kennlinie, Energiesignatur;
- in einer parametrischen 3D-Thermografie des Gebäudes mit allen Hydraulik- und Elektrokomponenten;
- in einem geführten Rundgang mit dem Roboter „Kelvin“ am Bildschirm oder in VR.

Einsatzbereich: Variantenbewertung in den HOAI-Leistungsphasen 1–2, Lehre und Wissenstransfer. Die Plattform **ersetzt nicht** die Heizlastberechnung nach DIN EN 12831 und nicht die JAZ-Berechnung nach VDI 4650-1.

### Funktionen (Auswahl)
- **Rechenkern** (`src/engine/engine.js`):
  - Stundensimulation über 8760 h mit Gebäudemodell (U·A + Lüftung) und Heizkurve;
  - Carnot-Gütegrad-Modell für den COP mit Abtauung, Bivalenz und Kostenoptimierung beim Hybrid;
  - PV, Solarthermie nach EN ISO 9806 und Fernwärme;
  - Kühlen: Taupunkt nach Magnus, natural cooling, reversible WP;
  - Wirtschaftlichkeit: Annuitäten, LCOH, CO₂;
  - Batteriespeicher-Auslegung als Rasterdurchlauf über die Kapazität.
- **3D-Ansicht** (`src/view3d/view3d.js`, three.js r128):
  - parametrisches Gebäude mit den Ansichten Real + Thermo, Natürlich (Materialaussehen ohne Temperaturfarben), Wärmebild und Röntgen (Bauteile gläsern, Strömung im Inneren sichtbar);
  - MAG im Schnitt (Membran, N₂-Polster, Wasservorlage nach n(θ)), Monoblock-Innenleben, 4-Wege-Ventil mit Kreisumkehr im aktiven Kühlbetrieb;
  - Fluidpartikel: Richtung, Tempo ∝ v = V̇/A, Temperatur, Fluidart;
  - Rohrkennzeichnung nach DIN 2403 und automatische Prüfung offener Rohrenden;
  - Zählerschrank nach VDE-AR-N 4100 mit Thermografie (I²·R);
  - äußerer Blitzschutz LPS III;
  - Wetterstation über First bzw. Attika;
  - Tagessimulation mit Sonnenstand;
  - Editor „Gebäude anpassen“: Dachneigung, Ziegelfarbe, Aufbauten, PV-Vollbelegung mit Randabstand, Ausrichtung, Balkon, Autofarbe.
- **Raumklima** (`src/room/room.js`): vereinfachte Zweischichten-CFD einer Etage.
- **VR und Kelvin** (`src/xr/`):
  - WebXR für Meta Quest mit Teleport, Gehen samt Treppenfolge, Messen und Türen/Fenster;
  - Variablenmenü, verschiebbare Panels (Grifftaste), Info-Schilder an den Bauteilen und Kopf-HUD;
  - neuronale Sprachausgabe (Piper/ONNX im Web Worker);
  - Formelsatz mit Legende, Kernpunkt je Station und Themenkatalog.
- **Excel-Export** mit Formeln (`src/export/xlsx.js`).

### Aufbau des Quellcodes
```
src/
  index.template.html   HTML-Gerüst mit Platzhaltern
  build.js              Build: setzt alles zu EINER HTML-Datei zusammen
  ui/styles.css         Gestaltung
  ui/app.js             Bedienoberfläche (keine Physik; ruft nur WPV.engine)
  engine/engine.js      Rechenkern (reine Funktionen, Tests: WPV.engine.runTests())
  i18n/i18n.js          Texte DE/EN/ES
  view3d/view3d.js      3D-Thermografie
  room/room.js          Raumklima
  export/xlsx.js        Excel-Vorlage und Export
  xr/xr_main.js         VR-Rundgang (Rig, Controller, Panels, HUD)
  xr/xr_text.js         Stationen von Kelvin (Texte, Formeln, Legenden, Kernpunkte)
  xr/xr_fx.js           Formelsatz, Diagramme, Sprachausgabe (TTS)
dist/index.html         erzeugte Datei (veröffentlichen)
```

### Bauen
```
node src/build.js dist/index.html
```
Dafür wird nur Node.js ≥ 16 benötigt; es gibt keine npm-Abhängigkeiten. Die Bibliotheken (three.js, ECharts, JSZip, ONNX Runtime, Piper) lädt die fertige Seite zur Laufzeit von CDNs.

### Veröffentlichen (GitHub Pages)
- Den Inhalt des Site-Pakets ins Repository laden:
  - `index.html` ist die aktuelle Version;
  - `versionen/` ist die Übersichtsseite;
  - `vX.Y/` enthält die Archivversionen.
- Für eine neue Version eine Ordnerkopie `vX.Y/index.html` anlegen, `index.html` ersetzen und in `versionen/index.html` einen Eintrag ergänzen.

### Prüfskripte (Entwicklung)
Unter `tests/` liegen Playwright-Skripte. Sie erwarten die gebaute Datei als `../v14.html` und ein lokales Chromium.
- `flow13.js`: Fließrichtungen und Anschlüsse aller Rohre in allen Konfigurationen (Topologie).
- `fe13.js`: freie Rohrenden je Heizflächenart.
- `mv13.js`: Rohre, deren Partikel stehen, obwohl die Anlage läuft.
- `keys13.js`: Beschriftungen und Messobjekte ohne Erklärtext oder Namen.
- `dbg13c.js` / `dbg13d.js`: Heizkörper vor Türen und bodentiefen Fenstern bzw. unter Waschtisch und Dusche.
- `x13.js`: VR-Emulation (IWER) mit Maquette 1:15.

### Meta Quest
- Die Plattform im Quest-Browser über die **HTTPS-Adresse** öffnen und auf „VR“ tippen.
- Beim ersten Sprechen lädt die neuronale Stimme einmalig etwa 60–90 MB; danach liegt sie im Browser-Cache.
- Steuerung:
  - Trigger rechts: teleportieren, Türen öffnen, Info-Schilder;
  - Trigger links: messen;
  - Stick links: gehen;
  - Stick rechts: drehen;
  - Grifftaste: Panel greifen;
  - Y: Menü;
  - X: Darstellung;
  - A/B: Stationen.

### Daten und Annahmen
- Ohne Import werden synthetische Daten verwendet und überall so gekennzeichnet:
  - Klima: Referenzjahr je Standort;
  - Strahlung: Klarhimmelmodell mit Bewölkung;
  - Haushaltslast: Profil ähnlich BDEW H0, abhängig von der Personenzahl; im MFH Ø 2 Personen je Wohnung.
- Alle Parameter stehen mit Quelle, Kategorie (Norm, Annahme, Schätzung) und Datum im Reiter „Parameter & Quellen“.
- Echte Fotos oder Schnittbilder von Bauteilen lassen sich über `WPV.FIG_IMAGES[key] = { src, author, license, page }` einbinden. Bitte nur frei lizenzierte Bilder verwenden oder solche, für die eine Erlaubnis vorliegt.

### Grenzen
- Vereinfachte Modelle: stationäre Betriebspunkte, Stundenauflösung, keine dynamische Gebäudesimulation.
- Die Raumklima-CFD ist eine Zweischichten-Näherung.
- Die 3D-Geometrie ist typologisch und bildet kein Aufmaß ab.

---

## English (short)
Interactive platform comparing heating systems for single- and multi-family houses. It shows energy flows, 3D thermography and a VR tour with the robot guide Kelvin on Meta Quest.

- Build: `node src/build.js dist/index.html`.
- Deploy: upload the site package to GitHub Pages.
- Rights: © 2026 Alirio Jiménez Franco, all rights reserved.

## Español (breve)
Plataforma interactiva para comparar sistemas de calefacción en viviendas unifamiliares y plurifamiliares. Muestra flujos de energía, termografía 3D y un recorrido en RV con el robot Kelvin en las Meta Quest.

- Compilar: `node src/build.js dist/index.html`.
- Publicar: subir el paquete del sitio a GitHub Pages.
- Derechos: © 2026 Alirio Jiménez Franco, todos los derechos reservados.
