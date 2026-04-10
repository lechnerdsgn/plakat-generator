# SafeNow Poster Generator — Projektdokumentation

## Was ist das?

Eine Web-Plattform, auf der SafeNow-Kunden Plakate erstellen. Der Kunde gibt Logo, Zonenname und QR-Code-Link ein, das Tool generiert daraus fertige Plakate im SafeNow-Design. Export als PDF oder JPG.

## Tech-Stack

- **Eine einzige Datei: `index.html`** (ca. 1200 Zeilen) — kein Framework, kein Build-Tool
- **Vanilla HTML + CSS + JavaScript**
- Externe Libraries via CDN:
  - `html2canvas` — HTML zu Canvas für JPG-Export
  - `jsPDF` — PDF-Generierung
  - `qrcodejs` — QR-Code Generierung aus Links
- **Font: Inter** (Google Fonts)

## Lokaler Server

```bash
cd "/Users/simonlechner/Desktop/SafeNow Poster Generator"
python3 -m http.server 8090
# Dann http://localhost:8090 im Browser öffnen
```

Ein lokaler Server ist nötig, weil die Assets (PNG, SVG) sonst wegen CORS nicht laden.

## Dateistruktur

```
SafeNow Poster Generator/
├── index.html          # Die komplette App (HTML + CSS + JS)
├── phone-hand.png      # Foto: Hand mit iPhone (transparenter Hintergrund)
├── safenow-logo.svg    # SafeNow Logo (Icon + "SafeNow" Text)
├── helpers-info.svg    # Original Zone-Bar SVG aus Figma (Referenz, wird nicht direkt verwendet)
├── robots.txt          # Blockiert Suchmaschinen-Indexierung
├── CLAUDE.md           # Diese Datei
└── README.md           # Kurzanleitung
```

## Aufbau von index.html

Die Datei hat 3 Hauptbereiche:

### 1. `<style>` (Zeile ~10–580)
Alles CSS für das Tool-UI (Input-Panel, Buttons, Slider, Preview-Bereich).

### 2. `<body>` HTML (Zeile ~580–710)
- **App-Header**: SafeNow Logo + Titel
- **Input-Panel** (linke Seite, 380px breit):
  - Logo-Upload (Drag & Drop)
  - Zonenname (Text)
  - Headline 2. Zeile (Text) + Breite-Slider
  - QR-Code Link + Generier-Button
  - Format-Presets (A1–A6)
  - Hoch-/Querformat Toggle
  - Custom Size (px)
  - Phone-Bild Slider (Drehung, Größe, Position X/Y)
  - Download-Buttons (PDF, JPG)
- **Preview-Area** (rechte Seite): Skalierte Live-Vorschau
- **Render-Target** (hidden): Unsichtbarer Container für Export-Rendering

### 3. `<script>` (Zeile ~710–1230)
Gesamte Logik:

#### State-Objekt (~Zeile 720)
```javascript
let state = {
  logoDataUrl: null,      // Base64 des hochgeladenen Logos
  zoneName: '',           // Name der Zone
  qrDataUrl: null,        // Base64 des generierten QR-Codes
  headlineText: '',       // 2. Zeile der Headline
  headlineWidth: 60,      // Breite der Headline in %
  widthMM: 297,           // Plakat-Breite in mm
  heightMM: 420,          // Plakat-Höhe in mm
  orientation: 'portrait',
  formatName: 'A3',
  phoneRotation: 20,      // Drehung des Phone-Bilds
  phoneScale: 154,        // Größe des Phone-Bilds in %
  phonePosX: 4,           // X-Position in %
  phonePosY: 24,          // Y-Position in %
  barPosX: 36,            // Zone-Bar X-Position auf Phone
  barPosY: -1,            // Zone-Bar Y-Position auf Phone
  barLandPosX: 36,        // Zone-Bar X im Querformat
  barLandPosY: -1,        // Zone-Bar Y im Querformat
  barLandZoom: 62,        // Zone-Bar Größe im Querformat
};
```

#### Wichtige Funktionen

| Funktion | Was sie tut |
|---|---|
| `safenowIconSVG` | Inline SVG des SafeNow Icons (rund, blauer Gradient) |
| `safenowFullLogoSVG` | Inline SVG des vollen SafeNow Logos (Icon + Text) |
| `getZoneBarSVG(logoUrl, zoneName)` | Generiert die Zone-Bar SVG (weißer Balken mit Logo, Zonenname, "SafeNow Zone") |
| `renderPoster(container, scale)` | Rendert das komplette Plakat in einen Container. `scale` für Export (3x) |
| `updatePreview()` | Rendert Vorschau und skaliert sie passend in den Preview-Bereich |
| `exportPDF()` | Rendert in 3x Auflösung, konvertiert mit html2canvas → jsPDF |
| `exportJPG()` | Rendert in 3x Auflösung, konvertiert mit html2canvas → DataURL Download |
| `generateQR()` | Generiert QR-Code aus dem Link-Input mit qrcodejs |
| `setOrientation(orient)` | Wechselt Hoch-/Querformat mit passenden Phone-Defaults |
| `setupUpload(zoneId, inputId, stateKey)` | Initialisiert Drag & Drop + Click Upload für Bilder |

