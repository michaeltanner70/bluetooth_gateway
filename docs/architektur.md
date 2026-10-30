# Architektur

```
BLE-Geräte (Sensoren, Thermometer, Tracker …)
        │  Bluetooth LE (Advertisements, aktive Verbindungen)
        ▼
Gateway (XIAO ESP32-C6, ESPHome Bluetooth-Proxy) ──WLAN──► Home Assistant
        ▲
        └── USB-C (Speisung)
```

## Hardware

Ein **Seeed Studio XIAO ESP32-C6**, sonst nichts: keine Platine, keine
Zusatzteile, Speisung über USB-C. Der Gateway kann damit an jedem USB-Netzteil
oder USB-Anschluss betrieben werden.

## Prinzip

Der Gateway wertet selbst nichts aus. Er hört Bluetooth LE mit und leitet die
Rohdaten über die ESPHome-API an Home Assistant weiter. Erkannt und dekodiert
werden die Geräte von den Bluetooth-Integrationen in Home Assistant
(z. B. BTHome, Xiaomi BLE, Govee BLE, SwitchBot).

Damit bleibt die Firmware klein und muss für neue Geräte nicht angepasst werden.

## Rollen

| Rolle    | Firmware | Anbindung                                      |
|----------|----------|------------------------------------------------|
| Gateway  | ESPHome  | BLE zu den Geräten, WLAN zu Home Assistant     |

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
- **Keine Zusatzhardware:** alles, was der Gateway braucht, ist auf dem
  XIAO-Board. WLAN und Bluetooth teilen sich dabei Funkteil und Antenne,
  ESPHome regelt das über Software-Koexistenz.
- **Standort:** zentral zwischen den BLE-Geräten und mit gutem WLAN-Empfang
  aufstellen, nicht in Metallgehäuse oder direkt neben den Router.
