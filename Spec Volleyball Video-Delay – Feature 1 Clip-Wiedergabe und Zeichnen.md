# Spec: Volleyball Video-Delay – Feature 1: Clip-Wiedergabe und Zeichnen

Nov 20, 2026 · @Radi5

## 1. Ziel und Kontext

Die App soll im Volleyballtraining Bewegungen direkt wiederholbar und analysierbar machen: erst mit dem Live-Delay, künftig mit einem frei steuerbaren Clip, den man bildweise durchspulen und mit Linien markieren kann.

**Nutzungsszenario:** Ein iPhone oder Android-Gerät steht auf einem Stativ und filmt das Feld. Der Bildschirm wird per AirPlay auf einen Fernseher gespiegelt. Trainer und Spieler schauen auf den TV, bedient wird am Handy. Deshalb muss alles im Querformat gut funktionieren und große, treffsichere Bedienelemente haben.

**Nutzen:**

- Spieler sehen die eigene Bewegung direkt nach der Aktion (Delay).
- Trainer können eine Aktion der letzten Sekunden bildweise zurückspulen und mit Linien markieren.

## 2. Rahmenbedingungen und Abgrenzung

Die App bleibt eine einzelne Web-App ohne Installation aus dem App Store, kostenlos und ohne Server.

| Thema | Festlegung |
| --- | --- |
| Plattform | Safari auf iOS (Hauptziel), Chrome auf Android; als Web-App zum Home-Bildschirm hinzufügbar |
| Auslieferung | Statisches Hosting über HTTPS (GitHub Pages oder Netlify), da die Kamera nur über HTTPS freigegeben wird |
| Kosten | Keine; nur kostenlose Bibliotheken und Modelle |
| Datenschutz | Alle Videodaten bleiben im Arbeitsspeicher des Geräts, nichts wird gespeichert oder hochgeladen |
| Ausrichtung | Hoch- und Querformat, Bild immer vollständig sichtbar |
| Ton | Nicht erforderlich, nur Bild |

**Nicht-Ziele:** Speichern oder Teilen von Clips, Mehrkamera-Betrieb, Cloud-Auswertung, Langzeit-Statistik über mehrere Trainings, automatische Körpererkennung, Schwerpunkt- und Sprunghöhenanalyse (bewusst später), exakte biomechanische Messung (Kraftplatten-Niveau).

## 3. Ausgangslage: bestehende Live-Delay-App

Die Basis ist eine einzelne `index.html`, die bereits funktioniert und erweitert wird, nicht neu geschrieben.

- **Kamera:** `getUserMedia`, Rückkamera als Standard, Wechsel auf Frontkamera möglich.
- **Ringpuffer:** Jedes Bild wird als JPEG (max. 960 px Breite, Qualität 0,7, bis 30 Bilder pro Sekunde) mit Zeitstempel im Arbeitsspeicher abgelegt. Ältere Bilder als 32 s werden verworfen.
- **Anzeige:** Das Bild von (jetzt minus Delay) wird auf ein Canvas gezeichnet. Delay einstellbar von 0 bis 30 s in 0,5-s-Schritten.
- **Bedienung:** Slider, ±1-s-Tasten, Standbild, Kamerawechsel, Spiegeln; Bedienelemente per Tippen ein- und ausblendbar.
- **Ausrichtung:** Layout reagiert auf Hoch- und Querformat; Bildschirm bleibt per Wake Lock an.

Der vorhandene Ringpuffer ist die Grundlage für das Clip-Feature: Der Clip ist ein eingefrorener Ausschnitt dieses Puffers.

## 4. Feature 1: Clip-Wiedergabe mit Zeitleiste und Zeichnen

Mit einem Tipp wird ein Clip der letzten bis zu 30 Sekunden eingefroren. Dieser Clip lässt sich frei vor- und zurückspulen, anhalten und mit dem Finger beschriften. Der Clip existiert nur im Arbeitsspeicher und wird nie abgelegt.

### 4.1 Ablauf

1. Im Live-Modus tippt der Nutzer auf **Clip**.
2. Die App kopiert die Bilder der letzten N Sekunden (Standard 30, einstellbar 5 bis 30) aus dem Ringpuffer in den Clip und pausiert die Aufnahme.
3. Der Clip startet angehalten am letzten Bild.
4. Mit **Zurück zu Live** wird der Clip verworfen, die Kamera nimmt wieder auf und der Delay-Puffer füllt sich neu.

### 4.2 Funktionale Anforderungen

