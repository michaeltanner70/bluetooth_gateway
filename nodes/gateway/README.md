# gateway

**Version:** 0.2.0 · Änderungen: [CHANGELOG.md](CHANGELOG.md)

ESPHome-Knoten. Minimalistischer Bluetooth-Gateway: empfängt BLE-Geräte in der
Umgebung und leitet sie als Bluetooth-Proxy an Home Assistant weiter.

## Stand

Läuft auf dem XIAO ESP32-C6 und ist in Home Assistant eingebunden. Der Gateway
meldet sich dort als Bluetooth-Adapter `bt-gateway`. BLE-Geräte in Reichweite
erscheinen als neu entdeckte Integrationen. Dazu kommen die Diagnose-Entitäten
aus `common/diagnostics.yaml` und ein Fallback-Hotspot.

Kontrolle in Home Assistant: *Einstellungen → Geräte & Dienste → Bluetooth →
bt-gateway → Konfigurieren* zeigt die empfangenen BLE-Geräte (Advertisement
Monitor) und die belegten Verbindungsplätze.

## Hardware

| Teil           | Wahl                                              |
|----------------|---------------------------------------------------|
| Board          | Seeed Studio XIAO ESP32-C6 (`seeed_xiao_esp32c6`) |
| Speisung       | USB-C, z. B. ein beliebiges USB-Netzteil          |
| Antenne        | eingebaute Keramikantenne des Boards              |
| Weitere Teile  | keine                                             |

Es gibt keine Verdrahtung: Board ans USB-Netzteil, fertig.

Das Board hat zusätzlich einen U.FL-Anschluss für eine externe Antenne. Er ist
nicht in Gebrauch. Umgeschaltet würde über GPIO14 (Antennenwahl), GPIO3
schaltet den HF-Schalter ein.

## Inbetriebnahme

1. `secrets.yaml.example` nach `secrets.yaml` kopieren und ausfüllen.
   `secrets.yaml` ist gitignoriert. Der API-Schlüssel sichert auch OTA, ein
   OTA-Passwort gibt es nicht. Benötigt wird ESPHome 2026.10.1 oder neuer.
2. Konfiguration prüfen: `esphome config gateway.yaml` (braucht
   `common/diagnostics.yaml` daneben, siehe *Betrieb im ESPHome-Add-on*)
3. Flashen: `esphome run gateway.yaml`. Beim ersten Mal per USB-C. Erkennt der
   PC das Board nicht, BOOT-Taste gedrückt halten und RESET kurz drücken.
   Danach geht es per OTA.
4. In Home Assistant das gefundene ESPHome-Gerät mit dem API-Schlüssel hinzufügen.
   BLE-Geräte in Reichweite erscheinen danach unter *Einstellungen → Geräte &
   Dienste* als neu entdeckte Integrationen.

## Betrieb im ESPHome-Add-on

Im Livesystem läuft die Konfiguration im ESPHome-Add-on von Home Assistant:

- Datei: `bt-gateway.yaml` im Add-on-Verzeichnis (`config/esphome/`), eine
  Kopie von `gateway.yaml`. Massgebend ist die Datei in diesem Repo, nach
  jeder Änderung wird sie dorthin kopiert.
- Schlüssel in der `secrets.yaml` des Add-ons: `bt_gateway_api_key`,
  `bt_gateway_fallback_ap_ssid` und `bt_gateway_fallback_ap_password`, dazu
  `wifi_ssid`, `wifi_password` und `mac_bssid_*`, die alle Geräte gemeinsam
  nutzen.
- Fallback-Hotspot: Findet der Gateway das WLAN nicht, öffnet er den Hotspot
  `bt-gateway`. Darüber lassen sich im Captive Portal neue WLAN-Daten eingeben.
- Diagnose: eingebunden wird `common/diagnostics.yaml` (Version 1.4.1, 5 Access
  Points) aus dem Add-on-Verzeichnis, wie bei allen anderen Geräten (Original
  im Repo `esphome_apps`). Die Datei liegt nicht in diesem Repo. Lokal prüfen deshalb im Spiegel `home_assistant/ESPHome`, wo
  `common/` und `secrets.yaml` nebeneinanderliegen.
