# gateway

**Version:** 0.1.0 · Änderungen: [CHANGELOG.md](CHANGELOG.md)

ESPHome-Knoten. Minimalistischer Bluetooth-Gateway: empfängt BLE-Geräte in der
Umgebung und leitet sie als Bluetooth-Proxy an Home Assistant weiter.

## Stand

Grundgerüst. Die Konfiguration bringt den Knoten ins WLAN, verbindet ihn mit
Home Assistant und schaltet den Bluetooth-Proxy mit aktiven Verbindungen ein.

## Hardware

Noch offen. In `gateway.yaml` steht vorläufig ein generisches ESP32-Board
(`esp32dev`). Sobald die Hardware feststeht, werden hier Board, Antenne,
Stromversorgung und ggf. Ethernet eingetragen.

## Inbetriebnahme

1. `secrets.yaml.example` nach `secrets.yaml` kopieren und ausfüllen.
   `secrets.yaml` ist gitignoriert.
2. Konfiguration prüfen: `esphome config gateway.yaml`
3. Flashen: `esphome run gateway.yaml`
4. In Home Assistant das gefundene ESPHome-Gerät mit dem API-Schlüssel hinzufügen.
   BLE-Geräte in Reichweite erscheinen danach unter *Einstellungen → Geräte &
   Dienste* als neu entdeckte Integrationen.
