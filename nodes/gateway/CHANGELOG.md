# Changelog

Alle Änderungen an diesem Knoten. Aufbau nach
[Keep a Changelog](https://keepachangelog.com/de/1.1.0/), Versionierung nach
[projekt.md](../../projekt.md).

## [Unveröffentlicht]

## [1.0.0] – 2026-10-31

Erste stabile Version. Zwei Gateways laufen parallel in Home Assistant
(`bt-gateway-47b10` mit externer, `bt-gateway-a8fc4` mit eingebauter Antenne).
Ab hier gilt die MAJOR-Version nach [projekt.md](../../projekt.md) als Stand
des Gesamtsystems.

### Hinzugefügt

- Zweiter Gateway `bt-gateway-a8fc4` (WLAN-MAC <MAC>) mit
  eingebauter Antenne, Instanzdatei `instanzen/bt-gateway-a8fc4.yaml`.
- README: Schritt-für-Schritt-Anleitung „Neuen Gateway hinzufügen“, so wie der
  zweite Gateway angelegt wurde, mit Befehlen, Prüfpunkten und Stolpersteinen.
- Status-LED (gelbe Benutzer-LED an GPIO15): blinkt 0,5 s an / 0,5 s aus,
  solange WLAN oder Home Assistant (API) nicht verbunden sind. Sind beide
  verbunden, blitzt sie jede Sekunde 100 ms. Eine Anzeige für empfangene
  BLE-Pakete wurde getestet und wieder verworfen: Bei vielen Geräten in
  Reichweite flackerte sie fast dauernd und sagte nichts aus.

## [0.3.0] – 2026-10-30

Ausgelegt für mehrere Gateways, Antennenwahl fest gesetzt. Secret-Namen
geändert: bestehende `secrets.yaml` müssen angepasst werden.

### Hinzugefügt

- Mehrere Gateways gleicher Bauart: `gateway.yaml` ist jetzt der gemeinsame
  Baustein (im Add-on `common/bt-gateway.yaml`), pro Gateway gibt es eine
  Instanzdatei unter `instanzen/` mit Name, Antenne und Schlüsseln.
- Antennenwahl pro Gateway über die Substitution `antenne` (`extern` oder
  `intern`). Ein ungültiger Wert lässt die Prüfung abbrechen.

### Geändert

- Gerätename `bt-gateway-<id>` mit den letzten 5 Stellen der WLAN-MAC. Der
  bestehende Gateway heisst jetzt `bt-gateway-47b10`. In Home Assistant bleiben
  Integration und Entitäts-IDs erhalten.
- Secrets pro Gateway: `bt_gateway_<id>_api_key`,
  `bt_gateway_<id>_fallback_ap_ssid`, `bt_gateway_<id>_fallback_ap_password`
  statt `bt_gateway_*`.
- Antennenwahl fest gesetzt statt dem Startzustand überlassen: GPIO3 LOW
  (HF-Schalter ein), GPIO14 HIGH (externe Antenne am U.FL-Anschluss). Beide
  als interne GPIO-Schalter. Der WLAN-Pegel stieg dadurch am selben Standort
  von -73…-79 dBm auf -52…-55 dBm, die eingebaute Antenne kam auf -54…-56 dBm.

## [0.2.0] – 2026-10-30

Erster lauffähiger Stand auf der Zielhardware. Geflasht und in Home Assistant
eingebunden, BLE-Geräte (z. B. Shelly BLU Door/Window Mini ZB über BTHome)
kommen über den Gateway an.

### Hinzugefügt

- Gemeinsames Diagnose-Package `common/diagnostics.yaml` eingebunden
  (WLAN, Speicher, Laufzeit, Versionen, Neustart-Tasten), Heap-Grenzwerte für
  den ESP32-C6 auf 80 000 / 40 000 Bytes gesetzt. Verwendet wird Version
  1.4.1 mit der Zuordnung der 5 Access Points aus 1.2.1.
- Fallback-Hotspot `bt-gateway` mit Captive Portal, falls das WLAN nicht
  erreichbar ist. Neue Secrets `bt_gateway_fallback_ap_ssid` und
  `bt_gateway_fallback_ap_password`.

### Geändert

- Hardware festgelegt: Seeed Studio XIAO ESP32-C6, Speisung über USB-C, keine
  weiteren Teile. `gateway.yaml` von `esp32dev` auf `seeed_xiao_esp32c6`
  umgestellt.
- OTA verschlüsselt statt mit Passwort gesichert (spart Flash und RAM). Der
  Schlüssel ist derselbe wie für die API, `ota_password` entfällt in
  `secrets.yaml`. Geprüft mit ESPHome 2026.10.1.
- Secret `api_key_gateway` umbenannt in `bt_gateway_api_key`, passend zur
  Benennung `<gerät>_api_key` in der `secrets.yaml` des ESPHome-Add-ons.

## [0.1.0] – 2026-10-30

Grundgerüst.

### Hinzugefügt

- `gateway.yaml`: ESPHome-Grundkonfiguration mit WLAN, Home-Assistant-API
  (verschlüsselt), OTA, Logger und Bluetooth-Proxy (aktiv), Board vorläufig
  `esp32dev`
- Vorlage `secrets.yaml.example`
