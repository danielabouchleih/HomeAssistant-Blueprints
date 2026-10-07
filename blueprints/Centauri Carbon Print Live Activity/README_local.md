# Live Activity – lokaler Countdown

## Umstellung
1. Bisherige Live-Activity-Automation deaktivieren und ihre Activity in der Companion App beenden.
2. `printer_live_activity_local.yaml` nach `/config/blueprints/automation/custom/printer_live_activity_local.yaml` kopieren. Beim Docker-Setup den auf `/config` gemounteten Ordner verwenden.
3. Unter Settings → Automations & scenes → Blueprints den neuen Blueprint auswählen und eine Automation erstellen. Alternativ das mitgelieferte CC-Beispiel in eine neue Automation unter Edit in YAML einfügen.
4. Save → Run actions beim laufenden Druck. Nach standardmäßig 20 Sekunden erfolgt der Versand.

Die Datei ist nicht öffentlich gehostet. Import blueprint per URL erfordert einen eigenen Gist/erreichbaren Blueprint-Link. Keine Änderung der configuration.yaml nötig.

## Anzeige und Daten
- Blaues Druckersymbol, Titel, lokaler Countdown und nativer Zeitbalken.
- Keine `progress`/`progress_max`-Felder: Diese würden einen statischen Sensorbalken statt des lokalen Timerbalkens erzeugen.
- Kein Prozent-Sensor erforderlich. Endzeit ist erforderlich, Startzeit und Dateiname optional für Texte.
- HA sendet die Endzeit als absoluten Unix-Zeitpunkt (`when_relative: false`), damit eine verspätete Zustellung nicht die Restzeit verlängert.
- Alle 5 Minuten sowie bei Statuswechseln/HA-Neustart wird ein neuer Stand geschickt; Intervall ist konfigurierbar. Keine sekündlichen Push-Nachrichten.
- Wiederverbindung allein garantiert keinen sofortigen Versand. Das nächste zugestellte Update korrigiert den Timer.
- Pause/Fehler schaltet den Timer nach Empfang auf einen pflegbaren Nachrichtentext um. Fortsetzen mit gültiger neuer Endzeit aktiviert ihn wieder.
- Ungültige Endzeit oder online bereits überschrittene Endzeit zeigt eine pflegbare Hinweismeldung. Der Status des Druckers entscheidet über Abschluss, nicht der Timer.
- Abschluss/Abbruch entfernt die Activity auch ohne gültige Zeitstempel. Die optionale normale Abschlussnachricht bleibt verfügbar.

## Wichtige Grenze gegenüber der ursprünglichen Idee
Der native Countdown-Balken wird in der geprüften App-Implementierung aus einem App-seitigen Zeitintervall bis zur Endzeit gezeichnet. Eine frei vorgegebene ursprüngliche Druckstartzeit wird durch die dokumentierten Countdown-Felder nicht übergeben. Deshalb ist dieser Balken KEIN verlässlicher Gesamtfortschritt `(jetzt - Druckbeginn) / (Ende - Druckbeginn)`. Bei Updates oder Neudarstellung kann er neu ansetzen. Mein vorheriger Hinweis auf eine sichere Korrektur des gesamten Fortschritts war zu weitgehend.

Der Countdown selbst läuft offline weiter. Offline erkennt das iPhone keine Pause, keinen Fehler und keinen Abschluss. Nach Ablauf kann der native Timer aufwärts zählen, bis ein Update die Anzeige korrigiert/entfernt. iOS beendet einzelne Activities spätestens nach acht Stunden. Ein späteres Update kann neu starten; das ist nicht garantiert.

Der Countdown ersetzt auf iOS den Nachrichtentext; Start/Ende und Dateiname werden daher nicht gleichzeitig in dieser Zeile gezeigt. Schriftgröße/Modellbild sind nicht konfigurierbar. Der Titel bleibt zur Kompatibilität statisch. Farben, Statuswerte, Empfänger und Nachrichten bleiben im Blueprint-Formular pflegbar. Eine feste Kennung und mode: single vermeiden parallele Läufe, garantieren aber keine Fehlerfreiheit des iOS-Push-/Token-Systems.

## Validierung
Lokal geprüft: YAML, Eingabereferenzen und Jinja-Vorlagen sowie Countdown-Payload, Pause, ungültige/überschrittene Endzeit, korrigierte Endzeit und Abschluss. Kein Test auf dem konkreten HA/iPhone.

Quellen:
- https://companion.home-assistant.io/docs/notifications/live-activities/
- https://github.com/home-assistant/iOS/blob/main/Sources/Extensions/Widgets/LiveActivity/HAActivityTimerProgressBar.swift