| Nr. | Anforderung | Priorität |
| --- | --- | --- |
| F1-1 | Clip-Button im Live-Modus; Cliplänge wählbar (5, 10, 20, 30 s) | Muss |
| F1-2 | Zeitleiste über die ganze Breite, per Touch ziehbar und antippbar; zeigt Position als Zeit vor Clipende (z. B. −12,4 s) und Bildnummer | Muss |
| F1-3 | Play/Pause, 1 Bild vor, 1 Bild zurück (größere Sprünge über die Zeitleiste; keine eigenen ±1-s-Tasten) | Muss |
| F1-4 | Wiedergabegeschwindigkeit 0,25×, 0,5×, 1× | Muss |
| F1-5 | Schleife ein/aus (Wiedergabe springt am Ende zum Anfang) | Soll |
| F1-6 | Zeichnen: Freihand, Linie, Kreis; 5 Farben; 2 Strichstärken; Rückgängig; Alles löschen | Muss |
| F1-7 | Zeichenmodus je Zeichnung wählbar: nur auf diesem Bild oder als Hilfslinie über den ganzen Clip | Muss |
| F1-8 | Zeichnungen gehen beim Verwerfen des Clips verloren; kein Speichern, kein Export | Muss |
| F1-9 | Hoch- und Querformat; Zeichnungen bleiben nach dem Drehen an derselben Bildstelle | Muss |
| F1-10 | Spiegeln und Kamerawahl aus dem Live-Modus wirken auch im Clip | Soll |
| F1-11 | Pinch-Zoom auf das Bild zum genaueren Zeichnen | Kann |

### 4.3 Bedienung

- **Zeichnen ist ein eigener Modus** (Button "Zeichnen"). Nur dort zeichnet der Finger; außerhalb davon bedient er Zeitleiste und Buttons. So kollidieren Wischen und Zeichnen nicht.
- **Touch-Ziele** mindestens 44 × 44 pt, weil am Handy bedient und am TV geschaut wird.
- **Bedienleiste** unten wie im Live-Modus, per Tippen aufs Bild ausblendbar. Beim Ausblenden bleiben Bild und Zeichnung sichtbar.
- **Anzeige** der aktuellen Zeit und Bildnummer bleibt auch bei ausgeblendeter Leiste klein am Rand sichtbar.

### 4.4 Technische Vorgaben

- **Zeitbasis:** Die Wiedergabe richtet sich nach den gespeicherten Zeitstempeln, nicht nach der Bildnummer, weil die Bildrate der Kamera schwanken kann. Angezeigt wird das letzte Bild mit Zeitstempel kleiner oder gleich der Abspielzeit.
- **Wiedergabe:** `requestAnimationFrame`-Schleife; Abspielzeit = Startzeit + verstrichene Zeit × Geschwindigkeit.
- **Speicher:** Die Bilder bleiben als JPEG-Blobs im Speicher. Dekodiert (`createImageBitmap`) wird nur das aktuelle Bild plus ein kleiner Vorlauf von etwa 5 Bildern. Zielwert: unter 100 MB für einen 30-s-Clip.
- **Zeichnungen** werden als Vektordaten gespeichert (Punkte in Koordinaten von 0 bis 1 relativ zum Videobild), nicht als Pixel. Dadurch bleiben sie beim Drehen und Skalieren sauber.
- **Ebenen:** Videobild, Zeichnungsebene (Canvas darüber), Bedienleiste (HTML). Eingaben über Pointer Events; `touch-action: none` nur auf der Zeichenebene im Zeichenmodus.

### 4.5 Akzeptanzkriterien

- [ ] Ein 30-s-Clip lässt sich auf einem iPhone ohne Absturz erzeugen, abspielen und mehrfach neu erzeugen.
- [ ] Ziehen an der Zeitleiste zeigt das Bild ohne spürbare Verzögerung (unter etwa 100 ms).
- [ ] Die Bildschritte bewegen genau ein aufgenommenes Bild vor oder zurück.
- [ ] Eine als "ganzer Clip" gezeichnete Linie bleibt an derselben Bildstelle, während durch den Clip gespult wird.
- [ ] Nach dem Drehen des Geräts liegen Zeichnungen weiterhin an der richtigen Stelle des Bildes.
- [ ] Nach "Zurück zu Live" und Seiten-Neuladen sind Clip und Zeichnungen nicht mehr vorhanden.

## 5. Nicht-funktionale Anforderungen

| Bereich | Anforderung |
| --- | --- |
| Bedienbarkeit | Alle Bedienelemente mit mindestens 44 × 44 pt; im Querformat bleibt das Bild so groß wie möglich, die Leiste liegt darüber und lässt sich ausblenden |
| Lesbarkeit am TV | Zeitanzeige, Bildnummer und Zeichnungen in großer, kontrastreicher Schrift; keine Information nur durch Farbe |
| Performance Live | Anzeige und Aufnahme bleiben flüssig (Ziel 30 Bilder pro Sekunde); im Clip-Modus reagiert die Zeitleiste ohne spürbare Verzögerung |
| Speicher | Clip unter etwa 100 MB; die App erkennt zu wenig Speicher und verkürzt den Clip oder senkt die Auflösung, statt abzustürzen |
| Geräte | iPhone mit aktueller iOS-Version in Safari; zusätzlich Android mit Chrome; ein älteres iPhone muss Clip und Zeichnen ohne Absturz unterstützen, gegebenenfalls mit verkürztem Clip |
| Datenschutz | Keine Daten verlassen das Gerät; keine Cookies, kein Tracking, keine Speicherung von Video oder Zeichnungen |
| Offline | Nach dem ersten Laden nutzbar ohne Netz (App-Datei lokal zwischengespeichert) |
| Wartbarkeit | Einzelne HTML-Datei; Code in klar getrennten Blöcken (Aufnahme, Clip, Zeichnen) mit Kommentaren |
| Sprache | Oberfläche auf Deutsch |

