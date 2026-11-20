# Video Delay

Browser-App, die das Kamerabild zeitversetzt anzeigt (0–30 s), z. B. fürs Volleyballtraining.
Läuft auf iOS und Android, Hoch- und Querformat, ohne Build-Schritt.

## Nutzung

1. Seite über HTTPS öffnen (GitHub Pages).
2. **Start** tippen und Kamerazugriff erlauben.
3. Verzögerung per Slider oder ±1 einstellen. Tippen aufs Bild blendet die Bedienung aus.

4. **Clip** friert die letzten 5–30 s ein: Zeitleiste, Bildschritte, 0,25×–1×, Schleife und Zeichnen
   (Freihand, Linie, Kreis; je Zeichnung nur dieses Bild oder ganzer Clip). Nichts wird gespeichert.

iPhone: Für Vollbild in Safari **Teilen → Zum Home-Bildschirm** und von dort starten.

## Anpassen

Konstanten oben im Script von `index.html`: `CAP_FPS`, `MAX_SIDE`, `JPEG_Q`, `MAX_BYTES` (Obergrenze des Bildpuffers, begrenzt die Cliplänge auf schwachen Geräten).
