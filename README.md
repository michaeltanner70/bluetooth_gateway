# bluetooth_gateway

Minimalistischer Bluetooth-Gateway auf ESP32-Basis für Home Assistant.
Der Gateway empfängt Bluetooth-LE-Geräte in seiner Umgebung (Sensoren,
Thermometer, Tracker …) und reicht sie per ESPHome-Bluetooth-Proxy an
Home Assistant weiter.

## Aufbau des Repos

```
docs/              Architektur, Konventionen
shared/esphome/    Wiederverwendbare ESPHome-Pakete
nodes/<knoten>/    Ein Ordner pro Gateway bzw. Hardware-Variante
```

Verbindliche Abmachungen (Versionierung, Pflichtdateien): [projekt.md](projekt.md).

Jeder Knoten ist in sich abgeschlossen und hat ein eigenes README.
Wie ein Knotenordner aufgebaut ist, steht in [docs/knoten.md](docs/knoten.md).