## 6. Risiken und Grenzen

Das größte Risiko ist der Speicherverbrauch auf älteren iPhones; alles Weitere ist technisch gut beherrschbar.

| Risiko | Auswirkung | Gegenmaßnahme |
| --- | --- | --- |
| Hoher Speicherverbrauch auf älteren iPhones | Absturz oder Neuladen der Seite | Bildgröße und Clip-Länge begrenzen; Speicher beobachten und Auflösung senken |
| Kamera liefert schwankende Bildrate | Wiedergabe ruckelt, Bildschritte unterschiedlich lang | Wiedergabe nach Zeitstempeln statt nach Bildnummer |
| Dekodieren beim Ziehen an der Zeitleiste zu langsam | Ruckeln beim Spulen | Vorlauf von etwa 5 Bildern, begrenzte Bildgröße |
| Wischen und Zeichnen kollidieren auf dem Touchscreen | Ungewolltes Zeichnen oder Spulen | Eigener Zeichenmodus; Zeichenebene nur dort aktiv |
| Bedienleiste verdeckt im Querformat zu viel Bild | Schlecht lesbar am TV | Leiste per Tippen ausblenden; kompakte Anordnung im Querformat |
| iOS beendet die Seite oder sperrt den Bildschirm | Clip geht verloren | Wake Lock; Hinweis, dass der Clip nicht gespeichert wird |

## 7. Umsetzungsplan und Testplan

Die Umsetzung erfolgt in zwei Schritten. Jeder Schritt ist für sich im Training nutzbar und wird abgenommen, bevor der nächste beginnt.

1. **M1 – Clip und Zeitleiste:** Clip einfrieren, Wiedergabe mit Zeitleiste, Bildschritten und Geschwindigkeit, Zurück zu Live (F1-1 bis F1-5, F1-9, F1-10). Abnahme: Akzeptanzkriterien 4.5, ohne Zeichnen.
2. **M2 – Zeichnen:** Zeichenmodus mit Freihand, Linie, Kreis, Farben, Rückgängig, Bild-/Clip-Ebene (F1-6 bis F1-8, optional F1-11). Abnahme: Zeichen-Kriterien in 4.5.

Nicht Teil dieses Specs und bewusst später: automatische Körpererkennung mit Schwerpunkt und Sprunghöhe.

### Testplan

| Test | Vorgehen | Ziel |
| --- | --- | --- |
| Gerätetest | App auf dem Trainings-iPhone und einem älteren Gerät installieren; 30-s-Clip mehrfach erzeugen | Kein Absturz, Bedienung flüssig |
| AirPlay-Test | Spiegelung auf den Trainings-TV im Hoch- und Querformat | Bild und Schrift gut lesbar |
| Zeichentest | Linien ziehen, Clip durchspulen, Gerät drehen | Zeichnung bleibt an der Bildstelle |
| Spultest | An der Zeitleiste ziehen und Bild für Bild vor- und zurückschalten | Bild folgt ohne spürbare Verzögerung (unter etwa 100 ms) |

## 8. Offene Fragen und Entscheidungen

Diese Punkte sind im Spec mit einer Annahme vorbelegt und sollten vor M1 bestätigt werden.

- [ ] **Cliplänge:** Reichen maximal 30 s? (Annahme: ja, wählbar 5 bis 30 s)
- [ ] **Aufnahme im Clip-Modus:** Soll sie pausieren und beim Zurückkehren neu starten? (Annahme: ja, spart Speicher und Rechenzeit)
- [ ] **Zeichnen:** Gelten beide Varianten, "nur dieses Bild" und "ganzer Clip"? (Annahme: ja, je Zeichnung wählbar)
- [ ] **Screenshot:** Soll ein Standbild mit Zeichnung in die Fotos gespeichert werden können? (Annahme: nein, widerspricht "nicht ablegen")
- [ ] **Gerät:** Welches iPhone-Modell und welche iOS-Version werden im Training genutzt? Davon hängt die Speichergrenze für den Clip ab.
- [ ] **Hosting:** Neues GitHub-Repository oder Netlify, damit die bestehende Seite unberührt bleibt? (Annahme: neues Repository)
