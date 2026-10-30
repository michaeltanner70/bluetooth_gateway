# Architektur

```
BLE-Geräte (Sensoren, Thermometer, Tracker …)
        │  Bluetooth LE (Advertisements, aktive Verbindungen)
        ▼
Gateway (ESP32, ESPHome Bluetooth-Proxy) ──WLAN/LAN──► Home Assistant
```

## Prinzip

Der Gateway wertet selbst nichts aus. Er hört Bluetooth LE mit und leitet die
Rohdaten über die ESPHome-API an Home Assistant weiter. Erkannt und dekodiert
werden die Geräte von den Bluetooth-Integrationen in Home Assistant
(z. B. BTHome, Xiaomi BLE, Govee BLE, SwitchBot).

Damit bleibt die Firmware klein und muss für neue Geräte nicht angepasst werden.

## Rollen

| Rolle    | Firmware | Anbindung                                      |
|----------|----------|------------------------------------------------|
| Gateway  | ESPHome  | BLE zu den Geräten, WLAN/LAN zu Home Assistant |

Mehrere Gateways an verschiedenen Orten erhöhen die Reichweite. Home Assistant
wählt pro Gerät automatisch den Gateway mit dem besten Empfang.

## Grundsätze

- **Minimalistisch:** nur `esp32_ble_tracker` und `bluetooth_proxy`, keine
  gerätespezifische Logik in der Firmware.
- **Aktive Verbindungen** (`active: true`) sind eingeschaltet, damit auch
  Geräte funktionieren, die eine Verbindung brauchen (z. B. Schlösser,
  Pflanzensensoren).
- **Framework `esp-idf`:** von ESPHome für den Bluetooth-Proxy empfohlen,
  braucht weniger Speicher als Arduino.
- **Kabel vor Funk:** wo möglich per Ethernet anbinden, weil WLAN und
  Bluetooth sich auf dem ESP32 die Antenne teilen.
