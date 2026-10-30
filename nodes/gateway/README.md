# gateway

**Version:** 0.2.0 · Änderungen: [CHANGELOG.md](CHANGELOG.md)

ESPHome-Knoten. Minimalistischer Bluetooth-Gateway: empfängt BLE-Geräte in der
Umgebung und leitet sie als Bluetooth-Proxy an Home Assistant weiter.

## Stand

Läuft auf dem XIAO ESP32-C6 und ist in Home Assistant eingebunden. Ausgelegt
für mehrere Gateways gleicher Bauart, z. B. einer pro Stockwerk. Jeder Gateway
meldet sich in Home Assistant als eigener Bluetooth-Adapter. BLE-Geräte in
Reichweite erscheinen als neu entdeckte Integrationen, Home Assistant wählt pro
Gerät den Gateway mit dem besten Empfang. Dazu kommen die Diagnose-Entitäten
aus `common/diagnostics.yaml` und ein Fallback-Hotspot.

Kontrolle in Home Assistant: *Einstellungen → Geräte & Dienste → Bluetooth →
bt-gateway-<id> → Konfigurieren* zeigt die empfangenen BLE-Geräte
(Advertisement Monitor) und die belegten Verbindungsplätze.

## Gateways

| Gateway            | WLAN-MAC          | Antenne | Standort |
|--------------------|-------------------|---------|----------|
| `bt-gateway-47b10` | <MAC> | extern  | offen    |

## Aufbau der Konfiguration

| Datei im Repo                   | Im Add-on                  | Inhalt                                   |
|---------------------------------|----------------------------|------------------------------------------|
| `gateway.yaml`                  | `common/bt-gateway.yaml`   | gemeinsamer Baustein für alle Gateways   |
| `instanzen/bt-gateway-<id>.yaml`| `bt-gateway-<id>.yaml`     | pro Gateway: Name, Antenne, Schlüssel    |

- **Name:** `bt-gateway-` plus die letzten 5 Stellen der WLAN-MAC, in
  Kleinbuchstaben. Beispiel: MAC <MAC> ergibt `bt-gateway-47b10`.
- **Antenne:** Substitution `antenne` in der Instanzdatei, `extern` (U.FL) oder
  `intern` (Keramik). Ein anderer Wert lässt die Prüfung mit einer Meldung
  abbrechen.
- **Reihenfolge der Packages:** zuerst `diagnostics`, dann `gateway`. Spätere
  Packages überschreiben frühere, so gelten die Heap-Grenzwerte des Bausteins.
- Eine Änderung an `gateway.yaml` gilt für alle Gateways. Danach jeden Gateway
  neu flashen.

## Hardware

| Teil           | Wahl                                              |
|----------------|---------------------------------------------------|
| Board          | Seeed Studio XIAO ESP32-C6 (`seeed_xiao_esp32c6`) |
| Speisung       | USB-C, z. B. ein beliebiges USB-Netzteil          |
| Antenne        | externe Antenne am U.FL-Anschluss                 |
| Weitere Teile  | keine ausser der Antenne                          |

Es gibt keine Verdrahtung: Board ans USB-Netzteil, fertig.

Das Board hat eine eingebaute Keramikantenne und einen U.FL-Anschluss für eine
externe Antenne. Gewählt wird pro Gateway mit der Substitution `antenne` in der
Instanzdatei (Vorgabe: `extern`). Die Pins setzt `gateway.yaml` beim Start:

| Pin    | Pegel                          | Bedeutung                   |
|--------|--------------------------------|-----------------------------|
| GPIO3  | LOW                            | HF-Schalter eingeschaltet   |
| GPIO14 | HIGH bei `extern`, LOW bei `intern` | Antennenwahl           |

Gemessen am selben Standort (WLAN-Pegel, gleicher Access Point):

| Einstellung                     | Pegel           |
|---------------------------------|-----------------|
| Pins nicht gesetzt (bis 0.2.0)  | -73 bis -79 dBm |
| eingebaute Antenne, fest        | -54 bis -56 dBm |
| externe Antenne, fest           | -52 bis -55 dBm |

## Neuen Gateway hinzufügen

1. **MAC auslesen:** Board per USB-C anstecken. Die WLAN-MAC zeigt z. B.
   `esptool read-mac` (die Basis-MAC ist die WLAN-MAC) oder das Log beim ersten
   Start. Daraus die `<id>` bilden: letzte 5 Stellen, Kleinbuchstaben.
2. **Instanzdatei:** `instanzen/bt-gateway-47b10.yaml` nach
   `instanzen/bt-gateway-<id>.yaml` kopieren. Name, Kommentarzeile,
   `friendly_name`, `antenne` und die drei Secret-Namen anpassen.
3. **Secrets:** in der `secrets.yaml` des Add-ons einen Block
   `bt_gateway_<id>_*` anlegen (Vorlage: `secrets.yaml.example`), mit eigenem
   API-Schlüssel und eigenem Hotspot-Passwort.
4. **Ins Add-on kopieren:** die Instanzdatei als `bt-gateway-<id>.yaml`.
5. **Prüfen und flashen:** `esphome config bt-gateway-<id>.yaml`, dann beim
   ersten Mal per USB-C `esphome run bt-gateway-<id>.yaml`. Erkennt der PC das
   Board nicht, BOOT gedrückt halten, RESET kurz drücken. Danach per OTA.
6. **Home Assistant:** das gefundene ESPHome-Gerät hinzufügen. Den Schlüssel
   holt Home Assistant aus dem Add-on.
7. **Tabelle oben** um den neuen Gateway ergänzen.

Benötigt wird ESPHome 2026.10.1 oder neuer.

## Betrieb im ESPHome-Add-on

Im Livesystem läuft die Konfiguration im ESPHome-Add-on von Home Assistant.
Massgebend sind die Dateien in diesem Repo, nach jeder Änderung werden sie ins
Add-on-Verzeichnis (`config/esphome/`) kopiert (Zuordnung siehe *Aufbau der
Konfiguration*).

- Schlüssel in der `secrets.yaml` des Add-ons, pro Gateway:
  `bt_gateway_<id>_api_key`, `bt_gateway_<id>_fallback_ap_ssid` und
  `bt_gateway_<id>_fallback_ap_password`. Dazu `wifi_ssid`, `wifi_password` und
  `mac_bssid_*`, die alle Geräte gemeinsam nutzen. Der API-Schlüssel sichert
  auch OTA, ein OTA-Passwort gibt es nicht.
- Fallback-Hotspot: Findet ein Gateway das WLAN nicht, öffnet er den Hotspot
  `bt-gateway-<id>`. Darüber lassen sich im Captive Portal neue WLAN-Daten
  eingeben.
- Diagnose: eingebunden wird `common/diagnostics.yaml` (Version 1.4.1, 5 Access
  Points) aus dem Add-on-Verzeichnis, wie bei allen anderen Geräten (Original
  im Repo `esphome_apps`). Die Datei liegt nicht in diesem Repo.
- Lokal prüfen im Spiegel `home_assistant/ESPHome`, wo `common/` und
  `secrets.yaml` nebeneinanderliegen: Baustein als `common/bt-gateway.yaml` und
  Instanzdatei dorthin kopieren, dann `esphome config bt-gateway-<id>.yaml`.
