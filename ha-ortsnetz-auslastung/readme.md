# ortsnetz-auslastung.de mit Homeassistant Boardmitteln

Diese Dateien beinhalten Konfiguration im yaml Format und sind an den relevanten Stellen kommentiert zur besseren Nachvollziehbarkeit.

Die Datein müssen an 2 Stellen in Homeassistant eingearbeitet werden, um den Automatismus zum Senden von Daten an die ortsnetz-auslastung.de API zu aktivieren.

## configuration.yaml

Diese Datei enthält die notwendigen Einrichtung des `ortsnetz-auslastung.de` API Endpunktes als sogenanntes `rest_command`, siehe [Homeassistant RESTful Command](https://www.home-assistant.io/integrations/rest_command/). 

Dieser Abschnitt muss in die globale Homeassistant-Konfiguration eingearbeitet werden. Typischerweise wird dazu eine Datei namens "configuration.yaml" bearbeitet, siehe [Homeassistant Configuration](https://www.home-assistant.io/docs/configuration/)

## ortsnetz-auslastung.yaml

Der Aktionsname lautet `rest_command.ortsnetz_auslastung_de`. Bei bestehenden Installationen den bisherigen Namen mit Bindestrichen sowohl in `configuration.yaml` als auch im Automationsaufruf durch Unterstriche ersetzen. Die API-URL bleibt unverändert. Anschließend die RESTful-Command-Konfiguration und die Automation neu laden oder Home Assistant neu starten.

Diese Datei enthält die Beschreibung eines Automatismus. Der Inhalt kann vollständig in den yaml-Editor einer neuen Automatisierung copy/pasted werden. 

Über die Homeassistant Web-Ansicht:

`Einstellungen > Automtisierungen > Automatisierung erstellen > neue Automatisierung > 3 Punkte Menü (⋮) > YAML bearbeiten`.

Nachdem die Inhalte eingefügt wurden, stehen die üblichen UI Elemente zur Einrichtung zur Verfügung.

## Tests

Mit installiertem Home Assistant und PyYAML prüft `python3 test_templates.py` die beiden YAML-Dateien gegen die Home-Assistant-Schemas und kontrolliert, dass Definition und Aufruf denselben Aktionsnamen verwenden.