#### Proportionale Skalierung

Alle Größen auf dem Plakat basieren auf `ref = Math.sqrt(actualW * actualH)`:
- **Schriftgrößen**: `fs(pct)` = `ref * pct / 100` (mindestens 10px)
- **Letter-Spacing**: `ls(pct)` = -3.6% für Headlines (>18px), -2% für Body
- **QR-Code**: `ref * 0.1`
- **Phone-Bild**: `ref * 0.55` Basisbreite, skaliert über Slider
- **Paddings**: `pad(pct)` = `actualW * pct / 100` (bleibt breiten-relativ)

#### Phone-Defaults pro Orientierung

```javascript
const phoneDefaults = {
  portrait:  { phoneRotation: 20, phoneScale: 154, phonePosX: 4, phonePosY: 24 },
  landscape: { phoneRotation: 24, phoneScale: 149, phonePosX: -19, phonePosY: 19 },
};
```

## SafeNow Brand (aus Flutter UI Kit)

### Farben
| Verwendung | Hex |
|---|---|
| Primary | `#0869D8` |
| Primary Deep (Hover) | `#0135E4` |
| Text 90% (Headlines) | `#1A1A1A` |
| Text 60% (Subtext) | `#666666` |
| Text 40% (Labels) | `#999999` |
| Background | `#FFFFFF` |

### Typografie
- **Font**: Inter (Google Fonts)
- **Weights**: Medium (500), Semi Bold (600), Bold (700)
- **Letter-Spacing**: -3.6% der Font-Size für Headlines, -2% für Body/Labels
- Alle Styles kommen aus `/Users/simonlechner/Desktop/XCode Test/safenow_ui_kit/lib/src/tokens/typography.dart`

## Plakat-Layout

### Hochformat
```
┌──────────────────────────────┐
│        [SafeNow Logo]        │
│                              │
│    Eine App für Deine        │
│    Sicherheit [headline]     │
│                              │
│    [Subtext]                 │
│                              │
│         ┌─────────┐          │
│         │  Phone  │          │
│         │  +Hand  │   [QR]   │
│         │  +Zone  │          │
│         │   Bar   │ safenow  │
│         │         │  .app    │
│ Initiative:       │          │
│ [Logo] SafeNow    │          │
└──────────────────────────────┘
```

### Querformat
```
┌─────────────────────────────────────────────┐
│ [SafeNow Logo]                              │
│                                             │
│ Eine App für Deine        ┌────────┐  [QR]  │
│ Sicherheit [headline]     │ Phone  │        │
│                           │ +Hand  │safenow │
│ [Subtext]                 │ +Zone  │ .app   │
│                           │  Bar   │        │
│ Initiative:               │        │        │
│ [Logo] SafeNow            └────────┘        │
└─────────────────────────────────────────────┘
```

## Häufige Anpassungen

### Neue Farbe ändern
Suche nach dem Hex-Wert (z.B. `#0869D8`) und ersetze ihn.

### Schriftgröße auf dem Plakat ändern
Die `fs()` Funktion steuert alles. Der Parameter ist ein Prozent-Wert relativ zur Poster-Referenzgröße:
- Headline: `fs(5)` (Portrait) / `fs(6)` (Landscape)
- Subtext: `fs(2.2)` (Portrait) / `fs(2)` (Landscape)
- "Eine Initiative von": `fs(1)` / `fs(1.3)`

### Statischen Text ändern
Suche nach dem deutschen Text (z.B. "Eine App für Deine Sicherheit") in der `renderPoster` Funktion.

### Neues Input-Feld hinzufügen
1. HTML-Input im `.input-panel` Bereich hinzufügen
2. State-Property in `state` Objekt hinzufügen
3. Event-Listener für `input` Event hinzufügen → `state.xyz = e.target.value; updatePreview();`
4. In `renderPoster()` verwenden

### Phone-Bild austauschen
Ersetze `phone-hand.png` mit einem neuen Bild. Transparenter Hintergrund (PNG) empfohlen.

### Export-Qualität ändern
In `exportPDF()` und `exportJPG()`: `EXPORT_SCALE` (aktuell 3) steuert die Auflösung. Höher = bessere Qualität, aber langsamer.

## Figma-Referenz
- Original Poster-Design: `https://www.figma.com/design/FujA7eucUJkYm2zyFaJ67x/UKF-Pilot?node-id=22075-27145`
- Zone-Bar Design: `https://www.figma.com/design/IaBHsiqLqLO9UugsRFEbkb/Göckelesmaier-Activation-Assets?node-id=32042-35938`

## Git
- Repo: `https://github.com/lechnerdsgn/plakat-generator`
- Branch: `main`
