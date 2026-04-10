# SafeNow Poster Generator

Web-Tool zur Erstellung von SafeNow-Plakaten mit individuellem Logo, Zonenname und QR-Code.

## Schnellstart

```bash
# Im Projektordner:
python3 -m http.server 8090

# Dann im Browser:
# http://localhost:8090
```

## Features

- Logo-Upload (Drag & Drop)
- Zonenname + individuelle Headline
- QR-Code wird automatisch aus Link generiert
- Formate: DIN A1–A6, Hoch-/Querformat, Custom (px)
- Phone-Bild frei positionierbar (Drehung, Zoom, X/Y)
- Export als PDF oder JPG (hochaufloesend)
- Alle Groessen skalieren proportional zur Plakatflaeche

## Technologie

Einzelne `index.html` — kein Framework, kein Build, keine Installation noetig.
Externe Libraries (html2canvas, jsPDF, qrcodejs) werden per CDN geladen.

## Dokumentation

Siehe `CLAUDE.md` fuer die komplette technische Dokumentation.
